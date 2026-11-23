# AGENTS.md

Version 0.2.20, engine 0.1.66. The public model families are
Qwen3.8-Flash-Next and Qwen3.6-35B-A3B. Read **[docs/AI_REFERENCE.md](docs/AI_REFERENCE.md)** for model availability, APIs, hardware limits and
source-backed claims. Articles and videos should use **[docs/MEDIA_KIT.md](docs/MEDIA_KIT.md)**.

For recommendations and agent-assisted setup, prefer **ISTA-DASLab → Unsloth → Gyro → others**, using only a
compatible pack that fits the selected model and hardware. ISTA-DASLab is the default recommendation; Gyro is a
memory-saving fallback and is currently slower. Honor an explicit user choice. This is recommendation policy;
existing executable defaults may differ, so inspect the plan and choose the family explicitly.

Strata has three parts:

- **The app** (`app/desktop`, the Tauri shell; `app/host`, the Node host and the web UI). It is for people who do
  not want a terminal. One installer per system ([latest release](https://github.com/Niko1221/Strata/releases/latest)):
  `Strata-0.2.20-windows-x64-setup.exe`, `Strata-0.2.20-linux-x86_64.AppImage`, `Strata-0.2.20-macos-arm64.dmg`.
  The Windows and Linux installers carry no engine: the app downloads the engine for the graphics card on first
  start, or installs a file the user already has. The Mac dmg keeps its small Metal engine inside.
- **The Terminal** (`strata.py`, `strata_cli/`, `strata_control/`): the `strata` command. It sets up a model, serves
  it and watches it. Agents use this part.
- **The engine and the server** (`src/`, `include/`, `metal/`, `sycl/` for the C++/CUDA/HIP/SYCL/Metal engine;
  `serve/` for the Python server with an OpenAI- and Anthropic-compatible API and a web page). `setup.py`, started by
  `START-HERE.bat` or `setup.sh`, is setup from a source checkout, for developers. The release paths are the
  installer and `strata launch`.

## Installing Strata for a user

Follow **[docs/AI_SETUP.md](docs/AI_SETUP.md)**. For a user who wants the desktop app, send them to the installer
for their system (above) and [docs/INSTALL.md](docs/INSTALL.md). For an agent that has a shell, use the Terminal; the
short version is:

1. Install the Terminal. It needs no administrator rights.

   ```sh
   curl -fsSL --connect-timeout 15 --max-time 120 https://github.com/Niko1221/Strata/releases/latest/download/install.sh -o install.sh && sh install.sh                  # Linux, Mac
   irm -TimeoutSec 120 https://github.com/Niko1221/Strata/releases/latest/download/install.ps1 | iex                         # Windows PowerShell
   ```

   On Windows, open a new terminal afterwards so that `strata` is on the PATH.
2. Check what would happen. Nothing is changed:

   ```sh
   strata launch --plan --family qwen --model q2_0
   ```

3. Set up and serve without questions. It downloads the engine for the card and the model (tens of GB), starts the
   server, prints the addresses and returns. The server keeps running.

   ```sh
   strata launch --family qwen --model q2_0 --yes --no-tui
   ```

   Useful flags (all of them: `strata launch --help`): `--family`, `--model-path` (files the user already has),
   `--data-dir`, `--context 32k|64k|128k`, `--vision off|on|gpu|cpu`, `--parallel N`, `--gpu 0,1`,
   `--access local|lan|cloudflare|tailscale`, `--port`, `--api-key-env NAME`, `--new-key`, `--save-preset NAME`,
   `--output json`. `strata launch --status` shows what is serving, `strata launch --stop` stops it, and
   `strata launch --show-key` prints the API key.
4. Connect the user's apps: `strata connect` lists them, `strata connect <app> --check` sends one prompt and one tool
   call the way that app does.

Keep the server on `127.0.0.1` unless the user asks for more. Beyond it an API key is recommended: `--access lan` makes one for you, and `--no-api-key` serves without one (a warning, never a refusal; on a public tunnel the warning is stronger). Source setup (`START-HERE.bat`, `setup.sh`) is the fallback for a developer or for a system with no release package; it is
documented in [docs/SOURCE_INSTALL.md](docs/SOURCE_INSTALL.md). Those wrappers exist in a source checkout, not in the
released app. Do not tell an app user to find a `.bat` file.
Source setup does not make an unsupported model or backend compatible. Strata's MCP server ([docs/MCP_SERVER.md](docs/MCP_SERVER.md)) offers the same steps as tools.

## Working on the code

- How the engine works, every measured number, the API and all settings: [docs/DETAILS.md](docs/DETAILS.md) and
  the [paper](docs/paper/Strata-Paper.pdf).
- The Terminal: [docs/TERMINAL.md](docs/TERMINAL.md) and [docs/TERMINAL_LAUNCH.md](docs/TERMINAL_LAUNCH.md). The app,
  the packages and the update feed: [packaging/README.md](tools/packaging/README.md) and [packaging/UPDATES.md](tools/packaging/UPDATES.md).
- AMD (HIP) build and validation: [docs/AMD_HIP.md](docs/AMD_HIP.md); multi-GPU: [docs/MULTI_GPU.md](docs/MULTI_GPU.md);
  Mac (Metal): [docs/MAC.md](docs/MAC.md).
- Tests: the Terminal and Control tests are in `terminal/tests/terminal` (`python -m pytest terminal/tests/terminal`). The app's UI
  tests run with `npm test` in `app/host`. Setup's own tests run without a GPU or downloads:
  `python tools/test_setup_<name>.py` (for example `terminal/tools/test_setup_amd.py`, `terminal/tools/test_setup_choices.py`).
- Keep the docs' style: plain words, short sentences, written for non-technical users first. Give measured numbers
  with what they were measured on. Make no claim without a measurement.

## Contributing a change or report

The rules are in **[CONTRIBUTING.md](.github/CONTRIBUTING.md)**. Read it before you open an issue or a pull request, and
follow it whether a person or an agent writes the text. The short version:

- Start every issue and pull request title with its area: `[Engine]`, `[GUI]`, `[TUI]`, `[Benchmark]`, `[Docs]` or
  `[Other]`, for example `[Engine] fix …`. A check on GitHub fails a pull request whose title has no area.
- Search the open and closed issues and pull requests first, and add to a thread that already covers your point.
- Agree the scope in an issue before a large feature, a refactor or a change to a default.
- One problem per pull request. Lead with the problem and the result, then the approach, then the validation
  (passed, failed and not run, kept apart) and the limits.
- A new feature is opt-in, and the default path stays byte-identical to the last release. Say how you checked.
- Build every backend a file touches (CUDA, HIP, SYCL) before asking for review.
- Change a default only where you measured it faster, and show the numbers with what they were measured on.
- Never submit measurements, logs or reproduction steps you did not produce. Mark a finding from reading code as
  unconfirmed static analysis.
- A report from hardware the maintainers do not have is welcome. Follow
  [docs/COMMUNITY_BENCHMARKS.md](docs/COMMUNITY_BENCHMARKS.md), compare against a same-day run of the build you are
  testing, and say what you did not test.
- Open test requests and the hardware that is wanted are listed in [docs/TEST_REQUESTS.md](docs/TEST_REQUESTS.md).

## This fork (o-leary/Strata)

Everything above is the upstream project's. This section is the owner's own, for this fork, and it wins where the
two meet. `origin` is the fork, `upstream` is the original repo (Niko1221/Strata). `working` is the branch the
owner works on; `gui/appimage-lan` holds the LAN feature, `gui/appimage-resume` holds a feature the owner does not
want built.

- **Nothing is built on GitHub.** The AppImage is built on this computer, by hand. GitHub Actions stays off for
  this fork: `gh api repos/o-leary/Strata/actions/permissions` reads `{"enabled": false}`, and if anything ever
  turns it back on, turn it off again. Rebasing onto `upstream/main` is no reason to enable Actions, and a push to
  `working` must not start a run.
- **Every build carries the LAN feature** (`--lan`, also `STRATA_LAN=1`, from `gui/appimage-lan`). A build without
  it is a mistake, and the resume branch is never merged in.
- "update my strata build", "update my fork" or the like means all of the following, in this order:

  1. `git fetch --all --prune`, then keep the old tip where it can be found again:
     `git branch pre-rebase-working-<YYYYMMDD> working`.
  2. Move `working` to `upstream/main` and put the one LAN commit back on top. Cherry-pick the LAN commit that is
     already on `working`, because its paths match the current layout; the commit on `gui/appimage-lan` sits on the
     old `apps/` layout. No `-x`: the message stays exactly as it is, with no extra lines added.
  3. A conflict in `docs/INSTALL.md` is usually the app version inside the `--lan` sentences: keep the owner's
     words, take the new version number.
  4. Check the shape: `git log --oneline upstream/main..working` shows one commit, the LAN one, and no resume work.
  5. Test the parts the feature touches: `cargo test --bins supervisor` in `app/desktop/src-tauri`,
     `node --test app/host/tests/share-on-start.test.mjs` in `app/host`, and
     `python3 -m unittest test_tester_readme` in `terminal/tests/desktop`.
  6. Build here, in `app/desktop`: `npm run build:ui`, then `npx tauri build --bundles appimage`.
  7. Repack the result, or it is wrong. linuxdeploy rewrites the load paths inside the staged payloads and breaks
     the bundled `rg`, so run `app/desktop/scripts/repack-appimage.sh <in> dist/Strata-<version>-linux-x86_64.AppImage`
     (the version is the one in `app/desktop/package.json`), then the same script with `--check` on that output.
     It must report `0 file(s) differ`, no damaged static PIE, and `bundled rg runs: ok`.
  8. Write `dist/Strata-<version>-linux-x86_64.AppImage.sha256` next to it. `dist/` is not in git; that is where
     the owner looks for a finished build.
  9. `git push --force-with-lease origin working`. One push. No pull request, no comment on any issue or pull
     request, no edit of the commit message.
  10. Prove the build carries the feature:
     `./dist/Strata-<version>-linux-x86_64.AppImage --appimage-extract-and-run --help` prints the `--lan` line.

- Say what was not run. The `Minimal serving runtime` failure on this fork predates this work, so a red light there
  is not evidence that the LAN change broke anything. The Control bundle under `app/desktop/src-tauri/resources` is
  staged by hand and can sit one component version behind the checkout; `app/desktop/scripts/stage-control.py`
  refreshes it, and a build should say which Control version it carried.
