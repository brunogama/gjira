# GJira

GJira fetches a Jira issue based on the current branch name and appends to the
commit body.

The current supported branch format is `<issue_id>/<any/<text>`. GJira will
first check whether the branch starts with the expected format, if not, it
exits with `0` without exiting Git, otherwise it connects to Jira and gets the
task/story ID and appends to the body of the commit.

## Why?

This came as a requirement from projects I work, where the branch name is
required to have the task id, separated by a `/` and commits need to have the
issue id and story id attached. This allows managers to view commit flow during
the week and visually team performance.

## Requirements

- Python >=3
- [pre-commit](https://pre-commit.com/)

## Setup

### Git commit template

GJira requires a commit template file. GJira supports Jinja2, which allows
customizable templates based on Jira context. For example:

```text
# The following is automatically 'commit.template'

Related Jira issue: {{ key }}
Related Jira story: {{ parent__key }}
Related Jira description: {{ summary }}
```

The keys are related to Jira issue attributes. For example:

```text
issue.fields.worklog.worklogs[0].author
issue.fields.worklog.worklogs[0].comment
issue.fields.worklog.worklogs[0].created
issue.fields.worklog.worklogs[0].id
issue.fields.worklog.worklogs[0].self
issue.fields.worklog.worklogs[0].started
issue.fields.worklog.worklogs[0].timeSpent
issue.fields.worklog.worklogs[0].timeSpentSeconds
issue.fields.worklog.worklogs[0].updateAuthor                # dictionary
issue.fields.worklog.worklogs[0].updated

issue.fields.timetracking.remainingEstimate           # may be NULL or string ("0m", "2h"...)
issue.fields.timetracking.remainingEstimateSeconds    # may be NULL or integer
issue.fields.timetracking.timeSpent                   # may be NULL or string
issue.fields.timetracking.timeSpentSeconds            # may be NULL or integer
```

Inner issue fields **require** `.` (dot) to be replaced with `__` (double
underscore).

### pre-commit

Add the following repository to your `.pre-commit-config.yml` file

```yaml
- repo: https://github.com/benmezger/gjira
  rev: master
  hooks:
    - id: gjira
      args: ["--board=<board/project name>", "--template=.commit.template"]
```

### Branch

Change `<board/projec name>` with your current Jira project.

### Environment variables

Set the following environment variables:

```sh
export jiraserver="https://domain.atlassian.net"
export jirauser="your@email.com"

# from: https://id.atlassian.com/manage-profile/security/api-tokens
export jiratoken="token"
```

### Installing the hook

Finally, install the hook with pre-commit: `pre-commit install --hook-type prepare-commit-msg`.

## Asciinema

[![asciicast](https://asciinema.org/a/331379.svg)](https://asciinema.org/a/331379)

## Troubleshooting

- GJira is not appending the issue/story to the commit message.

  That's probably because you are not checkout to a branch with the required
  format or credentials are possibly wrong.

- GJira is not appending the story ID

  That's probably because your issue is not a subtask of a story.

- I need it solved right now!

  Run `pre-commit uninstall --hook-type prepare-commit-msg`. That should disable
  `prepare-commit-msg`.

## Development

1. Install requirements
   `pip install -r requirements.txt`
2. Run `pytest`
   `pytest .`

There are two ways of manually running GJira.

1. `python -m gjira` which will run `main()` in `__main__`
2. Installing the cli to your system
   `pip install .`

## TODO

- Cache issues the board and check the cache before doing a HTTP request
  - add `--refresh` parameter to GJira
