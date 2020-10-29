title: "Codemods"
controls: false
progress: true
numbers: true
theme: tkers/cleaver-theme-sunset
--

# Codemods

## Refactoring JavaScript with JavaScript

![background](bg.jpg)

### 🙋‍♂️ Tijn Kersjes

--

### What are codemods?

- Also called program **transformations**
- Metaprogramming
  - Automatise programming / refactoring
- Can be applied to your entire code base
- Modifies program using its **AST**

--

<h1 style="margin-top: 4em">
code ➡ ✨ ➡ code
</h1>

--

### A simple scenario

Function name is updated:

```diff
export {
-  foo
+  bar
}
```

Find and replace 💪

```diff
import {
-  foo
+  bar
} from './myLib.js'

// ...

- foo()
+ bar()
```

--

### Another simple scenario

Old React code:

```js
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isOpen: true };
  }
}
```

Let's modernise it to this:

```js
function MyComponent() {
  const [isOpen, setIsOpen] = useState(true);
}
```

--

### Another simple scenario (or is it? 😰)

Old React code:

```diff
- class MyComponent extends React.Component {
-   constructor(props) {
-     super(props);
-     this.state = { isOpen: true };
-   }
  }
```

Let's modernise it to this:

```diff
+ function MyComponent() {
+   const [isOpen, setIsOpen] = useState(true);
  }
```

--

### Code is not just text

No access to the **structure** of your code:

```js
import { foo as fooA } from "./myLib.js";
import { foo as fooB } from "./otherLib.js";

obj.foo = 42;

sendMessage("foo");
```

--

### Abstract Syntax Tree

![ast](ast.png)

--

### Abstract Syntax Tree

- **ESLint** reads (and reports issues)
- **Prettier** reprints (changing whitespaces)
- **Babel** transforms (JSX, TS)
- ...

[astexplorer.net](https://astexplorer.net)

--

### JSCodeshift

- AST transformation tool
- Wrapper around **Recast**
- Update React APIs

```js
export default function transform(file, api) {
  const j = api.jscodeshift;
  const root = j(file.source);

  // ...transform root

  return root.toSource();
}
```

--

### Hooks transformation

```js
export default function transformer(file, api) {
  const j = api.jscodeshift;

  return j(file.source)
    .find(j.ClassDeclaration, {
      superClass: {
        object: { name: "React" },
        property: { name: "Component" },
      },
    })
    .forEach((path) => {
      const component = j(path);
      const name = component.get("id").value;
      const initialState = component
        .find(j.MethodDefinition, { kind: "constructor" })
        .find(j.AssignmentExpression, {
          left: { property: { name: "state" } },
        })
        .get("right").value;

      const body = j.blockStatement([
        j.variableDeclaration("const", [
          j.variableDeclarator(
            j.arrayPattern([j.identifier("state"), j.identifier("setState")]),
            j.callExpression(j.identifier("useState"), [initialState])
          ),
        ]),
      ]);
      const func = j.functionDeclaration(name, [], body, false, false);
      component.replaceWith(func);
    })
    .toSource();
}
```

--

### Some important painpoints

- Documentation is scarce
- JSCodeshift API a bit confusing
- JavaScript is messy
  - A lot of edge cases
  - Focus on the 99%

--

### Super fast refactoring 🚀

- Wow-factor
- Massive codebase changes (atomically!)
- Reusable transformations
- Postpone design decisions

-- dark

# That's it for today 👋

- [astexplorer.net](https://astexplorer.net)
- [facebook/jscodeshift](https://github.com/facebook/jscodeshift)
- [reactjs/react-codemod](https://github.com/reactjs/react-codemod)
- [cpojer/js-codemod](https://github.com/cpojer/js-codemod)

![background](bg.jpg)
