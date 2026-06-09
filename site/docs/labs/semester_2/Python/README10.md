title: Лабораторная работа №10: Разработка ML-сервиса для предсказания одобрения ипотеки

Веб-приложение для предсказания одобрения ипотечной заявки на основе данных клиента.

Проект выполнен в рамках лабораторной работы №10 по дисциплине **«Программирование на Python»** студентами:
* Ломаченко Ян Станиславович (505115)
* Богун Андрей Витальевич (505199)
* Шеберстов Арсений Алексеевич (501992)

## Тема

Разработка ML-сервиса для предсказания одобрения ипотеки.

## Цель работы

Освоить полный цикл разработки прикладного ML-сервиса:

* обработка данных;
* обучение и сравнение моделей машинного обучения;
* сериализация обученной модели;
* разработка API на FastAPI;
* разработка пользовательского интерфейса;
* интеграция frontend, backend и ML Pipeline;
* тестирование, линтинг и CI.

---

# Описание проекта

**Loan Approval Service** — это консультирующий ML-сервис, который предсказывает, будет ли ипотечная заявка клиента одобрена или отклонена.

Сервис принимает анкетные данные клиента, передаёт их в обученную ML-модель и возвращает:

* статус заявки;
* числовой код класса;
* вероятность одобрения.

Также сервис поддерживает пакетную обработку CSV-файлов и расчёт ROC-AUC, если в CSV присутствует целевая колонка `loan_status`.

---

# Основной функционал

В проекте реализовано:

* загрузка обученной модели в формате `.pkl` / `.joblib`;
* предсказание по одной или нескольким заявкам;
* обработка CSV-файла;
* расчёт ROC-AUC при наличии целевой переменной;
* сохранение полного CSV с предсказаниями;
* скачивание результата обработки CSV;
* Swagger-документация API;
* современный frontend-интерфейс;
* backend-тесты;
* backend-линтинг через Ruff;
* frontend-линтинг через ESLint;
* CI-сценарий SourceCraft.

---

# Стек технологий

## Backend

* Python 3.13
* FastAPI
* Uvicorn
* Pydantic
* Pandas
* Scikit-learn
* Joblib
* Pytest
* Ruff

## Frontend

* HTML
* CSS
* JavaScript
* ESLint

## ML

* Scikit-learn
* Pandas
* Joblib
* sklearn Pipeline
* ColumnTransformer
* StandardScaler
* OneHotEncoder
* SelectKBest
* GradientBoostingClassifier

## CI

* SourceCraft CI
* Docker images:

  * `python:3.13`
  * `node:22`

---

# Структура проекта

```text
loan-approval-service/
├── backend/
│   ├── __init__.py
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── model_service.py
│   │   └── schemas.py
│   ├── models/
│   │   └── model.pkl
│   ├── predictions/
│   │   └── predicted_loan_data.csv
│   ├── tests/
│   │   ├── __init__.py
│   │   ├── test_api.py
│   │   └── test_model.py
│   └── requirements.txt
├── data/
│   └── loan_data.csv
├── frontend/
│   ├── index.html
│   ├── script.js
│   └── style.css
├── ml/
│   ├── __init__.py
│   ├── predict.py
│   ├── preprocess.py
│   └── train.py
├── .sourcecraft/
│   └── ci.yaml
├── .gitignore
├── eslint.config.js
├── package.json
├── package-lock.json
├── pyproject.toml
└── README.md
```

---

# Данные

Для обучения используется датасет:

```text
data/loan_data.csv
```

Целевая переменная:

```text
loan_status
```

Значения целевой переменной:

```text
0 — отказ в выдаче ипотеки
1 — одобрение ипотеки
```

---

# Входные признаки модели

Модель принимает сырые, не предобработанные данные клиента.

Используемые признаки:

| Признак                          |    Тип | Описание                    |
| -------------------------------- | -----: | --------------------------- |
| `person_age`                     |  float | Возраст клиента             |
| `person_gender`                  | string | Пол клиента                 |
| `person_education`               | string | Уровень образования         |
| `person_income`                  |  float | Доход клиента               |
| `person_emp_exp`                 |  float | Опыт работы                 |
| `person_home_ownership`          | string | Тип владения жильём         |
| `loan_amnt`                      |  float | Сумма кредита               |
| `loan_intent`                    | string | Цель кредита                |
| `loan_int_rate`                  |  float | Процентная ставка           |
| `loan_percent_income`            |  float | Доля кредита от дохода      |
| `cb_person_cred_hist_length`     |  float | Длина кредитной истории     |
| `credit_score`                   |    int | Кредитный рейтинг           |
| `previous_loan_defaults_on_file` | string | Были ли дефолты по кредитам |

---

# ML-часть

## Постановка задачи

Задача машинного обучения — бинарная классификация:

```text
0 — отказ
1 — одобрение
```

Основная метрика качества:

```text
ROC-AUC
```

ROC-AUC показывает способность модели различать положительный и отрицательный классы. Чем ближе значение к `1.0`, тем лучше качество классификации.

---

# Сравнение моделей

Было обучено и сравнено три алгоритма классификации.

| Модель              | ROC-AUC на тесте |
| ------------------- | ---------------: |
| Logistic Regression |           0.8512 |
| Random Forest       |           0.9683 |
| Gradient Boosting   |           0.9727 |

Параметры моделей:

```python
# Logistic Regression
LogisticRegression(max_iter=1000, random_state=42)

# Random Forest
RandomForestClassifier(n_estimators=100, random_state=42)

# Gradient Boosting
GradientBoostingClassifier(n_estimators=100, random_state=42)
```

Все модели обучались на одинаковых данных после предобработки и отбора признаков.

---

# Итоговая модель

Итоговая модель:

```text
GradientBoostingClassifier
```

Качество модели:

```text
ROC-AUC на тестовой выборке: 0.9727
ROC-AUC на 5-fold cross-validation: 0.9701 ± 0.0023
```

В текущем локальном запуске скрипта `ml/train.py` после корректного разделения train/test было получено близкое значение:

```text
ROC-AUC на тесте: 0.9714
```

Модель демонстрирует высокое качество, что объясняется наличием сильных признаков:

* `credit_score`;
* `loan_percent_income`;
* `person_income`;
* `loan_int_rate`;
* `person_emp_exp`.

---

# Предобработка данных

Вся предобработка встроена в единый `sklearn.Pipeline`.

Это важно, потому что backend может передавать в модель сырые данные клиента, а все преобразования выполняются внутри Pipeline.

## 1. Очистка данных

Удаляются аномальные значения:

```python
df = df[(df["person_age"] <= 100) & (df["person_age"] >= 18)]
df = df[(df["person_emp_exp"] <= 80) | (df["person_emp_exp"].isna())]
df = df[df["cb_person_cred_hist_length"] <= 80]
```

Используемые ограничения:

* `person_age`: от 18 до 100 лет;
* `person_emp_exp`: не более 80 лет;
* `cb_person_cred_hist_length`: не более 80 лет.

Пропуски в данных отсутствовали, поэтому отдельное заполнение пропусков не потребовалось.

## 2. Разделение признаков

Числовые признаки:

```text
person_age
person_income
person_emp_exp
loan_amnt
loan_int_rate
loan_percent_income
cb_person_cred_hist_length
credit_score
```

Категориальные признаки:

```text
person_gender
person_education
person_home_ownership
loan_intent
previous_loan_defaults_on_file
```

## 3. Масштабирование числовых признаков

Для числовых признаков используется:

```python
StandardScaler()
```

Он нормализует данные к среднему `0` и дисперсии `1`.

## 4. Кодирование категориальных признаков

Для категориальных признаков используется:

```python
OneHotEncoder(drop="first", handle_unknown="ignore")
```

Параметры:

* `drop="first"` — удаляет первый уровень категории для уменьшения мультиколлинеарности;
* `handle_unknown="ignore"` — позволяет безопасно обрабатывать неизвестные категории при предсказании.

## 5. Отбор признаков

Используется:

```python
SelectKBest(f_classif, k=20)
```

Из всех признаков после one-hot кодирования выбираются 20 наиболее информативных.

## 6. Полный Pipeline

```python
Pipeline(
    steps=[
        (
            "preprocessor",
            ColumnTransformer(
                [
                    ("num", StandardScaler(), numerical_cols),
                    (
                        "cat",
                        OneHotEncoder(drop="first", handle_unknown="ignore"),
                        categorical_cols,
                    ),
                ]
            ),
        ),
        ("selector", SelectKBest(f_classif, k=20)),
        (
            "classifier",
            GradientBoostingClassifier(
                n_estimators=100,
                random_state=42,
            ),
        ),
    ]
)
```

---

# Сохранение модели

Модель сериализуется через `joblib`.

Файл модели для backend:

```text
backend/models/model.pkl
```

Пример загрузки:

```python
import joblib

model = joblib.load("backend/models/model.pkl")
```

Backend также поддерживает загрузку модели через endpoint:

```text
POST /upload-model
```

При загрузке модель сохраняется как:

```text
backend/models/model.pkl
```

---

# Обучение модели

Для повторного обучения модели используется скрипт:

```text
ml/train.py
```

Запуск из корня проекта:

```bash
python ml/train.py
```

После запуска модель обучается на датасете:

```text
data/loan_data.csv
```

и сохраняется в:

```text
backend/models/model.pkl
```

Пример вывода:

```text
ROC-AUC на тесте: 0.9714
Модель сохранена в .../backend/models/model.pkl
```

---

# Backend

Backend реализован на FastAPI.

Основной файл:

```text
backend/app/main.py
```

Файл работы с моделью:

```text
backend/app/model_service.py
```

Файл схем данных:

```text
backend/app/schemas.py
```

---

# API endpoints

## `GET /`

Возвращает frontend-приложение.

Адрес:

```text
http://127.0.0.1:8000/
```

---

## `GET /health`

Проверка состояния сервиса.

Пример ответа:

```json
{
  "status": "ok",
  "model_loaded": true
}
```

---

## `POST /upload-model`

Загрузка модели в формате `.pkl` или `.joblib`.

Принимает:

```text
multipart/form-data
file: model.pkl
```

Пример ответа:

```json
{
  "status": "success",
  "message": "Model uploaded and loaded successfully"
}
```

---

## `POST /predict`

Получение предсказания по одной или нескольким заявкам.

Пример запроса:

```json
{
  "records": [
    {
      "person_age": 35,
      "person_gender": "male",
      "person_education": "Bachelor",
      "person_income": 75000,
      "person_emp_exp": 5,
      "person_home_ownership": "RENT",
      "loan_amnt": 15000,
      "loan_intent": "EDUCATION",
      "loan_int_rate": 11.5,
      "loan_percent_income": 0.2,
      "cb_person_cred_hist_length": 6,
      "credit_score": 720,
      "previous_loan_defaults_on_file": "No"
    }
  ]
}
```

Пример ответа:

```json
{
  "predictions": [
    {
      "input_data": {
        "person_age": 35,
        "person_gender": "male",
        "person_education": "Bachelor",
        "person_income": 75000,
        "person_emp_exp": 5,
        "person_home_ownership": "RENT",
        "loan_amnt": 15000,
        "loan_intent": "EDUCATION",
        "loan_int_rate": 11.5,
        "loan_percent_income": 0.2,
        "cb_person_cred_hist_length": 6,
        "credit_score": 720,
        "previous_loan_defaults_on_file": "No"
      },
      "loan_status": "отказ",
      "loan_status_code": 0,
      "approval_probability": 0.15647665058728605
    }
  ]
}
```

---

## `POST /predict-from-csv`

Обработка CSV-файла.

Принимает:

```text
multipart/form-data
file: loan_data.csv
```

CSV может содержать колонку `loan_status` или не содержать её.

Если колонка `loan_status` есть, backend дополнительно считает ROC-AUC.

Пример ответа:

```json
{
  "roc_auc": 0.9727,
  "rows_count": 45000,
  "preview_rows_count": 20,
  "data_preview": [
    {
      "person_age": 35,
      "person_income": 75000,
      "loan_amnt": 15000,
      "predicted_loan_status_code": 0,
      "predicted_loan_status": "отказ",
      "approval_probability": 0.1564
    }
  ],
  "download_url": "/download-predictions"
}
```

Чтобы Swagger не зависал на большом датасете, API возвращает только первые 20 строк в `data_preview`.

Полный датасет с предсказаниями сохраняется в файл:

```text
backend/predictions/predicted_loan_data.csv
```

---

## `GET /download-predictions`

Скачивание полного CSV-файла с предсказаниями.

Адрес:

```text
http://127.0.0.1:8000/download-predictions
```

---

# Frontend

Frontend реализован на HTML, CSS и JavaScript.

Файлы:

```text
frontend/index.html
frontend/style.css
frontend/script.js
```

Frontend доступен по адресу:

```text
http://127.0.0.1:8000/
```

Frontend позволяет:

* загрузить `.pkl` / `.joblib` модель;
* ввести данные клиента;
* получить предсказание;
* увидеть вероятность одобрения;
* загрузить CSV-файл;
* получить ROC-AUC;
* скачать полный CSV с предсказаниями.

Frontend раздаётся через FastAPI, поэтому отдельный frontend-сервер не требуется.

---

# Swagger

Swagger-документация доступна по адресу:

```text
http://127.0.0.1:8000/docs
```

Через Swagger можно протестировать все endpoint’ы:

```text
GET /health
POST /upload-model
POST /predict
POST /predict-from-csv
GET /download-predictions
```

---

# Установка и запуск

## 1. Клонирование проекта

```bash
git clone <repository-url>
cd loan-approval-service
```

## 2. Создание виртуального окружения

```bash
python -m venv .venv
```

Активация на Windows:

```bash
.venv\Scripts\activate
```

Активация на Linux/macOS:

```bash
source .venv/bin/activate
```

## 3. Установка backend-зависимостей

```bash
pip install -r backend/requirements.txt
```

## 4. Установка frontend-зависимостей

```bash
npm install
```

## 5. Запуск backend и frontend

```bash
uvicorn backend.app.main:app --reload
```

После запуска:

```text
Frontend: http://127.0.0.1:8000/
Swagger:  http://127.0.0.1:8000/docs
```

---

# Проверка работы приложения

## Сценарий демонстрации

1. Запустить backend:

```bash
uvicorn backend.app.main:app --reload
```

2. Открыть frontend:

```text
http://127.0.0.1:8000/
```

3. Загрузить модель через интерфейс или Swagger.

4. Заполнить форму клиента.

5. Получить результат:

```text
одобрено / отказ
```

6. Загрузить CSV-файл.

7. Получить ROC-AUC и preview строк.

8. Скачать полный CSV с предсказаниями.

9. Открыть Swagger:

```text
http://127.0.0.1:8000/docs
```

---

# Тестирование

Для backend используются тесты на `pytest`.

Запуск:

```bash
PYTHONPATH=. pytest backend/tests
```

Для Windows PowerShell:

```powershell
$env:PYTHONPATH="."
pytest backend/tests
```

Текущий результат:

```text
10 passed
```

Тесты проверяют:

* `GET /health`;
* ошибку `/predict`, если модель не загружена;
* успешный `/predict`;
* загрузку модели;
* отказ при неправильном расширении модели;
* обработку CSV;
* отказ при неправильном расширении CSV;
* скачивание CSV с предсказаниями;
* работу функции предсказания из ML-модуля.

---

# Линтинг

## Backend

Для backend используется Ruff.

Проверка:

```bash
ruff check backend ml
```

Автоисправление:

```bash
ruff check backend ml --fix
```

Форматирование:

```bash
ruff format backend ml
```

## Frontend

Для frontend используется ESLint.

Запуск:

```bash
npm run lint:frontend
```

---

# CI SourceCraft

CI-сценарий находится в файле:

```text
.sourcecraft/ci.yaml
```

CI запускается при:

* push в `main` или `master`;
* pull request в `main` или `master`.

CI выполняет:

* установку Python-зависимостей;
* backend-линтинг через Ruff;
* backend-тесты через Pytest;
* установку Node-зависимостей;
* frontend-линтинг через ESLint.

Основные команды CI:

```bash
PYTHONPATH=. ruff check backend ml
PYTHONPATH=. pytest backend/tests
npm ci
npm run lint:frontend
```

---

# `.gitignore`

В проекте используется `.gitignore`, чтобы не добавлять в репозиторий временные и тяжёлые файлы:

```text
.venv/
node_modules/
__pycache__/
.pytest_cache/
.ruff_cache/
backend/predictions/
backend/models/model.pkl
.idea/
.vscode/
```

`backend/models/model.pkl` игнорируется как runtime-копия модели, которая может быть загружена через API.

---

# Воспроизводимость

Для воспроизводимости используются фиксированные значения:

```python
random_state=42
```

Они применяются в:

* `train_test_split`;
* `LogisticRegression`;
* `RandomForestClassifier`;
* `GradientBoostingClassifier`.

Также для воспроизводимости необходимо использовать зависимости из:

```text
backend/requirements.txt
package.json
package-lock.json
```

---

# Обработка ошибок

Backend обрабатывает основные ошибки:

* модель не загружена;
* неверный формат файла модели;
* неверный формат CSV;
* отсутствующие признаки в данных;
* невозможность расчёта ROC-AUC;
* отсутствие файла с предсказаниями для скачивания.

Пример ошибки, если модель не загружена:

```json
{
  "detail": "Model is not loaded"
}
```

---

# Роли участников

## ML Engineer

Зона ответственности:

* анализ данных;
* очистка данных;
* предобработка признаков;
* обучение нескольких моделей;
* сравнение моделей по ROC-AUC;
* выбор итоговой модели;
* сохранение модели через joblib;
* подготовка ML Pipeline.

## Backend / Frontend Developer

Зона ответственности:

* разработка FastAPI backend;
* реализация endpoint’ов:

  * `/upload-model`;
  * `/predict`;
  * `/predict-from-csv`;
  * `/download-predictions`;
* интеграция backend с ML Pipeline;
* обработка ошибок;
* разработка пользовательского интерфейса;
* подключение frontend к API;
* реализация загрузки модели;
* реализация формы клиента;
* реализация загрузки CSV;
* отображение результатов;
* настройка тестов;
* настройка backend/frontend линтинга;
* настройка SourceCraft CI.

---

# Итоговый результат

В результате работы разработано полноценное веб-приложение:

* обученная ML-модель сохранена в формате `.pkl`;
* модель встроена в FastAPI backend;
* API предоставляет все требуемые endpoint’ы;
* frontend позволяет работать с моделью без Swagger;
* CSV-файлы обрабатываются пакетно;
* результат можно скачать в виде CSV;
* добавлены тесты;
* добавлен линтинг backend и frontend;
* добавлен CI SourceCraft.

Проект готов к демонстрации и сдаче.
