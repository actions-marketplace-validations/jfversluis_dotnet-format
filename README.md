# GitHub Action for dotnet-format

[![CI Workflow Status](https://github.com/jfversluis/dotnet-format/workflows/CI/badge.svg)](https://github.com/jfversluis/dotnet-format/actions?query=workflow%3ACI)

Run [dotnet-format](https://github.com/dotnet/format) as part of your workflow to report formatting errors or auto fix violations as part of your pull request workflow.

## Usage

### Running on `pull_request`, adding a commit to the PR.
Use this only when the PRs are coming from the same repository. You won't have permission to add commits to PRs coming from a forked repository.

```yml
name: Format check on pull request
on: pull_request
jobs:
  dotnet-format:
    runs-on: windows-latest
    steps:
      - name: Install dotnet-format
        run: dotnet tool install -g dotnet-format

      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}

      - name: Run dotnet format
        id: format
        uses: jfversluis/dotnet-format@v1.0.5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          action: "fix"
          only-changed-files: true

      - name: Commit files
        if: steps.format.outputs.has-changes == 'true'
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git commit -a -m 'Automated dotnet-format update
          Co-authored-by: ${{ github.event.pull_request.user.login }} <${{ github.event.pull_request.user.id }}+${{ github.event.pull_request.user.login }}@users.noreply.github.com>'
      - name: Push changes
        if: steps.format.outputs.has-changes == 'true'
        uses: ad-m/github-push-action@v0.5.0
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          branch: ${{ github.head_ref }}

```

### Scheduled daily at midnight (UTC) and open a PR with the fixes
You should use this if you run a OSS project with outside contributors. You won't have permission to add commits to PRs coming from a forked repository.

```yml
name: Daily code format check
on:
  schedule:
    - cron: 0 0 * * * # Every day at midnight (UTC)
jobs:
  dotnet-format:
    runs-on: windows-latest
    steps:
      - name: Install dotnet-format
        run: dotnet tool install -g dotnet-format

      - name: Checkout repo
        uses: actions/checkout@v2
        with:
          ref: ${{ github.head_ref }}

      - name: Run dotnet format
        id: format
        uses: jfversluis/dotnet-format@v1.0.5
        with:
          repo-token: ${{ secrets.GITHUB_TOKEN }}
          action: "fix"
          workspace: "MySolution.sln"

      - name: Commit files
        if: steps.format.outputs.has-changes == 'true'
        run: |
          git config --local user.name "github-actions[bot]"
          git config --local user.email "41898282+github-actions[bot]@users.noreply.github.com"
          git commit -a -m 'Automated dotnet-format update'
      - name: Create Pull Request
        uses: peter-evans/create-pull-request@v3
        with:
          title: '[housekeeping] Automated PR to fix formatting errors'
          body: |
            Automated PR to fix formatting errors
          committer: GitHub <noreply@github.com>
          author: github-actions[bot] <41898282+github-actions[bot]@users.noreply.github.com>
          labels: housekeeping
          assignees: jfversluis
          reviewers: jfversluis
          branch: housekeeping/fix-codeformatting
```

## Options

### Required

Name | Allowed values | Description
-- | -- | --
`repo-token` | `GITHUB_TOKEN` or a custom value | The token used to call the GitHub api.

### Optional

Name | Allowed values | Description
-- | -- | --
`action` | `check` (default), `fix` | The primary action `dotnet format` should perform. Check doesn't make changes to your file, fix will do the actual formatting.
`only-changed-files` | `true`, `false` (default) | Only changed files in the current pull request should be formatted. Only works when the trigger is a pull request.
`fail-fast` | `true` (default), `false` | The job should fail if there's a formatting error. Only used with the `check` action.
`workspace` | `.` | The solution or project file to operate on. In case you want to process all files in a certain folder, set the root folder here and specify the `workspaceIsFolder` option.
`workspaceIsFolder` | `.` | Specifies if the value in `workspace` has to be treated as a simple folder. If true, set `workspace` to the root folder you want to process.
`include` | `.` | The files to include, delimited by space. Cannot be used together with the `workspace` option.
`exclude` | `.` | Space delimited list of files and/or folders to ignore.
`logLevel` | `q[uiet]`, `m[inimal]`, `n[ormal]`, `d[etailed]`,  `diag[nostic]` | Sets the logging verbosity of the dotnet format process
`fix-whitespace` | `true`, `false` (default) | Removes whitespaces according to formatting rules.
`fix-analyzers-level` | `info`, `warn`, `error` | Fixes styles from third-party analyzers. More on https://github.com/dotnet/format/blob/main/docs/3rd-party-analyzers.md.
`fix-style-level` | `info`, `warn`, `error` | Fixes styles according to formating rules.

## Outputs

Name | Description
-- | --
`has-changes` | If any files were found to have violations or had fixes applied. Will be a string value of `true` or `false`.

# Building from source

## Generating `dist/index.js`
`npm run release`

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)

## Acknowledgements

Thank you @victor-alcazar and @xt0rted for the first versions that I have forked this from
