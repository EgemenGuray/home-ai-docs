# Home Automation AI Roadmap: Epics, Stories, Tasks

Oct 17, 2026

## How to read this roadmap

Phase 0 replaces every Alexa device and routine with Home Assistant, a cloud Claude assistant, custom voice devices and the phone app. Phase 1 adds agentic work without the Mac Studio, Phase 2 moves AI and agents onto the Mac Studio, and Phase 3 holds the leftovers. The architecture and device choices come from the [Home Automation + AI Agent Design List](https://claude.ai/code/artifact/a2c0648c-0d0f-435d-9064-06828ca289de).

| Phase | Outcome | Runs on |
| --- | --- | --- |
| Phase 0: Home and voice | Secure network and always-on Mac mini; Alexa retired with every everyday feature and routine replaced; voice, app and widgets control the house; Claude answers and searches | Mac mini + cloud Claude |
| Phase 1: Agents, no Studio | Subagents with shell, browser and desktop on pinned routes; logins released from your phone; code changes by pull request; proactive briefings and alerts | Mac mini + cloud Claude + a rented cloud VM for workspaces |
| Phase 2: Mac Studio | Local models and voice, 20+ workspaces, macOS VMs, agents spin off their own subagents, screens and suggestions that anticipate | Mac Studio + Mac mini |
| Phase 3: Leftovers | Polish, extra resilience, vision and stretch features | Both |

**IDs.** Epics are `E<phase>.<n>`, stories are `S<phase>.<epic>.<n>`, and tasks are the checkboxes under each story, so you can tick them off here or copy them into a tracker.

**Assumptions.** Each phase starts only when the previous phase's exit criteria are met, except Phase 2, which starts the day the Mac Studio arrives. Cloud Claude stays the single decision maker until local models are proven in Phase 2. The Windows RTX 5090 PC is only a device (Wake on LAN), never compute. Prices are approximate and in CAD.

## Phase 0: Home automation and Alexa replacement

Phase 0 ends with every Echo unplugged, every Alexa routine running in Home Assistant, and Claude answering questions by voice, app and widget. Everything runs on the TP-Link network, the Mac mini and cloud Claude; because there is no Mac Studio yet, voice uses Home Assistant's Assist pipeline on the mini with cloud speech-to-text, and moves to the Studio in Phase 2.

**Exit criteria:** Main, IoT and Guest networks live; an external port scan of the home IP shows zero open ports; passkeys or two-factor on every account and no personal accounts on the server Macs; all devices controllable by voice and app; every everyday Alexa feature you used (timers, alarms, reminders, announcements, weather, news, stocks, music, lists, calendar) works by voice and app; every inventoried Alexa routine migrated and run for 2 weeks; simple commands under 1.5 s; the house keeps working with the internet down; app works away from home over Tailscale; all Echos removed.

### E0.1 TP-Link network, home IP and remote access

The TP-Link Wi-Fi 7 routers split the home into Main, IoT and Guest networks (a Lab network for agents follows in Phase 1), every server has a fixed address, and remote access goes only through Tailscale. NordVPN and the router's VPN client are agent routes, so they are set up in E1.3.

**S0.1.1 Routers and mesh are up to date**

- [ ] Confirm the TP-Link model line (Archer, Deco or Omada) and which features it supports: VLANs, mDNS forwarding, VPN client with policy routing
- [ ] Update firmware on every unit; main unit as router, the rest as mesh nodes with wired backhaul where possible
- [ ] Back up the router configuration

**S0.1.2 Home networks**

- [ ] Main (phones, laptops, both Macs, Apple TV, Plex, Windows PC), IoT (lights, Nest, Dreame, TVs, soundbars, hubs, voice devices) and Guest; the Lab network for agents comes in S1.3.3
- [ ] IoT gets internet only for devices that need the cloud (Nest, Dreame, Govee) and reaches only the Macs' service ports; no client isolation on IoT so HA can discover devices; Guest reaches nothing local
- [ ] mDNS reflection between Main and IoT; if the model can't, give the mini a second interface on IoT
- [ ] Wi-Fi 7 MLO on Main only; 2.4 GHz allowed on IoT and band steering off for devices that drop

**S0.1.3 Addresses, home IP and DNS**

- [ ] Confirm the home WAN IP with the ISP; it becomes the `home` route in E1.3
- [ ] DHCP reservations for the Mac mini, voice devices, Hue Bridge, SwitchBot Hub, TVs, soundbars, Apple TV, Plex and the Windows PC
- [ ] Local DNS names for the Mac mini and HA
- [ ] Allow Wake on LAN to the Windows PC (UPnP and port forwarding are covered in S0.1.5)

**S0.1.4 Remote access over Tailscale**

- [ ] Tailscale on the Mac mini as subnet router for the home networks, and on every phone
- [ ] Leave the TP-Link built-in VPN server off
- [ ] Test HA and the app from mobile data away from home

**S0.1.5 Nothing at home is reachable from the internet, and one break-in can't spread**

No machine is ever exposed to the internet, every account uses a long unique password with phishing-proof two-factor, and the server Macs carry none of your personal accounts, so a compromised Mac can't reach your Amazon, Gmail, bank or 2FA codes.

- [ ] Once the mini is up (E0.2), run Vaultwarden (self-hosted Bitwarden server) on it, reachable only over Tailscale, with the official Bitwarden apps and browser extension on your phone and laptop
- [ ] Move your Chrome passwords into it: export the CSV from Google Password Manager, import, delete the CSV, turn off Chrome's offer to save passwords
- [ ] Vault-generated passwords (20+ characters, unique) for the router admin, Mac accounts, Home Assistant, Tailscale, NordVPN, Apple, Google, Amazon, bank and Anthropic, starting with Google, Amazon, the bank and your Apple ID
- [ ] Passkeys or two hardware security keys (for example YubiKeys) for Google, Apple, Amazon and the vault; remove SMS as a 2FA fallback wherever the site allows; consider Google Advanced Protection
- [ ] Two-factor everywhere else it is offered: Tailscale login, NordVPN, Anthropic, TP-Link cloud account, the bank, and TOTP for every HA user
- [ ] On your iPhone, turn off Text Message Forwarding and iMessage on every Mac so 2FA texts never land on a computer
- [ ] Server Macs have no Apple ID, iCloud, Messages, FaceTime, Handoff, iPhone Mirroring, Chrome sign-in or personal email, and are never paired with your phone
- [ ] TP-Link: long unique admin password, remote (cloud or WAN) management off, WPS and UPnP off, 2FA on the TP-Link cloud account, WPA3 where devices allow
- [ ] Router port-forwarding table stays empty; Tailscale is the only way in, with access rules so only your phone and laptop reach admin ports
- [ ] macOS firewall on in stealth mode; Remote Login and Screen Sharing only over Tailscale, with Tailscale SSH and password logins off
- [ ] External port scan of the home IP from outside the house (phone on mobile data with nmap, or an online scanner): expect zero open ports; repeat after every router change and monthly
- [ ] Card and bank alerts for every transaction, and Amazon order notifications, pushed to your phone
- [ ] HA notification when an unknown device joins the network

**S0.1.6 Account hygiene sweep**

- [ ] From a clean device, change the password and 2FA of every account that was ever used on the server Macs, starting with email, shopping, Apple ID and the bank
- [ ] Sign out all other sessions and remove unknown devices in your main accounts
- [ ] Check email for unexpected forwarding rules, filters, app passwords and recovery email or phone changes
- [ ] Check shopping accounts for unexpected addresses, payment methods and gift card activity

### E0.2 House hub on the Mac mini

The Mac mini becomes a reliable, always-on house hub that restarts itself after power loss.

**S0.2.1 Clean start for the Mac mini (do this first)**

The Mac mini was previously a personal machine, so it is checked and reset before it becomes the house server; a clean reinstall stays the recommended path, because a scan can miss things that an erase can't.

- [ ] Unpair any phone, remove personal browser profiles and change the Mac password
- [ ] Run a security check: login items, launch agents and daemons, configuration profiles, remote access settings, unknown users and SSH keys
- [ ] Reinstall unless the check comes back clean: Erase All Content and Settings, or a DFU restore with Apple Configurator from another Mac, then a fresh macOS
- [ ] If reinstalled, set it up as new: no Time Machine restore and no Migration Assistant; copy back only plain documents you need, scanned first
- [ ] Set it up with no Apple ID (S0.1.5); passwords ever used on it are rotated in S0.1.6

**S0.2.2 Mac mini is set up as an always-on server**

- [ ] Never sleep and restart by itself: in System Settings, Energy, turn on "Prevent automatic sleeping", "Start up automatically after a power failure" and "Wake for network access", Low Power Mode off; then `sudo pmset -a sleep 0 disksleep 0 displaysleep 10 powernap 0 autorestart 1 womp 1`
- [ ] Restart after a freeze: `sudo systemsetup -setrestartfreeze on`
- [ ] Two accounts: a standard "hub" user that logs in automatically, only runs services and locks the screen right after login; a separate admin with a long vault passphrase, never used day to day
- [ ] FileVault off on the mini so the house comes back alone after a power cut; it stores nothing personal and vault data is end-to-end encrypted (the trade-off is in the risks table)
- [ ] Updates: automatic security responses on, automatic macOS upgrades off; update monthly after a fresh HA backup, using `sudo fdesetup authrestart`
- [ ] Services start at boot as launchd jobs: the HA VM through UTM's `utmctl` (App Nap off for UTM), containers with restart policies, and the standalone Tailscale system daemon so it is up before login
- [ ] Buy a UPS for the mini, router and Zigbee stick, connected to the mini by USB; the mini shuts down last, at about 20% battery, and comes back on its own when power returns
- [ ] Wired Ethernet on the Main network with its reserved address, Wi-Fi off
- [ ] Test it: unplug the mini and confirm HA, Tailscale and the VM are back within 5 minutes without touching it

**S0.2.3 Home Assistant runs on the Mac mini and survives reboots**

- [ ] Install UTM and create the Home Assistant OS VM (2 to 3 GB RAM, leave headroom for the Phase 1 proxies)
- [ ] Set the HA VM to start automatically when UTM launches

**S0.2.4 Local radios cover the house**

- [ ] Buy a Zigbee/Thread coordinator (SMLIGHT SLZB or Sonoff) and pass it through to the VM
- [ ] Flash 1 or 2 ESP32 Bluetooth proxies for SwitchBot and place them centrally
- [ ] Set up Zigbee (ZHA or Zigbee2MQTT) and the Thread border router

**S0.2.5 Backups and basic monitoring from day one**

- [ ] Nightly HA backups to a NAS or iCloud, plus HA config and the router config backup in git
- [ ] Uptime Kuma on the Mac mini checking HA, Tailscale and internet
- [ ] Do one restore test into a fresh VM

### E0.3 Device integrations

Every device in the house appears in Home Assistant with consistent names, rooms and areas.

**S0.3.1 Lights and switches**

- [ ] Philips Hue via the Hue Bridge
- [ ] Govee: enable LAN control per device, HACS cloud integration for older models
- [ ] SwitchBot buttons over Bluetooth; IR remotes learned on the SwitchBot Hub

**S0.3.2 Climate and cleaning**

- [ ] Nest thermostat: pay the Device Access fee (one-time, about CA$7), create the Google Cloud project, add the SDM integration
- [ ] Dreame vacuum via the Dreame HACS integration (Valetudo stays in Phase 3)

**S0.3.3 Media**

- [ ] Samsung TV (Smart TV + SmartThings, wake from off)
- [ ] Samsung soundbar (input, volume)
- [ ] B&O soundbar (Mozart platform)
- [ ] Apple TV (power, apps, playback)
- [ ] Plex Media Server (now playing, play to client)

**S0.3.4 Routers, PC and presence**

- [ ] TP-Link/Deco integration for presence, client lists, WAN status and remote reboot
- [ ] Windows PC: Wake on LAN plus HASS.Agent for sleep and shutdown
- [ ] Phone presence via the companion app

**S0.3.5 Clean model of the house**

- [ ] Name every room, area and device the way you say it out loud
- [ ] Add voice aliases ("big light", "telly") and expose only needed entities to Assist
- [ ] Build the base scenes (morning, leaving, movie, bedtime, away) and area groups so "turn off everything downstairs" works

**S0.3.6 Away and safety mode**

- [ ] Away mode simulates someone at home with lights and TV on a varied schedule
- [ ] Smoke, leak and door alerts (add Zigbee leak and door sensors where needed) go to the phone and every voice device, ignoring quiet hours

### E0.4 Alexa routine inventory and migration

Every Alexa routine, schedule, timer habit and announcement has a Home Assistant equivalent that has run without fault for 2 weeks.

**S0.4.1 Inventory what Alexa does today**

- [ ] List every routine in the Alexa app: trigger, conditions, actions, devices (the app has no export, so this is manual or screenshots)
- [ ] List recurring alarms, reminders, timers, shopping and to-do lists, Drop In and announcement use
- [ ] List Alexa skills in use and which device each Echo controls by room
- [ ] Put it all in a migration table: Alexa item, HA equivalent, status

**S0.4.2 Rebuild routines in Home Assistant**

- [ ] Map each voice-triggered routine to an HA script plus a custom sentence
- [ ] Map each schedule or sensor-triggered routine to an HA automation
- [ ] Rebuild announcements as TTS to the right room's voice device
- [ ] Move Alexa shopping and to-do lists into HA lists (used by voice in S0.5.7)

**S0.4.3 Parallel run, then cutover**

- [ ] Run Alexa and HA side by side for 2 weeks, disabling each Alexa routine as its HA version is confirmed
- [ ] Tick off the migration table only after 2 weeks without a miss

### E0.5 Claude assistant and everyday Alexa features

Cloud Claude is the one brain behind voice, app and chat: it answers questions, searches the web, controls the house through Home Assistant, and covers everything you used Alexa for day to day, on every voice device and in the app.

**S0.5.1 Claude is the conversation agent in Home Assistant**

- [ ] Create an Anthropic API account with a monthly spend cap and an alert threshold
- [ ] Add the Anthropic conversation integration in HA (Sonnet as default, Haiku for routing)
- [ ] Turn on "prefer handling commands locally" so simple commands skip the LLM
- [ ] Write the system prompt: house layout, people, tone, short spoken answers

**S0.5.2 Claude can search and answer**

- [ ] Give the agent web search and page reading for general questions
- [ ] Short spoken answers, with the full answer and sources in the app
- [ ] Test 30 everyday questions and compare against what Alexa used to answer

**S0.5.3 Timers, alarms and reminders**

- [ ] Several named timers per device ("pasta timer, 10 minutes"); ask how long is left, cancel or add time; countdown on the Dot and Show screens
- [ ] Alarms by voice and app: one-off and recurring, snooze, and wake-up with music or lights fading up
- [ ] Reminders by time ("remind me at 6 to call mom") and by arriving or leaving home through phone presence
- [ ] Delivered on the room's voice device and as a phone push; all timers, alarms and reminders listed and editable in the app

**S0.5.4 Announcements and intercom**

- [ ] "Announce dinner is ready" to every room or one room, spoken by the voice devices
- [ ] Automations announce too: doorbell, laundry done, door left open, rain coming
- [ ] Send a spoken announcement from the app; quiet hours mute announcements in bedrooms
- [ ] Live room-to-room talk (like Drop In) between Show devices follows with the voice server in Phase 2

**S0.5.5 Weather, news, stocks and a morning briefing**

- [ ] Weather for home and any city through the Environment Canada or Open-Meteo integration: now, hourly, tomorrow, rain and severe-weather alerts
- [ ] News briefing from sources you pick (for example CBC RSS feeds), summarized by Claude and read aloud, with links in the app
- [ ] Stock quotes and a watchlist through a quote API tool (pick a provider); "how's my portfolio", and price alerts through HA sensors
- [ ] transit service alerts for your usual routes
- [ ] "Good morning" briefing: weather, calendar, reminders, news, stocks and transit, triggered by voice or your first alarm, plus a short evening wrap-up

**S0.5.6 Music and media by voice**

- [ ] Music Assistant add-on in HA with Spotify or Apple Music, Plex, internet radio and podcasts; multi-room speaker groups ("play jazz in the kitchen")
- [ ] Play, pause, skip, volume and "play in the living room" on the soundbars, Apple TV and voice devices; move music to another room on request
- [ ] "Play the next episode of my show on the TV" through Plex and Apple TV
- [ ] Sleep sounds and white noise with a sleep timer

**S0.5.7 Lists and calendar**

- [ ] Add to and read the shopping and to-do lists by voice; the same lists in the app and widgets, optionally synced to Todoist or Google Tasks
- [ ] Google or iCloud calendar: "what's on today", add an event, and reminders before events

**S0.5.8 Everyday extras and parity check**

- [ ] Quick answers: unit and currency conversion, maths, spelling, definitions, translations, sports scores
- [ ] "Find my phone" rings your phone even on silent, through a critical notification from the app
- [ ] Good night by voice: lights off, doors checked, thermostat down, alarm confirmed
- [ ] Keep a parity checklist of every Alexa feature you used (from the S0.4.1 inventory) and test each one by voice and app

**S0.5.9 Safe control**

- [ ] Use a dedicated HA user with access only to exposed entities
- [ ] Locks, garage, purchases and anything irreversible require confirmation
- [ ] Log every conversation and tool call

**S0.5.10 Memory and follow-ups**

- [ ] Keep short-term conversation context per device for follow-up questions
- [ ] Store household facts (preferences, names, routines) the agent can read; answers come in the room you are in, and a conversation started on a voice device continues in the app

### E0.6 Voice pipeline on the Mac mini

A spoken command goes wake word, speech-to-text, Claude or local intent, and spoken reply, all through Home Assistant Assist, without the Mac Studio.

**S0.6.1 Assist pipeline is set up**

- [ ] Speech-to-text: cloud STT for open questions, local Speech-to-Phrase for fixed commands
- [ ] Text-to-speech: Piper locally on the mini; pick the voice
- [ ] Pipeline per room with the right language and voice; follow-up mode so you can keep talking for a few seconds without the wake word

**S0.6.2 Custom wake word**

- [ ] Pick the name; train a microWakeWord model
- [ ] Measure: at least 95% wakes at 3 m, under 1 false wake per day per device

**S0.6.3 Latency is measured**

- [ ] Log time per stage (wake, STT, intent or Claude, TTS)
- [ ] Hit under 1.5 s for simple commands and first spoken words under 3 s for answers

### E0.7 Custom voice devices, first generation

One custom device per room replaces each Echo; they are thin voice satellites on the IoT network that make no decisions.

**S0.7.1 Dot-style device replaces Echo Dot with Clock**

- [ ] Prototype on ESP32-S3 with a ReSpeaker XVF3800 and an I2S amp
- [ ] Fork the Voice PE ESPHome firmware; add the LED dot-matrix clock, timer countdown and status animations
- [ ] 3D-print an enclosure with a fabric front; tune the speaker

**S0.7.2 Show-mini replaces Echo Spot**

- [ ] Waveshare ESP32-P4 3.4" round board with ESPHome voice assistant
- [ ] LVGL screens: clock, weather, timers, now playing, the assistant's answer text

**S0.7.3 Show-style replaces Echo Show**

- [ ] Raspberry Pi 5, Touch Display 2 and XVF3800
- [ ] Linux kiosk showing the app's home dashboard, plus a voice satellite client for Assist
- [ ] Camera-free by default; add room controls and now-playing cards

**S0.7.4 Rollout by room**

- [ ] Build one device per Echo location; install room by room
- [ ] Each room's device announces, plays timers and handles that room's devices by default

### E0.8 Mobile app and widgets

One custom app on iOS and Android, forked from the Home Assistant Companion apps, controls the house and talks to Claude from anywhere.

**S0.8.1 Forks build and ship to your phones**

- [ ] Fork the HA Companion iOS (Swift) and Android (Kotlin) repos; rename, rebrand, new bundle IDs
- [ ] Apple developer account (about CA$141 a year); TestFlight and Android internal track
- [ ] Connect over Tailscale away from home

**S0.8.2 Home tab**

- [ ] Rooms, scenes and device tiles, live over HA's WebSocket API
- [ ] Media cards for TV, soundbars, Apple TV and Plex

**S0.8.3 Assistant tab**

- [ ] Text chat and push-to-talk to the same Claude agent as the voice devices
- [ ] Show the conversation history from voice and app in one place

**S0.8.4 Widgets and quick controls**

- [ ] iOS home and lock screen widgets (scenes, key lights, thermostat)
- [ ] iOS Control Center controls and Siri Shortcuts / Action button for "talk to the assistant"
- [ ] Android home screen widgets and Quick Settings tiles
- [ ] Actionable notifications (doorbell, laundry done, reminder)
- [ ] Apple Watch complication for scenes (optional)

**S0.8.5 Fallback chat channel**

- [ ] Telegram bot to the same agent for chat and alerts while the app matures

### E0.9 Echo retirement

The Alexa account no longer controls anything in the house.

**S0.9.1 Decommission**

- [ ] Unplug a room's Echo only after its device passes the parity checklist (S0.5.8) and its routines run 2 weeks without a miss
- [ ] Unlink smart-home skills from the Alexa account; deregister devices
- [ ] Factory-reset and sell or recycle the Echos

## Phase 1: Agentic work without the Mac Studio

Phase 1 ends with at least 3 subagents running at once, each with shell, browser and desktop, each exiting from its assigned IP, and all of them watchable and controllable from the app. The Mac mini has only 8 GB, so it runs the light parts (agent backend, message bus, egress proxies) and the heavy workspaces run on a rented cloud VM whose traffic comes back through the mini's routes over Tailscale. Everything is built in containers so Phase 2 can move it onto the Studio unchanged.

**Exit criteria:** 3 concurrent subagents with desktops; each job's public IP matches its route at start; a disallowed agent-to-agent message is blocked and logged; from the app you can watch, pause, approve, take over and kill; logins, purchases, sending, deletions and code changes wait for Face ID approval on your phone; passwords never appear in prompts, logs or screenshots; the transit reimbursement job (S1.10.1) runs end to end with two approvals; an agent code change that fails tests or health checks never stays deployed; 30-day searchable audit log; the morning briefing and leave-now alerts arrive unprompted in the room you are in, never during quiet hours.

### E1.1 Agent backend becomes the single decision maker

A custom agent service replaces the stock HA conversation agent, so voice, app, Telegram and subagents all share one brain and one memory.

**S1.1.1 Agent service runs on the Mac mini**

- [ ] Build the agent service on the Claude Agent SDK in a container on the mini (OrbStack or Docker), with a RAM limit
- [ ] Point HA Assist at it through a custom conversation agent integration
- [ ] Point the app's Assistant tab and the Telegram bot at the same service

**S1.1.2 Model routing**

- [ ] Haiku for intent routing and summaries, Sonnet as default, Opus for planning and hard jobs
- [ ] Per-request token budget and daily spend alert

**S1.1.3 Shared memory**

- [ ] One store for household facts, conversation history and job history (SQLite or Postgres on the mini)
- [ ] Migrate the Phase 0 prompt and facts into it; "what do you know about me?" and "forget that" work, and memory is editable in the app

### E1.2 Tools and MCP

The agent reaches the world through tools in a fixed order: APIs and MCP first, browser automation next, computer use last.

**S1.2.1 Core tool servers**

- [ ] Home Assistant MCP server (scoped HA user)
- [ ] Web search and fetch, calendar, email (read first, send with approval), Plex API, push notifications

**S1.2.2 Browser and computer use**

- [ ] Playwright or browser-use inside workspaces for websites
- [ ] Computer use (screenshots, mouse, keyboard) for apps with no other interface

### E1.3 VPN and egress routes

Every workspace exits the internet from a named route (home home IP, a NordVPN country, or the TP-Link router VPN), and the job refuses to start if the IP is wrong. NordVPN's own app is never installed on the Macs, because it would move Home Assistant onto the VPN too.

**S1.3.1 NordVPN routes on the Mac mini**

- [ ] Generate NordLynx (WireGuard) credentials in the Nord account for manual setup
- [ ] List the countries agents need; run one Gluetun container per country (for example `nord-de`, `nord-us-ny`), with HTTP and SOCKS proxy ports open only to workspaces
- [ ] Rely on Gluetun's kill switch and NordVPN DNS so a dropped tunnel or DNS lookup never leaks
- [ ] Measure the mini's RAM with HA plus all routes running; cap the number of containers

**S1.3.2 Home home IP route**

- [ ] Plain proxy container with no VPN as the `home` route, used the same way as the others

**S1.3.3 TP-Link router VPN route**

- [ ] Add the Lab network (VLAN or SSID) on the TP-Link routers for workspaces that must always exit from one country
- [ ] Connect the router's VPN client (NordLynx) to NordVPN for the Lab network only, with a kill switch
- [ ] Lab reaches only the mini's proxy and message-bus ports
- [ ] Confirm Main and IoT still exit from the home IP, and that Lab stops if the router VPN drops

**S1.3.4 Cloud workspaces exit through home routes**

- [ ] Join the cloud VM to Tailscale and force all workspace traffic to its assigned route on the mini
- [ ] Block direct internet from workspaces with a firewall rule so leaks fail closed

**S1.3.5 Route verification**

- [ ] Public IP and country check through the route before each job, and on a schedule per route
- [ ] Alert when a route's IP or country is not what's expected

### E1.4 Workspaces on a rented cloud VM

Each subagent gets its own disposable workspace with a shell, Chromium and a desktop.

**S1.4.1 Workspace host**

- [ ] Rent one Linux VM (about 16 to 32 GB RAM) from a cloud provider; Docker plus Tailscale
- [ ] Expose nothing publicly; reach it only over Tailscale

**S1.4.2 Workspace images**

- [ ] Fork Anthropic's computer-use demo container (or a Kasm image): Linux desktop, Chromium, VNC
- [ ] A CLI-only image for jobs that need no desktop

**S1.4.3 Workspace manager**

- [ ] Create, snapshot and destroy workspaces through an API the orchestrator calls
- [ ] Attach the egress route and secrets per job; never put API keys inside the workspace
- [ ] Job timeout and automatic cleanup

### E1.5 Orchestrator and agent-to-agent messaging

The orchestrator splits a request into subtasks and starts subagents; subagents talk only to peers the policy allows.

**S1.5.1 Orchestrator**

- [ ] Plan a request into subtasks; pick workspace type, route, peers and model for each
- [ ] "Handle this" from any voice device or the app starts a job; results come back wherever you are

**S1.5.2 Message bus and policy**

- [ ] NATS on the mini (light), one subject per task
- [ ] Allow-list per agent pair; anything else is blocked and logged
- [ ] Human messages join an agent's conversation on the same bus

**S1.5.3 Untrusted input**

- [ ] Treat page text, emails and other agents' messages as data; they cannot change policy or permissions

### E1.6 Human control from the app

The app's Agents tab shows every job and lets you step in at any point.

**S1.6.1 Agents tab**

- [ ] Task board: queued, running, waiting for you, done, failed
- [ ] Live screen per agent via noVNC (WebRTC later)

**S1.6.2 Approvals**

- [ ] An approval request pauses the agent and sends a push with Approve and Deny
- [ ] Approvals need Face ID and are signed by a key in your phone's Secure Enclave; the runtime checks the signature, so nothing on a server can approve
- [ ] Logins, purchases, sending messages, deletions and code changes always require approval
- [ ] A CAPTCHA or bot check sends a push asking you to take over

**S1.6.3 Take over and kill**

- [ ] Take over mouse and keyboard in the live view; the agent resumes with a note of what you did
- [ ] Pause, resume and kill (kill destroys the workspace and keeps its log)

### E1.7 Password vault with human approval

Agents never hold the vault: each login is released from your phone for one job after you approve it, and a hacked server only ever sees encrypted data. Your passwords are already in Vaultwarden from S0.1.5.

**S1.7.1 Agents collection**

- [ ] Put the logins agents may ever request in an "agents" collection; banking and card sites stay out until you opt a site in
- [ ] No vault key or service-account token on any server (this is why 1Password service accounts are ruled out)

**S1.7.2 Credential broker and release from your phone**

- [ ] Broker on the mini relays requests; the subagent gives the site, job, reason and route
- [ ] Your phone shows the job, site, exit IP and planned action; you approve with Face ID
- [ ] The app decrypts only that item on the phone and sends it encrypted to the workspace's one-time key; the broker relays only ciphertext
- [ ] The workspace browser fills the fields outside the model's view; screenshots mask password fields and logs never hold the value
- [ ] One-time codes come from the vault under the same approval, or you type them by taking over; passkey-only sites always need your take over

**S1.7.3 End of job, audit and revoke**

- [ ] When the job ends, log out of the site and destroy the workspace; email, bank and Amazon never keep a saved browser profile
- [ ] Log every request, approval, denial and fill; cancel a workspace's pending requests from the app

### E1.8 Agent observability and safety

Every agent action is traceable, searchable and paid for within a cap.

**S1.8.1 Logs and traces**

- [ ] OpenTelemetry traces per request into Langfuse (on the cloud VM, not the mini)
- [ ] Audit log of tool calls, messages, approvals and takeovers, searchable from the app, kept 30 days

**S1.8.2 Spend and limits**

- [ ] Per-job token budget, monthly hard cap on the Anthropic account, daily spend alert
- [ ] Monthly cost of the cloud VM tracked next to API spend

**S1.8.3 Money and account guardrails for agents**

- [ ] Agents never get saved card numbers; any purchase needs your approval in the app showing merchant, item and amount in CAD
- [ ] Hard limits enforced by the agent backend, not the agent: per-purchase cap and monthly cap
- [ ] Gift cards, crypto, money transfers and changes to account security settings are blocked outright
- [ ] Agent purchases use a separate virtual or low-limit card, never your main card
- [ ] Agents' Gmail access skips security, password-reset and 2FA emails unless you approve a specific one
- [ ] Any approved purchase is mirrored as a push alert, and an unexpected card alert pauses all agents

### E1.9 Agent code changes by pull request

Agents can write their own skills, tools, Home Assistant automations and integrations, but a change reaches the house only as a pull request you approve on your phone, and a failed deploy rolls back by itself. Phase 1 covers skills and automations; the runtime itself follows in S2.2.3.

**S1.9.1 Code and config in git**

- [ ] All code and config in git on a self-hosted Gitea or a private GitHub repository
- [ ] Protected paths: the policy file, approval rules, credential broker, egress routes and the deploy pipeline; an agent's pull request can't change them, only you can

**S1.9.2 Coding subagent opens pull requests**

- [ ] A coding subagent works on a branch in a CLI workspace, writes tests and opens a pull request with a plain summary
- [ ] Checks run unit tests, linters, Home Assistant's config check and a replay of recent requests against a staging copy

**S1.9.3 Approve from the app, deploy with rollback**

- [ ] The app shows the summary and diff; you approve with Face ID
- [ ] Deploy tags a release and runs health checks; any failure rolls back to the previous tag and alerts you

### E1.10 Example agent jobs

Phase 1 is proven by real errands that use released logins, egress routes, files, email and approvals together.

**S1.10.1 transit reimbursement report**

You ask "make my transit reimbursement for last month"; an agent logs in to your credit card site with your approval, pulls the transit charges, and emails the list through Gmail after you approve the draft.

- [ ] Gmail tool (API through MCP): create drafts and attach files; sending always waits for your approval
- [ ] Agent opens the credit card site in a Linux desktop workspace on the `home` route, so the bank sees your usual IP
- [ ] Login through the E1.7 vault broker after your approval; you enter any bank 2FA code in the live view
- [ ] Read-only: download the statement or transactions (CSV or PDF) for the requested dates; no payments or settings changes
- [ ] Extract transit pass charges into a table: date, merchant, amount in CAD, and a total
- [ ] Produce a PDF or spreadsheet plus the statement pages as evidence
- [ ] Draft the Gmail message to the recipient you name, with the attachments; you approve or edit in the app before it sends
- [ ] Log the run; save the merchant rules and recipient so next month needs one sentence

**S1.10.2 LinkedIn job applications**

LinkedIn's user agreement forbids automated tools on the site, so applications stay few and reviewed, and an account restriction is a risk you accept.

- [ ] Save your search criteria (roles, locations, salary) and a base résumé
- [ ] Agent searches matching roles on the `home` route and tailors the résumé and cover letter per role
- [ ] Fills each application up to Submit; you approve the login, then each submission
- [ ] Log every application in a spreadsheet

**S1.10.3 Spreadsheet work**

- [ ] Data work on .xlsx files with Python in a CLI workspace
- [ ] Files from OneDrive, Google Drive or iCloud through their APIs
- [ ] A copy is saved first; overwriting an original needs your approval
- [ ] Excel itself, for macros, pivots and charts, follows in a macOS VM in Phase 2 (S2.5.2)

**S1.10.4 Fix or extend itself**

- [ ] Ask for a new skill or automation in plain words; the agent builds it through the E1.9 pipeline and you approve the merge and deploy

**S1.10.5 Package tracking**

- [ ] "Where's my package?" answered from shipping emails, with Gmail read-only access
- [ ] Delivery-day heads-up and "delivered" announcements

**S1.10.6 Voice shopping**

- [ ] "Reorder paper towels" finds the past order and asks for approval on your phone
- [ ] Runs under the S1.8.3 money guardrails: approval per purchase, hard caps, separate card

### E1.11 Proactive assistant (Jarvis layer)

The agent watches the house, your calendar, transit and weather, and speaks up when something needs you. One rule decides how: urgent goes to the room you are in, everything else to your phone, and nothing during quiet hours except safety alerts.

**S1.11.1 One delivery rule**

- [ ] Delivery rule in the agent backend: urgency, the room you are in from presence, per-room quiet hours
- [ ] Safety alerts always get through; everything else waits or goes to the phone

**S1.11.2 Contextual briefings**

- [ ] Morning briefing starts on first motion in the kitchen instead of a fixed time: weather, calendar, transit delays, news, stocks, packages
- [ ] Evening wrap-up of tomorrow's calendar, reminders and deliveries

**S1.11.3 Heads-up alerts**

- [ ] "Leave in 10 minutes for your 3 pm" from the calendar plus transit or traffic times
- [ ] Rain coming before you head out, door left open, laundry done
- [ ] An unexpected card charge from the card alerts

### E1.12 Agent harness (any model, local first)

One harness gives the features of Meta Muse, Claude Cowork and Dispatch, OpenClaw and ChatGPT on our own hardware, with any model: Claude, OpenAI, Gemini, OpenRouter, or local Ollama, MLX and llama.cpp. Code is ready in home-agents (`harness/`, `docs/harness.md`); the feature comparison is in [Agent harness: feature comparison and plan](https://claude.ai/artifact/TP6vM4fUw5KPnHYGQRBGYy). Local models switch on with S2.3.

**S1.12.1 Any model provider**

- [ ] Claude plus any OpenAI-compatible API (OpenAI, Gemini, OpenRouter) and local servers (Ollama, mlx\_lm.server, llama.cpp, LM Studio)
- [ ] Model chains with failover; private mode uses local models only; every provider counts toward the CAD cap

**S1.12.2 Work like Cowork and Dispatch**

- [ ] Projects with standing instructions, chat sessions, background tasks with live steps, steer and stop
- [ ] Attachments in, artifacts out; done notifications to the phone
- [ ] Agent profiles with allow, ask, block per tool

**S1.12.3 Extend like OpenClaw**

- [ ] Skills in SKILL.md format, added only by pull request; MCP connectors where anything not read-only asks first
- [ ] Web search for every model through self-hosted SearXNG
- [ ] Heartbeat every 30 minutes (local model, silent unless needed); signed webhooks from HA and Uptime Kuma

**S1.12.4 Reach it from anywhere on the tailnet**

- [ ] Web console at /console; Telegram by long polling with pairing codes (no open port)
- [ ] Tasks, artifacts and pairing in the phone app; then Signal, WhatsApp and Slack

**S1.12.5 Prove it**

- [ ] End to end on the Mac mini with Claude; then on the Studio with a local model in private mode
- [ ] Voice moves onto the router after the local model benchmark (S2.3.1)

## Phase 2: Mac Studio brings agents and AI home

Phase 2 ends with the agent runtime, voice server and local models all on the Mac Studio, the cloud workspace VM switched off, and the orchestrator spinning off 5 or more subagents (Linux desktops and macOS VMs) on its own. The Mac mini goes back to only running the house and the egress routes.

**Exit criteria:** 5+ concurrent subagents with desktops, 20+ possible; up to 2 macOS VM workspaces; voice runs locally with cloud fallback; private and offline requests answered by the local model; house and basic voice keep working when the Studio is off; you can interrupt a long answer, and Show screens put up the right card on their own.

### E2.1 Mac Studio bring-up

**S2.1.1 Buy and install**

- [ ] Confirm memory options and the Canadian price in CAD; 256 GB is the suggested size (one large local model plus 20+ workspaces)
- [ ] Put it on the UPS by USB; wired Ethernet on the Main network; fixed IP; Tailscale
- [ ] OrbStack, Tart, secrets store, backups and git-managed config

**S2.1.2 Mac Studio is set up as an always-on server**

- [ ] Same baseline as the Mac mini (S0.2.2): never sleep, start after power failure, restart after freeze, "hub" user plus separate admin, no Apple ID, security responses on and upgrades off, SSH and Screen Sharing only over Tailscale
- [ ] FileVault on, because the Studio holds browser sessions and agent data; planned restarts use `sudo fdesetup authrestart`, and an unplanned power loss waits for your unlock and alerts you while the house runs on the mini
- [ ] Agent runtime, voice server, NATS and the local model server start at boot as launchd jobs or containers set to restart always
- [ ] UPS shuts the Studio down first (about 50% battery) so the mini, router and Zigbee stick keep the rest
- [ ] Monthly maintenance window: update one Mac at a time, never both on the same day
- [ ] Test a power cut: the Studio waits for unlock, voice falls back to the mini, and services come back by themselves after unlock

### E2.2 Move the agent runtime onto the Studio

**S2.2.1 Migrate services**

- [ ] Move the agent service, memory store, NATS and Langfuse from the mini and cloud VM to the Studio; the vault, credential broker and egress routes stay on the mini
- [ ] Run the workspace manager on OrbStack locally; same images as Phase 1
- [ ] Workspaces reach the mini's egress routes over the LAN, still fail-closed

**S2.2.2 Retire the cloud workspace VM**

- [ ] Run both hosts in parallel for a week, then cancel the VM

**S2.2.3 Runtime changes by pull request**

- [ ] Extend the E1.9 pipeline to the agent runtime itself, with a staging copy on the Studio
- [ ] Same protected paths, Face ID approval and automatic rollback

### E2.3 Local models

**S2.3.1 Local LLM serving**

- [ ] Serve a 100B to 200B class open-weight model at 4-bit with MLX or llama.cpp
- [ ] Benchmark tool use, speed and memory with 20 workspaces running alongside

**S2.3.2 Routing and fallback**

- [ ] Rules for private requests (local only) versus cloud (hard planning, images)
- [ ] Automatic fallback to local when the internet or Anthropic API is down; non-urgent jobs queue

### E2.4 Local voice server

Voice moves off HA Assist cloud STT to a streaming voice server on the Studio, with Assist on the mini kept as the fallback.

**S2.4.1 Voice server**

- [ ] LiveKit for WebRTC audio from devices and the app
- [ ] Local STT (Whisper or Parakeet) and streaming TTS (Piper or a better local voice)
- [ ] HA intent engine first for simple commands, then the agent; barge-in so you can interrupt a long answer

**S2.4.2 Devices v2**

- [ ] Show-style and Show-mini stream to the voice server and render screen cards over WebSocket; screens put up the relevant card on their own (timer, doorbell, next meeting, now playing)
- [ ] App hands-free voice mode and two-way intercom (like Drop In) through the same server
- [ ] Devices fall back to Assist on the mini when the Studio does not answer

### E2.5 Agents spin off and use GUIs at scale

**S2.5.1 Dynamic spin-off**

- [ ] Orchestrator starts and stops subagents on its own within a per-request budget (count, time, tokens)
- [ ] Subagents can request a peer; the policy approves or asks you

**S2.5.2 More workspace types**

- [ ] macOS VM workspaces with Tart (at most 2 at a time), with computer use and logins only through the E1.7 broker; Excel runs here for macros, pivots and charts
- [ ] Optional Windows on Arm VM (Parallels or UTM) for Windows-only apps
- [ ] Snapshot and roll back a workspace from the app

**S2.5.3 Better live view**

- [ ] Replace noVNC with WebRTC screens for lower latency in the app
- [ ] Grid view of all running agents

### E2.6 Two Macs watch each other

**S2.6.1 Cross-monitoring and recovery**

- [ ] Second Uptime Kuma on the Studio watching the mini, and the mini watching the Studio
- [ ] Host dashboards (Beszel or Prometheus plus Grafana) for both Macs
- [ ] Rehearse restoring the HA backup onto the Studio as a temporary host
- [ ] Monthly restore test on the calendar

### E2.7 Routine suggestions

**S2.7.1 The assistant suggests automations**

- [ ] Find repeated actions in HA history ("you dim the lights at 10 every night; make it a routine?")
- [ ] Accepted suggestions become automations through the E1.9 pull request pipeline

## Phase 3: Leftover and stretch work

Phase 3 is a backlog with no fixed order; pick items as they become worth doing.

### E3.1 Hardware polish

- [ ] Final enclosures and better speaker drivers to close the gap with Echo sound
- [ ] More rooms, outdoor or bathroom devices
- [ ] mmWave presence sensors so answers and alerts follow you room by room

### E3.2 Resilience

- [ ] 4G/5G backup internet line on the TP-Link router
- [ ] Hot spare or second HA host so the Mac mini is no longer a single point of failure

### E3.3 More local control

- [ ] Root the Dreame vacuum and switch to Valetudo, if the model is supported
- [ ] Expand Thread and Matter devices; replace cloud-only devices where possible

### E3.4 App and assistant extras

- [ ] Vision: "what am I holding?" from the phone camera, and "who came to the door?" from a doorbell or security camera with a local vision model so footage stays home
- [ ] Per-person voices: speaker recognition so your reminders and calendar stay separate from a guest's
- [ ] Apple Watch app and CarPlay
- [ ] Grafana dashboards (house, voice, agents, cost) inside the app
- [ ] Energy monitoring and cost dashboards

### E3.5 Agent extras

- [ ] Scheduled recurring agent jobs (monthly transit reimbursement, price watches, reports)
- [ ] Reusable agent skills library shared across subagents

## Dependencies, risks and open questions

The biggest risk is the Mac mini's 8 GB: it carries HA, egress proxies and the agent service until the Studio arrives, so its RAM is measured at each step.

| Item | Type | Affects | Mitigation or next step |
| --- | --- | --- | --- |
| Mac mini 8 GB runs out of RAM | Risk | E1.1, E1.3, E1.5 | Heavy workspaces and Langfuse live on the cloud VM; cap Gluetun containers; alert at 90% RAM |
| A hacked server tries to get your passwords or approve actions | Risk | E1.6, E1.7 | Logins released from your phone with only ciphertext on servers; approvals signed in the phone's Secure Enclave |
| FileVault off on the mini | Risk | S0.2.2 | Nothing personal stored and vault data end-to-end encrypted; rotate HA and Tailscale keys if the mini is ever stolen |
| TP-Link model lacks VLANs, mDNS reflection or VPN policy routing | Risk | E0.1, S1.3.3 | Check in S0.1.1; fall back to a second mini interface on IoT, or Gluetun-only routes |
| An agent code change breaks the house | Risk | E1.9 | Tests, staging replay, your approval, automatic rollback; protected paths |
| LinkedIn restricts the account for automated use | Risk | S1.10.2 | Few, reviewed applications; you approve each submission |
| Custom device speaker quality falls short of Echo | Risk | E0.7, E0.9 | Echos stay in place until each room passes 2 weeks |
| Alexa routines can't be exported | Risk | E0.4 | Manual inventory with screenshots before anything is unplugged |
| App store review for forked Companion apps | Risk | E0.8 | Ship via TestFlight and Android internal track first |
| Cloud STT needed for open questions in Phase 0 | Dependency | E0.6 | Pick the provider; moves local in E2.4 |
| Cloud VM cost in Phase 1 | Dependency | E1.4 | Track monthly in CAD; retire in E2.2 |
| Mac Studio arrival date | Dependency | All of Phase 2 | Phase 1 is built in containers so the move is a redeploy |
| Wake word name | Open question | E0.6 | Choose before training the model |
| Which NordVPN countries are needed | Open question | E1.3 | List them; each one is a Gluetun container |
| Which cloud provider and region for the Phase 1 VM | Open question | E1.4 | Pick a Canadian region close to home for low live-view latency |
