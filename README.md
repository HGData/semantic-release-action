# Semantic Release Action
[![CI](https://github.com/hgdata/semantic-release-action/actions/workflows/release.yml/badge.svg)](https://github.com/hgdata/semantic-release-action/actions/workflows/release.yml)

GitHub Action for [Semantic Release][https://semantic-release.gitbook.io/semantic-release/]. 

## Usage
1. Set any [Semantic Release configuration](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/configuration.md#configuration) in your repository.
1. [Add secrets](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets) in your repository for the [Semantic Release authentication](https://github.com/semantic-release/semantic-release/blob/master/docs/usage/ci-configuration.md#authentication) environment variables.
1. Add a [workflow file](https://help.github.com/en/articles/workflow-syntax-for-github-actions) to your repository to create custom automated processes.

#### Basic usage:
```yaml
steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Semantic release
  uses: hgdata/semantic-release-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    extends: |
      @hgdata/semantic-release-config
```

**IMPORTANT**: `GITHUB_TOKEN` does not have the required permissions to operate on protected branches.
If you are using this action for protected branches, replace `GITHUB_TOKEN` with [Personal Access Token](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line). If using the `@semantic-release/git` plugin for protected branches, avoid persisting credentials as part of `actions/checkout@v2` by setting the parameter `persist-credentials: false`. This credential does not have the required permission to operate on protected branches.

### Inputs
| Input Parameter  | Required | Description |
|:----------------:|:--------:|-------------|
| semantic_version | false    | Specify specifying version range for semantic-release. [[Details](#semantic_version)] |
| branches         | false    | The branches on which releases should happen.[[Details](#branches)]<br>Support for **semantic-release above v16**. |
| branch           | false    | The branch on which releases should happen.[[Details](#branch)]<br>Only support for **semantic-release older than v16**. |
| extra_plugins    | false    | Extra plugins for pre-install. [[Details](#extra_plugins)] |
| dry_run          | false    | Whether to run semantic release in `dry-run` mode. [[Details](#dry_run)] |
| extends          | false    | Use a sharable configuration [[Details](#extends)] |

#### semantic_version
> {Optional Input Parameter} Specify specifying version range for semantic-release.<br>If no version range is specified, latest version will be used by default.

```yaml
steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Semantic release
  uses: hgdata/semantic-release-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    semantic_version: 15.13.28  # It is recommended to specify specifying version range
                                # for semantic-release.
```

#### branches
> {Optional Input Parameter} The branches on which releases should happen.<br>`branches` supports for **semantic-release above v16**.

```yaml
steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Semantic release
  uses: hgdata/semantic-release-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    semantic_version: 16
    # you can set branches for semantic-release above v16.
    branches: |    
      [
        '+([0-9])?(.{+([0-9]),x}).x',
        'master', 
        'next', 
        'next-major', 
        {
          name: 'beta', 
          prerelease: true
        }, 
        {
          name: 'alpha', 
          prerelease: true
        }
      ]
```

`branches` will override the `branches` attribute in your configuration file. If the attribute is not configured on both sides, the default is: 
```
[
  '+([0-9])?(.{+([0-9]),x}).x',
  'master', 
  'next', 
  'next-major', 
  {name: 'beta', prerelease: true}, 
  {name: 'alpha', prerelease: true}
]
```

See [configuration#branches](https://semantic-release.gitbook.io/semantic-release/usage/configuration#branches) for more information.

#### branch
> {Optional Input Parameter} Similar to parameter `branches`. The branch on which releases should happen.<br>`branch` only supports for **semantic-release older than v16**.

```yaml
steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Semantic release
  uses: hgdata/semantic-release-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    semantic_version: 15.13.28
    # you can set branch for semantic-release older than v16.
    branch: your-branch
```

It will override the `branch` attribute in your configuration file. If the attribute is not configured on both sides, the default is `master`.

#### extra_plugins
> {Optional Input Parameter} Extra plugins for pre-install. 

The action can be used with `extra_plugins` option to specify plugins which are not in the [default list of plugins of semantic release](https://semantic-release.gitbook.io/semantic-release/usage/plugins#default-plugins). When using this option, please make sure that these plugins are also mentioned in your [semantic release config's plugins](https://semantic-release.gitbook.io/semantic-release/usage/configuration#plugins) array. 

For example, if you want to use `@semantic-release/git` and `@semantic-release/changelog` extra plugins, these must be added to `extra_plugins` in your actions file and `plugins` in your [release config file](https://semantic-release.gitbook.io/semantic-release/usage/configuration#configuration-file) as shown bellow:

Github Action Workflow:
```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v2

  - name: Semantic release
    uses: hgdata/semantic-release-action@v1
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      # You can specify specifying version range for the extra plugins if you prefer.
      extra_plugins: |
        @semantic-release/changelog@3.0.0
        @semantic-release/git
```

Similar to parameter `semantic_version`. *It is recommended to manually specify a version of semantic-release plugins to prevent errors caused.*

Release Config:
```diff
  plugins: [
    .
+   "@semantic-release/changelog"
+   "@semantic-release/git",
  ]
```

#### dry_run
> {Optional Input Parameter} Whether to run semantic release in `dry-run` mode.<br>It will override the dryRun attribute in your configuration file.

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v2

  - name: Semantic release
    uses: hgdata/semantic-release-action@v1
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    with:
      dry_run: true
```

#### extends
The action can be used with `extends` option to extend an existing [sharable configuration](https://semantic-release.gitbook.io/semantic-release/usage/shareable-configurations) of semantic-release. Can be used in combination with `extra_plugins`.

```yaml
steps:
- name: Checkout
  uses: actions/checkout@v2

- name: Semantic release
  uses: hgdata/semantic-release-action@v1
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  with:
    # You can extend an existing shareable configuration.
    extends: |
      @semantic-release/apm-config
```

### Outputs
| Output Parameter          | Description |
|:-------------------------:|---|
| new_release_published     | Whether a new release was published (`true` or `false`) |
| new_release_version       | Version of the new release. (e.g. `1.3.0`) |
| new_release_major_version | Major version of the new release. (e.g. `1`) |
| new_release_minor_version | Minor version of the new release. (e.g. `3`) |
| new_release_patch_version | Patch version of the new release. (e.g. `0`) |
| new_release_channel       | The distribution channel on which the last release was initially made available (undefined for the default distribution channel). |
| new_release_notes         | The release notes for the new release. |
| last_release_version      | Version of the previous release, if there was one. (e.g. `1.2.0`) |

#### Using output variables:
```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v2

  - name: Semantic release
    uses: hgdata/semantic-release-action@v1
    id: semantic   # Need an `id` for output variables
    env:
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      NPM_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
  - name: Do something when a new release published
    if: steps.semantic.outputs.new_release_published == 'true'
    run: |
      echo ${{ steps.semantic.outputs.new_release_version }}
      echo ${{ steps.semantic.outputs.new_release_major_version }}
      echo ${{ steps.semantic.outputs.new_release_minor_version }}
      echo ${{ steps.semantic.outputs.new_release_patch_version }}
```

## License
This project is released under the [MIT License][LICENSE].
