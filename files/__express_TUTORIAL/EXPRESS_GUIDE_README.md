# Повний курс Express.js для початківців 🚀

## 📚 Про курс

Це покроковий навчальний посібник з Express.js, Node.js та backend розробки. Курс розроблений для студентів які не мають досвіду з Node.js та хочуть навчитись створювати повноцінні RESTful API з авторизацією та базою даних.

---

## 🎯 Що ви навчитесь

- Створювати веб-сервери на Node.js та Express
- Працювати з middleware та конвеєром обробки
- Налаштовувати CORS для frontend-backend комунікації
- Підключатися до MongoDB та працювати з Mongoose
- Безпечно зберігати паролі з Bcrypt
- Реалізувати JWT авторизацію
- Структурувати код (MVC архітектура)
- Створювати захищені API endpoints

---

## 📖 Структура курсу

### [Частина 1: Вступ та перший сервер](EXPRESS_GUIDE_PART_1.md)
**Теми:**
- Що таке Node.js та Express
- Встановлення та налаштування
- Створення першого сервера
- HTTP методи (GET, POST, PUT, DELETE)
- Параметри URL (params та query)
- Nodemon для автоматичного перезапуску

**Що вміємо після цієї частини:**
- Запускати Express сервер
- Створювати маршрути
- Обробляти різні HTTP запити
- Працювати з динамічними URL

---

### [Частина 2: Middleware (Проміжні обробники)](EXPRESS_GUIDE_PART_2.md)
**Теми:**
- Концепція конвеєра middleware
- Функція `next()` та її важливість
- Вбудовані middleware (express.json, express.urlencoded)
- Створення власних middleware
- Порядок виконання middleware
- Error-handling middleware

**Що вміємо після цієї частини:**
- Створювати власні middleware
- Парсити JSON та form data
- Логувати запити
- Обробляти помилки централізовано

---

### [Частина 3: CORS та структура проекту](EXPRESS_GUIDE_PART_3.md)
**Теми:**
- Що таке CORS і як його налаштувати
- Preflight запити (OPTIONS)
- Express Router для модульності
- Controllers для бізнес-логіки
- Environment variables (.env файли)
- Організація структури проекту

**Що вміємо після цієї частини:**
- Вирішувати CORS проблеми
- Структурувати великі проекти
- Використовувати .env для конфігурації
- Розділяти код на модулі

---

### [Частина 4: MongoDB та Mongoose](EXPRESS_GUIDE_PART_4.md)
**Теми:**
- Що таке MongoDB (NoSQL база даних)
- Встановлення та підключення
- Mongoose схеми та моделі
- CRUD операції з базою даних
- Query операції та фільтрація
- Валідація даних
- Інтеграція з Express

**Що вміємо після цієї частини:**
- Підключатися до MongoDB
- Створювати схеми з валідацією
- Виконувати CRUD операції
- Фільтрувати та сортувати дані
- Створювати повноцінні REST API

---

### [Частина 5: Bcrypt та безпека паролів](EXPRESS_GUIDE_PART_5.md)
**Теми:**
- Чому не можна зберігати паролі як текст
- Що таке хешування та salt
- Bcrypt для безпечного зберігання паролів
- Rounds та складність хешування
- Автоматичне хешування в Mongoose
- Порівняння паролів при логіні
- Валідація паролів

**Що вміємо після цієї частини:**
- Хешувати паролі безпечно
- Реалізувати реєстрацію користувачів
- Перевіряти паролі при логіні
- Створювати middleware для Mongoose

---

### [Частина 6: JWT авторизація](EXPRESS_GUIDE_PART_6.md)
**Теми:**
- Що таке JWT (JSON Web Tokens)
- Структура JWT (header, payload, signature)
- Створення та перевірка токенів
- Middleware для захисту маршрутів
- Role-based access control (RBAC)
- Access vs Refresh tokens
- Де зберігати токени (localStorage vs cookies)

**Що вміємо після цієї частини:**
- Створювати JWT токени
- Захищати маршрути
- Реалізувати повну систему авторизації
- Використовувати refresh tokens
- Розділяти доступ за ролями

---

## 🛠️ Технології

- **Node.js** - JavaScript runtime
- **Express.js** - Web framework
- **MongoDB** - NoSQL база даних
- **Mongoose** - ODM для MongoDB
- **Bcrypt** - Хешування паролів
- **JWT** - Токени авторизації
- **CORS** - Cross-origin requests
- **dotenv** - Environment variables

---

## 📋 Передумови

### Що потрібно знати:
- Базовий JavaScript (змінні, функції, async/await)
- Основи HTTP (що таке GET, POST запити)
- Базова робота з терміналом

### Що встановити:
- Node.js (версія 18+)
- MongoDB (локально або MongoDB Atlas)
- Postman або Thunder Client (для тестування API)
- VS Code (рекомендований редактор)

---

## 🚀 Швидкий старт

### 1. Клонувати репозиторій або створити проект

```bash
mkdir my-express-api
cd my-express-api
npm init -y
```

### 2. Встановити залежності

```bash
npm install express mongoose bcryptjs jsonwebtoken cors dotenv
npm install --save-dev nodemon
```

### 3. Створити базову структуру

```
my-express-api/
├── config/
│   └── db.js
├── controllers/
│   └── authController.js
├── middleware/
│   └── auth.js
├── models/
│   └── User.js
├── routes/
│   └── auth.js
├── utils/
│   └── jwt.js
├── .env
├── .gitignore
├── package.json
└── server.js
```

### 4. Запустити сервер

```bash
npm run dev
```

---

## 📝 Практичні завдання

Після кожної частини є практичне завдання для закріплення знань:

1. **Частина 1:** Створити простий REST API для продуктів
2. **Частина 2:** Додати middleware для логування та валідації
3. **Частина 3:** Структурувати проект з роутерами та контролерами
4. **Частина 4:** Інтегрувати MongoDB та створити Product модель
5. **Частина 5:** Реалізувати реєстрацію з хешуванням паролів
6. **Частина 6:** Додати JWT авторизацію та захищені маршрути

---

## 🎓 Фінальний проект

Після завершення всіх частин, ви зможете створити повноцінний **E-commerce Backend API** з:

- ✅ Реєстрація та логін користувачів
- ✅ JWT авторизація
- ✅ CRUD для категорій товарів
- ✅ CRUD для товарів
- ✅ Фільтрація товарів за категорією
- ✅ Захищені адмін маршрути
- ✅ Валідація даних
- ✅ Обробка помилок

---

## 📚 Додаткові ресурси

### Офіційна документація:
- [Express.js](https://expressjs.com/)
- [MongoDB](https://www.mongodb.com/docs/)
- [Mongoose](https://mongoosejs.com/)
- [JWT.io](https://jwt.io/)

### Корисні інструменти:
- [Postman](https://www.postman.com/) - тестування API
- [MongoDB Compass](https://www.mongodb.com/products/compass) - GUI для MongoDB
- [REST Client](https://marketplace.visualstudio.com/items?itemName=humao.rest-client) - VS Code extension

### Відео курси (англійською):
- [Traversy Media - Express Crash Course](https://www.youtube.com/watch?v=L72fhGm1tfE)
- [Net Ninja - Node.js Crash Course](https://www.youtube.com/playlist?list=PL4cUxeGkcC9jsz4LDYc6kv3ymONOKxwBU)

---

## 🤝 Внесок

Якщо ви знайшли помилку або хочете покращити курс:
1. Опишіть проблему або пропозицію
2. Запропонуйте зміни
3. Додайте приклади коду

---

## 📄 Ліцензія

Цей навчальний матеріал вільний для використання в освітніх цілях.

---

## ✨ Автори

Створено для студентів курсу веб-розробки.

---

## 🎯 Наступні кроки після курсу

Після завершення курсу рекомендуємо:

1. **Практика** - створіть 2-3 власні проекти
2. **TypeScript** - вивчіть типізований JavaScript
3. **Testing** - навчіться писати тести (Jest, Supertest)
4. **Deployment** - задеплойте на Heroku, Railway, або Vercel
5. **GraphQL** - альтернатива REST API
6. **WebSockets** - real-time комунікація
7. **Docker** - контейнеризація додатків

---

**Успіхів у навчанні та розробці! 🚀**

---

## 📞 Контакти

Якщо виникають питання - пишіть у чат курсу або викладачу.

**Happy Coding! 💻**
