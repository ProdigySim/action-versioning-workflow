# Action Versioning Workflow

Implements tag-based verisoning based on [Github Action Versioning Recommendations](https://github.com/actions/toolkit/blob/main/docs/action-versioning.md).

On every push to main branches, major, minor, and patch version tags are updated and published.

Major and minor versions are controlled by a `version.rc` file checked in to the branch, and patch versions are auto-incremented.

## Installation

1. Create a `version.rc` file in your repo initialized to `1.0` (or whatever your current version is)
2. Create a `.github/workflows/versioning.yml` in your repository with the following content:

```yml
name: Action Versioning

on:
  push:
    branches:
      - main
      - releases/** # If you need to maintain an old major or minor version, branch it to (e.g.) releases/v1

jobs:
  versioning:
    name: "Release"
    permissions:
      contents: write
    uses: ProdigySim/action-versioning-workflow/.github/workflows/action_versioning.yml@v1
```

## Usage
New patch versions will automatically be created on commits to `main`.
To publish a new major or minor version, bump the version in `version.rc` and commit it back to main.

Any changes to main will be picked up by the versioning workflow and your tags will be updated based on the values in `version.rc` and the previously published patch version for the given major and minor versions.

### Examples
#### Patch version bump
If your latest published version is `v1.0.1`, and you push another commit to main without touching `version.rc`, this workflow will:

1. Publish a new `v1.0.2` tag pointing at your commit
2. Move the `v1` and `v1.0` tags to point at your commit

#### Minor version bump
If your latest published version is `v1.0.9`, and you push a commit to main that bumps `versions.rc` to `1.1`, this workflow will:

1. Publish new `v1.1` and `v1.1.0` tags pointing at your commit
2. Move the `v1` tag to point at your commit

#### Major version bump
If your latest published version is `v1.0.15`, and you push a commit to main that bumps `versions.rc` to `2.0`, this workflow will:

1. Publish new `v2`, `v2.0`, and `v2.0.0` tags pointing at your commit

### Developing old versions
This workflow can support development on old versions when using the branch triggers defined in "Installation" by moving development of those branches to `release/<something>`.

For example, if you want to release a `v2`, but continue releasing patches on `v1`, you can:

1. Release your version bump to `2.0` on `main`
2. Create a new branch from your last `v1` release. e.g.
   1. `git fetch origin --tags      # get up to date tags from origin`
   2. `git checkout v1              # get latest v1 release`
   3. `git checkout -b releases/v1  # create a branch for v1 releases`
   4. `git push origin releases/v1  # push it back to the repo`
3. Continue making PRs and commits against either `main` or `releases/v1` depending on your target

## Notes

This repo is also auto-versioned using its own script
