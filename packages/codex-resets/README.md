# @magpie-community/page-codex-resets

**Draft — not ready for the plugin market.** Automatic updates depend on [Magpie host-managed sync support (#3609)](https://github.com/yetone/magpie/issues/3609), which is not implemented in the current host. Approval of the third-party data source is also pending in [plugins#338](https://github.com/magpie-community/plugins/pull/338). The market listing is withheld until both are resolved.

A separate Codex reset news page with global-reset announcements, community probability estimates and reset-credit grants. It preserves Magpie's light/dark theme and the three-column layout, with collapsible sources. The complete interface is available in English, Simplified Chinese, Traditional Chinese, Japanese and German. Source-provided wording is retained and labelled as such.

## Architecture

The intended production flow is:

```text
Magpie owns scheduling and plugin lifecycle
  → permission-scoped public request
  → sync-core.mjs validates and reduces the response
  → host publishes this plugin's own snapshot
  → sandboxed page reads the snapshot and renders it
```

`syncOnce({request, save, options, signal, now})` performs one bounded check. It requires an explicit transport and plugin-scoped publisher, keeps a five-minute minimum check deadline and `Retry-After`, and accepts cancellation before requesting or publishing. It starts no scheduler and has no default global fetch. A failed save propagates to the caller. These injected functions are **our internal module boundary, not a claimed Magpie API**.

The future host adapter must supply the approved transport, isolated state and lifecycle: one in-flight job per plugin; persist backoff across restarts; stop/abort when disabled, removed or shut down; prevent stale publication and conflicting writes. The job should run on the Magpie host for both desktop and remote `magpie web`. See #3609 for acceptance criteria and its relationship to [page/backend RPC #3396](https://github.com/yetone/magpie/issues/3396).

The current page only uses `magpie.ready`, theme events and `magpie.options.get()`. It declares no private reads, makes no requests and exposes no unsupported host methods. The current host does **not** invoke the sync module. The empty state explains that automatic updates are not available. Do not merge or publish this preview as a working live-data plugin.

There is no shipped CLI, npm bin, terminal watcher, terminal bell, installation hook, daemon or OS scheduler. Desktop notifications and changes to the reset-card dialog remain out of scope. A new unexpired explicit announcement can be returned by the sync module as data; it does not send a notification.

## Meaning of the information

- Public announcements help users notice a possible upcoming reset before choosing to spend a saved credit. They offer no private or exclusive signal.
- Global resets, community forecasts and credit grants are distinct. A grant adds resets to use later; it does not refill the current allowance. Missing probability is not 0%.
- Scheduled announcements retain Tibo's original X URL. A vague announcement stays without an exact time. A community probability is interpretation, not an OpenAI commitment or advice to delay using a credit.
- An announced time passing becomes **Awaiting confirmation**, not proof of completion. Eligibility and actual allowance belong to the user's account.
- Failed, malformed or stale data is unavailable, never “no reset.” Snapshot freshness is 15 minutes; scheduled announcements older than 48 hours are treated as unverifiable; expired forecasts are omitted.
- The page rereads its snapshot every 30 seconds while loaded. **Refresh page** rereads that snapshot and does not fetch a public service. Source addresses can be selected and copied because the current iframe sandbox forbids popups.

## Source under review

The prototype uses [Codex Resets](https://codex-resets.com) and its [public API](https://codex-resets.com/api/docs). The documentation allows keyless use with attribution. This permission does not settle whether Magpie should accept the service, its interpretations or its probability estimates. Attribution and original X addresses are retained; no website code or X scraping is included.

The one required destination is **`https://codex-resets.com/api/v1/status`**, using GET, an eight-second timeout, no credentials, no redirects, and a 64 KiB body cap. Backoff respects `Retry-After` with a five-minute floor and 24-hour cap. Only required validated public fields are retained. No accounts, credentials, quotas or agent files are read.

## Development and verification

Install the local preview into a **dedicated sandbox HOME**, then seed fixture options through the official CLI or the browser test below. This is for reviewing the UI, not an end-user installation workflow:

```sh
magpie plugin add ./packages/codex-resets
```

`dev/sync.mjs` is an optional single-request smoke helper for a developer's sandbox, excluded from the package. It explicitly supplies `fetch` and uses the official CLI to write only this plugin's options. It has no watch/bell mode. Do not run it as a background service or recommend it to marketplace users. Normal tests never contact the public service.

Run the repository checks in a temporary HOME so no real sign-in is touched:

```sh
HOME=$(mktemp -d) bun test
bun scripts/check.mjs
```

Focused tests cover parsing, source URLs, expiry, probability boundaries, failure/backoff, data minimization, cancellation, explicit transport, script/permission policy, complete translated messages and placeholder parity. Node can also run them with `node --test packages/codex-resets/*.test.mjs`.

`browser.test.mjs` is optional and uses a **separately running, isolated** Magpie web and development Playwright. Set:

- `MAGPIE_UI_TEST=1`.
- `HOME` and `MAGPIE_TEST_HOME` to the same dedicated directory containing `sandbox` in its name; isolate `XDG_CONFIG_HOME` and `XDG_CACHE_HOME` there too. Install this preview into that Magpie.
- Configure its gateway on an unused port and run `magpie web --addr 127.0.0.1:<unused-web-port> --no-open`. Never target everyday Magpie.
- `MAGPIE_TEST_URL` to the localhost link including its test key; `MAGPIE_TEST_BIN` to the tested binary.
- `PLAYWRIGHT_MODULE` to Playwright's development `index.mjs`; `PLAYWRIGHT_BROWSERS_PATH` if its browsers are outside the sandbox.
- Optionally `MAGPIE_TEST_CHROMIUM_CHANNEL=chrome` to use installed Chrome with a fresh temporary browser profile; the default uses Playwright's bundled Chromium.
- Optionally `MAGPIE_TEST_ENGINES=chromium,webkit` (default Chromium). The tested host's scheme-less CSP sources are rejected by Playwright WebKit before scripts load; the test does not bypass that host limitation.
- Optionally `MAGPIE_SCREENSHOTS` for rendered previews.

Run `node --test packages/codex-resets/browser.test.mjs`. It restores the sandbox settings/options and checks all five languages, light/dark, 1100/760/420px, source selection, refresh, zero probability and unavailable/empty states. Fixtures and screenshots use fictional October 23, 2026 data, including 75%; they are not live forecasts.

中文：此版本仍为草稿，等待 Magpie 提供官方后台同步能力及维护者批准数据源，暂不上架。目标是启用插件后由 Magpie 自动同步，无需终端；本次仅完成同步模块边界、取消处理、页面及五种语言，尚未接入不存在的宿主接口。
