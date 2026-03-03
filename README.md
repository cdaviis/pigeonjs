<p align="center">
  <img src="assets/logo.png" alt="open-message logo" width="180" />
</p>

<h1 align="center">open-message</h1>

<p align="center">
  Template-first messaging for Slack and more.
</p>

---

open-message lets you define your messages as YAML or JSON templates — including the destination, formatting, and variables — and send them from the CLI or your Node.js code.

## Features

- **Template-first** — message structure, destination, and variables all live in one file
- **YAML or JSON** templates
- **CLI** for use in CI/CD pipelines
- **Programmatic API** for use in Node.js applications
- **Variable interpolation** with `{{var}}`, env vars (`{{MY_ENV_VAR}}`), and built-ins (`{{now}}`, `{{uuid}}`)
- Built-in adapter for **Slack** (Block Kit)

## Documentation

Detailed **template and DSL** docs (templates, variables, Slack blocks and elements):

| Doc | Description |
|-----|-------------|
| [DSL overview](docs/dsl/01-overview.md) | What the template DSL is, Slack shorthand vs raw Block Kit |
| [Template structure](docs/dsl/02-template-structure.md) | `version`, `name`, `destination`, `variables`, `message` |
| [Variables](docs/dsl/03-variables.md) | Interpolation, `{{tokens}}`, resolution order, built-ins, env |
| [Slack blocks](docs/dsl/04-slack-blocks.md) | header, section, context, divider, actions, image, video, markdown, file, table, raw |
| [Slack elements](docs/dsl/05-slack-elements.md) | button, overflow, select, multi_select, datepicker, timepicker |
| [Examples](docs/dsl/06-examples.md) | Full template examples |

Each doc has **Previous** / **Next** links at the bottom to move through the guide.

## Installation

```bash
npm install open-message
```

For CLI use, install globally:

```bash
npm install -g open-message
```

## Quick start

**1. Create a template** (`./templates/deploy.yml`):

```yaml
version: '1'
name: Deploy Notification
destination:
  service: slack
  settings:
    channel: '#deployments'
variables:
  app_name:
    required: true
  version:
    required: true
  environment:
    default: production
message:
  blocks:
    - type: header
      text:
        type: plain_text
        text: ':rocket: {{app_name}} deployed to {{environment}}'
    - type: section
      fields:
        - type: mrkdwn
          text: '*Version:*\n`{{version}}`'
        - type: mrkdwn
          text: '*Status:*\n:large_green_circle: Success'
    - type: context
      elements:
        - type: mrkdwn
          text: 'Deployed at {{now}}'
```

**2. Set your credentials:**

```bash
export SLACK_TOKEN=xoxb-your-token
```

**3. Send it:**

```bash
open-message send ./templates/deploy.yml --var app_name=api --var version=1.2.3
```

## CLI

```
open-message send <path> [options]        Send a message
open-message validate <path>              Validate a template
open-message list <dir>                   List templates in a directory
```

### `open-message send` options

| Flag | Description |
|---|---|
| `-v, --var <key=value>` | Set a template variable (repeatable) |
| `--dry-run` | Print resolved payload without sending |
| `--json` | Output result as JSON (for scripting) |
| `--config <file>` | Path to a credentials config file |
| `-e, --env-file <file>` | Load a `.env` file for credentials |

### Examples

```bash
# Send with variables
open-message send ./templates/deploy.yml --var app_name=api --var version=2.0.0

# Preview the resolved payload
open-message send ./templates/deploy.yml --var app_name=api --var version=2.0.0 --dry-run

# JSON output for CI/CD scripting
open-message send ./templates/build-report.yml --var branch=main --var status=passing --json

# Validate a template
open-message validate ./templates/slack-deploy.yml

# List templates in a directory
open-message list ./templates
```

## Programmatic API

```typescript
import openMessage from 'open-message';

// Send a message
await openMessage.send('./templates/deploy.yml', { app_name: 'api', version: '1.2.3' });

// With options (dry-run, limits/chunking for long messages)
await openMessage.send('./templates/deploy.yml', { app_name: 'api', version: '2.0.0' }, {
  dryRun: true,
  limits: { maxBlocksPerMessage: 40, maxMessageChars: 8000 },
  chunking: { enabled: true, footerTemplate: 'Part {{ index }} of {{ total }}' },
});
```

Templates can use **multiple destinations** (`destinations` array) to send the same message to several channels or services; see [Template structure](docs/dsl/02-template-structure.md#multiple-destinations).

## Template DSL

All templates share a common shape: `version`, `name`, `destination`, optional `variables`, and `message`. For Slack you can use **DSL shorthand** (e.g. `header`, `section`, `context`) or **raw Block Kit**.

Quick reference:

- **Variable tokens** — `{{my_var}}` (caller/API), `{{MY_ENV_VAR}}` (env, ALL_CAPS only), `{{now}}`, `{{timestamp}}`, `{{uuid}}`. Resolution order: built-ins → caller vars → env (ALL_CAPS) → template defaults.
- **Full reference** — See [Documentation](#documentation) above for the full DSL guide (template structure, variables, Slack blocks and elements, examples).

## Credentials

Credentials are resolved in this priority order (highest wins):

1. Programmatic `opts.credentials`
2. CLI `--config <file>`
3. Environment variables
4. `.open-message.yml` in the current directory
5. `~/.open-message/config.yml`

### Environment variables

```bash
SLACK_TOKEN=xoxb-...
```

### Config file (`.open-message.yml` or `~/.open-message/config.yml`)

```yaml
slack:
  botToken: xoxb-your-token
```

## License

MIT
