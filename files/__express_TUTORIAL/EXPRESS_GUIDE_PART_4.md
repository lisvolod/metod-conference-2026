# Express.js для початківців - Частина 4: MongoDB та Mongoose 🗄️

## Що таке MongoDB?

### Бази даних: SQL vs NoSQL

**SQL бази (MySQL, PostgreSQL):**
```
Таблиці з жорсткою структурою

Users Table:
┌────┬────────┬──────────────────┬─────┐
│ id │ name   │ email            │ age │
├────┼────────┼──────────────────┼─────┤
│ 1  │ Іван   │ ivan@example.com │ 25  │
│ 2  │ Марія  │ maria@example.com│ 30  │
└────┴────────┴──────────────────┴─────┘
```

**NoSQL бази (MongoDB):**
```
Колекції з гнучкими документами (JSON)

Users Collection:
{
  "_id": "507f1f77bcf86cd799439011",
  "name": "Іван",
  "email": "ivan@example.com",
  "age": 25,
  "hobbies": ["футбол", "програмування"]
}
```

### MongoDB = База даних + JSON

**Структура:**
```
MongoDB Server
  └── Database (база даних)
      ├── Collection (колекція = таблиця)
      │   ├── Document (документ = запис)
      │   ├── Document
      │   └── Document
      └── Collection
          ├── Document
          └── Document
```

---

## Встановлення MongoDB

### Варіант 1: MongoDB Atlas (Cloud, безкоштовно)

**Переваги:**
- Не треба встановлювати локально
- Безкоштовний план 512MB
- Доступ з будь-якого місця

**Кроки:**
1. Перейти на https://www.mongodb.com/cloud/atlas
2. Зареєструватись
3. Створити безкоштовний кластер (M0)
4. Отримати connection string

**Connection string виглядає так:**
```
mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/myDatabase?retryWrites=true&w=majority
```

### Варіант 2: MongoDB локально

**Windows:**
1. Завантажити з https://www.mongodb.com/try/download/community
2. Встановити MongoDB Community Server
3. Запустити MongoDB Compass (GUI інтерфейс)

**Linux/Mac:**
```bash
# Mac
brew install mongodb-community

# Ubuntu
sudo apt-get install mongodb
```

**Запуск:**
```bash
mongod
```

---

## Що таке Mongoose?

**Mongoose** - це ODM (Object Document Mapper) для MongoDB.

**Аналогія:**
- MongoDB = автомобіль
- Mongoose = автоматична коробка передач (спрощує керування)

**Mongoose надає:**
- Схеми (структура даних)
- Валідація
- Зручні методи для роботи з БД
- Middleware для БД операцій
- Зв'язки між колекціями

---

## Встановлення Mongoose

```bash
npm install mongoose
```

---

## Підключення до MongoDB

### Створюємо `config/db.js`

```javascript
const mongoose = require('mongoose');

const connectDB = async () => {
  try {
    // Connection string з .env файлу
    const conn = await mongoose.connect(process.env.MONGO_URI);
    
    console.log(`✅ MongoDB підключено: ${conn.connection.host}`);
  } catch (error) {
    console.error('❌ Помилка підключення до MongoDB:', error.message);
    process.exit(1); // Зупинити сервер якщо немає з'єднання
  }
};

module.exports = connectDB;
```

### Оновлюємо `.env`

```env
PORT=3000
NODE_ENV=development

# MongoDB Atlas
MONGO_URI=mongodb+srv://username:password@cluster0.xxxxx.mongodb.net/myDatabase

# Або локальний MongoDB
# MONGO_URI=mongodb://localhost:27017/myDatabase
```

### Підключаємо в `server.js`

```javascript
require('dotenv').config();
const express = require('express');
const connectDB = require('./config/db');

const app = express();

// Підключення до MongoDB
connectDB();

// Middleware
app.use(express.json());

// Routes
app.get('/', (req, res) => {
  res.send('API працює');
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Сервер на порту ${PORT}`);
});
```

**Вивід в консолі:**
```
✅ MongoDB підключено: cluster0-shard-00-00.xxxxx.mongodb.net
🚀 Сервер на порту 3000
```

---

## Mongoose Schema (Схема)

### Що таке Schema?

**Schema** - це "креслення" структури документа.

**Аналогія:** Як форма для випічки - визначає форму майбутнього документа.

### Створюємо схему користувача

#### `models/User.js`

```javascript
const mongoose = require('mongoose');

// Визначаємо схему
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'Ім\'я обов\'язкове'],
    trim: true, // Видаляє пробіли на початку/кінці
    minlength: [2, 'Ім\'я має містити мінімум 2 символи'],
    maxlength: [50, 'Ім\'я має містити максимум 50 символів']
  },
  email: {
    type: String,
    required: [true, 'Email обов\'язковий'],
    unique: true, // Унікальний email
    lowercase: true, // Перетворює в нижній регістр
    trim: true,
    match: [/^\S+@\S+\.\S+$/, 'Невірний формат email']
  },
  password: {
    type: String,
    required: [true, 'Пароль обов\'язковий'],
    minlength: [6, 'Пароль має містити мінімум 6 символів']
  },
  age: {
    type: Number,
    min: [18, 'Вік має бути мінімум 18'],
    max: [120, 'Невірний вік']
  },
  role: {
    type: String,
    enum: ['user', 'admin'], // Тільки ці значення
    default: 'user'
  },
  isActive: {
    type: Boolean,
    default: true
  }
}, {
  timestamps: true // Автоматично додає createdAt та updatedAt
});

// Створюємо модель
const User = mongoose.model('User', userSchema);

module.exports = User;
```

### Типи даних в Mongoose

```javascript
{
  // Рядок
  name: String,
  
  // Число
  age: Number,
  
  // Boolean
  isActive: Boolean,
  
  // Дата
  birthDate: Date,
  
  // Масив
  hobbies: [String],
  tags: Array,
  
  // Вкладений об'єкт
  address: {
    street: String,
    city: String,
    zipCode: String
  },
  
  // ObjectId (посилання на інший документ)
  userId: mongoose.Schema.Types.ObjectId,
  
  // Mixed (будь-який тип)
  metadata: mongoose.Schema.Types.Mixed
}
```

---

## CRUD операції з Mongoose

### CREATE - Створення документа

```javascript
const User = require('./models/User');

// Варіант 1: save()
const createUser1 = async () => {
  const user = new User({
    name: 'Іван Петренко',
    email: 'ivan@example.com',
    password: 'password123',
    age: 25
  });
  
  await user.save();
  console.log('Користувача створено:', user);
};

// Варіант 2: create()
const createUser2 = async () => {
  const user = await User.create({
    name: 'Марія Іваненко',
    email: 'maria@example.com',
    password: 'password123',
    age: 30
  });
  
  console.log('Користувача створено:', user);
};

// Варіант 3: insertMany() - для багатьох документів
const createMany = async () => {
  const users = await User.insertMany([
    { name: 'User 1', email: 'user1@example.com', password: '123456' },
    { name: 'User 2', email: 'user2@example.com', password: '123456' },
    { name: 'User 3', email: 'user3@example.com', password: '123456' }
  ]);
  
  console.log('Створено користувачів:', users.length);
};
```

### READ - Читання документів

```javascript
// Знайти всіх користувачів
const getAllUsers = async () => {
  const users = await User.find();
  console.log('Всі користувачі:', users);
};

// Знайти з умовою
const findByAge = async () => {
  const users = await User.find({ age: { $gte: 25 } }); // >= 25
  console.log('Користувачі 25+:', users);
};

// Знайти одного
const findOne = async () => {
  const user = await User.findOne({ email: 'ivan@example.com' });
  console.log('Знайдено:', user);
};

// Знайти по ID
const findById = async (id) => {
  const user = await User.findById(id);
  console.log('Користувач:', user);
};

// З вибірковими полями
const findWithSelect = async () => {
  const users = await User.find().select('name email -_id'); // Тільки name та email
  console.log(users);
};

// З сортуванням
const findWithSort = async () => {
  const users = await User.find().sort({ age: -1 }); // -1 = DESC, 1 = ASC
  console.log(users);
};

// З лімітом та пропуском (пагінація)
const findWithPagination = async () => {
  const page = 2;
  const limit = 10;
  const skip = (page - 1) * limit;
  
  const users = await User.find()
    .limit(limit)
    .skip(skip)
    .sort({ createdAt: -1 });
  
  console.log(users);
};
```

### UPDATE - Оновлення документів

```javascript
// Оновити один документ
const updateUser = async (id) => {
  const user = await User.findByIdAndUpdate(
    id,
    { age: 26, name: 'Іван Оновлений' },
    { new: true } // Повернути оновлений документ
  );
  
  console.log('Оновлено:', user);
};

// Оновити багато документів
const updateMany = async () => {
  const result = await User.updateMany(
    { age: { $lt: 25 } }, // Умова: вік < 25
    { isActive: false }    // Встановити isActive = false
  );
  
  console.log('Оновлено документів:', result.modifiedCount);
};

// Знайти та оновити
const findAndUpdate = async () => {
  const user = await User.findOneAndUpdate(
    { email: 'ivan@example.com' },
    { $inc: { age: 1 } }, // Збільшити age на 1
    { new: true }
  );
  
  console.log(user);
};
```

### DELETE - Видалення документів

```javascript
// Видалити один документ
const deleteUser = async (id) => {
  const user = await User.findByIdAndDelete(id);
  console.log('Видалено:', user);
};

// Видалити багато документів
const deleteMany = async () => {
  const result = await User.deleteMany({ isActive: false });
  console.log('Видалено документів:', result.deletedCount);
};

// Знайти та видалити
const findAndDelete = async () => {
  const user = await User.findOneAndDelete({ email: 'test@example.com' });
  console.log('Видалено:', user);
};
```

---

## Query операції (фільтрація)

### Оператори порівняння

```javascript
// $eq - дорівнює
User.find({ age: { $eq: 25 } });
// Або просто
User.find({ age: 25 });

// $ne - не дорівнює
User.find({ role: { $ne: 'admin' } });

// $gt - більше
User.find({ age: { $gt: 25 } });

// $gte - більше або дорівнює
User.find({ age: { $gte: 25 } });

// $lt - менше
User.find({ age: { $lt: 30 } });

// $lte - менше або дорівнює
User.find({ age: { $lte: 30 } });

// $in - в масиві значень
User.find({ role: { $in: ['admin', 'moderator'] } });

// $nin - не в масиві значень
User.find({ role: { $nin: ['banned'] } });
```

### Логічні оператори

```javascript
// $and - І (всі умови виконуються)
User.find({
  $and: [
    { age: { $gte: 18 } },
    { age: { $lte: 65 } },
    { isActive: true }
  ]
});

// $or - АБО (хоча б одна умова)
User.find({
  $or: [
    { role: 'admin' },
    { role: 'moderator' }
  ]
});

// $nor - НЕ АБО (жодна умова не виконується)
User.find({
  $nor: [
    { isActive: false },
    { role: 'banned' }
  ]
});

// $not - НЕ
User.find({
  age: { $not: { $lt: 18 } }
});
```

### Regex (регулярні вирази)

```javascript
// Пошук по частині тексту
User.find({ name: /іван/i }); // i = case insensitive

// Починається з
User.find({ name: /^Іван/ });

// Закінчується на
User.find({ email: /@gmail\.com$/ });

// Містить
User.find({ name: { $regex: 'петренко', $options: 'i' } });
```

---

## Інтеграція з Express

### Створюємо REST API для користувачів

#### `controllers/userController.js`

```javascript
const User = require('../models/User');

// GET /api/users - Отримати всіх користувачів
exports.getAllUsers = async (req, res) => {
  try {
    const users = await User.find().select('-password'); // Без паролів
    res.json(users);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// GET /api/users/:id - Отримати одного користувача
exports.getUserById = async (req, res) => {
  try {
    const user = await User.findById(req.params.id).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'Користувача не знайдено' });
    }
    
    res.json(user);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};

// POST /api/users - Створити користувача
exports.createUser = async (req, res) => {
  try {
    const { name, email, password, age } = req.body;
    
    // Перевірити чи email вже існує
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email вже використовується' });
    }
    
    const user = await User.create({ name, email, password, age });
    
    // Не повертаємо пароль
    const userResponse = user.toObject();
    delete userResponse.password;
    
    res.status(201).json(userResponse);
  } catch (error) {
    // Mongoose validation errors
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(err => err.message);
      return res.status(400).json({ error: errors });
    }
    
    res.status(500).json({ error: error.message });
  }
};

// PUT /api/users/:id - Оновити користувача
exports.updateUser = async (req, res) => {
  try {
    const { name, email, age } = req.body;
    
    const user = await User.findByIdAndUpdate(
      req.params.id,
      { name, email, age },
      { new: true, runValidators: true }
    ).select('-password');
    
    if (!user) {
      return res.status(404).json({ error: 'Користувача не знайдено' });
    }
    
    res.json(user);
  } catch (error) {
    if (error.name === 'ValidationError') {
      const errors = Object.values(error.errors).map(err => err.message);
      return res.status(400).json({ error: errors });
    }
    
    res.status(500).json({ error: error.message });
  }
};

// DELETE /api/users/:id - Видалити користувача
exports.deleteUser = async (req, res) => {
  try {
    const user = await User.findByIdAndDelete(req.params.id);
    
    if (!user) {
      return res.status(404).json({ error: 'Користувача не знайдено' });
    }
    
    res.json({ message: 'Користувача видалено' });
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
};
```

#### `routes/users.js`

```javascript
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

module.exports = router;
```

#### `server.js` (оновлений)

```javascript
require('dotenv').config();
const express = require('express');
const cors = require('cors');
const connectDB = require('./config/db');

const app = express();

// Підключення до MongoDB
connectDB();

// Middleware
app.use(cors());
app.use(express.json());

// Routes
const userRoutes = require('./routes/users');
app.use('/api/users', userRoutes);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`🚀 Сервер на порту ${PORT}`);
});
```

---

## Практичне завдання 4

### Створіть Product модель та API

#### Схема продукту:

```javascript
{
  name: String (required, 2-100 символів),
  description: String (required),
  price: Number (required, мінімум 0),
  stock: Number (default: 0),
  category: String (enum: ['electronics', 'clothing', 'food']),
  isAvailable: Boolean (default: true),
  images: [String],
  createdAt: Date,
  updatedAt: Date
}
```

#### API endpoints:

```
GET    /api/products           - Всі продукти
GET    /api/products/:id       - Один продукт
POST   /api/products           - Створити продукт
PUT    /api/products/:id       - Оновити продукт
DELETE /api/products/:id       - Видалити продукт
GET    /api/products?category=electronics - Фільтр по категорії
```

---

## Підсумок Частини 4

✅ **Що ми дізналися:**
- Що таке MongoDB та NoSQL
- Mongoose для роботи з MongoDB
- Створення схем та моделей
- CRUD операції з базою даних
- Query операції та фільтрація
- Інтеграція Mongoose з Express

✅ **Що вміємо:**
- Підключатися до MongoDB
- Створювати схеми з валідацією
- Виконувати CRUD операції
- Фільтрувати та сортувати дані
- Створювати REST API з MongoDB

📚 **Наступна частина:**
- Bcrypt для хешування паролів
- Безпечне зберігання паролів
- Реєстрація та логін користувачів

**Вперед до Частини 5! 🚀**
