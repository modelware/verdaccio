# Local npm registry for OML packages

A local [Verdaccio](https://verdaccio.org/) registry for testing `oml publish` and `oml install` on your own machine, without touching npmjs.org. One running registry serves every OML project: publish a method from one repository, install it into another.

## Prerequisites

Node.js 20 or later and the OML CLI (`npm install -g @oml/cli`). Verdaccio itself is fetched by `npx` on first run; nothing else to install.

## Start the registry

```bash
node registry.mjs
```

Leave it running. It serves http://localhost:4873, keeps its data in `.verdaccio/` (git-ignored), and logs npm in to the registry as a local user, so publishing needs no further setup. Ctrl-C stops it.

To use another port, set `OML_REGISTRY_PORT` (for example `OML_REGISTRY_PORT=4874 node registry.mjs`) and use that port in the commands below.

## Publish from an OML project

In the project that publishes a package (for example `sierra-method`):

```bash
mkdir -p build/dist
oml pack -o build/dist
oml publish -r http://localhost:4873 --dry-run
oml publish -r http://localhost:4873
```

A version can be published only once. To publish changes, raise `project.version` in the project's `.oml/settings.yml`.

## Install into an OML project

In a project that depends on the package (for example `fireforce6`):

```bash
oml install -r http://localhost:4873
```

Installing needs no login. A plain `oml install` keeps the version already installed; to move to a newer release, name it, for example `oml install @modelware/sierra-method@^0.1.1 -r http://localhost:4873`.

## See and remove versions

```bash
npm view @modelware/sierra-method versions --registry http://localhost:4873
oml unpublish 0.1.0 -r http://localhost:4873
```

npm refuses to remove a package's only version unless you add `--force`. You can also browse http://localhost:4873.

## Notes

- Packages in the `@modelware` scope come only from this registry; everything else is fetched from npmjs.org (see `verdaccio.yaml`). Change that scope for another organization.
- The registry listens on `localhost`, so publishers and consumers must be on the same machine.
- To start over, stop the registry and delete `.verdaccio/`.
