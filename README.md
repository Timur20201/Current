# World Event Tracker (OSINT-орієнтований додаток)

## 1. Огляд проекту

**World Event Tracker** — це OSINT-орієнтований додаток, призначений для відстеження та візуалізації світових подій у реальному часі. Проект реалізується як мінімально життєздатний продукт (MVP) з акцентом на стильний дизайн у дусі "DeepState" або "Delphi", що поєднує функціональність та естетику для аналізу геопросторових даних.

## 2. Архітектура системи

Система складається з двох основних компонентів: бекенду на Python/FastAPI та фронтенду на Flutter. Для зберігання та кешування даних використовуються PostgreSQL з розширенням PostGIS та Redis відповідно.

### 2.1. Бекенд (Python/FastAPI)

Бекенд відповідає за обробку запитів, взаємодію з базою даних, кешування та логіку push-повідомлень.

*   **Фреймворк**: FastAPI (Python)
*   **База даних**: PostgreSQL з розширенням PostGIS для зберігання геопросторових даних.
*   **Кешування**: Redis для швидкого доступу до останніх подій.
*   **Авторизація**: Інтеграція з Firebase Auth.
*   **Push-повідомлення**: Логіка підписки на категорії подій через Firebase Cloud Messaging (FCM).

#### 2.1.1. Схема бази даних PostgreSQL

Для зберігання подій буде використана наступна схема таблиці `events`:

| Поле         | Тип даних         | Опис                                         |
| :----------- | :---------------- | :------------------------------------------- |
| `id`         | `UUID`            | Унікальний ідентифікатор події               |
| `title`      | `VARCHAR(255)`    | Заголовок події                              |
| `description`| `TEXT`            | Детальний опис події                         |
| `coordinates`| `GEOMETRY(Point, 4326)` | Географічні координати (широта/довгота)      |
| `category`   | `VARCHAR(50)`     | Категорія події (наприклад, "Конфлікт", "Екологія") |
| `timestamp`  | `TIMESTAMP WITH TIME ZONE` | Час виникнення події                         |
| `source_url` | `VARCHAR(255)`    | Посилання на джерело інформації              |

#### 2.1.2. Кешування Redis

Redis буде використовуватися для кешування списку останніх подій. Це дозволить значно прискорити відгук на запити, що стосуються часто запитуваних даних.

### 2.2. Фронтенд (Flutter)

Фронтенд розробляється на Flutter для забезпечення кросплатформенності та сучасного інтерфейсу користувача.

*   **Платформа**: Flutter
*   **Дизайн**: Темна тема (Amoled Black) з акцентними кольорами для категорій подій.
*   **Карти**: Інтеграція з картографічним сервісом (Mapbox або Google Maps) з кастомними маркерами.
*   **UI/UX**: Skeleton Screens для покращення сприйняття завантаження даних.
*   **Авторизація**: Інтеграція з Firebase Auth (вхід через Google/Email).
*   **Push-повідомлення**: Підписка на категорії подій через FCM.
*   **Фільтрація**: Можливість фільтрації подій на карті за категоріями.

## 3. Функціонал

### 3.1. Авторизація

Реалізовано шаблон для Firebase Auth, що дозволяє користувачам входити в систему за допомогою облікового запису Google або електронної пошти.

### 3.2. Push-повідомлення

Користувачі зможуть підписуватися на push-повідомлення для конкретних категорій подій. Логіка підписки буде реалізована через Firebase Cloud Messaging (FCM).

### 3.3. Категорії та фільтрація

Події будуть класифіковані за категоріями. Фронтенд надасть можливість фільтрувати події на карті за цими категоріями, використовуючи відповідні акцентні кольори для візуального розрізнення.

## 4. Структура папок проекту

```
world_event_tracker/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models.py
│   │   ├── schemas.py
│   │   └── crud.py
│   ├── requirements.txt
│   └── Dockerfile
├── frontend/
│   ├── lib/
│   │   ├── main.dart
│   │   ├── screens/
│   │   │   ├── home_screen.dart
│   │   │   └── auth_screen.dart
│   │   ├── widgets/
│   │   │   ├── event_map.dart
│   │   │   └── skeleton_loader.dart
│   │   ├── services/
│   │   │   ├── api_service.dart
│   │   │   ├── auth_service.dart
│   │   │   └── notification_service.dart
│   │   └── utils/
│   │       └── app_theme.dart
│   ├── pubspec.yaml
│   ├── README.md
│   └── android/
│   └── ios/
├── .gitignore
├── README.md
└── docker-compose.yml
```

## 5. Стартовий код

### 5.1. Бекенд: `backend/app/main.py` (FastAPI)

```python
from fastapi import FastAPI, Depends, HTTPException
from redis import Redis
from typing import List
import json
import os

# Ініціалізація FastAPI додатку
app = FastAPI(title="World Event Tracker API")

# Налаштування Redis
REDIS_HOST = os.getenv("REDIS_HOST", "localhost")
REDIS_PORT = int(os.getenv("REDIS_PORT", 6379))
REDIS_DB = int(os.getenv("REDIS_DB", 0))

redis_client = Redis(host=REDIS_HOST, port=REDIS_PORT, db=REDIS_DB)

# Модель події (для прикладу, в реальному проекті буде Pydantic модель)
class Event:
    def __init__(self, id: str, title: str, description: str, latitude: float, longitude: float, category: str, timestamp: str, source_url: str):
        self.id = id
        self.title = title
        self.description = description
        self.latitude = latitude
        self.longitude = longitude
        self.category = category
        self.timestamp = timestamp
        self.source_url = source_url

    def to_dict(self):
        return {
            "id": self.id,
            "title": self.title,
            "description": self.description,
            "coordinates": {"latitude": self.latitude, "longitude": self.longitude},
            "category": self.category,
            "timestamp": self.timestamp,
            "source_url": self.source_url,
        }

# Залежність для отримання клієнта Redis
def get_redis_client():
    try:
        yield redis_client
    finally:
        pass # Redis connection management would be more robust in a real app

@app.get("/events", response_model=List[dict])
async def get_latest_events(redis: Redis = Depends(get_redis_client)):
    """
    Отримує список останніх подій з кешу Redis.
    Якщо кеш порожній, повертає порожній список.
    """
    cached_events = redis.get("latest_events")
    if cached_events:
        return json.loads(cached_events)
    return []

# Приклад додавання подій до кешу Redis (для тестування)
@app.post("/events/add_dummy")
async def add_dummy_event(redis: Redis = Depends(get_redis_client)):
    dummy_event = Event(
        id="123e4567-e89b-12d3-a456-426614174000",
        title="Dummy Conflict Event",
        description="A simulated conflict event for testing purposes.",
        latitude=48.8566,
        longitude=2.3522,
        category="Conflict",
        timestamp="2023-10-27T10:00:00Z",
        source_url="https://example.com/dummy-conflict"
    )
    events = json.loads(redis.get("latest_events") or "[]")
    events.append(dummy_event.to_dict())
    redis.set("latest_events", json.dumps(events))
    return {"message": "Dummy event added to cache"}

```

### 5.2. Фронтенд: `frontend/lib/main.dart` (Flutter)

```dart
import 'package:flutter/material.dart';
import 'package:world_event_tracker/screens/home_screen.dart';
import 'package:world_event_tracker/utils/app_theme.dart';
import 'package:firebase_core/firebase_core.dart';
// import 'firebase_options.dart'; // Створіть цей файл за допомогою `flutterfire configure`

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  // await Firebase.initializeApp(
  //   options: DefaultFirebaseOptions.currentPlatform,
  // );
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'World Event Tracker',
      theme: AppTheme.darkTheme, // Застосування темної теми
      home: const HomeScreen(), // Головний екран додатку
      debugShowCheckedModeBanner: false,
    );
  }
}

```

### 5.3. Фронтенд: `frontend/lib/utils/app_theme.dart` (Flutter Theme)

```dart
import 'package:flutter/material.dart';

class AppTheme {
  static final ThemeData darkTheme = ThemeData(
    brightness: Brightness.dark,
    scaffoldBackgroundColor: Colors.black, // Amoled Black
    primaryColor: Colors.blueGrey[900],
    colorScheme: const ColorScheme.dark(
      primary: Color(0xFF00BCD4), // Cyan accent
      secondary: Color(0xFFFFC107), // Amber accent
      surface: Color(0xFF121212), // Dark surface for cards/dialogs
      background: Colors.black,
      error: Color(0xFFCF6679),
      onPrimary: Colors.white,
      onSecondary: Colors.black,
      onSurface: Colors.white,
      onBackground: Colors.white,
      onError: Colors.black,
    ),
    appBarTheme: const AppBarTheme(
      backgroundColor: Colors.black,
      foregroundColor: Colors.white,
      elevation: 0,
    ),
    textTheme: const TextTheme(
      headlineLarge: TextStyle(color: Colors.white, fontWeight: FontWeight.bold),
      headlineMedium: TextStyle(color: Colors.white70),
      bodyLarge: TextStyle(color: Colors.white),
      bodyMedium: TextStyle(color: Colors.white70),
    ),
    cardTheme: CardTheme(
      color: Colors.grey[900],
      elevation: 5,
      shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(10)),
    ),
    buttonTheme: const ButtonThemeData(
      buttonColor: Color(0xFF00BCD4),
      textTheme: ButtonTextTheme.primary,
    ),
    floatingActionButtonTheme: const FloatingActionButtonThemeData(
      backgroundColor: Color(0xFF00BCD4),
      foregroundColor: Colors.white,
    ),
    // Акцентні кольори для категорій (приклад)
    // Ці кольори можна використовувати для маркерів на карті або фільтрів
    extensions: <ThemeExtension<dynamic>>[
      _CategoryColors(
        conflict: Colors.red[700]!,
        ecology: Colors.green[700]!,
        naturalDisaster: Colors.orange[700]!,
        political: Colors.blue[700]!,
      ),
    ],
  );
}

@immutable
class _CategoryColors extends ThemeExtension<_CategoryColors> {
  const _CategoryColors({
    required this.conflict,
    required this.ecology,
    required this.naturalDisaster,
    required this.political,
  });

  final Color conflict;
  final Color ecology;
  final Color naturalDisaster;
  final Color political;

  @override
  _CategoryColors copyWith({
    Color? conflict,
    Color? ecology,
    Color? naturalDisaster,
    Color? political,
  }) {
    return _CategoryColors(
      conflict: conflict ?? this.conflict,
      ecology: ecology ?? this.ecology,
      naturalDisaster: naturalDisaster ?? this.naturalDisaster,
      political: political ?? this.political,
    );
  }

  @override
  _CategoryColors lerp(ThemeExtension<_CategoryColors>? other, double t) {
    if (other is! _CategoryColors) {
      return this;
    }
    return _CategoryColors(
      conflict: Color.lerp(conflict, other.conflict, t)!,
      ecology: Color.lerp(ecology, other.ecology, t)!,
      naturalDisaster: Color.lerp(naturalDisaster, other.naturalDisaster, t)!,
      political: Color.lerp(political, other.political, t)!,
    );
  }
}

// Допоміжний метод для доступу до кольорів категорій
extension CategoryColorsExtension on ThemeData {
  _CategoryColors get categoryColors => extension<_CategoryColors>()!;
}

```

### 5.4. Фронтенд: `frontend/lib/screens/home_screen.dart` (Flutter Home Screen)

```dart
import 'package:flutter/material.dart';
import 'package:world_event_tracker/widgets/event_map.dart';
import 'package:world_event_tracker/widgets/skeleton_loader.dart';

class HomeScreen extends StatefulWidget {
  const HomeScreen({super.key});

  @override
  State<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends State<HomeScreen> {
  bool _isLoading = true;
  List<dynamic> _events = []; // Тут будуть завантажені події

  @override
  void initState() {
    super.initState();
    _loadEvents();
  }

  Future<void> _loadEvents() async {
    // Імітація завантаження даних
    await Future.delayed(const Duration(seconds: 2));
    setState(() {
      _isLoading = false;
      // Приклад даних, в реальності будуть завантажуватися з API
      _events = [
        {
          "id": "event1",
          "title": "Conflict in Region A",
          "description": "Details about conflict in region A.",
          "coordinates": {"latitude": 48.8566, "longitude": 2.3522},
          "category": "Conflict",
          "timestamp": "2023-10-27T10:00:00Z",
          "source_url": "https://example.com/event1"
        },
        {
          "id": "event2",
          "title": "Forest Fire in National Park",
          "description": "Large forest fire reported.",
          "coordinates": {"latitude": 34.0522, "longitude": -118.2437},
          "category": "Ecology",
          "timestamp": "2023-10-26T15:30:00Z",
          "source_url": "https://example.com/event2"
        },
      ];
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('World Event Tracker'),
        actions: [
          IconButton(
            icon: const Icon(Icons.filter_list),
            onPressed: () {
              // TODO: Реалізувати фільтрацію подій
            },
          ),
          IconButton(
            icon: const Icon(Icons.person),
            onPressed: () {
              // TODO: Перехід до екрану авторизації/профілю
            },
          ),
        ],
      ),
      body: _isLoading
          ? const SkeletonLoader() // Відображення Skeleton Screen під час завантаження
          : EventMap(events: _events), // Відображення карти з подіями
    );
  }
}

```

### 5.5. Фронтенд: `frontend/lib/widgets/event_map.dart` (Flutter Map Widget)

```dart
import 'package:flutter/material.dart';
// import 'package:flutter_map/flutter_map.dart'; // Для Mapbox або іншої бібліотеки карт
// import 'package:latlong2/latlong.dart'; // Для LatLng
import 'package:world_event_tracker/utils/app_theme.dart';

class EventMap extends StatelessWidget {
  final List<dynamic> events;

  const EventMap({super.key, required this.events});

  @override
  Widget build(BuildContext context) {
    // Приклад використання акцентних кольорів з теми
    final categoryColors = Theme.of(context).categoryColors;

    return Center(
      child: Column(
        mainAxisAlignment: MainAxisAlignment.center,
        children: [
          const Text(
            'Map Placeholder (Mapbox/Google Maps integration here)',
            style: TextStyle(fontSize: 18, color: Colors.white70),
          ),
          const SizedBox(height: 20),
          // TODO: Інтегрувати Mapbox або Google Maps тут
          // Приклад відображення маркерів (без реальної карти)
          ...events.map((event) {
            Color markerColor;
            switch (event['category']) {
              case 'Conflict':
                markerColor = categoryColors.conflict;
                break;
              case 'Ecology':
                markerColor = categoryColors.ecology;
                break;
              case 'Natural Disaster':
                markerColor = categoryColors.naturalDisaster;
                break;
              case 'Political':
                markerColor = categoryColors.political;
                break;
              default:
                markerColor = Colors.grey;
            }
            return Padding(
              padding: const EdgeInsets.all(4.0),
              child: Row(
                mainAxisSize: MainAxisSize.min,
                children: [
                  Icon(Icons.location_on, color: markerColor),
                  Text(
                    '${event['title']} (${event['category']})',
                    style: TextStyle(color: markerColor),
                  ),
                ],
              ),
            );
          }).toList(),
        ],
      ),
    );
    /*
    // Приклад інтеграції flutter_map
    return FlutterMap(
      options: MapOptions(
        initialCenter: LatLng(0, 0),
        initialZoom: 2.0,
      ),
      children: [
        TileLayer(
          urlTemplate: 'https://tile.openstreetmap.org/{z}/{x}/{y}.png',
          userAgentPackageName: 'com.example.app',
        ),
        MarkerLayer(
          markers: events.map((event) {
            Color markerColor;
            switch (event['category']) {
              case 'Conflict':
                markerColor = categoryColors.conflict;
                break;
              case 'Ecology':
                markerColor = categoryColors.ecology;
                break;
              case 'Natural Disaster':
                markerColor = categoryColors.naturalDisaster;
                break;
              case 'Political':
                markerColor = categoryColors.political;
                break;
              default:
                markerColor = Colors.grey;
            }
            return Marker(
              point: LatLng(event['coordinates']['latitude'], event['coordinates']['longitude']),
              width: 80,
              height: 80,
              child: Icon(Icons.location_on, color: markerColor, size: 40),
            );
          }).toList(),
        ),
      ],
    );
    */
  }
}

```

### 5.6. Фронтенд: `frontend/lib/widgets/skeleton_loader.dart` (Flutter Skeleton Screen)

```dart
import 'package:flutter/material.dart';

class SkeletonLoader extends StatelessWidget {
  const SkeletonLoader({super.key});

  @override
  Widget build(BuildContext context) {
    return ListView.builder(
      itemCount: 5, // Кількість елементів-заповнювачів
      itemBuilder: (context, index) {
        return Padding(
          padding: const EdgeInsets.all(8.0),
          child: Card(
            color: Colors.grey[900], // Темний колір для заповнювача
            child: const Padding(
              padding: EdgeInsets.all(16.0),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  // Заповнювач для заголовка
                  _SkeletonBox(width: 200, height: 20),
                  SizedBox(height: 10),
                  // Заповнювач для опису
                  _SkeletonBox(width: double.infinity, height: 15),
                  SizedBox(height: 5),
                  _SkeletonBox(width: double.infinity, height: 15),
                  SizedBox(height: 5),
                  _SkeletonBox(width: 150, height: 15),
                ],
              ),
            ),
          ),
        );
      },
    );
  }
}

class _SkeletonBox extends StatelessWidget {
  final double width;
  final double height;

  const _SkeletonBox({required this.width, required this.height});

  @override
  Widget build(BuildContext context) {
    return Container(
      width: width,
      height: height,
      decoration: BoxDecoration(
        color: Colors.grey[800], // Світліший сірий для анімації
        borderRadius: BorderRadius.circular(4),
      ),
    );
  }
}

```

## 6. Подальші кроки

1.  **Налаштування Firebase**: Для авторизації та push-повідомлень необхідно налаштувати проект Firebase та згенерувати `firebase_options.dart` для Flutter.
2.  **Інтеграція карт**: Вибрати та інтегрувати картографічний сервіс (Mapbox або Google Maps) у `EventMap` віджет.
3.  **Реалізація API**: Доповнити бекенд API для CRUD операцій з подіями, авторизації та підписки на FCM топіки.
4.  **База даних**: Налаштувати PostgreSQL з PostGIS та інтегрувати її з FastAPI.
5.  **Docker**: Використовувати `docker-compose.yml` для розгортання бекенду, Redis та PostgreSQL.


## 7. Посилання

*   [FastAPI Documentation](https://fastapi.tiangolo.com/)
*   [Flutter Documentation](https://docs.flutter.dev/)
*   [PostgreSQL Documentation](https://www.postgresql.org/docs/)
*   [PostGIS Documentation](https://postgis.net/documentation/)
*   [Redis Documentation](https://redis.io/docs/)
*   [Firebase Documentation](https://firebase.google.com/docs)
*   [Mapbox Flutter SDK](https://docs.mapbox.com/mapbox-gl-js/api/)
*   [Google Maps Flutter](https://pub.dev/packages/google_maps_flutter)
