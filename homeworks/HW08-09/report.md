# HW08-09 – PyTorch MLP: регуляризация и оптимизация обучения

## 1. Кратко: что сделано

- Какой датасет выбран (A/B/C) и почему.
- Что сравнивалось в части A (регуляризация) и в части B (оптимизация).

# HW08-09 – PyTorch MLP: регуляризация и оптимизация обучения

## 1. Кратко: что сделано

- Датасет: EMNIST (balanced) — выбор обусловлен задачей классификации символов и умеренной размерностью изображений.
- Часть A (регуляризация): сравнение конфигураций E1–E4 (baseline, Dropout, BatchNorm, EarlyStopping).
- Часть B (оптимизация): исследование эффекта LR и оптимизатора O1–O3 (экстремальные LR для Adam и SGD+momentum).

## 2. Среда и воспроизводимость

- Python: смотреть `HW08-09.ipynb` (точная версия в среде разработки).
- torch / torchvision: смотреть окружение (`torch.__version__` в ноутбуке).
- Устройство: cpu (как указано в `artifacts/best_config.json`).
- Seed: 42
- Как запустить: открыть `HW08-09.ipynb` и выполнить Run All — все основные результаты сохранены в `artifacts`.

## 3. Данные

- Датасет: `torchvision.EMNIST(split='balanced')` — 28×28, 47 классов.
- Разделение: train → (train/val), test — стандартный test-split из torchvision (см. ноутбук).
- Трансформации: ToTensor + нормализация (детали в ноутбуке).
- Комментарий: многоклассовая задача с небольшими изображениями; MLP применим как базовая модель для сравнений регуляризации и оптимизации.

## 4. Базовая модель и обучение

- Модель MLP: 2 скрытых слоя (256, 128), активация ReLU.
- Loss: CrossEntropyLoss.
- Базовый Optimizer: Adam (lr=0.001) для большинства прогонов.
- Batch size: см. `HW08-09.ipynb`.
- Epochs (макс): 50, EarlyStopping применялся в E4 (patience=4, min_delta=0.0005).

## 5. Часть A (S08): регуляризация (E1-E4)

- E1 (base): hidden=(256,128), relu, dropout=0.0, batchnorm=False — best_val_accuracy=0.77295, best_val_loss=0.76438.
- E2 (Dropout): как E1 + Dropout(p=0.3) — best_val_accuracy=0.76250, best_val_loss=0.73298.
- E3 (BatchNorm): как E1 + BatchNorm — best_val_accuracy=0.77998, best_val_loss=0.73672.
- E4 (Dropout + EarlyStopping): Dropout(0.3) + EarlyStopping — epochs_trained=25, best_val_accuracy=0.77334, best_val_loss=0.71316.

## 6. Часть B (S09): LR, оптимизаторы, weight decay (O1-O3)

- O1 (Adam, lr=0.1): слишком большой lr → быстрый спад качества, best_val_accuracy=0.13496, best_val_loss=3.5803.
- O2 (Adam, lr=1e-5): слишком маленький lr → почти отсутствие обучения, best_val_accuracy=0.02441, best_val_loss=3.8551.
- O3 (SGD+momentum): SGD (momentum=0.9, lr=0.1) — стабильнее, но хуже Adam; best_val_accuracy=0.64990, best_val_loss=1.1839.

## 7. Результаты

Ссылки на файлы в репозитории:

- Таблица результатов: [homeworks/HW08-09/artifacts/runs.csv](homeworks/HW08-09/artifacts/runs.csv)
- Конфиг лучшей модели: [homeworks/HW08-09/artifacts/best_config.json](homeworks/HW08-09/artifacts/best_config.json)
- Лучшая модель: [homeworks/HW08-09/artifacts/best_model.pt](homeworks/HW08-09/artifacts/best_model.pt)
- Кривые лучшего прогона: [homeworks/HW08-09/artifacts/figures/curves_best.png](homeworks/HW08-09/artifacts/figures/curves_best.png)
- Кривые “плохих LR”: [homeworks/HW08-09/artifacts/figures/curves_lr_extremes.png](homeworks/HW08-09/artifacts/figures/curves_lr_extremes.png)

Короткая сводка:

- Лучший эксперимент части A: E4 выбран как финальный (Dropout 0.3 + EarlyStopping) — он даёт низкий val_loss и стабильность.
- Лучшая val_accuracy в таблице: 0.77998 (E3, BatchNorm).
- Итоговая test_accuracy: не включена в `runs.csv`; для финальной test-оценки запустить оценку `best_model.pt` на test-split.
- O1 (слишком большой LR) и O2 (слишком маленький LR) демонстрируют плохоe поведение (расходимость/отсутствие обучения).
- O3 (SGD+momentum) уступает Adam при выбранных гиперпараметрах, но остаётся опцией при дополнительном подборе.

## 8. Анализ

- BatchNorm улучшил val_accuracy (E3), вероятно за счёт более предсказуемого распределения активаций и устойчивости при обучении.
- Dropout в комбинации с EarlyStopping (E4) дал наилучший компромисс по val_loss и стабильности; ранняя остановка остановила обучение на 25-й эпохе, предотвратив дальнейшее переобучение.
- Экстремальные lr (O1/O2) показывают классические проблемы: крупный lr → расходимость/шумные градиенты, малый lr → очень медленное сходение.
- SGD+momentum при больших lr показал возможность обучения, но требовал более тщательного подбора lr/weight_decay для сравнимых результатов с Adam.
- Таким образом, выбранный финальный конфиг (E4) разумен: он стабилен и воспроизводим, и даёт хороший баланс между accuracy и loss.

## 9. Итоговый вывод

- Базовый конфиг для дальнейшего использования: E4 (MLP(256,128), ReLU, Dropout(0.3), Adam(lr=0.001), EarlyStopping, seed=42).
- Следующие шаги: (1) получить и зафиксировать `test_accuracy` для `best_model.pt`; (2) выполнить более детальный grid-search по lr и weight_decay для SGD/Adam; (3) попробовать простую сверточную архитектуру для улучшения качества на EMNIST.

## 10. Приложение (опционально)

- В `artifacts/figures/` сохранены визуализации: `curves_best.png` и `curves_lr_extremes.png`.
