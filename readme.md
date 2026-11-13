# 💜 EmOS — a complete desktop operating system in a single HTML file

> **EmOS 2.0** is a full desktop environment that runs entirely inside a web browser. It has a boot screen, user accounts and a lock screen, a windowed desktop with four virtual workspaces, a file system, a terminal, an AI assistant, 40+ apps, a retro-game emulator covering 38 systems, and a media suite. It all ships as **one `.html` file**. There is no server, no build step, no framework and no install.

Open the file and the OS boots.

---

## Table of contents

1. [At a glance](#at-a-glance)
2. [Quick start](#quick-start)
3. [Accounts, lock screen & security](#accounts-lock-screen--security)
4. [The desktop environment](#the-desktop-environment)
5. [Keyboard shortcuts](#keyboard-shortcuts)
6. [System apps](#system-apps)
7. [BubberBot — the built-in AI assistant](#bubberbot--the-built-in-ai-assistant)
8. [The Terminal](#the-terminal)
9. [Productivity & tools](#productivity--tools)
10. [Creative apps](#creative-apps)
11. [Media suite (the "B" apps)](#media-suite-the-b-apps)
12. [Internet apps](#internet-apps)
13. [Games](#games)
14. [The Emulator (38 systems)](#the-emulator-38-systems)
15. [The Store](#the-store)
16. [Bubber, the virtual pet](#bubber-the-virtual-pet)
17. [Settings, personalization & accessibility](#settings-personalization--accessibility)
18. [Data, storage & backups](#data-storage--backups)
19. [Architecture & file anatomy](#architecture--file-anatomy)
20. [Network access & privacy](#network-access--privacy)
21. [Browser compatibility](#browser-compatibility)
22. [Known limitations](#known-limitations)
23. [Security notes for the author](#security-notes-for-the-author)
24. [Hacking on EmOS](#hacking-on-emos)
25. [Credits](#credits)

---

## At a glance

| | |
|---|---|
| **Name** | EmOS (window title: *EmOS*) |
| **Version** | 2.0 |
| **Format** | One self-contained HTML file (about 97 MB) |
| **Stack** | Vanilla JavaScript, HTML and CSS. No dependencies and no build tools |
| **Apps** | 40+ built-in, installable and uninstallable through the Store |
| **Emulation** | 38 retro systems through a bundled EmulatorJS build |
| **Virtual desktops** | 4 |
| **Users** | Multi-user with SHA-256 hashed passwords, admin and standard roles, and a Guest account |
| **Persistence** | Browser `localStorage`, with a restorable session |
| **AI** | BubberBot, an assistant that can control the OS (free provider by default; Anthropic and OpenAI supported) |
| **Offline** | The OS, emulator and most apps work offline. Some apps need the internet (see [Network access](#network-access--privacy)) |

---

## Quick start

### Run it locally

1. Download `EmmettOS.html`.
2. Double-click it, or drag it into Chrome, Edge, Firefox or Safari.
3. Wait about a second for the boot animation, then pick a user on the lock screen.

The file is large, so give it a few seconds to load the first time. The emulator payload is stored as inert text and decoded only when you launch a game.

### Host it

Because it is a static file, any static host works: GitHub Pages, Netlify, Cloudflare Pages, an S3 bucket, or `python3 -m http.server`.

```bash
# from the folder that contains the file
python3 -m http.server 8080
# then open http://localhost:8080/EmmettOS.html
```

Serving over **HTTPS or localhost** (a *secure context*) is recommended. Camera, microphone, clipboard reading, screen recording, wake lock and notifications all require a secure context, and the Security app reports whether you have one.

### First boot

- The OS creates your home folder at `Users/<name>/` with `Documents/` and `Pictures/`.
- A welcome sticky note and a `readme.txt` are pre-seeded.
- Fourteen apps are pinned to the desktop by default: Browser, BubberBot, Files, Settings, Terminal, Store, Editor, Calculator, Calendar, Clock, Paint, Notes, Weather and Task Manager.

---

## Accounts, lock screen & security

### Multi-user system

EmOS ships with three accounts:

| Account | Role | Notes |
|---|---|---|
| Emmett | Admin | Seeded with a password (see [Security notes](#security-notes-for-the-author)) |
| Ryan | Admin | Seeded with a password |
| Guest | Standard | No password |

Each user gets a **private home folder**, and the file system and terminal start in your own folder after sign-in.

**Admins** can add users, remove users, promote or demote others between admin and standard, and change their own password. You cannot remove or demote yourself.

### Lock screen & sign-in

- Pick your avatar and enter your password to sign in.
- **Brute-force protection**: three wrong passwords lock sign-in for 30 seconds. Each failure is written to the system log.
- **Sign-in history**: the last 20 sign-ins are recorded (viewable in the Security app).
- **Auto-lock**: after 0, 1, 5, 15 or 30 minutes of inactivity, in Settings or Security.
- **Lock now**: from the launcher, from `Ctrl+Alt+L`, from the terminal (`lock`) or by asking BubberBot.
- **Power states**: *Lock*, *Sleep* (click to wake), *Restart* (re-runs boot) and *Power off* (click to boot) are all in the launcher.

### Password handling

Passwords are stored as **SHA-256 hashes** with a fixed `emos` salt prefix, using the browser's Web Crypto API. Any plaintext password found in saved data is automatically hashed and removed on startup.

---

## The desktop environment

### Taskbar

The taskbar runs along the bottom of the screen.

- **Launcher button** (the Bubber blob): opens the app launcher.
- **Search button 🔍**: opens the launcher with the search box focused.
- **Window buttons**: one per open window on the current desktop. Click to focus, minimize or restore.
- **Virtual desktop switcher**: buttons **1 to 4**.
- **System tray**: online/offline indicator, mute indicator and battery level with a charging indicator. Click it for Quick Settings.
- **Clock**: time and date. Click it for the Notification Center.

### Launcher & universal search

Open it with **Ctrl+Space** or the taskbar button. It shows a grid of your installed apps and searches as you type across:

- **Apps** (by name)
- **Settings sections** (Display, Sound, Power, Accessibility, Users, Theme, Security, Network)
- **Files** anywhere in the file system (first 6 matches)
- **The web**: *"Search the web for …"* opens it in the built-in browser
- **BubberBot**: *"Ask BubberBot …"* sends your query to the assistant

The launcher footer shows the signed-in user, with **Lock**, **Sleep**, **Restart** and **Off** buttons.

### Window manager

Every app opens in its own window, with:

- **Dragging** by the title bar. A dragged window is clamped so it cannot slide above the desktop.
- **Resizing** from the window's corner.
- **Minimize, maximize and close**.
- **Snap**: drag a window to the left or right screen edge for a half-screen snap, or to the top edge to maximize.
- **Move to next desktop** (the ⧉ button): sends the window to the next of the four virtual desktops.
- **Focus management**: z-order tracking and highlighted borders on the active window.
- **Tile windows**: auto-arranges all visible windows on the current desktop into a grid.
- **Process model**: each window gets a PID, which the Task Manager and the terminal use.
- **Pop-in animation**, which you can turn off in Settings.

### Virtual desktops

Four workspaces (**Ctrl+Alt+1 to 4**). Windows belong to one desktop each, and the taskbar shows only the current desktop's windows.

### Desktop icons & context menus

- Pinned apps appear as desktop icons in a column grid.
- **Right-click the desktop**: next wallpaper, tile windows, open Settings, open Terminal.
- **Right-click an app icon**: pin to or unpin from the desktop, uninstall.

### Desktop widgets

A widget stack in the top-right corner shows:

- A live clock and date.
- A live status card with FPS, JS heap usage, online/offline state, battery percentage, open to-do count and today's calendar event count.

### Quick Settings panel

Click the tray area:

- **Toggle tiles**: Do Not Disturb, Night Light, Mute, Power Saver, Fullscreen, Keep Awake (screen wake lock).
- **Volume** and **brightness** sliders.
- Online status, a lock button and a shortcut to Settings.

### Notification Center

Click the clock. It lists every notification with a timestamp and a **Clear** button. Notifications also appear as toasts (unless Do Not Disturb is on), and when the tab is hidden and permission is granted they fire native **system notifications**.

### Session restore

EmOS remembers your open windows, their positions and sizes, your desktop, your clipboard history, your notifications, your log and your current folders. After you sign in as the same user, it **reopens your workspace**. Turn this off or clear it in Settings → Session & storage. Minecraft is never auto-restored, because it is a heavy download.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl` + `Space` | Open or close the launcher |
| `Alt` + `W` | Cycle through windows on the current desktop |
| `Alt` + `←` / `→` | Snap the focused window to the left or right half |
| `Ctrl` + `Alt` + `1`–`4` | Switch virtual desktop |
| `Ctrl` + `Alt` + `L` | Lock the screen |
| `Alt` + `B` | Open BubberBot |
| `Ctrl` + `S` | Save, in the Editor |

Games, the browser and other apps add their own keys. These are listed in each app's section.

---

## System apps

System apps (`sys`) cannot be uninstalled.

### 📁 Files
A full file manager over the virtual file system (`Root → Users / Drives`).
- Breadcrumb navigation and an **Up** button.
- Live **search filter**. Folders are sorted before files.
- **New file**, **new folder**, **rename**, **delete**, **copy**, **cut** and **paste**.
- **Upload** files from your real computer. Text-like files are stored as text and everything else as data URLs.
- **Smart open**: folders navigate, images open in Photos, audio and video in Media Player, `.html` files in the HTML Editor and everything else in the Editor.
- Deleted items go to the **Trash** and can be restored.

### 🗑 Trash
Lists deleted items with **Restore** and **Empty trash**. Restoring puts an item back in its original folder.

### ⚙️ Settings
Display, Sound, Personalization, Power, Accessibility, Users, Session & storage and About. See [Settings](#settings-personalization--accessibility).

### 📊 Task Manager
Updates once per second.
- **Frame rate** (FPS).
- **UI load**, measured as event-loop delay.
- **JS heap**, **storage used and quota**, **CPU cores**, **network** type and downlink, and **uptime**.
- A **process list** with each app's PID, runtime, DOM node count and an **End** button.

### ⌨️ Terminal
A shell-style command line. See [The Terminal](#the-terminal).

### 🛡️ Security
- Secure-context check and password-hash status.
- **Permission manager** for camera, microphone, geolocation, notifications and clipboard-read, with a **Request** button for each.
- Auto-lock timer, the sign-in brute-force policy and the **recent sign-in history**.
- Storage usage and an **Erase all data** option.

### 🌐 Network
- Online/offline state, connection type, downlink, round-trip time and data-saver state, from the Network Information API.
- **Public IP lookup**.
- **Latency test**: 5 probes with min, average and max.
- **Ping**: checks whether a host is reachable and how long it takes.

### 🔌 Devices
- **Audio and video devices**: scans microphones, cameras and speakers.
- **Bluetooth**: scans for a nearby device with Web Bluetooth.
- **Game controllers**: shows connected gamepads.
- **This device**: platform, GPU renderer string and more.

### 💾 Backup
- **Create restore points**: the last 3 are kept.
- **Restore** from a point.
- **Export** a `emos-backup.json` file and **import** one back.
- Backups cover the file system, calendar events, to-dos, notes, alarms and theme settings.

### 📜 Logs
A rolling log of up to 150 entries (app launches, notifications, failed sign-ins). **Run diagnostics** checks 12 browser capabilities: localStorage, WebAssembly, WebGL, Clipboard, Web Audio, camera/WebRTC, Notifications, Web Bluetooth, Gamepad, Wake Lock, Fullscreen and secure context.

### 📋 Clipboard
Keeps a **history of the last 20** things you copied or cut inside EmOS. You can re-copy any entry or import from the system clipboard.

### ✏️ Editor
A plain-text editor with a path bar for saving anywhere, adjustable font size (12, 14, 16, 20), a live line, word and character count, `Tab` indentation and `Ctrl+S` to save.

### 🧩 HTML Editor
A code editor for HTML files. Open any `.html` file from the Files app or the terminal to edit and preview it.

### 🖼 Photos
An image viewer with previous/next, a zoom slider and a **Set as wallpaper** button. It browses all images in the current folder.

### 🎵 Media Player
Plays local audio and video files with a playlist, playback speed (1×, 1.25×, 1.5×, 2×) and loop.

### 📷 Camera
Live webcam preview with **Start**, **Stop** and **Snapshot**. Snapshots are saved as JPEGs to `Pictures/`.

### 🔴 Screen Recorder
Uses the browser's screen-capture API (with audio when available) to record to WebM. You can preview and **download** the result.

---

## BubberBot — the built-in AI assistant

BubberBot is a chat assistant that can also **operate the OS for you**. Open it from the taskbar, from the launcher or with `Alt+B`.

### Two brains in one

1. **Instant local commands.** A rule-based parser answers common requests with no AI call, so they run immediately and offline:
   - *"open calculator"*, *"close files"*, *"close everything"*
   - *"set volume to 40"*, *"volume up"*, *"mute"*, *"brightness 70"*
   - *"turn on night light"*, *"enable high contrast"*, *"large text on"*
   - *"change accent to teal"* (or any `#rrggbb`), *"next wallpaper"*, *"wallpaper 3"*
   - *"lock"*, *"tile my windows"*, *"desktop 2"*
   - *"add todo buy milk"*, *"note: call Sam"*, *"timer 10 min"*, *"alarm 7:30 am"*
   - *"remind me in 20 minutes to stretch"*
   - *"search the web for …"*
2. **A full AI model.** Everything else goes to a language model that replies with structured JSON: a short message plus a list of OS actions to run.

### What BubberBot can do

**Windows and apps.** Open and close apps, close all, switch desktop, tile windows, lock, install and uninstall apps.

**Settings.** Accent color, wallpaper, volume, brightness, night light, Do Not Disturb, mute, animations, large text, high contrast, magnifier, captions, screen reader, power mode, auto-lock and interface scale.

**Productivity.** Add and complete to-dos, add sticky notes, add calendar events, set alarms, start timers, show notifications.

**Files.** Create, append to, read, list, open and delete files (paths are relative to your home folder).

**Web and utilities.** Web search, open URLs, copy to the clipboard, calculate, run **read-only** shell commands, inspect system state.

**Automation.**
- **Scheduling**: *"in 15 minutes, close all windows."*
- **Macros (routines)**: save a named sequence of actions and run it later with its name.
- Scheduled jobs run in the background (checked every 5 seconds) while you are signed in, and BubberBot reports back when they finish.

### Safety model

- **Approval for destructive actions.** Deleting files, uninstalling apps and overwriting files show an **approval card** in the chat. Nothing happens until you click Approve.
- **Sandboxed file paths.** `..` and `.` are rejected, and BubberBot can only touch **your own home folder**.
- **Allow-listed settings.** Only whitelisted settings can be changed, each with validated ranges.
- **Read-only terminal access.** Only these commands are allowed: `ls cat pwd tree find grep wc head date whoami uname uptime neofetch ps free df ping`.
- **Prompt-injection hardening.** The system prompt instructs the model to treat file contents, web text and tool results as **data, not instructions**, and tool results are labeled that way when fed back. Scheduled and macro actions are filtered so they cannot recursively create more macros or schedules.
- Up to 10 actions per reply and 4 tool-use rounds per message.

### Choosing the AI provider

Open ⚙ in BubberBot to configure:

| Provider | Default endpoint | Notes |
|---|---|---|
| **Free** (default) | `text.pollinations.ai/openai` | No key needed |
| **Anthropic** | `api.anthropic.com` | Needs your API key; default model preset `claude-haiku-5-5` |
| **OpenAI-compatible** | `api.openai.com/v1` (editable) | Works with any OpenAI-compatible URL, key and model |

Other options:
- 🎙 **Voice input** and 🔊 **spoken replies**, through the browser's speech APIs.
- Quick-start suggestion chips: *What can you do? · Open my files · Turn on night light · Plan my day · Tidy up my windows · What are my to-dos?*
- If the AI cannot be reached, BubberBot says so and keeps handling the simple local commands.

> 🔑 Your API key is stored in the browser's `localStorage` on your machine and sent only to the provider you choose.

---

## The Terminal

A scrollable shell over the virtual file system and the OS. Each user starts in their own home folder.

| Category | Commands |
|---|---|
| **Files** | `ls`, `cd` (`..`, `/`, `~`), `pwd`, `cat`, `mkdir`, `touch`, `rm`, `mv`, `cp`, `tree`, `find`, `head`, `wc`, `grep` |
| **Writing** | `echo text`, `echo text > file`, `echo text >> file` (overwrite and append) |
| **System info** | `whoami` (shows admin status), `uname`, `uptime`, `neofetch`, `date`, `free`, `df` |
| **Processes** | `ps` (lists PIDs), `kill <pid>` |
| **Network** | `ping <host>`, `curl <url>` (first 2000 characters) |
| **Scripting** | `js <expression>` evaluates JavaScript |
| **OS control** | `open <app>`, `edit <file>` (opens the Editor or HTML Editor), `theme <color>`, `notify <text>`, `lock`, `clear`, `help` |

`open` with an unknown name lists every app ID.

---

## Productivity & tools

| App | What it does |
|---|---|
| 🧮 **Calculator** | Arithmetic, parentheses, `sin cos tan √ π e ^ %` and keyboard entry with `Enter` to evaluate. Input is validated against a strict character allow-list before evaluation. |
| ⏰ **Clock** | Four tabs: **World clocks** (UTC, New York, Chicago, Los Angeles, London, Paris, Dubai, Kolkata, Tokyo, Sydney), **Stopwatch** with laps, **Timer** (minutes and seconds) and **Alarms** (add, enable, delete). Alarms and timers fire a notification and a tone. |
| 📅 **Calendar** | Month view with navigation, per-day events, highlighted days that have events, and a notification of today's events when you sign in. |
| ✅ **To-Do** | Add, complete and delete tasks, with a remaining counter that also shows on the desktop widget. |
| 📝 **Sticky Notes** | Colored notes with a color picker per note, saved automatically as you type. |
| 🍅 **Pomodoro** | Focus timer with work and break cycles. |
| 📄 **Markdown** | Markdown editor with live preview. |
| 💰 **Expenses** | Expense tracker for logging and totaling spending. |
| 🔥 **Habit Streaks** | Habit tracker built around streaks. |
| 🧰 **Dev Tools** | Text and developer utilities: hashing (SHA), **UUID** generator, **password** generator, **timestamp** converter, **word counter**, UPPER/lower, **sort lines** and **reverse**. |
| 🔐 **Password Forge** | Strong password generator. |
| 🎨 **Color Studio** | Color picker and palette builder. |
| 📐 **Converter** | Length, mass, data, time, speed, volume and temperature (C/F/K). |
| 🌤️ **Weather** | City search or **📍 Here** (geolocation), current conditions (temperature, feels-like, humidity, wind) and a **7-day forecast**. Toggle °F and °C, which defaults by locale. Data comes from Open-Meteo. |
| 🔤 **Typing Test** | Measures WPM and accuracy and remembers your best. |

---

## Creative apps

| App | What it does |
|---|---|
| 🎨 **Paint** | Canvas drawing with a color picker, brush size, eraser, **undo** (15 steps), clear and **save to `Pictures/`** as PNG. Pointer and touch friendly. |
| 🎹 **Piano** | Playable piano, with keys mapped to your keyboard. |
| 🥁 **BStudio** | A 16-step drum machine with 4+ synthesized drum voices, BPM (60 to 200), swing, a random pattern generator, per-track mute and **WAV export**. |
| 🖼️ **BPhotos** | Gallery of every image in your file system, with an upload button, a generative art maker, a slideshow, rotate and zoom, and filters (mono, sepia, vivid, cool, invert). |
| 🎙️ **BRecorder** | Microphone recorder with a live waveform, playback and save. |

---

## Media suite (the "B" apps)

### 🎧 BTunes
A full music player.
- **Built-in library** of generated tracks, plus any audio files in your file system.
- **Playlists**, **favorites**, play history, shuffle and repeat, playback speed and master volume.
- **10-band equalizer** (31 Hz to 16 kHz) with presets: Flat, Bass Boost, Treble, Vocal, Rock, Electronic and Acoustic.
- **Sleep timer**.
- **Lyrics** (synced lyrics through LRCLIB, with a fallback to lyrics.ovh).
- **Discovery** through the iTunes Search and Apple Marketing Tools charts APIs, with YouTube-backed playback for streamed tracks.

### ▶️ BTube
A YouTube front end. Search or paste a link, set a **start time**, **loop**, and use the category chips and **watch history and favorites**. Playback uses `youtube-nocookie.com` embeds.

### 🎬 BVision
A local video player: open files or URLs, **subtitles**, **picture-in-picture**, snapshots, playback speed (0.5× to 2×), **A-B loop**, **frame stepping** and brightness and contrast sliders. Drag and drop works.

### 📻 BRadio
Internet radio with SomaFM stations: Groove Salad, Drone Zone, Deep Space One, Space Station Soma, Beat Blender, Fluid, Secret Agent, Lush, Indie Pop Rocks, Underground 80s, Boot Liquor and DEF CON Radio. It tries several mirror servers and an AAC fallback before giving up, supports favorites and custom stations, and has a volume control.

---

## Internet apps

### 🪐 Web Browser (Orbit)
A tabbed browser with an Orbit start page, built on an embedded proxy-style engine that loads in WebAssembly.
- **Tabs** with titles and load indicators.
- **Back, forward, reload, home** and an address and search bar.
- **Bookmarks, history and downloads** pages.
- **Find in page**, a **zoom** control and **DevTools** panels (network, console, storage, info, settings).
- Keyboard shortcuts: `Alt+←/→`, `Ctrl+R`, `Ctrl+L`, `Ctrl+F`, `F1` (shortcuts), `F12` (DevTools).
- The launcher's *"Search the web for …"* opens here.

### 🌀 Warp
A standalone **single-file, no-tabs web proxy browser** from BGS Labs, in the same family as the Orbit engine.

### ⚡ SURGE and 🎮 Netlii
Two **portal hubs** that each bundle games, a browser, a chat AI and movies behind one launcher.

### 🌤 BluSKY
An **ambient glassmorphism hub** with a solar-time sky and selectable weather (sunny, starry night, rain, thunder, snow, fog, cloudy), plus search, games, movies and a browser.

> Several of these are third-party or community pages that are embedded as sandboxed `iframe`s. Their content and behavior are theirs, and some features need a network connection.

---

## Games

### Classic arcade (built in)

| Game | Controls & features |
|---|---|
| 🧱 **Tetris** | Arrow keys to move and rotate, soft drop, **Space to hard-drop**, scoring |
| 🏓 **Pong** | `↑` / `↓` paddle duel |
| 🐍 **Snake** | Arrows or **WASD**, with score and best score |
| 🔢 **2048** | Arrow keys, with score and best score |
| 💣 **Minesweeper** | Click-to-reveal board |
| 🃏 **Memory Match** | Flip cards and match pairs |
| ♠️ **Solitaire** | Klondike |
| 🧱 **Breakout** | Paddle and bricks |

High scores for several games persist across sessions.

### Bundled game pages

| Game | Description |
|---|---|
| 🕹 **Globby Chase** | A chase game (an embedded packaged build) |
| 🏎 **F1 Random GP** | Race a **randomly generated F1-style circuit** over 3 laps. Customize the car's body and accent colors, with a minimap, speedometer, crash effects and lap timer. Arrows or WASD on desktop, and hold-to-gas with swipe steering on mobile. Made by Emmett. |
| 🃏 **Solatro** | A poker-hand roguelike with antes, rounds, jokers, consumables and cash |
| ⛏️ **Minecraft : EmOS** | An in-browser Minecraft-style client (Eaglercraft). It **downloads about 56 MB on first load** from a jsDelivr-hosted copy, so it needs the internet |

### Retro games

Anything you can load in the [Emulator](#the-emulator-38-systems).

---

## The Emulator (38 systems)

EmOS bundles a full **EmulatorJS** build, with 45 WebAssembly emulator cores stored in the file itself (about 88 MB of the file's size). **No ROMs are included.** You supply your own game files. Once EmOS has loaded, emulation works fully offline.

### How to play

1. Open **Emulator** from the launcher or Store.
2. **Drag and drop** a game file onto the window, or click to browse.
3. EmOS detects the system from the extension. For ambiguous files (`.iso`, `.bin`, `.cue`, `.zip`, `.7z` and so on) it asks which system the file belongs to, with a sensible guess preselected. You can also choose the system yourself first.

Controls: **arrows** to move, **Z / X** for A / B, **Enter** for Start, **V** for Select. Remapping is in the in-game Control Settings, and **gamepads work automatically**. Save states and in-game saves are stored per game in your browser. The in-game UI is themed to match your EmOS accent color, and the emulator menus are available in many languages.

### Supported systems

| Group | Systems |
|---|---|
| **Nintendo** | NES / Famicom · SNES / Super Famicom · Nintendo 64 · Game Boy / Color · Game Boy Advance · Nintendo DS · Virtual Boy |
| **Sony** | PlayStation |
| **Sega** | Genesis / Mega Drive · Master System · Game Gear · Sega CD / 32X · Saturn |
| **Atari** | 2600 · 5200 · 7800 · Lynx · Jaguar |
| **Other consoles** | PC Engine / TurboGrafx · PC-FX · Neo Geo Pocket · WonderSwan · ColecoVision · 3DO |
| **Computers** | Commodore 64 · Commodore 128 · VIC-20 · Plus/4 · Commodore PET · Amiga · ZX Spectrum · ZX81 · Amstrad CPC |
| **Arcade & other** | Arcade (FBNeo) · Arcade (MAME 2003 Plus) · Arcade (MAME 2003) · Capcom CPS-1 · Capcom CPS-2 · Doom (PrBoom) |

Archive extractors for `.zip`, `.7z` and `.rar` are included, so zipped games work directly.

> ⚖️ Only play games you own or have the right to use.

---

## The Store

A polished app store with a **Discover** page (featured hero carousel, *New & noteworthy*, *Most used*, *Games* and more), category browsing, search, an **Installed** tab and per-app detail pages with star ratings.

**Categories**: Games, Tools, Creative, Productivity, Media, Internet, Fun and Education.

- **Install and uninstall** non-system apps with one click. Installed apps show up in the launcher, and you can pin them to the desktop.
- Usage counts drive the *Most used* row.
- System apps (Files, Settings, Terminal and so on) are always present and cannot be removed.

---

## Bubber, the virtual pet

**Bubber** is the purple blob mascot, who is also the OS logo and the BubberBot avatar. As a pet:

- **Stats**: hunger 🍎, happiness 💜 and energy ⚡. They **change in real time, even while the OS is closed**, because Bubber remembers when you left.
- **Actions**: Feed (costs 5 🪙), Play, Nap, pat Bubber to earn coins and XP, and play the **Snack Catch** mini-game.
- **Levels** based on XP, with a **level-up** notification.
- **Mood** (happy, hungry, sad or sleepy).
- **Hat shop**: 🎩 🎀 🌸 🧢 😎 🎧 👑, bought with coins and worn on Bubber's head.

---

## Settings, personalization & accessibility

### Display
- **Interface scale**: 0.9, 1, 1.1, 1.25 or 1.5.
- **Brightness** (30 to 100%) and **Night Light** (a warm color overlay).
- Screen resolution readout and a fullscreen toggle.

### Sound
Master volume (applies to EmOS sounds, media and games), mute and a test tone.

### Personalization
- **Accent color**: any color, and the whole UI re-themes.
- **Wallpaper**: four built-in gradient wallpapers or a **custom image**, which is automatically downscaled to 1600 px and compressed.
- **Font**: System, Georgia, Monospace or Trebuchet MS.
- **Corner radius** and **animations** on or off.

### Power
- **Balanced** or **Power Saver** mode. Saver disables blur effects and animations to save battery.
- Battery level and charging state, where the device reports them.
- Auto-lock timer, **Keep screen awake**, Lock and Sleep.

### Accessibility
- **Screen reader**: reads aloud what you click and your notifications.
- **Magnifier**: enlarges the interface.
- **High contrast** mode.
- **Large text**.
- **Notification captions**: on-screen text for alerts.

---

## Data, storage & backups

- **Everything lives in your browser's `localStorage`**, namespaced under `emos`, `emos_fs`, `emos_bk`, `emos_img` and `emos_sess`.
- The browser limit is roughly **5 MB**, and Settings shows your usage. If storage fills up, EmOS warns you instead of failing silently.
- **Back up** with restore points (3 kept) and JSON export and import. Use these before clearing browser data.
- Data is **per browser and per origin**. A copy of the file opened from a different location or host starts fresh.
- **Factory reset** (Settings) or **Erase all data** (Security) wipes everything.
- Large uploaded files, such as videos and big images, can exhaust the 5 MB budget quickly.

---

## Architecture & file anatomy

EmOS is a "hand-written" single-page application.

```
EmmettOS.html  (~97 MB)
├── <style>                      ~10 KB of core CSS (themeable via CSS variables)
├── <div id="os"> …              Desktop, taskbar, launcher, panels, lock/boot screens
├── <script> core OS (~600 KB)
│   ├── State store (S) + persistence (sv / localStorage)
│   ├── Window manager (op, cl, fo, mx, mn, snap, tile, drag)
│   ├── Panels, launcher, context menu, notifications
│   ├── Auth (SHA-256 hashes, lockout) + session save/restore
│   ├── File system (nested JS object) + file operations
│   ├── Terminal command interpreter
│   ├── BubberBot (local intent parser + LLM bridge + action engine)
│   ├── APPS registry – every app is one object
│   ├── Store, Emulator host, and the B-suite media apps
├── Base64-embedded apps         SURGE, Netlii, Warp, Orbit engine, Globby Chase, BluSKY, Solatro, F1 GP
└── 129 × <script type="text/plain" data-emu="…">
    └── EmulatorJS runtime, loaders, cores (45 WASM), core configs, translations
```

### Why it is so big

About **88 MB** of the file is the emulator payload: 45 WebAssembly cores plus the EmulatorJS runtime, base64-encoded. The largest cores are FBNeo (about 10.9 MB), MAME 2003 Plus (about 7.1 MB) and MAME 2003 (about 6.7 MB). The OS code itself is well under 1 MB. The cores are stored as **inert `text/plain` script blocks**, so the browser does not execute them at load time. They are decoded only when a game starts.

### The app model

An app is a plain JavaScript object in the `APPS` registry:

```js
APPS.myapp = {
  n: 'My App',          // display name
  i: '🚀',              // icon (emoji or HTML)
  w: 520, h: 420,       // default window size
  sys: 1,               // optional: system app, cannot be uninstalled
  st: 1,                // optional: stateful, so the window is not re-rendered globally
  r: () => '<p>Hello</p>',          // simple apps: return HTML, re-rendered on state changes
  // or
  init: (body, win) => { … }        // stateful apps: build the DOM once, own the window
};
```

Adding it to the Store catalog is one line in the `SXM` metadata map (`id: [category, tagline]`). Embedded third-party pages are loaded as base64 strings and mounted in a full-window `iframe` using `srcdoc`.

### Emulator isolation

Games run in a **sandboxed `iframe`**. The parent hands it a job record (file, core and theme color), and the iframe decodes only the data it needs. Cores come from `blob:` URLs that are created lazily and cached.

---

## Network access & privacy

EmOS keeps your files, settings and accounts **on your device**. There is no EmOS server and no telemetry. Some apps reach out to third-party services when you use them:

| Feature | Service contacted |
|---|---|
| BubberBot (free mode) | `text.pollinations.ai` |
| BubberBot (optional) | `api.anthropic.com` or `api.openai.com`, or your configured URL |
| Weather | Open-Meteo (weather and geocoding) |
| Public IP lookup | `api.ipify.org` |
| Latency test | `www.gstatic.com` |
| BTunes metadata and lyrics | iTunes Search, Apple Marketing Tools, LRCLIB, lyrics.ovh |
| BTube | YouTube (`youtube-nocookie.com`, thumbnails) |
| BRadio | SomaFM |
| Minecraft : EmOS | jsDelivr (about 56 MB, on first launch) |
| Browser, Warp, SURGE, Netlii, BluSKY | Whatever sites you browse or services they use |

Everything else, including the desktop, file system, terminal, editor, calculator, all the classic games, the emulator, Paint, notes, calendar and so on, works with **no connection at all**.

Camera, microphone, location, clipboard-read, notifications and Bluetooth are **only requested when you use a feature that needs them**, and you can review and request them in the Security app.

---

## Browser compatibility

| Browser | Support |
|---|---|
| Chrome / Edge / Brave (desktop) | ✅ Best experience, including Web Bluetooth, battery status and memory stats |
| Firefox | ✅ Works. Web Bluetooth, battery status and JS heap stats are unavailable |
| Safari (macOS / iOS) | ✅ Core OS works. Some device APIs are limited, and mobile Safari has stricter storage limits |
| Mobile browsers | ✅ Touch dragging is supported, and the layout respects safe-area insets. The desktop is best on larger screens |

The **Logs → Run diagnostics** button tells you exactly which features your browser supports. WebAssembly and WebGL are needed for the emulator.

---

## Known limitations

- **File size**: about 97 MB. Initial load takes a few seconds and uses real memory.
- **~5 MB storage ceiling** for user data, because of `localStorage`. This is a browser limit, not an EmOS one.
- **Single-file, single-origin data**: data does not sync between browsers or devices unless you export and import a backup.
- **Not a real OS or a security boundary**: accounts and the lock screen are a convenience and a demo. Anyone with access to the browser profile or the page's source can bypass them.
- **No ROMs or BIOS files are included.** Some systems (PlayStation, Saturn, Sega CD, 3DO, PC-FX and others) generally need BIOS files to run many games.
- **Minecraft** needs internet access on first load.
- **Embedded third-party apps** (SURGE, Netlii, Warp and others) depend on external services, and their behavior is outside EmOS's control.
- Some features depend on browser support: screen capture, Web Bluetooth, the Battery API and speech recognition.

---

## Security notes for the author

A few things worth knowing before you publish or share this file:

1. **Seed credentials are in the source.** The default `users` list in the file includes starter passwords for the Emmett and Ryan accounts. Those are hashed on first run, but the original strings remain in the HTML for anyone who views the source. Anyone who gets the file can read them. If this is hosted publicly, **change those passwords in Settings**, and consider replacing the seeded passwords with empty ones or removing the accounts from the default data before distributing.
2. **Client-side auth is not real security.** The lock screen protects against casual use only.
3. **API keys** entered in BubberBot are saved in `localStorage` in plain text. Use a restricted or low-limit key, and don't enter one on a shared computer.
4. **Embedded third-party code** (portals, proxy browsers, the Minecraft client) runs inside the page's origin context through `srcdoc` iframes. Only ship content whose licenses allow redistribution, and review it before hosting publicly.
5. **Emulator and games**: bundling emulator cores is generally fine under their open-source licenses, but make sure you comply with each core's license terms (several are GPL or non-commercial). Do not distribute copyrighted ROMs or BIOS files with the OS.

---

## Hacking on EmOS

Since there is no build step, you can edit the file directly in any text editor. Be careful: the file is large, so use an editor that handles big files, and avoid touching the base64 blocks.

Handy things to know:

- **Global state** is the object `S`, and **defaults** are the object `D`. Any new setting should be added to both and saved through `set(key, value)`.
- `ntf(title, body)` shows a notification. `op(id)` opens an app. `sv()` persists state. `R()` re-renders all simple apps.
- The virtual file system is `S.fs`, a nested object where **folders are objects and files are strings**. Images and audio are stored as data URLs.
- Helpers: `gt(path)`, `mk(path)`, `esc(html)`, `pick(accept, cb)` for file selection, `dl(name, data, type)` for downloads.
- To add a BubberBot action, add an entry to the `BBA` object, then list it in the system prompt in `bbSys()`.
- To add a terminal command, add a function to the `C` object inside `cmd()`.
- To add an emulator system, add a row to the `EMS` table (`[group, label, core, [extensions]]`), provided the matching core is bundled.

---

## Credits

- **EmOS** by **Emmett** (OS, apps, F1 Random GP, BubberBot and the Bubber mascot).
- **Emulation**: [EmulatorJS](https://emulatorjs.org) and the RetroArch/libretro cores.
- **Radio**: [SomaFM](https://somafm.com).
- **Weather data**: [Open-Meteo](https://open-meteo.com).
- **Lyrics and metadata**: LRCLIB, lyrics.ovh, Apple iTunes Search.
- **Warp** by BGS Labs; **Minecraft client** hosted from the `Bubbergamesites-EmOS` repository and based on Eaglercraft.
- Other embedded pages (SURGE, Netlii, BluSKY, Solatro, Globby Chase) belong to their respective creators.

All trademarks (Minecraft, Nintendo, PlayStation, YouTube and others) belong to their owners. EmOS is an independent project and is not affiliated with or endorsed by them.

---

<p align="center"><i>Made with 💜 — one file, one whole operating system.</i></p>
