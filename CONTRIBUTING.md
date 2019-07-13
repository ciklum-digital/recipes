# Contributing

The following is a set of guidelines for contributing to Ciklum digital organisation packages. Please spend several minutes in reading these guidelines before you create an issue or pull request.

## Code of Conduct

We have adopted a [Code of Conduct](https://github.com/ciklum-digital/recipes/blob/master/CODE_OF_CONDUCT.md) that we expect project participants to adhere to. Please read the full text so that you can understand what actions will and will not be tolerated.

## Open Development

All work happens directly on [GitHub](https://github.com/ciklum-digital). Both core team members and external contributors send pull requests which go through the same review process.

## Branch Organization

There are three types of branch we maintain: `master` `issue/*` `feat/*`.
 
- If you send a Fix Request, please do it in: `issue/short-issue-title`
- If you send a Feature Request, please do it in: `feat/short-feature-title`


## Sending a Pull Request

The core team is monitoring for pull requests. We will review your pull request and either merge it, request changes to it, or close it with an explanation.

**Before submitting a pull request**, please make sure the following is done:

1. Fork the repository and create your branch from [proper branch](#Branch-Organization).
1. Run `yarn install` in the repository root.
1. Run `yarn start` in the repository root for launching docs locally.
1. Add new documentation block or edit existing.
1. Submit a pull request, referencing any issues it addresses.
1. For PR creation use [template]([template](https://github.com/ciklum-digital/recipes/tree/master/.github/PULL_REQUEST_TEMPLATE.md))

## Development Workflow

After cloning proper module, run `yarn` to fetch its dependencies. Then, you can run several commands:

1. `yarn start` for starting documentation site locally

## License

By contributing your code to the ciklum-digital GitHub org repositories, you agree to license your contribution under the MIT license.
