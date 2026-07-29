# PALIMPSEST
![Python](https://img.shields.io/badge/python-3.11-blue)
![License](https://img.shields.io/badge/license-MIT-green)
![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-009688)

Платформа аналитики Telegram-чатов с комплексным сбором, обработкой и визуализацией данных.

## ⚠️ Disclaimer
This tool is intended for analyzing **public groups and channels** or **your own chats** only.  
The authors are not responsible for misuse, unauthorized data collection, or violations of Telegram ToS and local privacy laws.

## Возможности

- **Парсинг сообщений** из Telegram-групп (Telethon)
- **Real-time мониторинг** новых сообщений через WebSocket
- **Семантический NLP-анализ**: тональность, темы, NER, эмбеддинги
- **Стилометрический анализ**: хронология, поведенческие паттерны
- **Нейроконтекстный анализ** (OpenAI / локальные модели)
- **Поиск по сообщениям** с семантическим ранжированием
- **Связевой анализ** участников (графы, скоринг, кластеризация)
- **OCR** изображений (Tesseract, EasyOCR)
- **Транскрибация аудио** (Whisper / OpenAI Whisper API)
- **Генерация отчётов** (PDF, XLSX)
- **Dead account detection**
- **MITRE ATT&CK mapping**
- **OSINT-обогащение**
- **Demo-режим** с синтетическими данными
- **JWT-аутентификация** с RBAC (Admin / Analyst / Viewer)

## Архитектура

```
┌─────────────┐   ┌──────────┐   ┌─────────┐   ┌──────────┐
│  Telegram    │   │  Celery  │   │  Redis  │   │  Nginx   │
│  (Telethon)  │──▶│  Worker  │──▶│  (Queue │──▶│  (Proxy) │
└─────────────┘   └──────────┘   │  + WS)  │   └──────────┘
                                 └─────────┘
┌─────────────┐   ┌──────────┐        │        ┌──────────┐
│  Monitor    │   │  FastAPI │◀───────┘        │ Postgres │
│  Service    │──▶│  App     │─────────────────▶│ (pgvector)│
└─────────────┘   └──────────┘                 └──────────┘
```

## Быстрый старт

### Предварительные требования

- Docker и Docker Compose (или Python 3.11+)
- Telegram API ID, API Hash (для парсинга)
- OpenAI API Key (опционально, для нейро-анализа)

### Запуск через Docker

```bash
cp .env.example .env
# Отредактируйте .env при необходимости
docker compose up -d
```

Приложение будет доступно по адресу: `http://localhost:80`

**Учётные данные по умолчанию:**
- Логин: `admin`
- Пароль: `SecurePass2025!`

### Запуск вручную

```bash
# Установка зависимостей
pip install -r requirements.txt

# Скачать spaCy модель
python -m spacy download ru_core_news_sm

# Запуск
uvicorn backend.app.main:app --reload
```

## Переменные окружения

| Переменная | Описание | По умолчанию |
|---|---|---|
| `POSTGRES_*` | Параметры подключения к PostgreSQL | `tg_user` / `tg_password` / `tg_analytics` |
| `REDIS_*` | Параметры подключения к Redis | `redis:6379` |
| `SECRET_KEY` | Секретный ключ для JWT | `change-me-in-production` |
| `API_KEY_INTERNAL` | Внутренний API-ключ | `internal-api-key-change-me` |
| `TELEGRAM_API_ID` | Telegram API ID | — |
| `TELEGRAM_API_HASH` | Telegram API Hash | — |
| `OPENAI_API_KEY` | OpenAI API ключ | — |
| `CELERY_CONCURRENCY` | Кол-во Celery воркеров | `2` |

## Разработка

```bash
# Установка dev-зависимостей
pip install -r requirements.txt

# Миграции БД
alembic upgrade head

# Запуск тестов
pytest

# Проверка типов
mypy .

# Линтинг
ruff check .
```

## API Endpoints

- `GET /api/health` — проверка работоспособности
- `POST /api/auth/login` — JWT аутентификация
- `POST /api/auth/register` — регистрация (admin only)
- `GET /api/auth/me` — информация о текущем пользователе

Полная документация OpenAPI: `http://localhost:8000/docs`

## Структура проекта

```
PALIMPSEST/
├── backend/
│   ├── app/
│   │   ├── main.py              # Точка входа FastAPI
│   │   ├── config.py            # Конфигурация (pydantic-settings)
│   │   ├── db.py                # Подключение к БД
│   │   ├── dependencies.py      # DI: auth, rate limiting
│   │   ├── websocket.py         # WebSocket рассылка
│   │   ├── models/              # SQLAlchemy модели
│   │   ├── schemas/             # Pydantic схемы
│   │   ├── routers/             # FastAPI роутеры
│   │   ├── services/            # Бизнес-логика
│   │   ├── modules/             # Модули (парсер, NLP, аналитика)
│   │   ├── templates/           # Jinja2 шаблоны
│   │   └── static/              # Статические файлы
│   ├── celery_worker.py         # Celery приложение
│   └── monitor_service.py       # Real-time мониторинг
├── alembic/                     # Миграции БД
├── tests/                       # Тесты
├── storage/                     # Данные (медиа, кеш)
├── outputs/                     # Результаты (отчёты, графики)
├── docker-compose.yml
├── Dockerfile
├── nginx.conf
├── .env.example
├── .gitignore
├── pyproject.toml
└── requirements.txt
```

## Технологии

- **Backend**: Python 3.11, FastAPI, SQLAlchemy (async), Celery
- **БД**: PostgreSQL + pgvector, Redis
- **NLP**: spaCy, sentence-transformers, transformers, langdetect, natasha
- **ML**: scikit-learn, PyTorch (CPU), NetworkX
- **Telegram**: Telethon
- **Инфраструктура**: Docker, Nginx

## Лицензия

MIT
