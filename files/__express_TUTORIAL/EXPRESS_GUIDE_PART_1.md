# Express.js для початківців - Частина 1: Вступ та перший сервер 🚀

## Що таке Node.js та Express?

### Node.js
**Node.js** - це середовище виконання JavaScript на сервері (а не в браузері).

**Аналогія:**
- Браузер = кухня для приготування їжі (frontend)
- Node.js = ресторанна кухня з професійним обладнанням (backend)

**Що можна робити:**
- Створювати веб-сервери
- Працювати з файлами
- Підключатися до баз даних
- Обробляти запити від користувачів

### Express.js
**Express** - це фреймворк (набір інструментів) для створення веб-серверів на Node.js.

**Аналогія:**
- Node.js = кухня з плитою та холодильником
- Express = готові рецепти та інструкції як готувати

**Express спрощує:**
- Обробку HTTP запитів (GET, POST, PUT, DELETE)
- Роутинг (маршрутизація URL)
- Роботу з middleware (проміжні обробники)
- Відправку відповідей клієнту

---

## Встановлення Node.js

### 1. Завантажити Node.js
Перейти на https://nodejs.org/ та завантажити **LTS версію** (Long Term Support)

### 2. Перевірити встановлення
```bash
node --version
# Має показати: v20.x.x або новіше

npm --version
# Має показати: 10.x.x або новіше
```

**Що таке npm?**
npm (Node Package Manager) - це менеджер пакетів, як "магазин додатків" для Node.js.

---

## Створення першого Express проекту

### Крок 1: Створити папку проекту
```bash
mkdir my-first-server
cd my-first-server
```

### Крок 2: Ініціалізувати npm проект
```bash
npm init -y
```

**Що відбулося?**
Створився файл `package.json` - це "паспорт" вашого проекту.

```json
{
  "name": "my-first-server",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC"
}
```

### Крок 3: Встановити Express
```bash
npm install express
```

**Що відбулося?**
1. Створилась папка `node_modules` - тут зберігаються всі бібліотеки
2. Створився файл `package-lock.json` - точний список версій
3. В `package.json` додалась залежність:

```json
"dependencies": {
  "express": "^4.18.2"
}
```

---

## Перший Express сервер

### Створити файл `server.js`

```javascript
// 1. Імпортуємо Express
const express = require('express');

// 2. Створюємо додаток Express
const app = express();

// 3. Визначаємо порт (адресу) для сервера
const PORT = 3000;

// 4. Створюємо перший маршрут (route)
app.get('/', (req, res) => {
  res.send('Привіт! Це мій перший сервер! 🚀');
});

// 5. Запускаємо сервер
app.listen(PORT, () => {
  console.log(`Сервер запущено на http://localhost:${PORT}`);
});
```

### Пояснення коду:

#### `const express = require('express');`
- Завантажуємо бібліотеку Express
- `require()` - це спосіб імпорту в Node.js

#### `const app = express();`
- Створюємо наш веб-додаток
- `app` - це об'єкт з методами для роботи з сервером

#### `const PORT = 3000;`
- Визначаємо на якому порту слухатиме сервер
- Порт - це як "канал" на телевізорі
- `localhost:3000` = ваш комп'ютер, канал 3000

#### `app.get('/', (req, res) => {...})`
Це **маршрут (route)** - правило що робити коли хтось відкриває URL.

**Розбір по частинах:**
- `app.get` - метод HTTP GET (отримати дані)
- `'/'` - шлях URL (головна сторінка)
- `(req, res) => {}` - функція-обробник

**Параметри функції:**
- `req` (request) - запит від клієнта (що він хоче?)
- `res` (response) - відповідь сервера (що ми йому даємо?)

#### `res.send('...')`
- Відправити відповідь клієнту
- Можна відправляти текст, HTML, JSON

#### `app.listen(PORT, callback)`
- Запустити сервер на порту `PORT`
- `callback` - функція яка виконається після запуску

---

## Запуск сервера

### Метод 1: Звичайний запуск
```bash
node server.js
```

**Що відбувається:**
1. Node.js читає файл `server.js`
2. Виконує код
3. Сервер запускається і чекає на запити

**Відкрити в браузері:**
```
http://localhost:3000
```

Має показатись: `Привіт! Це мій перший сервер! 🚀`

**Як зупинити сервер:**
Натиснути `Ctrl + C` в терміналі

---

## Додаємо більше маршрутів

### Оновлений `server.js`:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

// Головна сторінка
app.get('/', (req, res) => {
  res.send('Головна сторінка');
});

// Сторінка "Про нас"
app.get('/about', (req, res) => {
  res.send('Це сторінка про нас');
});

// Сторінка контактів
app.get('/contact', (req, res) => {
  res.send('Контакти: email@example.com');
});

// API endpoint (повертає JSON)
app.get('/api/user', (req, res) => {
  res.json({
    name: 'Іван',
    age: 25,
    city: 'Київ'
  });
});

app.listen(PORT, () => {
  console.log(`Сервер на http://localhost:${PORT}`);
});
```

**Тепер можна відкрити:**
- `http://localhost:3000/` - Головна
- `http://localhost:3000/about` - Про нас
- `http://localhost:3000/contact` - Контакти
- `http://localhost:3000/api/user` - JSON відповідь

---

## Автоматичний перезапуск з Nodemon

### Проблема:
Після зміни коду треба зупиняти та запускати сервер заново. Незручно! 😤

### Рішення: Nodemon
**Nodemon** - це інструмент який автоматично перезапускає сервер при зміні файлів.

### Встановлення:
```bash
npm install --save-dev nodemon
```

`--save-dev` означає що це залежність тільки для розробки (не потрібна в production).

### Додати скрипти в `package.json`:
```json
{
  "name": "my-first-server",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "dev": "nodemon server.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  },
  "devDependencies": {
    "nodemon": "^3.0.2"
  }
}
```

### Запуск з nodemon:
```bash
npm run dev
```

**Тепер:**
1. Змініть щось у `server.js`
2. Збережіть файл
3. Сервер автоматично перезапуститься! ✅

---

## HTTP методи: GET, POST, PUT, DELETE

### Що це?
HTTP методи - це "типи" запитів до сервера.

**Аналогія з бібліотекою:**
- **GET** - "Дайте мені подивитись книгу" (читання)
- **POST** - "Хочу додати нову книгу" (створення)
- **PUT** - "Оновіть інформацію про книгу" (оновлення)
- **DELETE** - "Видаліть цю книгу" (видалення)

### Приклад всіх методів:

```javascript
const express = require('express');
const app = express();

// GET - отримати дані
app.get('/books', (req, res) => {
  res.json({ message: 'Список всіх книг' });
});

// POST - створити нові дані
app.post('/books', (req, res) => {
  res.json({ message: 'Книгу створено' });
});

// PUT - оновити дані
app.put('/books/1', (req, res) => {
  res.json({ message: 'Книгу оновлено' });
});

// DELETE - видалити дані
app.delete('/books/1', (req, res) => {
  res.json({ message: 'Книгу видалено' });
});

app.listen(3000);
```

**Як тестувати не-GET запити?**
Використовуйте:
- **Postman** (https://www.postman.com/)
- **Thunder Client** (розширення VS Code)
- **curl** (команда в терміналі)

---

## Параметри URL (Route Parameters)

### Динамічні маршрути

```javascript
// :id - це параметр (може бути будь-яким значенням)
app.get('/users/:id', (req, res) => {
  const userId = req.params.id;
  res.send(`Ви шукаєте користувача з ID: ${userId}`);
});

// Багато параметрів
app.get('/posts/:year/:month/:day', (req, res) => {
  const { year, month, day } = req.params;
  res.send(`Пости за дату: ${day}.${month}.${year}`);
});
```

**Приклади URL:**
- `/users/123` → userId = "123"
- `/users/abc` → userId = "abc"
- `/posts/2024/12/25` → year="2024", month="12", day="25"

---

## Query параметри

### Що це?
Параметри після `?` в URL.

**Приклад URL:**
```
http://localhost:3000/search?q=express&page=1&limit=10
```

### Як отримати в Express:

```javascript
app.get('/search', (req, res) => {
  const query = req.query.q;      // "express"
  const page = req.query.page;    // "1"
  const limit = req.query.limit;  // "10"
  
  res.json({
    query: query,
    page: page,
    limit: limit
  });
});
```

**Різниця між params та query:**
- **params** (`/users/:id`) - частина шляху, обов'язкові
- **query** (`?page=1`) - після `?`, опціональні

---

## Практичне завдання 1

### Створіть сервер з такими маршрутами:

1. `GET /` - Привітання
2. `GET /about` - Інформація про сайт
3. `GET /products` - Список продуктів (масив об'єктів JSON)
4. `GET /products/:id` - Один продукт по ID
5. `GET /search?q=...` - Пошук продуктів

**Приклад відповіді для `/products`:**
```json
[
  { "id": 1, "name": "Ноутбук", "price": 20000 },
  { "id": 2, "name": "Мишка", "price": 500 },
  { "id": 3, "name": "Клавіатура", "price": 1500 }
]
```

---

## Підсумок Частини 1

✅ **Що ми дізналися:**
- Що таке Node.js та Express
- Як створити базовий Express сервер
- Як створювати маршрути (routes)
- HTTP методи (GET, POST, PUT, DELETE)
- Параметри URL (params та query)
- Як використовувати nodemon

✅ **Що вміємо:**
- Запускати Express сервер
- Обробляти різні URL
- Повертати JSON відповіді
- Працювати з динамічними маршрутами

📚 **Наступна частина:**
- Middleware (проміжні обробники)
- Body parsing (обробка даних форм)
- CORS (доступ з інших доменів)
- Структура проекту

---

**Питання для самоперевірки:**

1. Що таке Express та навіщо він потрібен?
2. Яка різниця між `req.params` та `req.query`?
3. Як запустити сервер на порту 5000?
4. Навіщо потрібен nodemon?
5. Яка різниця між GET та POST запитами?

**Вперед до Частини 2! 🚀**
