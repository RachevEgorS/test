# PawsBoard

Маркетплейс и агрегатор зоосервисов Тюмени. Позволяет владельцам питомцев находить ветклиники, груминг-салоны и гостиницы, сравнивать их и записываться онлайн. Организации получают личный кабинет и уведомления через Telegram-бота.

---

## Структура проекта

Монорепо из трёх сервисов на общей PostgreSQL базе:

```
PawsBoard/
├── frontend/      # React SPA (порт 5173)
├── backend/       # Express REST API (порт 4000)
├── bot/           # Telegram-бот (Telegraf)
└── docker/        # Docker Compose для PostgreSQL
```

---

## Быстрый старт

### 1. База данных

```bash
docker-compose -f docker/docker-compose.yml up -d
```

PostgreSQL поднимается на порту `5432`. Параметры подключения — в `backend/.env`.

### 2. Backend

```bash
cd backend
cp .env.example .env   # заполни переменные
npm install
npm run dev            # http://localhost:4000
```

### 3. Frontend

```bash
cd frontend
npm install
npm run dev            # http://localhost:5173
```

Vite автоматически проксирует `/api/*` на `http://localhost:4000` — никаких CORS настроек не нужно.

### 4. Telegram-бот (опционально)

```bash
cd bot
cp .env.example .env   # заполни BOT_TOKEN и DATABASE_URL
npm install
npm start
```

---

## Переменные окружения

### `backend/.env`

| Переменная | Описание |
|---|---|
| `DATABASE_URL` | `postgresql://app:app@localhost:5432/pet_aggregator` |
| `PORT` | Порт API сервера (по умолчанию `4000`) |
| `TELEGRAM_BOT_TOKEN` | Токен бота от @BotFather |
| `TELEGRAM_BOT_LINK` | Ссылка на бота, напр. `https://t.me/your_bot` |
| `SMTP_HOST` | SMTP сервер для email |
| `SMTP_PORT` | Порт SMTP (обычно `465`) |
| `SMTP_USER` | Логин почты |
| `SMTP_PASS` | Пароль почты |
| `SMTP_FROM` | Email отправителя |

### `bot/.env`

| Переменная | Описание |
|---|---|
| `BOT_TOKEN` | Токен Telegram-бота |
| `DATABASE_URL` | Тот же что у бекенда |

---

## API эндпоинты

| Метод | Путь | Описание |
|---|---|---|
| GET | `/api/health` | Healthcheck |
| GET | `/api/organizations?type=vet\|grooming\|hotel` | Список организаций по типу |
| POST | `/api/users/register` | Регистрация (владелец или организация) |
| POST | `/api/requests` | Заявка от клиента (с Telegram-уведомлением) |
| GET | `/api/test-email` | Тест отправки письма |

> ⚠️ `POST /api/users/login` — пока не реализован.

### Регистрация — два формата тела запроса

**Владелец питомца** (`role: "pet_owner"`):
```json
{
  "role": "pet_owner",
  "firstName": "Иван",
  "lastName": "Иванов",
  "phone": "+79001234567",
  "email": "ivan@example.com",
  "password": "secret"
}
```

**Представитель организации** (`role: "org_rep"`):
```json
{
  "role": "org_rep",
  "organizationName": "Ветклиника Дружок",
  "city": "Тюмень",
  "address": "ул. Ленина, 1",
  "organizationPhone": "+73452000000",
  "email": "clinic@example.com",
  "password": "secret"
}
```

---

## Схема базы данных

```
users                        — аккаунты (pet_owner | org_rep)
organizations                — организации (vet | grooming | hotel)
organization_types           — справочник типов
branches                     — филиалы организаций
client_requests              — заявки от клиентов
organization_telegram_managers — привязка Telegram к организации
```

---

## Telegram-бот

Бот используется организациями для получения уведомлений о новых заявках.

**Как привязать организацию к боту:**
1. Пройди регистрацию на сайте как `org_rep` — на почту придёт `access code` вида `ORG-ABC123`
2. Открой бота в Telegram и отправь этот код
3. Бот привяжет твой Telegram-аккаунт к организации

**Что умеет бот:**
- Получать уведомления о новых заявках
- Просматривать pending-заявки
- Принимать (`✅`) или отклонять (`❌`) заявки через inline-кнопки

---

## Роуты фронтенда

| Путь | Страница |
|---|---|
| `/` | Главная |
| `/categories` | Список провайдеров (ветклиники / груминг / гостиницы) |
| `/marketplace` | Маркетплейс (заглушка) |
| `/blog` | Блог (заглушка) |
| `*` | 404 |

---

## Известные баги

- `POST /api/users/login` не реализован — вход не работает
- Бекенд отдаёт типы `vet` / `hotel`, фронт ожидает `vet_clinic` / `pet_hotel` — маппинг в `frontend/src/data/providers.ts:71`
