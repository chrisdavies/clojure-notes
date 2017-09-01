# TypeScript

Philosophically, statically typed languages encourage you to think of your program in terms of types and operations on those types.

In most systems I've written, there were thousands of types, and many methods per type. This is a lot of mental overhead if you don't have a good editor. Fortunately, static typing lends itself to good editor / IDE experiences.

Like JavaScript, TypeScript is multi-paradigm, supporting functional, OOP, and imperative approaches.


## SOLID

When working in statically typed, OOP world, SOLID principles are worth noting.

https://en.wikipedia.org/wiki/SOLID_(object-oriented_design)


S: single responsibility principle

A class should have only one responsibility... not always easy to identify, as responsibilities can be broad... (Kind of encouraged by default in functional languages.)


O: open / closed principle

Software entities should be open for extensions, closed for modification. Generally, in OOP languages, this means you extend by inheriting and overriding methods or by composing classes. (Ruby's monkey patching is an example of a violation of this principle.)


L: liskov substitution principle

Objects in a program should be replaceable with instances of their subtypes without changing the correctness of that program. Basically, this means that a child class should be interchangeable with its parent without breaking the program. If it breaks, it means the child is violating some guarantee of its parent. No bueno. (I've can't think of a time I ever violated this principle.)


I: interface segregation principle

Many client-specific interfaces are better than one general-purpose interface. Basically, write small, focused interfaces, don't write God-interfaces.


D: dependency inversion principle

One should depend on abstractions, not concretions. This is the key one, in my opinion. For example in C#, if you write a function that takes an array of numbers and performs a reduce to produce a single value, the function's signature should *not* be `int foo(int[] nums)`, it should be `int foo(IEnumerable<int> nums)`... And possibly shouldn't even specify int, but rather a more generic numeric type. (A freebie in dynamic languages.)


## Declaring values

`name: type`

```typescript
const x: number = 32;
const y = 32; // Works due to type-inference

// void methods have no return value
function say(message: string): void {
  alert(message);
}

```

## Interfaces

```typescript

// Here, we define a thing called a "Person" which has two properties.
export interface Person {
  readonly id: number; // you can mark things as readonly
  name: string;
  email?: string; // The ? means this is optional and can be omitted
}

// Here, we define a thing called a Chattable which can say things
export interface Chattable {
  say: (message: string) => boolean;
}

// Interfaces in TypeScript are implicitly implimented (structural typing)
// This is a Person
const foo = {id: 2, name: 'Tim', email: 'foo@bar.com'};

// So is this
const bar = {id: 1, name: 'Christine'}

// This is not, because it's missing name...
const baz = {id: 9, email: 'shmuk@baz.com'}

// This is a Chattable
const hoi = {
  say(message: string): boolean {
    alert(message);
    return true;
  }
}

// You can specify types in a terse way, like the following, but if you have
// more than 2 args, you really should define the types elsewhere
function sayHi(user: {name: string}): string {
  return `Hi, ${user.name}`;
}

sayHi(bar); // Works.
sayHi(baz); // Doesn't work


// You can specify that a dictionary is read-only 
interface UserMap {
  readonly[id: number]: User;
}

const users: UserMap = {1: {id: 1, name: 'Joe'}, 2: {id: 2, name: 'Shmo'}};

// Fails
users[1] = {id: 3, name: 'Jane'};

// Succeeds... the readonly is not deep
users[1].name = 'Chris';

```


## Tuples

```typescript

let x: [string, number];

x = ["Hello", 3]; // works
y = x[1] + 1;     // y = 4

x = [3, "Hello"]; // fails


```


## Enums

Super handy for saying, this thing can have these values and only these values.

```typescript
enum Role { Admin, Guide, Student };

interface User {
  name: string;
  role: Role;
}

const user = {
  name: 'Joe',
  role: Role.Admin
}

```


## Type Switching

In redux, you're often doing a switch on `action.type` in order to do something. Here's a contrived example (you wouldn't really have a differently named property for incAmount/decAmount most likely, but...):

```typescript

// count is a reducer...
function count(state, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + action.incAmount;
  case 'DECREMENT':
    return state - action.decAmount;
  default:
    return state;
  }
}

```

With TypeScript, you can do this in a typesafe way.

```typescript

interface Inc {
  type: 'INCREMENT';
  incAmount: number;
}

interface Dec {
  type: 'DECREMENT';
  decAmount: number;
}

// count is a reducer...
// Due to the fact that we are switching on type, TypeScript knows the
// type of action in each case (except default).
function count(state: number, action: Inc | Dec) {
  switch (action.type) {
  case 'INCREMENT':
    return state + action.incAmount;
  case 'DECREMENT':
    return state - action.decAmount; 
  default:
    return state;
  }
}


```


## Discriminated unions

When you want to say, this function takes either a foo or a bar, you use discriminated unions, designated by an `|`.

`function baz(thing: Foo | Bar): void { }`

These must then be handled with typeof or instanceof checks in order to safely cast them to their proper type.


## Intersections

Intersections are kind of the opposite of unions. Instead of saying, "this can be either an x or a y", an intersection says "this must be both an x and a y".

```typescript
// A user is both a person and a role
type User = Person & Role
````


## Casting

You can cast from one type to another using the `as` keyword. This is dangerous, as it is not checked by the compiler and leads to runtime errors in my experience. But sometimes, you gotsta do what you gotsta do.

`const y = x as number;`


## 


## Generics

TypeScript supports generic typing. This basically means, you can create a type which has properties whose types may change based on usage.


```typescript


interface Addable<T> {
  value: T;
  add: (x: T) => T;
}

// Addable<string>
const sayHello = {
  value: 'Hello, ',

  add(x: string): string {
    return this.value + x;
  },
};

// Addable<number>
const increment = {
  value: 1,

  add(x: number): number {
    return this.value + x;
  },
};

// Generic function can handle any type of Addable
function shtuff<T>(adder: Addable<T>, val: T): T {
  return adder.add(val);
}

console.log(shtuff(increment, 2)); // 3
console.log(shtuff(sayHello, 'world!')); // 'Hello, world!'

```

## Misc

You can specify that this type has these properties, and then a bunch of other properties that are dynamic:

https://basarat.gitbooks.io/typescript/content/docs/types/freshness.html




## Parting Thoughts


TypeScript supports immutability, but it is verbose and clunky, so it's unlikely to be used heavily. And there is no performant, idiomatic way to deal with immutable datatypes.


TypeScript's type system makes for good editor tooling and better refactoring. Many type errors are discovered at transpile time, rather than runtime.

Its type system is not as powerful as, say, F# or ReasonML, so it's important to define types properly, or you'll still end up with runtime errors. Often, I find myself adjusting my code to accomodate static typing. It's important to write code in such a way that misuse creates type errors.

It can take a little more work than dynamic systems, but the idea is that maintainability becomes easier. This is debatable. I used to think this and still lean that way. But I have definitely seen lots of systems where the type definitions themselves are convoluted and difficult to reason about. Still, you can rip things out and know (pretty precisely) all of the places that are affected.


## Resources

- https://www.typescriptlang.org/docs/handbook/basic-types.html
- https://basarat.gitbooks.io/typescript
- https://www.typescriptlang.org/docs/handbook/advanced-types.html
- https://learnxinyminutes.com/docs/typescript/






