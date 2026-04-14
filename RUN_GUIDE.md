# Інструкція з запуску World Event Tracker v2.0

Цей гайд допоможе вам розгорнути інфраструктуру, запустити бекенд та фронтенд проекту.

## 1. Попередня підготовка

Переконайтеся, що на вашій системі встановлені:
- [Docker](https://docs.docker.com/get-docker/) та [Docker Compose](https://docs.docker.com/compose/install/)
- [Python 3.10+](https://www.python.org/downloads/)
- [Flutter SDK](https://docs.flutter.dev/get-started/install)

---

## 2. Запуск інфраструктури (База даних та Кеш)

Ми використовуємо Docker для швидкого розгортання PostgreSQL з PostGIS та Redis.

1. Відкрийте термінал у кореневій папці проекту.
2. Запустіть контейнери:
   ```bash
   docker-compose up -d
   ```
3. Перевірте статус:
   ```bash
   docker ps
   ```
   Ви маєте побачити два працюючі контейнери: `world_event_tracker_db` та `world_event_tracker_redis`.

---

## 3. Запуск Бекенду (FastAPI)

1. Перейдіть до папки бекенду:
   ```bash
   cd backend
   ```
2. Створіть та активуйте віртуальне середовище:
   ```bash
   python -m venv venv
   # Для Windows:
   venv\Scripts\activate
   # Для Linux/macOS:
   source venv/bin/activate
   ```
3. Встановіть залежності:
   ```bash
   pip install -r requirements.txt
   ```
4. Створіть файл `.env` (за прикладом):
   ```env
   REDIS_HOST=localhost
   REDIS_PORT=6379
   DATABASE_URL=postgresql://user:password@localhost:5432/world_event_tracker
   ```
5. Запустіть сервер FastAPI:
   ```bash
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   ```
   Бекенд буде доступний за адресою: `http://localhost:8000`.
   Документація API (Swagger): `http://localhost:8000/docs`.

---

## 4. Запуск Фронтенду (Flutter)

1. Перейдіть до папки фронтенду:
   ```bash
   cd frontend
   ```
2. Отримайте залежності Flutter:
   ```bash
   flutter pub get
   ```
3. (Опціонально) Якщо ви запускаєте на Android/iOS, переконайтеся, що емулятор запущений або пристрій підключений.
4. Запустіть додаток:
   ```bash
   flutter run
   ```
   Для веб-версії використовуйте:
   ```bash
   flutter run -d chrome
   ```

---

## 5. Перевірка WebSockets (Реальний час)

Щоб перевірити роботу WebSockets та анімацію руху об'єктів:
1. Відкрийте Flutter додаток (екран карти).
2. Виконайте POST-запит до бекенду (через Swagger або `curl`), щоб імітувати рух:
   ```bash
   curl -X 'POST' \
     'http://localhost:8000/moving_objects/simulate_update' \
     -H 'Content-Type: application/json' \
     -d '{
     "id": "tank-001",
     "current_coordinates": {"latitude": 50.4501, "longitude": 30.5234},
     "destination_coordinates": {"latitude": 50.4550, "longitude": 30.5300},
     "speed_kmh": 40.0,
     "direction_degrees": 45.0
   }'
   ```
3. Об'єкт на карті у Flutter додатку має оновитися/заанімуватися миттєво без перезавантаження.

---

## 6. Налаштування Firebase та Mapbox

Для повноцінної роботи (Auth, Push, Maps) необхідно:
1. **Firebase:** Створити проект у [Firebase Console](https://console.firebase.google.com/), додати Android/iOS додатки та завантажити конфігураційні файли (`google-services.json` / `GoogleService-Info.plist`).
2. **Mapbox:** Отримати Access Token у [Mapbox Dashboard](https://account.mapbox.com/) та додати його до Flutter конфігурації.
3. **Stripe/RevenueCat:** Налаштувати ключі в `.env` бекенду та ініціалізувати у Flutter.
