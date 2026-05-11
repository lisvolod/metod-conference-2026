# Express.js для початківців - Частина 6: JWT авторизація 🎫

## Проблема: Як підтримувати сесію користувача?

### Сценарій:

```
1. Користувач логінується ✅
2. Отримує відповідь "Вхід успішний" ✅
3. Хоче переглянути свій профіль...
   
   Запит: GET /api/profile
   Сервер: "Хто ти? Я не знаю тебе!" ❌
```

**Проблема:** HTTP протокол **stateless** (не зберігає стан).

Кожен запит - окремий. Сервер не пам'ятає що користувач вже логінився!

### Традиційне рішення: Sessions + Cookies

```
1. Логін → Сервер створює сесію → ID сесії в cookie
2. Наступні запити → Cookie автоматично відправляється
3. Сервер читає ID → знаходить сесію → знає хто це
```

**Недоліки:**
- Треба зберігати сесії на сервері (пам'ять/БД)
- Не підходить для мікросервісів
- Проблеми з масштабуванням

### Сучасне рішення: JWT (JSON Web Tokens)

```
1. Логін → Сервер створює токен → відправляє клієнту
2. Клієнт зберігає токен (localStorage/cookie)
3. Наступні запити → Клієнт відправляє токен в заголовку
4. Сервер перевіряє токен → знає хто це
```

**Переваги:**
- Не треба зберігати сесії на сервері ✅
- Працює з мікросервісами ✅
- Легко масштабується ✅

---

## Що таке JWT?

### Структура JWT

JWT складається з **трьох частин**, розділених крапкою:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiI2NTRhYmMxMjMiLCJlbWFpbCI6Iml2YW5AZXhhbXBsZS5jb20ifQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
└──────────── Header ──────────────┘ └────────────── Payload ───────────────────┘ └───────── Signature ──────┘
```

### 1. Header (заголовок)

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Закодовано в Base64:**
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

### 2. Payload (дані)

```json
{
  "userId": "654abc123",
  "email": "ivan@example.com",
  "role": "user",
  "iat": 1704723600,
  "exp": 1704810000
}
```

**Закодовано в Base64:**
```
eyJ1c2VySWQiOiI2NTRhYmMxMjMiLCJlbWFpbCI6Iml2YW5AZXhhbXBsZS5jb20ifQ
```

**Стандартні поля (claims):**
- `iat` (issued at) - час створення
- `exp` (expiration) - час закінчення
- `sub` (subject) - тема (зазвичай userId)
- `iss` (issuer) - хто створив токен

### 3. Signature (підпис)

```
HMACSHA256(
  base64(header) + "." + base64(payload),
  secret_key
)
```

**Результат:**
```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### ⚠️ Важливо:

**JWT НЕ зашифрований!** Його можна декодувати на https://jwt.io/

**Тому:**
- ❌ НЕ зберігайте паролі в JWT
- ❌ НЕ зберігайте конфіденційні дані
- ✅ Зберігайте тільки userId, email, role

**Безпека забезпечується підписом:**
- Якщо хтось змінить payload → підпис не співпаде → токен недійсний

---

## Встановлення jsonwebtoken

```bash
npm install jsonwebtoken
```

---

## Створення JWT токенів

### Базовий приклад:

```javascript
const jwt = require('jsonwebtoken');

// Секретний ключ (має бути в .env!)
const SECRET_KEY = 'my-super-secret-key-change-in-production';

// Створити токен
function createToken() {
  const payload = {
    userId: '654abc123',
    email: 'ivan@example.com',
    role: 'user'
  };
  
  const token = jwt.sign(payload, SECRET_KEY, {
    expiresIn: '7d' // Токен дійсний 7 днів
  });
  
  console.log('Token:', token);
  return token;
}

const token = createToken();
```

### Варіанти expiresIn:

```javascript
expiresIn: '1h'      // 1 година
expiresIn: '2d'      // 2 дні
expiresIn: '7d'      // 7 днів
expiresIn: '30d'     // 30 днів
expiresIn: 3600      // 3600 секунд = 1 година
```

**Рекомендації:**
- **Access token:** 15 хвилин - 1 година
- **Refresh token:** 7-30 днів

---

## Перевірка JWT токенів

### Декодування та перевірка:

```javascript
const jwt = require('jsonwebtoken');

const SECRET_KEY = 'my-super-secret-key';
const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';

try {
  // Перевірити та декодувати токен
  const decoded = jwt.verify(token, SECRET_KEY);
  
  console.log('Токен дійсний!');
  console.log('User ID:', decoded.userId);
  console.log('Email:', decoded.email);
  console.log('Expires:', new Date(decoded.exp * 1000));
} catch (error) {
  console.log('Токен недійсний:', error.message);
  // Можливі помилки:
  // - "jwt expired" - токен прострочений
  // - "invalid signature" - токен підроблений
  // - "jwt malformed" - невірний формат
}
```

---

## Інтеграція JWT з авторизацією

### Оновлюємо `.env`

```env
PORT=3000
MONGO_URI=mongodb://localhost:27017/mydb
JWT_SECRET=your-super-secret-jwt-key-change-in-production-min-32-chars
JWT_EXPIRE=7d
```

### Створюємо `utils/jwt.js`

```javascript
const jwt = require('jsonwebtoken');

// Створити токен
exports.generateToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: process.env.JWT_EXPIRE }
  );
};

// Перевірити токен
exports.verifyToken = (token) => {
  try {
    return jwt.verify(token, process.env.JWT_SECRET);
  } catch (error) {
    return null;
  }
};
```

### Оновлюємо контролер авторизації

#### `controllers/authController.js`

```javascript
const User = require('../models/User');
const { generateToken } = require('../utils/jwt');

exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Перевірка існування
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email вже використовується' });
    }
    
    // Створення користувача
    const user = await User.create({ name, email, password });
    
    // Створення токену
    const token = generateToken(user._id);
    
    res.status(201).json({
      message: 'Реєстрація успішна',
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Валідація
    if (!email || !password) {
      return res.status(400).json({ error: 'Email та пароль обов\'язкові' });
    }
    
    // Знайти користувача
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // Перевірити пароль
    const isMatch = await user.comparePassword(password);
    if (!isMatch) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // Створення токену
    const token = generateToken(user._id);
    
    res.json({
      message: 'Вхід успішний',
      token,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

---

## Middleware для захисту маршрутів

### Створюємо `middleware/auth.js`

```javascript
const jwt = require('jsonwebtoken');
const User = require('../models/User');

// Перевірка чи користувач авторизований
exports.protect = async (req, res, next) => {
  try {
    let token;
    
    // 1. Отримати токен з заголовку
    if (req.headers.authorization && req.headers.authorization.startsWith('Bearer')) {
      token = req.headers.authorization.split(' ')[1];
    }
    
    // 2. Перевірити чи токен існує
    if (!token) {
      return res.status(401).json({ error: 'Необхідна авторизація' });
    }
    
    // 3. Верифікувати токен
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 4. Знайти користувача
    const user = await User.findById(decoded.userId).select('-password');
    if (!user) {
      return res.status(401).json({ error: 'Користувача не знайдено' });
    }
    
    // 5. Додати користувача в req
    req.user = user;
    next();
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Недійсний токен' });
    }
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Токен прострочений' });
    }
    res.status(500).json({ error: error.message });
  }
};

// Перевірка ролі (тільки для admin)
exports.adminOnly = (req, res, next) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({ error: 'Доступ заборонений' });
  }
  next();
};
```

### Використання middleware:

```javascript
const express = require('express');
const router = express.Router();
const { protect, adminOnly } = require('../middleware/auth');

// Публічний маршрут (без auth)
router.get('/public', (req, res) => {
  res.json({ message: 'Публічні дані' });
});

// Захищений маршрут (потрібна авторизація)
router.get('/profile', protect, (req, res) => {
  res.json({
    message: 'Ваш профіль',
    user: req.user
  });
});

// Тільки для адміна
router.get('/admin', protect, adminOnly, (req, res) => {
  res.json({ message: 'Адмін панель' });
});

module.exports = router;
```

---

## Тестування в Postman

### 1. Реєстрація

```
POST http://localhost:3000/api/auth/register
Content-Type: application/json

{
  "name": "Іван Петренко",
  "email": "ivan@example.com",
  "password": "SecurePass123"
}
```

**Відповідь:**
```json
{
  "message": "Реєстрація успішна",
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "654abc123",
    "name": "Іван Петренко",
    "email": "ivan@example.com",
    "role": "user"
  }
}
```

### 2. Логін

```
POST http://localhost:3000/api/auth/login
Content-Type: application/json

{
  "email": "ivan@example.com",
  "password": "SecurePass123"
}
```

**Відповідь:** (така ж як при реєстрації)

### 3. Доступ до захищеного маршруту

```
GET http://localhost:3000/api/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Як додати токен в Postman:**
1. Tabs → Authorization
2. Type → Bearer Token
3. Вставити токен

**Відповідь:**
```json
{
  "message": "Ваш профіль",
  "user": {
    "id": "654abc123",
    "name": "Іван Петренко",
    "email": "ivan@example.com",
    "role": "user"
  }
}
```

### 4. Без токену (помилка)

```
GET http://localhost:3000/api/profile
(без Authorization заголовку)
```

**Відповідь:**
```json
{
  "error": "Необхідна авторизація"
}
```

---

## Refresh Tokens (бонус)

### Проблема Access Token

**Access token** має короткий термін дії (15хв - 1год).

**Що якщо:**
- Користувач працює 2 години → токен протухає → треба логінитись знову 😤

### Рішення: Refresh Token

**Ідея:**
- **Access token:** короткий (15хв), для запитів
- **Refresh token:** довгий (7-30 днів), для оновлення access token

**Процес:**
```
1. Логін → отримуємо обидва токени
2. Робимо запити з access token
3. Access token протух → відправляємо refresh token
4. Отримуємо новий access token
5. Продовжуємо працювати
```

### Реалізація:

#### Оновлюємо модель User:

```javascript
const userSchema = new mongoose.Schema({
  // ... інші поля
  refreshToken: String
});
```

#### Оновлюємо utils/jwt.js:

```javascript
const jwt = require('jsonwebtoken');

exports.generateAccessToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_SECRET,
    { expiresIn: '15m' } // 15 хвилин
  );
};

exports.generateRefreshToken = (userId) => {
  return jwt.sign(
    { userId },
    process.env.JWT_REFRESH_SECRET,
    { expiresIn: '7d' } // 7 днів
  );
};

exports.verifyRefreshToken = (token) => {
  try {
    return jwt.verify(token, process.env.JWT_REFRESH_SECRET);
  } catch (error) {
    return null;
  }
};
```

#### Оновлюємо .env:

```env
JWT_SECRET=access-token-secret-min-32-chars
JWT_REFRESH_SECRET=refresh-token-secret-min-32-chars-different-from-access
```

#### Оновлюємо authController:

```javascript
const { generateAccessToken, generateRefreshToken } = require('../utils/jwt');

exports.login = async (req, res) => {
  try {
    // ... перевірка паролю
    
    // Генеруємо обидва токени
    const accessToken = generateAccessToken(user._id);
    const refreshToken = generateRefreshToken(user._id);
    
    // Зберігаємо refresh token в БД
    user.refreshToken = refreshToken;
    await user.save();
    
    res.json({
      message: 'Вхід успішний',
      accessToken,
      refreshToken,
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Оновити access token
exports.refresh = async (req, res) => {
  try {
    const { refreshToken } = req.body;
    
    if (!refreshToken) {
      return res.status(400).json({ error: 'Refresh token обов\'язковий' });
    }
    
    // Верифікувати refresh token
    const decoded = verifyRefreshToken(refreshToken);
    if (!decoded) {
      return res.status(401).json({ error: 'Недійсний refresh token' });
    }
    
    // Знайти користувача
    const user = await User.findById(decoded.userId);
    if (!user || user.refreshToken !== refreshToken) {
      return res.status(401).json({ error: 'Недійсний refresh token' });
    }
    
    // Створити новий access token
    const newAccessToken = generateAccessToken(user._id);
    
    res.json({
      accessToken: newAccessToken
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// Вийти (видалити refresh token)
exports.logout = async (req, res) => {
  try {
    const user = await User.findById(req.user.id);
    user.refreshToken = null;
    await user.save();
    
    res.json({ message: 'Вихід успішний' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

#### Додаємо маршрути:

```javascript
router.post('/refresh', authController.refresh);
router.post('/logout', protect, authController.logout);
```

---

## Де зберігати токени на frontend?

### Варіанти:

**1. localStorage**
```javascript
localStorage.setItem('token', token);
const token = localStorage.getItem('token');
```
- ✅ Просто
- ❌ Вразливий до XSS атак

**2. sessionStorage**
```javascript
sessionStorage.setItem('token', token);
```
- ✅ Автоматично очищається після закриття вкладки
- ❌ Вразливий до XSS атак

**3. httpOnly Cookie**
```javascript
// Backend
res.cookie('token', token, {
  httpOnly: true,  // Недоступний для JavaScript
  secure: true,    // Тільки HTTPS
  sameSite: 'strict'
});
```
- ✅ Захист від XSS
- ❌ Вразливий до CSRF атак (потрібен CSRF токен)

**Рекомендація:**
- **Access token:** httpOnly cookie або localStorage (якщо короткий термін)
- **Refresh token:** ТІЛЬКИ httpOnly cookie

---

## Практичне завдання 6

### Створіть повну систему авторизації з JWT:

1. **Реєстрація** - POST /api/auth/register
2. **Логін** - POST /api/auth/login
3. **Профіль** - GET /api/profile (захищений)
4. **Оновлення профілю** - PUT /api/profile (захищений)
5. **Адмін панель** - GET /api/admin (тільки admin)

### Бонус:

- Додати refresh token
- Endpoint для оновлення токену
- Logout endpoint

---

## Підсумок Частини 6

✅ **Що ми дізналися:**
- Що таке JWT та як він працює
- Структура JWT (header, payload, signature)
- Створення та перевірка токенів
- Захист маршрутів з middleware
- Access vs Refresh tokens
- Де зберігати токени

✅ **Що вміємо:**
- Створювати JWT токени
- Верифікувати токени
- Захищати маршрути
- Реалізувати повну авторизацію
- Використовувати refresh tokens

---

## 🎉 Вітаємо! Ви завершили курс Express.js!

### Що ми пройшли:

**Частина 1:** Основи Express, HTTP методи, роутинг
**Частина 2:** Middleware та конвеєр обробки
**Частина 3:** CORS та структура проекту
**Частина 4:** MongoDB та Mongoose
**Частина 5:** Bcrypt та безпека паролів
**Частина 6:** JWT авторизація

### Ви тепер вмієте:

- ✅ Створювати RESTful API
- ✅ Працювати з базою даних
- ✅ Реалізувати авторизацію
- ✅ Захищати маршрути
- ✅ Організовувати код
- ✅ Обробляти помилки

### Наступні кроки:

1. **Практика** - створіть власний проект (todo, blog, shop)
2. **Поглиблення** - вивчіть TypeScript, GraphQL, WebSockets
3. **Deployment** - задеплойте на Heroku, Vercel, Railway
4. **Тестування** - навчитесь писати тести (Jest, Supertest)

**Успіхів у розробці! 🚀**
