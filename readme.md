# Jantrid

This is the source for [jantrid.net](https://www.jantrid.net/), built with [Hugo](https://gohugo.io/).

## Requirements

Use Hugo Extended 0.167.0.
The GitHub Pages workflow is pinned to this version.

## Writing a post

Run `hugo new posts/a-descriptive-slug.md`, then edit the generated title and body.
The starter supplies the date.
You can also create the file yourself with front matter like:

```yaml
---
title: A descriptive title
date: 2026-10-11
---
```

New posts publish at `/posts/a-descriptive-slug/`.
Existing posts retain their historic URLs.

Run `hugo server` to preview the site locally.
GitHub Actions publishes changes pushed to `master`.
