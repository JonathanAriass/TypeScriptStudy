## Property Modifiers
### Optional Properties
We can read from optional properties, but when we do under strictNullChecks, TypeScript will tell us they're potentially `undefined`.
```ts
interface PaintOptions {
	shape: Shape;
	xPos?: number;
	yPos?: number;
}

function getShape(opts: PaintOptions) {
	let xPos = opts.xPos; // (property) PaintOptions.xPos?: number | undefined
	let yPos = opts.yPos; // (property) PaintOptions.yPos?: number | undefined
}

// Here destructuring pattern is applied and default values are provided to optional xPos and yPos
function paintShape({ shape, xPos = 0, yPos = 0 }: PaintOptions) {
	console.log("x coordinate at", xPos);	
	console.log("y coordinate at", YPos);
}
```

### `readonly` Properties
Properties can also be marked as `readonly` for TypeScript. Makes property marked as `readonly` unwritable during type-checking:
```ts
interface SomeType {
	readonly prop: string;
}

function doSomething(obj: SomeType) {
	let proper = obj.prop;

	// Can't re-assign it
	obj.prop = "hello";
}
```

`readonly` properties can change via aliasing.
```ts
interface Person {
	name: string;
	age: number;
}

interface ReadOnlyPerson {
	name: string;
	age: number;
}

let writablePerson: Person = {
	name: "Person",
	age: 42
}

let readOnlyPerson: ReadOnlyPerson = writablePerson;

writablePerson.age++; // works
readOnlyPerson.age++; // Cannot assign to 'age' because it is a read-only property
```