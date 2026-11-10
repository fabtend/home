# fab0 home

The public home of fab0: the published JSON Schemas, served at `https://schema.fab0.dev/`, the guide for writing a fab0 plugin, the site at `fab0.dev`, the recipe library of tested pipelines, and the setup script that prepares a GitHub account for them.

Everything here is authored and tested in `fabtend/fab0`, and copied here by one of two releases there. Each release opens a pull request from `fab0-release[bot]` that a person reviews and merges. Don't edit these files by hand.

| Path | Written by | What it holds |
| --- | --- | --- |
| `schema/` | fab0's engine release, on its tag `v<version>` | the JSON Schemas (`pipeline.v2.json`, `envelope.v3.json`) and `_headers`, the output directory Cloudflare Pages serves at `https://schema.fab0.dev/` |
| `plugins.md` | fab0's engine release, on its tag `v<version>` | the guide for writing a fab0 plugin |
| `site/` | fab0's engine release, on its tag `v<version>` | the site at `fab0.dev`, served by Cloudflare Pages: `index.html`, `privacy.html` (the privacy notice, at `/privacy`), `style.css` and `_headers` |
| `recipes/` | the `fab0-setup` plugin's release, on its tag `fab0-setup-v<version>` | pipeline documents for `.fab0/pipelines.yml`, each valid against `pipeline.v2.json`; this repository's CI validates them |
| `setup/` | the `fab0-setup` plugin's release, on its tag `fab0-setup-v<version>` | `fab0-setup.mjs`, the script that makes the issue types and status labels a pipeline uses |

The engine's release replaces `schema/` and `site/` whole on each tag. The two releases are independent: a `fab0-setup` release changes only `recipes/` and `setup/`, and an engine release never touches them. This README and `.github/`, its CI, are this repository's own.

This repository holds no skill and no routine settings. A routine that fab0 fires works its step through fab0's Work connector, and its repository needs nothing from fab0: [the `fab0` plugin's README](https://github.com/fabtend/claude-plugin/tree/main/fab0#run-steps-in-a-routine) says how to set one up.
