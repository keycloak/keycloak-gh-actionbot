# Keycloak GitHub Action Bot

Custom GitHub Actions for Keycloak projects.

## Using the bot in a repository

The bot contains the following functionality:

- When adding a new comment `/rerun` as is on a pull request that is open, the bot will re-run any failed jobs in a workflow run.
  The user commenting needs to be either a collaborator, an owner, a member of the organization or a contributor.

After processing the comment, the bot adds a +1 reaction to the comment.

**NOTE:** If there are queued GitHub actions in the GitHub organization, it might take some time until bot can start, trigger the re-run of the workflow and add the reaction.

## Adding the bot to the repository

To add it to a repository, add the following GitHub workflow to the repository in its main branch.
It will then run the action on each comment on either an issue or a pull request to see if one of the commands listed above has been added.
When it runs, it will first check if the conditions mentioned above are met, and only then take action.

It needs write-permissions on `pull-requests` to be able to add the reaction, and it needs write access to `actions` to re-run the failed actions.

```yaml
name: Keycloak GitHub Action Bot

on:
  issue_comment:
    types:
      - created

permissions:
  actions: write
  pull-requests: write

jobs:
  act:
    runs-on: ubuntu-latest
    steps:
      - uses: keycloak/keycloak-gh-actionbot@v0.2.0
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Developing changes for bot

1. Fork the repository, clone it and create a feature branch.
2. Run `npm install` to install the dependencies.
3. Make changes to `index.js`.
4. Run `npm run format` and `npm run dist` to package the changes to `dist/index.js`.
5. Commit and push the changes to the feature branch.
6. In the main branch of the forked repository, add the action bot that points to the feature branch.
7. Test the changes on the forked repository until satisfied, for example, with a PR and a GitHub action that always fails, and therefore can be re-run:

   ```yaml
   name: Always Fail

   on:
     pull_request:

   jobs:
     act:
       runs-on: ubuntu-latest
       steps:
         - run: "false"
   ```

8. Create a PR for the main project.

## Testing the GitHub action locally

A GitHub action environment can be simulated via environment variables, and the event can be provided via a file.

The following example assumes that the event is stored in the file `event.json`.
If environment variable `INPUT_GITHUB_TOKEN` is set, the action will run with anonymous/read only access.

```bash
export GITHUB_EVENT_NAME=issue_comment
export GITHUB_EVENT_PATH=event.json
export INPUT_GITHUB_TOKEN=ghp_...
export INPUT_DEBUG=true
```

After the environment variables are set, use the following command to execute the action:

```bash
node index.js
```

## Reading about the GitHub APIs

- [GitHub's REST API docs](https://docs.github.com/en/rest)
- [GitHub's guide to JavaScript actions](https://docs.github.com/en/actions/creating-actions/creating-a-javascript-action)

## Releasing changes for the bot

- Create a new tag for the release using for example `npm version patch` (like `v1.0.1`)
- Push the changes to the release branch for those who track only the major version (like `v1`)
