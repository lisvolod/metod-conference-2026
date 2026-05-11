# CSS Позиціонування — Матеріал для викладача

> Навчальний посібник з методичними нотатками, прикладами для пояснення на парі та типовими помилками студентів.

---

## Структура теми (рекомендований порядок)

```
1. position: static → relative        (~15 хв)
2. position: absolute + батьківський контекст  (~20 хв)
3. position: fixed                     (~10 хв)
4. z-index                             (~10 хв)
5. overflow                            (~15 хв)
6. visibility vs display: none         (~10 хв)
7. position: sticky                    (~10 хв)
8. Практичні патерни (navbar, modal)   (~20 хв)
```

---

## Блок 1 — `position: static` та `position: relative`

### Концепція для пояснення

Починати варто з аналогії: елементи на сторінці стоять у черзі (потік документа). `static` — стоїть у черзі. `relative` — злегка виступив з черги, але місце не залишив.

### Приклад для демонстрації на парі

```html
<!-- Показати студентам живу демонстрацію -->
<div class="container">
  <div class="box box-a">A</div>
  <div class="box box-b">B (relative)</div>
  <div class="box box-c">C</div>
</div>
```

```css
.container { display: flex; gap: 10px; align-items: flex-start; }

.box {
  width: 80px; height: 80px;
  background: #4f8ef7;
  color: white;
  display: flex; align-items: center; justify-content: center;
  font-size: 20px; font-weight: bold;
  border-radius: 8px;
}

.box-b {
  position: relative;
  top: 20px;     /* зміщення вниз від свого місця */
  left: 10px;
  background: #f5a623;
}
```

**Запитати студентів:** "Де стоїть блок C? Чому він не зайняв місце B?"

> 💡 **Методична нотатка:** Більшість студентів очікують, що C підтягнеться. Саме цей момент найкраще закріплює різницю між `relative` (місце зберігається) і `absolute` (місце звільняється).

---

## Блок 2 — `position: absolute` + батьківський контекст

### Найважливіша концепція теми

`absolute` шукає **найближчого позиціонованого предка** (position ≠ static). Якщо не знайшов — прив'язується до `<body>`.

### Демонстрація "пастки"

```html
<!-- ПОМИЛКА — студенти роблять часто -->
<div class="card">               <!-- position: static (за замовч.) -->
  <span class="badge">NEW</span> <!-- position: absolute → прив'яжеться до body! -->
</div>

<!-- ПРАВИЛЬНО -->
<div class="card" style="position: relative;">
  <span class="badge">NEW</span> <!-- тепер відносно .card ✅ -->
</div>
```

### Розширений приклад — картка з overlay (рекомендую показати живим кодом)

```html
<div class="product-card">
  <div class="product-card__image"></div>
  <span class="product-card__badge">−30%</span>
  <div class="product-card__overlay">
    <p>Якісний продукт з натуральних матеріалів</p>
  </div>
  <div class="product-card__info">
    <h3>Назва товару</h3>
    <p class="price">₴ 699</p>
  </div>
</div>
```

```css
.product-card {
  position: relative;       /* контейнер для absolute нащадків */
  width: 220px;
  overflow: hidden;
  border-radius: 12px;
  border: 1px solid #e0e0e0;
}

.product-card__image {
  height: 160px;
  background: linear-gradient(135deg, #667eea, #764ba2);
}

/* Значок — абсолютно в кут картки */
.product-card__badge {
  position: absolute;
  top: 10px;
  left: 10px;
  background: #f76f6f;
  color: white;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 12px;
  font-weight: 700;
  z-index: 2;
}

/* Overlay — покриває всю картку */
.product-card__overlay {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.75);
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
  text-align: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: 1;
}

.product-card:hover .product-card__overlay {
  opacity: 1;
}

.product-card__info {
  padding: 12px 16px;
}

.price { color: #f5a623; font-weight: 700; margin-top: 4px; }
```

> 💡 **Методична нотатка:** Зверніть увагу на `z-index: 2` у `.badge` і `z-index: 1` у `.overlay`. Без цього значок буде перекритий overlay при hover. Це гарний момент для пояснення порядку шарів.

---

## Блок 3 — `position: fixed`

### Практичний приклад — фіксована навігація

```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 60px;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,.1);
  z-index: 1000;
  display: flex;
  align-items: center;
  padding: 0 24px;
}

/* ВАЖЛИВО: компенсуємо висоту навбару */
body {
  padding-top: 60px;
}
```

> ⚠️ **Типова помилка студентів:** забувають `padding-top` на body — контент ховається під навбаром. Покажіть це навмисно.

### Кнопка "прокрутити вгору"

```css
.scroll-top {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: #4f8ef7;
  color: white;
  border: none;
  cursor: pointer;
  font-size: 18px;
  visibility: hidden;   /* не display:none — для анімації */
  opacity: 0;
  transition: opacity 0.3s, visibility 0.3s;
  z-index: 500;
}
```

```js
window.addEventListener('scroll', () => {
  const btn = document.querySelector('.scroll-top');
  if (window.scrollY > 200) {
    btn.style.visibility = 'visible';
    btn.style.opacity = '1';
  } else {
    btn.style.visibility = 'hidden';
    btn.style.opacity = '0';
  }
});
```

> 💡 **Чому `visibility` а не `display: none`?** Властивість `display` не анімується. `visibility: hidden` + `opacity: 0` дозволяє плавне зникнення через `transition`. Це чудова точка переходу до наступного блоку теми.

---

## Блок 4 — `z-index`

### Демонстрація "стопки паперів"

```html
<div class="stack-demo">
  <div class="layer" style="background:#4f8ef7; z-index:1; top:0; left:0">z:1</div>
  <div class="layer" style="background:#7c5cfc; z-index:3; top:30px; left:40px">z:3</div>
  <div class="layer" style="background:#3dd68c; z-index:2; top:60px; left:20px">z:2</div>
</div>
```

```css
.stack-demo { position: relative; height: 160px; }
.layer {
  position: absolute;
  width: 120px; height: 80px;
  border-radius: 8px;
  display: flex; align-items: center; justify-content: center;
  color: white; font-weight: 700; font-size: 18px;
}
```

### Stacking context — для поглибленого розгляду

> 💡 **Для сильніших студентів:** кожен елемент з `position` + `z-index` або `opacity < 1` або `transform` створює новий **stacking context**. `z-index` дочірніх елементів порівнюється лише всередині свого контексту. Це пояснює чому іноді `z-index: 9999` не працює — батько має низький `z-index`.

---

## Блок 5 — `overflow`

### Практичний приклад — блок коду з горизонтальним скролом

```css
.code-snippet {
  background: #1e1e1e;
  border-radius: 8px;
  padding: 16px;
  overflow-x: auto;    /* горизонтальний скрол для довгих рядків */
  overflow-y: hidden;
  white-space: nowrap; /* заборонити перенос рядків */
}
```

### Практичний приклад — скролящий список у сайдбарі

```css
.sidebar-list {
  max-height: 300px;
  overflow-y: auto;
  /* overflow-x не вказано → за замовч. visible */
}
```

> ⚠️ **Підводний камінь:** `overflow: hidden` на батьку **вимикає** `position: sticky` у нащадків. Покажіть студентам цей баг навмисно — це запам'ятовується краще ніж будь-яке пояснення.

---

## Блок 6 — `visibility` vs `display: none`

### Порівняльна демонстрація

```html
<div class="row">
  <button onclick="toggleDisplay()">display: none</button>
  <div id="d-elem" class="box">A</div>
  <div class="box green">B</div>
</div>

<div class="row">
  <button onclick="toggleVisibility()">visibility: hidden</button>
  <div id="v-elem" class="box">A</div>
  <div class="box green">B</div>
</div>
```

```js
function toggleDisplay() {
  const el = document.getElementById('d-elem');
  el.style.display = el.style.display === 'none' ? '' : 'none';
}
function toggleVisibility() {
  const el = document.getElementById('v-elem');
  el.style.visibility = el.style.visibility === 'hidden' ? '' : 'hidden';
}
```

**Запитати:** "Що відбувається з блоком B у кожному випадку?"

---

## Блок 7 — `position: sticky`

### Демонстрація з таблицею

```html
<div class="table-wrap">
  <table>
    <thead>
      <tr>
        <th style="position: sticky; top: 0; background: white;">Назва</th>
        <th style="position: sticky; top: 0; background: white;">Ціна</th>
      </tr>
    </thead>
    <tbody>
      <!-- 20+ рядків для появи скролу -->
    </tbody>
  </table>
</div>
```

```css
.table-wrap {
  max-height: 300px;
  overflow-y: auto; /* скролимо саму таблицю, не сторінку */
}
```

> 💡 **Важлива деталь:** sticky в таблиці прив'язується до скролящого контейнера `.table-wrap`, а не до viewport. Тому `overflow-y: auto` тут не заважає sticky — це інший контекст.

---

## Блок 8 — Патерн: Модальне вікно

Це фінальний практичний приклад, який об'єднує `fixed`, `z-index`, `overflow` і `transform`.

```html
<button id="open-modal">Відкрити модалку</button>

<div class="backdrop" id="backdrop">
  <div class="modal" id="modal">
    <button class="modal__close" id="close-modal">✕</button>
    <h2>Заголовок</h2>
    <p>Будь-який вміст модалки...</p>
  </div>
</div>
```

```css
.backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 1000;
  display: none;          /* приховано за замовч. */
  align-items: center;
  justify-content: center;
}

.backdrop.active {
  display: flex;          /* показуємо через flex для центрування */
}

.modal {
  position: relative;     /* для кнопки ✕ */
  background: white;
  border-radius: 12px;
  padding: 32px;
  max-width: 480px;
  width: 90%;
  animation: slideUp 0.3s ease;
}

.modal__close {
  position: absolute;
  top: 12px;
  right: 16px;
  background: none;
  border: none;
  font-size: 20px;
  cursor: pointer;
}

@keyframes slideUp {
  from { opacity: 0; transform: translateY(20px); }
  to   { opacity: 1; transform: translateY(0); }
}
```

```js
const openBtn  = document.getElementById('open-modal');
const closeBtn = document.getElementById('close-modal');
const backdrop = document.getElementById('backdrop');

function openModal() {
  backdrop.classList.add('active');
  document.body.style.overflow = 'hidden'; /* зупиняємо скрол */
}

function closeModal() {
  backdrop.classList.remove('active');
  document.body.style.overflow = '';
}

openBtn.addEventListener('click', openModal);
closeBtn.addEventListener('click', closeModal);

/* Закрити по кліку на backdrop (не на модалку) */
backdrop.addEventListener('click', (e) => {
  if (e.target === backdrop) closeModal();
});
```

> 💡 **Методична нотатка:** Цей приклад — чудове підсумкове завдання. Він містить `position: fixed` (backdrop), `position: relative` + `position: absolute` (кнопка ✕), `z-index`, `overflow: hidden` на body та `transform` в анімації. Фактично — весь матеріал теми в одному компоненті.

---

## Типові помилки студентів — зведена таблиця

| Помилка | Симптом | Виправлення |
|---------|---------|-------------|
| `absolute` без `relative` на батьку | Елемент "вилітає" на всю сторінку | Додати `position: relative` батьку |
| `z-index` без `position` | Порядок шарів не змінюється | Додати `position: relative/absolute` |
| `sticky` не "прилипає" | Елемент просто скролиться | Перевірити `overflow` у батьків |
| Забутий `padding-top` при `fixed` navbar | Контент захований під навбаром | `body { padding-top: висота_навбару }` |
| `display: none` замість `visibility` | Анімація "стрибає" без переходу | Замінити на `visibility` + `opacity` |
| `top: 50%` без `transform` | Модалка не по центру | Додати `transform: translate(-50%, -50%)` |

---

*Тернопільський фаховий коледж ТНТУ ім. Івана Пулюя*  
*Дисципліна: Веб-технології*
