# plusminus-workflows

Reusable GitHub Actions workflows shared by the Plusminus projects.

## `maven-central-release.yml`

Builds, tests and publishes Maven artifacts to Maven Central. It is a `workflow_call`
(reusable) workflow — other repositories call it instead of duplicating the release steps.

### Usage

```yaml
jobs:
  release:
    uses: plusminus-software/plusminus-workflows/.github/workflows/maven-central-release.yml@main
    with:
      java-version: "8"        # optional, default "21"
    secrets: inherit
```

### Inputs

| Input | Default | Description |
|---|---|---|
| `java-version` | `"21"` | Temurin JDK version used for the build |
| `module` | `""` | Release only this module (comma-separated). Empty = resolve automatically |
| `create-git-tag` | `true` | Tag the released versions after a successful deploy |

### Secrets

`MAVEN_CENTRAL_USERNAME`, `MAVEN_CENTRAL_PASSWORD`, `GPG_PRIVATE_KEY` and `GPG_PASSPHRASE`
are all required.

### What it does

1. **Resolves the target modules.** An explicit `module` input always wins. Otherwise, on a
   `push` it diffs against the previous commit and releases only the top-level directories that
   changed and contain a `pom.xml`; a change to the root `pom.xml`, `.mvn/` or `.github/`, an
   unavailable base commit, or any non-push trigger falls back to releasing everything.
2. **Plans the release.** Three repository layouts are supported: a *reactor* (root `pom.xml`
   with `<modules>`), a *single* project (root `pom.xml`, no modules) and *standalone* (no root
   `pom.xml`; every top-level directory with a `pom.xml` is its own project, released in
   alphabetical order so dependencies publish before their consumers).
3. **Picks the action per project.** `-SNAPSHOT` versions deploy on any trigger; release
   versions deploy — GPG-signed — only on a manual `workflow_dispatch` run, and the plan fails
   early if that exact version is already on Central.
4. **Builds, deploys, tags** the released versions (unless `create-git-tag: false`) and
   publishes a job summary.
