# Cypress CircleCI Orb

The Cypress CircleCI Orb is a piece of configuration set in your `circle.yml` file to correctly install, cache and run [Cypress.io](https://www.cypress.io) tests on [CircleCI](https://circleci.com) with very little effort.

**Note:** To use CircleCI Orbs in your projects, you might have to enable beta features per-project within CircleCI. See the official CircleCI documentation.

## Examples

Install dependencies (using `npm ci`) and run all Cypress tests:

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.0.0
workflows:
  build:
    jobs:
      - cypress/run
```

A more complex project that needs to install dependencies, build an application and run tests across 10 CI machines [in parallel](https://on.cypress.io/parallelization) may have:

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.0.0
workflows:
  build:
    jobs:
      - cypress/install:
          build: 'npm run build' # run a custom app build step
      - cypress/run:
          requires:
            - cypress/install
          record: true # record results on Cypress Dashboard
          parallel: true # split all specs across machines
          parallelism: 10 # use 10 CircleCI machines to finish quickly
          group: 'all tests' # name this group "all tests" on the dashboard
          start: 'npm start' # start server before running tests
```

In all cases, you are using `run` and `install` job definitions that Cypress provides inside the orb. Using the orb brings simplicity and static checks of parameters to CircleCI configuration.

For more examples, see the [examples.md](examples.md) generated from the [src/orb.yml](src/orb.yml). Also take a look at [cypress-io/cypress-example-circleci-orb](https://github.com/cypress-io/cypress-example-circleci-orb) and [cypress-io/cypress-example-kitchensink](https://github.com/cypress-io/cypress-example-kitchensink/pull/148/files).

## Jobs and executors

See [jobs.md](jobs.md) and [executors.md](executors.md) for a full list of public jobs and executors that this orb provides.

The CircleCI Orb exports the following job definitions to be used by the user projects:

### `run`

This job allows you to run Cypress tests on a one or more machines, record screenshots and videos, use the custom Docker image, etc.

A typical example:

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.0.0
workflows:
  build:
    jobs:
      # checks out code, installs npm dependencies
      # and runs all Cypress tests and records results on Cypress Dashboard
      - cypress/run:
          record: true
```

See all its parameters at the [cypress/run job example](jobs.md#run)

### `install`

{% note warning %}
This job is only necessary if you plan to execute the `run` job later. If you only want to run all tests on a single machine, then you do not need a separate `install` job.
{% endnote %}

Sometimes you need to install the project's npm dependencies and build the application _once_ before running tests, especially if you run the tests on multiple machines in parallel. For example:

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.0.0
workflows:
  build:
    jobs:
      # install dependencies first
      - cypress/install
      # now run tests
      - cypress/run:
          requires:
            - cypress/install
          record: true # record results to Cypress Dashboard
          parallel: true # run tests in parallel
          parallelism: 2 # use 2 CircleCI machines
          group: 2 machines # name this group "2 machines"
```

See available parameters at the [cypress/install job example](jobs.md#install)

## Versions

Cypress orb is _versioned_ so you can be sure that the configuration will _not_ suddenly change as we change orb commands. We follow semantic versioning to make sure you can upgrade project configuration to minor and patch versions without breaking changes.

You can find all changes and published orb versions at [cypress-io/circleci-orb/releases](https://github.com/cypress-io/circleci-orb/releases).

## Effective config

You can see the final _effective_ configuration your project resolves to by running `circleci config process <config filename>` from the terminal.

For example, if your current CircleCI configuration file is `.circleci/config.yml` and it contains the following:

```yaml
version: 2.1
orbs:
  cypress: cypress-io/cypress@1.0.0
workflows:
  build:
    jobs:
      - cypress/run
```

The fully resolved configuration will show:

```shell
$ circleci config process .circleci/config.yml
# Orb 'cypress-io/cypress@1.0.0' resolved to 'cypress-io/cypress@1.0.0'
version: 2
jobs:
  cypress/run:
    docker:
    - image: cypress/base:10
    parallelism: 1
    steps:
    - checkout
    - restore_cache:
        keys:
        - cache-{{ .Branch }}-{{ checksum "package.json" }}
    - run:
        name: Npm CI
        command: npm ci
    - run:
        command: npx cypress verify
    - save_cache:
        key: cache-{{ .Branch }}-{{ checksum "package.json" }}
        paths:
        - ~/.npm
        - ~/.cache
    - persist_to_workspace:
        root: ~/
        paths:
        - project
        - .cache/Cypress
    - attach_workspace:
        at: ~/
    - run:
        name: Run Cypress tests
        command: 'npx cypress run'
workflows:
  build:
    jobs:
    - cypress/run
  version: 2
```

### Ejecting

If you want to customize the orb configuration, you can save and tweak the output of the `circleci config process ...` to suit your needs.

{% note warning %}
There is no automated way to go from the "ejected" configuration back to using the orb.
{% endnote %}

## Development

If you want to develop this orb and publish new versions, see our [Contributing Guide](CONTRIBUTING.md).

## License

This project is licensed under the terms of the [MIT license](/LICENSE.md).
