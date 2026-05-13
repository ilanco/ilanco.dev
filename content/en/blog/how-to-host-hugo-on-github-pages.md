---
title: 'How to Host Hugo on Github Pages'
date: 2025-04-02T23:49:50+03:00
draft: false
tags:
  - "blog"
  - "Hugo"
  - "github"
  - "domain"
params:
  author: Ilan Cohen
---

# Host Your Site on GitHub Pages

## Prerequisites

Before getting started, make sure you’ve completed the following:

1. Create and log in to your GitHub account
2. Create a repository for your project on GitHub
3. Initialize a local Git repository and connect it to your GitHub repo
4. Set up a Hugo site inside your local repository
5. Verify it works locally using `hugo server`
6. Commit your changes and push them to GitHub

## Setup Guide

### Step 1: Enable GitHub Pages

Go to your repository on GitHub.
Navigate to **Settings → Pages**, then set the **Source** to **GitHub Actions**.
This change applies immediately. No need to save.


### Step 2: Configure Image Caching

Update your Hugo configuration to use the built-in cache directory:

```
[caches]
  [caches.images]
    dir = ':cacheDir/images'
```


### Step 3: Create the Workflow File

Create a GitHub Actions workflow file:

```
mkdir -p .github/workflows
touch .github/workflows/hugo.yaml
```


### Step 4: Add the Deployment Workflow

Paste the following into `.github/workflows/hugo.yaml`:

```
name: Build and deploy
on:
  push:
    branches:
      - main
  workflow_dispatch:
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
defaults:
  run:
    shell: bash
jobs:
  build:
    runs-on: ubuntu-latest
    env:
      DART_SASS_VERSION: 1.98.0
      GO_VERSION: 1.26.1
      HUGO_VERSION: 0.158.0
      NODE_VERSION: 24.14.0
      TZ: Europe/Oslo
    steps:
      - name: Checkout
        uses: actions/checkout@v6
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Go
        uses: actions/setup-go@v6
        with:
          go-version: ${{ env.GO_VERSION }}
          cache: false
      - name: Setup Node.js
        uses: actions/setup-node@v6
        with:
          node-version: ${{ env.NODE_VERSION }}
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Create directory for user-specific executable files
        run: mkdir -p "${HOME}/.local"
      - name: Install Dart Sass
        run: |
          curl -sLJO "https://github.com/sass/dart-sass/releases/download/${DART_SASS_VERSION}/dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          tar -C "${HOME}/.local" -xf "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          rm "dart-sass-${DART_SASS_VERSION}-linux-x64.tar.gz"
          echo "${HOME}/.local/dart-sass" >> "${GITHUB_PATH}"
      - name: Install Hugo
        run: |
          curl -sLJO "https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          mkdir "${HOME}/.local/hugo"
          tar -C "${HOME}/.local/hugo" -xf "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          rm "hugo_extended_${HUGO_VERSION}_linux-amd64.tar.gz"
          echo "${HOME}/.local/hugo" >> "${GITHUB_PATH}"
      - name: Verify installations
        run: |
          echo "Dart Sass: $(sass --version)"
          echo "Go: $(go version)"
          echo "Hugo: $(hugo version)"
          echo "Node.js: $(node --version)"
      - name: Install Node.js dependencies
        run: |
          [[ -f package-lock.json || -f npm-shrinkwrap.json ]] && npm ci || true
      - name: Configure Git
        run: git config core.quotepath false
      - name: Cache restore
        id: cache-restore
        uses: actions/cache/restore@v5
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: hugo-${{ github.run_id }}
          restore-keys: hugo-
      - name: Build the site
        run: |
          hugo build \
            --gc \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/" \
            --cacheDir "${{ runner.temp }}/hugo_cache"
      - name: Cache save
        id: cache-save
        uses: actions/cache/save@v5
        with:
          path: ${{ runner.temp }}/hugo_cache
          key: ${{ steps.cache-restore.outputs.cache-primary-key }}
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
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
```


### Step 5: Push Your Changes

Commit the workflow file and push it to GitHub.


### Step 6: Monitor the Build

Go to the **Actions** tab in your repository to track the deployment process.


### Step 7: Wait for Deployment

Once the workflow completes successfully, the status indicator will turn green.


### Step 8: View Your Live Site

Click on the latest workflow run, then open the **Deploy** step to find your site URL.


## What Happens Next

Every time you push changes to your repository, GitHub Actions will automatically rebuild and deploy your site to GitHub Pages.

## DNS Records for Your Custom Domain

Use the table below to find the correct DNS records to setup your custom domain for GitHub Pages.


### Apex Domain Configuration

To set up your root domain (e.g. `example.com`), you have two options:

* **Option 1:** Add all listed **A** and **AAAA** records
* **Option 2:** Add a single **ALIAS** or **ANAME** record

If you want both the apex domain and a `www` subdomain (e.g. `example.com` and `www.example.com`), configure the apex domain first, then add the subdomain.

### DNS Records Reference

| Scenario                                          | Record Type   | Name      | Value                                                                                    |
| ------------------------------------------------- | ------------- | --------- | ---------------------------------------------------------------------------------------- |
| Apex domain (`example.com`)                       | A             | @         | 185.199.108.153<br>185.199.109.153<br>185.199.110.153<br>185.199.111.153                 |
| Apex domain (`example.com`)                       | AAAA          | @         | 2606:50c0:8000::153<br>2606:50c0:8001::153<br>2606:50c0:8002::153<br>2606:50c0:8003::153 |
| Apex domain (`example.com`)                       | ALIAS / ANAME | @         | `USERNAME.github.io` or `ORGANIZATION.github.io`                                         |
| Subdomain (`www.example.com`, `blog.example.com`) | CNAME         | subdomain | `USERNAME.github.io` or `ORGANIZATION.github.io`                                         |

### Summary

* Use **A/AAAA records** or an **ALIAS/ANAME** for apex domains
* Use a **CNAME record** for subdomains

