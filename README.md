# Ariawalk accessibility tests

Run [Ariawalk](https://ariawalk.com) tests in GitHub Actions. An AI agent tries each test's task the way a blind person would, with the NVDA screen reader, and the job fails when a screen reader user can't finish it (or on regressions, new serious issues or low scores, if you ask).

The browser runs on the runner, so it can test `localhost`, a preview deployment or staging through a VPN step. Results go to the job summary and, on pull requests, to a comment that's kept up to date.

```yaml
permissions:
  contents: read
  pull-requests: write   # for the results comment

jobs:
  accessibility:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - run: npm ci && npm run build && (npm start &)   # or deploy a preview
      - uses: AndreHogberg/ariawalk-action@v1
        with:
          api-key: ${{ secrets.ARIAWALK_API_KEY }}
          app: My shop
          base-url: http://localhost:3000
          fail-on: blocked,regression
          data: |
            password=${{ secrets.STAGING_PASSWORD }}
```

Create the API key in Ariawalk under **Settings → API and MCP** with *Read and run* access, and store it as a repository secret.

## Inputs

| Input | Default | What it does |
| --- | --- | --- |
| `api-key` | (required) | An Ariawalk API key with *Read and run* access. |
| `tests` | | Test ids to run, separated by spaces or new lines. |
| `app` | | Or every test of this application (name or id). |
| `base-url` | | Run on this origin, keeping each test's path. Localhost and private hosts need no domain verification; public hosts must be verified in Ariawalk. |
| `fail-on` | `blocked` | What fails the job, comma-separated: `blocked`, `regression`, `new-serious`, `none`. |
| `min-score` | | Also fail tests scoring lower (0-100). |
| `concurrency` | `1` | Tests at once, each in its own browser. |
| `data` | | Test data, one `KEY=value` per line, e.g. secrets Ariawalk never sends. Values are masked in logs. |
| `junit` | | Write JUnit XML results to this path. |
| `version` | `latest` | The `ariawalk` CLI version to use. |
| `comment` | `true` | Keep one results comment on the pull request up to date. |
| `comment-key` | the job id | Tells comments apart when several jobs comment. |
| `api-url` | | Another Ariawalk instance. |

## Outputs

| Output | What it is |
| --- | --- |
| `exit-code` | `0` passed, `1` an accessibility check failed, `2` a run couldn't finish (setup, quota, login, network). |
| `summary` | Path of the Markdown summary. |
| `results` | Path of the results as JSON. |

Runs in CI belong to the branch's own environment in Ariawalk (`ci:<branch>`), so they're compared with the branch's previous run, else the default branch's, and never become the baseline for production. Chromium is cached between runs. Needs Node.js 20.3 or later on the runner (GitHub-hosted runners have it).

Full documentation: [ariawalk.com/docs/cli](https://ariawalk.com/docs/cli). For GitLab, Jenkins and other CI systems that run jobs in containers, use the image `ghcr.io/andrehogberg/ariawalk`.
