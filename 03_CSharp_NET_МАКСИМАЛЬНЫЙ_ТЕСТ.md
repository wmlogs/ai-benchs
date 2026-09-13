# МАКСИМАЛЬНЫЙ ТЕСТ: C# / .NET

TEST_VERSION: 2.0

## ПРАВИЛА

Выполни все 32 задания одним ответом без интернета, IDE, выполнения кода и уточняющих вопросов.

Требуется:
- не выдумывать .NET API;
- код должен компилироваться концептуально и быть согласован с объявленными типами;
- явно учитывать cancellation, disposal, async, concurrency, exception policy, ownership;
- если есть trade-off — назвать его;
- не использовать «магические» внешние пакеты, если они не нужны.

Начало:
```text
MODEL: <название модели или UNKNOWN>
MODE: <thinking|non-thinking|UNKNOWN>
TEST_VERSION: 2.0
```

Конец:
```text
FINAL
Выполнено заданий: <число>
Пропущено заданий: <число>
Использованы внешние инструменты: НЕТ
END_OF_TEST
```

Всего заданий: 32.

---

## КОНТЕКСТ ПРОЕКТА VEGA CORE

Запомни:
- production .NET 8;
- ASP.NET Core;
- PostgreSQL;
- Redis Streams;
- public API `/api/v5`;
- единственный legacy upload `/api/v4/files/upload`;
- tenant header `X-Org-Id`;
- idempotency `Idempotency-Key`;
- worker `OrionWorker`;
- external billing timeout 7 seconds;
- максимум 3 попытки всего;
- retry billing только если операция доказуемо идемпотентна;
- upload max 32 MiB;
- события хранятся минимум 90 дней;
- production Linux containers;
- Redis private only;
- secrets forbidden in Git;
- logs must not contain full PAN or bearer tokens;
- public API upgrades require zero downtime.

---

## 1. ASYNC/AWAIT И TASK SEMANTICS

### CSX-1
Укажи точный порядок и объясни:

```csharp
Console.WriteLine("A");

var t = Task.Run(async () =>
{
    Console.WriteLine("B");
    await Task.Yield();
    Console.WriteLine("C");
});

Console.WriteLine("D");
await t;
Console.WriteLine("E");
```

Назови, что здесь гарантировано, а что зависит от scheduler timing.

### CSX-2
Исправь:

```csharp
public async Task<List<string>> LoadAllAsync(List<string> urls)
{
    var result = new List<string>();
    urls.ForEach(async url =>
    {
        using var client = new HttpClient();
        result.Add(await client.GetStringAsync(url));
    });
    return result;
}
```

Требования: общий `HttpClient`, max parallelism 8, `CancellationToken`, сохранение порядка, явная exception policy.

### CSX-3
Объясни, почему `async void` почти всегда плох, где он допустим, и как исключения `async void` отличаются от `Task`.

### CSX-4
Что такое sync-over-async deadlock? Покажи классический пример с `.Result`/`.Wait()` и SynchronizationContext. Отдельно объясни, почему в ASP.NET Core ситуация отличается.

### CSX-5
`Task.WhenAll` при нескольких ошибках: что окажется в returned Task, что бросит `await`, и как надёжно собрать все exceptions?

---

## 2. CANCELLATION, TIMEOUT, HTTP

### CSX-6
Напиши метод `GetJsonAsync<T>(HttpClient client, string url, TimeSpan timeout, CancellationToken externalCt)`:
- timeout и внешняя отмена различаются;
- linked CTS корректно dispose;
- HTTP non-success -> exception;
- JSON deserialize;
- не терять stack trace.

### CSX-7
Почему создавать новый `HttpClient` на каждый запрос плохо? Какие проблемы решает `IHttpClientFactory`, а какие не решает автоматически?

### CSX-8
Спроектируй retry policy billing-вызова без Polly: максимум 3 попытки, 7 sec timeout на попытку, exponential backoff+jitter, retry только transient + idempotent. Не повторять 4xx по умолчанию.

---

## 3. THREAD SAFETY, MEMORY MODEL

### CSX-9
Исправь thread-unsafe counter двумя способами: `Interlocked` и `lock`. Объясни, почему `volatile int` не делает `++` атомарным.

### CSX-10
Double-checked locking singleton: когда он безопасен в современном C# и почему `Lazy<T>` обычно лучше? Дай корректный код.

### CSX-11
Есть `ConcurrentDictionary<string,List<int>>`. Почему `GetOrAdd(key, _ => new List<int>()).Add(x)` всё ещё может быть race? Дай два корректных дизайна.

### CSX-12
Спроектируй bounded producer/consumer на `Channel<T>`: несколько producers, 4 consumers, backpressure, graceful completion, cancellation и обработка ошибок.

---

## 4. DISPOSAL, OWNERSHIP, MEMORY

### CSX-13
Разница `IDisposable`, finalizer, `SafeHandle`, `IAsyncDisposable`. Когда нужен `await using`?

### CSX-14
Почему нельзя бездумно `Dispose()` зависимость, переданную в constructor? Объясни ownership и DI container lifetime.

### CSX-15
`Span<T>`, `Memory<T>`, `ReadOnlySpan<T>`: где их можно хранить, почему `Span<T>` нельзя держать в обычном heap-object или across `await`, и когда это важно.

### CSX-16
Назови минимум 8 причин роста памяти в долгоживущем .NET service, включая events, static caches, timers, unbounded channels, LOH, pinned memory, native handles, EF tracking.

---

## 5. LINQ / EF CORE / DATABASE

### CSX-17
Почему это опасно?

```csharp
var users = db.Users.ToList();
var result = users.Where(x => x.Active).OrderBy(x => x.Name).Take(20).ToList();
```

Дай правильный EF Core-вариант и индексационные рекомендации.

### CSX-18
Что такое N+1 в EF Core? Сравни `Include`, projection, explicit loading и split query. Когда `Include` сам создаёт проблему cartesian explosion?

### CSX-19
Опиши optimistic concurrency через concurrency token/row version. Как обрабатывать `DbUpdateConcurrencyException` без автоматической потери данных?

### CSX-20
Почему нельзя держать EF transaction открытой во время 7-секундного billing HTTP? Дай outbox/saga архитектуру.

---

## 6. LANGUAGE SEMANTICS

### CSX-21
Объясни разницу `throw;` и `throw ex;`. Важно: не утверждай, что `throw ex;` создаёт новый exception object, если это неверно.

### CSX-22
Nullable reference types: что гарантирует компилятор и чего НЕ гарантирует runtime? Приведи пример `null!`, reflection или deserialization.

### CSX-23
Records: value equality, `with`, shallow copy и mutable nested reference. Покажи ловушку «immutable-looking record».

### CSX-24
Deferred execution LINQ: что выведет и почему?

```csharp
var list = new List<int> {1,2,3};
var q = list.Where(x => x > 1);
list.Add(4);
Console.WriteLine(string.Join(",", q));
```

---

## 7. ASP.NET CORE / ARCHITECTURE

### CSX-25
Спроектируй idempotent `POST /api/v5/payments`: tenant header, idempotency record, request hash, concurrent duplicate request, processing/succeeded/failed/unknown, stored response.

### CSX-26
BackgroundService `OrionWorker` читает Redis Streams. Опиши consumer group, pending entries, retry, poison message, dedupe, graceful shutdown и crash after side-effect before ack.

### CSX-27
Как безопасно принимать файл до 32 MiB в ASP.NET Core без чтения целиком в memory? Укажи streaming, limits, validation, temp storage, authz/tenant и cleanup.

### CSX-28
Middleware order: exception handler, forwarded headers, HTTPS, routing, authentication, authorization, rate limiting, endpoints. Дай разумный порядок и объясни чувствительные зависимости.

---

## 8. LONG CONTEXT / SYSTEM DESIGN

### CSX-29
Перечисли по памяти:
- API prefix;
- legacy upload endpoint;
- tenant header;
- idempotency header;
- worker;
- billing timeout;
- max attempts;
- upload max;
- retention.

### CSX-30
Спроектируй полный payment flow в .NET: minimal API/controller → validation → idempotency → DB/outbox → billing 7 s → retry policy → unknown outcome → event → worker → logs/metrics/traces.

### CSX-31
Напиши компактный, но production-minded `BackgroundService`, который читает `Channel<WorkItem>` с max 4 параллельными обработчиками, поддерживает graceful shutdown, не теряет исключения и не запускает новые items после cancellation.

### CSX-32
Self-audit CSX-30 и CSX-31: минимум 10 конкретных failure modes, включая race, duplicate side effect, cancellation leak, deadlock, unbounded memory, shutdown loss и observability gap.

