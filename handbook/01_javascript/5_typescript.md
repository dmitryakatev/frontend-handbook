# TypeScript. Система типов

TypeScript — это надстройка над JavaScript, которая добавляет статическую типизацию.

Важно понимать, что TypeScript работает только на этапе разработки.

Во время компиляции вся информация о типах полностью удаляется.

### Пример

```ts
const sum = (a: number, b: number): number => a + b;
```

После компиляции:

```js
const sum = (a, b) => a + b;
```

Поэтому TypeScript не влияет на производительность приложения во время выполнения.

## Структурная типизация (Structural Typing)

TypeScript использует **структурную типизацию**.

Совместимость типов определяется структурой объекта, а не его именем.

### Пример

```ts
interface User {
  name: string;
}

interface Person {
  name: string;
}

const user: User = { name: "Alex" };

const person: Person = user;
```

Несмотря на разные названия интерфейсов, присваивание допустимо, поскольку структура объектов совпадает.

В отличие от TypeScript, языки с номинальной типизацией (например, Java или C#) учитывают принадлежность объекта к конкретному типу.

## Generics

Generics позволяют писать универсальный код без потери информации о типах.

### Пример

```ts
function identity<T>(value: T): T {
  return value;
}

identity(5);
identity("text");
```

TypeScript автоматически выводит тип аргумента и возвращаемого значения.

Generics могут использоваться:

- в функциях;
- интерфейсах;
- классах;
- type alias.

## Ограничение Generics

Иногда необходимо ограничить допустимые типы.

### Пример

```ts
function printLength<T extends { length: number }>(value: T) {
  console.log(value.length);
}
```

В этом случае функция принимает только значения, имеющие свойство `length`.

## Conditional Types

Conditional Types позволяют выбирать один тип или другой в зависимости от условия.

Общий вид:

```ts
T extends U ? X : Y
```

### Пример

```ts
type IsString<T> = T extends string ? true : false;
```

```ts
type A = IsString<string>; // true
type B = IsString<number>; // false
```

На основе Conditional Types реализовано большинство встроенных Utility Types.

## infer

`infer` позволяет извлекать часть существующего типа.

### Пример

```ts
type Return<T> = T extends (...args: any[]) => infer R ? R : never;
```

```ts
type A = Return<() => number>; // number
```

Через `infer` реализованы многие встроенные Utility Types:

- `ReturnType`;
- `Parameters`;
- `ConstructorParameters`;
- `Awaited`.

## Mapped Types

Mapped Types позволяют преобразовывать свойства существующего типа.

### Пример

```ts
type ReadonlyUser = {
  readonly [K in keyof User]: User[K];
};
```

Так реализован встроенный `Readonly`.

Другой пример:

```ts
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};
```

В результате получится новый тип, в котором каждое свойство может принимать значение `null`.

## Template Literal Types

Template Literal Types позволяют создавать новые строковые типы на основе существующих.

### Пример

```ts
type Event = "click" | "focus";

type Name = `on${Capitalize<Event>}`;
```

Результат:

```ts
type Name = "onClick" | "onFocus";
```

Часто используются при генерации API и имен событий.

## Discriminated Union

Discriminated Union позволяет безопасно описывать несколько вариантов одного типа.

### Пример

```ts
type Shape =
  | { type: "circle"; radius: number }
  | { type: "square"; size: number };
```

После проверки:

```ts
if (shape.type === "circle") {
  // shape имеет тип Circle
}
```

TypeScript автоматически сужает тип.

Этот механизм называется **Type Narrowing**.

## never

`never` обозначает значение, которое никогда не может существовать.

Используется:

- в функциях, которые никогда не завершаются;
- при `throw`;
- в бесконечных циклах;
- для проверки полноты `switch`.

### Пример

```ts
function check(shape: Shape) {
  switch (shape.type) {
    case "circle":
      break;

    case "square":
      break;

    default:
      const value: never = shape;
  }
}
```

Если добавить новый вариант в `Shape`, TypeScript сообщит об ошибке.

## unknown

`unknown` — безопасная альтернатива `any`.

Использовать значение напрямую нельзя — сначала необходимо проверить его тип.

### Пример

```ts
let value: unknown;

if (typeof value === "string") {
  console.log(value.toUpperCase());
}
```

| `any`                          | `unknown`                    |
| ------------------------------ | ---------------------------- |
| Можно выполнять любые операции | Требуется проверка типа      |
| Отключает проверку типов       | Сохраняет безопасность типов |

## satisfies

Оператор `satisfies` появился в TypeScript 4.9.

Он позволяет проверить соответствие типу, не изменяя вывод типов.

### Пример

```ts
const routes = {
  home: "/",
  about: "/about",
} satisfies Record<string, string>;
```

В отличие от обычной аннотации типа, литеральные типы сохраняются.

Часто используется при описании конфигурационных объектов.

## Variance

Variance определяет, можно ли заменить один тип другим.

Различают несколько видов.

### Covariance

Используется для возвращаемых значений.

Более конкретный тип может использоваться вместо более общего.

### Contravariance

Используется для параметров функций.

Проверяется в противоположном направлении относительно возвращаемых значений.

### Bivariance

Особенность TypeScript.

Для совместимости с JavaScript некоторые callback-параметры проверяются менее строго.

## Declaration Merging

TypeScript умеет объединять несколько объявлений одного интерфейса.

### Пример

```ts
interface User {
  name: string;
}

interface User {
  age: number;
}
```

После объединения получится:

```ts
interface User {
  name: string;
  age: number;
}
```

Этот механизм активно используют многие библиотеки.

## Module Augmentation

Module Augmentation позволяет расширять существующие модули.

### Пример

```ts
declare module "express" {
  interface Request {
    user: User;
  }
}
```

После этого свойство `req.user` станет доступно во всем проекте.

## Встроенные Utility Types

Большинство встроенных Utility Types реализованы с использованием Conditional Types, Mapped Types и `infer`.

| Utility Type    | Назначение                                    |
| --------------- | --------------------------------------------- |
| `Partial<T>`    | Делает все свойства необязательными           |
| `Required<T>`   | Делает все свойства обязательными             |
| `Readonly<T>`   | Делает свойства только для чтения             |
| `Pick<T, K>`    | Выбирает указанные свойства                   |
| `Omit<T, K>`    | Исключает указанные свойства                  |
| `Record<K, T>`  | Создает объект с ключами `K` и значениями `T` |
| `Exclude<T, U>` | Исключает типы из объединения                 |
| `Extract<T, U>` | Оставляет пересечение типов                   |
| `ReturnType<T>` | Возвращает тип результата функции             |
| `Parameters<T>` | Возвращает типы параметров функции            |
| `Awaited<T>`    | Извлекает тип значения `Promise`              |

## Практические рекомендации

- Используйте Generics, если необходимо сохранить информацию о типах.
- Предпочитайте `unknown` вместо `any`.
- Используйте Discriminated Union вместо большого количества необязательных свойств.
- Для преобразования существующих типов применяйте Mapped Types и Conditional Types.
- Используйте `satisfies` при описании конфигурационных объектов.
- Не злоупотребляйте сложной типовой логикой — иногда простой интерфейс читается значительно лучше.

## Частые ошибки

❌ Считать, что TypeScript существует во время выполнения программы.

После компиляции вся информация о типах удаляется.

❌ Использовать `any` без необходимости.

В большинстве случаев безопаснее использовать `unknown`.

❌ Использовать Generics там, где достаточно обычного интерфейса.

Не каждый тип должен быть универсальным.

❌ Злоупотреблять сложными Conditional Types и Mapped Types.

Слишком сложная типовая логика делает код трудным для понимания.

❌ Путать структурную и номинальную типизацию.

В TypeScript совместимость определяется структурой типов, а не их именем.
