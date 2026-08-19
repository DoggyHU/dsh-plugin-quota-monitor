# dsh-plugin-quota-monitor

A DSH (DeepSeek Harness) **quota & balance monitor** in the sidebar footer: an always-on Rage bar plus per-provider status rows, with a full config page under **Settings → Plugin Management → Balance Monitor**.

RPG mapping: **HP (red) = monthly · MP (blue) = weekly · SP (yellow) = 5h quota · Rage (gold) = DeepSeek balance**.

## Features

- **Rage (gold, always shown)**: live DeepSeek balance from the official API (`api.deepseek.com/user/balance`), remaining ¥.
- **OpenCode Go** (`opencode-go`): the **monthly / weekly / 5h** windows from the official usage API → HP / MP / SP, remaining `$` + thin bar.
- **SCNet (国家超算中心)**: Token Plan **Credits remaining** (single green row). SCNet has **no public usage API**, so this plugin estimates it **locally**: it reads DSH's own session logs (`$DSH_HOME/sessions/**/*.jsonl(.zstd)`, pure-Node multi-frame Zstandard decode), sums the tokens DSH consumed through scnet this month, and converts them to Credits using the official scnet rate table.
- **Auto provider detection**: default `auto` — resolves from `agent-default-model.provider` in DSH `settings.yaml` (opencode-go / scnet), with a manual override.
- **Settings page**: data source switch, per-meter toggles, scnet monthly quota and an **editable model rate table** (JSON; includes 13 official rates such as DeepSeek-V4-Flash-0731 / GLM-5 / Kimi-K3 / MiniMax-M3 / Qwen3.8-Max).
- 60s polling + instant refresh on tab focus; collapses to a round badge in the narrow rail.

## Install

```bash
dsh plugin --profile web add dsh-plugin-quota-monitor-<version>.tgz
# or from npm
dsh plugin --profile web add dsh-plugin-quota-monitor
```

Restart the dsh web process after install (both the client bundle and the host need a process restart; a page refresh is not enough).

Keys are read from env vars or `~/.dsh/.credentials.yaml` (env wins):

| Source | Key |
|---|---|
| DeepSeek Rage | `DEEPSEEK_API_KEY` |
| OpenCode Go | `OPENCODE_GO_API_KEY` |
| SCNet Credits (local estimate) | no key — reads local session logs |

## Settings

Open **Settings → Plugin Management → Balance Monitor**:

- **Data source**: Auto (follow default model) / OpenCode Go / SCNet.
- **OpenCode meters shown**: monthly (HP) / weekly (MP) / 5h (SP) toggles.
- **SCNet Credits**: monthly quota (default 60,000, 基础版) and the model rate table (Credits per M tokens, editable JSON).
- **Always show the DeepSeek Rage row**.

Config is stored at `$DSH_HOME/storages/quota-monitor-config.json`.

## On the accuracy of the SCNet estimate

- SCNet's official docs confirm the Token Plan has **no public usage/balance API**; usage is only visible in the web console.
- This plugin counts **tokens DSH actually recorded** (`inputTokens / outputTokens / cacheReadTokens` per turn) and converts them with the official scnet rates, so it **tracks your real DSH usage automatically** and backfills the whole natural month.
- Models without a public rate (e.g. SCNet-Max, Qwen3.6-Flash, DeepSeek-R1 series) fall back to a default rate; add their rates in the settings page to refine.
- Usage of scnet **outside DSH** is not included.

## Development

```bash
npm pack                                   # build tarball
dsh plugin --profile web remove dsh-plugin-quota-monitor
dsh plugin --profile web add ./dsh-plugin-quota-monitor-<version>.tgz
```

- `lib/index.js` — host half: `/balance` RPC channel (snapshot / opencode / scnet / configGet / configSet / detect).
- `lib/client.js` — browser half: sidebar card + settings page (hand-written classic-script bundle, zero build).

## Credits / Derivation

Heavily extended from [jelly-000/dsh-balance-monitor](https://github.com/jelly-000/dsh-balance-monitor) (MIT).

## License

[MIT](./LICENSE)
