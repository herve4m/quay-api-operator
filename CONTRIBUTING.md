# How to Contribute

We welcome contributions from the community.
There are a few ways you can help us improve.

## Opening an Issue

If you see something you would like changed, but you are not sure how to change
it, then submit an issue describing what you'd like to see.

## Working Locally

Before committing your contribution, install and then run the `pre-commit` tool to verify your work.

Git automatically runs `pre-commit` every time you commit your work.
You can also run `pre-commit` at any time by using the `pre-commit run --all` command from inside your local copy of the GitHub repository.

See the [pre-commit](https://pre-commit.com/) documentation for more details.

## Submitting a Pull Request

Prepare and submit pull requests as follows:

1. Fork the repository on GitHub and then clone it locally.
2. Create a branch named appropriately for the change you are going to make.
3. Make your code change.
4. If you are creating a CRD, then add a usage example in the `config/samples/` directory.
5. Push your code change to your forked repository.
6. Use the GitHub web UI to navigate to the original repository https://github.com/herve4m/quay-api-operator/pulls (not your forked repository).
   Open a pull request.
7. All pull requests go to a validation process.
   Make sure to run `pre-commit` before submitting your code.

For more details on forks and pull requests, see the [Creating a Pull Request from a Fork](https://docs.github.com/en/github/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request-from-a-fork) and [How to Create a Pull Request in GitHub](https://opensource.com/article/19/7/create-pull-request-github) documentations.
