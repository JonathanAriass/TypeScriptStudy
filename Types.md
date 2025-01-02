## Defining Types
To create an object with an inferred type:
```ts
const user = {
	name: "Test",
	id: 0
}
```

Object's shape can be explicitly described using an `interface` declaration:
```ts
interface User {
	name: string;
	id: number;
}
```

Then the object can be defined using the declared interface:
```ts
const user: User = {
	name: "Test",
	id: 0
}
```

Since JavaScript support classes and object-oriented programming, so does TypeScript, interface declaration with classes can be used:
```ts
interface User {
	name: string;
	id: number;
}

class UserAccount {
	name: string;
	id: number;

	constructor(name: string, id: number) {
		this.name = name;
		this.id = id;
	}
}

const user: User = new UserAccount("Test", 0);
```

Interfaces can be used to annotate parameters and return values to functions:
```ts
function deleteUser(user: User) {
	// ...
}

function getAdminUser(): User {
	// ...
}
```

## Composing types
In TypeScript complex types can be created (unions and generics).

### Unions
With a union, you can declare that a type could be one of many types:
```ts
type MyBool = true | false;
```

A popular use-case for union types is to describe the set of `string` or `number` literals:
```ts
type WindowStates = "open" | "closed" | "minimized";
type PositiveOddNumberUnderTen: 1 | 3 | 5 | 7 | 9;
```

It also works with functions:
```ts
function getLength(obj: string | string[]) {
	return obj.length;
}
```

To learn the type of a variable, use `typeof`:
```ts
function wrapInArray(obj: string | string[]) {
	if (typeof obj === "string") {
		return [obj];
	}
	return obj;
}
```

### Generics
Generics  provide variables to types. An array without generics could contain anything. An array with generics can describe the values that the array contains.
```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

You can declare your own types that use generics:
```ts
interface Backpack<Type> {
	add: (obj: Type) => void;
	get: () => Type;
}

declare const backpack: Backpack<string>; // Backpack only works with strings

const obj = backpack.get();

backpack.add(23); // This will generate an error since the declare type is string
```

## Structural Type System
One of TypeScripts's core principles is that type checking focuses on the `shape` that values have (duck typing or structural typing).
```ts
interface Point {
	x: number;
	y: number;
}

function logPoint(p: Point) {
	console.log(`${p.x}, ${p.y}`);
}

const point = { x: 12, y: 26};
logPoint(point); // As point var has the same shape as Point interface it will log "12, 26"

const rect = { x: 12, y: 26, width: 30, height: 80};
logPoint(rect); // This will log "12, 26"

const color = { hex: "#FFFFF"};
logPoint(color); // This will give an error

class VirtualPoint {
	x: number;
	y: number;

	constructor(x: number, y: number) {
		this.x = x;
		this.y = y;
	}
}

const newPoint = new VirtualPoint(13, 26);
logPoint(newPoint); // This will log "12,26" as the shape is the same as the interface
```


## Literal Types
We can refer to specific strings and numbers in type positions.
```ts
let changingString = "Hello World";
changingString = "Hola mundo";
// As 'changingString' can represent any possible string, the type will be string
changingString; // let changingString: string

const constantString = "Hello world";
// As 'constantString' can only represent one string, this has a literal type representation
constantString // const constantString: "Hello world"
```

The most typical use-case for literal types comes handy when using union types:
```ts
function printText(s: string, alignment: "left" | "right" | "center") {
	// ...
}

printText("Hello world", "left");
printText("Hello world", "bottom"); // This will raise an error as alignment is not one of the literal type ("left", "right", "center")
```
This way we can define what are the valid inputs of a function or the valid returns of a function too.

They can be combined with non-literal types too:
```ts
interface Options {
	width: number;
}

function configure(x: Options | "auto") {
	//...
}

configure({width: 100});
configure("auto");
configure("automatic"); // This will raise an error
```

### Literal Inference
When initializing a variable with an object, TypeScript assumes that the properties will change its values later. For that reason in the following example the first call to function will not work (because it has general types - string, instead of literal types - "GET" | "POST"):
```ts
declare function handleRequest(url: string, method: "GET" | "POST"): void;

const req = { url: "https://example.com", method: "GET" };
// req.method: string
handleRequest(req.url, req.method); // This will not work because method is a string and does not match the literal type "GET" | "POST"
  
// req.method: "GET" (casted)
handleRequest(req.url, req.method as "GET"); // This will work as it cast the req.method into the literal type "GET"


const req2 = { url: "https://example.com", method: "GET" } as const;
// req2.method: "GET"
handleRequest(req2.url, req2.method); // This will work because req2.method is a literal type because of the const assertion
```

## Type predicates
To define a user-defined type guard, we need to define a function whose return type is a _type predicate_:
```ts
function isFish(pet: Fish | Bird): pet is Fish {
	return (pet as Fish).swim !== undefined;
}
```
`pet is Fish` is our type predicate. So any time `isFish` is called, TypeScript will _narrow_ the variable to that specific type:
```ts
let pet = getSmallPet(); // getSmallPet(): Fish | Bird

if(isFish(pet)) {
	pet.swim(); // pet: Fish
} else {
	pet.fly(); // pet: Bird
}
```

The `isFish` type guard can be used to filter an array of Fish | Bird and obtain an array of Fish:
```ts
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];

const underWater1: Fish[] = zoo.filter(isFish);
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];

// For more controll
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
	if (pet.name === "sharkey") return false;
	return isFish(pet);
});
```

## Discriminated unions
```ts
interface Shape {
	kind: "circle" | "square";
	radius?: number;
	sideLength?: number;
}

function getArea(shape: Shape) {
	if (shape.kind === "circle") {
		return Math.PI * shape.radius ** 2; 
		// 'shape.radius' is possibly 'undefined', this happens because it is an optional and it can't identify if it is a filled value
	}
}
```

To solve this we need to create a union type:
```ts
interface Circle {
	kind: "circle";
	radius: number;
}

interface Square {
	kind: "square";
	sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
	switch (shape.kind) {
		case "circle":
			return Math.PI * shape.radius ** 2; // shape: circle
		case "square":
			return shape.sideLength ** 2; // shape: Square
		default:
			const _exhaustiveCheck: never = shape;
			return _exhaustiveCheck; 
			// If we decided to add a new type to Shape, this will raise an error
			// as it will try to add never type to Triangle type for example.
	}
}
```

