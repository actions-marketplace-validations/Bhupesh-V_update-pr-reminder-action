# update-pr-reminder-demo

This repository demos the github action workflow to automatically remind pull request authors to update their branch as soon as there is a new commit (or merge) in the base branch

### Checkout [this pull request for demo](https://github.com/Bhupesh-V/update-pr-reminder-demo/pull/1)

### Workflow

```yaml
name: PR Update Reminder
on:
  workflow_dispatch:
  push:
    branches:
      - main
      - dev
     
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

jobs:
  pr_update:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Get current branch
        id: current_branch
        run: echo ::set-output name=short_ref::${GITHUB_REF#refs/*/}
      - name: Add comment to remind authors
        shell: bash
        run: |
          all_open_prs=$(gh pr list --base ${{ steps.current_branch.outputs.short_ref }} --json author,number)
          prs_count=$(echo "$all_open_prs" | jq length)
          echo "There are $prs_count currently open"
          for (( c=0; c<$prs_count; c++ )); do
            pr_id=$(basename "$(echo "$all_open_prs" | jq .["$c"].number)")
            author=$(echo "$all_open_prs" | jq .["$c"].author.login | tr -d '"')
            echo "Author for PR $pr_id is $author"
            gh pr comment $pr_id --body "Hey @$author 👋🏽 friendly reminder to update your PR/branch because there was a recent commit ($(git rev-parse HEAD)) to the base branch"
          done
 ```
