# runovate
An experimental GitHub Action to manage a renovate branch, for rolling up updates easily and safely

## How it works

Add a workflow to your repo with the following:

```yaml
name: deps-sync
on:
  push:
    branches: [main, deps]

jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: mmkal/runovate@v1
```

This will create a `deps` branch if it doesn't already exist, a renovate config, and will open up a pull request to update your dependencies.

The pull request will be opened with the `deps` branch as the head, and the `main` branch as the base (these branches are configurable, see below).

All successful updates will automatically go into the `deps` branch, and the pull request will be updated. You should merge this manually whenever you're ready. (The idea being to use squash-and-merge so your main branch is not cluttered with dependency update commits).

Failing updates will go into a separate pull request against the `deps` branch. The failure should be looked at and fixed. Then you can merge the pull request into deps, so it'll be ready to merge into `main` when you're ready along with the other updates.

Changes to `main` will be automatically merged into `deps`, so you can just merge `deps` into `main` when you're ready - `deps` should always be strictly ahead of `main`.

Try to avoid pushing anything into `deps` other than dependency updates, and related changes.

After merging the pull request into `main`, a new one will be created whenever renovate updates another dependency.

Your renovate config will be automatically created or updated to include the settings required by this action. Most existing settings will be preserved, but `baseBranches`, `automerge`, `automergeType`, and `fetchChangeLogs` will be set to the values required by this action.

## Configuration

You can configure the branches used for `deps` and `main` by setting the `BRANCH_NAME` and `BASE_BRANCH` environment variables.

```yaml
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: mmkal/runovate@v1
      with:
        deps-branch: dependencies
        main-branch: master
```

You can also set a GitHub token to use for the action. This is useful for when main syncs to deps - otherwise GitHub Actions may not run some of your checks (commits made with the default `GITHUB_TOKEN` [don't trigger workflows](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/triggering-a-workflow#triggering-a-workflow-from-a-workflow)).

```yaml
jobs:
  run:
    runs-on: ubuntu-latest
    steps:
      - uses: mmkal/runovate@v1
      with:
        github-token: ${{ secrets.MY_GITHUB_TOKEN }}
```

## What's going on

There's not all that much to this, it's just using the GitHub API and git CLI to manage branches, pull requests etc. It's all in a single [action.yml](./action.yml) file which is fairly easy to understand if you know bash and JavaScript.

## Ejecting

If you decide you want to go back to a "normal" renovate setup, just delete the workflow you created, and update your renovate config to use the settings you want.
