---
title: Host on Vercel
description: Host your site on Vercel with continuous deployment.
categories: [hosting and deployment]
keywords: [hosting]
menu:
  docs:
    parent: hosting-and-deployment
toc: true
---

## Prerequisites

Please complete the following tasks before continuing:

1. [Create a Vercel account]
1. [Install Go]
1. [Install Hugo]
1. [Create a Hugo site] and test it locally with `hugo server`.

[Create a Vercel account]: https://vercel.com/signup
[Install Go]: https://go.dev/doc/install
[Install Hugo]: https://gohugo.io/installation/
[Create a Hugo site]: /getting-started/quick-start/

## Procedure

Step 1
: Create a Vercel project.

Step 2
: Connect your repository with Vercel.

Step 3
: Visit your Vercel project. From the main menu choose **Settings**&nbsp;>&nbsp;**Pages**. In the center of your screen you will see this:

![screen capture](gh-pages-1.png)
{style="max-width: 280px"}

Step 4
: Change the **Source** to `GitHub Actions`. The change is immediate; you do not have to press a Save button.

![screen capture](gh-pages-2.png)
{style="max-width: 280px"}

Step 5
: Create a file named `hugo.yaml` in a directory named `.github/workflows`.

```text
mkdir -p .github/workflows
touch hugo.yaml
```

Step 6
: Copy and paste the YAML below into the file you created. Change the branch name and Hugo version as needed.

{{< code file=.github/workflows/hugo.yaml copy=true >}}
# Sample workflow for building and deploying a Hugo site to GitHub Pages
name: Deploy Hugo site to Pages

on:
  # Runs on pushes targeting the default branch
  push:
    branches:
      - main

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# Sets permissions of the GITHUB_TOKEN to allow deployment to GitHub Pages
permissions:
  contents: read
  pages: write
  id-token: write

# Allow only one concurrent deployment, skipping runs queued between the run in-progress and latest queued.
# However, do NOT cancel in-progress runs as we want to allow these production deployments to complete.
concurrency:
  group: "pages"
  cancel-in-progress: false

# Default to bash
defaults:
  run:
    shell: bash

jobs:
  # Build job
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.141.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Install Dart Sass
        run: sudo snap install dart-sass
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Install Node.js dependencies
        run: "[[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true"
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: America/Los_Angeles
        run: |
          hugo \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  # Deployment job
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
{{< /code >}}

Step 7
: Commit and push the change to your GitHub repository.

```sh
git add -A
git commit -m "Create hugo.yaml"
git push
```

Step 8
: From GitHub's main menu, choose **Actions**. You will see something like this:

![screen capture](gh-pages-3.png)
{style="max-width: 350px"}

Step 9
: When GitHub has finished building and deploying your site, the color of the status indicator will change to green.

![screen capture](gh-pages-4.png)
{style="max-width: 350px"}

Step 10
: Click on the commit message as shown above. You will see this:

![screen capture](gh-pages-5.png)
{style="max-width: 611px"}

Under the deploy step, you will see a link to your live site.

In the future, whenever you push a change from your local repository, GitHub will rebuild your site and deploy the changes.

## Customize the workflow

The example workflow above includes this step, which typically takes 10&#8209;15 seconds:

```yaml
- name: Install Dart Sass
  run: sudo snap install dart-sass
```

You may remove this step if your site, themes, and modules do not transpile Sass to CSS using the [Dart Sass] transpiler.

[Dart Sass]: /hugo-pipes/transpile-sass-to-css/#dart-sass

## Other resources

- [Learn more about GitHub Actions](https://docs.github.com/en/actions)
- [Caching dependencies to speed up workflows](https://docs.github.com/en/actions/using-workflows/caching-dependencies-to-speed-up-workflows)
- [Manage a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages)
