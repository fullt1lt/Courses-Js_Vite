# Лекция 8. События и формы: как страница «реагирует» на пользователя

![](./images/events.webp)

## Вступление

В прошлой лекции мы рассмотрели, как браузер отображает страницу, и узнали о том, что такое DOM.  
Но сам по себе DOM - это просто структура данных, которая описывает элементы на странице.

Чтобы сделать страницу интерактивной и позволить ей реагировать на действия пользователя, нам нужны **события**.

Например:
- пользователь *кликает* по кнопке,
- заполняет форму,
- перемещает мышь по странице,
- нажимает клавиши.

Браузер фиксирует эти действия и может вызывать ваш код в нужный момент.

### Что такое события

**События (events)** - это действия или происшествия, которые происходят в браузере и могут быть обнаружены с помощью JavaScript.  
События могут быть вызваны пользователем (клики мышью, нажатия клавиш, отправка форм) или происходить автоматически (загрузка страницы, изменение размера окна). [Документация по событиям](https://doka.guide/js/events/)

Примеры событий:
- `click` - происходит при клике мышью на элемент
- `submit` - происходит при отправке формы
- `keydown` - происходит при нажатии клавиши на клавиатуре
- `load` - происходит, когда страница полностью загружена

То есть события позволяют нам “слушать” действия пользователя и реагировать на них, выполняя код или изменяя содержимое страницы.

> **Важно:** события - фундаментальная часть взаимодействия между пользователем и веб-страницей. Они позволяют создавать динамические и интерактивные интерфейсы, которые реагируют на действия пользователя в реальном времени.

### Обработчик событий - это реакция на событие

Главная идея проста: есть элемент на странице, на нём происходит событие, и мы хотим, чтобы при этом событии выполнялась функция.  
Эта функция называется **обработчиком события**.

Схема взаимодействия:

```text
Событие → Обработчик события → Реакция (код)
```

Пример:
1. пользователь кликает по кнопке “Добавить в корзину”
2. код реагирует на это событие и добавляет товар в корзину
3. счётчик товаров увеличивается
4. пользователь видит, что товар добавлен

### Почему формы важны

Форма - один из самых распространённых способов взаимодействия пользователя с веб-страницей.

Регистрация, логин, поиск, отправка комментариев, фильтры - всё это формы.

Форма почти всегда требует одного и того же:
1. прочитать данные, которые пользователь ввёл в форму
2. проверить данные на валидность
3. обработать “отправку” (в этой лекции - без API)
4. показать результат пользователю (успех или ошибки)

И тут важный момент: браузер по умолчанию отправляет форму и перезагружает страницу.  
Но с помощью JavaScript мы можем предотвратить это поведение и управлять формой самостоятельно.

---

## Обработчики событий

Чтобы работать с событиями, нам нужно уметь делать главное:
- найти элемент в DOM
- “подписаться” на событие
- выполнить код, когда событие произошло

### Способ 1. События в HTML

Самый простой способ назначить обработчик - использовать атрибуты в HTML.

```html
<button onclick="alert('Button clicked!')">Click me</button>
```

При клике по кнопке выполнится код из `onclick`. Но этот способ не рекомендуется, потому что:
- смешивает разметку и логику
- сложно поддерживать и масштабировать
- нельзя назначить несколько обработчиков на одно событие


### Способ 2. `addEventListener` - основной способ работать с событиями

Самый распространённый способ добавить обработчик - использовать метод `addEventListener()`.

Синтаксис:

```javascript
element.addEventListener("event", handler);
```

**Пример:**

```html
<button id="myButton">Нажми меня</button>
```

```javascript
const btn = document.getElementById("myButton");

btn.addEventListener("click", function () {
  alert("Кнопка была нажата!");
});
```

Теперь при клике по кнопке выполняется обработчик.

### Важный момент: передаём функцию, а не вызываем её

Очень частая ошибка:

```javascript
btn.addEventListener("click", handleClick());
```

Так делать нельзя, потому что `handleClick()` вызовется сразу, а в `addEventListener` попадёт результат вызова.

**Правильно:**

```javascript
function handleClick() {
  console.log("Кнопка была нажата!");
}

btn.addEventListener("click", handleClick);
```

Или так (для коротких обработчиков):

```javascript
btn.addEventListener("click", () => {
  console.log("Кнопка была нажата!");
});
```

**Мини-правило:**
> в `addEventListener` мы передаём ссылку на функцию.

### Почему обработчик лучше выносить в отдельную функцию

Вынесенный обработчик:
- проще читать
- проще переиспользовать
- можно удалить через `removeEventListener`

```javascript
const buyBtn = document.getElementById("buy-btn");

function addToCart() {
  console.log("Product added");
}

buyBtn.addEventListener("click", addToCart);
```

### Несколько обработчиков на один элемент

`addEventListener` позволяет добавлять несколько обработчиков на одно событие:

```javascript
buyBtn.addEventListener("click", () => console.log("Log 1"));
buyBtn.addEventListener("click", () => console.log("Log 2"));
```

Обе функции выполнятся при клике.

### Удаление обработчика события: `removeEventListener`

Иногда обработчик нужно удалить. Это делается через `removeEventListener()`.

```javascript
const buyBtn = document.getElementById("buy-btn");

function addToCart() {
  console.log("Added once");
  buyBtn.removeEventListener("click", addToCart);
}

buyBtn.addEventListener("click", addToCart);
```

**Важно:** удалить можно только ту функцию, на которую есть ссылка.

Неправильно (не сработает):

```javascript
buyBtn.addEventListener("click", () => console.log("Click"));
buyBtn.removeEventListener("click", () => console.log("Click"));
```

Правильно:

```javascript
const handleClick = () => console.log("Click");

buyBtn.addEventListener("click", handleClick);
buyBtn.removeEventListener("click", handleClick);
```

### Способ 3. `onclick` - альтернативный способ

Можно назначить обработчик как DOM-свойство:

```javascript
buyBtn.onclick = function () {
  console.log("Clicked");
};
```

Но если назначить обработчик повторно, предыдущий будет перезаписан:

```javascript
buyBtn.onclick = function () {
  console.log("First handler");
};

buyBtn.onclick = function () {
  console.log("Second handler");
};
```

Поэтому в проектах чаще используют `addEventListener()`.

---

#### События, которые чаще всего используют на практике

Событий в браузере много, но в реальных проектах постоянно встречаются одни и те же. Удобно держать их “группами”.

**1) События мыши**
- `click` - обычный клик
- `dblclick` - двойной клик
- `contextmenu` - клик правой кнопкой
- `mousedown / mouseup` - нажал / отпустил кнопку мыши
- `mousemove` - движение мыши (осторожно: событие частое)
- `mouseenter / mouseleave` - курсор вошёл / вышел (не всплывают)
- `mouseover / mouseout` - навёл / ушёл (всплывают)

Мини-правило: для “наведения” чаще используют `mouseenter / mouseleave`.

**2) События клавиатуры**
- `keydown` - клавиша нажата
- `keyup` - клавиша отпущена

`keypress` сейчас почти не используют.

**3) События фокуса (важны для форм)**
- `focus` - поле получило фокус
- `blur` - поле потеряло фокус
- `focusin / focusout` - то же самое, но всплывают

Мини-правило: валидацию часто делают на `blur`, а подсказки - на `focus`.

**4) События ввода**
- `input` - значение меняется “в реальном времени”
- `change` - значение зафиксировано
- `paste` - вставка из буфера
- `cut` - вырезание
- `copy` - копирование

**5) События формы**
- `submit` - отправка формы
- `reset` - сброс формы

**6) События документа и окна**
- `DOMContentLoaded` - DOM построен
- `load` - страница полностью загрузилась
- `scroll` - прокрутка (событие частое)
- `resize` - изменение размера окна

Мини-правило: если нужно работать с DOM - чаще хватает `DOMContentLoaded`.

---

## Объект события `event`

![](./images/event_delegation.png)

Мы научились слушать события и реагировать на них.  
Но как узнать, что именно произошло и на каком элементе?

Для этого браузер передаёт в обработчик объект события - `event`.

### Что такое `event`

Когда вы вешаете обработчик, браузер вызывает его с аргументом `event`.

```javascript
const buyBtn = document.getElementById("buy-btn");

buyBtn.addEventListener("click", function (event) {
  console.log(event);
});
```

`event` содержит информацию о событии: тип, целевой элемент, координаты, нажатые клавиши и т.д.

Примерный вид (упрощённо):

```text
{
  type: "click",
  target: button#buy-btn,
  clientX: 100,
  clientY: 200,
  ...
}
```

### `event.type` - какое событие произошло

```javascript
buyBtn.addEventListener("click", function (event) {
  console.log(event.type); // "click"
});
```

### `event.target` - на каком элементе произошло событие

`event.target` указывает на элемент, по которому кликнули реально.

```html
<button id="buy-btn">
  <span class="icon">🛒</span>
  Buy
</button>
```

Если кликнуть по `span.icon`, то `target` будет `span`, а не `button`:

```javascript
buyBtn.addEventListener("click", function (event) {
  console.log(event.target);
});
```

### `event.currentTarget` - элемент, на котором висит обработчик

`event.currentTarget` - элемент, на который повешен обработчик.

```javascript
buyBtn.addEventListener("click", function (event) {
  console.log(event.currentTarget); // button#buy-btn
});
```

**Мини-правило:**
- `target` - где событие произошло
- `currentTarget` - где стоит обработчик

### `event.preventDefault()` - отменить стандартное поведение

Иногда браузер выполняет действие сам (переход по ссылке, отправка формы).  
Если мы хотим управлять этим - используем `event.preventDefault()`.

**Пример со ссылкой:**

```html
<a href="https://google.com" id="link">Go</a>
```

```javascript
const link = document.getElementById("link");

link.addEventListener("click", function (event) {
  event.preventDefault();
  console.log("Ссылка не перешла, потому что мы отменили поведение");
});
```

**Пример с формой:**

```javascript
const form = document.getElementById("login-form");

form.addEventListener("submit", function (event) {
  event.preventDefault();
  console.log("Форма отправлена, но страница не перезагрузилась");
});
```

### `event.stopPropagation()` - остановить всплытие (база)

Иногда нужно остановить распространение события вверх по DOM:

```javascript
const child = document.getElementById("child");

child.addEventListener("click", function (event) {
  event.stopPropagation();
  console.log("Клик обработан, но не всплывает дальше");
});
```

### Мини-вывод

- `event.type` - тип события
- `event.target` - где событие произошло
- `event.currentTarget` - где стоит обработчик
- `event.preventDefault()` - отмена стандартного поведения
- `event.stopPropagation()` - остановка всплытия

---

## Делегирование событий

`Делегирование событий` - это техника, которая позволяет обрабатывать события на родительском элементе вместо того, чтобы назначать обработчики на каждый дочерний элемент.

В реальных проектах элементы часто:
- добавляются динамически
- удаляются
- изменяются
- появляются после фильтрации или поиска
- отображаются списками

Если на каждый элемент вешать обработчик вручную, код усложняется и легче допустить ошибки.

Делегирование решает проблему проще:
- назначаем один обработчик на родительский элемент
- внутри обработчика используем `event.target`, чтобы определить, где произошло событие, и реагируем на это

### Почему делегирование работает: всплытие событий

Делегирование работает благодаря механизму `всплытия событий`.  
Событие “поднимается” от дочернего элемента к его родителям.

Пример структуры:

```html
<div id="products">
  <div class="product">
    <button data-action="cart">Add</button>
  </div>
</div>
```

Схема всплытия клика:

```text
document
  |
  div#products (обработчик)
    |
    div.product
      |
      button[data-action] (клик)
```

### Как использовать делегирование на практике

У каждой карточки товара есть две кнопки:
- добавить в избранное
- добавить в корзину

```html
<div id="products">
  <div class="product" data-id="1">
    <h3 class="title">Milk</h3>
    <button class="btn" data-action="favorite">☆ Favorite</button>
    <button class="btn" data-action="cart">🛒 Add to cart</button>
  </div>

  <div class="product" data-id="2">
    <h3 class="title">Bread</h3>
    <button class="btn" data-action="favorite">☆ Favorite</button>
    <button class="btn" data-action="cart">🛒 Add to cart</button>
  </div>

  <div class="product" data-id="3">
    <h3 class="title">Eggs</h3>
    <button class="btn" data-action="favorite">☆ Favorite</button>
    <button class="btn" data-action="cart">🛒 Add to cart</button>
  </div>
</div>
```

Один обработчик на контейнер:

```javascript
const products = document.getElementById("products");

products.addEventListener("click", function (event) {
  const button = event.target.closest("button[data-action]");
  if (!button) return;

  const action = button.dataset.action; // "favorite" или "cart"
  const product = button.closest(".product");
  const productId = product.dataset.id;

  if (action === "favorite") {
    product.classList.toggle("is-favorite");
    console.log("toggle favorite:", productId);
  }

  if (action === "cart") {
    console.log("add to cart:", productId);
  }
});
```

Тут важно:
- обработчик висит на `#products`, а не на каждой кнопке
- кнопку ищем через `closest("button[data-action]")`
- `data-action` говорит, что именно нажали
- `data-id` говорит, какой товар нажали

### Про `closest()`

`closest()` поднимается вверх по DOM и ищет ближайшего родителя (или сам элемент), который подходит под селектор.

Мы используем `closest()` по двум причинам:
1) клик может быть по вложенному элементу внутри кнопки
2) нужно быстро найти “контекст” - карточку товара `.product`

### Мини-вывод

- делегирование позволяет назначить один обработчик на родителя и обрабатывать события от всех дочерних элементов
- это работает благодаря всплытию
- в делегировании часто используют `event.target` + `closest()`

---

## Формы: чтение данных

![](./images/form-navigation.svg)

Форма - это главный способ получить данные от пользователя.  
Регистрация, логин, поиск, фильтры, оформление заказа - всё это формы.

Чтобы работать с формой в JavaScript, нужно уметь:
- получить доступ к форме и полям
- прочитать значения полей
- понимать разницу между `input`, `checkbox`, `radio`, `select`, `textarea`

### Главная идея

Форма - это DOM-элемент.  
А её поля - DOM-элементы внутри формы.

---

### 1) Как найти форму

```html
<form id="login-form">
  <input type="email" name="email" placeholder="Email" />
  <input type="password" name="password" placeholder="Password" />
  <button type="submit">Login</button>
</form>
```

```javascript
const form = document.getElementById("login-form");
console.log(form);
```

---

### 2) `form.elements` - доступ ко всем полям формы

`form.elements` содержит все поля управления формы.

```javascript
console.log(form.elements);
```

#### Доступ к полю по `name`

```javascript
const emailInput = form.elements.email;
const passwordInput = form.elements.password;

console.log(emailInput.value);
console.log(passwordInput.value);
```

---

### 3) `input` и `textarea`: читаем через `value`

```html
<input type="text" name="username" />
<textarea name="about"></textarea>
```

```javascript
const username = form.elements.username.value;
const about = form.elements.about.value;

console.log(username, about);
```

---

### 4) `checkbox`: читаем через `checked`

```html
<label>
  <input type="checkbox" name="agree" />
  I agree
</label>
```

```javascript
const agree = form.elements.agree.checked;
console.log(agree); // true/false
```

---

### 5) `radio`: читаем выбранную

```html
<label><input type="radio" name="delivery" value="pickup" /> Pickup</label>
<label><input type="radio" name="delivery" value="courier" /> Courier</label>
```

```javascript
const selected = form.querySelector('input[name="delivery"]:checked');
console.log(selected?.value); // "pickup" или "courier"
```

---

### 6) `select`: читаем через `value`

```html
<select name="city">
  <option value="prague">Prague</option>
  <option value="brno">Brno</option>
  <option value="ostrava">Ostrava</option>
</select>
```

```javascript
const city = form.elements.city.value;
console.log(city);
```

---

### 7) Чтение данных формы: вручную vs `FormData`

#### Вручную (когда важны типы и логика)

```javascript
const data = {
  email: form.elements.email.value.trim(),
  password: form.elements.password.value,
  agree: form.elements.agree.checked,
};

console.log(data);
```

#### `FormData` (собрать всё разом)

```javascript
const data = Object.fromEntries(new FormData(form).entries());
console.log(data);
```

**Важно:** `FormData` работает “как форма”:
- неотмеченный checkbox может не попасть в данные
- отмеченный checkbox обычно приходит как `"on"` или как заданный `value`

---

### Мини-правила

- текстовые поля читаем через `value`
- чекбоксы читаем через `checked`
- radio читаем через `:checked`
- `select` читаем через `value`
- если нужно собрать всё разом - `FormData`
- если нужны типы и логика - читаем вручную

---

## Формы: отправка (`submit`)

Главное действие формы - отправка.

Когда пользователь нажимает кнопку `type="submit"` или жмёт `Enter`, браузер генерирует событие `submit`.

### Что делает браузер по умолчанию

По умолчанию браузер:
1. собирает данные
2. отправляет форму на URL из `action`
3. перезагружает страницу

В этой лекции мы будем **перехватывать отправку** и управлять ей самостоятельно.

### Событие `submit`

```javascript
form.addEventListener("submit", function (event) {
  console.log("form submitted");
});
```

Но страница всё равно перезагрузится, если не отменить поведение браузера.

### `event.preventDefault()` при `submit`

```javascript
form.addEventListener("submit", function (event) {
  event.preventDefault();
  console.log("page not reloaded");
});
```

### Чтение данных формы при `submit`

```javascript
form.addEventListener("submit", function (event) {
  event.preventDefault();

  const email = form.elements.email.value.trim();
  const password = form.elements.password.value;

  console.log({ email, password });
});
```

---

## Валидация и `preventDefault()` (практика)

Валидация - это проверка данных формы перед обработкой.

### Разметка: поля + места под ошибки

```html
<form id="login-form" novalidate>
  <div class="field">
    <label>Email</label>
    <input type="email" name="email" />
    <small class="error" data-error-for="email"></small>
  </div>

  <div class="field">
    <label>Password</label>
    <input type="password" name="password" />
    <small class="error" data-error-for="password"></small>
  </div>

  <div class="field">
    <label>
      <input type="checkbox" name="agree" />
      I agree
    </label>
    <small class="error" data-error-for="agree"></small>
  </div>

  <button type="submit">Send</button>
</form>
```

`novalidate` отключает встроенную браузерную валидацию, чтобы мы управляли всем сами.

### Инструменты: показать ошибку и очистить ошибки

```javascript
function setError(fieldName, message) {
  const errorEl = form.querySelector(`[data-error-for="${fieldName}"]`);
  if (errorEl) errorEl.textContent = message;
}

function clearErrors() {
  form.querySelectorAll(".error").forEach((el) => (el.textContent = ""));
}
```

### Функция валидации

```javascript
function validateForm() {
  const errors = {};

  const email = form.elements.email.value.trim();
  const password = form.elements.password.value;
  const agree = form.elements.agree.checked;

  if (!email) {
    errors.email = "Email is required";
  } else if (!email.includes("@")) {
    errors.email = "Email must contain @";
  }

  if (!password) {
    errors.password = "Password is required";
  } else if (password.length < 6) {
    errors.password = "Password must be at least 6 characters";
  }

  if (!agree) {
    errors.agree = "You must accept the agreement";
  }

  return errors;
}
```

### Валидация на `submit`

```javascript
form.addEventListener("submit", function (event) {
  event.preventDefault();

  clearErrors();

  const errors = validateForm();

  if (Object.keys(errors).length > 0) {
    for (const [field, message] of Object.entries(errors)) {
      setError(field, message);
    }
    return;
  }

  const data = {
    email: form.elements.email.value.trim(),
    password: form.elements.password.value,
    agree: form.elements.agree.checked,
  };

  console.log("Form is valid:", data);
});
```

### Когда валидировать: `input` vs `submit`

В базовом варианте достаточно валидации на `submit`.  
Но чтобы улучшить UX, можно очищать ошибку при вводе:

```javascript
form.addEventListener("input", function (event) {
  const field = event.target.name;
  if (!field) return;

  setError(field, "");
});
```

И валидировать поле, когда пользователь закончил ввод.

Чтобы не усложнять `blur` и capture, используем `focusout` (он всплывает):

```javascript
function validateField(fieldName) {
  const errors = validateForm();
  return errors[fieldName] ?? "";
}

form.addEventListener("focusout", function (event) {
  const field = event.target.name;
  if (!field) return;

  setError(field, validateField(field));
});
```
Продолжаем проект `Storefront`. На этой лекции витрина становится интерактивной: добавляем действия на карточках, фильтры и форму быстрого заказа.

## Практика

1. Добавьте в карточку товара кнопки:
   - `data-action="favorite"`
   - `data-action="cart"`
   - `data-action="details"`

   Повесьте **один** обработчик `click` на контейнер каталога и определяйте действие через `event.target.closest("[data-action]")`.

2. Реализуйте сценарий `favorite`:
   - переключайте класс `.is-favorite` у карточки;
   - меняйте текст кнопки `Favorite` / `Saved`;
   - храните выбранные `id` в массиве `favoriteIds`;
   - обновляйте бейдж “Избранное: N”.

3. Реализуйте сценарий `cart`:
   - добавляйте товар в корзину;
   - увеличивайте счётчик товаров;
   - показывайте рядом с шапкой краткое сообщение `Added to cart`;
   - клик по кнопке внутри карточки не должен ломать делегирование.

4. Добавьте форму фильтров:
   - поле поиска `search`;
   - `select` по категории;
   - `checkbox` “Только в наличии”;
   - `select` сортировки (`price-asc`, `price-desc`, `rating-desc`).

   На `submit` формы:
   - делайте `event.preventDefault()`;
   - собирайте данные через `FormData`;
   - фильтруйте массив товаров;
   - заново рендерьте каталог.

5. Добавьте кнопку `Reset filters`.
   Она должна:
   - очищать форму;
   - возвращать полный список товаров;
   - сбрасывать сообщение об empty-state.

6. Создайте форму быстрого заказа:
   - `name`
   - `email`
   - `phone`
   - `comment`
   - `agree`

   Под формой предусмотрите блоки ошибок через `data-error-for`.

7. Реализуйте:
   - `setError(fieldName, message)`
   - `clearErrors()`
   - `validateOrderForm(values)`

   Минимальные правила:
   - имя не пустое;
   - email содержит `@`;
   - телефон не короче 10 символов;
   - `agree === true`.

8. Добавьте “живую” валидацию:
   - на `input` очищайте ошибку только у текущего поля;
   - на `focusout` проверяйте именно то поле, с которого ушёл фокус;
   - после успешной отправки показывайте текст `Order request sent`.

## Домашняя работа

Соберите страницу **`Storefront — интерактивный каталог`**.

### Что должно быть на странице

1. Каталог товаров с карточками.
2. Бейджи:
   - количество товаров в корзине;
   - количество избранных товаров.
3. Форма фильтров.
4. Форма быстрого заказа.
5. Блок сообщения об успехе или ошибке.

### Обязательная логика каталога

1. Один делегированный обработчик `click` на список товаров.
2. Поддержка действий:
   - `favorite`
   - `cart`
   - `details`
3. При `favorite`:
   - карточка получает активное состояние;
   - бейдж избранного обновляется.
4. При `cart`:
   - увеличивается количество товаров в корзине;
   - пересчитывается итоговая сумма корзины.
5. Кнопка `Clear cart`:
   - очищает корзину;
   - сбрасывает счётчик и сумму.

### Обязательная логика фильтров

1. Поиск по названию без учёта регистра.
2. Фильтр по категории.
3. Фильтр “только в наличии”.
4. Сортировка по цене или рейтингу.
5. Повторный рендер каталога после каждого применения фильтров.
6. Empty-state, если ничего не найдено.

### Обязательная логика формы

1. Чтение данных формы:
   - через `form.elements`;
   - и отдельно через `FormData`.
2. `event.preventDefault()` на `submit`.
3. Валидация и вывод ошибок рядом с полями.
4. Очистка ошибок на `input`.
5. Точечная проверка поля на `focusout`.
6. После успешной отправки:
   - показать сообщение об успехе;
   - очистить форму;
   - сохранить корзину как есть.