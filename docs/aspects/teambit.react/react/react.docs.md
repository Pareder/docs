---
id: react
title: React
description: A Bit development environment for React Components
labels: ['react', 'environment', 'env', 'aspect', 'extension']
---

The built-in [React Component Development Environment](https://bit.dev/teambit/react/react) is a concrete composition of the [Env Aspect](https://bit.dev/teambit/envs/envs). It compose different tools and configs that fit practices implemented in [Create React App](https://create-react-app.dev). Use it when getting started with React components with Bit and later as a base for any future customization of your React-based workflow.

## Use React environment

To use this environment for your components, add it to any of the `variants` in your `workspace.jsonc` file as follows:

```json title="workspace.jsonc"
{
  "teambit.workspace/variants": {
    "some/path": {
      "teambit.react/react": {}
    }
  }
}
```

## Create React components

React implements several component templates:

- `react-component` a basic React UI component.
- `react-context` a react context component.
- `react-hook` a react hook component.
- `mdx-component` an MDX component.
- `react-component-js` a basic React component in JS.
- `react-env` boilerplate for a customized environment configuration.

Use any of these templates with the `bit create` command:

```sh
bit create <template name> [components...]
```

## Runtime (framework) dependencies

As with many Frontend frameworks React requires a singleton instance in your app's runtime. When building reuseable components that means setting `react` and `react-dom` as `peerDependencies`, thus allowing the consuming app to determine the runtime version. Bit's base React environment implements this via the **Dependencies** service which is used to override [dependency-resolver](https://bit.dev/teambit/dependencies/dependency-resolver) and set your preferred dependencies.
It is recommended for you to extend the base React environment and define a semantic version rule and range to fit your current tech stack and guidelines for reuseable React components.

## Development services

React, like all other Bit Environments must implement a set of Service Handlers. Each service corresponds to a specific tool or stage in the component life-cycle, with a default config as set by the base React environment:

| Service              | Aspect                                                      | Base Configuration                                                                                                                 |
| -------------------- | ----------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Compilation          | [TypeScript](https://bit.dev/teambit/typescript/typescript) | [tsconfig.json](https://bit.dev/teambit/react/react/~code/typescript/tsconfig.json)                                                |
| Testing              | **Jest**                                                    | [jest.config.js](https://bit.dev/teambit/react/react/~code/jest/jest.config.js)                                                    |
| Linting              | **ESLint**                                                  | [eslintrc.ts](https://bit.dev/teambit/react/react/~code/eslint/eslintrc.ts)                                                        |
| DevServer            | **Webpack**                                                 | [webpack.config.preview.dev.ts](https://bit.dev/teambit/react/react/~code/webpack/webpack.config.preview.dev.ts)                   |
| Preview (simulation) | **Webpack**                                                 | [webpack.config.preview.ts](https://bit.dev/teambit/react/react/~code/webpack/webpack.config.preview.ts)                           |
| Package              | **PKG**                                                     | Base `package.json` props from [TypeScript Aspect](https://bit.dev/teambit/typescript/typescript/~code/typescript.main.runtime.ts) |
| Bundling             | **Webpack**                                                 | [webpack.config.preview.ts](https://bit.dev/teambit/react/react/~code/webpack/webpack.config.preview.ts)                           |
| Documentation        | _Core implementation_                                       | [Docs template](https://bit.dev/teambit/react/react/~code/docs/index.tsx)                                                          |
| Build pipeline       | [Builder](https://bit.dev/teambit/pipelines/builder)        | [Build pipeline](https://bit.dev/teambit/react/react/~code/react.env.ts)                                                           |
| Dependencies         | _Core implementation_                                       | [Env-dependencies](https://bit.dev/teambit/react/react/~code/react.env.ts)                                                         |
| Component Generator  | [Generator](https://bit.dev/teambit/generator/generator)    | [example template](https://bit.dev/teambit/react/react/~code/templates/react-component.ts)                                         |

## Customize environment

All environments are extendible. You can take any pre-existing environment, and create a component to extend it, creating a customized environment. That customized environment can use APIs from the environment it is extending to:

- Override default configurations.
- Replace composed tools with others (for example - use Babel to transpile instead of TypeScript).
- Add new services and capabilities.

### Create an extension

The first step is to create a component that extends React. Use the `react-env` template from React env.

```sh
bit create react-env extensions/custom-react
```

As with any component, environments must themselves have an environment which compiles, builds, transpiles, etc, them. Bit has a special environment that knows how to create other environments - `teambit.harmony/aspect`. So for any component that is meant to be a Bit environment, you must set `teambit.harmony/aspect` as the environment for that component.

```json title="add variant for the extension and set the aspect env"
{
  //...
  "teambit.workspace/variants": {
    //...
    "extensions/custom-react": {
      "teambit.harmony/aspect": {}
    }
    //...
  }
}
```

Validate the above by running `bit env` and see that the extension-component has `teambit.harmony/aspect` set as its environment.

Now that you have a basic customized extension to start from, you can go ahead and configure it for your components in `workspace.json`:

```json title="edit variants and set the new env"
{
  //...
  "teambit.workspace/variants": {
    //...
    "[some]/[variant]": {
      "[yourscope]/extensions/custom-react": {}
    }
    //...
  }
}
```

### Customize configuration

React implements a set of APIs you can use to merge your preferred configuration with its defaults. These APIs are called **transformers** and they all start with the `override` pre-fix. Find [Available transformers here](#transformers-api-docs).
In case of a conflict, your config will override the default.

```typescript {4,14} title="Customized TypeScript configuration"
import { EnvsMain, EnvsAspect } from '@teambit/envs';
import { ReactAspect, ReactMain } from '@teambit/react';

const tsconfig = require('./typescript/tsconfig.json');

export class CustomReactExtension {
  constructor(private react: ReactMain) {}

  static dependencies: any = [EnvsAspect, ReactAspect];

  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const customReactEnv = react.compose([react.overrideTsConfig(tsconfig)]);

    envs.registerEnv(customReactEnv);

    return new CustomReactExtension(react);
  }
}
```

> To override any specific configuration it's recommended to create a separate config file with your customized configuration and import it into the main `.extension.ts` file where you can supply it to the relevant **transformers**. See the tsconfig example in the above snippet.

### Composing tools and services

Environments use Environment Services by implementing a special class of methods called Service Handlers.
The supplied transformers allow overriding of these services - for instance the `overrideCompiler` transformer allows overriding the environment's `getCompiler()` method.

The below example uses the `overrideCompiler` transformer to override the `getCompiler()` Service Handler to change the compilation service.

1. Add a Babel config file to your custom environment and import/require it in your `.extension` file
1. Create a new Babel compiler using the Babel aspect, and initialize it with the above babelConfig
1. Use the `compose` Env API to apply the compiler override transformer and add Babel as a transpiler in the environment

```typescript {3,5,10-12,15,17-18,22-23}
import { EnvsMain, EnvsAspect } from '@teambit/envs';
import { ReactAspect, ReactMain } from '@teambit/react';
import { BabelAspect, BabelMain } from '@teambit/babel';

const babelConfig = require('./babel/babel.config');

export class CustomReactExtension {
  constructor(private react: ReactMain) {}

  static dependencies: any = [EnvsAspect, ReactAspect, BabelAspect];

  static async provider([envs, react, babel]: [
    EnvsMain,
    ReactMain,
    BabelMain
  ]) {
    const babelCompiler = babel.createCompiler({
      babelTransformOptions: babelConfig
    });

    const customReactEnv = react.compose([
      react.overrideCompiler(babelCompiler),
      react.overrideCompilerTasks([babelCompiler.createTask()])
    ]);

    envs.registerEnv(customReactEnv);
    return new CustomReactExtension(react);
  }
}
```

<!-- ## Composition Providers

The React environment is able to "wrap" component compositions with an array of providers, each of which is simply a component which wraps its `children` with functionality, such as a context, styling, theme, etc.

When Bit renders your component compositions and previews, it wraps each one with with the providers you configure.
These providers can be used to render compositions with a theme or a react context such as an api context.

These provider components are registered using the `registerProvider`.

:::info
Providers are added by Bit to the component compositions and documentation bundle that is served by the environment's server and rendered by the browser.
As such, they run in the environment's Preview runtime and not the Main runtime.
:::

For example, a provider that centers compositions in their rendering page, will look like this:

```tsx title="A composition provider example"
import React, { ReactReact, ReactElement } from 'react'

const style = {
  display: 'flex',
  justifyContent: 'center',
  alignItems: 'center',
  height: '100vh'
}

export const Center = ({
  children
}: {
  children: ReactReact
}): ReactElement => {
  return <div style={style}>{children}</div>
}
```

This `Center` provider component will be registered using the registerProvider method in the React extension's `*.preview.runtime.tsx` file:

```tsx title="react-with-providers.preview.runtime.tsx"
import { PreviewRuntime } from '@teambit/preview'
import { ReactAspect, ReactPreview } from '@teambit/react'
import { ReactWithProvidersAspect } from './react-with-providers.aspect'
import { Center } from './composition-providers/center'

export class ReactWithProvidersPreview {
  static runtime = PreviewRuntime
  static dependencies = [ReactAspect]

  static async provider([react]: [ReactPreview]) {
    react.registerProvider([Center])

    return ReactWithProvidersPreview
  }
}

ReactWithProvidersAspect.addRuntime(ReactWithProvidersPreview)
```

> See the full demo project [here](https://github.com/teambit/react-env-with-providers). -->

### Transformers API docs

Use these APIs to customize the base React environment default configuration with your extension. [Read more here](#customizing-configuration).

#### `overrideTsConfig(tsconfig: TsConfigSourceFile): EnvTransformer`

Merges the environment's default TypeScript configuration with the contents of your [typescript configuration file](https://www.typescriptlang.org/handbook/tsconfig-json.html).
This config affects the transpilation carried out when rendering components on the local dev UI. To override the config used when building components, use `overrideBuildTsConfig` below.

```ts
// ...
const tsconfig = require('./typescript/tsconfig.json');
export class ReactExtension {
// ...

  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overrideTsConfig(tsconfig)
    ]);
}
// ...
```

#### `overrideBuildTsConfig(tsconfig: TsConfigSourceFile): EnvTransformer`

Merges the environment's default TypeScript configuration with the contents of your [typescript configuration file](https://www.typescriptlang.org/handbook/tsconfig-json.html).
This config affects transpilation of component code during the build process, i.e. in preparation for packaging and exporting the component. To override the local ts config used when rendering components in the local UI, please use `overrideTsConfig` above.

```ts
// ...
const tsconfig = require('./typescript/tsconfig.json');
export class ReactExtension {
// ...

  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overrideTsConfig(tsconfig)
    ]);
}
// ...
```

#### `overridePreviewConfig(config: Configuration): EnvTransformer`

Merges the Webpack configuration for the 'Preview' environment service, with the contents of your [webpack configuration file](https://webpack.js.org/configuration/).
This webpack configuration is used for building and rendering your components on the local dev server.

```ts
// ...
const webpackConfig = require('./webpack/webpack.config');
export class ReactExtension {
// ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overridePreviewConfig(webpackConfig)
    ]);
}
// ...
```

#### `overrideDevServerConfig(config: Configuration): EnvTransformer`

Merges the Webpack configuration for the 'DevServer' environment service, with the contents of your [webpack configuration file](https://webpack.js.org/configuration/).
This configuration is used for building and rendering the dev server UI (but not your components, which use the PreviewConfig)

```ts
// ...
const webpackConfig = require('./webpack/webpack.config');
export class ReactExtension {
// ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overrideDevServerConfig(webpackConfig)
    ]);
}
// ...
```

#### `overrideJestConfig(jestConfigPath: string): EnvTransformer`

Merges the default configuration for the Jest test runner with the contents of your ([jest.config configuration file](https://jestjs.io/en/configuration)).

```ts
// ...
const jestConfig = require.resolve('./jest/jest.config');
export class ReactExtension {
// ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overrideJestConfig(jestConfig)
    ]);
}
// ...
```

#### `overrideBuildPipe(tasks: BuildTask[]): EnvTransformer`

This method receives an array of build tasks. It merges the provided tasks with the environment's default build pipeline (initiated either on a `bit tag` or `bit build` command).

```ts
// ...
// Import the task
import { CustomTask } from './custom.task';
export class CustomReact {
  // ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    // Get the environment's default build pipeline using the 'getBuildPipe' service handler
    const reactPipe = react.env.getBuildPipe();
    // Add the custom task to the end of the build tasks sequence.
    const tasks = [...reactPipe, new CustomTask()];
    const newReactEnv = react.compose([react.overrideBuildPipe(tasks)]);
    // ...
  }
}
```

#### `overrideDependencies(dependencyPolicy: DependenciesPolicy): EnvTransformer`

This method receives a dependency-policy object and merges it with the environment's default dependency policy for components using this environment.
Each key-value pair in a dependency-policy object signifies the package and the version to be used. Use also the `-` and `+` notation to signify a module should moved between dependency types (dev, peer or standard).

```ts
// ...
const newDependencies = {
  dependencies: {
    react: '-'
  },
  devDependencies: {
    '@types/jest': '~26.0.9'
  },
  peerDependencies: {
    react: '^17.0.2'
  }
};

export class CustomReact {
  // ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overrideDependencies(newDependencies)
    ]);
    // ...
  }
}
```

> The above example shows the 'React' library being removed as a (runtime) dependency and added as a peer dependency.

#### `overridePackageJsonProps(props: PackageJsonProps): EnvTransformer`

Merges the provide props with the default properties added to the `package.json` file of every package generated from components using this environment.

```ts
// ...
const newPackageProps = {
  main: 'dist/{main}.js',
  types: '{main}.ts'
};

export class CustomReact {
  // ...
  static async provider([envs, react]: [EnvsMain, ReactMain]) {
    const newReactEnv = react.compose([
      react.overridePackageJsonProps(newPackageProps)
    ]);
    // ...
  }
}
```
