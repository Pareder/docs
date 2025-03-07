---
id: generator
title: Generator
slug: /aspect/generator
description: Generator extension enable generating new components by pre-defined templates
---

Generator extension enable generating new components by pre-defined templates

### Component location

Component location in the workspace directory tree is defined with the `bit create` command (see below). For example, a component named `ShoppingCart` created in the `acme.shopper` and the namespace `ui` will be generated in the following directory:

```
acme.shopper/ui/shopping-cart
```

### Component name

When using templates Bit will use CamelCasing for passing the component-name to the template, but generated file structure should be in kebab-case, for better cross-operating-system compatibility of the component.

### Automatically `add` component

Bit should automatically register the new component to the `.bitmap` file with a symmetrical name to the component-location in the workspace.

## Register a template

Any aspect (include envs) can register templates. Each template should have a name and a list of files. Each file has a relative-path to the component-dir and template content. See the `component-template.ts` file for more info about the exact API.

- Component name should be available as a param for the file-content-template.
- TBD: An environment must have a default template (if not set, use first template in array?).

To register a template, use the Generator API: `registerComponentTemplate(templates: ComponentTemplate[])`.

To make the templates of an aspect available on a workspace, they need to be added to the workspace.jsonc. For example:

```json
"teambit.generator/generator": {
    "aspects": [
      "teambit.harmony/aspect"
    ]
  },
```

In the example above, the aspect `teambit.harmony/aspect` is configured to be available for the generator.

## Show all available templates

Introduce a new command `bit templates`, which groups all available templates by aspects.

## Generate a template from CLI

Introduce a `create` command to use templates.

```sh
bit create <template-name> <component-name...> [--scope | -s] [--namespace | -n] [--aspect | -a]
```

### Args

#### `<component name>`

Name of the component to create. Will be used as the component's dir name and fed to the component template.

**generated file structure should use kebab-case, while the template itself be in camel case**.

### Options

#### `[--scope | -s]`

Sets the component's scope and base directory. If not defined, use the `defaultScope` from `teambit.workspace/workspace` config.

#### `[--namespace | -n]`

Sets the component's namespace and nested dirs inside the scope. If not define, use empty string.

#### `[--aspect | -a]`

Aspect ID that registered this template, required only if there are two templates with the same name from several aspects in the workspace.

## Creating a custom template generator

create a custom template generator using the generator command and add a name.

```bash
bit create generator <my-components>
```

### Configuring your Generator

Edit your `workspace.jsonc` file and set this component to use the `teambit.harmony/aspect` env under the variants object.

```json
"teambit.workspace/variants": {
  "{my-components}": {
    "teambit.harmony/aspect": {}
  },
}
```

To check if your component is using the correct env you can run `bit envs` or `bit show my-components`

Edit your `workspace.jsonc` file and add the component id, (scope name / component name) to teambit generator. This should go at root level. The component id can be found in the `aspect.ts` file:

```bash
{
  "teambit.generator/generator": {
      "aspects": ["my-scope-name/my-components"]
    },
}
```

This registers the component aspect so that your templates will appear in the CLI when you run `bit templates`.

### Modifying your Generator

The `my-components.main.runtime.ts` file contains an array of templates that you can modify and add to to create different templates and numerous files to be generated.

### Using your Generator

Use your generator to create the component files

```bash
bit create component1 button
```

### Exporting your Generator

Tag and export your generator component so you can use it in any other workspace. Make sure the scope name is set correctly in the `aspect.ts` file before tagging and exporting. Once you have tagged and exported the component you can add it to the `workspace.jsonc` file in the workspace where you want to use this generator.

```bash
{
  "teambit.generator/generator": {
      "aspects": ["my-scope-name/my-components"]
    },
}
```

Run `bit templates` to see your generator and then use it to generate you components.
