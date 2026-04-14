# World Event Tracker: Web Edition

Ваш проект на **Flutter** уже підтримує Web-платформу. Ось як запустити його як повноцінний сайт та розгорнути його.

## 1. Запуск у режимі розробки (Web)

Щоб побачити сайт локально, виконайте:

1. Перейдіть до папки фронтенду:
   ```bash
   cd frontend
   ```
2. Активуйте підтримку Web (якщо ще не зроблено):
   ```bash
   flutter config --enable-web
   ```
3. Запустіть у браузері Chrome:
   ```bash
   flutter run -d chrome
   ```

---

## 2. Налаштування для Web (Mapbox & Firebase)

Для коректної роботи карти та авторизації у браузері:

### Mapbox у Web
Додайте скрипт Mapbox у файл `frontend/web/index.html` перед тегом `</head>`:
```html
<script src='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.js'></script>
<link href='https://api.mapbox.com/mapbox-gl-js/v2.15.0/mapbox-gl.css' rel='stylesheet' />
```

### Firebase Auth у Web
Переконайтеся, що ви додали "Web App" у вашому Firebase Console та оновили конфігурацію:
```bash
flutterfire configure
```
Це згенерує `firebase_options.dart` з ключами для вебу.

---

## 3. Збірка для Production

Щоб створити оптимізовані файли сайту:
```bash
flutter build web --release --web-renderer canvaskit
```
*Примітка: `--web-renderer canvaskit` забезпечує кращу продуктивність для складних карт.*

Результат збірки буде у папці `frontend/build/web`. Ці файли можна завантажити на будь-який хостинг (Firebase Hosting, Vercel, Netlify або власний сервер).

---

## 4. Хостинг через Docker (Nginx)

Я підготував `Dockerfile.web` для запуску вашого сайту на власному сервері.

### Як запустити сайт у Docker:
1. Зберіть образ:
   ```bash
   docker build -f Dockerfile.web -t tracker-web .
   ```
2. Запустіть контейнер:
   ```bash
   docker run -d -p 80:80 tracker-web
   ```
Тепер ваш сайт буде доступний за адресою вашого сервера (або `http://localhost`).

---

## 5. Поради для Веб-версії (DeepState Style)

1. **Responsive Design:** У Flutter використовуйте `LayoutBuilder`, щоб інтерфейс адаптувався під великі монітори (бічна панель з подіями) та мобільні браузери.
2. **SEO:** Flutter Web — це Single Page Application (SPA). Для кращого SEO використовуйте мета-теги у `index.html`.
3. **PWA:** Flutter автоматично створює `manifest.json`, тому ваш сайт можна буде "встановити" як додаток на робочий стіл.
