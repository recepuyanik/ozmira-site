# OZMIRA

[Türkçe](README.md) · English · [Deutsch](README.de.md) · [Español](README.es.md)

A local app that shows your own AI systems in one 3D scene. Each system appears in its own color and shape (galaxy, hologram fiber, double helix, black hole, sphere). When an agent starts working, a signal travels from the center to it, returns when the job is done, and the tokens spent show up in the scene.

No account, no cloud. Your system definitions, events and access key stay on your computer only. There is an optional anonymous usage sharing that is off by default (details below).

## Starting

You need: Python 3.8 or later (tested on 3.13) and an up-to-date browser. Nothing else to install.

| System | Run |
| --- | --- |
| Windows | double-click `ozmira.bat` |
| macOS | double-click `ozmira.command` (first time: right-click, Open) |
| Linux | `./ozmira.sh` |
| All | `python3 ozmira.py` (on Windows: `py ozmira.py`) |

The browser opens by itself. If it does not, go to the `http://localhost:8787/` address printed in the terminal. Press Ctrl+C in the terminal to stop.

On first start you get a sample scene and a setup wizard. In the wizard you name the center and add your systems. There are ready templates (Claude, ChatGPT, Gemini, Grok, Other); you can change name, style and color. Later you can change everything with the Edit button at the bottom.

## Opening as an app window

You can open OZMIRA in its own window without an address bar instead of a browser tab. This needs Chrome, Edge or Brave on your computer (Edge is already on Windows 10 and 11).

| System | Run |
| --- | --- |
| Windows | double-click `ozmira-uygulama.bat` (opens no console window) |
| macOS | double-click `ozmira-uygulama.command` |
| Linux | `./ozmira-uygulama.sh` |
| All | `python3 ozmira.py --uygulama` |

OZMIRA stops when you close the window. The window uses a separate profile folder (`.ozmira/pencere`), so your personal browser sessions and sign-ins stay out of it.

To put a shortcut on the desktop (and, on Linux, in the application menu), run this once:

```
python3 ozmira.py kisayol
```

The shortcut opens the app window and uses the OZMIRA icon (`ozmira.ico`, `web/icon.png`). It is a shortcut, not a single-file `.exe` or an installer, and it needs Python to be installed.

## Language

The interface comes in Turkish, English, German and Spanish. On first start the browser's language is used (English if the language is not supported). You can change it any time with the language button at the right end of the dock (TR, EN, DE, ES); your choice is remembered in this browser. Command-line output follows the system language; force it with `OZMIRA_DIL=tr|en|de|es`. This README is also available as `README.md` (Turkish), `README.de.md` and `README.es.md`.

Note: the command words are Turkish and stay the same in every language: `olay` = event, `basla` = start, `bitti` = done, `kisayol` = shortcut, `--uygulama` = app window.

## Free and Pro

| | Free | Pro |
| --- | --- | --- |
| 3D scene, event reporting, token counter, local operation | Yes | Yes |
| AI systems (Claude, ChatGPT, Gemini etc.) | Up to 2 | No limit* |
| Agents | Up to 7 in total | No limit* |
| Visual styles | Galaxy, Sphere | All five (including Fiber, Double helix, Black hole) |

\* Pro has no paid limit. There is only a technical ceiling so the scene stays smooth: 24 systems and 200 agents per system.

On the free version, if your profile contains more than 7 agents (including those read from a folder) you get a warning when saving. If your agent folder grows later, the first 7 agents appear in the scene and the rest is noted on screen as "+N agents beyond the Free limit"; nothing is deleted.

You paste the Pro license key from the Account button in the dock (it says "Free" while you are on the free version). The key is verified offline on this computer; it does not contact a server. A license can be time-limited; when it expires the app returns to the free limits, your saved profile is not deleted but the limits apply when you save again.

## Adding your agents

There are two ways for each system, and you can use both together:

- **Folder:** point to an agent folder made of `.md` files. Each file's name becomes the agent name; if the file starts with `name:` and `description:` lines, those are used. Only the first 4 KB of a file is read and only these two lines are taken; the content is not stored.
- **By hand:** type agent names one per line or separated by commas.

## Showing work live (Connect)

OZMIRA cannot know on its own when an agent is working; you need to tell it. The Connect button in the dock lets you copy ready-made commands for your systems.

```
python3 ozmira.py olay <system> <agent> basla
python3 ozmira.py olay <system> <agent> bitti 1200 "task label"
```

`<system>` is the system's short ID (derived from its name; the Connect window lists the IDs). `1200` is the number of tokens spent and is optional. You can do the same over HTTP:

```
curl -X POST http://localhost:8787/api/olay \
  -H "X-Token: <your key>" -H "Content-Type: application/json" \
  -d '{"sistem":"<system>","ajan":"<agent>","olay":"bitti","token":1200}'
```

You can call this from your own scripts, automations, or the before and after steps of your tools. The only things sent are system, agent name, event type, token count and, if you like, a task label of up to 120 characters. No prompts, replies or code are sent.

If an agent reports `basla` and does not report `bitti` within 30 minutes, it stops showing as running in the scene.

## Privacy and security

- **Where the data is:** in the `.ozmira` folder in your home directory (`profil.json`, `olaylar.jsonl`, `ayar.json`, `kullanim.json`, `.token`). Set the `OZMIRA_HOME` environment variable for another place. Unless you share this folder, nobody sees your system. The app folder itself holds no personal data and can be shared.
- **Closed to the network:** the server listens on `localhost` only. Other computers cannot connect.
- **Key:** a random key is created on first run (`.ozmira/.token`; on macOS and Linux only your user can read it; on Windows file permissions work differently, so if you share your computer with others, put the folder in a protected place). All API requests require this key; Host and Origin headers are checked.
- **Sharing is off by default.** If you allow it in the Account window and a collector address (`geri_bildirim_adresi.txt`) came with this copy, the following is sent anonymously at most once a day: version, plan, a random installation ID and counters (events received, times the profile was saved). Your system names, agent names, task labels, folder paths, token amounts and IP address are not sent. Text you write in the feedback window is also sent only with this permission; what you write is in every case also saved in the data folder on this computer.
- **Other external connection:** loading the three.js library from jsdelivr. To turn this off, see "Offline use" below.
- **Browser protection:** a strict Content-Security-Policy is used; text that comes from files, such as agent names or descriptions, is written to the page as plain text, not as HTML.
- **To reset:** stop the app and delete the `.ozmira` folder.

## Offline use

```
python3 ozmira.py vendor
```

This downloads three.js once and puts it in `web/vendor/three` (with its MIT license). After that OZMIRA runs without connecting to the internet at all. If there is no internet, the command stops with a clear error message and leaves no partial file.

## Viewing from a phone (optional, use with care)

```
python3 ozmira.py --lan
```

The server becomes reachable from the same network and the terminal prints an address for the phone. The page does not open without the `?k=` key at the end of the address. Still, anyone on the same network can reach the server; do not use it on shared Wi-Fi (cafe, hotel, school). Restart normally when you are done.

## Files

| File | Purpose |
| --- | --- |
| `ozmira.py` | Server and command line (start, `--uygulama`, olay, vendor, bilgi, kisayol) |
| `ozmira.bat`, `ozmira.command`, `ozmira.sh` | Launchers for the browser |
| `ozmira-uygulama.bat`, `.command`, `.sh` | Launchers for the app window |
| `ozmira.ico`, `web/icon.png`, `web/icon.svg` | Icons |
| `web/index.html`, `web/stil.css` | Page and look |
| `web/app.js` | Panel, setup wizard, Connect, Account, Feedback and Language windows |
| `web/i18n.js` | Texts of the four languages and translation of server error messages |
| `lisans_acik.json` | License verification key (may be public; the seller adds it) |
| `web/sahne.js` | The 3D scene (five styles, 1 to 24 systems) |

`python3 ozmira.py bilgi` prints the data folder and the event count.

## Limits

- The scene is a heavy 3D image. On old graphics cards or in battery saver mode the quality drops by itself; glow may switch off.
- Technical ceiling (Pro included): 24 systems, 200 agents per system. With more than 8 systems the scene grows and the camera moves back; this layout has not yet been visually tried on a real graphics card.
- Version 0.3.0. Designed for local use only.
- The app window and desktop shortcut were tested with automated tests on Linux; the shortcut command has not yet been tried on a real Windows or macOS computer. If it does not work, use `ozmira-uygulama.bat` / `.command`.
- The translations of the four languages were prepared with AI; a native speaker of English, German or Spanish should review them.
- The scene was tried on a single Windows computer with a real graphics card. If the scene does not appear on first start on other graphics cards, check that WebGL is enabled in the browser.

## New in 0.3.0

- **Panel:** the Panel button in the bar opens the control panel: summary tiles, bars for the last 7 days, a searchable and sortable agent table, an event list and CSV export. Start and Finish record an event by hand; they do not run the agent.
- **Guide:** the Guide button explains setup and further topics in ten short lessons.
- **Claude Code connection:** `ozmira_hook.py` sends an event to OZMIRA when Claude Code starts and finishes a subagent. In the `hooks` section of your Claude Code settings, add this command for `PreToolUse` and `PostToolUse` (matcher: `Task|Agent`): `python ozmira_hook.py` (with the full file path). If your system id is not `claude`, set the `OZMIRA_SISTEM` environment variable. Tested with simulated input; not yet tried with a real Claude Code.
- **Flow:** the Flow tab in the panel shows input and output tokens per system and agent. To report the split: `ozmira.py olay <system> <agent> bitti girdi=800 cikti=400` (girdi = input, cikti = output). If you enter prices it estimates the cost; prices are kept only in the browser.
- **Copyright:** © 2026 OZMIRA, all rights reserved. See `LISANS.txt`.
