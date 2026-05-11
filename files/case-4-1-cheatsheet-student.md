# CSS Позиціонування — Шпаргалка

> Коротко про головне. Зберігай, використовуй на практиці.

---

## `position` — значення

| Значення | Де "живе" елемент | Виходить з потоку? |
|----------|-------------------|--------------------|
| `static` | Нормальний потік (за замовчуванням) | ❌ |
| `relative` | Там само, але можна зміщувати | ❌ (місце зберігається) |
| `absolute` | Відносно найближчого позиціонованого батька | ✅ |
| `fixed` | Відносно вікна браузера (viewport) | ✅ |
| `sticky` | Як relative → "прилипає" при скролі | Частково |

---

## `top` / `right` / `bottom` / `left`

```css
/* Працюють ТІЛЬКИ якщо position ≠ static */

.box {
  position: relative;
  top: 20px;    /* зміщення від верхнього краю */
  left: 30px;   /* зміщення від лівого краю */
}
```

> ⚠️ При `position: static` ці властивості **ігноруються**.

---

## `z-index` — хто поверх кого

```css
.behind  { position: absolute; z-index: 1; }
.infront { position: absolute; z-index: 10; } /* поверх */
.back    { position: absolute; z-index: -1; } /* позаду тексту */
```

> ⚠️ `z-index` **не працює** без `position` (крім `static`).

---

## `overflow` — що робити якщо вміст не влізає

```css
.box { overflow: visible; }  /* виходить за межі (за замовч.) */
.box { overflow: hidden; }   /* обрізається */
.box { overflow: scroll; }   /* завжди scrollbar */
.box { overflow: auto; }     /* scrollbar тільки якщо потрібно ✅ */

/* Окремо по осях */
.code-block {
  overflow-x: auto;    /* горизонтальний скрол */
  overflow-y: hidden;  /* вертикальне обрізання */
}
```

---

## `visibility` vs `display: none`

```css
.elem { visibility: hidden; } /* невидимий, але МІСЦЕ ЗАЙМАЄ */
.elem { display: none; }      /* невидимий + МІСЦЯ НЕМАЄ */
.elem { opacity: 0; }         /* прозорий, місце є, можна анімувати */
```

**Коли що:**
- Потрібно анімувати появу → `opacity` + `transition`
- Сусідні елементи не повинні зміщуватись → `visibility: hidden`
- Повністю прибрати з розмітки → `display: none`

---

## `position: sticky` — умови роботи

```css
.header {
  position: sticky;
  top: 0; /* ОБОВ'ЯЗКОВО вказати поріг */
}
```

**Не працює якщо:**
- не вказано `top` / `bottom` / `left` / `right`
- батько має `overflow: hidden` або `overflow: auto`

---

## Типові патерни

### Значок поверх картки
```css
.card        { position: relative; }  /* батько */
.card__badge { position: absolute; top: 8px; left: 8px; z-index: 2; }
.card__hover { position: absolute; inset: 0; z-index: 1; }
```

### Фіксована навігація
```css
.navbar {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  z-index: 100;
}
body { padding-top: 60px; } /* компенсація висоти навбару */
```

### Модальне вікно по центру
```css
.modal {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%); /* центрування */
  z-index: 200;
}
.backdrop {
  position: fixed;
  inset: 0; /* top:0; right:0; bottom:0; left:0 */
  background: rgba(0,0,0,.6);
  z-index: 199;
}
```

### Зупинка скролу при відкритій модалці
```js
// відкрити
document.body.style.overflow = 'hidden';

// закрити
document.body.style.overflow = '';
```

---

## Швидка шпаргалка `inset`

```css
/* Повний запис */
top: 0; right: 0; bottom: 0; left: 0;

/* Скорочення (сучасний CSS) */
inset: 0;
```

---

*Тернопільський фаховий коледж ТНТУ ім. Івана Пулюя*
