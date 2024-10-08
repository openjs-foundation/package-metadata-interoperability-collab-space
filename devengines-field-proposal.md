# Proposal: `package.json` `devEngines` Field

This is a specification a `package.json` field to define the runtime and package manager for _developing_ a project or package (_not_ consuming/installing it as a dependency in another project). The field is named `devEngines` and is a new top-level field defined with this schema:

```typescript
interface DevEngines {
  os?: DevEngineDependency | DevEngineDependency[];
  cpu?: DevEngineDependency | DevEngineDependency[];
  libc?: DevEngineDependency | DevEngineDependency[];
  runtime?: DevEngineDependency | DevEngineDependency[];
  packageManager?: DevEngineDependency | DevEngineDependency[];
}

interface DevEngineDependency {
  name: string;
  version?: string;
  onFail?: 'ignore' | 'warn' | 'error' | 'download';
}
```

The `os`, `cpu`, `libc`, `runtime`, and `packageManager` sub-fields could either be an object or an array of objects, if the user wants to define multiple acceptable OSes, CPUs, C compilers, runtimes or package managers. The first acceptable option would be used, and `onFail` would be triggered for the last defined option if none validate. If unspecified, `onFail` defaults to `error` for the non-array notation; or it defaults to `error` for the last element in the array and `ignore` for all prior elements, for the array notation. Validation would check the name and version ranges.

The `name` field would be a string, corresponding to different sources depending on the parent field:

 - `os`: [NPM docs for `engines.os`](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#os).
 - `cpu`: [NPM docs for `engines.cpu`](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#cpu).
 - `libc`: `glibc` or `musl`; see [NPM docs for `config.libc`](https://docs.npmjs.com/cli/v10/using-npm/config#libc).
 - `runtime`: [WinterCG Runtime Keys](https://runtime-keys.proposal.wintercg.org/).
 - `packageManager` . . . I don’t know, we could define the list as part of the spec, or perhaps it would need to correspond to an npm registry package name. Suggestions welcome.

The `version` field syntax would match that defined for [`engines.node`](https://docs.npmjs.com/cli/v10/configuring-npm/package-json#engines), so something like `">= 16.0.0 < 22"` or `">= 20"`. If unspecified, any version matches.

The `onFail` field defines what should happen if validation fails:

 - `ignore`: nothing.
 - `warn`: print something and continue.
 - `error`: print something and exit.
 - `download`: remediate the validation failure by downloading the requested tool/version.

In the event of `onFail: 'download'`, it would be the responsibility of the tool to determine what and how to download, perhaps by looking in the tool’s associated lockfile for a specific version and integrity hash. It could also be supported on a case-by-case basis, like perhaps Yarn and pnpm could support downloading a satisfactory version while npm would error.

## Examples

### Typical example

```json
"devEngines": {
  "runtime": {
    "name": "node",
    "version": ">= 20.0.0",
    "onFail": "error"
  },
  "packageManager": {
    "name": "yarn",
    "version": "3.2.3",
    "onFail": "download"
  }
}
```

### “Uses every possible field” example:

```json
"devEngines": {
  "os": {
    "name": "darwin",
    "version": ">= 23.0.0"
  },
  "cpu": [
    {
      "name": "arm"
    }, {
      "name": "x86"
    }
  ],
  "libc": {
    "name": "glibc"
  },
  "runtime": [
    {
      "name": "bun",
      "version": ">= 1.0.0",
      "onFail": "ignore"
    },
    {
      "name": "node",
      "version": ">= 20.0.0",
      "onFail": "error"
    },
  ],
  "packageManager": [
    {
      "name": "bun",
      "version": ">= 1.0.0",
      "onFail": "ignore"
    },
    {
      "name": "yarn",
      "version": "3.2.3",
      "onFail": "download"
    }
  ]
}
```

## Future Expansions

Some potential future expansions of this spec that have been discussed are:

 - `runtime` and `packageManager` might take shorthand string values defining the desired name or name with version/version range.

## References

- Inspiration: https://github.com/nodejs/node/issues/51888#issuecomment-1967102442

- Initial discussion: https://github.com/openjs-foundation/package-metadata-interoperability-collab-space/issues/15

## Implementations

- https://github.com/npm/npm-install-checks/pull/116

- https://github.com/npm/cli/pull/7766
