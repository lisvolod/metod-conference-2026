# Express.js для початківців - Частина 3: CORS та структура проекту 🌐

## Що таке CORS?

### Cross-Origin Resource Sharing

**CORS** - це механізм безпеки браузера, який контролює які сайти можуть звертатися до вашого API.

### Проблема:

```
Frontend:  http://localhost:5173  (React/Vite)
    ↓ AJAX запит
Backend:   http://localhost:3000  (Express)
    ↓
❌ CORS Error: "Access to fetch has been blocked by CORS policy"
```

### Чому виникає помилка?

**Same-Origin Policy** - правило безпеки браузера:
- Frontend на порту 5173
- Backend на порту 3000
- Це **різні origin** (різні порти) → браузер блокує запит

**Аналогія:**
Як паспортний контроль на кордоні - без дозволу не пропустять!

---

## Як працює CORS?

### Процес:

```
1. Frontend відправляє запит:
   GET http://localhost:3000/api/users

2. Браузер додає заголовок:
   Origin: http://localhost:5173

3. Backend відповідає з заголовками:
   Access-Control-Allow-Origin: http://localhost:5173
   Access-Control-Allow-Methods: GET, POST, PUT, DELETE

4. Браузер перевіряє заголовки:
   ✅ Origin дозволений → дані доходять до frontend
   ❌ Origin заборонений → блокує відповідь
```

---

## Встановлення CORS

### Крок 1: Встановити пакет

```bash
npm install cors
```

### Крок 2: Підключити в Express

```javascript
const express = require('express');
const cors = require('cors');

const app = express();

// ✅ Дозволити запити з БУДЬ-ЯКИХ доменів (для розробки)
app.use(cors());

app.get('/api/users', (req, res) => {
  res.json([
    { id: 1, name: 'Іван' },
    { id: 2, name: 'Марія' }
  ]);
});

app.listen(3000);
```

**Тепер frontend може робити запити!** ✅

---

## Налаштування CORS

### Варіант 1: Дозволити всі домени (НЕ для production!)

```javascript
app.use(cors());
```

**Небезпечно для production!** Будь-хто може звертатися до вашого API.

### Варіант 2: Дозволити конкретні домени

```javascript
const corsOptions = {
  origin: 'http://localhost:5173', // Тільки цей домен
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  credentials: true // Дозволити cookies
};

app.use(cors(corsOptions));
```

### Варіант 3: Багато доменів

```javascript
const allowedOrigins = [
  'http://localhost:5173',
  'http://localhost:3000',
  'https://mysite.com'
];

const corsOptions = {
  origin: function (origin, callback) {
    // Дозволити запити без origin (наприклад, Postman)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
};

app.use(cors(corsOptions));
```

### Варіант 4: Різні налаштування для різних маршрутів

```javascript
const cors = require('cors');

// Публічні маршрути - всім дозволено
app.get('/public', cors(), (req, res) => {
  res.json({ message: 'Публічні дані' });
});

// Приватні маршрути - тільки наш frontend
const privateCors = cors({
  origin: 'http://localhost:5173'
});

app.get('/private', privateCors, (req, res) => {
  res.json({ message: 'Приватні дані' });
});
```

---

## Preflight запити (OPTIONS)

### Що це?

Для складних запитів (PUT, DELETE, або з кастомними заголовками) браузер спочатку відправляє **OPTIONS** запит для перевірки дозволів.

### Приклад:

```
1. Frontend робить: DELETE /api/users/123
2. Браузер спочатку відправляє: OPTIONS /api/users/123
3. Backend відповідає: "DELETE дозволено"
4. Браузер відправляє фактичний: DELETE /api/users/123
```

### CORS автоматично обробляє OPTIONS:

```javascript
const cors = require('cors');
app.use(cors());

// CORS автоматично відповідає на OPTIONS запити
app.delete('/api/users/:id', (req, res) => {
  res.json({ message: 'Видалено' });
});
```

---

## Організація структури проекту

### Проблема:

Весь код в одному файлі `server.js` стає нечитабельним:

```javascript
// server.js - 500+ рядків коду 😱
const express = require('express');
const app = express();

app.get('/users', ...);
app.post('/users', ...);
app.get('/products', ...);
app.post('/products', ...);
app.get('/orders', ...);
// ... ще 100 маршрутів ...
```

### Рішення: Розділити на модулі

```
project/
├── server.js           ← Головний файл
├── routes/             ← Маршрути
│   ├── users.js
│   ├── products.js
│   └── orders.js
├── controllers/        ← Логіка обробки
│   ├── userController.js
│   ├── productController.js
│   └── orderController.js
├── middleware/         ← Middleware функції
│   ├── auth.js
│   └── errorHandler.js
└── config/            ← Налаштування
    └── db.js
```

---

## Express Router

### Що це?

**Router** - це міні-додаток Express для організації маршрутів.

### Створюємо окремий файл для маршрутів:

#### `routes/users.js`

```javascript
const express = require('express');
const router = express.Router();

// GET /api/users
router.get('/', (req, res) => {
  res.json([
    { id: 1, name: 'Іван' },
    { id: 2, name: 'Марія' }
  ]);
});

// GET /api/users/:id
router.get('/:id', (req, res) => {
  const userId = req.params.id;
  res.json({ id: userId, name: 'Іван' });
});

// POST /api/users
router.post('/', (req, res) => {
  const { name, email } = req.body;
  res.json({ 
    message: 'Користувача створено',
    user: { name, email }
  });
});

// DELETE /api/users/:id
router.delete('/:id', (req, res) => {
  res.json({ message: 'Користувача видалено' });
});

module.exports = router;
```

#### `server.js`

```javascript
const express = require('express');
const cors = require('cors');
const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Імпортуємо роутер
const usersRouter = require('./routes/users');

// Підключаємо роутер з префіксом
app.use('/api/users', usersRouter);

app.listen(3000, () => {
  console.log('Сервер на http://localhost:3000');
});
```

**Тепер маршрути:**
- `GET /api/users` → `router.get('/', ...)`
- `GET /api/users/123` → `router.get('/:id', ...)`
- `POST /api/users` → `router.post('/', ...)`

---

## Контролери (Controllers)

### Виносимо логіку в окремі функції:

#### `controllers/userController.js`

```javascript
// Отримати всіх користувачів
exports.getAllUsers = (req, res) => {
  // Тут може бути запит до бази даних
  const users = [
    { id: 1, name: 'Іван', email: 'ivan@example.com' },
    { id: 2, name: 'Марія', email: 'maria@example.com' }
  ];
  res.json(users);
};

// Отримати одного користувача
exports.getUserById = (req, res) => {
  const userId = req.params.id;
  const user = { id: userId, name: 'Іван', email: 'ivan@example.com' };
  res.json(user);
};

// Створити користувача
exports.createUser = (req, res) => {
  const { name, email } = req.body;
  
  // Валідація
  if (!name || !email) {
    return res.status(400).json({ error: 'Name та email обов\'язкові' });
  }
  
  const newUser = { id: Date.now(), name, email };
  res.status(201).json(newUser);
};

// Оновити користувача
exports.updateUser = (req, res) => {
  const userId = req.params.id;
  const { name, email } = req.body;
  
  const updatedUser = { id: userId, name, email };
  res.json(updatedUser);
};

// Видалити користувача
exports.deleteUser = (req, res) => {
  const userId = req.params.id;
  res.json({ message: `Користувача ${userId} видалено` });
};
```

#### `routes/users.js` (оновлений)

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

// Маршрути використовують функції з контролера
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

**Чистіше та зрозуміліше!** ✨

---

## Environment Variables (Змінні середовища)

### Проблема:

```javascript
// ❌ Погана практика - хардкодимо значення
const PORT = 3000;
const DB_URL = 'mongodb://localhost:27017/mydb';
const JWT_SECRET = 'super-secret-key-123';
```

Коли переносите на production, треба міняти код!

### Рішення: .env файл

#### Крок 1: Встановити dotenv

```bash
npm install dotenv
```

#### Крок 2: Створити файл `.env`

```env
PORT=3000
NODE_ENV=development
DB_URL=mongodb://localhost:27017/mydb
JWT_SECRET=my-super-secret-key-change-in-production
```

#### Крок 3: Завантажити змінні

```javascript
// server.js
require('dotenv').config();

const express = require('express');
const app = express();

// Читаємо змінні з process.env
const PORT = process.env.PORT || 3000;
const DB_URL = process.env.DB_URL;
const JWT_SECRET = process.env.JWT_SECRET;

console.log('Port:', PORT);
console.log('Database:', DB_URL);

app.listen(PORT, () => {
  console.log(`Сервер на порту ${PORT}`);
});
```

#### Крок 4: Додати .env в .gitignore

```
# .gitignore
node_modules/
.env
```

**Чому?** `.env` містить секретні ключі які не можна публікувати на GitHub!

---

## Повна структура проекту

```
ecommerce-backend/
├── node_modules/         ← npm пакети (не комітити)
├── config/               ← Налаштування
│   └── db.js            ← Підключення до MongoDB
├── controllers/          ← Бізнес-логіка
│   ├── authController.js
│   ├── productController.js
│   └── userController.js
├── middleware/           ← Middleware функції
│   ├── auth.js          ← Перевірка JWT
│   └── errorHandler.js  ← Обробка помилок
├── models/              ← Mongoose моделі
│   ├── User.js
│   └── Product.js
├── routes/              ← Маршрути
│   ├── auth.js
│   ├── products.js
│   └── users.js
├── utils/               ← Допоміжні функції
│   └── jwt.js           ← Робота з токенами
├── .env                 ← Змінні середовища (не комітити!)
├── .gitignore           ← Ігнорувати файли для Git
├── package.json         ← npm залежності
├── package-lock.json
└── server.js            ← Головний файл
```

---

## Приклад повної структури

### `server.js` (головний файл)

```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');

const app = express();

// Middleware
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
const authRoutes = require('./routes/auth');
const productRoutes = require('./routes/products');
const userRoutes = require('./routes/users');

app.use('/api/auth', authRoutes);
app.use('/api/products', productRoutes);
app.use('/api/users', userRoutes);

// Error handler (в кінці!)
const errorHandler = require('./middleware/errorHandler');
app.use(errorHandler);

// Start server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Сервер запущено на порту ${PORT}`);
});
```

### `routes/products.js`

```javascript
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');
const auth = require('../middleware/auth');

// Публічні маршрути
router.get('/', productController.getAllProducts);
router.get('/:id', productController.getProductById);

// Захищені маршрути (потрібна авторизація)
router.post('/', auth, productController.createProduct);
router.put('/:id', auth, productController.updateProduct);
router.delete('/:id', auth, productController.deleteProduct);

module.exports = router;
```

### `controllers/productController.js`

```javascript
exports.getAllProducts = (req, res) => {
  // TODO: Отримати з бази даних
  res.json([
    { id: 1, name: 'Ноутбук', price: 20000 },
    { id: 2, name: 'Мишка', price: 500 }
  ]);
};

exports.getProductById = (req, res) => {
  const productId = req.params.id;
  // TODO: Знайти в базі даних
  res.json({ id: productId, name: 'Ноутбук', price: 20000 });
};

exports.createProduct = (req, res) => {
  const { name, price } = req.body;
  
  // Валідація
  if (!name || !price) {
    return res.status(400).json({ error: 'Name та price обов\'язкові' });
  }
  
  // TODO: Зберегти в базі даних
  res.status(201).json({ id: Date.now(), name, price });
};

exports.updateProduct = (req, res) => {
  const productId = req.params.id;
  const { name, price } = req.body;
  
  // TODO: Оновити в базі даних
  res.json({ id: productId, name, price });
};

exports.deleteProduct = (req, res) => {
  const productId = req.params.id;
  
  // TODO: Видалити з бази даних
  res.json({ message: `Продукт ${productId} видалено` });
};
```

### `middleware/errorHandler.js`

```javascript
module.exports = (err, req, res, next) => {
  console.error('Помилка:', err);
  
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Внутрішня помилка сервера';
  
  res.status(statusCode).json({
    error: message,
    ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
  });
};
```

---

## Практичне завдання 3

### Створіть структурований проект:

```
my-api/
├── routes/
│   ├── books.js
│   └── authors.js
├── controllers/
│   ├── bookController.js
│   └── authorController.js
├── middleware/
│   ├── logger.js
│   └── errorHandler.js
├── .env
├── .gitignore
└── server.js
```

### Вимоги:

1. **CORS** налаштувати для `http://localhost:5173`
2. **Environment variables** для PORT
3. **Роутери** для books та authors
4. **Контролери** з CRUD операціями
5. **Logger middleware** для всіх запитів
6. **Error handler** в кінці

### API endpoints:

```
GET    /api/books
GET    /api/books/:id
POST   /api/books
PUT    /api/books/:id
DELETE /api/books/:id

GET    /api/authors
GET    /api/authors/:id
POST   /api/authors
```

---

## Підсумок Частини 3

✅ **Що ми дізналися:**
- Що таке CORS і чому він потрібен
- Як налаштувати CORS в Express
- Як організувати структуру проекту
- Express Router для модульності
- Controllers для бізнес-логіки
- Environment variables для налаштувань

✅ **Що вміємо:**
- Вирішувати CORS проблеми
- Розділяти код на модулі (routes, controllers)
- Використовувати .env файли
- Створювати чисту архітектуру проекту

📚 **Наступна частина:**
- MongoDB та Mongoose
- Підключення до бази даних
- Створення схем та моделей
- CRUD операції з базою даних

---

**Питання для самоперевірки:**

1. Що таке CORS і коли виникає помилка?
2. Навіщо потрібен Express Router?
3. Яка різниця між routes та controllers?
4. Чому .env файл не треба комітити в Git?
5. Де має бути error-handling middleware?

**Вперед до Частини 4! 🚀**
