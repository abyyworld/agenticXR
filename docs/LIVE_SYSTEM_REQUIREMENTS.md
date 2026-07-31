# AgenticXR live-system requirements

Last reviewed: 2026-07-25

This is the living checklist for running the complete AgenticXR system with Unity,
Meta Quest, Ubiq, speech recognition, Claude orchestration, continuous activity
monitoring, bounded goals, and optional future-goal preparation.

Update this file whenever live testing reveals another credential, dependency,
hardware requirement, configuration step, or acceptance check.

## 1. Information required from the user

Record these before live integration:

- [ ] Quest model:
- [ ] Quest OS/version:
- [ ] Server PC operating system:
- [ ] Server PC LAN IPv4 address:
- [ ] Quest and server PC are on the same non-isolated LAN:
- [ ] Unity version is `6000.3.9f1`:
- [ ] Unity Android Build Support is installed:
- [ ] Android SDK, NDK, and OpenJDK are installed through Unity Hub:
- [ ] Faster Whisper STT endpoint:
- [ ] STT health endpoint, if available:
- [ ] First test target: Unity Editor or deployed Quest:
- [ ] Continuous proactive assistance enabled: yes/no
- [ ] Idle future-goal preparation enabled: yes/no
- [ ] Initial experience mode: productivity/training/entertainment/exploration
- [ ] Approved test scenario and expected behavior:

Do not write API keys, participant names, emails, raw transcripts, or other secrets
into this file.

## 2. Required credentials and services

### Anthropic

- [ ] Anthropic API account exists.
- [ ] Billing or API credits are enabled.
- [ ] `ANTHROPIC_API_KEY` is set only in the server PowerShell environment.

```powershell
$env:ANTHROPIC_API_KEY="your-real-key"
```

Never commit the key or place it in Unity assets, JSON configuration, documentation,
Git history, screenshots, or chat.

### Speech-to-text

- [ ] A Faster Whisper-compatible service is reachable from the Node server PC.
- [ ] It accepts `POST /stt/transcribe`.
- [ ] It accepts multipart field `file` containing 16 kHz mono 16-bit WAV.
- [ ] It accepts `language=en` and `beam_size=1`.
- [ ] It returns recognized text as the response body.
- [ ] `STT_HTTP_URL` points to the complete transcription endpoint.

```powershell
$env:STT_HTTP_URL="http://STT-HOST:50101/stt/transcribe"
```

The repository contains the STT client, not the Faster Whisper HTTP server. If no
endpoint exists, deployment requires a target machine, operating system, CPU/GPU
details, network address, and permission to install/run the service.

### Credentials not required

The AgenticXR Claude path does not require:

- OpenAI API key;
- Azure Speech key;
- separate MCP key;
- Claude Code login;
- legacy Python virtual environment.

## 3. Server software

On a macOS server host, read section 13 first. Two platform blockers there stop
`npm install` before it finishes.

- [ ] Node.js and npm are installed.
- [ ] Repository dependencies are installed from `Server`.
- [ ] Ubiq, Claude Agent SDK, MCP SDK, and Zod pass the setup doctor.
- [ ] TCP port `8009` is available.
- [ ] The runtime and scene bridge use the same Ubiq `roomGuid` and port.

```powershell
cd "D:\Research_Activities\agenticXR\Real-Time-Immersive-Speech-Programming\Server"
npm install

$env:AGENTICXR_MODE="claude"
$env:ANTHROPIC_API_KEY="your-real-key"
$env:STT_HTTP_URL="http://STT-HOST:50101/stt/transcribe"

npm run doctor
npm test
npm run test:integration
```

Expected:

- [ ] `npm run doctor` ends with `Setup looks complete.`
- [ ] `npm test` passes.
- [ ] `npm run test:integration` ends with `[mock_integration] PASS`.

## 4. Windows and LAN configuration

For a macOS host, see section 13.4 instead of the PowerShell firewall rule below.

- [ ] Server PC and Quest are on the same LAN.
- [ ] Guest/client isolation is disabled.
- [ ] A VPN is not separating the devices.
- [ ] Windows Firewall allows inbound TCP `8009`.
- [ ] Unity uses the server PC LAN address, not `localhost`.

Administrator PowerShell:

```powershell
New-NetFirewallRule `
  -DisplayName "AgenticXR Ubiq TCP 8009" `
  -Direction Inbound `
  -Protocol TCP `
  -LocalPort 8009 `
  -Action Allow
```

## 5. Unity requirements

- [ ] Open the `Unity` directory with Unity `6000.3.9f1`.
- [ ] Open `Assets/Demos/DynamicCompiler/DynamicCompiler.unity`.
- [ ] Configure the Ubiq Room Client with the server PC LAN IPv4 address.
- [ ] Configure TCP port `8009`.
- [ ] Confirm the room GUID matches the server configuration.
- [ ] Authorable objects have the `game` tag.
- [ ] Android is the active build platform.
- [ ] Oculus/Quest XR provider is enabled for Android.
- [ ] Android microphone permission is present.
- [ ] The project compiles without C# errors.

The automatic AgenticXR bootstrap should install the cache exchange manager, scene
publisher, runtime compiler integration, continuous sensor publication, visible
status, and world-space Approve/Reject/Undo panel.

## 6. Quest requirements

- [ ] Meta Quest Developer Mode is enabled.
- [ ] USB debugging is approved.
- [ ] The application is built and installed.
- [ ] `RECORD_AUDIO` permission is approved in the headset.
- [ ] Quest can reach the server PC on TCP `8009`.
- [ ] The Ubiq room connection succeeds.
- [ ] Push-to-talk sends audio.
- [ ] Agent status is visible.
- [ ] Approve, Reject, and Undo can be selected with the XR pointer/controller.

The world-space button/ray interaction is not yet live-validated. If the panel
renders but buttons cannot be selected, inspect the tracked-device UI input module,
event camera, raycaster, and Quest controller bindings.

## 7. Continuous and predictive features

The long-lived activity observer starts by default. Proactive model calls and idle
prediction remain opt-in because they consume API credits.

### Continuous activity assistance

```powershell
$env:AGENTICXR_MONITOR_ENABLED="true"
$env:AGENTICXR_CONTINUOUS_ASSIST_ENABLED="true"
$env:AGENTICXR_ACTIVITY_THRESHOLD="1.1"
$env:AGENTICXR_ACTIVITY_WINDOW_MS="5000"
$env:AGENTICXR_ACTIVITY_COOLDOWN_MS="30000"
$env:AGENTICXR_CONTINUOUS_ASSIST_TIMEOUT_MS="120000"
```

- [ ] Continuous monitor joins Ubiq.
- [ ] Gaze/proximity/locomotion observations appear in the activity stream.
- [ ] Threshold crossing surfaces visible status.
- [ ] Context assistance remains L2.
- [ ] High-risk or unverifiable assistance requires confirmation/dialogue.
- [ ] Push-to-talk preempts continuous assistance.

### Idle future-goal preparation

```powershell
$env:AGENTICXR_IDLE_PREDICTION_ENABLED="true"
$env:AGENTICXR_IDLE_PREDICTION_THRESHOLD_MS="60000"
$env:AGENTICXR_IDLE_PREDICTION_COOLDOWN_MS="300000"
```

- [ ] Idle preparation never proposes or commits directly.
- [ ] Prepared candidates are tied to the current scene snapshot.
- [ ] Stale candidates are rejected.
- [ ] Reused candidates pass the complete normal pipeline.

Both features are suppressed during study trials unless the approved protocol sets:

```powershell
$env:AGENTICXR_STUDY_ALLOW_CONTINUOUS_ASSIST="true"
$env:AGENTICXR_STUDY_ALLOW_SPECULATION="true"
```

## 8. Complete startup

From `Server`, in the PowerShell window containing the environment variables:

```powershell
npm run start:agenticxr
```

This should start:

- Ubiq room server on TCP `8009`;
- Quest audio receiver;
- Faster Whisper STT client;
- Claude Agent SDK orchestration;
- MCP Unity scene bridge for agent turns;
- long-lived continuous activity monitor;
- Shared XR Memory and temporal logging;
- bounded goal and verification routing.

Start the server before launching the Unity Editor client or Quest application.

## 9. Live acceptance sequence

Run in this order:

- [ ] Server doctor, deterministic tests, and mock integration pass.
- [ ] Unity Editor joins the Ubiq room.
- [ ] Scene/sensor observations reach `get_activity_stream`.
- [ ] A typed/manual Claude request succeeds with mock or Editor Unity.
- [ ] Quest joins the same room.
- [ ] Quest push-to-talk reaches STT.
- [ ] Transcript starts a Claude orchestration turn.
- [ ] Scene grounding returns the selected stable object.
- [ ] Three candidates are generated and validated.
- [ ] Verification Space simulation succeeds.
- [ ] Low-risk automatic policy is enforced correctly.
- [ ] Confirmation-required proposal appears in the headset.
- [ ] Approve commits the generated behavior.
- [ ] Reject prevents application.
- [ ] Undo rolls back the latest behavior.
- [ ] Explicit speech preempts a continuous assistance turn.
- [ ] A bounded goal terminates or escalates rather than spinning.
- [ ] Application restart restores supported checkpoints and reports orphaned targets.

## 10. Evidence to provide when something fails

Provide only non-secret diagnostics:

- exact failed checklist step;
- server terminal output around the failure;
- `npm run doctor` output;
- Unity Console error text;
- Unity Player log or relevant `adb logcat` excerpt;
- Quest screenshot/video of visible status or consent UI;
- server PC IP and Quest network subnet, with sensitive addresses redacted if needed;
- whether the same step works in Unity Editor;
- whether continuous assistance and idle prediction were enabled.

Never include API keys, raw participant audio, identifying participant data, or
unredacted credentials.

## 11. Current verification boundary

Confirmed as of the review date:

- [x] Node deterministic tests pass.
- [x] Local Ubiq + mock Unity integration passes.
- [x] Long-lived monitor joins mock Ubiq and observes activity.
- [x] Mock threshold crossing and monitor-only suppression pass.
- [x] Unity `6000.3.9f1` batch compilation succeeds.

Still requires user/device validation:

- [ ] Real Anthropic request.
- [ ] Real Faster Whisper request.
- [ ] Unity Editor live orchestration.
- [ ] Quest LAN connection.
- [ ] Quest microphone and push-to-talk.
- [ ] Quest world-space consent controls.
- [ ] Real continuous-assistance behavior and preemption.
- [ ] Real generated-code compilation, attachment, watchdog, and undo.
- [ ] Performance, API cost, and latency measurement.
- [ ] Entertainment/productivity/training application-level evaluation.

## 12. Related documentation

- `docs/SETUP_INSTRUCTIONS.md`
- `docs/continuous-human-centered-runtime.md`
- `docs/goal-loops-and-speculative-futures.md`
- `docs/study-logging-schema.md`
- `docs/progress-log.md`

## 13. macOS server host

Sections 3 and 4 assume a Windows server PC and PowerShell. This section covers an
Apple Silicon Mac acting as the server host. Verified on macOS `26.5.1`, `arm64`,
Node `18.20.8`, npm `10.8.2`.

### 13.1 The repository path must not contain a colon

- [ ] No directory in the absolute path to the repository contains `:`.

This is not cosmetic. npm prepends `node_modules/.bin` to `PATH`, and `PATH` is
colon-delimited, so a colon anywhere in the path splits that entry into two invalid
directories. Every package with a build script then fails to find its own tooling.
The symptom is misleading:

```
npm error code 127
npm error command sh -c node-gyp-build
npm error sh: node-gyp-build: command not found
```

Reproduced with two otherwise identical empty projects: installing `bufferutil` under
`.../hci:ai proj/` fails with the error above, while `.../hci-ai proj/` succeeds.
Rename the directory. There is no workaround that keeps the colon.

### 13.2 wrtc does not build on Apple Silicon

- [ ] `wrtc` is absent from `Server/vendor/ubiq/package.json` dependencies.

`wrtc@0.4.7` publishes no `darwin-arm64` prebuilt binary, so its install script 404s
on `https://node-webrtc.s3.amazonaws.com/wrtc/v0.4.7/Release/darwin-arm64.tar.gz` and
aborts the entire install before the Claude Agent SDK is fetched. The package is
referenced only by `vendor/ubiq/samples/rtcpeerconnection/app.js`, a Ubiq sample this
project never runs. It is removed from the vendored dependency list. Restore it only
if that sample is ever needed, and not on an arm64 host.

### 13.3 Environment and commands

Node 18 is sufficient. The Claude Agent SDK declares `engines.node >= 18.0.0`.

```bash
cd "/path/to/agenticXR/Server"
npm install

export AGENTICXR_MODE="claude"
export ANTHROPIC_API_KEY="your-real-key"
export STT_HTTP_URL="http://STT-HOST:50101/stt/transcribe"

npm run doctor
npm test
npm run test:integration
```

Keeping the key out of shell history is easier with a gitignored file:

```bash
set -a; . ./.env.local; set +a
```

### 13.4 Firewall and LAN

macOS has no `New-NetFirewallRule`. The application firewall is off by default, in
which case inbound TCP `8009` needs no rule. Check and configure with:

```bash
/usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
/usr/libexec/ApplicationFirewall/socketfilterfw --add "$(which node)"
/usr/libexec/ApplicationFirewall/socketfilterfw --unblockapp "$(which node)"
```

Find the LAN address Unity and the Quest must target with `ipconfig getifaddr en0`,
falling back to `en1` on wired adapters. Do not use `localhost`.

- [ ] If the Mac is on an institutional or campus network, confirm client isolation is
      disabled. Large managed subnets frequently block peer-to-peer traffic between
      wireless clients, which prevents the Quest from reaching the Mac even though
      both devices report a valid address on the same network.

### 13.5 Verified state on macOS

Confirmed after 13.1 and 13.2 are satisfied:

- [x] `npm install` completes.
- [x] `npm test` passes, 258 assertions.
- [x] `npm run test:integration` ends with `[mock_integration] PASS`, using a real
      local Ubiq room server and real MCP sessions against a mock Unity peer.
- [x] `AGENTICXR_MODE=claude npm run doctor` reports only `ANTHROPIC_API_KEY` and
      `STT_HTTP_URL` missing, confirming the legacy Python environment and OpenAI key
      are not required on the Claude path.

Still unverified from a macOS host, and blocked only by credentials rather than by
platform: every item in section 11's "Still requires user/device validation" list.
