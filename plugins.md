# Writing a fab0 plugin

fab0 drives pipelines and keeps the record of their runs. Its core decides nothing for you. Its MCP server teaches fab0 itself: the `guide` tool answers the pipeline format, the recipes and the path from nothing to a first run, so a person starts by chat with the server alone, and the `fab0` plugin installs that server and nothing else. A plugin outside the core adds what a chat can't do, in a shell or a repository, such as `fab0-setup`, which keeps pipelines in a repository. This guide says how you write one.

## 1. What a fab0 plugin is

A fab0 plugin is a Claude Code plugin that helps people use fab0. It might write pipelines in a house style, check a repository before its first run, or explain a run's record. It is yours: you version it, release it and list it in your own marketplace, and it is never tied to a release of fab0.

fab0 keeps no list of plugins. Nothing in fab0 registers, approves or loads yours.

## 2. What it may depend on

Only fab0's public contract, which changes only with a version:

- **The tools and their schemas,** published as `schema/tools/*.json` and answered by `tools/list`: their names, inputs and answers.
- **The JSON Schemas** at `https://schema.fab0.dev/`: the pipeline (`pipeline.v2.json`) and the envelope (`envelope.v5.json`).
- **The envelope `fab0: 5`,** what an executor reads with `get_step`.

Everything else may change without notice: the text of a tool's description, the server's instructions, what a `guide` topic says, the shape of a log line, and how fab0 stores a run.

The contract is v5, which adds to v4 and takes nothing from it: a v4 client and a v4 pipeline work as they did. What it adds:

- **An executor's `rights`,** in the pipeline: `step`, the default, or `runs`, on an `http` or `claude-routine` executor. With `runs`, the executor's step id also calls `floor`, `list_runs`, `get_run`, `get_record`, `start_run`, `signal` and `cancel_run` on its tenant's runs, while its attempt is live.
- **`executor:<name>` in a transition's `by`,** beside logins and groups: the agent of that executor of the same pipeline may send the signal.
- **The envelope is `fab0: 5`,** v4's with `rights`.
- **An agent as an actor.** In `get_record`, an `attempt` actor may carry `run`, `pipeline` and `executor`: the attempt whose step id made the call. `get_run` names it `attempt:<run>:<id>`, and a waiting signal's `by` may hold `executor:<name>`.
- **The validation error `executor-rights`:** a `by` names an executor whose step id sends no signal.
- **A step id on seven tools.** `floor`, `list_runs`, `get_run`, `get_record`, `start_run`, `signal` and `cancel_run` take an optional `step`, an agent's step id, and `POST /api/v1/<tool>` takes one as its bearer.
- **One MCP server for a person and an agent.** `https://mcp.fab0.dev/mcp` serves a signed-in person, and, with no sign-in, an agent that passes its step id.
- **`guide { topic? }`,** which needs no sign-in: fab0's own guide, by topic. It answers the pipeline format, executors, GitHub's events, secrets, rights and the recipes, as the server that answers serves them. A plugin's skill may send its model there for the format, and keeps no copy of it. Three prompts, `foreman`, `start` and `pipeline`, and the recipes as resources, `fab0://recipes/<name>`, come with it.
- **`floor { tenant? }`,** one read of a tenant's active runs: each run's state and what it waits on, what needs attention, and the runs that failed or were cancelled in the last 24 hours, with a text a person can read as it is. Whoever may call `list_runs` may call it. It reads up to 100 active runs and has no pages.

What v4 changed from v3:

- **`sync_pipelines` is gone.** fab0 reads no file of a repository: put each pipelines file with `put_pipeline`, the one way in. A tenant's pipelines may sit in any number of files, by convention `.fab0/pipelines/<name>.yml`, one pipeline each.
- **`remove_pipeline { tenant?, name }`** retires a pipeline, which a put never does by leaving its name out. Runs already started finish on their version, and a later put makes it active again. It needs the tenant's admin.
- **The resource `fab0://pipelines/{tenant}/{name}` is listed** by `resources/list`, each active pipeline with its version and source.
- **`whoami`** answers each tenant's `app`, whether the fab0 GitHub App is installed on it, and always answers `accounts`.
- **The store warning `github-without-app`:** a pipeline waits on a `github.` signal in a tenant that has neither the App nor a `github` source, which receives none.
- **The envelope became `fab0: 4`,** otherwise as v3's; it is `fab0: 5` now.
- **Two widenings made within v3** are part of it too: `get_record`'s actor may be a source's (`ext.<source>`) or a workflow's, and the warning `ext-unchecked`.

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

An agent that works pipeline steps needs no plugin. fab0 fires it with a step id, and it works the step through fab0's MCP server (`https://mcp.fab0.dev/mcp`), where the tools that work a step take the step id as `id` and no sign-in is needed, or through `POST /api/v1/work/<tool>` with the step id as its bearer. The executor contract describes both. A call that carries a step id is the step's, whoever is signed in on the connector it came through, and a call that leaves the id out on a signed-in connector is the person's. So one connector serves a person and their routines, and a person whose agents read input they don't trust gives their routines a second connector, never signed in.

## 6. An example

`fab0-setup`, in [`fabtend/claude-plugin`](https://github.com/fabtend/claude-plugin), is fabtend's own. It depends on `fab0`, reads the pipeline format, the validation codes and the recipes from `guide` and keeps no copy of them, calls `whoami`, `validate_pipeline`, `put_pipeline` and `remove_pipeline`, commits each pipeline's file through a pull request, puts each pipelines file itself after a merge, retires with the person's yes the pipelines no file holds any more, which it finds from the merge's diff and the listed `fab0://pipelines/` resources, ships a script that prepares a GitHub account, and has its own version and release. Its 4.0.0 needs contract v5, for `guide`; its 3.0.0 needed contract v4.
