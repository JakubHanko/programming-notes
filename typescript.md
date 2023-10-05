## Adding TS to existing solution

1. Add **tsconfig.json** file to the solution

```json
{
    "compilerOptions": {
        "allowJs": true,
        "checkJs": true,
        "outDir": "build",
        "target": "esnext",
        "noEmit": true,
    },
    "include": ["src/**/*"]
}
```

* **include** - scope of the project
* **compilerOptions** - TS settings
  * **allowJs** - allows JavaScript files to be imported inside your project, instead of just .ts and .tsx files.
  * **checkJs** - turns on type checking in .js files.
  * **target** - version of ECMAScript; defaults to "ES3". esnext results in the latest version of JavaScript.
  * **outDir** - folder where the transpiled files will be generated.
  * **noEmit** - if set to true, no output is generated. Useful if you want only type checking.

2. Run TS compiler
   
```bash
tsc
```

This compiles everything in the scope of the project into **.ts** counterparts -- the so-called transpilation.


### Installing third-party types

1. Search for **@types** at npmjs.com
2. Execute the installation in the root of the folder
3. Type declarations are found in the **node_modules/types** folder in the **.d.ts** files


## Basic type usage

### Primitive types

```typescript
let x: number
let y: string
let z: boolean
let d: Date
let a: string[]

x = "Hello" // Type 'string' is not assignable to type 'number' error
x = "Hello" as any // Discouraged
```

### Custom types

```typescript
interface Address {
    line1: string;
    city: string;
    zip: string;
}

interface Contact extends Address {
    id: number; // a property/field
    name: string;
    birthDate?: Date; // optional field
}

let contact: Contact = {
    id: 1,
    name: "Kubo",
    // and the address fields
};
```

**Note:** interfaces are only used at compile-tile to provide type checking. They are never available in the runtime code.

### Type aliases

```typescript
type ContactName = string
```

**Alias** provides an alternate name for an existing type. They can be used interchangeably with the type they alias. They add a little more meaning to the variable they describe.

### Enumerable types

```typescript
enum ContactStatus {
    Active, // at runtime, the value will be 0
    Inactive,
    New
}

let status: ContactStatus = ContactStatus.Active;
```

By default, enum's values are represented by numbers from 0 upwards. The values have to match in type; i.e. they have to all be numbers, or all strings.

**Note:** enums do get compiled along with the rest of the code and stay available even at runtime (unlike many other parts of TypeScript syntaxes that get stripped down at compile time).

### Functions

```typescript
function clone(source: Contact): Contact {
    return Object.apply({}, source);
}

function foo(source: Contact, bar: (source: Contact) => Contact): Contact {
    return bar(source);
}

interface Contact {
    // fields
    clone(name: string): Contact // a method 
}
```

### Generics

#### Generic functions

```typescript
function clone<T>(source: T): T {
    return Object.apply({}, source);
}

const a: Contact = { id: 123, name: "Homer Simpson" };
const b = clone(a); // b is assumed to be of type Contact

function clone<T1, T2>(source: T1): T2 {
    return Object.apply({}, source);
}

const c: Contact = { id: 123, name: "Homer Simpson" };
const d = clone<Contact, Contact>(c); // we have to explicitly give the type parameters since TS is unable to infer correctly the types
```

Generic type is a meta type -- a type that represents any other type that you decide to substitute in.

#### Generic constraint

```typescript
function clone<T1, T2 extends T1>(source: T1): T2 { // so-called "generic constraint";
    return Object.apply({}, source);
}

function getNextId<T extends { id: number }>(items: T[]): number {
    return items.reduce((max, x) => x.id > max ? x.id : max, 0) + 1
}
```

T2 has to at least match the properties of T1.

**Note:** T2 doesn't have to literally extend or derive from T1. It only has to match it and then add any other properties it likes.

**Note 2:** ```{ id: number }``` is a type as well. Therefore the extended type doesn't have to be an explicitly defined type.

#### Generic interfaces

```typescript
interface UserContact<TExternalId> {
    id: number;
    name: string;
    externalId: TExternalId;
    loadExternalId(): Task<TExternalId>;
}
```

## More complex type usage

### Union types

```typescript
interface Contact {
    id: number;
    birthDate: ContactBirthDate;
}

type ContactBirthDate = Date | string | number // union of types

type AddressableContact = Contact & Address // combines the two types together

type ContactStatus = "active" | "inactive" | "new" // alternative to enums
```

### Keyof operator

keyof is a type operator that takes an object type and produces a string or numeric literal union of its keys. The following type ```P``` is the same type as ```type P = "x" | "y"```:

```typescript
type Point = { x: number; y: number };
type P = keyof Point;
```

```typescript
function getValue<T, U extends keyof T>(source: T, propertyName: U) {
    return source[propertyName];
}

const value = getValue({ min: 1, max: 200 }, /* you can only put "min" or "max" here*/)
```


### Typeof operator

typeof is a type operator that you can use in a type context to refer to the type of a variable or property.

```typescript
const myType = { min: 1, max: 200 };

function save(source: typeof myType) {}
```

Probably better to use an explicitly created interface. However, this approach could be useful if we are dealing with types that are unknown to us until the runtime.

### Indexed access types

```typescript
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number

type I1 = Person[keyof Person]; // string | number | boolean
     
type AliveOrName = "alive" | "name";
type I2 = Person[AliveOrName]; // string | boolean
```

### Records

Records are useful if you want to extend the object with properties that were not available at the time of object's initialization. It provides type checking while still retaining some of the flexibility and dynamicism of the ```any``` type.

```typescript
let x: Record<string, string> = { name: "Bruce Wayne" };
x.number = 1234; // 'number' is not assignable to 'string'


let x: Record<string, string | number> = { name: "Bruce Wayne" };
x.number = 1234; // OK
```

Two generic parameters: possible property name types and possible property types.


```typescript
interface Query {
    matches(val): boolean;
}

function searchContacts(contacts: Contact[], query: Record<keyof Contact, Query>) {}
```

```keyof Contact``` allows the Record to only have the same properties as Contact.

#### Record type modifiers

There exist some utility types (wrappers) that allow us to extend from existing types (like record) but pick and choose which properties to include and exclude in your new type.

```typescript
type ContactQuery = Record<keyof Contact, Query>
type PartialContactQuery = Partial<ContactQuery>
type OmittedContactQuery = Omit<ContactQuery, "address" | "id">
type PickedContactQuery = Pick<ContactQuery, "id" | "name">
type RequiredContactQuery = Required<PartialContactQuery>
```

* **Partial** - new type from the wrapped type with all of its properties defined as optional (by default the Record sets them all as required)
* **Omit** - excludes certain properties from the cloned definition
* **Pick** - limits the allowable properties
* **Required** - all of the wrapped properties are required


### Mapped type

```typescript
interface Query<TProp> {
    matches(val: TProp): boolean;
}

type ContactQuery = {
    [TProp in keyof Contact]?: Query<Contact[TProp]>
}
```

```[ ... ]``` is a property indexor syntax. In this case, each Query object has been parametrized by the type of the respective Contact property. We now have full static typing in the query objects.


## Decorators

Metadata that we can add to our classes, methods and even getter and setter properties. On top of that it allows us to extend them with additional functionality -- adding behaviour to code without actually changing the code itself.

For example adding authorization logic or logging to an existing code. If written manually, it could be seen as a noise with no relation to the responsibilities of the respective class/method.


### Method decorator

```typescript
function authorize(target: any, property: string, descriptor: PropertyDescriptor) {
    const wrapped = descriptor.value; // the original behavior

    descriptor.value = function() { // rewrites the original method
        if (!currentUser.isAuthenticated()) {
            throw Error("...");
        }

        return wrapped.apply(this, arguments); // calls the original method
    }
}

class A {
    ...

    @authorize("ContactViewer")
    getContactById(id: number): Contact | null;
}
```

* ```target``` -- instance of the object to which the method belongs
* ```property``` -- name of the property that the decorator is applied to
* ```descriptor``` -- object containing the current metadata about the property

To allow decorator parameters, we need a **decorator factory** -- a function that creates decorators.

```typescript
function authorize(role: string) {
    return function authorizeDecorator(target: any, property: string, descriptor: PropertyDescriptor) {
        const wrapped = descriptor.value;

        descriptor.value = function() {
            if (!currentUser.isInRole(role)) {
                throw Error("...");
            }

            return wrapped.apply(this, arguments);
        }
    }
}
```

### Class decorator

```typescript
function freeze(constructor: Function) {
    Object.freeze(constructor);
    Object.freeze(constructor.prototype);
}

@freeze
class A {

}

function singleton<T extends { new(...args: any[]): {} }>(constructor: T) {
    return class Singleton extends constructor {
        static _instance = null;

        constructor(...args) { // spread operator to accept any and all parameters input into the ctor
            super(...args);

            if (Singleton._instance) {
                throw Error("...");
            }

            Singleton._instance = this;
        }
    }
}

@singleton
class B {

}
```

```constructor``` is the class's constructor function.

### Property decorator

```typescript
function auditable(target: object, key: string | symbol) {
    // get the initial value, before the decorator is applied
    let val = target[key];

    // then overwrite the property with a custom getter and setter
    Object.defineProperty(target, key, {
        get: () => val,
        set: (newVal) => {
            console.log(`${key.toString()} changed: `, newVal);
            val = newVal;
        },
        enumerable: true,
        configurable: true
    })
}

class A {
    @auditable
    private contacts: Contact[] = [];
}
```

* ```target``` - instance of the target object being decorated
* ```key``` - name of the property being decorated
 

 ## Modules

Without modules, the contents of the **.js** files are copy-pasted (loaded) into the page source. I.e. the order of the files matters and if we omit one of the files while the other depends on some part of it, our code probably won't work anymore. In case of shadowing, the last file where the object is referenced defines its value/properties.

Converting one of the files into a **.ts** file leads to a similar problem. We can no longer access the functions/whatever from the converted file. We need to export it in the original file and import it in the .ts file.

**Export** makes a function or variable defined in the module, available to other modules. export keyword is required for TypeScript to treat the corresponding file as a module.

**Import** imports symbols from a module.

```javascript
// utils.js

export function foo() {}
```

```typescript
// app.ts

import { foo } from "./utils" // local path or a name of module

const x = foo();
```

