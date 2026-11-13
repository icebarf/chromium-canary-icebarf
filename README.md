# Chromium Canary Submission — Amritpal Singh [@icebarf](https://github.com/icebarf)

## 1. YouTube Walkthrough Video
* **Video URL:** [Watch 5-Minute Walkthrough](https://youtu.be/VOa7O6Z7pWA)

## 2. Build Info
* **Chromium Commit ID (`git rev-parse HEAD`):** `c87698ceb42766c833327e820fd8ce76d3fafd24`
* **Target Built:** `chrome`
* **My Laptop OS & RAM:** CachyOS and 16GB RAM + 48GB SWAP

## 3. Architecture Notes

* **Feature A (`X-Candidate-Canary` Header):** The header is added in the **Network Service** utility process — the sandboxed child that owns the sockets and sees every browser-initiated navigation. The browser forwards `--inject-candidate-id` by listing it in `kSwitchNames` in `content/browser/service_host/utility_process_host.cc`, which `UtilityProcessHost::ConstructCommandLine` copies onto the child's command line, and `URLLoaderFactory::CreateLoaderAndStartWithSyncClient` (`services/network/url_loader_factory.cc`) reads it from there and stamps `X-Candidate-Canary` on http/https requests only when the value is non-empty.

* **Feature B (`[CANARY_ACK]` Console Hook):** `console.warn` executes in the **Blink Renderer**, where every console-API message reaches `FrameConsole::ReportMessageToClient` (`third_party/blink/renderer/core/frame/frame_console.cc`). I match on the console-API source plus the exact text `"canary-ping"`, so only that call logs `[CANARY_ACK]` to stderr and every other console message is untouched.


## 4. AI Agent Workflow

* **AI Tool & Model Used:** DeepSeek Harness driven via browser using `deepseek-flash (1017)`

* **How I Used Sub-Agents:** I split research from editing: three sub-agents ran in parallel with mutually exclusive write scopes — one searched `third_party/blink/renderer/core/frame/` for the console funnel and checked Blink's `DEPS` file for legal `base/` includes, one traced how `base::CommandLine` switches get copied onto a utility process's command line under `content/browser/service_host/`, and one located the network chokepoint and proved a browser-initiated top-level navigation actually crosses it. Each was required to return file paths and line numbers as evidence rather than a suggestion, which is how the renderer-side-throttle dead end was ruled out before any code was written. The main agent then made the C++ edits and owned the single `autoninja` build, so no two agents ever wrote the same file or ran `ninja` concurrently.
