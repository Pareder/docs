---
id: node
title: Node
description: A Bit development environment for Node Components
labels: ['node', 'environment', 'env', 'aspect', 'extension']
---

The built-in [Node Component Development Environment](https://bit.dev/teambit/harmony/node) is a concrete composition of the [Env Aspect](https://bit.dev/teambit/envs/envs). Use it when getting started with Node components with Bit and later as a base for any future customization of your Node-based workflow.

Node environment is composed out of the base [React Environment](https://bit.dev/teambit/react/react) with some specific overrides for dependency management.

## Use Node environment

To use this environment for your components, add it to any of the `variants` in your `workspace.jsonc` file as follows:

```json title="workspace.jsonc"
{
  "teambit.workspace/variants": {
    "some/path": {
      "teambit.harmony/node": {}
    }
  }
}
```

## Create Node components

Node implements several component templates:

- `node-env` boilerplate for customizing configuration.

Use any of these templates with the `bit create` command:

```sh
bit create <template name> [components...]
```

## Default Configuration and Services

Node, like all over Environments must implement a set of Service Handlers. For each service, Node compose a different tool and config by default.

> Node is a composition of the React environment with some specific modifications. Most of the links here direct to the actual configs in React environment.

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
| Build pipeline       | [Builder](https://bit.dev/teambit/pipelines/builder)        | [Build pipeline](https://bit.dev/teambit/react/react/~code/node.env.ts)                                                            |
| Dependencies         | _Core implementation_                                       | [Env-dependencies](https://bit.dev/teambit/harmony/node/~code/node.env.ts)                                                         |
| Component Generator  | [Generator](https://bit.dev/teambit/generator/generator)    | [example template](https://bit.dev/harmony/node/~code/templates/node-component.ts)                                                 |

## Customize environment

All environments are extendible. You can take any pre-existing environment, and create a component to extend it. That component can then use APIs to:

- Override default configurations.
- Replace composed tools with others (for example - use Babel instead of TypeScript).
- Add new services and capabilities.

### Create an extension

The first step is to create a component that extends Node. Use the `node-env` template from Node env.

```sh
bit create node-env extensions/custom-node
```

As with any component, environments must themselves have an environment which compiles, builds, transpiles, etc, them. Bit has a special environment that knows how to create other environments - `teambit.harmony/aspect`. So for any component that is meant to be a Bit environment, you must set `teambit.harmony/aspect` as the environment for that component.

```json title="add variant for the extension and set the aspect env"
{
  //...
  "teambit.workspace/variants": {
    //...
    "extensions/custom-node": {
      "teambit.harmony/aspect": {}
    }
    //...
  }
}
```

Validate the above by running `bit env` and see that the extension-component has `teambit.harmony/aspect` set as an environment.

Now that you have a basic customized extension to start from, you can go ahead and configure it for your components in `workspace.json`:

```json title="edit variants and set the new env"
{
  //...
  "teambit.workspace/variants": {
    //...
    "[some]/[variant]": {
      "[yourscope]/extensions/custom-node": {}
    }
    //...
  }
}
```

### Customize configuration

Node implements a set of APIs you can use to merge your preferred configuration with its defaults. These APIs are called **transformers** and they all start with the `override` pre-fix. Find [Available transformers here](#transformers-api-docs).
In case of a conflict, your config will override the default.

```typescript {4,14} title="Customized TypeScript configuration"
import { EnvsMain, EnvsAspect } from '@teambit/envs';
import { NodeAspect, NodeMain } from '@teambit/node';

const tsconfig = require('./typescript/tsconfig.json');

export class CustomNodeExtension {
  constructor(private node: NodeMain) {}

  static dependencies: any = [EnvsAspect, NodeAspect];

  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const customNodeEnv = node.compose([node.overrideTsConfig(tsconfig)]);

    envs.registerEnv(customNodeEnv);

    return new CustomNodeExtension(node);
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
import { NodeAspect, NodeMain } from '@teambit/node';
import { BabelAspect, BabelMain } from '@teambit/babel';

const babelConfig = require('./babel/babel.config');

export class CustomNodeExtension {
  constructor(private node: NodeMain) {}

  static dependencies: any = [EnvsAspect, NodeAspect, BabelAspect];

  static async provider([envs, node, babel]: [EnvsMain, NodeMain, BabelMain]) {
    const babelCompiler = babel.createCompiler({
      babelTransformOptions: babelConfig
    });

    const customNodeEnv = node.compose([
      node.overrideCompiler(babelCompiler),
      node.overrideCompilerTasks([babelCompiler.createTask()])
    ]);

    envs.registerEnv(customNodeEnv);
    return new CustomNodeExtension(node);
  }
}
```

### Transformers API docs

Use these APIs to customize the base Node environment default configuration with your extension. [Read more here](#customizing-configuration).

#### `overrideTsConfig(tsconfig: TsConfigSourceFile): EnvTransformer`

Merges the environment's default TypeScript configuration with the contents of your [typescript configuration file](https://www.typescriptlang.org/handbook/tsconfig-json.html).
This config affects the transpilation carried out when rendering components on the local dev UI. To override the config used when building components, use `overrideBuildTsConfig` below.

```ts
// ...
const tsconfig = require('./typescript/tsconfig.json');
export class NodeExtension {
// ...

  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overrideTsConfig(tsconfig)
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
export class NodeExtension {
// ...

  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overrideTsConfig(tsconfig)
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
export class NodeExtension {
// ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overridePreviewConfig(webpackConfig)
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
export class NodeExtension {
// ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overrideDevServerConfig(webpackConfig)
    ]);
}
// ...
```

#### `overrideJestConfig(jestConfigPath: string): EnvTransformer`

Merges the default configuration for the Jest test runner with the contents of your ([jest.config configuration file](https://jestjs.io/en/configuration)).

```ts
// ...
export class NodeExtension {
// ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overrideJestConfig(require.resolve('./jest/jest.config'))
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
export class CustomNode {
  // ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    // Get the environment's default build pipeline using the 'getBuildPipe' service handler
    const nodePipe = node.env.getBuildPipe();
    // Add the custom task to the end of the build tasks sequence.
    const tasks = [...nodePipe, new CustomTask()];
    const newNodeEnv = node.compose([node.overrideBuildPipe(tasks)]);
    // ...
  }
}
```

#### `overrideDependencies(dependencyPolicy: DependenciesPolicy): EnvTransformer`

This method receives a dependency-policy object and merges it with the environment's default dependency policy for components using this environment.
Each key-value pair in a dependency-policy object signifies the package and the version to be used. Use also the `-` and `+` notation to signify a module should moved between dependency types (dev, peer or standard).

```js
// ...
const newDependencies = {
  devDependencies: {
    '@types/jest': '~26.0.9'
  }
};

export class CustomNode {
  // ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overrideDependencies(newDependencies)
    ]);
    // ...
  }
}
```

> The above example shows the 'Node' library being removed as a (runtime) dependency and added as a peer dependency.

#### `overridePackageJsonProps(props: PackageJsonProps): EnvTransformer`

Merges the provide props with the default properties added to the `package.json` file of every package generated from components using this environment.

```ts
// ...
const newPackageProps = {
  main: 'dist/{main}.js',
  types: '{main}.ts'
};

export class CustomNode {
  // ...
  static async provider([envs, node]: [EnvsMain, NodeMain]) {
    const newNodeEnv = node.compose([
      node.overridePackageJsonProps(newPackageProps)
    ]);
    // ...
  }
}
```
