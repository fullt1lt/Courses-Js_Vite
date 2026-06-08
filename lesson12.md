# Лекция 12. HTTP, fetch и API: запросы, JSON и работа с данными

![](./images/API.avif)

## Введение

В прошлой лекции мы разобрали фундамент асинхронности:

- почему браузер не должен блокировать интерфейс;
- как работают `callback`, `Promise`, `async/await`;
- что такое `event loop`;
- как строить последовательные и параллельные шаги;
- как показывать `loading` и правильно обрабатывать ошибки.

Теперь пришло время применить этот фундамент к одной из самых важных задач во frontend:

> **получать и отправлять данные по сети**

Почти любое современное приложение делает именно это:

- загружает список товаров;
- получает профиль пользователя;
- отправляет форму;
- обновляет данные;
- удаляет запись;
- показывает состояние загрузки или ошибку.

То есть теперь мы переходим от *“асинхронности вообще”* к очень конкретной прикладной теме:

- как браузер общается с сервером;
- что такое `HTTP` и `API`;
- как использовать `fetch`;
- как читать `JSON`;
- как строить UI вокруг сетевых запросов.

## Кратко

Если собрать всю лекцию в несколько опорных мыслей, получится так:

- браузер не берёт данные “из воздуха” - он отправляет запросы на сервер;
- сервер отвечает через `HTTP`;
- `fetch()` возвращает `Promise`, поэтому с ним работают через `then` или `async/await`;
- объект `Response` - это ещё не сами данные;
- данные часто приходят в формате `JSON`;
- `fetch` не падает на `404` или `500` автоматически - это нужно проверять через `response.ok`;
- хороший frontend показывает пользователю `loading`, `success`, `empty`, `error`.

## Как браузер общается с сервером

Когда вы открываете сайт или нажимаете кнопку “Загрузить”, браузер не *“подсматривает”* данные где-то у себя внутри. Он общается с внешним сервером.

В этой модели есть две стороны:

- **клиент** - браузер, в котором работает ваш JavaScript;
- **сервер** - компьютер или сервис, который принимает запросы и возвращает ответы.

Простейшая схема:

```text
Browser -> request -> Server
Browser <- response <- Server
```

### Клиент и сервер

**Клиент**:

- показывает интерфейс пользователю;
- отправляет запросы;
- получает ответы;
- рендерит данные на страницу.

**Сервер**:

- принимает запрос;
- выполняет бизнес-логику;
- может обращаться к базе данных;
- возвращает результат.

### Почему это асинхронная операция

Сеть всегда требует времени:

- сервер может быть далеко;
- запрос может обрабатываться не мгновенно;
- сеть может быть медленной;
- ответ может быть большим.

Поэтому запрос к серверу нельзя делать как обычную мгновенную функцию. Именно поэтому работа с сетью во frontend всегда связана с асинхронностью.

## HTTP и HTTPS

Для общения клиента и сервера обычно используется протокол `HTTP`.

`HTTP` - это набор правил, по которым:

- клиент отправляет запрос;
- сервер отвечает;
- обе стороны понимают, что происходит.

`HTTPS` - это защищённая версия `HTTP`, где данные передаются по зашифрованному соединению.

### Из чего состоит HTTP-запрос

У запроса обычно есть:

- `URL` - адрес ресурса;
- `method` - что мы хотим сделать;
- `headers` - служебная информация;
- `body` - данные, которые мы отправляем.

### Из чего состоит HTTP-ответ

У ответа обычно есть:

- `status` - числовой код ответа;
- `headers` - служебная информация;
- `body` - сами данные.

### Основные HTTP-методы

| Метод | Что делает |
|------|------------|
| `GET` | Получить данные |
| `POST` | Создать запись или отправить данные |
| `PUT` | Полностью заменить ресурс |
| `PATCH` | Частично обновить ресурс |
| `DELETE` | Удалить ресурс |

### Основные статусы ответа

| Статус | Смысл |
|-------|-------|
| `200` | Всё успешно |
| `201` | Ресурс создан |
| `400` | Ошибка в запросе |
| `401` | Нужна авторизация |
| `403` | Доступ запрещён |
| `404` | Ресурс не найден |
| `500` | Ошибка на сервере |

> **Важно:** `404` и `500` - это не “сеть упала”, а нормальный ответ сервера с ошибочным статусом.

## Что такое API

`API (Application Programming Interface)` - это интерфейс, через который программы обмениваются данными.

Во frontend чаще всего под API понимают набор URL-адресов, куда можно:

- отправить запрос;
- получить данные;
- создать запись;
- обновить запись;
- удалить запись.

Например, если у нас есть сервер с постами, у него могут быть такие точки входа:

- `GET /posts`
- `GET /posts/1`
- `POST /posts`
- `PATCH /posts/1`
- `DELETE /posts/1`

То есть API - это по сути договор:

> “Вот по этим адресам и по этим правилам ты можешь получить или изменить данные.”

## `fetch()`: главный браузерный инструмент для запросов

Во frontend основным встроенным способом отправлять HTTP-запросы является `fetch()`.

### Что такое `fetch`

`fetch()`:

- отправляет запрос;
- возвращает `Promise`;
- после завершения даёт объект `Response`.

Пример:

```javascript
const responsePromise = fetch("https://jsonplaceholder.typicode.com/posts/1");

console.log(responsePromise); // Promise
```

То есть `fetch()` не возвращает данные мгновенно. Он запускает асинхронную операцию и отдаёт `Promise`, который завершится позже.

## Первый `GET`-запрос

Самый базовый пример:

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => {
    console.log(response);
  })
  .catch((error) => {
    console.log("Ошибка сети:", error);
  });
```

Здесь важно увидеть главное:

- `response` - это **не данные поста**;
- это объект ответа;
- данные нужно прочитать отдельно.

## Объект `Response`

Когда `fetch` завершается, он отдаёт объект `Response`.

У него есть важные свойства:

- `response.status`
- `response.ok`
- `response.headers`
- `response.url`

И важные методы:

- `response.text()`
- `response.json()`
- `response.blob()`

Пример:

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => {
    console.log("status:", response.status);
    console.log("ok:", response.ok);
  });
```

Если ответ успешный, обычно увидим:

- `status: 200`
- `ok: true`

## Почему `fetch` не падает на `404` и `500`

Это один из самых частых источников путаницы.

Посмотрите на код:

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/9999")
  .then((response) => {
    console.log(response.status);
  })
  .catch((error) => {
    console.log("Не попадём сюда на 404");
  });
```

Новички часто ждут, что `404` сразу пойдёт в `catch`. Но этого не происходит.

Почему:

- сеть сработала;
- сервер ответил;
- `fetch` получил ответ;
- значит для `fetch` это не “ошибка сети”.

То есть `404` - это не сбой транспорта, а просто ответ сервера.

### Как правильно обрабатывать это

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/9999")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`Ошибка сервера: ${response.status}`);
    }

    return response.json();
  })
  .then((data) => {
    console.log("Данные:", data);
  })
  .catch((error) => {
    console.log("Ошибка:", error.message);
  });
```

Здесь мы сами проверяем `response.ok` и вручную создаём ошибку через `throw`.

## `JSON`: в каком виде приходят данные

Веб-приложения очень часто обмениваются данными в формате `JSON`.

### Что такое `JSON`

`JSON (JavaScript Object Notation)` - это текстовый формат для передачи данных.

Важно разделять:

- **объект JavaScript**
- **JSON-строку**

Пример объекта:

```javascript
const user = {
  name: "Alex",
  age: 27,
};
```

Пример JSON-строки:

```javascript
const jsonString = '{"name":"Alex","age":27}';
```

Снаружи это похоже, но это не одно и то же:

- объект живёт в коде;
- JSON - это текст.

## `JSON.stringify()` и `JSON.parse()`

### Превратить объект в строку

```javascript
const user = {
  name: "Alex",
  age: 27,
};

const jsonString = JSON.stringify(user);
console.log(jsonString);
```

Это нужно, когда вы отправляете данные на сервер.

### Превратить JSON-строку обратно в объект

```javascript
const jsonString = '{"name":"Alex","age":27}';

const user = JSON.parse(jsonString);
console.log(user);
```

Это нужно, когда вы получили строку и хотите работать с данными как с объектом.

## Почему `response.json()` возвращает `Promise`

Многих удивляет, что `response.json()` тоже асинхронный.

Причина простая:

- сначала нужно прочитать тело ответа;
- потом распарсить текст в объект;
- это не считается мгновенной операцией.

Поэтому:

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => response.json())
  .then((data) => {
    console.log(data.title);
  });
```

`response.json()` возвращает `Promise`, а не готовый объект сразу.

## `GET`, `POST`, `PATCH`, `DELETE` на практике

Теперь посмотрим на основные действия, которые frontend делает с сервером.

### `GET`: получить данные

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }

    return response.json();
  })
  .then((post) => {
    console.log(post.title);
  })
  .catch((error) => {
    console.log(error.message);
  });
```

### `POST`: отправить данные

```javascript
const newPost = {
  title: "Новый пост",
  body: "Содержимое поста",
  userId: 1,
};

fetch("https://jsonplaceholder.typicode.com/posts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify(newPost),
})
  .then((response) => response.json())
  .then((data) => {
    console.log("Ответ сервера:", data);
  });
```

Здесь важно:

- указать `method`;
- добавить `headers`;
- превратить объект в строку через `JSON.stringify()`.

### `PATCH`: обновить часть данных

```javascript
const updatedPost = {
  title: "Обновлённый заголовок",
};

fetch("https://jsonplaceholder.typicode.com/posts/1", {
  method: "PATCH",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify(updatedPost),
})
  .then((response) => response.json())
  .then((data) => {
    console.log("Ответ сервера:", data);
  });
```

### `DELETE`: удалить данные

```javascript
fetch("https://jsonplaceholder.typicode.com/posts/1", {
  method: "DELETE",
})
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }

    console.log("Пост удалён");
  })
  .catch((error) => {
    console.log("Ошибка:", error.message);
  });
```

> **JSONPlaceholder** - учебный сервис. Он имитирует создание, изменение и удаление записей, но не хранит их по-настоящему навсегда.

## Универсальная функция `request()`

В реальных проектах редко пишут `fetch` “в лоб” в каждом месте.

Почти всегда повторяются одни и те же вещи:

- метод;
- заголовки;
- `JSON.stringify()`;
- проверка `response.ok`;
- чтение `response.json()`;
- обработка пустого ответа.

Поэтому удобно вынести это в одну функцию.

### Пример `request()`

```javascript
async function request(url, method = "GET", data = null) {
  const options = { method };

  if (data !== null) {
    options.headers = {
      "Content-Type": "application/json; charset=UTF-8",
    };
    options.body = JSON.stringify(data);
  }

  const response = await fetch(url, options);

  if (!response.ok) {
    throw new Error(`HTTP ошибка: ${response.status}`);
  }

  const contentLength = response.headers.get("content-length");

  if (response.status === 204 || contentLength === "0") {
    return null;
  }

  return response.json();
}
```

### Почему это удобно

Теперь в приложении вы пишете уже не детали сетевого уровня, а смысл:

```javascript
async function run() {
  try {
    const post = await request("https://jsonplaceholder.typicode.com/posts/1");
    console.log(post);
  } catch (error) {
    console.log(error.message);
  }
}
```

Так код становится чище и легче поддерживается.

## UI-состояния: `loading`, `success`, `empty`, `error`

Хороший frontend не просто отправляет запрос. Он ещё и честно показывает пользователю, что происходит.

Минимальный набор состояний:

1. **loading** - данные загружаются;
2. **success** - данные пришли и отображены;
3. **empty** - данные пришли, но их нет;
4. **error** - произошла ошибка.

### Почему это важно

Без этих состояний пользователь не понимает:

- запрос ещё идёт или всё сломалось;
- данные пустые или ещё не пришли;
- можно ли нажимать кнопку снова;
- что делать, если произошла ошибка.

## Мини-пример интерфейса

### Разметка

```html
<div id="status"></div>
<ul id="posts"></ul>

<button id="loadBtn">Загрузить посты</button>
```

### Функции для интерфейса

```javascript
const statusEl = document.getElementById("status");
const postsEl = document.getElementById("posts");

function setStatus(text) {
  statusEl.textContent = text;
}

function clearPosts() {
  postsEl.innerHTML = "";
}

function renderPosts(posts) {
  postsEl.innerHTML = posts
    .map((post) => `<li><strong>${post.title}</strong><br>${post.body}</li>`)
    .join("");
}
```

### Загрузка постов

```javascript
const API_URL = "https://jsonplaceholder.typicode.com/posts";
const loadBtn = document.getElementById("loadBtn");

async function loadPosts() {
  if (loadBtn.disabled) return;

  loadBtn.disabled = true;

  try {
    setStatus("Загрузка...");
    clearPosts();

    const posts = await request(API_URL);

    if (!posts || posts.length === 0) {
      setStatus("Постов нет");
      return;
    }

    setStatus("");
    renderPosts(posts.slice(0, 10));
  } catch (error) {
    setStatus(`Ошибка: ${error.message}`);
  } finally {
    loadBtn.disabled = false;
  }
}

loadBtn.addEventListener("click", loadPosts);
```

Здесь уже видно важный реальный паттерн:

- блокируем кнопку;
- показываем `loading`;
- ждём ответ;
- рендерим успех или ошибку;
- в `finally` восстанавливаем интерфейс.

## Query params: параметры в URL

Очень часто API позволяет не получать “всё подряд”, а запрашивать данные более точно:

- фильтровать;
- ограничивать количество;
- передавать номер страницы;
- сортировать.

Такие параметры передаются в URL после `?` и называются **query params**.

Пример:

```text
/posts?userId=1&_limit=5
```

### Пример руками

```javascript
fetch("https://jsonplaceholder.typicode.com/posts?userId=1&_limit=5")
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }

    return response.json();
  })
  .then((posts) => {
    console.log("Постов получено:", posts.length);
  });
```

### Безопасная сборка через `URLSearchParams`

```javascript
const params = new URLSearchParams({
  userId: 1,
  _limit: 5,
});

const url = `https://jsonplaceholder.typicode.com/posts?${params.toString()}`;

fetch(url)
  .then((response) => {
    if (!response.ok) {
      throw new Error(`HTTP ошибка: ${response.status}`);
    }

    return response.json();
  })
  .then((posts) => {
    console.log("URL:", url);
    console.log("Постов получено:", posts.length);
  });
```

Это лучше, чем собирать URL руками строкой.

## Частые ошибки новичков

### 1) Думать, что `fetch()` сразу возвращает данные

Нет. `fetch()` возвращает `Promise`.

### 2) Думать, что `response` и есть готовый объект данных

Нет. `response` - это объект ответа. Обычно нужно ещё вызвать `response.json()`.

### 3) Не проверять `response.ok`

Тогда `404` и `500` пройдут незаметно как “обычный ответ”.

### 4) Забывать `JSON.stringify()` при `POST` и `PATCH`

Сервер часто ждёт строку, а не живой JavaScript-объект.

### 5) Не ставить `Content-Type: application/json`

Тогда сервер может не понять формат данных.

### 6) Не использовать `finally` для восстановления интерфейса

Кнопка может остаться заблокированной после ошибки.

### 7) Не делать `empty`-состояние

Пустой экран без объяснения для пользователя выглядит как поломка.

### 8) Смешивать сеть, DOM и обработчики в одну огромную функцию

Лучше разделять:

- функции для сети;
- функции рендера;
- обработчики событий.

## Заключение

В этой лекции вы разобрали прикладную сетевую часть frontend-разработки:

- как клиент и сервер обмениваются данными;
- что такое `HTTP`, `status`, `method`, `API`;
- как использовать `fetch()`;
- почему `fetch` не падает на `404` автоматически;
- как читать данные через `response.json()`;
- зачем нужны `JSON.stringify()` и `JSON.parse()`;
- как сделать универсальную функцию `request()`;
- как выстроить `loading / success / empty / error`.

Главная мысль этой лекции:

> **Работа с API - это не только “отправить fetch”. Это ещё и правильно прочитать ответ, обработать ошибку и честно показать пользователю состояние интерфейса.**

Следующая тема - модули и структура проекта. Именно там мы начнём раскладывать эту логику по отдельным файлам, чтобы приложение не превращалось в один огромный `main.js`.

## Чек-лист после лекции

После этой лекции вы должны уметь:

- объяснить разницу между клиентом и сервером;
- объяснить, что такое `HTTP` и `API`;
- назвать базовые методы `GET`, `POST`, `PATCH`, `DELETE`;
- объяснить, что такое `Response`;
- использовать `fetch` с `then` и с `async/await`;
- проверить `response.ok`;
- читать ответ через `response.json()`;
- отправлять данные через `JSON.stringify()`;
- собрать query params через `URLSearchParams`;
- реализовать `loading`, `empty`, `error` в интерфейсе.

## Практика

> Условия:
>
> - если задание про сеть, используйте `jsonplaceholder`;
> - если задание про интерфейс, вывод должен быть не только в `console.log`, но и в `DOM`;
> - обрабатывайте ошибки через `try/catch` или `.catch()`.

1. Отправьте `GET`-запрос на:

`https://jsonplaceholder.typicode.com/posts/1`

Выведите:

- `status`
- `ok`
- `title` полученного поста

2. Отправьте запрос на несуществующий ресурс:

`https://jsonplaceholder.typicode.com/posts/999999`

Сделайте проверку через `response.ok` и выведите текст ошибки.

3. Создайте объект:

```javascript
const user = {
  name: "Alex",
  age: 27,
};
```

Превратите его в строку через `JSON.stringify()`, потом обратно в объект через `JSON.parse()`.

4. Отправьте `POST`-запрос на:

`https://jsonplaceholder.typicode.com/posts`

С телом:

```javascript
{
  title: "Hello",
  body: "World",
  userId: 1
}
```

5. Отправьте `PATCH`-запрос на:

`https://jsonplaceholder.typicode.com/posts/1`

И обновите только `title`.

6. Напишите свою функцию `request(url, method = "GET", data = null)`.

Проверьте её на:

- `GET`
- `POST`
- `PATCH`

7. Сделайте простой интерфейс:

```html
<button id="loadBtn">Load posts</button>
<div id="status"></div>
<ul id="posts"></ul>
```

По клику:

- показать `Loading...`;
- загрузить посты;
- вывести первые `10`;
- при ошибке показать сообщение;
- в конце вернуть кнопку в рабочее состояние.

8. Добавьте поле `userId` и кнопку фильтра.

Загрузка должна идти по URL вида:

`/posts?userId=1&_limit=10`

Собирайте параметры через `URLSearchParams`.

9. Добавьте `empty`-состояние:

- если массив пустой, выводите `Постов нет`.

10. Дополнительно:

- добавьте форму создания поста;
- после успешного `POST` добавляйте новый пост в начало списка на странице.

## Домашняя работа

Сделайте мини-проект **“Работа с API и постами”**.

### Что нужно создать

Создайте папку проекта, например:

`hw12_fetch_posts/`

Внутри должны быть файлы:

- `index.html`
- `style.css`
- `main.js`

### Интерфейс

На странице должны быть:

- блок статуса `#status`
- список постов `#posts`
- кнопка `Load posts`
- поле `userId`
- кнопка `Filter`
- форма создания поста:
  - `title`
  - `body`
  - кнопка `Create`

У каждого поста в списке должны быть:

- кнопка `Delete`
- кнопка `Change title`

### Что должно работать

#### 1) Загрузка постов

По кнопке `Load posts`:

- показать `Loading...`
- очистить старый список
- отправить `GET /posts`
- показать первые `10` постов
- при ошибке показать `Error: ...`

#### 2) Фильтр по `userId`

По кнопке `Filter`:

- взять `userId` из поля
- собрать URL через `URLSearchParams`
- отправить `GET /posts?userId=...&_limit=10`
- отрисовать результат

#### 3) Создание поста

По отправке формы:

- собрать данные формы
- отправить `POST /posts`
- после успеха добавить новый пост в начало списка
- очистить форму

#### 4) Удаление поста

По кнопке `Delete`:

- отправить `DELETE /posts/{id}`
- если всё успешно, удалить пост из DOM

#### 5) Изменение заголовка

По кнопке `Change title`:

- спросить новый текст через `prompt`
- отправить `PATCH /posts/{id}`
- после успеха обновить заголовок поста на странице

### Обязательные технические требования

1. Все сетевые запросы должны идти через одну функцию:

```javascript
async function request(url, method = "GET", data = null) {}
```

2. В `request()` обязательно должны быть:

- `method`
- `headers`, если есть `data`
- `JSON.stringify(data)`, если есть тело запроса
- проверка `response.ok`
- возврат `response.json()` или `null`, если тело пустое

3. В интерфейсе обязательно должны быть состояния:

- `loading`
- `success`
- `empty`
- `error`

4. Пока идёт запрос по кнопке загрузки или фильтрации, соответствующая кнопка должна быть `disabled`.

5. Для возврата кнопки в рабочее состояние используйте `finally`.

### Дополнительно

Если хотите усложнить проект:

1. Сделайте отдельный блок для ошибок красным цветом.
2. Добавьте кнопку `Reload`.
3. Добавьте кнопку `Load comments` для конкретного поста.
4. Сделайте универсальный рендер функции `renderPosts(posts)`.

### Что важно в решении

У вас должно быть видно не просто “я могу вызвать `fetch`”, а именно зрелый учебный сценарий:

- есть одна точка входа для сети через `request()`;
- есть проверка ошибок;
- есть чтение `JSON`;
- есть работа с `DOM`;
- есть понятные статусы интерфейса;
- есть `GET`, `POST`, `PATCH`, `DELETE`;
- есть фильтрация через query params.
