---
id: variants
title: Variants
---

`teambit.workspace/variants` enables cascading configurations by selection of component groupings or sets in the workspace and applying configurations to these cascading groups.
Configurations set on a certain set of components can:

1. Affect only the selected set of components
2. Inherit policies set on a more general set of components (including the workspace default configs)
3. Override conflicting configurations inherited from more general component selections
4. Propagate configurations downwards to more specific sub-sets of components

## Variants Selector Examples

### Select a rule set by directory

To select a set using a directory path, use the relative path to the components' parent directory from the workspace root. In the following example, all components under the `components/utility-functions` directory
(and any sub-directories) will be included in this set:

```json
"teambit.workspace/variants": {
    "components/utility-functions": {
        "teambit.harmony/node": {}
    },
}
```

### Select a rule set via namespace

This option is recommended as it decouples your components' configurations from the workspace's file structure. It handles components using fundamental definitions that pertain to function and purpose, via their namespace.
The namespace selector behaves like a glob pattern, with the component name (including its namespace) being the equivalent of a file path being matched against the pattern.
Specifically, this means that namespace selectors support the location-specific `*` matcher, and the 'anywhere' `**` matcher for matching the component name.

In the following example, any component under the `utility-functions` namespace (and it's sub-namespaces) will be included in this rule set:

```json
"teambit.workspace/variants": {
    "{utility-functions/**}": { // Match any component name starts with utility-functions
        "teambit.harmony/node": {}
    },
}
```

In the following example however, only components **directly** under the `utility-functions` namespace will be included in this rule set:

```json
"teambit.workspace/variants": {
    "{utility-functions/*}": { // Match utility-functions/sort-array but not utility-functions/string/reverse
        "teambit.harmony/node": {}
    },
}
```

### Grouping selectors

You can add several sets for the same variant configuration by grouping selectors together:

```json title="Multiple directory paths"
"teambit.workspace/variants": {
    "components/utils, components/react-ui": {
        "teambit.harmony/node": {}
    },
}
```

```json title="Multiple namespaces"
"teambit.workspace/variants": {
    "{utility-functions/**}, {react-ui/**}": {
        "teambit.harmony/node": {}
    },
}
```

```json title="Paths and namespaces"
"teambit.workspace/variants": {
    "{utility-functions/**}, {react-ui/**}, components/utils, components/react-ui": {
        "teambit.harmony/node": {}
    },
}
```

### Exclude directories/components from a rule

Using the `!` deselector you can exclude a set of components from a selector.
The `!` deselector works both for directories and namespaces, for example:

#### Exclude a sub-directory directory from a rule

For example, apply the `teambit.harmony/node` environment on the `utility-functions` set, but exclude the `utility-functions/react-utils` folder from that set:

```json title="workspace.json
"teambit.workspace/variants": {
    "components/utility-functions, !components/utility-functions/react-utils": {
        "teambit.harmony/node": {}
    },
}
```

#### Exclude namespaces from a rule

The following example applies the `teambit.harmony/node` environment on every component under the `utils` namespace, but excludes the `utils/react` namespace and its children from this set:

```json title="workspace.json
"teambit.workspace/variants": {
    "{utils/**}, !{utils/react/**}": {
        "teambit.harmony/node": {}
    },
}
```

### Special Variants

#### The Wildcard (\*) variant

To select all components in your workspace use the wildcard variant `*`. This is useful when you want to apply very general configurations, especially default or backup configurations,
on all components. Using this selector can produce unexpected consequences if the rules aren't general enough, so we recommend using this selector sparingly!
For example:

```json
"teambit.workspace/variants": {
    "*": {
        "teambit.harmony/node": {}
    },
}
```

## Merging Configurations

The same component may have several rules applied to it, including several versions of the same configuration. This works very much like CSS rules where rules cascade
but the more specific variant "wins" when there are rule 'conflicts'.

The following example shows how Bit does not apply `aspect1-components-key` nor the `aspect1-root-key` for components under the `components/ui` directory, as the `my-aspect1` extension
was re-set by a more specific variant.

```json title="workspace.json
{
  "teambit.workspace/variants": {
    "*": {
      "my-aspect1": {
        "aspect1-root-key": "aspect1-root-val"
      },
      "my-aspect2": {
        "aspect2-root-key": "aspect2-root-val"
      },
      "my-aspect4": {
        "aspect4-root-key": "aspect4-root-val"
      }
    },
    "components": {
      "my-aspect1": {
        "aspect1-components-key": "aspect1-components-val"
      },
      "my-aspect2": {
        "aspect2-components-key": "aspect2-components-val"
      }
    },
    "components/ui": {
      "my-aspect1": {
        "aspect1-components-ui-key": "aspect1-components-ui-val"
      },
      "my-aspect3": {
        "aspect3-components-ui-key": "aspect3-components-ui-val"
      }
    }
  }
}
```

```json title="components/ui/button's calculated configuration"
{
  "my-aspect1": {
    "aspect1-components-ui-key": "aspect1-components-ui-val"
  },
  "my-aspect2": {
    "aspect2-components-key": "aspect2-components-val"
  },
  "my-aspect3": {
    "aspect3-components-ui-key": "aspect3-components-ui-val"
  },
  "my-aspect4": {
    "aspect4-root-key": "aspect4-root-val"
  }
}
```

## Variants Flags

### Propagate

When using selectors which can propagate down to sub-sets, such as with directory selectors (where all sub-directories are included) or {namespace/\*\*} type selectors,
you can prevent this propagation for specific set of inheritors, by setting set the `propogate` value of an inheriting group of components to `false`.
Once bit sees `"propagate": false` it uses only the configuration for this set and does not inherit.

```json title="workspace.json
"teambit.workspace/variants": {
  "components/react": {
    "my-aspect2": {
      "aspect2-react-key": "aspect2-react-val"
    }
  },
  "components/react/ui": {
    "propagate": false, // take this config, and don't propagate parent configs down to here
    "my-aspect1": {
      "aspect1-react-ui-key": "aspect1-react-ui-val"
    }
  }
}
```

```json title="components/react/ui/button's calculated configuration"
{
  "my-aspect1": {
    "aspect1-react-ui-key": "aspect1-react-ui-val"
  }
}
```

## Removing aspects

> Note: Once a component has been tagged, any aspect configured for that component **can only** be removed from the component via the following `remove` method. (if you haven't exported yet then `untag` would reset the effect of the tag)

There are numerous scenarios where you would not want a specific aspect to be defined on a subgroup but you don't want to exclude the sub-group from upstream rules, or use the `propagate: false` flag, since you want to receive the
other configurations from the parent group rule/s.

In that case, removing a specific aspect can be achieved using `"-"` as the value for an aspect's configuration. This will remove this aspect from the current component set.

For instance, the following will remove `my-aspect2` from components in the `components/react/ui` set, while still inheriting other configs such as the `my-aspect3` aspect.

```json title="workspace.json
"teambit.workspace/variants": {
  "components/react": {
    "my-aspect2": {
      "aspect2-react-key": "aspect2-react-val"
    },
    "my-aspect3": {
      "aspect3-react-key": "aspect3-react-val"
    }
  },
  "components/react/ui": {
    "my-aspect1": {
      "aspect1-react-ui-key": "aspect1-react-ui-val"
    },
    "my-aspect2": "-" // Remove my-aspect2 from this set's configuration
  }
}
```

```json title="components/react/ui/button's calculated configuration"
{
  "my-aspect1": {
    "aspect1-react-ui-key": "aspect1-react-ui-val"
  },
  "my-aspect3": {
    "aspect3-react-key": "aspect3-react-val"
  }
}
```
