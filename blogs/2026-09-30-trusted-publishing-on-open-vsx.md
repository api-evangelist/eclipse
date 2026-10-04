---
title: "Trusted Publishing on Open VSX"
url: "https://blogs.eclipse.org/post/tamas-cservenak/trusted-publishing-open-vsx"
date: "2026-09-30"
author: "Tamas Cservenak"
feed_url: "https://blogs.eclipse.org/blog/feed"
---
Trusted Publishing on Open VSX If you publish a VS Code extension to Open VSX through GitHub Actions, Trusted Publishing lets you replace a stored personal access token with short-lived publishing credentials. This reduces credential management and limits the scope and lifetime of the token used to publish each release. The short version Instead of storing a token, you tell the registry which CI workflow is allowed to publish your extension.
