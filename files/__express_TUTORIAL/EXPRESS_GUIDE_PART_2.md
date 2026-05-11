# Express.js для початківців - Частина 2: Middleware (Проміжні обробники) 🔄

## Що таке Middleware?

### Концепція конвеєра

**Middleware** (міддлвеєр) - це функції які виконуються **між** отриманням запиту та відправкою відповіді.

**Аналогія з аеропортом:**
```
Пасажир (запит) → Реєстрація → Перевірка паспорта → Контроль безпеки → Посадка (відповідь)
                      ↑              ↑                  ↑
                  Middleware 1   Middleware 2      Middleware 3
```

**В Express:**
```
Запит → Middleware 1 → Middleware 2 → Middleware 3 → Route Handler → Відповідь
```

### Візуальна схема:

```
HTTP Request
    ↓
┌─────────────────┐
│  Middleware 1   │ ← Логування (console.log)
│  (logging)      │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Middleware 2   │ ← Парсинг JSON
│  (body parser)  │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Middleware 3   │ ← Перевірка авторизації
│  (auth check)   │
└────────┬────────┘
         ↓
┌─────────────────┐
│  Route Handler  │ ← Ваша логіка (app.get, app.post)
│  (your code)    │
└────────┬────────┘
         ↓
HTTP Response
```

---

## Перший простий Middleware

### Приклад: Логування запитів

```javascript
const express = require('express');
const app = express();

// Middleware функція
function logger(req, res, next) {
  console.log(`${req.method} ${req.url}`);
  next(); // ← ВАЖЛИВО! Передаємо управління далі
}

// Використовуємо middleware для ВСІХ маршрутів
app.use(logger);

// Маршрути
app.get('/', (req, res) => {
  res.send('Головна');
});

app.get('/about', (req, res) => {
  res.send('Про нас');
});

app.listen(3000);
```

**Що відбувається:**
1. Запит приходить на `/`
2. Express викликає `logger` middleware
3. `logger` виводить в консоль: `GET /`
4. `next()` передає управління далі
5. Express викликає route handler `app.get('/', ...)`
6. Відправляється відповідь

---

## Структура Middleware функції

```javascript
function middlewareName(req, res, next) {
  // 1. Робимо щось з запитом (req)
  console.log('Middleware виконується');
  
  // 2. Можемо модифікувати req або res
  req.customProperty = 'Моє значення';
  
  // 3. ОБОВ'ЯЗКОВО викликаємо next() або відправляємо відповідь
  next(); // Передаємо управління далі
  
  // АБО
  // res.send('Зупиняємо обробку тут');
}
```

### Три параметри:
- **req** - об'єкт запиту (дані від клієнта)
- **res** - об'єкт відповіді (що відправити клієнту)
- **next** - функція для передачі управління наступному middleware

### ⚠️ Важливо про `next()`

**Якщо НЕ викликати `next()`:**
```javascript
app.use((req, res, next) => {
  console.log('Middleware без next()');
  // Забули викликати next()
});

app.get('/', (req, res) => {
  res.send('Ця відповідь НІКОЛИ не надійде!');
});
```
Результат: Запит **зависне** і браузер чекатиме вічно! ⏳

**Правильно:**
```javascript
app.use((req, res, next) => {
  console.log('Middleware з next()');
  next(); // ✅ Передаємо далі
});
```

---

## Типи Middleware

### 1. Application-level (на рівні додатку)

Виконується для **всіх** маршрутів:

```javascript
// Для ВСІХ запитів
app.use((req, res, next) => {
  console.log('Час запиту:', Date.now());
  next();
});
```

### 2. Router-level (на рівні роутера)

Виконується тільки для певного шляху:

```javascript
// Тільки для маршрутів які починаються з /api
app.use('/api', (req, res, next) => {
  console.log('API запит');
  next();
});
```

### 3. Built-in (вбудовані)

Express має вбудовані middleware:

```javascript
// Для обробки JSON
app.use(express.json());

// Для обробки даних форм
app.use(express.urlencoded({ extended: true }));

// Для статичних файлів (CSS, images, JS)
app.use(express.static('public'));
```

### 4. Third-party (сторонні бібліотеки)

Встановлюються через npm:

```javascript
const cors = require('cors');
const morgan = require('morgan');

app.use(cors());        // Дозволяє запити з інших доменів
app.use(morgan('dev')); // Логування запитів
```

---

## Порядок виконання Middleware

### ⚠️ Порядок має значення!

```javascript
const express = require('express');
const app = express();

// 1. Виконається ПЕРШИМ
app.use((req, res, next) => {
  console.log('1. Перший middleware');
  next();
});

// 2. Виконається ДРУГИМ
app.use((req, res, next) => {
  console.log('2. Другий middleware');
  next();
});

// 3. Виконається ТРЕТІМ
app.get('/', (req, res) => {
  console.log('3. Route handler');
  res.send('OK');
});

app.listen(3000);
```

**Вивід в консолі:**
```
1. Перший middleware
2. Другий middleware
3. Route handler
```

### Неправильний порядок:

```javascript
// ❌ ПОМИЛКА: маршрут ПЕРЕД middleware
app.get('/', (req, res) => {
  res.send('Hello');
});

// Цей middleware НІКОЛИ не виконається для '/'
app.use((req, res, next) => {
  console.log('Цей код не виконається');
  next();
});
```

**Правило:** Middleware має бути **ДО** маршрутів!

---

## Express.json() - Парсинг JSON

### Проблема:

```javascript
app.post('/users', (req, res) => {
  console.log(req.body); // undefined ❌
  res.send('OK');
});
```

Коли клієнт відправляє JSON, Express не розуміє його автоматично!

### Рішення: express.json()

```javascript
const express = require('express');
const app = express();

// ✅ Додаємо middleware для парсингу JSON
app.use(express.json());

app.post('/users', (req, res) => {
  console.log(req.body); // { name: "Іван", age: 25 } ✅
  
  const { name, age } = req.body;
  res.json({ 
    message: 'Користувача створено',
    user: { name, age }
  });
});

app.listen(3000);
```

**Тестування в Postman:**
```
POST http://localhost:3000/users
Headers: Content-Type: application/json
Body (raw JSON):
{
  "name": "Іван",
  "age": 25
}
```

---

## Express.urlencoded() - Парсинг форм

### Коли використовувати:

Коли дані приходять з HTML форми:

```html
<form action="/login" method="POST">
  <input type="text" name="email">
  <input type="password" name="password">
  <button type="submit">Увійти</button>
</form>
```

### Код:

```javascript
const express = require('express');
const app = express();

// ✅ Для обробки даних з форм
app.use(express.urlencoded({ extended: true }));

app.post('/login', (req, res) => {
  const { email, password } = req.body;
  
  console.log('Email:', email);
  console.log('Password:', password);
  
  res.send('Ви увійшли!');
});

app.listen(3000);
```

**Що робить `extended: true`?**
- `true` - дозволяє складні об'єкти (arrays, nested objects)
- `false` - тільки прості пари key-value

---

## Приклад: Middleware для логування

### Створюємо власний logger:

```javascript
const express = require('express');
const app = express();

// Middleware для детального логування
function requestLogger(req, res, next) {
  const timestamp = new Date().toISOString();
  const method = req.method;
  const url = req.url;
  const ip = req.ip;
  
  console.log(`[${timestamp}] ${method} ${url} - IP: ${ip}`);
  next();
}

app.use(requestLogger);

app.get('/', (req, res) => {
  res.send('Головна');
});

app.listen(3000);
```

**Вивід в консолі:**
```
[2024-01-08T12:30:45.123Z] GET / - IP: ::1
```

---

## Middleware з параметрами

### Створюємо middleware-фабрику:

```javascript
// Функція яка повертає middleware
function logWithPrefix(prefix) {
  return function(req, res, next) {
    console.log(`[${prefix}] ${req.method} ${req.url}`);
    next();
  };
}

// Використання
app.use(logWithPrefix('API'));

app.get('/', (req, res) => {
  res.send('OK');
});
```

**Вивід:**
```
[API] GET /
```

---

## Middleware для конкретних маршрутів

### Варіант 1: Inline middleware

```javascript
// Middleware тільки для цього маршруту
function checkAuth(req, res, next) {
  const token = req.headers.authorization;
  
  if (!token) {
    return res.status(401).json({ error: 'Немає токену' });
  }
  
  // Якщо токен є - продовжуємо
  next();
}

// Використання: другий параметр = middleware
app.get('/profile', checkAuth, (req, res) => {
  res.json({ user: 'Іван' });
});

// Без middleware - доступно всім
app.get('/', (req, res) => {
  res.send('Публічна сторінка');
});
```

### Варіант 2: Декілька middleware

```javascript
function middleware1(req, res, next) {
  console.log('Middleware 1');
  next();
}

function middleware2(req, res, next) {
  console.log('Middleware 2');
  next();
}

function middleware3(req, res, next) {
  console.log('Middleware 3');
  next();
}

// Масив middleware
app.get('/test', 
  [middleware1, middleware2, middleware3], 
  (req, res) => {
    console.log('Route handler');
    res.send('OK');
  }
);
```

**Вивід:**
```
Middleware 1
Middleware 2
Middleware 3
Route handler
```

---

## Error-handling Middleware

### Спеціальний middleware з 4 параметрами:

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// Звичайні маршрути
app.get('/', (req, res) => {
  res.send('OK');
});

app.get('/error', (req, res) => {
  throw new Error('Щось пішло не так!');
});

// ⚠️ Error middleware ЗАВЖДИ в кінці!
app.use((err, req, res, next) => {
  console.error('Помилка:', err.message);
  
  res.status(500).json({
    error: 'Внутрішня помилка сервера',
    message: err.message
  });
});

app.listen(3000);
```

**Параметри error middleware:**
- `err` - об'єкт помилки
- `req` - запит
- `res` - відповідь
- `next` - наступний middleware

---

## Практичне завдання 2

### Створіть сервер з такими middleware:

1. **Logger** - виводить час, метод, URL кожного запиту
2. **JSON Parser** - обробляє JSON дані
3. **Auth Checker** - перевіряє наявність заголовка `x-api-key`
4. **Error Handler** - ловить всі помилки

### Маршрути:

```javascript
// Публічний (без auth)
GET /

// Приватний (з auth)
GET /profile
POST /users

// Маршрут з помилкою
GET /error
```

### Підказка:

```javascript
const express = require('express');
const app = express();

// TODO: Додати middleware

app.get('/', (req, res) => {
  res.send('Публічна сторінка');
});

app.get('/profile', /* auth middleware */, (req, res) => {
  res.json({ user: 'Іван' });
});

app.get('/error', (req, res) => {
  throw new Error('Тестова помилка');
});

// TODO: Додати error handler

app.listen(3000);
```

---

## Підсумок Частини 2

✅ **Що ми дізналися:**
- Що таке middleware та навіщо він потрібен
- Як працює конвеєр middleware
- Важливість функції `next()`
- Вбудовані middleware (express.json, express.urlencoded)
- Порядок виконання middleware
- Error-handling middleware

✅ **Що вміємо:**
- Створювати власні middleware
- Використовувати middleware для логування
- Парсити JSON та form data
- Обробляти помилки
- Застосовувати middleware до конкретних маршрутів

📚 **Наступна частина:**
- CORS (Cross-Origin Resource Sharing)
- Робота зі статичними файлами
- Організація коду (роутери)
- Environment variables

---

**Питання для самоперевірки:**

1. Що таке middleware і як він працює?
2. Що станеться якщо забути викликати `next()`?
3. Чи має значення порядок middleware?
4. Яка різниця між `express.json()` та `express.urlencoded()`?
5. Скільки параметрів має error-handling middleware?

**Вперед до Частини 3! 🚀**
