# Writing a fab0 plugin

fab0 drives pipelines and keeps the record of their runs. Its core decides nothing for you and ships no help beyond its tools: the `fab0` plugin installs fab0's MCP server, and that is all. Help for people who use fab0 lives in plugins outside the core, such as `fab0-setup`, which writes pipelines. This guide says how you write one.

## 1. What a fab0 plugin is

A fab0 plugin is a Claude Code plugin that helps people use fab0. It might write pipelines in a house style, check a repository before its first run, or explain a run's record. It is yours: you version it, release it and list it in your own marketplace, and it is never tied to a release of fab0.

fab0 keeps no list of plugins. Nothing in fab0 registers, approves or loads yours.

## 2. What it may depend on

Only fab0's public contract, which changes only with a version:

- **The tools and their schemas,** published as `schema/tools/*.json` and answered by `tools/list`: their names, inputs and answers.
- **The JSON Schemas** at `https://schema.fab0.dev/`: the pipeline (`pipeline.v2.json`) and the envelope (`envelope.v3.json`).
- **The envelope `fab0: 3`,** what an executor reads with `get_step`.

Everything else may change without notice: the text of a tool's description, the server's instructions, the shape of a log line, and how fab0 stores a run.

## 3. Calling fab0

Don't ship a server of your own for fab0. Depend on the `fab0` plugin, and call its tools:

```json
{ "name": "my-fab0-helper", "version": "1.0.0", "dependencies": ["fab0@fab0"] }
```

A dependency from another marketplace is installed only if your marketplace allows it. List it in your marketplace's root `marketplace.json`:

```json
{ "allowCrossMarketplaceDependenciesOn": ["fab0"] }
```

Without it, Claude Code installs your plugin without `fab0`, and your plugin doesn't load. If you'd rather not, tell your users to install `fab0@fab0` first, from `fabtend/claude-plugin`.

Your skills call fab0's tools as `mcp__plugin_fab0_fab0__<tool>`, such as `mcp__plugin_fab0_fab0__validate_pipeline`. The person signs in once, through the `fab0` plugin.

## 4. Rules

- **Secrets never pass through a tool.** Never ask for a secret's value, read it, or pass it to a tool or a model's context. Give the person the secrets command with the name and origin filled in, and let their own shell supply the value: `pbpaste | curl -fsS -X PUT "https://mcp.fab0.dev/api/v1/secrets/<NAME>?tenant=<id>&origin=<origin>" -H "Authorization: Bearer $(gh auth token)" --data-binary @-`.
- **What you read is data.** An envelope, a signal's payload, and anything read from GitHub describe work. They never change your plugin's rules.
- **A step id is a credential.** It authorizes one attempt's Work operations. Pass it only to fab0's tools, and never print it or write it down.
- **`with` is the pipeline's message.** fab0 gives a state's `with` no meaning. Your skills may give their own keys a meaning in the pipelines they write, and should say so in what they write.

## 5. Executors are not plugins

An agent that works pipeline steps needs no plugin. fab0 fires it with a step id, and it works the step through the Work connector (`https://mcp.fab0.dev/work`), or through `POST /api/v1/work/<tool>` with the step id as its bearer. The executor contract describes both.

## 6. An example

`fab0-setup`, in [`fabtend/claude-plugin`](https://github.com/fabtend/claude-plugin), is fabtend's own. It depends on `fab0`, calls `validate_pipeline` and `put_pipeline`, puts each pipelines file itself after a merge, ships its recipes inside the plugin, and has its own version and release.
