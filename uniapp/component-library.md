# Component Library for UniApp

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
## Table of Contents

- [Differences from Vue](#differences-from-vue)
- [Typical Project Configuration](#typical-project-configuration)
  - [The `package.json` file](#the-packagejson-file)
  - [Export Vue Components](#export-vue-components)
  - [TypeScript Type Definitions](#typescript-type-definitions)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Differences from Vue

Normally when we are developing a Vue 3 component library, we will use [`vite`](https://vitejs.dev/) to generate the library code, and use [`vue-tsc`](https://www.npmjs.com/package/vue-tsc) to generate the `.d.ts` type definition files.

But it would be different to develop a component library for UniApp.
UniApp would use their own build system, combined with `@vue/compiler-sfc` et cetera, to generate code for different platforms with their own components and `VNode` implementations.

So we'll have to distribute the `.vue` files as-is, and let the users use UniApp's build system to generate the code for their own projects.

## Typical Project Configuration

Say we have a project named `uniapp-component-library` with the following structure:

```text
uniapp-component-library
├── src/
│   ├── components/
│   │   ├── Button.vue
│   │   └── index.js
│   ├── index.js
│   └── types/
├── package.json
└── tsconfig.json
```

You might noticed that we are using `*.js` rather than `*.ts` files, but there is a `tsconfig.json` file in the project root. That is because we should distribute `*.vue` files as-is, only the `*.js` files can be imported without any extra configuration.

### The `package.json` file

It is recommended to set the `type` in the `package.json` file to be `module` rather than default `commonjs`.
Since we are using `vite` to build all the code, ES Module is more suitable for vite.

Example `package.json`:

```json
{
  "name": "uniapp-component-library",
  "version": "0.0.0",
  "module": "src/index.js",
  "exports": {
    ".": {
      "import": "./src/index.js"
    },
    "./package.json": "./package.json"
  },
  "peerDependencies": {
    "@dcloudio/uni-app": "*",
    "@dcloudio/uni-ui": "*",
    "vue": "^3.2.0"
  },
}
```

As you can see, we should add `vue`, `@dcloudio/uni-app` and `@dcloudio/uni-ui` as peer dependencies, since we are using them in our components but we don't want to depend on them directly, and they are likely to be installed in the user's project.

### Export Vue Components

We can use the `export` keyword or `module.exports` to export the Vue components into the `src/index.js` file:

`src/components/index.js`

```js
import Button from './Button.vue'

export {
  Button
}
```

`src/index.js`

```js
export * from './components'
```

### TypeScript Type Definitions

As mentioned above, we can only use `*.js` files in our component library, but is it possible to provide type definitions for TypeScript users? The answer is an uppercase YES.

To achieve this, we should use `vue-tsc` to generate the type definitions files just like we do in a Vue 3 project:

```json
{
  "scripts": {
    "build:dts": "vue-tsc --emitDeclarationOnly --declaration --outDir types"
  }
}
```

> Note: You should make sure that `*.vue` files are listed in the `include` field in the `tsconfig.json` file.

Then we can use the `types` field in the `package.json` file to point to the generated type definitions files:

```json
{
  "types": "types/index.d.ts",
  "exports": {
    ".": {
      "types": "./types/index.d.ts",
      "import": "./src/index.js"
    }
  }
}
```

> Note: `types` in `exports` should always be before `import` or `require` so that TypeScript can find the type definitions. [Reference](https://www.typescriptlang.org/docs/handbook/esm-node.html#packagejson-exports-imports-and-self-referencing)

You might found that some of the types are resulted to `any` or `unknown` since we are using JS only. In this case, we can declare the types with JSDoc comments.

#### Component Props

```js
const props = defineProps({
  /**
   * The button type
   */
  type: {
    type: /** @type {'default' | 'compact'} */ (String), // use parentheses to coerce the type
    default: 'default'
  },
})
```

#### Template Refs

```js
const buttonRef = ref(/** @type {HTMLButtonElement | null} */ (null))
```

Wrap the components with `InstanceType` and `typeof` operator to get the correct type of the component instance:

```js
import { RouterLink } from 'vue-router'

const routerLinkRef = ref(/** @type {InstanceType<typeof RouterLink> | null} */ (null))
```

#### Define Complex Types with JSDoc

Complex types can be defined with `@typedef`:

Say we have a `Person` interface like this:

```ts
interface Person {
  name: string
  age?: number
}
```

We can define it with JSDoc like this:

```js
/**
 * @typedef {{ name: string; age?: number }} Person
 */
```

And then use it like this:

```js
/**
 * @type {Person}
 */
const person = {
  name: 'John'
}
```
