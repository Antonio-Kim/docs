## Introduction to TypeScript

**NOTE! Lots of gibberish here in this TypeScript folders. These are self-note, not full documentation so there might not be some sort of logical explanation. Treat it like scratch pad of notes**

### TypeScript Setup

Open an empty folder and add a package.json like this:

```json
{
  "name": "tsc-play",
  "dependencies": {
    "typescript": "^4.6.4"
  },
  "scripts": {
    "build": "tsc src/product.ts"
  }
}
```

Type "npm install" to install typescript in the directory. Now create a folder named /src and add product.ts file inside. Add some typescript code and then run npm run build. A JavaScript file mirroring the typescript file will appear.

You can also configure the TypeScript transpiler by adding tsconfig.json. These are the common setup for the transpiler

```json
{
  "compilerOptions": {
    "outDir": "build",
    "target": "esnext",
    "module": "esnext",
    "lib": ["DOM", "esnext"],
    "strict": true,
    "jsx": "react",
    "moduleResolution": "node",
    "noEmitOnError": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "build"]
}
```

Once you have the tsconfig.json on, you can remove the configuration inside package.json:

```json
{
  "name": "tsc-play",
  "dependencies": {
    "typescript": "^4.6.4"
  },
  "scripts": {
    "build": "tsc"
  }
}
```

### Random notes

- Type Predicate: can return once the type is narrowed down to the specified type. It looks like this:

```TypeScript
function isCharacter(character : any) : character is { name : string} {
	return "name" in character
}
```

it takes the character as parameter, type of "any". It then checks whether the character variable has value of name with type string. Only then the return type is executed.

- Type Alias: creating your own set of types for brevity. Use `type` to set your own custom type. It can also be a function

```TypeScript
type Table = { name: string, unitPrice?: number };
type Purchase = (quantity: number) => void;
type Product = {
    name: string,
    unitPrice?: number,
    purchase: Purchase
};

let product: Product = {
    name: "Table",
    purchase: (quantity) => {
        console.log(`Purchased ${quantity} tables`);
    }
};
product.purchase(4);
```

Type Alias are very similar to interfaces. So when should you use Type Alias and when should you use Interface? Stick with one and set that as a standard.

- Classes: you can declare fields in your class without declaring them, so long as you have `public` in the constructor parameters and then set the body of the constructor as below:

```TypeScript
class Product {
    constructor(public name: string, public unitPrice: number)
    {
        this.name = name;
        this.unitPrice = unitPrice;
    }
}
```

- Like string-based union types, string-based enumerations are great for a specific set of strings. A string union type is the simplest approach if the strings are meaningful. If the strings aren’t meaningful, then a string enumeration can be used to make them readable.
