# Home Automation + AI Agent Design List

Oct 16, 2026

One Claude-based agent becomes the single decision maker for the home and for background agent work, replacing Alexa entirely. Home Assistant on the Mac mini executes device actions; custom voice devices and a custom mobile app are the only front ends. It runs in one of two setups: Mac mini only (Phases 0 and 1, with cloud Claude and, for agents, a rented cloud VM), then Mac mini + Mac Studio (Phase 2 on, with local AI and agents on the Studio M5 Ultra). The mini-only setup is complete on its own if the Studio is never bought. Security comes first throughout: nothing is reachable from the internet, and no agent can use a password, send or spend without your approval on your phone.

## Glossary

| Term | Meaning here |
| --- | --- |
| Phase | A delivery stage from the roadmap: 0 home and voice, 1 agents without the Studio, 2 Mac Studio, 3 leftovers; Phases 0 and 1 use the Mac mini only setup, Phase 2 on the Mac mini + Mac Studio setup |
| Mac mini only setup | The 8GB Mac mini runs the house, voice and the agent service with cloud models; agent workspaces run on a rented cloud VM. Phases 0 and 1, and the permanent setup if no Studio is bought |
| Mac mini + Mac Studio setup | The mini keeps the house, vault and egress routes; the Studio runs voice, agents, workspaces and local models, and the cloud VM is retired. Phase 2 on |
| Home Assistant (HA) | Open-source home hub on the Mac mini; holds every device and executes commands |
| Agent | The single Claude-based decision maker that answers voice, app and chat requests |
| Orchestrator | The part of the agent that plans work and starts subagents |
| Subagent | A worker agent doing one job (research, an app task) in its own workspace |
| Workspace | A container or VM where a subagent runs, with shell, browser and optional desktop |
| Computer use | Claude controlling a desktop with screenshots, mouse clicks and keystrokes |
| Egress route | The internet exit a workspace uses: home IP, a NordVPN country, or the router VPN |
| Message bus | The channel subagents, the orchestrator and you use to talk; a policy decides who may talk to whom |
| Vault | Vaultwarden, the self-hosted password manager that replaces Chrome's saved passwords |
| Credential release | Sending one login from your phone to one agent job after you approve it |
| Passkey / security key | Sign-in that cannot be phished or read off a linked device the way an SMS code can |
| Pull request (PR) | A proposed code change you review and approve before it is deployed |
| MCP | Model Context Protocol; the standard way the agent calls tools such as Home Assistant |
| Voice server | Service on the Mac Studio (Phase 2) that turns speech to text, calls the agent, and speaks the reply |
| Show-style / Dot-style device | Custom voice devices: one with a touch screen, one with an LED dot-matrix display |
| Wake word | The spoken name that starts listening, detected on the device itself |
| STT / TTS | Speech-to-text and text-to-speech |
| Zigbee / Thread / Matter | Local smart-home radio and device standards handled by a USB stick on the Mac mini |
| VLAN | A separate network on the same routers, so groups of devices are kept apart |
| Tailscale | Private network that lets the app reach home without opening ports |
| WireGuard / NordLynx | Fast VPN protocol; NordLynx is NordVPN's version of it |
| Gluetun | Container that connects to a VPN (here NordVPN) and offers it as a proxy |

## Goals

- **One decision maker that feels like Jarvis.** Every request, from any room, the app or chat, goes to the same agent with the same memory; it follows the conversation as you move and speaks up when something needs you.
- **Reliable home control.** Lights, climate, media and appliances respond fast and keep working when the AI side is down; both Macs run unattended and recover by themselves after power cuts.
- **Custom front ends.** Echo Show-like and Echo Dot-with-clock-like devices built in-house, plus one polished mobile app for control, chat (text, voice, image, video) and agent work.
- **Agentic work.** Several subagents run at once, each using CLI and GUI tools from an assigned location or IP, talking to each other only where allowed, acting on your accounts (expenses, job applications, spreadsheets, even their own code) with you able to approve or step in from the app.
- **Secure by default.** Nothing is reachable from the internet, the servers hold none of your personal accounts, and a compromised server can neither spend money nor read your 2FA codes.
- **Hybrid AI.** Cloud Claude for hard reasoning, local models on the Mac Studio (Phase 2) for privacy, speed and offline use.

Non-goals: Alexa or Echo devices in any role; the Windows PC as compute (it stays a controlled device only); automated CAPTCHA solving; any open inbound port or internet-facing service; agents reading Chrome's saved passwords; unattended bulk actions that break a site's terms.

## Acceptance criteria

Targets are proposals to confirm. Each is grouped by the phase that must first meet it and is checked by a test or a dashboard in the last section.

**Phase 0, home and voice (Mac mini only setup):**

- Every listed device (Nest, Dreame, Hue, Govee, SwitchBot, Samsung TV and soundbar, B&O, Plex, Apple TV, TP-Link routers, Windows PC power) is controllable from voice and the app, every Phase 0 feature in the Alexa parity table works by voice, and every Alexa routine you use today has been migrated.
- Simple voice commands ("lights off") complete in under 1.5 s from end of speech; answers start speaking in under 3 s.
- The wake word triggers on at least 95% of attempts at 3 m in a normal room, with under 1 false wake per day per device.
- Lights, climate and automations keep working when the internet, or in the Mac mini + Mac Studio setup the Mac Studio, is down.
- After a power cut the Mac mini comes back and every service restarts with nobody touching it; neither Mac ever sleeps.
- An external port scan of the home IP finds no open ports; the app works away from home only over Tailscale.
- No server Mac is signed into your Apple ID, iCloud, Messages, Chrome or personal email, so no 2FA code can be read on a server.
- Monthly cloud AI spend has an alert threshold and a hard cap.

**Phase 1, agents (Mac mini only setup; still met after the move to the Studio):**

- At least 3 subagents run at once, each with shell, browser and desktop (Mac mini + Mac Studio setup: 5 or more, with 20+ possible).
- Each subagent's public IP matches its assigned route at job start, and its traffic stops if the route drops.
- A message between two agents not allowed by policy is blocked and logged.
- From the app you can watch any agent's screen live, pause it, approve or deny a step, take over mouse and keyboard, and kill it.
- Logins, purchases, sending messages, deletions and code changes always wait for an approval given on your phone with Face ID; nothing on a server can approve.
- Passwords never appear in model prompts, logs or screenshots, and a released login ends with the job.
- Agents cannot buy gift cards, transfer money or buy crypto; any purchase is capped and alerts you.
- The transit reimbursement job (see Example jobs) runs end to end with two approvals: the login and the email.
- An agent's code change that fails tests or health checks never stays deployed: it is blocked or rolled back automatically.
- Heads-up alerts (leave now, rain, door left open) arrive unprompted in the room you are in or on your phone, and never during quiet hours.
- Every agent action and message is logged and searchable for 30 days.

## Overall design

The system runs in one of two setups, and in both the house never depends on the AI side. **Mac mini only** (Phases 0 and 1): the mini runs Home Assistant, voice, the agent service, the vault and the network routes with cloud models, plus a rented cloud VM for agent workspaces from Phase 1. **Mac mini + Mac Studio** (Phase 2 on, the target design drawn below): the mini keeps the house, the vault and the routes, and the Studio takes the voice server, the agent runtime, workspaces and local models. Every front end talks only to the agent, and the agent controls devices only through Home Assistant's MCP server. Both setups run the same containers, so moving to the Studio is a redeploy.

&#91;embedded content: target design (Phase 2) · two hosts\]

**Phases:** each phase is usable on its own; tasks for each are in the [roadmap](https://claude.ai/code/artifact/d7976fe8-3e15-48d9-9e10-f86c6c4a57c3), and sections below say where a phase differs.

| Phase | What you get | Setup |
| --- | --- | --- |
| 0: Home and voice | Alexa retired with full everyday parity (timers, alarms, reminders, announcements, music, weather, news, stocks, lists); Home Assistant, voice devices, app and widgets; morning briefing; network, VPN and always-on Mac setup | Mac mini only, with cloud Claude |
| 1: Agents, no Studio | 3+ subagents with shell, browser and desktop; egress routes; password vault with phone approval; approvals and take over from the app; agent code changes by pull request; full memory, heads-up alerts and delegation | Mac mini only, with cloud Claude and one rented cloud VM for workspaces |
| 2: Mac Studio | Local models and voice server with interruptible replies; 5+ subagents (20+ possible) and up to 2 macOS VMs; screens that anticipate and routine suggestions; cloud VM retired; the Macs watch each other | Mac mini + Mac Studio |
| 3: Leftovers | Vision, per-person voices, per-room presence, 4G/5G backup line, hot spare, Valetudo, app extras | Mac mini + Mac Studio |

**What runs where in each setup:**

| Part | Mac mini only (Phases 0 and 1) | Mac mini + Mac Studio (Phase 2 on) |
| --- | --- | --- |
| Home Assistant OS VM, Zigbee/Thread stick, Bluetooth proxies | Mac mini (8GB), always on, on a UPS | Mac mini |
| Voice pipeline | Assist on the mini: cloud speech-to-text for open questions, local Speech-to-Phrase for fixed commands, Piper | Voice server on the Studio (local Whisper or Parakeet, LiveKit, interruptible replies); Assist on the mini as fallback |
| Agent service: orchestrator, memory, harness, app backend | Mac mini, calling cloud models | Mac Studio |
| AI models | Cloud only; offline, only Home Assistant's local intents answer | Cloud plus a local 100B to 200B model; private and offline requests use it |
| Agent workspaces | None in Phase 0; in Phase 1 a rented cloud VM (16 to 32GB): 3 Linux desktops, all traffic routed through home | Studio: 5+ Linux desktops (20+ possible) and up to 2 macOS VMs; cloud VM retired |
| Message bus (NATS) | Mac mini (Phase 1) | Mac Studio |
| Traces (Langfuse) | Cloud VM (Phase 1) | Mac Studio |
| Vault, credential broker, egress routes | Mac mini (Phase 1) | Mac mini |
| Tailscale subnet router, backups | Mac mini | Mac mini |
| Monitoring (Uptime Kuma, host health) | Mac mini watches everything | Mini and Studio watch each other |
| Plex server | Unchanged; used through its API | Same |
| Windows PC (RTX 5090) | Not compute; controlled device only (Wake on LAN, power) | Same |

The mini-only setup is also the long-term fallback: without a Studio, the cloud VM stays, workspaces stay Linux-only, and private or offline requests have no local model.

**Key decisions:**

| Decision | Chosen | Rejected and why |
| --- | --- | --- |
| Voice front end | Custom Show-style and Dot-style devices | Alexa/Echo: no raw audio for third parties, unreliable, Alexa stays the decision maker |
| Decision maker | One Claude-based agent | Split brains (Alexa + HA + agents) give inconsistent answers |
| Home hub | Home Assistant OS VM on the Mac mini | Homey, Hubitat, SmartThings: fewer integrations, no agent hooks |
| App | Custom app forked from HA Companion | Off-the-shelf apps: too much friction; WhatsApp: business-account rules |
| AI | Cloud Claude plus local models on the Studio | Local only: weaker at multi-step tool use; cloud only: no offline or private path |
| Agent isolation | One container or VM per subagent | Agents directly on the host: one bad page can reach your files and home |
| Phase 1 workspaces | Rented cloud VM routed through home | Workspaces on the 8GB mini: not enough memory |
| Passwords | Vaultwarden on the mini; each login released from your phone | Agents reading Chrome's passwords: no safe API, all or nothing; vault key on a server: one break-in leaks every password |
| Remote access | Tailscale only, zero open ports | Port forwarding or the router's VPN server: an exposed machine is the most common way in |
| Agent self-changes | Pull request, tests, your approval, automatic rollback | Agents editing live config: one bad change breaks the house |

## Detailed design

### Home control

Home Assistant OS runs in a UTM VM on the Mac mini (Phase 0), set to auto-start after power loss, with a Zigbee/Thread USB coordinator (SMLIGHT SLZB or Sonoff, about CA$35 to 85) and one or two ESP32 Bluetooth proxies for SwitchBot. Nest and Dreame stay cloud-dependent; everything else is local.

| Device | Integration | Local or cloud | Notes |
| --- | --- | --- | --- |
| Nest thermostat | Google Nest (SDM API) | Cloud push | One-time US$5 (about CA$7) Device Access fee and a Google Cloud project ([HA docs](https://www.home-assistant.io/integrations/nest)) |
| Dreame vacuum | Dreame Vacuum (HACS) or Valetudo firmware | Cloud, or local with Valetudo | Valetudo needs rooting; check model support ([guide](https://joinhomeshift.com/home-assistant-vacuum/)) |
| Philips Hue | Hue (official) | Local | Via the Hue Bridge |
| Govee lights | Govee lights local + HACS cloud for older models | Mostly local | Enable LAN control per device |
| SwitchBot buttons and IR | SwitchBot Bluetooth, Cloud or Matter via Hub | Local | IR remotes learned on the SwitchBot Hub |
| Samsung TV | Samsung Smart TV + SmartThings | Local | Wake from off via SmartThings or Wake on LAN |
| Samsung soundbar | SmartThings or HACS soundbar integration | Cloud | Input and volume |
| B&O soundbar | Bang & Olufsen (Mozart platform) | Local | Recent Mozart-based models |
| Windows PC | Wake on LAN + HASS.Agent | Local | Power on, sleep, shutdown |
| TP-Link Wi-Fi 7 routers | TP-Link router or Deco (HACS) | Local | Presence detection, reboot |
| Plex | Plex Media Server | Local | Now playing, play to clients |
| Apple TV | Apple TV (pyatv) | Local | Power, apps, playback |

Integration names other than Nest and Dreame are from general knowledge; confirm per model during setup.

### Voice devices

The devices are thin clients: they detect the wake word locally, stream audio to the voice pipeline (Assist on the mini in Phases 0 and 1, the voice server on the Studio from Phase 2), and show what the agent sends back. They make no decisions.

|  | Show-style | Show-mini | Dot-style |
| --- | --- | --- | --- |
| Like | Echo Show 8 | Echo Spot | Echo Dot with Clock |
| Brain | Raspberry Pi 5 | Waveshare ESP32-P4 3.4" round board ([about CA$93](https://www.waveshare.com/esp32-p4-wifi6-touch-lcd-3.4c.htm?sku=31523), mics and speaker included) | ESP32-S3 |
| Microphones | ReSpeaker XVF3800 4-mic array, USB ([about CA$100](https://thepihut.com/products/respeaker-xmos-xvf3800-ai-powered-4-mic-array-for-clear-voice-even-in-noise)) | Onboard dual mics with echo cancellation | ReSpeaker XVF3800, I2S |
| Display | Pi Touch Display 2, 7" ([about CA$85](https://raspberrypi.com/products/touch-display-2)) | Built-in 800×800 round | 32×8 or 64×32 LED matrix behind fabric |
| Speaker | 2 drivers + class-D amp | Included 2W | 1 driver + I2S amp |
| Software | Linux kiosk with the app UI + voice client | ESPHome or ESP-IDF with LVGL | Forked Voice PE ESPHome firmware |
| Parts (approx.) | CA$330 to 430 | CA$100 to 130 | CA$130 to 185 |

**Pipeline, Mac mini only setup (Phases 0 and 1):** devices are Home Assistant Assist satellites on the mini, with cloud speech-to-text for open questions, local Speech-to-Phrase for fixed commands and Piper for speech.

**Pipeline, Mac mini + Mac Studio setup (Phase 2 on):** on-device wake word (microWakeWord or openWakeWord, custom name) → audio over WebRTC (LiveKit) to the voice server on the Studio → local STT (Whisper or Parakeet) → agent → streaming TTS (Piper locally, or a cloud voice) → screen cards over WebSocket. Assist on the mini stays as the fallback. In both, simple device commands are matched by Home Assistant's intent engine first, so they skip the LLM.

**Enclosures:** 3D-printed with fabric fronts; speaker quality is the hardest part to match an Echo.

### Mobile app and chat

One custom app (iOS and Android) is the main interface, forked from the Home Assistant Companion apps so login, push notifications and widgets come for free.

- Home tab (Phase 0): rooms, scenes and device tiles, live from Home Assistant's WebSocket API; widgets, Control Center controls and an Action button shortcut to talk to the assistant.
- Assistant tab (Phase 0): text, push-to-talk and hands-free voice with streaming replies; photo and video input to the same agent in Phase 3.
- Agents tab (Phase 1): task board, live screen per agent, pause, take over, kill, and Face ID approvals for logins, purchases, sending and code changes (see Multi-agent runtime).
- Stack: native Swift and Kotlin from the Companion forks; realtime voice via LiveKit; remote access only over Tailscale; Apple Developer Program US$99/yr (about CA$141).
- Fallback channel: a Telegram bot for chat and notifications while the app is being built; WhatsApp is out of scope because it needs a business account and a separate number.

### Everyday assistant (Alexa parity)

Everything you use Alexa for today has a home here. Home Assistant owns anything time- or device-based, so timers, alarms and routines keep working without the internet; the agent answers questions and calls web services.

| Feature | How it works | Handled by | Phase |
| --- | --- | --- | --- |
| Timers | Several named timers per room ("pasta timer, 10 minutes"); countdown on the screen or dot matrix; rings on that device | Assist timers on each voice device | 0 |
| Alarms | Per-room alarms with snooze and stop; a wake-up can fade lights in and start music | Home Assistant calendar or helpers + automations | 0 |
| Reminders | One-off or recurring, set by voice or app; spoken in the room you are in, plus a phone push | Home Assistant calendar and to-do + presence | 0 |
| Announcements and intercom | "Announce dinner is ready" to every room or one; talk back from the app like Drop In | TTS to chosen voice devices; two-way audio through LiveKit | 0 (two-way in 2) |
| Weather | Current conditions, forecast, rain and weather alerts; card on screens | Environment Canada or Open-Meteo integration | 0 |
| News briefing | Morning briefing from sources you choose, summarized and spoken; news radio or podcasts on request | Agent with RSS or a news API; Music Assistant for audio | 0 |
| Stocks | Watchlist quotes, "how's my portfolio", price alerts | Agent tool on a market data API; Home Assistant sensors for alerts | 0 |
| Music and radio | Spotify, Apple Music over AirPlay, Plex music, internet radio; multi-room groups; "play jazz in the kitchen" | Music Assistant add-on to voice devices, soundbars and Apple TV | 0 |
| Shopping and to-do lists | Add by voice, see in the app | Home Assistant to-do lists, optionally synced to Todoist or Google Tasks | 0 |
| Calendar | "What's on tomorrow?", add events | Google or iCloud calendar through Home Assistant + agent | 0 |
| Questions | General knowledge, maths, conversions, translation, spelling, with web search | Agent (Claude) | 0 |
| Routines | "Good morning", "leaving", "bedtime", time- or sensor-triggered | Home Assistant scripts and automations; new ones by voice through Agent code changes | 0 |
| Groups and scenes | "Turn off everything downstairs", room-aware commands | Home Assistant areas and local intents | 0 |
| Find my phone | Rings your phone even on silent | Critical notification through the app | 0 |
| Sleep sounds and quiet hours | White noise with a sleep timer; no announcements and lower volume at night per room | Music Assistant + a schedule helper | 0 |
| Packages | "Where's my package?" | Agent reading shipping emails (Gmail, read-only) | 1 |
| Voice shopping | "Reorder paper towels" | Agent with phone approval and the spending limits in Security | 1 |
| Per-person voices | Your reminders and calendar versus a guest's | Speaker recognition | 3 |

Not copied: phone calls through the speakers and the Alexa skills store; the agent's tools replace skills.

### Proactive assistant (Jarvis layer)

On top of Alexa parity, the agent knows the context of the house and acts on it. Every proactive message goes through one rule: speak in the room you are in if it is urgent, otherwise push to the phone, and stay silent during quiet hours unless it is a safety alert.

| Capability | What it feels like | How | Phase |
| --- | --- | --- | --- |
| Memory | Knows people, preferences and past requests; "what do you know about me?" and "forget that" work | Shared memory store in the agent service, editable from the app | 0 basic, 1 full |
| Natural conversation | Follow-ups without the wake word for a few seconds; you can interrupt a long answer | Assist follow-up mode; barge-in on the voice server | 0, interrupt in 2 |
| Follows you | Answers in the room you are in; a conversation started in the kitchen continues on the phone | Presence from phones and devices; one conversation per person across front ends | 0 house-level, 3 per-room with mmWave |
| Briefings | Morning briefing on first motion in the kitchen: weather, calendar, transit delays, news, stocks, packages; short evening wrap-up | Scheduled agent job with presence trigger | 0 scheduled, 1 contextual |
| Heads-up alerts | "Leave in 10 minutes for your 3 pm", rain before you head out, door left open, laundry done, an unexpected card charge | Agent watches calendar, transit, weather and Home Assistant events and decides whether to speak, push or stay quiet | 1 |
| Screens that anticipate | Show devices put up the relevant card on their own: timer, doorbell, next meeting, now playing | Agent pushes cards to screens | 2 |
| Delegation | "Handle this" starts agent jobs and reports back wherever you are | Orchestrator and subagents | 1 |
| Suggestions | "You dim the lights at 10 every night; make it a routine?" | Pattern mining over Home Assistant history; accepted ideas become automations through Agent code changes | 2 |
| Vision | "What am I holding?" from the phone camera; "who came to the door?" from a doorbell or security camera | Claude image input; local vision model for cameras so footage stays home | 3 |
| Away and safety mode | Simulates presence when away, alerts on smoke, leaks or doors | Home Assistant automations; safety alerts ignore quiet hours | 0 |

### AI models

The agent routes each request to the cheapest model that can do it well, and falls back to local models when the cloud is unreachable.

The agent service doubles as a local harness (E1.12) with the features of Muse, Cowork, Dispatch, OpenClaw and ChatGPT: projects, chat sessions, background tasks you can steer or stop, skills, MCP connectors, a heartbeat, Telegram and a web console on the tailnet. Its model layer is provider-neutral: Claude stays the default decision maker, and OpenAI, Gemini, OpenRouter or a local model on the Studio can answer under the same approvals and CAD cap; private requests use local models only. Feature comparison: [Agent harness: feature comparison and plan](https://claude.ai/artifact/TP6vM4fUw5KPnHYGQRBGYy).

| Model | Where | Price per million tokens in / out | Used for | From |
| --- | --- | --- | --- | --- |
| Claude Opus 5.5 | Cloud | US$4 / 20 (about CA$5.70 / 28.50) | Planning, hard multi-step agent jobs | Phase 0 |
| Claude Sonnet 5.5 | Cloud | US$2 / 10 (about CA$2.85 / 14.25) | Default conversation brain, tool use, images | Phase 0 |
| Claude Haiku 5.5 | Cloud | US$0.10 / 0.50 (about CA$0.14 / 0.71) | Intent routing, summaries, notification text | Phase 0 |
| Open-weight LLM (100B to 200B class, 4-bit) | Mac Studio | Electricity only | Private requests, offline fallback | Phase 2 |
| Whisper or Parakeet, Piper | Mac Studio | Electricity only | Speech-to-text, text-to-speech | Phase 2 |

Cloud prices are Anthropic list prices, billed in US dollars; consumer subscriptions do not cover API use. 256GB of Studio memory is the suggested size (one large local model plus 20+ workspaces). In the Mac mini only setup only the cloud rows apply: the 8GB mini cannot hold a useful local model, so offline answers come only from Home Assistant's local intents.

### Multi-agent runtime

Each subagent runs in its own workspace with a shell, a browser and a desktop, exits through an assigned route, talks only to allowed peers, and can be watched and taken over from the app.

&#91;embedded content: multi-agent runtime · control, workspaces, egress\]

| Component | Job | Build on |
| --- | --- | --- |
| Orchestrator | Splits a request into subtasks; picks workspace, route, peers and model for each; collects results | Claude Agent SDK |
| Workspace manager | Creates, snapshots and destroys workspaces; attaches the egress route | Docker (Phase 1 cloud VM); OrbStack and Tart for macOS VMs (Phase 2); Parallels or UTM for an occasional Windows on Arm VM |
| Workspaces | Linux desktop (shell, Chromium, VNC), macOS VM, CLI-only | Fork of Anthropic's computer-use demo container or Kasm Workspaces images |
| Message bus + policy | Agent-to-agent and human-to-agent messages; allow-list per pair; full log | NATS (subjects per task) |
| Credential broker | Relays login requests to your phone and encrypted logins back to one workspace; never holds the vault key | Small service on the mini; Bitwarden SDK in the app |
| Control plane | Live screens, approvals, pause, take over, kill | App Agents tab; screens via noVNC, WebRTC in Phase 2 |
| Tools | Home Assistant MCP server, web search, calendar, Gmail, file storage, Plex API, notifications | MCP servers and APIs |

**Placement:** in the Mac mini only setup (Phase 1) the orchestrator, message bus, broker and routes run on the mini, and workspaces plus Langfuse run on the cloud VM; in the Mac mini + Mac Studio setup everything except the routes, vault and broker moves to the Studio with the same images.

**Tool order for a task:** APIs and MCP first, then browser automation (Playwright or browser-use) for websites, and computer use (screenshots, mouse, keyboard) only for apps with no other interface.

**Per-agent policy (example):**

```csv
agent,workspace,route,may_talk_to,may_request_logins,needs_approval_for
expenses,linux-desktop,home,orchestrator;writer,card-site;gmail,each login;sending
jobs,linux-desktop,home,orchestrator;writer,linkedin,each login;each application
research-de,linux-desktop,nord-de,orchestrator;writer,none,purchases
writer,cli-only,none,orchestrator;expenses;jobs;research-de,none,nothing
```

**Human intervention:** an approval request pauses the agent and sends a push notification with Approve and Deny. Approvals are signed by a key in your phone's Secure Enclave after Face ID and checked by the runtime, so nothing on a server can approve on your behalf. Take over gives you mouse and keyboard in the live view; the agent resumes with a note of what you did. Typed messages join the agent's conversation; Kill destroys the workspace and keeps its log.

**Capacity (approximate):** the Mac mini only setup runs 3 Linux desktop workspaces at once on a 16 to 32GB cloud VM, with no macOS VMs. In the Mac mini + Mac Studio setup a Linux desktop workspace uses 1 to 2GB, so a 256GB Studio runs 20+ alongside a large local model; macOS allows at most 2 macOS VMs at a time per Mac.

### Passwords and credentials

Your passwords move from Chrome into a self-hosted vault. Agents never hold the vault: they get one login at a time, released from your phone after you approve it (Phase 1).

**Moving off Chrome:**

1. Run Vaultwarden (open-source Bitwarden server, about 100MB of memory) on the mini, reachable only over Tailscale, and use the official Bitwarden apps and browser extension with it on your phone and laptop.
2. Export Chrome's passwords (Google Password Manager > Settings > Export passwords, a CSV file), import them, then delete the CSV and turn off Chrome's offer to save passwords so the vault is the only copy.
3. Rotate weak or reused passwords, starting with Google, Amazon, your bank and your Apple ID, and move those to passkeys or security keys.
4. Put logins agents may ever request in an "agents" collection; banking stays out until you opt a site in.

**Releasing a login to an agent:**

1. The subagent asks the credential broker for a login, giving the site, job, reason and route.
2. Your phone shows the job, site, exit IP and planned action; you approve with Face ID.
3. The app decrypts only that item on the phone and sends it encrypted to the requesting workspace's one-time key; the server relays only ciphertext, so a hacked server cannot get passwords.
4. The workspace browser fills the fields directly, outside the model's view; screenshots mask password fields and logs never hold the value.
5. One-time codes come from the vault under the same approval, or you type them by taking over; passkey-only sites always need your take over.
6. When the job ends the site is logged out and the workspace destroyed. Low-risk sites may keep an encrypted browser profile on the Studio; email, bank and Amazon never do.

1Password with a service account is simpler to set up but leaves a token on the server, which this threat model rules out.

### Agent code changes

Agents may write code for their own skills, tools, Home Assistant automations and integrations, but a change reaches the house only as a pull request you approve: skills and automations from Phase 1, the runtime itself from Phase 2.

1. All code and config live in git on a self-hosted Gitea or a private GitHub repository.
2. A coding subagent works on a branch in a CLI workspace, writes tests, and opens a pull request with a plain summary.
3. Checks run unit tests, linters, Home Assistant's config check, and a replay of recent requests against a staging copy.
4. The app shows the summary and diff, and you approve on your phone.
5. Deploy tags a release and runs health checks; any failure rolls back to the previous tag and alerts you.

The policy file, approval rules, credential broker, egress routes and this deploy pipeline are protected: an agent's pull request cannot change them, only you can.

### Example jobs

These jobs show the pieces working together; each uses a logged-in browser, released logins, files and email APIs, and stops for approval where shown.

| Job | How it runs | Route | You approve |
| --- | --- | --- | --- |
| transit reimbursement | Logs into the card site, downloads the month's transactions, picks out transit pass charges, builds an Excel sheet and summary, drafts the email in Gmail through the Gmail API | home | The login, then sending |
| LinkedIn job applications | Searches roles matching saved criteria, tailors the résumé and cover letter, fills each application up to Submit | home | The login, then each submission |
| Spreadsheets | Data work on .xlsx files with Python in a CLI workspace; Excel itself in a macOS VM (Mac mini + Mac Studio setup only) for macros, pivots and charts; files from OneDrive, Google Drive or iCloud through their APIs | none | Overwriting an original (a copy is saved first) |
| Fix or extend itself | Writes a new skill or automation as described in Agent code changes | none | The merge and deploy |
| Monthly paperwork | Downloads bills and receipts from utility, phone and card sites, files them, and flags unusual charges | home | Each new login |
| Bookings and price watches | Compares flights, hotels or products from several countries, then books a restaurant or appointment | nord-\<country> to compare, home to book | Any booking or purchase |

LinkedIn's user agreement forbids automated tools on the site, so applications stay few and reviewed, and an account restriction is a real risk you accept.

### Always-on Mac setup

Both Macs run as headless servers: they never sleep, restart by themselves after a power cut, start every service at boot, and can be reached only over Tailscale. The mini is set up in Phase 0 and is the only server in the Mac mini only setup; the Studio joins in Phase 2 for the Mac mini + Mac Studio setup.

**Before anything else:** a Mac that was previously a personal machine starts clean before it becomes a server. Erase it (Erase All Content and Settings), reinstall macOS, restore no apps or settings from old backups, and rotate every password ever used on it.

**Power and restart:**

1. `sudo pmset -a sleep 0 disksleep 0 displaysleep 10 powernap 0 autorestart 1 womp 1`, and in System Settings > Energy turn on "Prevent automatic sleeping", "Start up automatically after a power failure" and "Wake for network access"; Low Power Mode off.
2. `sudo systemsetup -setrestartfreeze on` so a frozen Mac restarts itself.
3. UPS over USB: in the UPS settings the Studio shuts down first (for example at 50%) and the mini last (at 20%); the mini, router and Zigbee stick share the UPS.
4. Services start at boot as launchd jobs: the Home Assistant VM through UTM's `utmctl` (App Nap off for UTM), containers with restart policies in OrbStack or Docker, and the standalone Tailscale system daemon so it is up before anyone logs in.
5. Automatic security responses on, automatic macOS upgrades off; install updates monthly in a maintenance window with `sudo fdesetup authrestart` so the Mac comes back by itself.

**Logging in by itself without being exposed:**

|  | Mac mini (house) | Mac Studio (agents) |
| --- | --- | --- |
| FileVault | Off, so it boots straight to its services after a power cut | On; planned restarts use authrestart, and an unplanned power loss waits for your unlock and alerts you |
| Login | Auto-login to a standard "hub" user that only runs services; the screen locks immediately after login | Same "hub" user after unlock |
| Why | The house must come back alone; nothing personal is stored on it and vault data is encrypted end to end | It holds browser sessions and agent data, so encryption at rest wins |

**Hardening on both Macs:**

- A separate admin account with a long unique passphrase from the vault, never used day to day; the hub user cannot install software.
- No Apple ID, iCloud, Messages, FaceTime, Handoff, iPhone Mirroring, Chrome sign-in or personal email; your phone is never paired to a server.
- macOS firewall on in stealth mode; remote login and screen sharing only over Tailscale, with Tailscale SSH keys and password logins turned off.
- Tailscale access rules: only your phone and laptop reach admin ports; workspaces reach only the proxy and message-bus ports.

### Network, VPN and TP-Link setup

The network is set up once in Phase 0: the TP-Link Wi-Fi 7 mesh splits the home into four networks, nothing is forwarded from the internet, and from Phase 1 agent traffic exits through named routes. Feature names vary by TP-Link model and firmware (Archer, Deco or Omada), so confirm each one on your model.

| Network (VLAN or SSID) | Members | Internet | Can reach |
| --- | --- | --- | --- |
| Main | Phones, laptops, both Macs, Apple TV, Plex, Windows PC | Full, through the home IP | Everything |
| IoT | Lights, Nest, Dreame, TVs, soundbars, hubs, voice devices | Only the cloud services a device needs | Only the Macs' service ports (no client isolation, so HA can discover devices) |
| Guest | Visitors | Full | Nothing local |
| Lab (Phase 1) | Workspaces that must exit through the router VPN | Only through the router's VPN client, with a kill switch | Only the mini's proxy and message-bus ports |

**Router configuration:**

1. Update firmware on every unit; the main unit is the router and the rest are mesh nodes, with wired backhaul where possible; Wi-Fi 7 MLO on the main network only.
2. Long unique admin password, remote management off, WPS off, UPnP off, no port forwarding; the TP-Link cloud account uses 2FA.
3. Networks as above, with 2.4 GHz allowed on IoT and band steering off for devices that drop.
4. DHCP reservations for both Macs, the Hue Bridge, SwitchBot Hub, TVs, soundbars, Apple TV, Plex and the Windows PC; Wake on LAN allowed to the PC.
5. mDNS reflection between Main and IoT so Home Assistant finds devices; if the model cannot, give the mini a second interface on IoT.
6. Local DNS on the router or the AdGuard Home add-on in Home Assistant.
7. Tailscale on the mini as subnet router and on every phone; the TP-Link VPN server stays off and is only a documented fallback.
8. Home Assistant TP-Link or Deco integration for presence, WAN status and reboot; the router configuration is backed up with the HA backups.
9. A 4G/5G backup line comes later (Phase 3).

**Egress routes (Phase 1 onward):** each route is a proxy on the mini; the workspace manager attaches one per workspace, and the job checks its public IP and country before starting. In the Mac mini only setup the cloud VM's workspaces send all traffic over Tailscale to these routes and are blocked from the internet otherwise, so a dropped route fails closed; in the Mac mini + Mac Studio setup the Studio's workspaces reach them over the LAN.

| Route | How | Use for |
| --- | --- | --- |
| home | Home IP through a plain proxy | Logged-in accounts, lowest bot-check friction |
| nord-\<country> | Gluetun container per NordVPN location | Location-specific results and prices |
| router-vpn | TP-Link VPN client (NordLynx) on the Lab network, with a kill switch | A dedicated workspace that must always exit from one country |

CAPTCHAs are handled by using a stable IP, the site's official API, or a push notification asking you to solve it; no automated solvers.

**NordVPN setup:**

1. Generate NordLynx (WireGuard) credentials in your Nord account for manual setup.
2. Run one Gluetun container per needed country on the mini (for example nord-de, nord-us-ny), with proxy ports open only to workspaces; cap the count to fit the mini's memory.
3. Gluetun's kill switch and NordVPN DNS stop traffic and lookups from leaking if a tunnel drops.
4. Test the router client on the Lab network only, and confirm Main and IoT still exit from the home IP.

NordVPN's own app is not used on the Macs, because it would move all traffic, including Home Assistant, onto the VPN.

### Security and threat model

Assume any server can be broken into. The model is a common attack chain: an internet-exposed machine with a weak password leads to email and shopping account takeovers, fraudulent purchases, and 2FA codes read through a phone synced to that machine. Every step of that chain is closed here:

| Attack step | Control |
| --- | --- |
| Reach a Mac from the internet | No port forwards, UPnP or remote router management; Tailscale only, with access rules; a monthly external port scan |
| Guess a weak password | Long unique passwords from the vault; SSH keys only; a separate admin account |
| Use the Mac's signed-in accounts | Servers hold no Apple ID, iCloud, Chrome profile or personal email |
| Read 2FA codes from a linked phone | No Messages or Continuity on servers; passkeys or security keys on Google, Amazon, the bank and Apple ID; SMS 2FA removed where a site allows |
| Spend money | Agents cannot buy gift cards, transfer money or buy crypto; purchases need phone approval and use a low-limit virtual card; every card charge alerts you |
| Get passwords or approvals from a server | The vault key and approval key live only on your phone |
| Trick an agent (prompt injection) | Pages, emails and other agents' messages are data; they cannot change policy, approvals or routes |

Agents also use a Home Assistant user limited to the entities they need (locks and garage doors need approval), API keys are released to services from the vault rather than stored in workspaces, and every job has a token budget under a monthly spend cap.

## Cost

All amounts are in Canadian dollars and approximate; Sources explains the conversions.

| Item | Phase | One-time | Monthly |
| --- | --- | --- | --- |
| Radios: Zigbee/Thread stick, 2 Bluetooth proxies | 0 | CA$60 to 115 |  |
| UPS for the Mac mini, router and Zigbee stick | 0 | CA$140 to 285 |  |
| Voice devices: e.g. 2 Show-style + 4 Dot-style | 0 | CA$1,180 to 1,600 |  |
| Nest Device Access | 0 | CA$7 |  |
| Apple Developer Program | 0 |  | CA$141 per year |
| Cloud Claude API | 0 |  | CA$14 to 71 at household use, more with heavy agent work (estimate) |
| Vaultwarden | 1 | Free |  |
| Rented cloud VM, 16 to 32GB | 1 only |  | CA$40 to 150 depending on provider (estimate) |
| Mac Studio M5 Ultra | 2 | From CA$7,799 (M5 Max from CA$3,499); 256GB memory costs more, so confirm Apple Canada's configured price | Electricity |

NordVPN, the TP-Link routers and the Mac mini are already owned. Totals per setup, from the rows above (monthly includes the developer fee spread over 12 months):

| Setup | One-time | Monthly |
| --- | --- | --- |
| Mac mini only | CA$1,390 to 2,010 | CA$26 to 83 in Phase 0; CA$66 to 233 once the cloud VM starts in Phase 1 |
| Mac mini + Mac Studio | From about CA$9,190 (adds the Studio) | CA$26 to 83 plus electricity; the cloud VM is retired |

## Monitoring, observability and high availability

The house must keep working when the internet or, from Phase 2, the Mac Studio is down, and every other failure must alert you and be recoverable from backups. In the Mac mini only setup the mini watches everything; the Mac mini + Mac Studio setup adds a watcher on the Studio so the two Macs watch each other.

### Monitoring and alerts

| What | Tool | Alert when |
| --- | --- | --- |
| Host health of the Macs (CPU, memory, disk, temperature) | Beszel or Prometheus node exporter + Grafana | Disk over 85%, Mac mini memory over 90%, an unexpected reboot or sleep |
| Power | UPS status into Home Assistant | On battery, battery low |
| Service uptime: HA, voice, agent runtime, NATS, vault, broker, Tailscale | Uptime Kuma on the mini, plus one on the Studio from Phase 2 | A check fails twice in a row |
| Network: WAN, router, VPN tunnels | HA TP-Link integration, Gluetun health checks | WAN down, tunnel down, kill switch engaged |
| Devices: offline, batteries, Zigbee link quality | Home Assistant | Unavailable over 10 min, battery under 20% |
| Egress routes | Scheduled public-IP and country check per route | IP or country differs from expected |
| Exposure | Monthly external port scan of the home IP | Any open port |
| Logins and spending | Broker audit log, card issuer alerts | A login request outside a job, repeated denials, any card charge |
| Agent code changes | Deploy health checks | A failed check or a rollback |
| Voice quality | Pipeline metrics: wake count, latency per stage | p95 above the acceptance targets |
| AI and cloud spend, errors | Anthropic usage data, cloud VM billing (Phase 1), agent runtime metrics | Daily spend over threshold, error spike |

Alerts arrive as app push notifications, with Telegram as the second channel.

### Observability

- Logs: every service ships logs to Loki (or plain files collected by Vector), kept 30 days; passwords and codes are never logged.
- Traces: the agent runtime emits OpenTelemetry traces, one per request, covering speech-to-text, model calls, tool calls and the Home Assistant action; Langfuse (open source) stores them, on the cloud VM in Phase 1 and the Studio from Phase 2.
- Agent audit log: every tool call, message, approval, login release and takeover, searchable from the app.
- Dashboards: Grafana boards for house, voice, agents and cost, also reachable from the app.

### High availability and recovery

| Failure | Effect | Mitigation |
| --- | --- | --- |
| Mac Studio down (Mac mini + Mac Studio setup) | Voice server and agents stop | HA automations, app tiles and wall switches keep working; voice devices fall back to Assist on the mini |
| Cloud workspace VM down (Mac mini only setup) | Agents stop | The house is unaffected; jobs queue; rebuild the VM from its image |
| Internet down | Cloud Claude, Nest, Dreame and VPN routes unavailable | Local devices keep working, plus the local model from Phase 2; agent jobs queue and resume |
| Anthropic API outage | Cloud brain unavailable | Simple commands still run through HA's local intents; from Phase 2 the local model takes over; non-urgent jobs queue |
| Mac mini down | House control, vault and routes stop | UPS, automatic restart and login, services at boot; watchdog alerts; HA backup restorable onto the Studio as a temporary host |
| Power cut | Everything off | UPS on the mini, router and Zigbee stick; the Studio shuts down first; the mini restarts by itself, and the Studio waits for your FileVault unlock and alerts you |
| Server broken into | Attacker on a Mac | It holds no personal accounts, vault key or approval key; erase and rebuild from git and backups, and rotate any released logins |
| Agent stuck or misbehaving | Wasted spend, wrong actions | Job timeouts and token budgets, kill from the app, workspace snapshot rollback |
| Bad agent code change | Broken skill or automation | Automatic rollback to the previous release |
| Data loss | Config or history gone | Nightly HA, vault and agent-database backups to a NAS or iCloud, configs in git, a monthly restore test |

Remaining single points of failure are the Mac mini (no hot spare), the router and the internet line; a 4G/5G backup line and a hot spare are Phase 3.

## Sources

Prices are in Canadian dollars. The Mac Studio uses Apple Canada list prices; other prices are converted from US dollars or pounds at October 2, 2026 rates (1 USD = 1.4247 CAD, 1 GBP = 1.8867 CAD) and are approximate. Integration and project names without a link are from general knowledge and should be checked before buying or forking. Claude API prices are Anthropic list prices as of October 2026.

- [Home Assistant: Google Nest integration](https://www.home-assistant.io/integrations/nest)
- [Best robot vacuums for Home Assistant (HomeShift)](https://joinhomeshift.com/home-assistant-vacuum/)
- [Waveshare ESP32-P4 3.4" round touch display](https://www.waveshare.com/esp32-p4-wifi6-touch-lcd-3.4c.htm?sku=31523)
- [ReSpeaker XVF3800 4-mic array (The Pi Hut)](https://thepihut.com/products/respeaker-xmos-xvf3800-ai-powered-4-mic-array-for-clear-voice-even-in-noise)
- [Raspberry Pi Touch Display 2](https://raspberrypi.com/products/touch-display-2)
- [Mac Studio M5 Max and M5 Ultra Canadian prices (iPhone in Canada)](https://www.iphoneincanada.ca/2026/08/25/apple-unveils-new-mac-studio-with-m5-max-with-huge-price-hike-in-canada/)
- [USD to CAD rate, October 2, 2026](https://www.exchange-rates.org/exchange-rate-history/cad-usd-2026-10-02) and [GBP to CAD rate](https://www.exchange-rates.org/exchange-rate-history/cad-gbp-2026-10-02) (exchange-rates.org)
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden)
- [Export passwords from Chrome (Google help)](https://support.google.com/chrome/answer/95606)
- [LinkedIn User Agreement](https://www.linkedin.com/legal/user-agreement)
