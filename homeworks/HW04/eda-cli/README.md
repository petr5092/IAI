# HW04 – eda_cli: HTTP-сервис качества датасетов

Небольшое CLI-приложение для базового анализа CSV-файлов с HTTP-сервисом на FastAPI.
Используется в рамках Семинара 04 курса «Инженерия ИИ».
Проект опирается на результаты HW03 и переиспользует доработанный eda-cli.

## Требования

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) установлен в систему

## Инициализация проекта

В корне проекта (HW04/eda-cli):

```bash
uv sync
```

Эта команда:

- создаст виртуальное окружение `.venv`;
- установит зависимости из `pyproject.toml`;
- установит сам проект `eda-cli` в окружение.

## Запуск CLI

### Краткий обзор

```bash
uv run eda-cli overview data/example.csv
```

Параметры:

- `--sep` – разделитель (по умолчанию `,`);
- `--encoding` – кодировка (по умолчанию `utf-8`).

### Полный EDA-отчёт

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

В результате в каталоге `reports/` появятся:

- `report.md` – основной отчёт в Markdown;
- `summary.csv` – таблица по колонкам;
- `missing.csv` – пропуски по колонкам;
- `correlation.csv` – корреляционная матрица (если есть числовые признаки);
- `top_categories/*.csv` – top-k категорий по строковым признакам;
- `hist_*.png` – гистограммы числовых колонок;
- `missing_matrix.png` – визуализация пропусков;
- `correlation_heatmap.png` – тепловая карта корреляций.

Дополнительные опции команды `report`:

- `--title`: заголовок, который будет записан в `report.md` (по умолчанию `Report`).
- `--top-k-categories`: сколько топовых значений сохранять для каждой категориальной/строковой колонки в папке `top_categories/` (по умолчанию `5`).

Эти опции влияют на содержимое отчёта:

- `--title` меняет заголовок в `report.md` и позволяет пометить отчёт удобным именем.
- `--top-k-categories` задаёт глубину списка популярных значений в CSV-файлах внутри `top_categories/` и также отражается в тексте `report.md`.

Пример вызова с новыми опциями:

```powershell
uv run eda-cli report data/example.csv --out-dir reports_example --title "Sales report" --max-hist-columns 4 --top-k-categories 10
```

## Запуск API

Для запуска веб-сервиса используйте:

```bash
uv run uvicorn eda_cli.api:app --reload --port 8000 
```



### Доступные эндпоинты

- `GET /health` - Проверка здоровья сервиса
- `POST /quality` - Оценка качества датасета по агрегированным признакам
- `POST /quality-from-csv` - Оценка качества по загруженному CSV-файлу
- `POST /quality-flags-from-csv` - Получение флагов качества по CSV-файлу


## Тесты

```bash
uv run pytest -q
```
