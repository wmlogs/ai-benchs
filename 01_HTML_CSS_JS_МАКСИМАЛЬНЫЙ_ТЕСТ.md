# МАКСИМАЛЬНЫЙ ТЕСТ: HTML + CSS + JavaScript

TEST_VERSION: 2.0

## ОБЯЗАТЕЛЬНЫЕ ПРАВИЛА

Ты проходишь закрытый инженерный тест. Выполни ВСЕ задания за один ответ.

Запрещено:
- задавать уточняющие вопросы;
- использовать интернет, внешние инструменты, IDE, браузер, выполнение кода или поиск;
- выдумывать несуществующие Web API, CSS-свойства, DOM-методы или поведение браузера;
- заменять точный ответ общими рекомендациями.

Требуется:
- если условие невозможно или противоречиво — сказать это прямо;
- если есть несколько допустимых трактовок — назвать их и выбрать наиболее строгую;
- production-код должен быть завершённым и внутренне согласованным;
- учитывать accessibility, XSS, CSP, performance, race conditions, cancellation, browser semantics;
- не использовать фреймворки и внешние библиотеки, кроме случаев, когда задание прямо разрешает их;
- не пропускать ID.

В начале ответа выведи ровно:

```text
MODEL: <название модели или UNKNOWN>
MODE: <thinking|non-thinking|UNKNOWN>
TEST_VERSION: 2.0
```

В конце ответа выведи:

```text
FINAL
Выполнено заданий: <число>
Пропущено заданий: <число>
Использованы внешние инструменты: НЕТ
END_OF_TEST
```

Всего заданий: 32.

---

## КОНТЕКСТ ПРОЕКТА HELIOS UI

Запомни эти факты до конца теста:
- production поддерживает последние две стабильные версии Chrome, Edge, Firefox и Safari;
- приложение работает без React/Vue/Angular;
- публичный API начинается с `/api/v3`;
- единственный legacy endpoint: `/api/v2/avatar/upload`;
- tenant передаётся заголовком `X-Workspace-Id`;
- операции изменения требуют `Idempotency-Key`;
- поиск должен отменять предыдущий HTTP-запрос;
- максимальный размер аватара — 8 MiB;
- CSP запрещает inline-script и `unsafe-eval`;
- основной список может содержать 200 000 строк;
- приложение должно быть полностью keyboard-accessible;
- запрещено вставлять непроверенный пользовательский HTML;
- локальная тема хранится в `localStorage` под ключом `helios.theme`;
- WebSocket endpoint: `/api/v3/live`;
- сервер может присылать события повторно;
- все пользовательские времена в API приходят в UTC ISO-8601.

---

## 1. EVENT LOOP, PROMISES, RACE CONDITIONS

### WEBX-1
Без запуска кода укажи точный порядок вывода и объясни очередь microtask/macrotask:

```js
console.log('A');

setTimeout(() => console.log('B'), 0);

Promise.resolve()
  .then(() => {
    console.log('C');
    queueMicrotask(() => console.log('D'));
  })
  .then(() => console.log('E'));

queueMicrotask(() => console.log('F'));

(async () => {
  console.log('G');
  await 0;
  console.log('H');
})();

console.log('I');
```

### WEBX-2
Дан поиск, где поздний ответ старого запроса может перезаписать новый результат. Напиши production-функцию `searchUsers(query, externalSignal)` без библиотек, которая:
- отменяет предыдущий поиск;
- поддерживает внешний `AbortSignal`;
- имеет timeout 4 секунды;
- различает timeout, внешнюю отмену и HTTP-ошибку;
- не оставляет listeners/timers;
- никогда не рендерит устаревший ответ.

### WEBX-3
Объясни, почему `Promise.race([fetch(url), timeoutPromise])` сам по себе не отменяет `fetch`, и покажи корректную реализацию отмены.

### WEBX-4
Есть 10 000 независимых URL. Нельзя запускать больше 12 запросов одновременно. Напиши `mapConcurrent(items, limit, mapper, signal)` без библиотек. Требования:
- сохранять порядок результатов;
- при первой ошибке отменять дальнейший запуск;
- уже запущенные операции получают общий signal;
- не создавать 10 000 одновременно активных async-задач.

---

## 2. DOM, XSS, CSP, TRUST BOUNDARIES

### WEBX-5
Почему это XSS и как исправить для двух разных требований?

```js
result.innerHTML = user.profile.bio;
```

Требование A: bio — только текст.
Требование B: разрешены `<b>`, `<i>`, `<a href>`, но запрещены script/event handlers/javascript URLs.

### WEBX-6
При CSP без inline-script и без `unsafe-eval` нужно передать серверные данные в страницу. Дай два безопасных способа без inline JavaScript-кода и объясни риски JSON внутри HTML.

### WEBX-7
Объясни разницу XSS, CSRF и CORS на одном примере SPA с cookie-сессией. Для каждого назови минимальную корректную защиту.

### WEBX-8
Дан код:

```js
const el = document.querySelector('#status');
el.setAttribute('aria-label', userText);
el.innerHTML = `<span>${userText}</span>`;
```

Какая строка опасна, какая сама по себе не создаёт HTML-инъекцию и почему?

---

## 3. ACCESSIBILITY И ФОРМЫ

### WEBX-9
Напиши доступную форму email+пароль с:
- явными label;
- подсказками;
- ошибками, связанными с полями;
- `aria-invalid` только при ошибке;
- summary ошибок после submit;
- корректным focus management;
- без ARIA там, где хватает native HTML.

Нужны HTML и минимальный JS.

### WEBX-10
Спроектируй доступное modal-dialog без `<dialog>`. Требования:
- focus trap;
- возврат фокуса;
- Escape;
- блокировка фоновой интерактивности;
- screen-reader semantics;
- nested modal не требуется.

Дай HTML+JS и объясни ограничения.

### WEBX-11
Почему `role="button" tabindex="0"` на `<div>` всё ещё хуже `<button>`? Перечисли поведение, которое пришлось бы реализовать вручную.

### WEBX-12
SVG-иконка «галочка» стоит рядом с текстом «Сохранено». Покажи корректный SVG. Затем покажи вариант, когда иконка сама является единственным носителем смысла.

---

## 4. CSS: CASCADE, LAYOUT, RESPONSIVE

### WEBX-13
Определи итоговый цвет текста и объясни cascade layers, specificity и `!important`:

```css
@layer reset, components, overrides;

@layer components {
  .card .title { color: blue; }
}

@layer reset {
  #app .title { color: red !important; }
}

@layer overrides {
  .title { color: green; }
}

.title { color: purple; }
```

HTML:
```html
<div id="app" class="card"><h2 class="title">X</h2></div>
```

### WEBX-14
Почему flex/grid-item с длинной строкой иногда не сжимается и ломает layout? Объясни роль `min-width: auto` и покажи исправление.

### WEBX-15
Напиши CSS layout карточек: минимум 260px, без media queries, 1–N колонок по ширине контейнера, gap 16px. Затем вариант на container query, где карточка при ширине контейнера <420px переключается на вертикальный layout.

### WEBX-16
Объясни разницу `display:none`, `visibility:hidden`, `opacity:0`, `content-visibility:auto` с точки зрения layout, paint, hit-testing и accessibility tree.

### WEBX-17
Почему `position: sticky` может «не работать»? Назови минимум пять реальных причин/ограничений и способ диагностики.

---

## 5. PERFORMANCE

### WEBX-18
Список содержит 200 000 строк. Нельзя рендерить их все. Опиши архитектуру windowing/virtualization без библиотеки: расчёт видимого диапазона, overscan, высота spacer, переменная высота строк, keyboard navigation, screen-reader компромиссы.

### WEBX-19
Что такое layout thrashing? Исправь:

```js
for (const row of rows) {
  row.style.width = container.offsetWidth + 'px';
  row.style.height = row.offsetHeight + 1 + 'px';
}
```

### WEBX-20
Когда `DocumentFragment` реально помогает, а когда браузер и так оптимизирует вставки? Сравни с `innerHTML`, `replaceChildren` и incremental DOM updates.

### WEBX-21
Страница тормозит, но CPU profile показывает мало JS. Назови системный порядок диагностики: style recalculation, layout, paint, compositing, network, image decode, GC, long tasks.

---

## 6. NETWORK, WEBSOCKET, OFFLINE

### WEBX-22
Напиши устойчивый WebSocket-клиент для `/api/v3/live`:
- exponential backoff + jitter;
- максимум 30 секунд между попытками;
- не reconnect при явном `close()` пользователем;
- heartbeat/ping если протокол приложения это поддерживает;
- dedupe событий по `eventId`;
- bounded очередь исходящих сообщений;
- cleanup listeners/timers.

Код может быть классом.

### WEBX-23
Объясни, почему reconnect WebSocket без jitter опасен после массового рестарта сервера.

### WEBX-24
Service Worker: спроектируй cache strategy для shell-ассетов и API. Требования:
- HTML не должен навсегда застревать в старой версии;
- POST никогда не кешировать;
- API-ответы tenant-specific;
- logout не должен оставить чувствительные данные другого пользователя.

### WEBX-25
Почему кешировать `/api/v3/users` только по URL опасно, если tenant задаётся `X-Workspace-Id`? Где должна быть граница кеша?

---

## 7. STORAGE, TIME, MODULES

### WEBX-26
Реализуй `getTheme()`/`setTheme(theme)` для ключа `helios.theme`. Допустимы только `light|dark|system`. Нужно безопасно пережить недоступный `localStorage` и синхронизировать изменение темы между вкладками.

### WEBX-27
API присылает `2026-11-03T01:30:00Z`. Как корректно показать локальное время пользователя? Почему нельзя просто обрезать `Z` или вручную прибавлять timezone offset?

### WEBX-28
Объясни live bindings в ES modules. Какой будет результат и почему?

`counter.js`
```js
export let n = 0;
export function inc() { n++; }
```

`main.js`
```js
import { n, inc } from './counter.js';
console.log(n);
inc();
console.log(n);
```

---

## 8. LONG CONTEXT / PRODUCTION DESIGN

### WEBX-29
Без перечитывания условий перечисли:
1. API prefix;
2. legacy upload endpoint;
3. tenant header;
4. idempotency header;
5. max avatar size;
6. localStorage theme key;
7. WebSocket endpoint.

### WEBX-30
Спроектируй production flow загрузки аватара с клиентской стороны. Нужно учесть 8 MiB, legacy endpoint, progress, отмену, retry только когда безопасно, idempotency, tenant, MIME/extension не считать достаточной защитой.

### WEBX-31
Напиши в одном ответе минимальный, но законченный компонент «поиск пользователей» на vanilla HTML+CSS+JS:
- доступное поле поиска;
- debounce 250 ms;
- отмена старого запроса;
- timeout 4 s;
- состояние loading/error/empty;
- keyboard-accessible список;
- защита от XSS;
- не более 50 DOM-строк за один рендер;
- API `/api/v3/users?q=...`;
- `X-Workspace-Id` берётся из заранее известной переменной `workspaceId`.

### WEBX-32
Проведи self-audit ответа WEBX-31. Назови минимум 8 потенциальных production-рисков или edge cases и для каждого конкретное улучшение. Не переписывай всё решение заново.

