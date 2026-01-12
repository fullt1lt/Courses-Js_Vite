# JavaScript — план лекций

Этот репозиторий содержит материалы и практику по JavaScript (после блока HTML/CSS). План построен по логике: **основа языка → работа в браузере (DOM/Events) → асинхронность и API → модули и инструменты → тестирование и деплой**.

## План (16 лекций)

1. **Введение в JavaScript + среда разработки**

   - Подключение JS (`defer/async`), базовый синтаксис, `console`
   - DevTools: Sources, breakpoints, отладка
2. **Переменные, типы, операторы + приведение типов**

   - `let/const`, примитивы, `typeof`
   - truthy/falsy, `==` vs `===`, преобразования типов
3. **Управляющие конструкции**

   - `if/else`, `switch`
   - циклы `for/while`, `break/continue`
4. **Функции, область видимости, hoisting, замыкания**

   - виды функций, параметры, `return`
   - scope, hoisting, closures
5. **Массивы и методы массивов**

   - базовые операции
   - `map`, `filter`, `reduce`, `find`, `some/every`, сортировка
6. **Объекты в JS + `this` (база)**

   - свойства, методы, перебор
   - деструктуризация, spread/rest (минимум), контекст `this`
7. **Прототипы и классы (ООП)**

   - prototype-цепочка (понятийно)
   - `class`, constructor, методы, наследование
8. **DOM: поиск, создание, изменение**

   - `querySelector`, `classList`
   - `createElement`, вставка/удаление, атрибуты
9. **События**

   - `addEventListener`, объект события
   - bubbling/capturing (понятийно), `preventDefault`, делегирование
10. **Хранение данных в браузере**

- Cookie vs LocalStorage vs SessionStorage
- JSON-сериализация, хранение состояния UI

11. **Асинхронность: promises, async/await + event loop**

- callbacks vs promises
- `then/catch/finally`, `async/await`, основы event loop

12. **Fetch + HTTP + JSON + обработка ошибок + CORS**

- `fetch`, методы, статус-коды, заголовки
- `try/catch`, типовые ошибки, что такое CORS

13. **Модули и npm (экосистема)**

- `import/export`
- `package.json`, установка пакетов
- ESM и CommonJS (кратко)

14. **Vite и сборка проекта**

- dev server, build
- структура проекта, env, assets

15. **Тестирование JavaScript (Jest)**

- unit-тесты функций
- базовые матчи, фикстуры, организация тестов

16. **Подготовка к продакшену и деплой**

- оптимизация, минификация, Lighthouse (основы)
- деплой (GitHub Pages/Netlify)
