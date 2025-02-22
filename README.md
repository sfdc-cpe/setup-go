# setup-go

[![build-test](https://github.com/actions/setup-go/actions/workflows/workflow.yml/badge.svg)](https://github.com/actions/setup-go/actions/workflows/workflow.yml)
[![Validate 'setup-go'](https://github.com/actions/setup-go/actions/workflows/versions.yml/badge.svg)](https://github.com/actions/setup-go/actions/workflows/versions.yml)

This action sets up a go environment for use in actions by:

- Optionally downloading and caching a version of Go by version and adding to `PATH`.
- Registering problem matchers for error output.

# V3

The V3 edition of the action offers:

- Adds `GOBIN` to the `PATH`
- Proxy support
- Check latest version
- Caching packages dependencies
- stable and oldstable aliases
- Bug Fixes (including issues around version matching and semver)

The action will first check the local cache for a version match. If a version is not found locally, it will pull it from the `main` branch of the [go-versions](https://github.com/actions/go-versions/blob/main/versions-manifest.json) repository. On miss or failure, it will fall back to downloading directly from [go dist](https://storage.googleapis.com/golang). To change the default behavior, please use the [check-latest input](#check-latest-version).

**Note:** The `setup-go` action uses executable binaries which are built by Golang side. The action does not build golang from source code.

Matching by [semver spec](https://github.com/npm/node-semver):

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '^1.13.1' # The Go version to download (if necessary) and use.
  - run: go version
```

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '>=1.17.0'
  - run: go version
```

Matching an unstable pre-release:

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.18.0-rc.1' # The Go version to download (if necessary) and use.
  - run: go version
```

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.16.0-beta.1' # The Go version to download (if necessary) and use.
  - run: go version
```

# Usage

See [action.yml](action.yml)

## Basic

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.16.1' # The Go version to download (if necessary) and use.
  - run: go run hello.go
```


## Check latest version

The `check-latest` flag defaults to `false`. Use the default or set `check-latest` to `false` if you prefer stability and if you want to ensure a specific Go version is always used.

If `check-latest` is set to `true`, the action first checks if the cached version is the latest one. If the locally cached version is not the most up-to-date, a Go version will then be downloaded. Set `check-latest` to `true` if you want the most up-to-date Go version to always be used.

> Setting `check-latest` to `true` has performance implications as downloading Go versions is slower than using cached versions.

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.14'
      check-latest: true
  - run: go run hello.go
```

## Using stable/oldstable aliases

If `stable` is provided, action will get the latest stable version from the [`go-versions`](https://github.com/actions/go-versions/blob/main/versions-manifest.json) repository manifest. 

If `oldstable` is provided, when current release is 1.19.x, action will resolve version as 1.18.x, where x is the latest patch release.

**Note:** using these aliases will result in same version as using corresponding minor release with `check-latest` input set to `true`

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: 'stable'
  - run: go run hello.go
```

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: 'oldstable'
  - run: go run hello.go
```

## Caching dependency files and build outputs:

The action has a built-in functionality for caching and restoring go modules and build outputs. It uses [actions/cache](https://github.com/actions/cache) under the hood but requires less configuration settings. The `cache` input is optional, and caching is turned off by default.

The action defaults to search for the dependency file - go.sum in the repository root, and uses its hash as a part of the cache key. Use `cache-dependency-path` input for cases when multiple dependency files are used, or they are located in different subdirectories.

**Caching without specifying dependency file path**
```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.17'
      check-latest: true
      cache: true
  - run: go run hello.go
```

**Caching in monorepos**
```yaml
steps:
  - uses: actions/checkout@v3
  - uses: actions/setup-go@v3
    with:
      go-version: '1.17'
      check-latest: true
      cache: true
      cache-dependency-path: subdir/go.sum
  - run: go run hello.go
  ```
## Getting go version from the go.mod file

The `go-version-file` input accepts a path to a `go.mod` file or a `go.work` file that contains the version of Go to be used by a project. As the `go.mod` file contains only major and minor (e.g. 1.18) tags, the action will search for the latest available patch version sequentially in the runner's directory with the cached tools, in the [version-manifest.json](https://github.com/actions/go-versions/blob/main/versions-manifest.json) file or at the go servers.

If both the `go-version` and the `go-version-file` inputs are provided then the `go-version` input is used.
> The action will search for the `go.mod` file relative to the repository root

```yaml
steps:
- uses: actions/checkout@v3
- uses: actions/setup-go@v3
  with:
    go-version-file: 'path/to/go.mod'
- run: go version
```

## Matrix testing

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        go: [ '1.14', '1.13' ]
    name: Go ${{ matrix.go }} sample
    steps:
      - uses: actions/checkout@v3
      - name: Setup go
        uses: actions/setup-go@v3
        with:
          go-version: ${{ matrix.go }}
      - run: go run hello.go
```

### Supported version syntax

The `go-version` input supports the following syntax:

- Specific versions: `1.15`, `1.16.1`, `1.17.0-rc.2`, `1.16.0-beta.1`
- SemVer's version range syntax: `^1.13.1`, `>=1.18.0-rc.1`

For more information about semantic versioning, please refer to [semver](https://github.com/npm/node-semver) documentation.

## Using `setup-go` on GHES

`setup-go` comes pre-installed on the appliance with GHES if Actions is enabled. When dynamically downloading Go distributions, `setup-go` downloads distributions from [`actions/go-versions`](https://github.com/actions/go-versions) on github.com (outside of the appliance). These calls to `actions/go-versions` are made via unauthenticated requests, which are limited to [60 requests per hour per IP](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting). If more requests are made within the time frame, then you will start to see rate-limit errors during downloading that looks like: `##[error]API rate limit exceeded for...`. After that error the action will try to download versions directly from https://storage.googleapis.com/golang, but it also can have rate limit so it's better to put token.

To get a higher rate limit, you can [generate a personal access token on github.com](https://github.com/settings/tokens/new) and pass it as the `token` input for the action:

```yaml
uses: actions/setup-go@v3
with:
  token: ${{ secrets.GH_DOTCOM_TOKEN }}
  go-version: 1.18
```

If the runner is not able to access github.com, any Go versions requested during a workflow run must come from the runner's tool cache. See "[Setting up the tool cache on self-hosted runners without internet access](https://docs.github.com/en/enterprise-server@3.2/admin/github-actions/managing-access-to-actions-from-githubcom/setting-up-the-tool-cache-on-self-hosted-runners-without-internet-access)" for more information.

# License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

# Contributions

Contributions are welcome! See [Contributor's Guide](docs/contributors.md)

## Code of Conduct

:wave: Be nice. See [our code of conduct](CODE_OF_CONDUCT.md)
