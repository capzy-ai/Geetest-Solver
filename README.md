<div align="center">

<img src="https://capzy.ai/capzy-logo.svg" alt="Capzy" width="220" />

# GeeTest v3 + v4 Solver

**Solve GeeTest slider, click, and icon challenges. Auto-routes v3 vs v4.**

[![Solve cost](https://img.shields.io/badge/from-%240.001%20%2F%20solve-%23ff5d2a)](https://capzy.ai/solvers)
[![Speed](https://img.shields.io/badge/avg%20solve-~8%20seconds-%2322c55e)](https://capzy.ai/solvers/geetest)
[![Uptime](https://img.shields.io/badge/uptime-99.9%25-%2322c55e)](https://capzy.ai/status)
[![License: MIT](https://img.shields.io/badge/license-MIT-%23ff5d2a)](LICENSE)

[Live Demo](https://capzy.ai/solvers/geetest/demo) ·
[Get Free $0.10 Credit](https://capzy.ai/auth/register) ·
[Dashboard](https://capzy.ai/dashboard) ·
[Full Docs](https://capzy.ai/docs) ·
[Pricing](https://capzy.ai/solvers)

</div>

---

## What this repo is

Copy-pasteable examples for solving **GeeTest v3 + v4** through the
[Capzy](https://capzy.ai) HTTP API — no SDK required. Pure curl, Python,
and Node.js using the raw API. Easy to read, easy to port, easy to audit.

## What is GeeTest v3 + v4?

GeeTest is a Chinese / international captcha used on e-commerce, gaming, and financial sites. Offers slider puzzles (drag to complete), click-in-order, and icon selection. v3 uses `gt` + `challenge`; v4 uses `captchaId`. Send whichever you find — Capzy's router detects the version automatically.

## Why Capzy

- **From $0.001 per solve.** Flat pricing — no tiers, no retainer, no monthly minimum.
- **~8 seconds average solve.** Production-grade speed.
- **Drop-in compatible.** `createTask` / `getTaskResult` protocol. If your code already speaks the standard solver shape, swap the host to `https://api.capzy.ai`.
- **$0.10 in real credits on sign-up.** No card. 100 free test solves.

## Pricing

| Task type | When to use | Cost / solve |
|-----------|-------------|-------------:|
| `GeeTestTaskProxyLess`             | Proxyless (Capzy supplies the IP) | **$0.001**   |
| `GeeTestTask`                       | You supply the proxy              | **$0.001**   |

For consistency across the target site, use the proxy variant with the
**same proxy your session is already running through** — the solver
mints the token from that IP, so when you submit it back through the
same proxy everything looks consistent.

## 60-second quickstart

```bash
# 1. Sign up — gets you $0.10 in free credits (100 solves)
open https://capzy.ai/auth/register

# 2. Copy your API key from the dashboard
#    https://capzy.ai/dashboard/api-keys

# 3. Run any example
export CAPZY_KEY="capzy_..."
bash examples/curl/basic.sh
```

Minimal Python:

```python
import requests, time

KEY = "capzy_xxxxxxxxxxxxxxxxxxxxxxxx"

# 1) Create the task
created = requests.post("https://api.capzy.ai/createTask", json={
    "clientKey": KEY,
    "task": {
        "type": "GeeTestTaskProxyLess",
        "websiteURL": "https://example.com",
        "captchaId": "647f5ed2ed8acb4be36784e01556bb71"
    },
}).json()
task_id = created["taskId"]

# 2) Poll until ready
while True:
    result = requests.post("https://api.capzy.ai/getTaskResult", json={
        "clientKey": KEY, "taskId": task_id,
    }).json()
    if result["status"] == "ready":
        break
    time.sleep(2)

print(result["solution"])
```

That's the whole protocol. The rest of this repo is just that, in every
language we could think of.

## Pick your language

| Language        | Example                                       |
|-----------------|-----------------------------------------------|
| **curl / bash** | [`examples/curl/basic.sh`](examples/curl/basic.sh)    |
| **Python**      | [`examples/python/basic.py`](examples/python/basic.py) |
| **Node.js**     | [`examples/nodejs/basic.js`](examples/nodejs/basic.js) |

See [`examples/README.md`](examples/README.md) for setup details.

## Request envelope

```json
{
  "clientKey": "capzy_xxxxxxxxxxxxxxxxxxxxxxxx",
  "task": {
    "type": "GeeTestTaskProxyLess",
    "websiteURL": "https://example.com",
    "captchaId": "647f5ed2ed8acb4be36784e01556bb71"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|:--------:|-------|
| `type` | `string` | yes | GeeTestTaskProxyLess or GeeTestTask |
| `websiteURL` | `string` | yes | Full URL of the page |
| `captchaId` | `string` | no | GeeTest v4: the captchaId. Use this OR (gt + challenge). |
| `gt` | `string` | no | GeeTest v3: the `gt` parameter |
| `challenge` | `string` | no | GeeTest v3: the `challenge` parameter (single-use, time-limited) |
| `proxyType` | `string` | no  | http | https | socks4 | socks5 (only for `GeeTestTask`) |
| `proxyAddress` | `string` | no  | IP or hostname of your proxy (only for `GeeTestTask`) |
| `proxyPort` | `integer` | no  | Port number of your proxy (only for `GeeTestTask`) |
| `proxyLogin` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `GeeTestTask`) |
| `proxyPassword` | `string` | no  | Optional — omit if your proxy doesn't require auth (only for `GeeTestTask`) |

Full reference in [`docs/parameters.md`](docs/parameters.md).

## Response shape

When the task is ready (`status: "ready"`), `solution` contains:

| Field | Type | Notes |
|-------|------|-------|
| `captcha_id` | `string` | GeeTest v4: the captchaId echoed back |
| `lot_number` | `string` | GeeTest v4: lot number |
| `pass_token` | `string` | GeeTest v4: pass token for server validation |
| `gen_time` | `string` | GeeTest v4: generation timestamp |
| `captcha_output` | `string` | GeeTest v4: captcha output string |

### Example

```json
{
  "status": "ready",
  "solution": {
    "captcha_id": "647f5ed2ed8acb4be36784e01556bb71",
    "lot_number": "8a2c4e5b3f1d4567a9b8c0d2e3f4a5b6",
    "pass_token": "<long pass token from validation server>",
    "gen_time": "1735689600",
    "captcha_output": "<base64-encoded captcha output>"
  }
}
```

### How to use the result

v4: submit `captcha_id`, `lot_number`, `pass_token`, `gen_time`, and `captcha_output` to the target site's form or API exactly as Capzy returns them.

## Features

- Supports GeeTest v3 (gt + challenge) and v4 (captchaId)
- Slider, click-in-order, and icon-selection variants
- Auto-detects version from the parameters you send
- ~8 second average solve time

## FAQ

**v3 or v4 — how do I tell?** If the page uses `gt` + `challenge`, it's v3. If it uses `captchaId`, it's v4.

**The `challenge` parameter for v3 keeps expiring** It's single-use and time-limited. Fetch it fresh from the site's API right before creating each task.

## What you'll need

- A Capzy API key — [sign up](https://capzy.ai/auth/register) (free, $0.10 credit).
- Network access to `https://api.capzy.ai`.

## Other captcha types

Capzy solves 25+ captcha types. Full catalog at
[capzy.ai/solvers](https://capzy.ai/solvers). Each type has its own
solver repo on [github.com/capzy-ai](https://github.com/capzy-ai).

## License

[MIT](LICENSE).

---

<div align="center">

**[Sign up for free credits →](https://capzy.ai/auth/register)**

Built by [Capzy](https://capzy.ai). Issues + PRs welcome.

</div>
