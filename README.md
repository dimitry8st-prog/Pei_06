# Langflow FastAPI Proxy

FastAPI-сервис — прокси для AI-агента умножения чисел, созданного в Langflow. Принимает JSON-запрос с числами, отправляет выражение в Langflow и возвращает результат в удобном формате.

Подходит для подключения из **Telegram-бота**, **веб-формы** (Lovable/React) или любого другого клиента.

## Возможности

- `POST /multiply` — умножение чисел через Langflow-агента
- `user_id` и `session_id` — идентификация пользователя и сессии
- `source` — метка источника: `telegram`, `web`, `api`
- `response_mode` — короткий (`short`) или развёрнутый (`detailed`) ответ
- `processing_time_ms` — время обработки запроса
- Логирование запросов и ответов
- CORS для веб-клиентов
- Swagger UI: `/docs`

## Стек

- Python 3.12+
- FastAPI
- httpx
- Langflow (внешний AI-агент)

## Быстрый старт

### 1. Клонирование и виртуальное окружение

```bash
cd Pei_06
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate
```

### 2. Установка зависимостей

```bash
pip install -r requirements.txt
```

### 3. Настройка переменных окружения

Создайте файл `.env` в корне проекта:

```env
LANGFLOW_URL=http://127.0.0.1:7860
LANGFLOW_FLOW_ID=your-flow-id
LANGFLOW_API_KEY=your-api-key

# опционально
LANGFLOW_INPUT_TYPE=chat
LANGFLOW_OUTPUT_TYPE=chat
LOVEABLE_ORIGIN=https://your-app.lovable.app
CORS_ALLOW_ALL=false
```

| Переменная | Обязательная | Описание |
|------------|:------------:|----------|
| `LANGFLOW_URL` | да | URL инстанса Langflow |
| `LANGFLOW_FLOW_ID` | да | ID flow в Langflow |
| `LANGFLOW_API_KEY` | да | API-ключ Langflow |
| `LANGFLOW_INPUT_TYPE` | нет | Тип входа (по умолчанию `chat`) |
| `LANGFLOW_OUTPUT_TYPE` | нет | Тип выхода (по умолчанию `chat`) |
| `LOVEABLE_ORIGIN` | нет | Разрешённый origin для CORS |
| `CORS_ALLOW_ALL` | нет | `true` — разрешить все origins (только для отладки) |

### 4. Запуск

```bash
uvicorn main:app --reload
```

Сервис будет доступен по адресу: http://127.0.0.1:8000

## API

### `GET /`

Проверка, что сервис запущен.

```json
{"status": "up"}
```

### `GET /health`

Healthcheck.

```json
{"status": "ok"}
```

### `POST /multiply`

Отправляет выражение умножения в Langflow и возвращает результат.

**Content-Type:** `application/json`

#### Тело запроса

| Поле | Тип | Обязательное | По умолчанию | Описание |
|------|-----|:------------:|--------------|----------|
| `numbers` | `float[]` | да | — | Минимум 2 числа |
| `session_id` | `string` | нет | из `user_id` или `multiply-session` | ID сессии Langflow |
| `user_id` | `string` | нет | — | ID пользователя |
| `source` | `string` | нет | `api` | `telegram`, `web` или `api` |
| `response_mode` | `string` | нет | `short` | `short` или `detailed` |

#### Пример запроса

```bash
curl -X POST http://127.0.0.1:8000/multiply \
  -H "Content-Type: application/json" \
  -d '{
    "numbers": [2, 3, 4],
    "user_id": "test-1",
    "source": "api",
    "response_mode": "short"
  }'
```

#### Пример ответа (`short`)

```json
{
  "input": "2 * 3 * 4",
  "result_text": "24",
  "session_id": "user-test-1",
  "user_id": "test-1",
  "source": "api",
  "response_mode": "short",
  "processing_time_ms": 842.15
}
```

#### Пример ответа (`detailed`)

Дополнительно возвращает `auth_used` и полный `raw`-ответ Langflow.

## Использование из интерфейсов

### Telegram-бот

```json
{
  "numbers": [5, 6],
  "user_id": "987654321",
  "session_id": "tg-987654321",
  "source": "telegram",
  "response_mode": "short"
}
```

- `numbers` — из текста сообщения пользователя
- `user_id` — Telegram ID
- `response_mode: "short"` — бот показывает только `result_text`

### Веб-форма

```json
{
  "numbers": [5, 6],
  "user_id": "user-42",
  "session_id": "web-session-abcd",
  "source": "web",
  "response_mode": "detailed"
}
```

- `numbers` — из полей формы
- `session_id` — из `localStorage` / cookie
- `response_mode: "detailed"` — для отладки на фронтенде

## Документация API

После запуска сервера:

- Swagger UI: http://127.0.0.1:8000/docs
- ReDoc: http://127.0.0.1:8000/redoc

## Структура проекта

```
Pei_06/
├── main.py           # FastAPI-приложение
├── requirements.txt  # Зависимости
├── .env              # Переменные окружения (не коммитить!)
├── README.md         # Этот файл
├── HOMEWORK.md       # Описание для ДЗ
└── ДЗ.md             # Оформленное домашнее задание
```

## Деплой

Сервис можно развернуть на Railway, Render или любом VPS.

1. Установите зависимости из `requirements.txt`
2. Задайте переменные окружения (`LANGFLOW_URL`, `LANGFLOW_FLOW_ID`, `LANGFLOW_API_KEY`)
3. Запустите:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000
```

## Безопасность (для продакшена)

- Не храните `.env` в git
- Используйте HTTPS
- Ограничьте CORS доверенными доменами
- Добавьте аутентификацию API и rate limiting
- Не отдавайте клиенту внутренние ошибки Langflow

## Лицензия

Учебный проект.
