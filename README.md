# Everyday TypeScript Reference

## Type Intersections

`A & B` means "any type that's compatible with both A and B".

```typescript
type HasEmail = {email: string};
type CanBeAdmin = {admin: boolean};

// This type is {email: string, admin: boolean}.
type User = HasEmail & CanBeAdmin;
```

## Pick and Omit

The `Pick` type lets us create a smaller object type from an existing object type. We choose which properties to include.

```typescript
type User = {
  name: string
  email: string
  age: number
};

// This type is {name: string}.
type NameOnly = Pick<User, 'name'>;
```

The `Omit` type lets us create a new object type with some properties removed.

```typescript
type User = {
  name: string
  email: string
  age: number
};

// This type is {name: string}.
type NameOnly = Omit<User, 'email' | 'age'>;
```

## Impossible Conditions

Sometimes, TypeScript can tell that the condition in an if is impossible. It tells us that with a type error.

```typescript
let s;
if (1 === 0) {
  s = 'it worked';
}
// Result: type error: This comparison appears to be unintentional because the types '1' and '0' have no overlap.
```

## Functions as Arguments

Functions can take other functions. We use the standard function type syntax to declare them.

```typescript
function callFunction(f: () => number) {
  return f();
}

callFunction(() => 1);
// Result: 1
```

## Discriminated Unions

Discriminated unions are unions of multiple object types. They share a property called the discriminator. Each object type specifies a different literal value for the discriminator.

You can think of it as a label.

At runtime, we can use the discriminator property to decide which union alternative we're looking at. Discriminators can be any literal type. You might differentiate between types with true vs. false, 'user' vs. 'admin', etc.

In this example, the started property is the discriminator.

```typescript
type StartedCourse = {
  started: true
  lastInteractionTime: Date
};
type UnstartedCourse = {
  started: false
};
type Course = StartedCourse | UnstartedCourse;
```

## Shared Fields

We can use intersections to share fields between object types.

```typescript
type HasEmail = {email: string};

type User = HasEmail & {
  admin: boolean
};

type Company = HasEmail & {
  accountNumber: number
};
```

## Error Handling With Unions

Discriminated unions are useful for handling errors. The "success" side of the union contains the result value, and the "failure" side contains details about what went wrong.

This technique forces callers to handle the error case. In this example, if they try to access .value, they'll get a type error because only one union alternative has that property. That forces them to check .kind to find out whether the conversion was successful or not. If kind is 'success', then they can access .value.

```typescript
type ConversionSucceeded = {
  kind: 'success'
  value: number
};

type ConversionFailed = {
  kind: 'failure'
  reason: string
};

type Conversion =
  ConversionSucceeded | ConversionFailed;

function safeNumber(s: string): Conversion {
  /* convert to number here */
}
```

## Logical Operator Narrowing

Logical operators like `&&` narrow types. The right side of the `&&` is only evaluated if the left side is true, so it acts like a conditional.

```typescript
function arrayLength(
  strings: string[] | undefined
): number | undefined {
  return strings && strings.length;
}
```

Ternary conditionals also narrow types.

```typescript
function lengthOrNumber(
  n: number | number[]
): number {
  return Array.isArray(n) ? n.length : n;
}
```

## Optional Chaining

The `?.` operator is like the regular `.` operator for property access. However, if the value to the left of the `?.` is null or undefined, then the overall expression returns undefined.

```typescript
function getUser(): User | undefined {
  return {name: 'Amir'};
}

getUser()?.name;
```

## Nullish Coalescing

The `??` operator is similar to `||`, but more safe. It returns its right side when its left side is either undefined or null.

It's most often used to specify default values for when we encounter an undefined or null.

```typescript
const n: number | null = null;
n ?? 1;
// Result: 1

const n: number | undefined = undefined;
n ?? 'fallback';
// Result: 'fallback'

const b: boolean = false;
b ?? [2];
// Result: false
```

## Index Signatures

Index signatures allow us to access any property name on an object. We don't have to declare the properties one-by-one. All of the properties must have the same static type.

```typescript
type LoginCounts = {
  [userName: string]: number
};
const loginCounts: LoginCounts = {
  Amir: 5,
  Betty: 7,
};
```

TypeScript can't know whether any given property name will actually exist or not. If we access a property that doesn't exist, we'll get undefined, which violates our declared static type.

```typescript
type UserEmails = {[name: string]: string};
const userEmails: UserEmails = {
  Amir: 'amir@example.com'
};

userEmails.Betty;
// Result: undefined
```

## Type Soundness

In a sound type system, all static type guarantees remain true at runtime. TypeScript is unsound: in some situations, the static types will be violated at runtime.

```typescript
let names: string[] = ['Amir', 'Betty'];
let unsoundNames: (string | number)[] = names;
unsoundNames.push(5);
names;
// Result: ['Amir', 'Betty', 5]
```

## Interfaces

Interfaces are another way to declare object types.

```typescript
interface User1 {
  name: string
}

type User2 = {
  name: string
};
```

## Any

The `any` type allows any value. Each use of `any` effectively disables the type system for that expression. Using it can cause static types elsewhere in the system to be violated as well. We recommend using `any` only as a last resort.

```typescript
const n: any = 5;
const s: string = n;
s;
// Result: 5
```

## Exhaustiveness Checking

We can get TypeScript to check that our switch statements exhaustively consider all union alternatives. To do that, we declare an explicit return type for our function, then write a switch where each case returns.

```typescript
type User = {kind: 'user', userName: string};
type Cat = {kind: 'cat', catName: string};
type Dog = {kind: 'dog', dogName: string};

function getName(
  hasName: User | Cat | Dog,
  names: string[]
): true {
  switch (hasName.kind) {
    case 'user':
      names.push(hasName.userName);
      return true;
    case 'cat':
      names.push(hasName.catName);
      return true;
  }
}
// Result: type error: Function lacks ending return statement and return type does not include 'undefined'.
```

## Unknown

The `unknown` type is like a safe version of `any`. TypeScript won't let us use an `unknown` unless we narrow it to a more specific type.

```typescript
const n: unknown = 5;
const n2: number = typeof n === 'number'
  ? n
  : 0;
n2;
// Result: 5
```

## The Object Type

The `object` type allows objects with any properties, but we can't access those properties. You may see this type in legacy code, but it's rarely used now.

```typescript
const user: object = {
  name: 'Amir',
};

user.name;
// Result: type error: Property 'name' does not exist on type 'object'.
```

## As

The `as` operator forces a value to have a certain static type. It's very unsafe, like `any`, because it lets us specify types that are incorrect.

```typescript
const obj = {};
const wronglyTypedObj: string = obj as string;
```

## As Is Dangerous

The `as` operator works even if the conversion doesn't make sense, so it's unsafe. We recommend using it only as a last resort.

```typescript
const aNumber: unknown = 5;
const aString = aNumber as string;
const length: number = aString.length;
length;
// Result: undefined
```

## Object Values at Runtime

TypeScript's structural type system allows extra properties on objects. At runtime, our objects may have properties that aren't specified in their static types.

```typescript
const user = {
  name: 'Amir',
  email: 'amir@example.com'
};
const hasEmail: {email: string} = user;

/* `hasEmail`'s static type doesn't have a
 * `name` property, but the property is still
 * present at runtime. */
(hasEmail as any).name;
// Result: 'Amir'
```

## Type Widening

TypeScript widens the types of literal values where possible.

Sometimes we don't want TypeScript to widen the types in this way. We can add as const to prevent widening of a specific expression.

```typescript
// This type is {name: string}.
let user = {name: 'Amir'};

// This type is {name: 'Amir'}.
let user = {name: 'Amir' as const};
```

## The Empty Object Type

The empty object type `{}` allows any object. It specifies no properties, so we can't access any properties on the object.

```typescript
function takesAnObject(obj: {}) {
  return 'it worked';
}

[
  takesAnObject({name: 'Amir'}),
  takesAnObject({loginCount: 55}),
];
// Result: ['it worked', 'it worked']
```

## Arrays Are Objects

Arrays are technically objects, so `typeof` returns 'object'. This behavior originally comes from JavaScript.

Conditional narrowing on arrays is complicated by the fact that typeof someArray returns 'object'. To narrow an array, we have to use `Array.isArray` instead.

```typescript
typeof [1, 2, 3];
// Result: 'object'

function maybeLength(maybeArray: unknown) {
  if (Array.isArray(maybeArray)) {
    return maybeArray.length;
  } else {
    return undefined;
  }
}
```

## Void

Functions without return values have `void` as their return type.

```typescript
function f() {
}
const n: number = f();
// Result: type error: Type 'void' is not assignable to type 'number'.
```

## Never

`never` is used for values that will never actually occur at runtime. For example, functions that never return should have a return type of `never`.

```typescript
function throws(): never {
  throw new Error('oh no');
}
throws();
// Result: Error: oh no
```

## Classes

TypeScript supports static typing of JavaScript classes.

```typescript
class Cat {
  name: string;
  #vaccinated: boolean;

  constructor(
    name: string,
    vaccinated: boolean
  ) {
    this.name = name;
    this.#vaccinated = vaccinated;
  }

  needsVaccination(): boolean {
    return !this.#vaccinated;
  }
}
```

## Strict Compiler Flags

The TypeScript compiler has many compiler flags. We recommend enabling the 'strict' flag, which automatically enables several more fine-grained strictness flags.

```json
{
  "compilerOptions": {
    "strict": true,
  },
}
```

Some important strict flags include:

- `noImplicitAny`: Disallows function parameters with no type.
- `strictNullChecks`: Doesn't allow assigning null or undefined to variables that don't include those types.
- `strictPropertyInitialization`: Forces us to assign class properties in the constructor.

## Methods in Object Types

Object types can contain method definitions in addition to regular properties.

```typescript
type Dog = {
  name: string
  bark(): string
};

const dog: Dog = {
  name: 'Luna',
  bark: () => 'woof',
};

dog.bark();
// Result: 'woof'
```

## Extending Types

Interfaces can extend other interfaces as well as classes. The child interface gets all of its parent's properties.

```typescript
interface CanBeAdmin {
  admin: boolean
}

interface User extends CanBeAdmin {
  name: string
}

const user: User = {name: 'Amir', admin: true};
user;
// Result: {name: 'Amir', admin: true}
```

## Impossible Intersections

Some type intersections can't actually happen at runtime. Those intersections give us the type `never`.

```typescript
type HasEmail1 = {
  email: string
};
type HasEmail2 = {
  email: number
};

/* The `email` property's type is `never`
 * because no value can be both a string and
 * a number. */
type User = HasEmail1 & HasEmail2;
const user: User = {
  email: 'amir@example.com'
};
// Result: type error: Type 'string' is not assignable to type 'never'.
```

## Implementing Types

Classes can implement one or more interface. TypeScript will ensure that the class has all of the properties declared in the implemented interfaces. However, we have to manually declare those properties in the child class.

```typescript
type HasName = {
  name: string
};

class User implements HasName {
  name: string;
  
  constructor(name: string) {
    this.name = name;
  }
}

const amir = new User('Amir');
const hasName: HasName = amir;
hasName.name;
// Result: 'Amir'
```

## Single and Multiple Inheritance

JavaScript and TypeScript don't support multiple inheritance. However, classes can implement multiple interfaces, which serves some of the use cases of multiple inheritance.

## Optional Properties

Object properties can be optional by adding the `?` syntax. We don't have to specify optional properties when building an object at runtime. From a type perspective, all optional properties are unioned with `undefined`.

```typescript
type User = {
  name: string
  postalCode?: string
};

const amir: User = {
  name: 'Amir',
};

amir.postalCode;
// Result: undefined
```

## Object Spread

The object spread operator works in TypeScript, and it enforces types as usual.

```typescript
const partialAmir = {name: 'Amir'};
const fullAmir = {...partialAmir, admin: true};
fullAmir;
// Result: {name: 'Amir', admin: true}
```

## Function Parameters

Function parameters can be optional, have default values, or be rest parameters.

```typescript
function add(x: number, y?: number) {
  return x + (y ?? 1);
}
```

Parameters can have default values.


```typescript
function add(x: number, y: number = 1) {
  return x + y;
}
```

Functions can have "rest parameters", which gather an arbitrary number of arguments into a single local variable.
```typescript
function add(...numbers: number[]) {
  let sum = 0;
  for (const n of numbers) {
    sum += n;
  }
  return sum;
}
[add(), add(1, 2), add(100, 200, 300)];
// Result:
// [0, 3, 600]
```

## Object Literal May Only Specify Known Properties

If we specify unknown properties when defining an object literal, TypeScript gives us a type error. This stops us from specifying useless properties, and it can even prevent bugs in some cases.

```typescript
const user: {name: string} = {
   name: 'Amir',
   age: 36
};
// Result: type error: Object literal may only specify known properties, and 'age' does not exist in type '{ name: string; }'.
```

## Types for Options Objects

We can statically type complex functions with option types by combining default parameter values and optional object properties.

```typescript
/* Callers can specify none, some, or all of
 * `opts`. We'll use default values for any
 * options that are missing. */
function sendEmail(
  to: string,
  opts: {
    tracking?: boolean
    retries?: number
  } = {}
) {
  const tracking = opts.tracking ?? true;
  const retries = opts.retries ?? 3;
  /* Send email here. */
}
```

## Indexing Into Object Types

We can index into object types directly, giving us the types of individual properties.

```typescript
type Album = {
  name: string
  copiesSold: number
};

type AlbumName = Album['name'];
```

## Indexing Into Tuple and Array Types

Indexing into an array type gives us the array's element type, no matter which index we ask for.

```typescript
type Names = string[];
const username: Names[0] = 'cindy';
```

Indexing into a tuple type gives us that specific element's type.

```typescript
type StringAndNumber = [string, number];
const age: StringAndNumber[1] = 29;
```

## Sets

JavaScript sets are statically typed in TypeScript.

```typescript
const s: Set<number> = new Set([1, 2, 3]);
s.add(4);
s.has(4);
// Result:
// true

const s: Set<number> = new Set([1, 2, 3]);
s.add('this is not a number');
// Result:
// type error: Argument of type 'string' is not assignable to parameter of type 'number'.
```

## Promises

The Promise type is generic. Its type parameter represents the value type when fulfilled.

```typescript
const promise: Promise<number> =
  Promise.resolve(100);
promise;
// Async Result:
// {fulfilled: 100}
```

## ReturnType and Parameters

The ReturnType generic type gives us a function's return type.

```typescript
type StringFunction = () => string;
type OurString = ReturnType<StringFunction>;
```

The Parameters generic type gives us a function's parameters as a tuple type.

```typescript
type DoubleFunction = (n: number) => number;
type OurNumber = Parameters<DoubleFunction>[0];
```

## Async Await

Async functions must have a return type of Promise<T>. The T will depend on what our function actually returns.

```typescript
async function asyncDouble(
  x: Promise<number>
): Promise<number> {
  return 2 * await x;
}

asyncDouble(Promise.resolve(5));
// Async Result: {fulfilled: 10}
```

## Partial

The Partial generic type comes with TypeScript. It makes every property in an object type optional.

```typescript
type User = {
  name: string
  postalCode: string
};

const partialAmir: Partial<User> = {
  postalCode: '75010'
};
partialAmir.postalCode;
// Result: '75010'
```

## Partial in Practice

We can use the Partial type to create object template systems. This is useful in situations like unit testing, where we create many repetitive objects with small differences.

```typescript
type User = {
  name: string
  country: string
};

const template: User = {
  name: 'Amir',
  country: 'France',
};

/* This function combines the template object
 * and the overrides to produce a new object. */
function override(
  overrides: Partial<User>
): User {
  return {...template, ...overrides};
}

override({name: 'Betty'});
// Result: {name: 'Betty', country: 'France'}
```

## Exceptions

TypeScript can't use concrete types for caught exceptions. We can choose to type them as any, which is dangerous, or we can type them as unknown, which we recommend.

```typescript
let error: Error;

try {
  throw new Error('something broke');
} catch (e: unknown) {
  /* We can only catch exceptions as `unknown`
   * or `any`. We use `unknown` because it's
   * safer, but then we have to narrow the type
   * with a conditional. */
  if (e instanceof Error) {
    error = e;
  } else {
    throw new Error("We can't catch that!");
  }
}
error.message;
// Result: 'something broke'
```

## Rejected Promises

In JavaScript and TypeScript, rejected promises have rejection reasons. But TypeScript can't statically type the reasons. As with exceptions, we can use any, which is unsafe, or we can use unknown, which is a safer alternative.

```typescript
Promise.reject('it failed')
  .catch((reason: unknown) => {
    /* We can only catch rejections as `unknown`
     * or `any`. We use `unknown` because it's
     * safer, but then we have to narrow the
     * type with a conditional. */
    if (typeof reason === 'string') {
      const theReason: string = reason;
      return theReason;
    } else {
      throw new Error("We can't handle that!");
    }
  });
// Async Result: {fulfilled: 'it failed'}
```

## ReadonlyArray

Read-only arrays act like normal arrays, except that we can't modify their elements. We also can't call any methods that would change the array, like .push(...) or .sort().

```typescript
const strings: ReadonlyArray<string> = [
  'a',
  'b',
  'c'
];
strings.push('d');
// Result: type error: Property 'push' does not exist on type 'readonly string[]'.
```

## Enum

Enums allow us to express types that only accept certain literal values. TypeScript supports enums, but they're generally discouraged in new code. We recommend using unions of literal types instead.

```typescript
enum HttpMethod {
  Get = 'GET',
  Post = 'POST',
}
const method: HttpMethod = HttpMethod.Post;
method;
// Result: 'POST'
```


## Readonly

The readonly modifier makes an individual object property read-only.

```typescript
>
interface User {
  readonly name: string
}

const amir: User = {name: 'Amir'};
amir.name = 'Betty';
Result:
type error: Cannot assign to 'name' because it is a read-only property.
```

The Readonly generic type makes all properties on an object type read-only.


```typescript
>
type User = Readonly<{
  name: string
  age: number
}>;

const amir: User = {name: 'Amir', age: 36};
amir.name = 'Betty';
// Result:
// type error: Cannot assign to 'name' because it is a read-only property.
```

## Namespaces

Namespaces are like modules, except that more than one namespace can live in a single file. TypeScript supports namespaces, but they're generally discouraged in new code. We recommend using separate modules in their own files instead.

```typescript
>
namespace Util {
  export function wordCount(s: string) {
    return s.split(/\b\w+\b/g).length - 1;
  }
}

namespace Tests {
  export function testWordCount() {
    if (Util.wordCount('hello there') !== 2) {
      throw new Error('Word count mismatch');
    }
    return 'ok';
  }
}

Tests.testWordCount();
// Result:
// 'ok'
```

## Readonly Properties vs. Values

There are two senses in which an object property can be read-only, and they're easily confused.

First, the property itself can be read-only, which means that we're not allowed to replace the property's value with an entirely new value.

Second, the value held in the property can be read-only, like a read-only array. We can replace the entire array if we want to, but we can't modify the array once it's in the property.

```typescript
>
type User = {
  name: string
  readonly catNames: string[]
};

/* The `catNames` property is read-only, so we
 * can't do `amir.catNames = ...`. But the array
 * itself is read-write, so we can do
 * `amir.catNames.push(...)`. */
const amir: User = {
  name: 'Amir',
  catNames: ['Ms. Fluff'],
};
```
