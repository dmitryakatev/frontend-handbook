# CORS (Cross-Origin Resource Sharing)

CORS (Cross-Origin Resource Sharing) — это механизм безопасности браузера, который определяет, разрешено ли JavaScript выполнять HTTP-запросы к другому Origin.

Важно понимать, что CORS — это **не защита сервера**, а ограничение, реализованное в браузере.

Если отправить тот же запрос через Postman, curl или серверное приложение (например, Node.js), никаких ограничений CORS не будет.

## Почему появился CORS

Представим ситуацию:

1. Пользователь авторизован в интернет-банке.
2. В соседней вкладке открывает вредоносный сайт.
3. Если бы браузер разрешал любые AJAX-запросы между сайтами, злоумышленник мог бы выполнять запросы к банку от имени пользователя.

Именно поэтому браузер ограничивает выполнение JavaScript-запросов между разными Origin.

## Что такое Origin

Origin состоит из трех частей:

- схема (http / https);
- домен;
- порт.

Например:

```
https://example.com:443
```

Запрос считается **cross-origin**, если отличается хотя бы одна из этих частей.

Например:

```
https://example.com
https://api.example.com   ❌ другой домен

https://example.com
http://example.com        ❌ другая схема

https://example.com
https://example.com:8080  ❌ другой порт
```

## Same-Origin Policy (SOP)

По умолчанию браузер использует правило **Same-Origin Policy (SOP)**.

Оно означает:

> JavaScript может свободно обращаться только к своему Origin.

Все остальные запросы считаются **cross-origin** и требуют проверки по правилам CORS.

## Как работает CORS

Если сервер разрешает выполнять запросы с другого Origin, он должен вернуть соответствующие CORS-заголовки.

Например:

```http
Access-Control-Allow-Origin: https://app.example.com
```

После получения ответа браузер сравнивает Origin страницы со значением заголовка.

Если Origin разрешен — JavaScript получает доступ к ответу.

Если нет — браузер блокирует доступ.

> Важно понимать: HTTP-запрос обычно уже был отправлен и обработан сервером. Браузер блокирует именно доступ JavaScript к ответу.

## Простые запросы (Simple Request)

Некоторые запросы браузер отправляет сразу, без предварительной проверки.

К ним обычно относятся:

- GET;
- HEAD;
- POST (только с определенными `Content-Type` и без нестандартных заголовков).

Если сервер возвращает корректные CORS-заголовки, JavaScript получает доступ к ответу.

## Preflight Request

Если запрос считается потенциально небезопасным, браузер сначала автоматически отправляет дополнительный запрос:

```http
OPTIONS
```

Это называется **Preflight Request**.

Он нужен, чтобы узнать, разрешает ли сервер выполнить основной запрос.

Например:

```http
OPTIONS /users

Origin: https://app.example.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
```

Если сервер разрешает запрос, он отвечает примерно так:

```http
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Headers: Authorization, Content-Type
```

После этого браузер отправляет основной запрос.

## Когда выполняется Preflight

Preflight обычно появляется при использовании:

- PUT;
- PATCH;
- DELETE;
- нестандартных HTTP-заголовков;
- некоторых типов `Content-Type`.

Например:

```js
fetch(url, {
  method: "PUT",
  headers: {
    Authorization: token,
  },
});
```

Перед выполнением такого запроса браузер автоматически отправит `OPTIONS`.

## Cookies и CORS

По умолчанию браузер **не отправляет Cookies** в cross-origin AJAX-запросах.

Чтобы это работало, должны быть выполнены условия **и на клиенте, и на сервере**.

### Клиент

```js
fetch(url, {
  credentials: "include",
});
```

### Сервер

```http
Access-Control-Allow-Credentials: true
Access-Control-Allow-Origin: https://app.example.com
```

При этом использовать

```http
Access-Control-Allow-Origin: *
```

вместе с

```http
Access-Control-Allow-Credentials: true
```

нельзя.

Если Cookie должна участвовать в кросс-сайтовых запросах, она также должна иметь атрибуты:

```text
SameSite=None; Secure
```

## Основные CORS-заголовки

| Заголовок                          | Назначение                                       |
| ---------------------------------- | ------------------------------------------------ |
| `Access-Control-Allow-Origin`      | Какие Origin имеют доступ                        |
| `Access-Control-Allow-Methods`     | Разрешенные HTTP-методы                          |
| `Access-Control-Allow-Headers`     | Разрешенные пользовательские заголовки           |
| `Access-Control-Allow-Credentials` | Разрешает отправку Cookies и HTTP-аутентификации |
| `Access-Control-Max-Age`           | Позволяет кэшировать результат Preflight         |

## Что CORS НЕ защищает

Важно понимать ограничения CORS.

CORS **не защищает сервер** и не предотвращает:

- DDoS;
- SQL Injection;
- XSS;
- отправку запросов через Postman;
- отправку запросов с backend-приложений.

Он лишь запрещает JavaScript в браузере получать доступ к ответам другого Origin без разрешения сервера.

## Частые ошибки

❌ Говорить, что CORS — это защита сервера.

Нет. Это механизм безопасности браузера.

❌ Считать, что сервер блокирует запрос.

Чаще всего сервер уже получил и обработал запрос. Браузер блокирует доступ JavaScript к ответу.

❌ Говорить, что CORS работает в Postman.

Postman не реализует Same-Origin Policy, поэтому ограничения CORS на него не распространяются.

❌ Путать Origin и Domain.

Origin = схема + домен + порт.

❌ Думать, что `credentials: "include"` автоматически отправит Cookies.

Для этого одновременно необходимы:

- `credentials: "include"` на клиенте;
- `Access-Control-Allow-Credentials: true` на сервере;
- конкретный `Access-Control-Allow-Origin` (не `*`);
- `SameSite=None; Secure` у Cookie (для кросс-сайтовых сценариев).
