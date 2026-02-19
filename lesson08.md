# Лекция 8. DOM: как JavaScript управляет страницей

![](./images/DOM.png)

## Вступление

После ООП мы научились строить систему из сущностей: классы, объекты, методы, правила.  
Но пока всё это живёт *“внутри кода”* и видно только в `console.log`.

Следующий шаг логичный: научиться показывать результат пользователю.

В браузере JavaScript может не только считать и хранить данные - он может **управлять страницей**:
- находить элементы,
- менять текст и разметку,
- добавлять классы,
- создавать новые блоки и вставлять их в документ.

Этим и занимается **DOM**.

## Что такое DOM и как браузер его строит

![](./images/DOM-Structure.png)

`DOM (Document Object Model)` - это объектная модель документа.  
То есть браузер берёт ваш HTML и превращает его **в дерево объектов**, с которыми можно работать из JavaScript.

**Важно:** `DOM` - это **не строка HTML** и не “файл”, который лежит где-то отдельно.  
DOM - это **живая структура в памяти браузера**. Вы меняете DOM - вы меняете то, что видит пользователь на странице.

### Как из HTML появляется DOM-дерево

Допустим, у нас есть такой HTML:

```html
<body>
  <h1>Shop</h1>
  <ul>
    <li>Milk</li>
    <li>Bread</li>
  </ul>
</body>
```

Браузер превратит это в дерево:

```text
Document
└── body
    ├── h1
    └── ul
        ├── li
        └── li
```

То есть каждый тег становится узлом (`node`), и внутри узлов строится иерархия *“родитель → дети”*.


### Какие бывают узлы в DOM

В DOM есть разные типы узлов:
- **Element** - это узел, который соответствует HTML-тегу. Например, `<h1>`, `<ul>`, `<li>`.
- **Text** - это узел, который содержит текст внутри элемента. Например, в `<h1>Shop</h1>` текстовый узел - это “Shop”.
- **Comment** - это узел, который соответствует HTML-комментарию. Например, `<!-- This is a comment -->`.

На практике чаще всего мы работаем с элементами (`Element`), потому что именно они позволяют нам менять структуру страницы и её внешний вид.

### Главный вход в DOM - `document`

В JavaScript у нас есть глобальный объект `document`, который является **главным входом в DOM**. Через `document` мы можем находить элементы, создавать новые узлы и управлять всем деревом.

Простейшие примеры:

```javascript
console.log(document); // Показывает весь DOM
console.log(document.body); // Показывает элемент <body>
console.log(document.body.children); // Показывает коллекцию дочерних элементов <body>
```

## Момент выполнения скрипта и доступность элементов

**DOM** - это структура, которую браузер строит из `HTML`. И важный момент: **DOM появляется не мгновенно**. Браузер читает HTML сверху вниз. Если JavaScript выполнится слишком рано - нужных элементов ещё не будет в `DOM`. И тогда любая попытка *“достать элемент и поменять его”* заканчивается тем, что мы работаем с `null`.

### Почему порядок загрузки важен

Рассмотрим на примере такой структуры:

```html
<head>
  <script src="index.js"></script>
</head>
<body>
  <h1>Shop</h1>
</body>
```

Здесь скрипт подключён в `<head>`, то есть он выполняется **до того, как браузер построит DOM-дерево для `<body>`**. Поэтому если в `index.js` мы попробуем обратиться к `<h1>`, мы получим `null`, потому что его ещё нет.

### Самый простой способ: подключать скрипт в конце body

```javascript
<body>
  <h1>Shop</h1>

  <script src="index.js"></script>
</body>
```

Теперь браузер сначала построит все элементы страницы, а потом выполнит JavaScript. То есть `DOM` уже готов, и с ним можно работать.

### Современный способ: атрибут `defer`

Подключать скрипт внизу удобно, но не всегда. Поэтому в реальных проектах чаще используют `defer`:

```html
<head>
  <script src="index.js" defer></script>
</head>
```

`defer` означает:
- скрипт скачивается параллельно с `HTML`,
- но выполняется только после того, как `DOM` построен.

Это один из самых безопасных вариантов подключения.

## Поиск элементов в DOM

![](./images/search_in_dom.jpeg)

Чтобы управлять страницей, нам сначала нужно получить ссылку на элемент из `DOM`. То есть найти его в дереве и сохранить в переменную. В `JavaScript` для этого есть разные способы. Мы начнём с самых популярных.

---

### `getElementById` - поиск по id

Самый простой и быстрый способ найти элемент - по `id`.

Давайте добавим `id` к нашему заголовку:

```html
<body>
  <h1 id="main-title">Shop</h1>
</body>
```

Теперь в JavaScript мы можем найти этот элемент так:

```javascript
const title = document.getElementById('main-title');
console.log(title); // Покажет элемент <h1 id="main-title">Shop</h1>
console.log(title.textContent); // Покажет текст "Shop"
```

`id` на странице должен быть **уникальным**, то есть не должно быть двух элементов с одинаковым `id`. Поэтому `getElementById` всегда возвращает **один элемент** или `null`, если такого `id` нет.


### `querySelector` - поиск по CSS-селектору

`querySelector` - это более универсальный способ найти элемент. Он позволяет использовать **CSS-селекторы** для поиска.

Например, если у нас есть такой HTML:

```html
<div class="card"></div>
<div class="card"></div>
```

Мы можем найти первый элемент с классом `card` так:

```javascript
const card = document.querySelector('.card');
console.log(card); // Покажет первый элемент <div class="card"></div>
```

`querySelector` возвращает **первый элемент**, который соответствует селектору, или `null`, если такого элемента нет.

Вы можете искать так же, как в CSS:

```javascript
document.querySelector("h1");          // по тегу
document.querySelector(".menu-item");  // по классу
document.querySelector("#title");      // по id
document.querySelector("ul li");       // вложенность
```

**Важно**: `querySelector` всегда возвращает только один элемент (первый подходящий).


### `querySelectorAll` - поиск всех элементов по селектору

Если нам нужно найти **все элементы**, которые соответствуют селектору, мы используем `querySelectorAll`:

```html
<li class="item">Milk</li>
<li class="item">Bread</li>
<li class="item">Eggs</li>
```

```javascript
const items = document.querySelectorAll('.item');
console.log(items); // Покажет NodeList из всех элементов с классом "item"
console.log(items.length); // Покажет количество найденных элементов (3)
```

`querySelectorAll` возвращает **NodeList** - это коллекция, которая похожа на массив, но не совсем. Она содержит все элементы, которые соответствуют селектору. Поэтому мы не можем делать так:

```javascript
const items = document.querySelectorAll('.item');
items.textContent = "New Text"; // Это не сработает, потому что items - это коллекция, а не один элемент
```

Вместо этого, если мы хотим изменить текст всех элементов, нам нужно пройтись по коллекции:

```javascript
const items = document.querySelectorAll('.item');
items.forEach(item => {
  item.textContent = "New Text";
});
```

Так же можно использовать цикл `for`:

```javascript
const items = document.querySelectorAll('.item');
for (let i = 0; i < items.length; i++) {
  items[i].textContent = "New Text";
}
```

### `getElementsByClassName` и `getElementsByTagName` - старые методы поиска

Раньше, до появления `querySelector`, были другие методы для поиска элементов:
- `getElementsByClassName` - возвращает коллекцию всех элементов с заданным классом.
- `getElementsByTagName` - возвращает коллекцию всех элементов с заданным тегом.

Например:

```html
<div class="card"></div>
<div class="card"></div>
```
```javascript
const cards = document.getElementsByClassName('card');
console.log(cards); // Покажет HTMLCollection из всех элементов с классом "card"
const divs = document.getElementsByTagName('div');
console.log(divs); // Покажет HTMLCollection из всех элементов <div> 
```

Эти методы возвращают **HTMLCollection** - это тоже коллекция, похожая на массив, но не совсем. Она обновляется автоматически при изменении DOM. Поэтому если мы добавим новый элемент с классом `card`, он сразу появится в `cards`:

```javascript
const cards = document.getElementsByClassName('card');
console.log(cards.length); // Покажет 2
```

```html
<div class="card"></div>
<div class="card"></div>
<div class="card"></div> <!-- Новый элемент -->
```

```javascript
console.log(cards.length); // Теперь покажет 3, потому что коллекция обновилась
```

Важно: эти методы **не удалены** и продолжают работать. Просто в современных проектах чаще используют `querySelector` и `querySelectorAll`, потому что они более гибкие и поддерживают сложные селекторы.

### `NodeList` vs `HTMLCollection`: в чём практическая разница

Когда вы пишете `querySelectorAll`, вы получаете `NodeList` (статический список).  
Когда пишете `getElementsByClassName` / `getElementsByTagName`, вы получаете `HTMLCollection` (живую коллекцию).

- **NodeList (статический)**: не обновляется сам после изменений DOM.
- **HTMLCollection (живая)**: обновляется автоматически, если DOM изменился.

Пример:

```html
<ul id="list">
  <li class="item">Milk</li>
  <li class="item">Bread</li>
</ul>
```

```javascript
const staticItems = document.querySelectorAll(".item"); // NodeList
const liveItems = document.getElementsByClassName("item"); // HTMLCollection
const list = document.getElementById("list");

console.log(staticItems.length); // 2
console.log(liveItems.length); // 2

list.insertAdjacentHTML("beforeend", '<li class="item">Eggs</li>');

console.log(staticItems.length); // 2 (не изменился)
console.log(liveItems.length); // 3 (обновился автоматически)
```

**Мини-правило:** для большинства задач новичкам проще использовать `querySelectorAll`, потому что поведение у него более предсказуемое.


### Проверка наличия элемента в DOM

Если селектор неправильный или элемента нет на странице, то `querySelector` вернёт `null`, а `querySelectorAll` - пустой `NodeList`. Поэтому всегда полезно проверять результат перед тем, как работать с элементом:

```javascript
const title = document.querySelector('#main-title');
if (title) {
  console.log(title.textContent); // Покажет текст, если элемент найден
}


const items = document.querySelectorAll('.item');
if (items.length > 0) {
  items.forEach(item => {
    console.log(item.textContent); // Покажет текст всех найденных элементов
  });
}
```

### Мини правило по поиску элементов в DOM:
- Если есть один элемент с **уникальным** `id` - используйте `getElementById`.
- Если нужно найти **один элемент** - используйте `querySelector` с подходящим селектором.
- Если нужно найти **все элементы** - используйте `querySelectorAll` и потом работайте с коллекцией.
- Если элемент не найден - всегда проверяйте результат, чтобы избежать ошибок при работе с `null` или пустой коллекцией.

### Навигация по DOM: как двигаться от найденного элемента

Поиск селектором - это только половина работы. Часто вы нашли один элемент и хотите перейти к соседу, родителю или вложенному узлу.

Для этого используют навигационные свойства:

- `parentElement` - родительский элемент
- `children` - дочерние элементы
- `nextElementSibling` - следующий соседний элемент
- `previousElementSibling` - предыдущий соседний элемент
- `closest(selector)` - ближайший родитель (или сам элемент), который подходит под селектор

Пример:

```html
<ul id="products">
  <li class="item" data-id="1">Milk</li>
  <li class="item is-active" data-id="2">Bread</li>
  <li class="item" data-id="3">Eggs</li>
</ul>
```

```javascript
const activeItem = document.querySelector(".item.is-active");

console.log(activeItem.parentElement.id); // "products"
console.log(activeItem.previousElementSibling?.dataset.id); // "1"
console.log(activeItem.nextElementSibling?.dataset.id); // "3"
console.log(activeItem.closest("#products")); // <ul id="products">...</ul>
```

## Изменение содержимого: `textContent` и `innerHTML`

![](./images/innerText-vs-innerHTML.webp)

Мы нашли элемент, теперь хотим изменить его содержимое. Для этого есть два основных способа: `textContent` и `innerHTML`.

Давайте разбираться в чем разница.

### `textContent` - изменение текста

`textContent` - это свойство, которое позволяет получить или установить **текстовое содержимое** элемента. Если внутри строки будет `<b>...</b>`, браузер не превратит это в тег - он покажет это как текст.

```html
<h1 id="main-title">Shop</h1>
```

```javascript
const title = document.getElementById('main-title');
console.log(title.textContent); // Покажет "Shop"
title.textContent = "New Shop"; // Изменит текст на "New Shop"
console.log(title.textContent); // Покажет "New Shop"
```

Если мы сделаем так:

```javascript
title.textContent = "<b>New Shop</b>";
```
То на странице будет отображаться именно `<b>New Shop</b>`, а не жирный текст. Это безопасный способ изменить текст, потому что он не позволяет вставить HTML-код.

### `innerText` vs `textContent`: важная разница

В браузере есть ещё одно похожее свойство - `innerText`.  
На старте чаще используют `textContent`, но разницу знать обязательно.

- `textContent` - работает с “сырым” текстом узла (включая скрытый текст).
- `innerText` - возвращает только текст, который реально виден пользователю на странице.

Пример:

```html
<p id="msg">Hello <span style="display: none">hidden</span> world</p>
```

```javascript
const msg = document.getElementById("msg");

console.log(msg.textContent); // "Hello hidden world"
console.log(msg.innerText); // "Hello world"
```

**Мини-правило:** если нужно просто читать/писать текст в DOM-узле - используйте `textContent`. `innerText` оставляйте для задач, где важен именно “видимый” текст.

### `innerHTML` - изменение разметки

`innerHTML` - это свойство, которое позволяет получить или установить **HTML-содержимое** элемента. Если внутри строки будет `<b>...</b>`, браузер превратит это в тег и отобразит жирный текст.

```html
<h1 id="main-title">Shop</h1>
```

```javascript
const title = document.getElementById('main-title');
console.log(title.innerHTML); // Покажет "Shop"
title.innerHTML = "<b>New Shop</b>"; // Изменит текст на "New Shop" и сделает его жирным
console.log(title.innerHTML); // Покажет "<b>New Shop</b>"
```

На странице будет отображаться **New Shop** жирным шрифтом, потому что браузер интерпретирует строку как HTML.

### Когда использовать `textContent`, а когда `innerHTML`
- Если вам нужно просто изменить текст и не вставлять HTML - используйте `textContent`. Это безопаснее, потому что он не позволяет вставить вредоносный код.
- Если вам нужно вставить HTML-разметку - используйте `innerHTML`. Но будьте осторожны, потому что если строка содержит пользовательский ввод, это может привести к уязвимостям типа XSS (Cross-Site Scripting).

### Важный момент про `innerHTML`

`innerHTML` перезаписывает содержимое целиком.

```javascript
const box = document.querySelector(".box");

box.innerHTML = "<p>One</p>";
box.innerHTML = "<p>Two</p>";
```

В итоге останется только `Two`, потому что второй вызов полностью заменил первый.

Если хотите добавлять, а не заменять - либо конкатенация:

```javascript
box.innerHTML += "<p>Two</p>"; // Добавит "Two" после "One"
```
Либо использовать другие методы для создания и вставки элементов, о которых мы поговорим позже.

## Атрибуты, data-атрибуты, классы и стили

Мы уже умеем находить элементы и менять текст/разметку. Следующий уровень - управлять тем, *как элемент выглядит и как он “настроен”*:

- атрибуты (`src`, `href`, `type`, `disabled`…)
- `data-*` атрибуты (данные прямо в HTML)
- классы (`classList`)
- inline-стили (`style`)

---

## Атрибуты и `data-*`: как “передавать данные” из HTML в JavaScript (и обратно)

С DOM есть одна важная идея:

**HTML - это разметка. JavaScript - это логика.** Но иногда им нужно “договориться” и обменяться данными.

И самый простой способ - **атрибуты**.

- обычные атрибуты (`id`, `href`, `src`, `type`) описывают поведение/настройки элемента
- `data-*` атрибуты - это **наши собственные данные**, которые мы добавляем для JavaScript

---

### Зачем вообще нужны атрибуты, если есть переменные в JS?

Потому что переменные живут в коде, а DOM - это то, что реально находится на странице.

Атрибуты дают нам **привязку данных к конкретному элементу**.

Например:
- у кнопки есть язык `data-lang="en"`
- у карточки есть `data-id="12"`
- у блока переводов есть `data-i18n="title"`

То есть элемент сам “несёт” информацию, и JavaScript может её считать.

---

### Базовые методы работы с атрибутами

Кроме `dataset`, вам часто нужны обычные операции с атрибутами:

- `getAttribute(name)` - получить значение
- `setAttribute(name, value)` - установить/изменить
- `hasAttribute(name)` - проверить наличие
- `removeAttribute(name)` - удалить атрибут

Пример:

```html
<a id="site-link" href="https://example.com" target="_blank">Open site</a>
```

```javascript
const link = document.getElementById("site-link");

console.log(link.getAttribute("href")); // "https://example.com"
console.log(link.hasAttribute("target")); // true

link.setAttribute("href", "https://developer.mozilla.org");
link.setAttribute("rel", "noopener noreferrer");

link.removeAttribute("target");
console.log(link.hasAttribute("target")); // false
```

Для булевых атрибутов (например, `disabled`, `checked`) тоже работает простая логика:

```javascript
const input = document.querySelector("input[type='text']");
input.setAttribute("disabled", "");
input.removeAttribute("disabled");
```

### Пример: переключатель языка через `data-lang`

Разметка:

```html
<button class="lang-btn" data-lang="en">EN</button>
<button class="lang-btn" data-lang="uk">UK</button>
<button class="lang-btn" data-lang="cs">CS</button>
```

JavaScript (пока без событий - просто идея):

```javascript
const buttons = document.querySelectorAll(".lang-btn");

buttons.forEach(btn => {
  console.log(btn.dataset.lang); // "en", "uk", "cs"
});
```
Здесь `data-lang` - это чистые данные, которые мы положили в `HTML`, чтобы JS мог их достать.


### data-* и dataset - это способ хранить свои данные прямо в разметке.

`data-*` атрибуты читаются через `dataset`.

```html
<div class="box" data-lang="en" data-theme="dark"></div>
```

```javascript
const box = document.querySelector('.box');
console.log(box.dataset.lang); // "en"
console.log(box.dataset.theme); // "dark"
```

**Важно:** `dataset` - это объект, который содержит все `data-*` атрибуты элемента. Имена атрибутов в `dataset` преобразуются из `kebab-case` в `camelCase`.

Например, `data-user-id` будет доступен как `dataset.userId`.

Вы можете не только читать `dataset`, но и записывать:

```javascript
const box = document.querySelector(".box");

box.dataset.lang = "ru"; // станет data-lang="ru"
box.dataset.userId = "42"; // станет data-user-id="42"

console.log(box.getAttribute("data-user-id")); // "42"
```

#### Переводы с помощью `data-i18n`

Когда проект становится большим, тексты уже нельзя хранить “как попало”. Обычно делают так:

1. В HTML ставят ключ, что именно нужно перевести
2. В JS есть словарь переводов
3. JS находит элементы с ключами и подставляет текст

Пример небольшой страницы:

```html
<h1 data-i18n="title"></h1>
<p data-i18n="subtitle"></p>
<button data-i18n="buy"></button>
```

Словарь переводов:

```javascript
const translations = {
  en: {
    title: "Welcome to our shop",
    subtitle: "We have the best products",
    buy: "Buy Now"
  },
  ru: {
    title: "Добро пожаловать в наш магазин",
    subtitle: "У нас лучшие товары",
    buy: "Купить"
  }
};
```

Теперь главное: DOM содержит ключи, а JS подставляет значения.

```javascript
const lang = 'ru'; // допустим, выбрали русский
const elements = document.querySelectorAll('[data-i18n]');
elements.forEach(el => {
  const key = el.dataset.i18n; // получаем ключ из data-i18n - "title", "subtitle", "buy"
  el.textContent = translations[lang][key]; // подставляем перевод
});
```

Такой подход позволяет легко управлять переводами и поддерживать много языков, просто расширяя словарь. И при этом HTML остаётся чистым и понятным. 

> Главное не запутаться в ключах. По-хорошему, ключи должны быть логичными и отражать смысл текста, который они представляют. Ключи могут повторяться на разных элементах, потому что они описывают **тип текста**, а не конкретный элемент. Например, `data-i18n="title"` может быть у заголовка на главной странице и у заголовка в карточке товара - это нормально, потому что оба элемента отображают **заголовок**.

### Классы: Почему чаще меняют классы, а не стили

Когда мы хотим изменить внешний вид элемента, у нас есть два пути:
1. Изменить **стили** напрямую через `style`
2. Изменить **классы** элемента, а стили прописать в CSS

Оба способа работают. Но в реальных проектах почти всегда стараются менять классы, потому что так код получается чище и управляемее.

#### Вариант 1: меняем стили напрямую (style)

Это самый **“быстрый”** путь: прямо из `JavaScript` задаём `CSS`-свойства.

```html
<button id="buy-btn">Buy</button>
```

```javascript
const btn = document.getElementById('buy-btn');

btn.style.backgroundColor = "black";
btn.style.color = "white";
btn.style.padding = "12px 20px";
btn.style.borderRadius = "8px";
```

Это работает, но проблема видна сразу: стили расползаются в JS. Если нам нужно изменить внешний вид кнопки, мы должны прописать все стили прямо в коде. Это не только неудобно, но и нарушает принцип разделения ответственности: стили должны быть в `CSS`, а логика - в JS. При дальнейшем развитии проекта такой код становится трудно поддерживать и масштабировать.


#### Вариант 2: меняем классы (classList) и оформляем в CSS

Правильная идея - создать в `CSS` класс, который описывает нужный стиль, а в `JavaScript` просто добавлять или удалять этот класс.

```html
<button id="buy-btn">Buy</button>
```

```css
.btn {
  padding: 12px 20px;
  border-radius: 8px;
}

.btn--primary {
  background: black;
  color: white;
}
```

```javascript
const btn = document.getElementById('buy-btn');
btn.classList.add('btn', 'btn--primary');
```

Теперь все стили описаны в `CSS`, и если нам нужно изменить внешний вид кнопки, мы просто правим `CSS`, не трогая `JavaScript`. Это делает код более чистым, понятным и поддерживаемым.

#### classList: Основные методы

`classList` - это специальный объект, который позволяет управлять классами элемента. Вот его основные методы:

**1. add(className)** - добавляет класс к элементу

```javascript
btn.classList.add('btn--primary'); // Добавит класс "btn--primary"
```
**2. remove(className)** - удаляет класс из элемента

```javascript
btn.classList.remove('btn--primary'); // Удалит класс "btn--primary"
```

**3. toggle(className)** - если класса нет - добавляет, если есть - удаляет

```javascript
btn.classList.toggle('btn--primary'); // Если класса нет - добавит, если есть - удалит
```

**4. contains(className)** - проверяет, есть ли класс у элемента

```javascript
console.log(btn.classList.contains("btn--primary")); // true/false
```

**5. replace(oldClass, newClass)** - заменяет один класс на другой

```javascript
btn.classList.replace('btn--primary', 'btn--secondary'); // Заменит "btn--primary" на "btn--secondary"
```

Использование `classList` - это лучший способ управлять классами элемента, потому что он обеспечивает удобный и безопасный интерфейс для добавления, удаления и проверки классов. Это помогает поддерживать чистоту кода и разделение ответственности между стилями и логикой.

**Вывод**

В DOM-логике мы обычно делаем так:

- классы = состояния интерфейса (`is-active`, `is-hidden`, `is-error`, `is-loading`)
- `CSS` описывает, как выглядит каждое состояние
- `JS` только включает/выключает классы, не размазывая стили по коду

---
## Создание и добавление элементов

До этого момента мы только находили существующие элементы и меняли их. Но что если нам нужно создать новый элемент и добавить его на страницу? Для этого в DOM есть специальные методы.

Часто нам нужно создавать новый блок, список товаров или пользователей на основе данных, которые приходят с сервера. И для этого мы можем динамически создавать элементы и вставлять их в `DOM`.

### Главная идея

Элемент можно создать в памяти, настроить, и только потом вставить в DOM. То есть сначала он существует как объект в JS, но его ещё не видно пользователю. И только когда мы его вставляем в DOM, он появляется на странице.

**Шаг 1: document.createElement()**

```javascript
const p = document.createElement("p");
console.log(p); // <p></p>
```

Сейчас `<p>` - это просто объект в памяти, он ещё не на странице. Мы можем настроить его, например, добавить текст:

**Шаг 2: Настраиваем элемент**

Обычно настраивают элемент, добавляя текст, классы, атрибуты:

```javascript
const p = document.createElement("p");

p.textContent = "Hello from JS";
p.classList.add("message");
p.dataset.type = "info";

console.log(p);
// <p class="message" data-type="info">Hello from JS</p>
```

**Шаг 3: Вставляем элемент в DOM**

Теперь, когда элемент готов, мы можем вставить его в нужное место на странице. Для этого есть разные методы:

- `append` - добавляет элемент в конец родителя
- `prepend` - добавляет элемент в начало родителя
- `before` - вставляет элемент перед другим элементом
- `after` - вставляет элемент после другого элемента
- `insertBefore` - вставляет элемент перед указанным дочерним элементом

Но для этого нам нужен родительский элемент, в который мы будем вставлять новый элемент. Например:

```html
<div id="container"></div>
```

Теперь можно вставить новый элемент в этот контейнер:

```javascript
const container = document.getElementById("container");

const p = document.createElement("p");
p.textContent = "Hello from JS";

container.append(p);   // в конец контейнера
```

В итоге на странице появится:

```html
<div id="container">
  <p>Hello from JS</p>
</div>
```

Такой способ создания и добавления элементов позволяет нам динамически строить интерфейс на основе данных, которые приходят с сервера, или в ответ на действия пользователя. Мы можем создавать сложные структуры, настраивать их и вставлять в нужное место на странице, не трогая остальной код.


**Важно:** мы не можем вставить элемент, который уже находится в DOM, в другое место. Если мы попытаемся это сделать, элемент просто переместится, а не создастся новый. Поэтому если нам нужно создать несколько похожих элементов, мы должны вызывать `createElement` для каждого из них.

```javascript
const p = document.createElement("p");
p.textContent = "Move me";

container1.append(p);
container2.append(p); // теперь элемент будет только в container2
```

**Примеры создания и добавления нескольких элементов:**

```javascript
const container = document.getElementById("container");
for (let i = 1; i <= 3; i++) {
  const p = document.createElement("p");
  p.textContent = `Paragraph ${i}`;
  container.append(p);
}
```
В итоге на странице будет:

```html
<div id="container">
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
  <p>Paragraph 3</p>
</div>
```

**Примеры разных методов вставки:**

```javascript
const container = document.getElementById("container");
const p1 = document.createElement("p");
p1.textContent = "First Paragraph";
container.prepend(p1); // Вставляет в начало контейнера
const p2 = document.createElement("p");
p2.textContent = "Second Paragraph";
container.append(p2); // Вставляет в конец контейнера
const p3 = document.createElement("p");
p3.textContent = "Inserted Before";
p2.before(p3); // Вставляет перед p2
const p4 = document.createElement("p");
p4.textContent = "Inserted After";
p1.after(p4); // Вставляет после p1
```

В итоге на странице будет:

```html
<div id="container">
  <p>First Paragraph</p>
  <p>Inserted After</p>
  <p>Inserted Before</p>
  <p>Second Paragraph</p>
</div>
```


### Паттерн: “создать → настроить → вернуть” (функция-фабрика)

Когда элементов становится много, код быстро превращается в кашу, если всё писать в одном месте.
Поэтому часто делают так:

- есть функция, которая создаёт один элемент
- она принимает данные
- внутри настраивает DOM
- и возвращает готовый узел

Пример: хотим создавать “карточку товара”.

```javascript
function createProductCard(product) {
  const card = document.createElement("div");
  card.classList.add("product-card");
  card.dataset.id = product.id;

  const title = document.createElement("h3");
  title.textContent = product.title;

  const price = document.createElement("p");
  price.textContent = `${product.price} CZK`;
  price.classList.add("product-card__price");

  card.append(title, price);
  return card;
}
```

Здесь важно:
- функция принимает **данные** (объект `product`)
- внутри создаёт и настраивает элемент
- возвращает готовый узел, который можно вставить в DOM

Такой паттерн позволяет нам держать логику создания элемента в одном месте, а в остальном коде просто вызывать эту функцию и вставлять результат в нужное место. Это делает код более чистым, понятным и поддерживаемым.

Пример использования:

```javascript
const products = [
  { id: 1, title: "Milk", price: 20 },
  { id: 2, title: "Bread", price: 15 },
  { id: 3, title: "Eggs", price: 30 }
];
const container = document.getElementById("product-list");
products.forEach(product => {
  const card = createProductCard(product);
  container.append(card);
});
```

В итоге на странице будет список карточек товаров, созданных динамически на основе данных из массива `products`. И вся логика создания карточки сосредоточена в функции `createProductCard`, что делает код более организованным и удобным для поддержки.

**Важный момент: не забывайте “чистить контейнер” при повторном рендере**

Если вы будете запускать рендер ещё раз - элементы начнут дублироваться. Поэтому перед перерисовкой обычно очищают контейнер.

Самый простой вариант:

```javascript
container.innerHTML = ""; // Очищаем контейнер перед рендером
```

После чего можно снова отрендерить список.

### Мини-правило по стилю кода

Если у вас:

- один элемент → создаём и вставляем
- много элементов → делаем функцию `create...()` и отдельный рендер из массива

Так DOM-код остаётся читаемым и масштабируемым. Если же вы начнёте создавать много элементов прямо в теле функции, которая отвечает за рендер, код быстро превратится в кашу, и будет сложно понять, что происходит. Поэтому лучше придерживаться этого простого правила: если элемент один - создаём и вставляем сразу, если много - делаем функцию-фабрику для создания элемента и отдельный рендер для вставки всех элементов.

---
## Удаление и замена элементов: remove() и replaceWith()

Иногда элемент нужно не *“поправить”*, а убрать полностью или заменить другим. Для этого в DOM есть два простых метода:

- `remove()` - удаляет элемент из DOM
- `replaceWith(newNode)` - заменяет элемент на другой

### Удаление элемента: remove()

`remove()` удаляет элемент из DOM. Допустим, у нас есть блок:

```html
<div class="banner">Sale -50%</div>
```

Находим его и удаляем:

```javascript
const banner = document.querySelector(".banner");

if (banner) {
  banner.remove();
}
```

После выполнения этого кода элемент `<div class="banner">Sale -50%</div>` будет удалён со страницы. Если мы попытаемся обратиться к `banner` после удаления, он всё равно будет существовать в памяти, но уже не будет частью DOM, и любые попытки взаимодействовать с ним через DOM будут неэффективными.

### Замена элемента: replaceWith()

`replaceWith(newNode)` заменяет один элемент другим. 

Например, у нас есть такой HTML:

```html
<p id="status">Loading...</p>
```

Мы хотим заменить его на “готовый” блок:

```javascript
const status = document.getElementById("status");

if (status) {
  const done = document.createElement("p");
  done.id = "status";
  done.classList.add("status--done");
  done.textContent = "Done";

  status.replaceWith(done);
}
```

**Важно:** старый элемент полностью исчезает, вместо него в DOM появляется новый.

Мини-правило

- если нужно убрать элемент со страницы - `remove()`
- если проще “пересобрать” элемент заново, чем менять 10 строк внутри - `replaceWith()`


## Рендер через строку: генераторы HTML и вставка через innerHTML

Иногда интерфейс нужно собрать быстро: есть данные, и нужно *“выложить”* кусок разметки на страницу. В таких случаях часто используют подход рендера через строку.

Идея очень простая:

- мы генерируем HTML как строку
- вставляем эту строку в DOM через innerHTML

Это не *“единственно правильный способ”*, но он очень распространён: быстро собрать шаблон, вывести список, показать блоки.

### Шаг 1. Генератор атрибутов

Нам удобно передавать атрибуты как объект:

```javascript
{ class: "red", style: "color:blue;", "data-id": "12" }
```

И превращать его в строку:

```javascript
class="red" style="color:blue;" data-id="12"
```

Для этого можно написать функцию:

```javascript
const propsGenerator = (props = {}) =>
  Object.entries(props)
    .map(([key, value]) => `${key}="${value}"`)
    .join(" ");
```

### Важный момент по безопасности в шаблонах-строках

Если вы вставляете в HTML пользовательский ввод (форма, комментарий, данные от API без очистки), нельзя подставлять строку “как есть” в `innerHTML`.

Базовый подход - экранировать специальные символы:

```javascript
function escapeHTML(value = "") {
  return String(value)
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#39;");
}
```

И использовать так:

```javascript
const safeTitle = escapeHTML(product.title);
const html = `<h3>${safeTitle}</h3>`;
```

Это не “полная защита от всех угроз”, но для базовой лекции важно сразу формировать правильную привычку: всё, что пришло от пользователя, считаем потенциально небезопасным.

### Шаг 2. Генераторы элементов

Теперь мы пишем небольшие функции, которые возвращают HTML-строку.
```javascript
const titleGenerator = ({ text, h = 1, props = {} }) =>
  `<h${h} ${propsGenerator(props)}>${text}</h${h}>`;

const divGenerator = ({ text, props = {} }) =>
  `<div ${propsGenerator(props)}>${text}</div>`;
```

### Шаг 3. “Таблица генераторов”

Можно сделать объект, который будет хранить все генераторы:

```javascript
const state = {
  h: titleGenerator,
  div: divGenerator,
};
```

**Пример использования:**

```javascript
const h = state.h({
  text: "text1",
  h: 3,
  props: { class: "red", style: "color:blue;", "data-lang": "en" },
});

const div = state.div({
  text: "Text2",
  props: { class: "block" },
});

console.log(h);
console.log(div);
```

На выходе мы получаем строки:

```html
<h3 class="red" style="color:blue;" data-lang="en">text1</h3>
<div class="block">Text2</div>
```

### Шаг 4. Вставляем строку в DOM через innerHTML

Теперь самое главное: строка сама по себе не отображается. Её нужно вставить в документ.

Допустим, есть контейнер:

```html
<div id="container"></div>
```

Мы можем вставить нашу строку так:

```javascript
const container = document.getElementById("container");
container.innerHTML = h + div; // Вставляем обе строки в контейнер
```

В итоге на странице будет:

```html
<div id="container">
  <h3 class="red" style="color:blue;" data-lang="en">text1</h3>
  <div class="block">Text2</div>
</div>
```

После этого элементы реально появляются на странице.

**Важно:** при вставке через `innerHTML` старые элементы внутри контейнера будут удалены, а новые - созданы заново. Поэтому если у вас есть какие-то обработчики событий или ссылки на старые элементы, они перестанут работать после замены `innerHTML`. Поэтому этот способ подходит для случаев, когда нужно быстро отрендерить новый контент, и нет необходимости сохранять состояние старых элементов. Если же вам нужно сохранить обработчики событий или состояние, лучше использовать методы создания и добавления элементов, о которых мы говорили ранее.

## Типовой паттерн рендера: список из массива объектов

До этого мы работали с DOM *“вручную”*: создали один элемент, вставили, поменяли текст.
Но реальная работа почти всегда выглядит иначе:

- есть данные (массив объектов),
- есть контейнер на странице,
- нужно отрисовать всё в DOM.

И важно понимать одну вещь:

**DOM - это отображение данных. Если данные поменялись - мы чаще всего перерисовываем DOM заново.**

### Данные → рендер

Возьмём простой пример: список товаров.

```javascript
const products = [
  { id: 1, title: "Milk", price: 20 },
  { id: 2, title: "Bread", price: 15 },
  { id: 3, title: "Eggs", price: 30 }
];
```

И контейнер в HTML:

```html
<div id="product-list"></div>
```

### 1 Рендер через createElement (DOM-узлы)

Здесь мы создаём элементы как объекты, настраиваем и вставляем.

```javascript
function renderProducts(products, container) {
  // 1) очищаем контейнер перед рендером
  container.innerHTML = "";

  // 2) пустое состояние
  if (products.length === 0) {
    const empty = document.createElement("p");
    empty.textContent = "Нет товаров";
    empty.classList.add("empty");
    container.append(empty);
    return;
  }

  // 3) рендерим список
  products.forEach((product) => {
    const card = document.createElement("div");
    card.classList.add("product-card");
    card.dataset.id = product.id;

    const title = document.createElement("h3");
    title.textContent = product.title;

    const price = document.createElement("p");
    price.textContent = `${product.price} CZK`;
    price.classList.add("product-card__price");

    card.append(title, price);
    container.append(card);
  });
}

const container = document.getElementById("product-list");
renderProducts(products, container);
```

Что здесь важно:

- контейнер всегда очищается перед повторным рендером
- есть *“пустое состояние”*
- данные не смешиваются с DOM-логикой: есть отдельная функция `renderProducts`


**Почему всегда нужен clear перед повторным рендером**

Если не очищать контейнер, то при повторном вызове функции карточки будут дублироваться.

```javascript
renderProducts(products, container); // первый рендер
renderProducts(products, container); // второй рендер - карточки будут повторяться
```

### 2 Рендер через строку (innerHTML)

Иногда удобнее собирать интерфейс не через createElement, а через HTML-строку.

Идея простая:

- данные → превращаем в строку HTML
- один раз вставляем в контейнер через innerHTML

Это быстрый способ отрисовать список, особенно когда разметка простая.

Данные

```javascript
const products = [
  { id: 1, title: "Milk", price: 20 },
  { id: 2, title: "Bread", price: 15 },
  { id: 3, title: "Eggs", price: 30 }
];
```

Контейнер:

```html
<div id="product-list"></div>
```

**1) Генератор атрибутов (props → строка)**

Мы хотим передавать атрибуты объектом:

```javascript
{ class: "product-card", "data-id": "12" }
```

И превращать в строку:

```html
class="product-card" data-id="1"
```

```javascript
const propsGenerator = (props = {}) =>
  Object.entries(props)
    .map(([key, value]) => `${key}="${value}"`)
    .join(" ");
```

**2) Генераторы элементов (данные → HTML-строка)**

```javascript
const productCardGenerator = (product) => `
  <div ${propsGenerator({ class: "product-card", "data-id": product.id })}>
    <h3 ${propsGenerator({ class: "product-card__title" })}>${product.title}</h3>
    <p ${propsGenerator({ class: "product-card__price" })}>${product.price} CZK</p>
  </div>
`;
```

**3) Рендер списка (массив → innerHTML)**

```javascript
function renderProductsHTML(products, container) {
  // 1) пустое состояние
  if (products.length === 0) {
    container.innerHTML = `<p ${propsGenerator({ class: "empty" })}>Нет товаров</p>`;
    return;
  }

  // 2) собираем одну строку из массива
  const html = products.map(productCardGenerator).join("");

  // 3) вставляем одним действием
  container.innerHTML = html;
}

const container = document.getElementById("product-list");
renderProductsHTML(products, container);
```

### Почему .join("") обязателен

`map()` возвращает массив строк. Если вы вставите его напрямую, получится запятая между элементами:

```javascript
const html = products.map(productCardGenerator);
container.innerHTML = html;
```

Поэтому всегда:

```javascript
const html = products.map(productCardGenerator).join("");
container.innerHTML = html;
```

### Мини-вывод: когда строка - ок, а когда лучше DOM-узлы

- `innerHTML` удобно, когда нужно быстро отрисовать блок и у вас нет сложной логики.
- `createElement` удобнее, когда вы:

  - собираете большие структуры по шагам
  - хотите точнее контролировать элементы
  - не хотите пересоздавать весь контейнер целиком при обновлении


## Частые ошибки новичков при работе с DOM

Когда `DOM` только начинается, почти все ошибки одинаковые. Важно не *“зазубрить”*, а понимать почему так происходит.

### Ошибка 1: неправильный селектор и null

Самая частая ситуация:

```javascript
const title = document.querySelector(".main-title"); // а в HTML id="main-title"
title.textContent = "Hello"; // TypeError
```

Проблема не в `textContent`. Проблема в том, что `title === null`.

Мини-правило:

```javascript
const title = document.querySelector("#main-title");
if (title) {
  title.textContent = "Hello";
}
```
### Ошибка 2: коллекция - это не элемент

`querySelectorAll` возвращает коллекцию (NodeList), а не один элемент.

```javascript
const items = document.querySelectorAll(".item");
items.textContent = "X"; // не работает
```

Правильно - пройтись циклом:

```javascript
items.forEach((item) => {
  item.textContent = "X";
});
```

### Ошибка 3: перезапись innerHTML внутри цикла

Такое выглядит логично, но работает плохо:

```javascript
products.forEach((p) => {
  container.innerHTML += `<div>${p.title}</div>`;
});
```

Проблема в том, что `innerHTML +=` - это не просто добавление строки. Это значит: “возьми текущий HTML, добавь к нему строку, и вставь обратно”. Поэтому при каждом вызове происходит полная перерисовка контейнера, и если элементов много - это будет работать очень медленно.

### Ошибка 4: забыли очистить контейнер перед повторным рендером

```javascript
renderProducts(products, container);
renderProducts(products, container); // дубли
```

Мини-правило: рендер всегда начинается с очистки

```javascript
container.innerHTML = "";
```

### Ошибка 5: слишком много запросов к DOM

Новички часто пишут так:

```javascript
document.querySelector("#product-list").innerHTML = "";
document.querySelector("#product-list").innerHTML = "<p>...</p>";
```

Каждый `querySelector` - это поиск. Лучше:

```javascript
const container = document.querySelector("#product-list");
container.innerHTML = "";
container.innerHTML = "<p>...</p>";
```

Мини-правило: один раз нашли → сохранили в переменную → работаем

## Заключение

`DOM` - это мощный инструмент для создания динамических интерфейсов. Но важно понимать, как он работает, чтобы использовать его эффективно и избегать распространённых ошибок. В этом уроке мы рассмотрели основные методы работы с DOM: поиск элементов, изменение содержимого, управление атрибутами и классами, создание новых элементов и рендер через строки. Теперь у вас есть базовые знания, чтобы начать создавать свои собственные динамические страницы и приложения!

## Чек-лист после лекции

После этой лекции вы должны уметь:

- объяснить, что такое DOM и почему `document` - главный вход в него
- корректно выбирать метод поиска: `getElementById`, `querySelector`, `querySelectorAll`
- понимать разницу между `NodeList` и `HTMLCollection` (static vs live)
- менять контент через `textContent`, `innerText`, `innerHTML` и понимать разницу
- работать с атрибутами через `getAttribute` / `setAttribute` / `removeAttribute`
- использовать `data-*` и `dataset` для связи HTML и JS
- управлять внешним видом через `classList`, а не “размазывать” стили в JS
- создавать, вставлять, удалять и заменять элементы (`createElement`, `append`, `remove`, `replaceWith`)
- делать базовый рендер списка из массива и избегать частых ошибок при повторном рендере

## Практика

1. Создайте HTML-блок с заголовком, кнопкой и списком из 3 элементов. Найдите их через `querySelector`/`querySelectorAll` и выведите в консоль:
   - текст заголовка;
   - количество элементов списка.

2. Добавьте кнопке атрибут `data-action="buy"`. В JS:
   - прочитайте его через `dataset`;
   - поменяйте значение на `checkout`;
   - проверьте результат в консоли.

3. Сделайте блок:
   ```html
   <p id="text">Hello <span style="display:none">hidden</span> world</p>
   ```
   Выведите в консоль `textContent` и `innerText`. Объясните разницу своими словами.

4. Создайте контейнер `#products` и массив из 3 товаров. Отрисуйте карточки через `createElement`, добавив каждой карточке:
   - класс `product-card`;
   - `data-id`;
   - заголовок и цену.

5. Сделайте тот же рендер товаров, но через `innerHTML` + `map().join("")`.
   Сравните оба подхода: какой код вам проще читать и почему.

6. Создайте элемент `<div class="alert">Warning</div>`, вставьте на страницу, затем:
   - удалите его через `remove()`;
   - снова создайте и замените на `<div class="alert alert--ok">Done</div>` через `replaceWith()`.

7. Проверьте разницу `NodeList` и `HTMLCollection`:
   - получите элементы по классу двумя способами;
   - динамически добавьте ещё один элемент;
   - сравните длину коллекций до и после.

8. Найдите активный элемент списка `.item.is-active` и выведите:
   - его родителя (`parentElement`);
   - предыдущий и следующий элементы;
   - ближайший контейнер через `closest()`.

## Домашняя работа

1. Сделайте небольшой мини-интерфейс “Каталог товаров”:
   - контейнер `#product-list`;
   - массив минимум из 6 товаров (`id`, `title`, `price`, `inStock`).

2. Реализуйте функцию `createProductCard(product)` через `createElement`, где:
   - у карточки есть `data-id`;
   - название выводится в `<h3>`;
   - цена в `<p>`;
   - если `inStock === false`, добавляется класс `is-out`.

3. Реализуйте функцию `renderProducts(products, container)`:
   - очищает контейнер перед рендером;
   - если массив пустой, показывает “Нет товаров”;
   - иначе вставляет все карточки.

4. Добавьте вторую версию рендера `renderProductsHTML(products, container)` через `innerHTML` + `map().join("")`.
   Для `title` обязательно используйте `escapeHTML`.

5. Сделайте блок языка:
   ```html
   <button data-lang="en">EN</button>
   <button data-lang="ru">RU</button>
   ```
   И словарь переводов минимум для 3 текстов (`title`, `subtitle`, `buy`).
   По клику на кнопку (можно через `onclick`) подставляйте переводы по `data-i18n`.

6. Добавьте в итоговый проект небольшой отчёт в комментарии в конце JS-файла:
   - где использовали `textContent`, где `innerHTML` и почему;
   - где использовали `dataset`;
   - где применили `classList` вместо inline-стилей;
   - какие 2 ошибки из блока “Частые ошибки новичков” вы специально избежали.
