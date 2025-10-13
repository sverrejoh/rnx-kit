# @rnx-kit/eslint-plugin

[![Build](https://github.com/microsoft/rnx-kit/actions/workflows/build.yml/badge.svg)](https://github.com/microsoft/rnx-kit/actions/workflows/build.yml)
[![npm version](https://img.shields.io/npm/v/@rnx-kit/eslint-plugin)](https://www.npmjs.com/package/@rnx-kit/eslint-plugin)

`@rnx-kit/eslint-plugin` is a set of configurations and rules that can be used
as is, or extended in your own ESLint config.

Note that this plugin requires the
[new ESLint configuration format](https://eslint.org/blog/2022/08/new-config-system-part-2/).
If you still rely on the previous format, use version 0.5.x instead.

## Install

```
yarn add @rnx-kit/eslint-plugin --dev
```

or if you're using npm:

```
npm add --save-dev @rnx-kit/eslint-plugin
```

## Usage

This ESLint plugin exports multiple configurations. For instance, to use the
[`recommended`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/configs/recommended.js)
configuration, you can re-export it in your
[flat config](https://eslint.org/docs/latest/use/configure/configuration-files-new)
like below:

```js
module.exports = require("@rnx-kit/eslint-plugin/recommended");
```

Alternatively, if you want to add customizations:

```js
const rnx = require("@rnx-kit/eslint-plugin");
module.exports = [
  ...rnx.configs.recommended,
  {
    rules: {
      "@rnx-kit/no-const-enum": "error",
      "@rnx-kit/no-export-all": "error",
      // Mores rules ...
    },
  },
];
```

If you're just interested in the rules, you can use it as a plugin and enable
the rules you're interested in:

```js
module.exports = [
  {
    plugins: {
      "@rnx-kit": require("@rnx-kit/eslint-plugin"),
    },
    rules: {
      "@rnx-kit/no-const-enum": "error",
      "@rnx-kit/no-export-all": "error",
      // Mores rules ...
    },
  },
];
```

## Recommended Configurations

- [`@rnx-kit/eslint-plugin/recommended`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/configs/recommended.js)
  extends:
  - [`eslint:recommended`](https://eslint.org/docs/rules/)
  - [`plugin:@typescript-eslint/recommended`](https://typescript-eslint.io/linting/configs#recommended)
  - [`plugin:react-hooks/recommended`](https://github.com/facebook/react/tree/main/packages/eslint-plugin-react-hooks#readme)
  - [`plugin:react/recommended`](https://github.com/yannickcr/eslint-plugin-react#recommended)
  - It also includes and enables the following rules:
    - [`@react-native/platform-colors`](https://github.com/facebook/react-native/tree/main/packages/eslint-plugin-react-native#readme)
- [`@rnx-kit/eslint-plugin/strict`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/configs/strict.js)
  extends `@rnx-kit/eslint-plugin/recommended` with rules that enables better
  tree shaking:
  - [`@rnx-kit/no-const-enum`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-const-enum.js)
  - [`@rnx-kit/no-export-all`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-export-all.js)
  - [`@rnx-kit/no-foreach-with-captured-variables`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-foreach-with-captured-variables.js)
  - [`no-restricted-exports`](https://archive.eslint.org/docs/rules/no-restricted-exports)

## Supported Rules

- ✓: Enabled with `@rnx-kit/eslint-plugin/recommended`
- 🔧: Fixable with `--fix`
- 💡: Provides suggestions in IDE/editor

|  ✓  | 🔧  | Rule                                                                                                                                                                   | Description                                                                                                                                                                                                                                                                                                     |
| :-: | :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|  ✓  |     | [`@rnx-kit/no-const-enum`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-const-enum.js)                                           | disallow `const enum` ([why is it bad?](https://hackmd.io/bBcd6R-1TB6Zq95PSquooQ))                                                                                                                                                                                                                              |
|  ✓  | 🔧💡 | [`@rnx-kit/no-export-all`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-export-all.js)                                           | disallow `export *` ([why is it bad?](https://hackmd.io/Z021hgSGStKlYLwsqNMOcg)) - also provides suggestions 💡 when auto-fix isn't safe                                                                                                                                                                        |
|  ✓  |     | [`@rnx-kit/no-foreach-with-captured-variables`](https://github.com/microsoft/rnx-kit/blob/main/packages/eslint-plugin/src/rules/no-foreach-with-captured-variables.js) | disallow `forEach` with outside variables; JavaScript is not efficient when it comes to using variables defined outside of its scope, and repeatedly calling that function can lead to performance issues. By using a `for...of` loop, you can avoid these performance pitfalls and also it is easier to debug. |

### `@rnx-kit/no-export-all` Options

This rule automatically expands `export *` statements into explicit named exports when the target module has named exports. When a module has no named exports (only default export or side effects), the rule provides suggestions by default.

#### `fixEmptyExports`

Controls how to handle `export *` from modules with no named exports. This is a common logical error where `export *` does nothing.

**Type:** `"suggest-only" | "import" | "export-default" | "remove"`
**Default:** `"suggest-only"`

**Options:**

- `"suggest-only"` (default) - Don't auto-fix, only provide IDE suggestions. Safest option that requires manual review.
- `"import"` - Auto-fix to `import "./module";` (preserves side effects, exports nothing)
- `"export-default"` - Auto-fix to `export { default } from "./module";` when default exists, falls back to `import` otherwise
- `"remove"` - Auto-fix by deleting the line (assumes no side effects needed)

**Examples:**

```javascript
// Default behavior - provides suggestions only
"@rnx-kit/no-export-all": "error"

// Auto-fix to import statements (preserves runtime behavior)
"@rnx-kit/no-export-all": ["error", { fixEmptyExports: "import" }]

// Auto-fix to export defaults (changes what gets exported)
"@rnx-kit/no-export-all": ["error", { fixEmptyExports: "export-default" }]
```

**When to use each option:**

- Use `"suggest-only"` (default) for codebases where you want to manually review each case
- Use `"import"` for bulk fixes when you want to preserve current runtime behavior
- Use `"export-default"` when you're confident the defaults should be exported
- Use `"remove"` when you're confident the imports are unnecessary dead code

**Note:** `export *` from a module with no named exports does nothing - it doesn't export anything and only runs the module's side effects. This is usually a logical error that should be fixed.
