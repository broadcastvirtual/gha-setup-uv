# setup-uv

Set up your GitHub Actions workflow with a specific version of [uv](https://docs.astral.sh/uv/).

- Install a version of uv and add it to PATH
- Cache the installed version of uv to speed up consecutive runs on self-hosted runners
- Register problem matchers for error output
- (Optional) Persist the uv's cache in the GitHub Actions Cache
- (Optional) Verify the checksum of the downloaded uv executable

## Contents

- [Usage](#usage)
  - [Install the latest version (default)](#install-the-latest-version-default)
  - [Install a specific version](#install-a-specific-version)
  - [Install a version by supplying a semver range](#install-a-version-by-supplying-a-semver-range)
  - [Validate checksum](#validate-checksum)
  - [Enable Caching](#enable-caching)
    - [Cache dependency glob](#cache-dependency-glob)
  - [Local cache path](#local-cache-path)
  - [GitHub authentication token](#github-authentication-token)
  - [UV_TOOL_DIR](#uv_tool_dir)
  - [UV_TOOL_BIN_DIR](#uv_tool_bin_dir)
- [How it works](#how-it-works)
- [FAQ](#faq)

## Usage

### Install the latest version (default)

```yaml
- name: Install the latest version of uv
  uses: astral-sh/setup-uv@v3
  with:
    version: "latest"
```

For an example workflow, see
[here](https://github.com/charliermarsh/autobot/blob/e42c66659bf97b90ca9ff305a19cc99952d0d43f/.github/workflows/ci.yaml).

> [!TIP]
>
> Using `latest` requires that uv download the executable on every run, which incurs a cost
> (especially on self-hosted runners). As a best practice, consider pinning the version to a
> specific release.

### Install a specific version

```yaml
- name: Install a specific version of uv
  uses: astral-sh/setup-uv@v3
  with:
    version: "0.4.4"
```

### Install a version by supplying a semver range

You can also specify a [semver range](https://github.com/npm/node-semver?tab=readme-ov-file#ranges)
to install the latest version that satisfies the range.

```yaml
- name: Install a semver range of uv
  uses: astral-sh/setup-uv@v3
  with:
    version: ">=0.4.0"
```

```yaml
- name: Pinning a minor version of uv
  uses: astral-sh/setup-uv@v3
  with:
    version: "0.4.x"
```

### Validate checksum

You can also specify a checksum to validate the downloaded file. Checksums up to the default version
are automatically verified by this action. The sha256 hashes can be found on the
[releases page](https://github.com/astral-sh/uv/releases) of the uv repo.

```yaml
- name: Install a specific version and validate the checksum
  uses: astral-sh/setup-uv@v3
  with:
    version: "0.3.1"
    checksum: "e11b01402ab645392c7ad6044db63d37e4fd1e745e015306993b07695ea5f9f8"
```

### Enable caching

If you enable caching, the [uv cache](https://docs.astral.sh/uv/concepts/cache/) will be cached to
the GitHub Actions Cache. This can speed up runs that reuse the cache by several minutes.

> [!TIP]
>
> On self-hosted runners this is usually not needed since the cache generated by uv on the runner's
> filesystem is not removed after a run. For more details see [Local cache path](#local-cache-path).

You can optionally define a custom cache key suffix.

```yaml
- name: Enable caching and define a custom cache key suffix
  id: setup-uv
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
    cache-suffix: "optional-suffix"
```

When the cache was successfully restored, the output `cache-hit` will be set to `true` and you can
use it in subsequent steps. For example, to use the cache in the above case:

```yaml
- name: Do something if the cache was restored
  if: steps.setup-uv.outputs.cache-hit == 'true'
  run: echo "Cache was restored"
```

#### Cache dependency glob

If you want to control when the cache is invalidated, specify a glob pattern with the
`cache-dependency-glob` input. The cache will be invalidated if any file matching the glob pattern
changes. The glob matches files relative to the repository root.

> [!NOTE]
>
> The default is `**/uv.lock`.

```yaml
- name: Define a cache dependency glob
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
    cache-dependency-glob: "**/requirements*.txt"
```

```yaml
- name: Define a list of cache dependency globs
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
    cache-dependency-glob: |
      **/requirements*.txt
      **/pyproject.toml
```

```yaml
- name: Never invalidate the cache
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
    cache-dependency-glob: ""
```

### Local cache path

This action controls where uv stores its cache on the runner's filesystem by setting `UV_CACHE_DIR`.
It defaults to `setup-uv-cache` in the `TMP` dir, `D:\a\_temp\uv-tool-dir` on Windows and
`/tmp/setup-uv-cache` on Linux/macOS. You can change the default by specifying the path with the
`cache-local-path` input.

```yaml
- name: Define a custom uv cache path
  uses: astral-sh/setup-uv@v3
  with:
    cache-local-path: "/path/to/cache"
```

### GitHub authentication token

This action uses the GitHub API to fetch the uv release artifacts. To avoid hitting the GitHub API
rate limit too quickly, an authentication token can be provided via the `github-token` input. By
default, the `GITHUB_TOKEN` secret is used, which is automatically provided by GitHub Actions.

If the default
[permissions for the GitHub token](https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication#permissions-for-the-github_token)
are not sufficient, you can provide a custom GitHub token with the necessary permissions.

```yaml
- name: Install the latest version of uv with a custom GitHub token
  uses: astral-sh/setup-uv@v3
  with:
    github-token: ${{ secrets.CUSTOM_GITHUB_TOKEN }}
```

### UV_TOOL_DIR

On Windows `UV_TOOL_DIR` is set to `uv-tool-dir` in the `TMP` dir (e.g. `D:\a\_temp\uv-tool-dir`).
On GitHub hosted runners this is on the much faster `D:` drive.

On all other platforms the tool environments are placed in the
[default location](https://docs.astral.sh/uv/concepts/tools/#tools-directory).

If you want to change this behaviour (especially on self-hosted runners) you can use the `tool-dir`
input:

```yaml
- name: Install the latest version of uv with a custom tool dir
  uses: astral-sh/setup-uv@v3
  with:
    tool-dir: "/path/to/tool/dir"
```

### UV_TOOL_BIN_DIR

On Windows `UV_TOOL_BIN_DIR` is set to `uv-tool-bin-dir` in the `TMP` dir (e.g.
`D:\a\_temp\uv-tool-bin-dir`). On GitHub hosted runners this is on the much faster `D:` drive. This
path is also automatically added to the PATH.

On all other platforms the tool binaries get installed to the
[default location](https://docs.astral.sh/uv/concepts/tools/#the-bin-directory).

If you want to change this behaviour (especially on self-hosted runners) you can use the
`tool-bin-dir` input:

```yaml
- name: Install the latest version of uv with a custom tool bin dir
  uses: astral-sh/setup-uv@v3
  with:
    tool-bin-dir: "/path/to/tool-bin/dir"
```

## How it works

This action downloads uv from the uv repo's official
[GitHub Releases](https://github.com/astral-sh/uv) and uses the
[GitHub Actions Toolkit](https://github.com/actions/toolkit) to cache it as a tool to speed up
consecutive runs on self-hosted runners.

The installed version of uv is then added to the runner PATH, enabling subsequent steps to invoke it
by name (`uv`).

## FAQ

### Do I still need `actions/setup-python` alongside `setup-uv`?

No. This action is modelled as a drop-in replacement for `actions/setup-python` when using uv. With
`setup-uv`, you can install a specific version of Python using `uv python install` rather than
relying on `actions/setup-python`.

For example:

```yaml
- name: Checkout the repository
  uses: actions/checkout@main
- name: Install the latest version of uv
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
- name: Test
  run: uv run --frozen pytest
```

To install a specific version of Python, use
[`uv python install`](https://docs.astral.sh/uv/guides/install-python/):

```yaml
- name: Install the latest version of uv
  uses: astral-sh/setup-uv@v3
  with:
    enable-cache: true
- name: Install Python 3.12
  run: uv python install 3.12
```

### What is the default version?

By default, this action installs the latest version of uv.

If you require the installed version in subsequent steps of your workflow, use the `uv-version`
output:

```yaml
- name: Checkout the repository
  uses: actions/checkout@main
- name: Install the default version of uv
  id: setup-uv
  uses: astral-sh/setup-uv@v3
- name: Print the installed version
  run: echo "Installed uv version is ${{ steps.setup-uv.outputs.uv-version }}"
```

## Acknowledgements

`setup-uv` was initially written and published by [Kevin Stillhammer](https://github.com/eifinger)
before moving under the official [Astral](https://github.com/astral-sh) GitHub organization. You can
support Kevin's work in open source on [Buy me a coffee](https://www.buymeacoffee.com/eifinger) or
[PayPal](https://paypal.me/kevinstillhammer).

## License

MIT

<div align="center">
  <a target="_blank" href="https://astral.sh" style="background:none">
    <img src="https://raw.githubusercontent.com/astral-sh/uv/main/assets/svg/Astral.svg" alt="Made by Astral">
  </a>
</div>
