---
date: '2025-06-18T09:52:33+08:00'
draft: false
title: 'Initial_post'
---

Finally get things working!

[x] test image uploading before writing more. 

### Github Actions

Remember to checkout using private token

```yaml
name: Release

# Triggered by pushing on main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
        with:
          submodules: true
          fetch-depth: 0
          token: ${{ secrets.PRIVATE_TOKEN }}

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest

      - name: Build
        run: hugo --minify

      - name: Deploy
        if: github.ref == 'refs/heads/main'
        run: |
          cd public
          git config user.email '<your.email>'
          git config user.name '<your.name>'
          git add .
          git commit -m '<your.upload.commit>'
          git push --force origin HEAD:main
```