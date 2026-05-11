# Express.js для початківців - Частина 5: Bcrypt та безпека паролів 🔐

## Чому не можна зберігати паролі як текст?

### Проблема:

```javascript
// ❌ ЖАХЛИВА ПРАКТИКА - НІКОЛИ ТАК НЕ РОБІТЬ!
const user = {
  email: 'ivan@example.com',
  password: 'mySecretPassword123' // Зберігається як текст!
};

await User.create(user);
```

**Що станеться якщо хакер отримає доступ до бази даних?**
```
Database dump:
- ivan@example.com : mySecretPassword123
- maria@example.com : qwerty12345
- admin@example.com : admin2024
```

Хакер отримує **всі паролі** одразу! 😱

### Додаткова проблема:

Більшість людей використовують **той самий пароль** для різних сайтів:
- Gmail: mySecretPassword123
- Facebook: mySecretPassword123
- Bank: mySecretPassword123

Якщо хакер дізнається пароль з одного сайту → доступ до всіх акаунтів! 💀

---

## Що таке хешування?

### Концепція

**Хешування** - це перетворення паролю в незворотний "хеш" (нечитабельний рядок).

**Процес:**
```
Пароль (input)  →  Hash функція  →  Hash (output)
"myPassword123"  →   bcrypt        →  "$2b$10$Ke8xZ..."
```

**Властивості хешування:**
1. **Односторонній процес** - неможливо отримати пароль з хешу
2. **Детермінований** - той самий пароль завжди дає той самий хеш
3. **Унікальний** - різні паролі дають різні хеші

### Приклад:

```javascript
// Оригінальні паролі (ми НЕ зберігаємо їх!)
"password123"  → хеш → "$2b$10$Ke8xZ7vL..."
"password124"  → хеш → "$2b$10$Df5tR8mN..."  // Зовсім інший!
"password123"  → хеш → "$2b$10$Ke8xZ7vL..."  // Той самий що перший
```

### База даних з хешами:

```
Зберігається в БД:
┌─────────────────────┬────────────────────────────────┐
│ Email               │ Password (hashed)              │
├─────────────────────┼────────────────────────────────┤
│ ivan@example.com    │ $2b$10$Ke8xZ7vL...            │
│ maria@example.com   │ $2b$10$Df5tR8mN...            │
└─────────────────────┴────────────────────────────────┘
```

**Навіть якщо хакер отримає доступ до БД, він НЕ МОЖЕ дізнатись паролі!** ✅

---

## Що таке Bcrypt?

**Bcrypt** - це алгоритм хешування паролів, спеціально розроблений для безпеки.

**Особливості:**
1. **Повільний** (спеціально!) - захист від brute-force атак
2. **Salt** - додає випадковість до хешу
3. **Rounds** - можна регулювати складність

### Salt (сіль)

**Salt** - це випадковий рядок який додається до паролю перед хешуванням.

```
Без salt:
"password123" → хеш → завжди однаковий хеш

З salt:
"password123" + "randomSalt1" → хеш1
"password123" + "randomSalt2" → хеш2  // Різні хеші!
```

**Навіщо?** Якщо два користувачі мають однаковий пароль, їхні хеші будуть **різними**!

---

## Встановлення Bcrypt

```bash
npm install bcryptjs
```

**Чому bcryptjs а не bcrypt?**
- `bcryptjs` - написаний на JavaScript, працює скрізь
- `bcrypt` - швидший але потребує компіляції (можуть бути проблеми на Windows)

Для навчання краще `bcryptjs`!

---

## Базове використання Bcrypt

### Хешування паролю

```javascript
const bcrypt = require('bcryptjs');

async function hashPassword() {
  const password = 'mySecretPassword123';
  
  // Хешуємо пароль
  const salt = await bcrypt.genSalt(10); // Генеруємо salt
  const hashedPassword = await bcrypt.hash(password, salt);
  
  console.log('Оригінальний пароль:', password);
  console.log('Хешований пароль:', hashedPassword);
}

hashPassword();
```

**Вивід:**
```
Оригінальний пароль: mySecretPassword123
Хешований пароль: $2a$10$Ke8xZ7vLn3pN5YwP8EGJxeZ.vH1CjR9wW8tK5mL2nO3pQ4rS5tU6v
```

### Що означають частини хешу?

```
$2a$10$Ke8xZ7vLn3pN5YwP8EGJxeZ.vH1CjR9wW8tK5mL2nO3pQ4rS5tU6v
 │   │  │                            └─ Хеш
 │   │  └─ Salt
 │   └─ Rounds (складність)
 └─ Алгоритм (bcrypt)
```

### Скорочений запис (рекомендований):

```javascript
const bcrypt = require('bcryptjs');

async function hashPassword(password) {
  // Одразу з salt (10 rounds)
  const hashedPassword = await bcrypt.hash(password, 10);
  return hashedPassword;
}

const hash = await hashPassword('myPassword123');
console.log(hash);
```

---

## Порівняння паролів

### Як перевірити чи пароль правильний?

```javascript
const bcrypt = require('bcryptjs');

async function checkPassword() {
  const password = 'myPassword123';
  const hashedPassword = '$2a$10$Ke8xZ7vL...'; // З бази даних
  
  // Порівнюємо
  const isMatch = await bcrypt.compare(password, hashedPassword);
  
  console.log('Пароль співпадає:', isMatch); // true або false
}

checkPassword();
```

**Як це працює?**
1. Bcrypt бере salt з хешу
2. Хешує введений пароль з цим salt
3. Порівнює отриманий хеш з збереженим хешем
4. Якщо однакові → пароль правильний ✅

---

## Інтеграція з Mongoose

### Метод 1: Хешування в контролері

```javascript
// controllers/authController.js
const bcrypt = require('bcryptjs');
const User = require('../models/User');

exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // 1. Перевірити чи користувач вже існує
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email вже використовується' });
    }
    
    // 2. Хешувати пароль
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // 3. Створити користувача
    const user = await User.create({
      name,
      email,
      password: hashedPassword // Зберігаємо хеш, не оригінальний пароль!
    });
    
    // 4. Відправити відповідь (БЕЗ паролю!)
    const userResponse = user.toObject();
    delete userResponse.password;
    
    res.status(201).json({
      message: 'Користувача створено',
      user: userResponse
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // 1. Знайти користувача
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // 2. Перевірити пароль
    const isPasswordCorrect = await bcrypt.compare(password, user.password);
    if (!isPasswordCorrect) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // 3. Пароль правильний - користувач авторизований!
    res.json({
      message: 'Успішний вхід',
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
```

### Метод 2: Автоматичне хешування в моделі (кращий підхід!)

#### `models/User.js`

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true
  },
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true
  },
  password: {
    type: String,
    required: true,
    minlength: 6
  }
}, {
  timestamps: true
});

// Middleware: Хешувати пароль перед збереженням
userSchema.pre('save', async function(next) {
  // Хешувати тільки якщо пароль змінився
  if (!this.isModified('password')) {
    return next();
  }
  
  // Хешуємо пароль
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Метод для перевірки паролю
userSchema.methods.comparePassword = async function(candidatePassword) {
  return await bcrypt.compare(candidatePassword, this.password);
};

const User = mongoose.model('User', userSchema);

module.exports = User;
```

#### Оновлений контролер:

```javascript
const User = require('../models/User');

exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Перевірка чи існує користувач
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email вже використовується' });
    }
    
    // Створюємо користувача (пароль автоматично хешується!)
    const user = await User.create({ name, email, password });
    
    // Відповідь без паролю
    res.status(201).json({
      message: 'Користувача створено',
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

exports.login = async (req, res) => {
  try {
    const { email, password } = req.body;
    
    // Знайти користувача
    const user = await User.findOne({ email });
    if (!user) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // Перевірити пароль (використовуємо метод моделі!)
    const isPasswordCorrect = await user.comparePassword(password);
    if (!isPasswordCorrect) {
      return res.status(401).json({ error: 'Невірний email або пароль' });
    }
    
    // Успішний вхід
    res.json({
      message: 'Успішний вхід',
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
```

---

## Rounds (складність хешування)

### Що таке rounds?

**Rounds** - це кількість ітерацій хешування. Більше rounds = повільніше але безпечніше.

```javascript
const rounds = 10;
const hash = await bcrypt.hash('password', rounds);
```

### Порівняння швидкості:

```
Rounds | Час хешування | Безпека
-------|---------------|----------
  4    |   ~5ms        | Низька
  6    |   ~20ms       | Низька
  8    |   ~80ms       | Середня
 10    |   ~300ms      | Висока ✅
 12    |   ~1200ms     | Дуже висока
 14    |   ~5000ms     | Надмірна
```

**Рекомендації:**
- **10 rounds** - оптимальний баланс (рекомендовано)
- **12 rounds** - для високої безпеки
- **< 8 rounds** - небезпечно
- **> 12 rounds** - повільно для користувача

### Чому повільність = безпека?

**Brute-force атака** - перебір всіх можливих паролів:

```
10 rounds → 300ms на спробу
1 мільйон спроб = 300,000 секунд = 83 години

4 rounds → 5ms на спробу
1 мільйон спроб = 5,000 секунд = 1.4 години
```

Чим повільніше хешування, тим довше хакер перебиратиме паролі!

---

## Валідація паролів

### Мінімальні вимоги:

```javascript
function validatePassword(password) {
  const errors = [];
  
  // Мінімум 8 символів
  if (password.length < 8) {
    errors.push('Пароль має містити мінімум 8 символів');
  }
  
  // Має містити цифру
  if (!/\d/.test(password)) {
    errors.push('Пароль має містити хоча б одну цифру');
  }
  
  // Має містити велику літеру
  if (!/[A-Z]/.test(password)) {
    errors.push('Пароль має містити хоча б одну велику літеру');
  }
  
  // Має містити малу літеру
  if (!/[a-z]/.test(password)) {
    errors.push('Пароль має містити хоча б одну малу літеру');
  }
  
  // Має містити спецсимвол
  if (!/[!@#$%^&*]/.test(password)) {
    errors.push('Пароль має містити спецсимвол (!@#$%^&*)');
  }
  
  return {
    isValid: errors.length === 0,
    errors
  };
}

// Використання в контролері
exports.register = async (req, res) => {
  const { password } = req.body;
  
  const validation = validatePassword(password);
  if (!validation.isValid) {
    return res.status(400).json({ errors: validation.errors });
  }
  
  // Продовжуємо реєстрацію...
};
```

---

## Повний прикладAuth API

### Структура файлів:

```
project/
├── models/
│   └── User.js
├── controllers/
│   └── authController.js
├── routes/
│   └── auth.js
├── middleware/
│   └── validate.js
├── .env
└── server.js
```

### `models/User.js`

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Ім\'я обов\'язкове'],
    trim: true
  },
  email: {
    type: String,
    required: [true, 'Email обов\'язковий'],
    unique: true,
    lowercase: true,
    match: [/^\S+@\S+\.\S+$/, 'Невірний формат email']
  },
  password: {
    type: String,
    required: [true, 'Пароль обов\'язковий'],
    minlength: [6, 'Пароль має містити мінімум 6 символів']
  },
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  }
}, {
  timestamps: true
});

// Хешування паролю перед збереженням
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next();
  this.password = await bcrypt.hash(this.password, 10);
  next();
});

// Метод для перевірки паролю
userSchema.methods.comparePassword = async function(password) {
  return await bcrypt.compare(password, this.password);
};

// Не повертати пароль в JSON
userSchema.methods.toJSON = function() {
  const obj = this.toObject();
  delete obj.password;
  return obj;
};

module.exports = mongoose.model('User', userSchema);
```

### `controllers/authController.js`

```javascript
const User = require('../models/User');

exports.register = async (req, res) => {
  try {
    const { name, email, password } = req.body;
    
    // Перевірка чи користувач існує
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email вже використовується' });
    }
    
    // Створення користувача
    const user = await User.create({ name, email, password });
    
    res.status(201).json({
      message: 'Реєстрація успішна',
      user
    });
  } catch (error) {
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(e => e.message);
      return res.status(400).json({ error: errors });
    }
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
    
    res.json({
      message: 'Вхід успішний',
      user
    });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

### `routes/auth.js`

```javascript
const express = require('express');
const router = express.Router();
const authController = require('../controllers/authController');

router.post('/register', authController.register);
router.post('/login', authController.login);

module.exports = router;
```

### Тестування в Postman:

**1. Реєстрація:**
```
POST http://localhost:3000/api/auth/register
Content-Type: application/json

{
  "name": "Іван Петренко",
  "email": "ivan@example.com",
  "password": "SecurePass123"
}
```

**2. Логін:**
```
POST http://localhost:3000/api/auth/login
Content-Type: application/json

{
  "email": "ivan@example.com",
  "password": "SecurePass123"
}
```

---

## Практичне завдання 5

### Створіть систему авторизації:

1. **User модель** з хешуванням паролів
2. **Register endpoint** - POST /api/auth/register
3. **Login endpoint** - POST /api/auth/login
4. **Валідація паролів** (мінімум 8 символів, містить цифру)
5. **Тестування** через Postman

### Додаткові завдання:

- Додати поле `confirmPassword` при реєстрації
- Обмежити кількість спроб входу (rate limiting)
- Додати endpoint для зміни паролю

---

## Підсумок Частини 5

✅ **Що ми дізналися:**
- Чому не можна зберігати паролі як текст
- Що таке хешування та salt
- Як працює bcrypt
- Rounds та складність хешування
- Інтеграція bcrypt з Mongoose

✅ **Що вміємо:**
- Хешувати паролі
- Порівнювати паролі
- Створювати безпечну реєстрацію
- Реалізувати логін з перевіркою паролю

📚 **Наступна частина:**
- JWT (JSON Web Tokens)
- Створення та перевірка токенів
- Захищені маршрути
- Refresh tokens

**Вперед до Частини 6! 🚀**
