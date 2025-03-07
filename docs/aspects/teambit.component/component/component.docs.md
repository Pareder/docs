---
id: component
title: Component
---

A [Bit component](https://bit.dev/teambit/component/component) is a super-set of a node package. In addition to its distributable code, a Bit component has everything it needs to be maintained and developed as an independent building block.

That includes its source code, version history, development environment configurations and documentation.

A Bit component is authored in a Bit workspace. It is where its configurations are set, where it is rendered in isolation and where it is being compiled and built.

When a component gets 'tagged', its release version gets stored in the workspace local scope as a completely isolated unit.
That means, all its configurations and built artifacts are encapsulated as a single, immutable and transferable object.
This object can then be exported and stored in a remote scope, hosted on a Bit server like [Bit.dev](https://bit.dev), where it is made available to other web projects, both as a Bit component but also as a node package.

Bit components 'imported' into a Bit workspace, can be modified, versioned and exported again.

## A Bit component common structure

Below is an example of a Bit component file structure. The types of files are determined by the environment in use although there are a few constants to be aware of:

- A documentation file (`*.docs.*`). That is a combination of an auto-generated and manually customized documentation for the component.
- 'Compositions' files (`*.compositions.*`). These are examples of that component - relevant only to UI components.
- A "barrel" or entry file (`index.ts`).

```bash
├── account/login-form           # root directory for storing all component files
    ├── index.tsx                # barrel/entry file to export modules
    ├── login-form.composite.tsx # examples of that component being used
    ├── login-form.docs.mdx      # documentation
    ├── login-form.spec.tsx      # tests
    └── login-form.tsx           # implementation
```

> All _internal dependencies_ of a component must be in the same directory (its styling, documentation, compositions, etc.)

## Stages in a Bit component's life

A Bit component goes through a number of stages before it can be used across repositories:

### 1. Authored & tracked in a Bit workspace

- A component is created and tracked by Bit.
- A package for that component is created in the workspace' `node_modules` directory (at this point this package can only be used internally).
- Configurations are set for the component using the `workspace.json` configuration file. That includes its development environment and dependency policies.
- A component is rendered in an isolated "controlled environment". These is done by authoring and rendering component 'compositions', which are instances of that component as it is being used in different context and variations.
- A component is documented (in addition to the auto-generated documentation created by its environment)

### 2. Versioned

A component gets a new release version. That happens only after it is built using its environment's build pipeline. The new release version can be determined locally or as part of a remote CI (after being 'suggested for new version release' by the local workspace).

### 3. Exported

A component's release version, encapsulated with all its configurations and built artifacts, gets uploaded to a Remote Scope (e.g, [Bit.dev](https://bit.dev)).

This exported version is now available to be used and maintained in other Bit Workspaces.

## Component Id

Each Bit component has a unique identifier with the following pattern:<br />
`<scope-owner>.<scope-name>/<namespace(s)>/<name>`. <br />
A component ID is generated when a component gets tracked by Bit for the first time.

> Note that not all Bit servers will have a 'scope-owner'

- **Scope** - The component's scope as applied by the `workspace.json` file. It can be a `scope` property as defined for the component's `variant` or the `defaultScope` configured to the `teambit.workspace/workspace` extension. `scope` is usually a combination of the scope owner and scope name (e.g, `<my-org><my-scope>`)

- **Namespaces** (optional) - Set with the `--namespace` or `-n` flag when adding the component (supports nesting - `--namespace nesting/namespace/yay`).
- **Name** - The name of the component, according to the component's root directory name.

Bit uses these IDs when listing or running operations and commands on components.

## Component package

Bit creates a package from each component in the workspace' root `node_modules` directory. This package contains the component's transpiled code for other components to import.  
The package name is defined by the component ID. However, Node supports a single forward slash (`/`) in a module name (to set the module scope). Bit uses the `<scope>` defined for the components as such (with the `/` separator). All other `/` between namespaces (if found) are translated to dots (`.`).

For example the component ID `my-org.design-system/component/base/button` will result in a module called `@my-org/design-system.components.base.button`.

> Why doesn't Bit keep the same module naming as node?
>
> The future of JavaScript development brings tools like ESM and Deno that have a different approach for module consumption with namespacing support. As Bit components needs to be future-proof to support additional tooling it keeps its own naming scheme that can be translated according to the consumption method.

## Component configurations

A component's configurations are set by the `workspace.jsonc` configuration file. These settings include rules and policies regarding its dependencies, the development environment to support it, the properties for its generated package, and so on.

A component can also have its configurations "ejected". That means it will stop inheriting its settings from the `workspace.jsonc` file. Instead, these will be managed using the `component.json` configuration file, located in its directory (this file will only appear after ejecting).

To eject a component, run the following:

```bash
bit eject-conf <component-id>
```

> Ejecting a component's configurations is not recommended! Use the `workspace.jsonc` instead, whenever possible.
