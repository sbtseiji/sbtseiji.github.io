# Modernist - Automagic 🦄 Project Scaffolding with Code Generation.

Use Modernist to build generate code for your projects or as a tool for more complex code generation needs.

The CLI requires a file named `.modernistrc.js` anywhere in the directory tree you're in, e.g.: next to `package.json`.

```sh
yarn add modernist
```

## Example

```js
// .modernistrc.js
module.exports = {
  example({ name }) {
    return {
      [name]: ({ name }) => `Hello, ${name}!`,
    };
  },
};
```

Now when you run

```sh
modernist example --name World
```

The following structure will be generated (relative to `.modernistrc.js`):

```
.modernistrc.js
  └── World # with the contents: Hello, World!
```

It also supports async functions for template and structure generation.

More complex example can be found in this repository in [`.modernistrc.js`](./.modernistrc.js).
