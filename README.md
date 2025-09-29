# Trusted publishing example

This is an example package published using npm's [Trusted Publishing](https://docs.npmjs.com/trusted-publishers) and GitHub Actions.

You can use it as a reference for your own package, customizing the `.github/workflows/publish.yml` file to your needs.

## Requirements

- `pnpm`: You need a recent version of `pnpm` installed globally. The `package.json#packageManager` field specifies the exact version used in this package.

## Publishing workflow

This package publishes automatically every time you push a new `v*` tag to the repository.

The workflow is split into three jobs. This keeps each logical step isolated from the others.

The jobs are:

1. The `prepare` job clones your repository, installs its dependencies, builds your project, and packs it into a tarball.
2. The `review` job compares the tarball you just created with the latest version in the npm registry. You should manually inspect the changes and decide whether to publish the new version.
3. The `publish` job uses the tarball created in the first step and publishes it to npm.

The `publish` job runs in the `npm-publish` [GitHub Environment](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/manage-environments), which requires approval from another maintainer and has a wait time before proceeding.

The `npm-publish` environment is set up in npm's trusted publishing configuration, so only jobs using it will have access to a publishing token. This means that only uploading the tarball runs in a privileged environment.

### How to publish a new version

To publish a new version, push a new `v*` tag to the repository, and the workflow will run.

The first two jobs run automatically, but the `publish` job must be approved by another authorized maintainer.

To approve it, they should inspect the logs, paying special attention to the `review` job output and any steps in `prepare` that may be useful. If everything looks good, they can approve the job, and the package will be published.

As an extra security measure, the `publish` job has a wait time before proceeding, so that a job has a window of time to be canceled.

## Replicating this setup

To replicate this setup, copy the `.github/workflows/publish.yml` file to your repository, and change the `NPM_TAG` variable to the tag you want to use.

You also need to configure your build step if needed. This project doesn’t have one, it just prints a message instead.

Additionally, you may want to change how the workflow is triggered, for example, to deploy from a branch instead of a tag. If you do, update the workflow so it can detect when a new version needs to be published.

Deploying from a branch can be more complex, but it allows you to use branch protection rules in the `npm-publish` GitHub environment.

### Setting up the GitHub environment

The key to making this setup secure is properly configuring the `npm-publish` GitHub environment, both in the npm package settings and in the GitHub repository.

Make sure to, at a minimum:

- Require approvals from a set of trusted maintainers.
- Forbid self-approvals and admin overrides.
- Add a wait time before the job can run.

Note that the wait time starts running when the job is proposed, not approved. So you need a way to get notified about it to be able to cancel it on time.

### Other work

Note that this is an example, and not a complete setup. There are some open questions and further work that are needed. I created a few issues with more info about them. 

## Other security measures

This repository also uses `minimum-release-age`, defined in the `.npmrc` file, to avoid installing dependencies that have been published too recently and could be compromised.

In this example, we set a value high enough that it doesn’t install the latest version of `chalk` at the time of writing. In practice, you should use a value that is appropriate for your project, such as 7 days.

If you are using a `pnpm` monorepo, configure this in the `pnpm-workspace.yaml` file instead, as `minimumReleaseAge`.

## Read more

- [GitHub: Our plan for a more secure npm supply chain](https://github.blog/security/supply-chain-security/our-plan-for-a-more-secure-npm-supply-chain/)
- [Trusted publishing for npm packages](https://docs.npmjs.com/trusted-publishers)
- [`minimumReleaseAge` docs](https://pnpm.io/settings#minimumreleaseage)
