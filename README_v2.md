# World Event Tracker v2.0: Real-time OSINT Platform

## 1. Огляд проекту та філософія

**World Event Tracker v2.0** — це професійна платформа для відстеження подій у реальному часі, розроблена з урахуванням принципів OSINT (Open Source Intelligence) та функціоналу, що відповідає високим стандартам "DeepState" або "Delphi". Система призначена для агрегації, валідації, візуалізації та аналізу геопросторових подій з відкритих джерел, надаючи користувачам інструменти для глибокого розуміння динаміки світових процесів.

## 2. Архітектурна схема взаємодії компонентів

Система має мікросервісну архітектуру, що забезпечує масштабованість, відмовостійкість та гнучкість у розробці. Основні компоненти взаємодіють через RESTful API, WebSockets та черги повідомлень.

```mermaid
graph TD
    subgraph Frontend
        A[Flutter App (iOS/Android/Web)]
        B(Mapbox SDK)
        C(Firebase Auth UI)
        D(Payment UI)
    end

    subgraph Backend (FastAPI)
        E[API Gateway]
        F[Event Service]
        G[OSINT Service]
        H[Auth/RBAC Service]
        I[Subscription Service]
        J[WebSocket Service]
        K[Admin Service]
    end

    subgraph Data Stores
        L[PostgreSQL + PostGIS]
        M[Redis Cache]
        N[Firebase Auth DB]
    end

    subgraph External Services
        O[Telegram API (Telethon)]
        P[News/Govt. Websites]
        Q[Stripe/RevenueCat]
        R[Firebase Cloud Messaging (FCM)]
    end

    subgraph Admin Panel
        S[Flutter Web Admin App]
    end

    A -- REST/WS --> E
    A -- Auth --> N
    A -- Payments --> Q
    A -- Push Notifications --> R
    B -- Map Data --> Mapbox Servers

    E -- REST --> F
    E -- REST --> G
    E -- REST --> H
    E -- REST --> I
    E -- WS --> J
    E -- REST --> K

    F -- Read/Write --> L
    F -- Cache --> M
    G -- Scrape --> O
    G -- Scrape --> P
    G -- Write Validated Events --> F
    H -- Manage Users/Roles --> L
    I -- Manage Subscriptions --> L
    I -- Process Payments --> Q
    J -- Real-time Events --> M
    K -- Admin Operations --> L
    K -- Admin Operations --> F
    K -- Admin Operations --> H

    S -- REST/WS --> E
    S -- Auth --> N

    F -- Send Alerts --> R

    style A fill:#00BCD4,stroke:#333,stroke-width:2px
    style S fill:#FFC107,stroke:#333,stroke-width:2px
    style E fill:#4CAF50,stroke:#333,stroke-width:2px
    style F fill:#4CAF50,stroke:#333,stroke-width:2px
    style G fill:#4CAF50,stroke:#333,stroke-width:2px
    style H fill:#4CAF50,stroke:#333,stroke-width:2px
    style I fill:#4CAF50,stroke:#333,stroke-width:2px
    style J fill:#4CAF50,stroke:#333,stroke-width:2px
    style K fill:#4CAF50,stroke:#333,stroke-width:2px
    style L fill:#2196F3,stroke:#333,stroke-width:2px
    style M fill:#F44336,stroke:#333,stroke-width:2px
    style N fill:#FF9800,stroke:#333,stroke-width:2px
    style O fill:#9C27B0,stroke:#333,stroke-width:2px
    style P fill:#9C27B0,stroke:#333,stroke-width:2px
    style Q fill:#673AB7,stroke:#333,stroke-width:2px
    style R fill:#FF5722,stroke:#333,stroke-width:2px
```

**Пояснення компонентів:**

*   **Flutter App (Frontend):** Кроссплатформенний клієнт для iOS, Android та Web. Включає UI для відображення карти, подій, авторизації та оплати. Використовує Mapbox SDK для інтерактивної карти. 
*   **FastAPI Backend:** Головний API-шлюз, що маршрутизує запити до різних сервісів. 
    *   **Event Service:** Керує CRUD операціями для подій, взаємодіє з PostgreSQL/PostGIS та Redis. 
    *   **OSINT Service:** Відповідає за збір, парсинг та попередню валідацію даних з різних джерел. 
    *   **Auth/RBAC Service:** Обробляє автентифікацію (через Firebase Auth) та авторизацію на основі ролей (Role-Based Access Control). 
    *   **Subscription Service:** Керує підписками користувачів, взаємодіє з платіжними шлюзами. 
    *   **WebSocket Service:** Забезпечує двосторонній зв'язок у реальному часі для оновлення подій та анімації руху. 
    *   **Admin Service:** Надає API для адмін-панелі. 
*   **PostgreSQL + PostGIS:** Основна база даних для зберігання всіх подій, користувачів, ролей, підписок та геопросторових даних. PostGIS критично важливий для ефективної роботи з координатами. 
*   **Redis Cache:** Використовується для кешування останніх подій, даних для WebSockets та інших часто запитуваних даних для прискорення відгуку. 
*   **Firebase Auth DB:** База даних Firebase для управління користувачами та їх автентифікацією. 
*   **External Services:** 
    *   **Telegram API (Telethon):** Для збору даних з офіційних Telegram-каналів. 
    *   **News/Govt. Websites:** Джерела для скрапінгу новин та офіційних заяв. 
    *   **Stripe/RevenueCat:** Платіжні шлюзи для обробки підписок та платежів. 
    *   **Firebase Cloud Messaging (FCM):** Для push-повідомлень. 
*   **Admin Panel (Flutter Web App):** Окремий інтерфейс для модераторів та адміністраторів для управління системою. 

## 3. Модулі та їх реалізація

### 3.1. Джерела даних та OSINT-модуль

**Мета:** Автоматичний збір, парсинг та валідація інформації з відкритих джерел.

**Реалізація:**

*   **Scrapers/Parsers (OSINT Service):** 
    *   **Telethon:** Для Telegram-каналів буде використовуватися бібліотека `Telethon` (Python). Вона дозволяє програмно взаємодіяти з Telegram API, отримувати повідомлення з вказаних каналів, парсити їх на предмет ключових слів, географічних назв та інших індикаторів подій. 
    *   **Web Scrapers:** Для офіційних сайтів урядів та новинних агенцій будуть розроблені спеціалізовані скрапери (наприклад, з використанням `BeautifulSoup` або `Scrapy` у Python). Ці скрапери будуть налаштовані на моніторинг конкретних сторінок або RSS-стрічок. 
    *   **Розклад:** Запуск скраперів буде здійснюватися за розкладом (наприклад, за допомогою `Celery Beat` або `APScheduler`), щоб забезпечити регулярне оновлення даних. 
*   **Логіка валідації:** 
    *   **Первинна фільтрація:** Зібрані дані проходять первинну обробку для вилучення потенційних подій (наприклад, за наявністю географічних координат, ключових слів, згадок про конфлікти, стихійні лиха тощо). 
    *   **Підтвердження з декількох джерел:** Інформація вважається "офіційною" або "підтвердженою" лише після того, як вона буде знайдена та співставлена з мінімум `N` (конфігуроване значення) незалежних джерел. Це може включати: 
        *   Співставлення за часом, місцем та ключовими словами. 
        *   Використання алгоритмів обробки природної мови (NLP) для визначення схожості змісту. 
        *   Система оцінки довіри до джерела (Source Trust Score), де офіційні урядові сайти мають вищий рейтинг, ніж менш відомі новинні портали. 
    *   **Ручна модерація:** Для критично важливих або неоднозначних подій передбачається етап ручної модерації через Адмін-панель, де оператор може підтвердити, відхилити або відредагувати подію. 

### 3.2. Реальний час та Динаміка

**Мета:** Миттєве оновлення даних на карті та візуалізація руху об'єктів.

**Реалізація:**

*   **WebSockets (FastAPI WebSocket Service):** 
    *   FastAPI має вбудовану підтримку WebSockets, що дозволяє легко створювати двосторонні канали зв'язку між сервером та клієнтом. 
    *   Клієнти (Flutter App) підключаються до WebSocket-сервісу та підписуються на оновлення подій або рух об'єктів. 
    *   Бекенд надсилає оновлення лише тим клієнтам, які підписані на відповідні категорії або географічні області. 
*   **Анімація руху об'єктів:** 
    *   **Бекенд:** Якщо OSINT-модуль виявляє об'єкт, що рухається (наприклад, військова техніка, корабель), бекенд зберігає його поточні координати, напрямок та швидкість. Ці дані надсилаються через WebSocket у вигляді "векторів руху" (початкова точка, кінцева точка, час, швидкість). 
    *   **Фронтенд (Flutter + Mapbox SDK):** 
        *   Mapbox SDK дозволяє відображати кастомні шари та анімувати маркери. 
        *   При отриманні даних про рух, Flutter-додаток буде інтерполювати позицію об'єкта між початковою та кінцевою точками, створюючи плавну анімацію. 
        *   Для інтерполяції можна використовувати лінійну або криволінійну інтерполяцію в залежності від вимог до реалізму. 

### 3.3. Монетизація та Преміум

**Мета:** Впровадження системи підписок для доступу до розширеного функціоналу.

**Реалізація:**

*   **Платіжний шлюз (Subscription Service):** 
    *   **Stripe:** Для веб-версії та як основний бекенд для мобільних додатків. Stripe надає потужний API для управління підписками, виставлення рахунків та обробки платежів. 
    *   **RevenueCat:** Для мобільних додатків (iOS/Android) RevenueCat спрощує інтеграцію з App Store Connect та Google Play Console, обробляючи покупки в додатку, підписки, пробні періоди та webhook-и. Він інтегрується зі Stripe для синхронізації даних про підписки. 
*   **Система рівнів доступу (Auth/RBAC Service):** 
    *   **Firebase Auth:** Використовується для автентифікації користувачів. 
    *   **RBAC (Role-Based Access Control):** 
        *   Кожен користувач матиме роль (наприклад, `free`, `premium`, `admin`, `moderator`). 
        *   Бекенд перевірятиме роль користувача при кожному запиті до захищених ресурсів (наприклад, запит детальної аналітики, доступ до закритих шарів карти). 
        *   **Free-користувачі:** Базовий доступ до карти та загальних подій, обмежена кількість push-повідомлень. 
        *   **Premium-користувачі:** 
            *   **Детальна аналітика:** Доступ до історичних даних, трендів, розширених фільтрів. 
            *   **Закриті шари карти:** Відображення додаткових шарів інформації (наприклад, дані з більш чутливих джерел, прогнози). 
            *   **Миттєві Push-повідомлення:** Необмежена кількість push-повідомлень з можливістю тонкого налаштування категорій та географічних зон. 

### 3.4. Адмін-панель (Back-office)

**Мета:** Інструмент для управління системою та модерації контенту.

**Реалізація:**

*   **Окремий інтерфейс (Flutter Web Admin App):** 
    *   Розроблений як окремий Flutter Web додаток, що використовує ті ж компоненти UI, що й основний додаток, але з розширеним функціоналом. 
    *   **Можливості модераторів:** 
        *   **Додавання/редагування подій:** Форми для ручного введення або корекції даних про події, включаючи географічні координати, категорії, описи та джерела. 
        *   **Управління користувачами:** Перегляд, блокування, зміна ролей користувачів. 
        *   **Статистика навантаження:** Дашборди для моніторингу продуктивності системи, кількості зібраних подій, активності користувачів. 
        *   **Управління джерелами OSINT:** Додавання/видалення Telegram-каналів, веб-сайтів для скрапінгу. 
*   **Інструмент для малювання об'єктів та векторів:** 
    *   Інтегрований Mapbox SDK в адмін-панелі дозволить модераторам малювати полігони, лінії та точки безпосередньо на карті. 
    *   Для об'єктів, що рухаються, модератор зможе вказати початкову та кінцеву точки, напрямок та швидкість, які потім будуть збережені в PostGIS як географічні об'єкти з атрибутами. 

## 4. Розширена структура БД (PostgreSQL + PostGIS)

Додамо таблиці для управління користувачами, ролями та підписками.

```sql
-- Таблиця користувачів (розширення Firebase Auth UID)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    firebase_uid VARCHAR(128) UNIQUE NOT NULL, -- UID з Firebase Auth
    email VARCHAR(255) UNIQUE NOT NULL,
    display_name VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Таблиця ролей
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL -- 'free', 'premium', 'admin', 'moderator'
);

-- Таблиця для зв'язку користувачів та ролей (RBAC)
CREATE TABLE user_roles (
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER NOT NULL REFERENCES roles(id) ON DELETE CASCADE,
    PRIMARY KEY (user_id, role_id)
);

-- Таблиця тарифних планів (для Stripe/RevenueCat)
CREATE TABLE subscription_plans (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL, -- 'Basic', 'Premium Monthly', 'Premium Annual'
    stripe_product_id VARCHAR(255) UNIQUE, -- ID продукту в Stripe
    revenuecat_product_id VARCHAR(255) UNIQUE, -- ID продукту в RevenueCat
    price DECIMAL(10, 2) NOT NULL,
    currency VARCHAR(3) DEFAULT 'USD',
    description TEXT,
    is_active BOOLEAN DEFAULT TRUE
);

-- Таблиця підписок користувачів
CREATE TABLE user_subscriptions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    plan_id INTEGER NOT NULL REFERENCES subscription_plans(id) ON DELETE RESTRICT,
    stripe_subscription_id VARCHAR(255) UNIQUE, -- ID підписки в Stripe
    revenuecat_entitlement_id VARCHAR(255) UNIQUE, -- ID права в RevenueCat
    start_date TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    end_date TIMESTAMP WITH TIME ZONE,
    status VARCHAR(50) NOT NULL, -- 'active', 'cancelled', 'expired', 'trialing'
    auto_renew BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Таблиця для преміум-функцій (які можуть бути прив'язані до планів)
CREATE TABLE premium_features (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) UNIQUE NOT NULL, -- 'detailed_analytics', 'closed_map_layers', 'instant_push_notifications'
    description TEXT
);

-- Таблиця для зв'язку тарифних планів та преміум-функцій
CREATE TABLE plan_features (
    plan_id INTEGER NOT NULL REFERENCES subscription_plans(id) ON DELETE CASCADE,
    feature_id INTEGER NOT NULL REFERENCES premium_features(id) ON DELETE CASCADE,
    PRIMARY KEY (plan_id, feature_id)
);

-- Оновлена таблиця подій (з можливістю прив'язки до джерел та статусу валідації)
CREATE TABLE events (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    coordinates GEOMETRY(Point, 4326) NOT NULL, -- PostGIS Point
    category VARCHAR(50) NOT NULL,
    timestamp TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    source_url VARCHAR(255),
    confidence_score DECIMAL(3, 2) DEFAULT 0.0, -- Оцінка достовірності (0.0 - 1.0)
    validation_status VARCHAR(50) DEFAULT 'pending', -- 'pending', 'validated', 'rejected', 'manual_override'
    is_premium BOOLEAN DEFAULT FALSE, -- Чи є подія доступною лише для преміум-користувачів
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Таблиця для зберігання джерел OSINT
CREATE TABLE osint_sources (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL, -- 'telegram', 'website', 'news_agency'
    url TEXT NOT NULL, -- URL каналу/сайту
    trust_score DECIMAL(3, 2) DEFAULT 0.5, -- Оцінка довіри до джерела
    is_active BOOLEAN DEFAULT TRUE,
    last_scraped TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Таблиця для зв'язку подій з джерелами
CREATE TABLE event_sources (
    event_id UUID NOT NULL REFERENCES events(id) ON DELETE CASCADE,
    source_id UUID NOT NULL REFERENCES osint_sources(id) ON DELETE CASCADE,
    PRIMARY KEY (event_id, source_id)
);

-- Таблиця для рухомих об'єктів (для анімації)
CREATE TABLE moving_objects (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    current_coordinates GEOMETRY(Point, 4326) NOT NULL,
    destination_coordinates GEOMETRY(Point, 4326), -- Якщо відома кінцева точка
    speed_kmh DECIMAL(10, 2),
    direction_degrees DECIMAL(5, 2), -- Напрямок руху в градусах
    last_updated TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    category VARCHAR(50), -- 'vehicle', 'ship', 'aircraft'
    is_premium BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Індекси для прискорення запитів
CREATE INDEX idx_events_coordinates ON events USING GIST(coordinates);
CREATE INDEX idx_events_timestamp ON events(timestamp DESC);
CREATE INDEX idx_user_firebase_uid ON users(firebase_uid);
CREATE INDEX idx_user_subscriptions_user_id ON user_subscriptions(user_id);
CREATE INDEX idx_moving_objects_coordinates ON moving_objects USING GIST(current_coordinates);
```

## 5. Приклад коду на Python для обробки WebSockets (рух об'єкта)

Цей приклад демонструє, як FastAPI може використовувати WebSockets для надсилання оновлень про рух об'єктів клієнтам. У реальному сценарії дані про рух надходили б від OSINT-модуля або адмін-панелі.

Файл: `backend/app/main.py` (доповнення до існуючого)

```python
import asyncio
import json
from typing import List, Dict
from fastapi import FastAPI, WebSocket, WebSocketDisconnect, Depends
from pydantic import BaseModel
from datetime import datetime
import uuid
from redis import Redis
import os

# ... (існуючі імпорти та ініціалізація FastAPI, Redis) ...

# Pydantic моделі для рухомих об'єктів
class MovingObjectCoordinates(BaseModel):
    latitude: float
    longitude: float

class MovingObjectUpdate(BaseModel):
    id: str
    current_coordinates: MovingObjectCoordinates
    destination_coordinates: MovingObjectCoordinates | None = None
    speed_kmh: float | None = None
    direction_degrees: float | None = None
    timestamp: datetime = Field(default_factory=datetime.utcnow)

# Менеджер WebSocket з'єднань
class ConnectionManager:
    def __init__(self):
        self.active_connections: List[WebSocket] = []

    async def connect(self, websocket: WebSocket):
        await websocket.accept()
        self.active_connections.append(websocket)

    def disconnect(self, websocket: WebSocket):
        self.active_connections.remove(websocket)

    async def send_personal_message(self, message: str, websocket: WebSocket):
        await websocket.send_text(message)

    async def broadcast(self, message: str):
        for connection in self.active_connections:
            try:
                await connection.send_text(message)
            except RuntimeError: # WebSocket connection closed
                self.active_connections.remove(connection)

manager = ConnectionManager()

@app.websocket("/ws/moving_objects")
async def websocket_endpoint(websocket: WebSocket, redis: Redis = Depends(get_redis_client)):
    await manager.connect(websocket)
    try:
        while True:
            # Можна отримувати повідомлення від клієнта, наприклад, для підписки на конкретні об'єкти
            data = await websocket.receive_text()
            print(f"Received from client: {data}")
            # Наприклад, клієнт може надіслати запит на історичні дані або фільтри
            # await manager.send_personal_message(f"You said: {data}", websocket)

    except WebSocketDisconnect:
        manager.disconnect(websocket)
        print("Client disconnected")

# Ендпоінт для імітації оновлення руху об'єкта (в реальності це робив би OSINT-сервіс)
@app.post("/moving_objects/simulate_update")
async def simulate_moving_object_update(update: MovingObjectUpdate, redis: Redis = Depends(get_redis_client)):
    """
    Імітує оновлення позиції рухомого об'єкта та розсилає його через WebSockets.
    """
    # Зберігаємо оновлення в Redis або DB (для персистентності та подальшої обробки)
    # Для простоти, просто транслюємо
    message = json.dumps(update.dict(by_alias=True, exclude_unset=True, exclude_none=True, 
                                     json_encoders={datetime: lambda dt: dt.isoformat()}))
    await manager.broadcast(message)
    return {"message": "Moving object update broadcasted"}

# ... (існуючі ендпоінти FastAPI) ...

```

**Пояснення коду:**

*   `MovingObjectUpdate` Pydantic модель визначає структуру даних для оновлення рухомого об'єкта. 
*   `ConnectionManager` керує активними WebSocket-з'єднаннями, дозволяючи надсилати повідомлення окремим клієнтам або транслювати їх усім. 
*   `@app.websocket("/ws/moving_objects")` створює WebSocket-ендпоінт. Кожен клієнт, що підключається до нього, додається до `manager`. 
*   `simulate_moving_object_update` - це REST-ендпоінт, який приймає оновлення про рухомий об'єкт і транслює його всім підключеним WebSocket-клієнтам. У реальному проекті цей ендпоінт викликався б внутрішньо OSINT-сервісом або адмін-панеллю після виявлення або ручного введення даних про рух. 

## 6. Покроковий гайд: Інтеграція системи оплати в Flutter-додаток (Stripe/RevenueCat)

Цей гайд описує загальні кроки для інтеграції платіжних систем. Вибір між Stripe та RevenueCat залежить від ваших пріоритетів (веб vs мобільні додатки).

### 6.1. Налаштування Stripe (для бекенду та веб-версії)

1.  **Створіть обліковий запис Stripe:** Зареєструйтесь на [Stripe.com](https://stripe.com/).
2.  **Отримайте API ключі:** В панелі керування Stripe знайдіть `Publishable key` та `Secret key`. Зберігайте `Secret key` безпечно на бекенді, `Publishable key` можна використовувати на фронтенді.
3.  **Створіть продукти та ціни:** В панелі керування Stripe створіть продукти (наприклад, "Premium Access") та відповідні ціни (наприклад, "Monthly Premium", "Annual Premium"). Запишіть їхні `Product ID` та `Price ID`.
4.  **Бекенд інтеграція (FastAPI Subscription Service):**
    *   Встановіть бібліотеку `stripe` для Python: `pip install stripe`.
    *   Реалізуйте ендпоінти для:
        *   Створення сесії чекауту (Checkout Session) для одноразових платежів або підписок.
        *   Обробки вебхуків (Webhooks) від Stripe. Це критично важливо для оновлення статусу підписки користувача в вашій БД після успішної оплати, скасування, поновлення тощо. Stripe надсилає події (наприклад, `checkout.session.completed`, `customer.subscription.updated`, `customer.subscription.deleted`) на ваш бекенд.
        *   Управління підписками (скасування, оновлення) через Stripe API.
    *   Зберігайте `stripe_customer_id` та `stripe_subscription_id` у вашій таблиці `user_subscriptions`.

### 6.2. Налаштування RevenueCat (для мобільних додатків)

1.  **Створіть обліковий запис RevenueCat:** Зареєструйтесь на [RevenueCat.com](https://www.revenuecat.com/).
2.  **Підключіть магазини додатків:** Підключіть ваш обліковий запис App Store Connect та Google Play Console до RevenueCat.
3.  **Налаштуйте продукти в магазинах:** Створіть продукти для покупки в додатку (In-App Purchases) та підписки в App Store Connect та Google Play Console. Важливо, щоб `Product ID` відповідали тим, що ви будете використовувати в RevenueCat.
4.  **Налаштуйте продукти та права (Entitlements) в RevenueCat:**
    *   Створіть продукти в RevenueCat, пов'язавши їх з продуктами з магазинів додатків.
    *   Створіть права (Entitlements), які представляють ваш преміум-доступ (наприклад, "premium_access"). Прив'яжіть ваші продукти до цих прав.
5.  **Бекенд інтеграція (FastAPI Subscription Service):**
    *   RevenueCat може надсилати вебхуки на ваш бекенд про зміни статусу підписок. Налаштуйте ендпоінт для їх обробки, щоб синхронізувати статус підписки в вашій БД.
    *   Зберігайте `revenuecat_entitlement_id` у вашій таблиці `user_subscriptions`.

### 6.3. Flutter-інтеграція

1.  **Додайте залежності:**
    *   Для Stripe: `flutter_stripe` (офіційний плагін Stripe).
    *   Для RevenueCat: `purchases_flutter` (офіційний плагін RevenueCat).
    *   Додайте до `pubspec.yaml`:
        ```yaml
        dependencies:
          flutter:
            sdk: flutter
          # Для Stripe
          flutter_stripe: ^latest_version
          # Для RevenueCat
          purchases_flutter: ^latest_version
          # Для HTTP запитів до вашого бекенду
          http: ^latest_version
        ```
2.  **Ініціалізація:**
    *   **Stripe:** В `main.dart` або на екрані оплати ініціалізуйте Stripe за допомогою `Stripe.publishableKey = 'YOUR_STRIPE_PUBLISHABLE_KEY';`.
    *   **RevenueCat:** В `main.dart` ініціалізуйте RevenueCat: `await Purchases.configure(PurchasesConfiguration('YOUR_REVENUECAT_API_KEY'));`.
3.  **Логіка покупки/підписки:**
    *   **Stripe (для веб/через бекенд):**
        *   Користувач натискає кнопку "Купити Premium".
        *   Flutter-додаток робить запит до вашого FastAPI бекенду (`/create-checkout-session`), який у свою чергу викликає Stripe API для створення `Checkout Session`.
        *   Бекенд повертає `session_id` та `publishable_key`.
        *   Flutter використовує `Stripe.redirectToCheckout(sessionId: session_id)` для перенаправлення користувача на сторінку оплати Stripe.
        *   Після успішної оплати Stripe перенаправляє користувача на `success_url` (який ви вказали при створенні сесії), де ваш додаток може підтвердити статус підписки.
    *   **RevenueCat (для мобільних):**
        *   Отримайте доступні продукти: `Offerings offerings = await Purchases.getOfferings();`.
        *   Відобразіть продукти користувачеві.
        *   Коли користувач вибирає продукт, здійсніть покупку: `CustomerInfo customerInfo = await Purchases.purchaseProduct(product);`.
        *   RevenueCat автоматично обробляє взаємодію з магазинами додатків. Після успішної покупки `customerInfo` міститиме інформацію про права користувача. 
        *   Ви можете перевірити статус права: `customerInfo.entitlements.all['premium_access']?.isActive`.
4.  **Управління станом підписки:**
    *   Після успішної покупки (через Stripe webhook або RevenueCat `CustomerInfo`) ваш бекенд оновлює статус підписки користувача в таблиці `user_subscriptions`.
    *   Flutter-додаток може запитувати статус підписки користувача з вашого бекенду при запуску або при спробі доступу до преміум-функцій.
    *   Використовуйте `StreamBuilder` або `ChangeNotifier` для реактивного оновлення UI залежно від статусу підписки.

## 7. Посилання

*   [FastAPI Documentation](https://fastapi.tiangolo.com/)
*   [Flutter Documentation](https://docs.flutter.dev/)
*   [PostgreSQL Documentation](https://www.postgresql.org/docs/)
*   [PostGIS Documentation](https://postgis.net/documentation/)
*   [Redis Documentation](https://redis.io/docs/)
*   [Firebase Documentation](https://firebase.google.com/docs)
*   [Mapbox Flutter SDK](https://docs.mapbox.com/mapbox-gl-js/api/)
*   [Telethon Documentation](https://docs.telethon.dev/en/stable/)
*   [Stripe Documentation](https://stripe.com/docs)
*   [RevenueCat Documentation](https://www.revenuecat.com/docs/)
