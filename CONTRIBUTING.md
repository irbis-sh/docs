# Contributing to Irbis docs

> [!NOTE]
> Before proceeding, please read our [Code of Conduct](https://docs.irbis.sh/code-of-conduct), including the section on AI use. Documentation counts as human-to-human communication, so AI tools may only be used for light proofreading.

This repository holds the source for [docs.irbis.sh](https://docs.irbis.sh), built with [Hugo](https://gohugo.io) and the [Hextra](https://github.com/imfing/hextra) theme.

## Reporting a problem

If a page is wrong, out of date or missing something important, and you'd rather not fix it yourself, please [open a new issue](https://github.com/irbis-sh/docs/issues/new) in this repository.

If you have a problem with Zen itself, please use Zen's [issue triage form](https://github.com/irbis-sh/zen-desktop/discussions/new?category=issue-triage) instead.

## What to contribute

We accept two kinds of changes:

1. Fixes to existing pages, like a typo, a broken link, an outdated step or a factual error. Open a pull request directly – no issue is needed.
2. New pages, and rewrites or restructuring of existing ones. Please open an issue first, so we can agree on what the page should cover and where it belongs before you write it.

## Making a change

The docs follow [Diátaxis](https://diataxis.fr). Each page is a how-to guide, reference or explanation, and lives in the matching folder under `content/docs/zen/`. Use British spelling, and en dashes (–) rather than em dashes (—).

To submit:

1. Edit the page. For a fix, the **Edit this page on GitHub** link on the page is enough. For anything larger, install [Hugo](https://gohugo.io/installation/) and [Go](https://go.dev/doc/install), and run `hugo server` in the repository root to preview your changes.
2. If you're adding a page, give it a `title` and a `description` in its front matter, and add a card linking to it in the section's `_index.md`.
3. Open a pull request. Describe what you changed and why, and link the issue if there is one.

By contributing, you agree that your work will be licensed under the [MIT Licence](LICENSE).
