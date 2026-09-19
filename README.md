# AMP Templates — Alchemy Factory

Custom [CubeCoders AMP](https://cubecoders.com/AMP) application template for the
**Alchemy Factory** dedicated server.

- **Game:** [Alchemy Factory](https://store.steampowered.com/app/3669570/) (Steam app 3669570)
- **Dedicated server app:** `4550060` (downloadable anonymously via SteamCMD)
- **Server type:** Unreal Engine 5 (Windows-only binary → runs under **Proton GE** on Linux)
- **Tested on:** AMP 2.8.0.4 (Linux, container mode `cubecoders/ampbase:debian`)

## Files

| File | Purpose |
| --- | --- |
| `alchemy-factory.kvp` | Main AMP application config |
| `alchemy-factoryconfig.json` | Settings shown in the AMP config UI |
| `alchemy-factorymetaconfig.json` | Maps `ServerConfig.ini` keys to AMP settings |
| `alchemy-factoryports.json` | Game port (9877) + query port (9878), UDP |
| `alchemy-factoryupdates.json` | Install/update stages (SteamCMD + Proton GE) |
| `manifest.json` | Template source manifest |

## Install the template in AMP

1. Open the AMP ADS (Application Deployment Server) panel.
2. **Settings → Configuration Repositories** and add:
   ```
   horve000/amp-templates:main
   ```
   (The ADS fetches this git repo; "Fetch Latest" or the next auto-update pulls it in.)
3. In the ADS, **Create Instance** → pick **Alchemy Factory** → configure → deploy.

Alternatively, if you self-host the repo, edit
`ADS.ConfigurationRepositories` in the ADS instance's `ADSModule.kvp` and restart the ADS.

## Create an instance

- Choose the **Alchemy Factory** template.
- The deploy runs three update stages:
  1. **SteamCMD** — downloads the dedicated server app `4550060`.
  2. **ServerConfig.ini** — creates a default config if missing.
  3. **Proton GE** — downloads `GE-Proton9-27` (Linux only; required because the server is a Windows UE5 binary).

## Configuration

Edit settings in the AMP instance's **Config** tab (they write to `ServerConfig.ini`):

| Setting | Default | Notes |
| --- | --- | --- |
| Server Name | `AMP Powered Alchemy Factory Server` | Shown in the server list |
| Server Public | `1` | Visible in the Steam server browser |
| Steam Relay | `1` | Routes traffic via Steam. `1` = join via **join code**; `0` = direct IP join (needs public IP + UDP 9877/9878 open) |
| LAN Mode | `0` | Only visible on the local network |
| Server Password | *(empty)* | Password players need to join |
| Admin Password | *(empty)* | Become admin in-game with `/admin <password>` in chat |
| Max Clients / Max Admins | `10` / `1` | |
| Autosave Mode | `1` | 0=on sleep, 1=every 5 min, 2=every 10 min, 3=never |
| Pause When Empty | `1` | |
| Session Retry Seconds | `10` | Retry delay for Steam session creation |

**Ports:** Game Port `9877` UDP, Query Port `9878` UDP (used for direct IP joins; in relay mode only the query port binds locally).

**Save location:** `./AlchemyFactory/Saved/SaveGames`

Config changes take effect after a server restart.

## Known quirks (read before using)

- **Proton version is pinned to `GE-Proton9-27`.** The latest GE-Proton (11.x) hangs the server
  during startup (CPU drops to ~0.5%, log stalls after plugin mounting). 9-27 boots reliably.
  You can override the version via the *Proton GE Release Version (Linux)* setting, but expect
  breakage on 11.x.
- **`ApplicationReadyMode` is `Immediate`.** AMP cannot capture the game's stdout through
  `proton runinprefix`, so regex-based ready detection never matches. The instance is marked
  Running as soon as the process is up (the game takes ~30 s to create its session).
- **Image source** uses the hashed Steam CDN URL because `steam:3669570` resolves to a `header.jpg`
  that does not exist for this app (Steam moved to hashed asset paths).
- **Relay mode by default:** players join with a **join code** shown in the server console/log
  (`join code: XXXXX`). For direct IP joining, set *Steam Relay* to `0` and open UDP 9877/9878.
- The dedicated server app is still marked **experimental** by the developer; expect occasional
  instability in early versions.
