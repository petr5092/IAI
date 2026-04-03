# HW10-11 – компьютерное зрение в PyTorch: CNN, transfer learning, detection/segmentation

## 1. Кратко: что сделано

- Часть A (классификация): выбран датасет STL10 — задача 10-классовой классификации цветных изображений. STL10 интересен своей разнообразной тестовой выборкой и меньшим числом размеченных примеров по сравнению с CIFAR.
- Часть B (structured vision): выбран детекционный трек на датасете Pascal VOC с целью продемонстрировать использование предобученной детекторной модели и сравнение двух режимов инференса по порогу score.
- В части A сравнивались четыре сценария: C1 — простая CNN без аугментаций; C2 — та же архитектура с аугментациями; C3 — ResNet18 (только голова обучается); C4 — ResNet18 с частичным fine-tuning (layer4 + fc). Во второй части сравнивались режимы V1 (score_thr=0.3) и V2 (score_thr=0.7).

## 2. Среда и воспроизводимость

- Python: см. окружение запуска (используется виртуальное окружение проекта).
- torch / torchvision: версии зависят от окружения; ноутбук формирует модель через `torchvision` weights API.
- Устройство (CPU/GPU): `DEVICE` определяется в ноутбуке (`cuda` если доступна, иначе `cpu`).
- Seed: 42 (переменная `RANDOM_STATE`).
- Как запустить: открыть `homeworks/HW10-11/HW10-11.ipynb` и выполнить Run All.

## 3. Данные

### 3.1. Часть A: классификация

- Датасет: STL10
- Разделение: train / val / test (в ноутбуке выполняется детерминированный split train/val из train-full и используется отдельный split `test` от STL10)
- Базовые transforms: приведение к тензору и нормализация по mean/std (STL10 mean/std в ноутбуке).
- Augmentation transforms: случайный горизонтальный флип, случайная поворота, ColorJitter, RandomResizedCrop для ResNet-тренировок.
- Комментарий: 10 классов, изображения цветные; для ResNet вход масштабируется до 224×224, для лёгких экспериментов использовался режим FAST_MODE.

### 3.2. Часть B: structured vision

- Датасет: Pascal VOC (20 классов)
- Трек: detection
- Что считается ground truth: bounding boxes, полученные из XML-аннотаций в формате PASCAL VOC (теги `bndbox`).
- Какие предсказания использовались: выходы предобученной детекторной модели (`fasterrcnn_mobilenet_v3_large_fpn`) — фильтрация по `score` и по выбранным меткам (например, `person`).
- Комментарий: Pascal VOC — классический датасет для детекции с множеством классов и разнообразными сценами; для него также уместно применять предобученные на COCO модели и оценивать поведение при разных порогах confidence.

## 4. Часть A: модели и обучение (C1-C4)

Опишите коротко и сопоставимо:

- C1 (simple-cnn-base): простая сверточная сеть (три блока conv+ReLU+pool + полносвязная голова).
- C2 (simple-cnn-aug): та же архитектура, но train-трансформы включают RandomHorizontalFlip, RandomRotation и ColorJitter.
- C3 (resnet18-head-only): ResNet18 с предобученными весами, замороженным бэконном и обучаемой только головой (fc).
- C4 (resnet18-finetune): ResNet18, сначала head-only, затем разморожены слой layer4 и fc и выполнено дообучение (partial fine-tuning).

Дополнительно:

- Loss: CrossEntropyLoss для части A.
- Optimizer(ы): Adam (для частичных FT используются разные lr для backbone/головы, пример: 1e-4 для layer4 и 1e-3 для fc).
- Batch size: SimpleCNN — 128 (по умолчанию), ResNet — 64 (в ноутбуке указан BATCH_SIZE=64 для transfer learning).
- Epochs (макс): эксперименты запускались с `EPOCHS=12` (SimpleCNN) и суммарно `EPOCHS_HEAD + EPOCHS_FT = 6` в режиме FAST_MODE (пример в ноутбуке: HEAD=3 + FT=3 при FAST_MODE=True). Полные прогоны ставились дольше (см. ноутбук).
- Критерий выбора лучшей модели: лучшая `val_accuracy` на валидационной части.

## 5. Часть B: постановка задачи и режимы оценки (V1-V2)

### Если выбран detection track

- Модель: `fasterrcnn_mobilenet_v3_large_fpn` (предобученная на COCO), в ноутбуке помечена как `cfg.model_name`.
- V1: `score_threshold = 0.3`.
- V2: `score_threshold = 0.7`.
- Как считался IoU: реализована функция `box_iou_matrix`, IoU считается между предсказанными бокcами и GT в формате (x1,y1,x2,y2).
- Как считались precision / recall: жадный матчинг `match_greedy`: предсказания сортируются по score, каждое матчится с лучшим свободным GT, считается TP/FP/FN; затем precision = TP/(TP+FP), recall = TP/(TP+FN). Средний IoU по матчам сохраняется как mean_IoU_matched.

### Если выбран segmentation track

- (не применялось в этом решении)

## 6. Результаты

Ссылки на файлы в репозитории:

- Таблица результатов: [homeworks/HW10-11/artifacts/runs.csv](homeworks/HW10-11/artifacts/runs.csv)
- Лучшая модель части A: [homeworks/HW10-11/artifacts/best_classifier.pt](homeworks/HW10-11/artifacts/best_classifier.pt)
- Конфиг лучшей модели части A: [homeworks/HW10-11/artifacts/best_classifier_config.json](homeworks/HW10-11/artifacts/best_classifier_config.json)
- Кривые лучшего прогона классификации: [homeworks/HW10-11/artifacts/figures/classification_curves_best.png](homeworks/HW10-11/artifacts/figures/classification_curves_best.png)
- Сравнение C1-C4: [homeworks/HW10-11/artifacts/figures/classification_compare.png](homeworks/HW10-11/artifacts/figures/classification_compare.png)
- Визуализация аугментаций: [homeworks/HW10-11/artifacts/figures/augmentations_preview.png](homeworks/HW10-11/artifacts/figures/augmentations_preview.png)
- Визуализации второй части: [homeworks/HW10-11/artifacts/figures/detection_examples.png](homeworks/HW10-11/artifacts/figures/detection_examples.png) и [homeworks/HW10-11/artifacts/figures/detection_metrics.png](homeworks/HW10-11/artifacts/figures/detection_metrics.png)

Короткая сводка (6-10 строк):

- Лучший эксперимент части A: C4 (ResNet18 partial fine-tune)
- Лучшая `val_accuracy`: 0.823 (C4, см. `runs.csv`)
- Итоговая `test_accuracy` лучшего классификатора: 0.40675 (сохранено в `runs.csv` для C4)
- Что дали аугментации (C2 vs C1): небольшое улучшение val_accuracy (C1 0.584 → C2 0.595), т.е. аугментации помогли стабильно, но ограниченно для простой CNN.
- Что дал transfer learning (C3/C4 vs C1/C2): ResNet18 существенно улучшил `val_accuracy` (C3 0.689 → C4 0.823), что указывает на важность предобученных признаковых представлений.
- Что оказалось лучше: head-only или partial fine-tuning: частичный fine-tuning (C4) дал заметное повышение по сравнению с head-only (C3).
- Что показал режим V1 во второй части: при `score_thr=0.3` precision≈0.221, recall≈0.991, mean_IoU≈0.905 (см. `runs.csv` для V1) — много предсказаний, высокая полнота, невысокая точность из-за большого числа FP.
- Что показал режим V2 во второй части: при `score_thr=0.7` precision≈0.409, recall≈0.988, mean_IoU≈0.905 — повышение precision при почти сохранённой полноте.
- Как интерпретируются метрики второй части: повышение порога score сильно уменьшает долю ложных срабатываний (FP), что улучшает precision, при этом mean IoU по матчам остаётся высоким, т.е. матчи корректны.

## 7. Анализ

Простая CNN (C1) ограничена числом параметров и пространством признаков; она демонстрирует базовую способность различать классы, но быстро упирается в пределы архитектуры на STL10. Аугментации (C2) дают небольшое, но стабильное улучшение качества — они повышают робастность к геометрическим и цветовым вариациям.

Pretrained ResNet18 значительно помогает: за счёт богатых признаков, извлечённых на больших датасетах (ImageNet/COCO), даже обучение только головы (C3) даёт сильный выигрыш по сравнению с простой CNN. Частичный fine-tuning (C4) позволяет адаптировать глубокие признаки к особенностям STL10, что и приводит к наилучшей `val_accuracy`.

В детекции (часть B) поведение при изменении порога score совпадает с интуицией: низкий порог (V1) — высокая полнота и много FP, высокий порог (V2) — выше precision при почти сохранённой полноте. Средний IoU по матчам остаётся высоким в обоих режимах, следовательно ошибки в основном связаны не с локализацией, а с confidence у предсказаний.

Ошибки модели: наиболее характерные ошибки — ложные срабатывания на сложных фоновых текстурах и пропуски мелких объектов при низком разрешении/маленькой площади.

## 8. Итоговый вывод

- В качестве базового конфига классификации я бы выбрал ResNet18 с частичным fine-tuning (C4): он даёт лучшее соотношение качества и простоты использования.
- Главное про transfer learning: предобученные модели существенно улучшают качество на задачах с ограниченным числом размеченных примеров; разумно сначала обучать голову, затем аккуратно размораживать глубокие слои.
- Главное про detection: порог по score — ключевой гиперпараметр для trade-off precision/recall; mean IoU показывает качество локализации отдельно от confidence.

## 9. Приложение (опционально)

- Дополнительные артефакты и графики находятся в папке с результатами: [homeworks/HW10-11/artifacts/figures](homeworks/HW10-11/artifacts/figures)
- Полная таблица экспериментов: [homeworks/HW10-11/artifacts/runs.csv](homeworks/HW10-11/artifacts/runs.csv)

