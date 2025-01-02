## Function Type Expressions
The simplest way to describe a function is with a _function type expression_.
```ts
function greeter(fn: (a: string) => void) {
	fn("Hello World");
}

function printToConsole(s: string) {
	console.log(s);
}

greeter(printToConsole);
```

## Call Signatures
If we want to describe something callable with properties, we can write a _call signature_ in an object type:
```ts
type DescribableFunction = {
	description: string;
	(someArg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
	console.log(fn.description + " returned " + fn(6));
}

function myFunc(someArg: number) {
	return someArg > 3;
}
myFunc.description = "default description";

doSomething(myFunc);
```

## Construct Signatures
JavaScript functions can also be invoked with the `new` operator. These are refered as _constructors_ in TypeScript.
```ts
interface CallOrConstruct {
	(n?: number): string;
	new (s: string): Date;
}

function fn(ctor: CallOrConstruct) {
	// (parameter) ctor: CallOrConstruct
	// (n?: number) => string
	console.log(ctor(10));

	// (parameter) ctor: CallOrConstruct
	// new (s: string) => Date
	console.log(new ctor("10"));
}

fn(Date);
```

## Generic Functions
```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
	return arr[0]; // This way we ensure returned type is the same as array
}

const s = firstElement(["a", "b", "c"]); // s is type string

const n = firstElement([1, 2, 3]); // n is type number

const u = firstElement([]); // u is type undefined
```

### Inference
In the example above `Type` is inferred automatically by TypeScript. We can use multiple type parameters as well:
```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
	return arr.map(func);
}

// Parameter n is type string and parsed is type number[]
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```

### Constraints
Sometimes we want functions to operate with certain value types only. We can use _constraints_ for achieving this:
```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
	if (a.length >= b.length) {
		return a;
	} else {
		return b;
	}
}

// longerArray is type number[]
const longerArray = longest([1, 2], [1, 2, 3]);

// longerString is type 'alice' | 'bob' as they have to be the same type
const longerString = longest("alice", "bob");

const notOK = longest(10, 100); // number has no .length property
```

### Specifying Type Arguments
TypeScript can infer the intended type arguments in a generic call, but not always.
```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
	return arr1.concat(arr2);
}

// string is not assignable to type number (as this is the first type)
const arr = combine([1, 2, 3], ["hello"]);

// To solve this we can manually specify `Type`
const arr = combine<string | number>([1, 2, 3], ["hello"])
```

### Guidelines for Writing Good Generic Functions
#### Push Type Parameters Down
```ts
// In this function the returned type is infered from the parameter
function firstElement1<Type>(arr: Type[]) {
	return arr[0];
}

// As this function uses any for the constraint it will return `any` Type
function firstElement2<Type extends any[]>(arr: Type) {
	return arr[0];
}

```

#### Use Fewer Type Parameters
```ts
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
	return arr.filter(func);
}

// This function is much harder to use as user needs to specify Func type
function filter2<Type, Func extends (arg: Type) => boolean>(
	arr: Type[], 
	func: Func
): Type[] {
	return arr.filter(func);
}
```

#### Type Parameters Should Appear Twice
```ts
function greet<Str extends string>(s: Str) {
	// ...
}

// This is much better as generic type in function above is useless
function greet(s: string) {
	// ...
}
```

## Optional Parameters
```ts
// x parameter will have type number | undefined
function f(x?: number) {
	// ...
}
```

### Optional Parameters in Callbacks
When writing a function type for a callback, never write an optional parameter unless you intend to _call_ the function without passing that argument.
```ts
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
	for (let i=0; i < arr.length; i++) {
		callback(arr[i]);
	}
}

myForEach([1, 2, 3], (a, i) => {
	console.log(i.toFixed()); // 'i' is possible 'undefined'
});
```

## Function Overloads
In TypeScript, we can specify a function that can be called in different ways by writing _overload signatures_.
```ts
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;
function makeDate(mOrTimestamp: number, d? number, y?: number): Date {
	if (d !== undefined && y !== undefined) {
		return new Date(y, mOrTimestamp, d);
	} else {
		return new Date(mOrTimestamp);
	}
}

const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);

// this will not work as no overload expects 2 arguments
const d3 = makeDate(1, 3); 
```
The signature of the implementation (in this case the third signature) is not visible from the outside. When writing and overloaded function, you should always have two or more signatures above the implementation of the function.

A common mistake is not to follow implementation signature to define overload signatures:
```ts
function fn(x: string): string;
// Return type isn't right
function fn(x: number): boolean;
//This overload signature is not compatible with its implementation signature. This overload signature is not compatible with its implementation signature.
function fn(x: string | number) {
	return "oops";
}
```

### Write Good Overloads
```ts
function len(s: string): number;
function len(arr: any[]): number;
function len(x: any) {
	return x.length;
}

// This will not work as it does not know which overload to call because
// type is array | string
len(Math.random() > 0.5 ? "hello" : [0]); 

// To improve this we can remove both overloads and write a non-overloaded func
function len(x: any[] | string) {
	return x.length;
}
```

## Declaring `this` in a Function
TypeScript will infer what the `this` should be in a function via code flow analysis, but sometimes we need to control what `this` should be manually.
```ts
interface DB {
	filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function (this: User) {
	return this.admins; // this will ensure this refers to User (User.admins)
})
```
_Note: `function` is neeeded to get this behavior, as arrow functions doesn't bind their own `this`._

