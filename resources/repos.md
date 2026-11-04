# Repository quick map

Flat checkout:

```
~/repos/xtop-cli/
  api/        contracts
  xtop/       kernel
  widgets/    renderers
  layouts/    arrangement
  plugins/    functionality and runtime widget hosts
  extensions/ server-style integrations
  effects/    frame effects
  web/        site and docs mirror
  context/    this repository
```

## When to touch which repository

| Task | Repository | Contract to read |
|---|---|---|
| Add or change a metric type | `api` | `plugin-api` data model |
| Add a plugin context capability | `api` | `plugin-contract` |
| Change widget rendering inputs | `api` | `widget-contract` |
| Add or change an extension protocol item | `api` | `extension-contract` |
| Add or change an effect protocol item | `api` | `effect-contract` |
| Add a widget | `widgets` | `widget-contract` |
| Add a layout | `layouts` | layout schema |
| Add a plugin | `plugins` then `xtop` | `plugin-contract` |
| Add a runtime widget example | `plugins` | runtime widget docs |
| Add an extension | `extensions` then `xtop` | `extension-contract` |
| Add an effect | `effects` then `xtop` | `effect-contract` |
| Change kernel behavior | `xtop` | data model and render contract |
| Change platform probes | `xtop` | data model |
| Change public docs | upstream repo then `web` | web content model |
| Package a release | `xtop` and AUR | kernel operations |

## Verification entry points

```
cd ~/repos/xtop-cli/api        && ./scripts/ci.sh
cd ~/repos/xtop-cli/xtop       && ./scripts/ci.sh && ./scripts/audit.sh
cd ~/repos/xtop-cli/widgets    && ./scripts/ci.sh
cd ~/repos/xtop-cli/layouts    && ./scripts/ci.sh
cd ~/repos/xtop-cli/plugins    && ./scripts/ci.sh
cd ~/repos/xtop-cli/extensions && ./scripts/ci.sh
cd ~/repos/xtop-cli/effects    && ./scripts/ci.sh
cd ~/repos/xtop-cli/web        && npm ci && npm run build
```
