---
title: Creating a scheduled Broken Links Checker using Github Actions
date: 2020-01-10 09:19
tags:
  - github-actions
categories:
  - ci
---

## Introduction

Follow this tutorial if you want to prevent dead links creeping into your website.

After completing all steps you will have created a simple Github action that:

- does a monthly check against your live website
- creates a Github Issue if broken links are detected
- optionally assigns a label and/or assignee to the issue

## Example Github Issue

{% asset_img github-issue-example.png 'Example Github Issue if Broken Links are detected' %}

## 1. Create the Action

1. create folder `.github/workflows`
2. in that folder, create file `check-broken-links.yml`
3. insert below code into the newly created file and make sure to update `WEBSITE_URL` so it matches your live website:

```yml
name: Broken Links Checker
on:
  schedule:
    - cron:  '0 1 1 * *'
env:
  WEBSITE_URL: "https://docusaurus-powershell.netlify.com/"
  ISSUE_TEMPLATE: ".github/workflows/check-broken-links.md"

jobs:
  check:
    runs-on: ubuntu-latest

    steps:
    - name: Run Broken Links Checker
      run: npx broken-link-checker $WEBSITE_URL --ordered --recursive

    - uses: actions/checkout@v2
      if: failure()

    - uses: JasonEtco/create-an-issue@master
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      with:
        filename: ${{ env.ISSUE_TEMPLATE }}
      if: failure()
```

**Explanation:**

- the cron schedule runs every 1st day of the month and was [generated here](https://crontab.guru/#0_1_1_*_*)
- the website will be analyzed using [broken-link-checker](https://github.com/stevenvachon/broken-link-checker)
- the github issue will be created using [create-an-issue](https://github.com/marketplace/actions/create-an-issue)

## 2. Create the Issue Template

All that is left now is creating a markodwn template that will be used to create the Github Issue.

1. inside folder `.github/workflows`, create file `check-broken-links.md`
2. insert below code into the newly created file and make sure to:
   - replace the website URL
   - replace the github repository

```markdown
---
title: Website Contains Broken Links
labels: housekeeping
assignees: ''
---

## Website Contains Broken Links

Broken Link Checker found :coffin: links on https://docusaurus-powershell.netlify.com/

[View Results](https://github.com/alt3/Docusaurus.Powershell/commit/{{sha}}/checks)

_Use search filter `─BROKEN─` to highlight failures_
```

**Explanation:**

- The `View Results` link uses the commit hash to point directly to the corresponding Github Actions log

## Done!

All done, you will now be notified if any dead links are detected on your live website.

To run the analysis from your local machine use:

```bash
npx broken-link-checker https://docusaurus-powershell.netlify.com/ --ordered --recursive
```

## Fixing Broken Links

To determine which links need fixing:

1. open the Github Actions log by clicking on the `View results` link (in the Github Issue)
2. expand the log for action "Run Broken Links Checker"
3. optionally use search filter `─BROKEN─` to highlight dead links
4. analyze and solve as needed

{% asset_img github-action-log-example.png 'Example Github Action Log' %}


