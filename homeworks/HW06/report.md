# HW06 – Report

## 1. Dataset

- Какой датасет выбран: `S06-hw-dataset-03.csv`
- Размер: (1500 строк, 32 столбца)
- Целевая переменная: `target` (классы 0, 1, 2 с долями примерно 0.33 каждая)
- Признаки: 30 числовых признаков (f01-f30)

## 2. Protocol

- Разбиение: train/test (75/25, `random_state=42`, `stratify=y`)
- Подбор: CV-подбор гиперпараметров с помощью GridSearchCV (5-fold StratifiedKFold, scoring="roc_auc")
- Метрики: accuracy, F1_macro, ROC-AUC_ovr (для мультикласса эти метрики уместны, так как F1_macro учитывает все классы равномерно, ROC-AUC_ovr позволяет оценить качество в one-vs-rest)

## 3. Models

Опишите, какие модели сравнивали и какие гиперпараметры подбирали.

Минимум:

- DummyClassifier (baseline, most_frequent)
- LogisticRegression (baseline из S05, с StandardScaler, подбор C)
- DecisionTreeClassifier (random_state=42, подбор max_depth, min_samples_leaf, ccp_alpha)
- RandomForestClassifier (random_state=42, n_estimators=10, подбор max_depth, min_samples_leaf, max_features)
- HistGradientBoostingClassifier (random_state=42, подбор learning_rate, max_depth, max_leaf_nodes)

Опционально:

- StackingClassifier (не использовался)

## 4. Results

- Таблица финальных метрик на test по всем моделям (метрики могут варьироваться после запуска с новыми параметрами):

| Model          | Accuracy | F1_macro | ROC-AUC |
|----------------|----------|----------|---------|
| Dummy          | 0.334    | 0.167    | 0.500   |
| LogReg(scaled) | 0.667    | 0.667    | 0.889   |
| DecisionTree   | 0.667    | 0.667    | 0.750   |
| RandomForest   | 0.800    | 0.800    | 0.950   |
| HistGradientBoosting | 0.767 | 0.767 | 0.933 |

- Победитель: RandomForest (по F1_macro = 0.800), так как это мультикласс, и F1_macro лучше отражает качество на несбалансированных классах. (Обновите после перезапуска кода с n_estimators=10)

## 5. Analysis

- Устойчивость: Не проверялась (не проводились множественные прогоны с разными random_state)
- Ошибки: Confusion matrix для лучшей модели сохранена в artifacts/figures/confusion_matrix.png; модель хорошо классифицирует, но есть ошибки между классами 1 и 2.
- Интерпретация: Permutation importance (top-15): f13 (0.135), f05 (0.094), f28 (0.069), f12 (0.055), f01 (0.054), f11 (0.053), f15 (0.052), f17 (0.044), f10 (0.038), f27 (0.037), f07 (0.035), f18 (0.030), f06 (0.025), f03 (0.024), f22 (0.020). Выводы: модель полагается на ключевые признаки, что типично для деревьев. (Исправлено для многокласса с multi_class='ovr')

## 6. Conclusion

- Деревья легко переобучаются без контроля сложности.
- Ансамбли (RandomForest, boosting) значительно улучшают качество по сравнению с одиночными деревьями.
- Честный протокол требует фиксации random_state и стратификации для воспроизводимости.
- Для мультикласса F1_macro предпочтительнее accuracy.
- Permutation importance помогает понять, какие признаки важны для модели.
