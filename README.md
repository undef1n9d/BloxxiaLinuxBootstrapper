# Bloxxia Linux Bootstrapper

Bloxxia launcher for Linux. Installs the Windows clients (2017/2018/2020/2021) into Wine and launches them from `bloxxiaclient://` links on the site. No anticheat - install and launch only.

## Requirements

- 64-bit Linux (tested on CachyOS/Arch)
- Wine. Easiest is staging + winetricks:
  - Arch/CachyOS: `sudo pacman -S wine-staging winetricks`
  - Ubuntu/Debian: `sudo apt install wine64 winetricks`
  - Fedora: `sudo dnf install wine winetricks`
- Or Proton - then you don't need Wine at all, the bootstrapper downloads Proton-GE itself (`runner=proton`, ~500 MB first time)

## Install

Grab `bloxxia-bootstrapper` from the releases page, make it executable and run it:

```
chmod +x bloxxia-bootstrapper
./bloxxia-bootstrapper
```

On first run it copies itself to `~/.bloxxia/`, registers the `bloxxiaclient://` protocol, downloads the clients, and updates itself from then on. Uninstaller (`bloxxia-uninstaller`) is next to it in releases.

## Usage

Just hit Play on the site - a link like `bloxxiaclient://join?place=...&ticket=...&2020=true` opens in the bootstrapper. Without a link it only checks for updates and exits.

```
bloxxia-bootstrapper [bloxxiaclient://...] [flags]
  -u, --uninstall   remove everything: clients, prefix, desktop entries
  --repair          rebuild the prefix, re-download broken clients, then launch
  --runner wine|proton   runner override for one run
```

## Config

File `~/.bloxxia/bootstrapper.conf`, `key=value`, everything optional:

| Key | Default | What it does |
|---|---|---|
| `runner` | `wine` | `proton` - run clients through Proton-GE instead of system Wine |
| `proton_version` | empty | pinned Proton-GE version (`GE-Proton11-7`), empty - latest |
| `wine_version` | empty | Kron4ek Wine build (`11.0`, `latest`), empty - system Wine |
| `daemon` | `1` | `0` - stay attached in the terminal while the game runs |
| `fps_unlock` | `0` | `1` - uncap FPS (in-memory patch, 999 fps) |
| `proton_stealth` | `1` | Proton masking (no ntsync, 60 fps cap). `0` - off |
| `proton_fps_cap` | `60` | FPS cap on Proton via DXVK. `0` - uncapped |
| `mouse_warp_override` | empty | `force` / `disable` - Wine cursor behavior, empty - stock |
| `mouse_warp_force` | `1` | `0` - do not apply MouseWarpOverride unless set above. Only relevant on X11 |
| `update_check_interval` | `0` | Hours to trust the last client version check. `0` - check every launch |
| `discord_rpc` | `1` | `0` - disable Discord Rich Presence |
| `legacy_rpc` | `0` | `1` - old status format |
| `show_account_rpc` | `1` | `0` - no avatar/nick in status |
| `show_profile_button` | `1` | `0` - no View Profile button |
| `show_game_button` | `1` | `0` - no View Game button |
| `wine_log` | `0` | `1` - show full Wine output (attached mode only) |
| `debug_logs` | `0` | `1` - extra debug lines |
| `log` | `1` | `0` - don't write `~/.bloxxia/bootstrapper.log` |

Env vars: `BLOXXIA_RPC_URL` + `BLOXXIA_RPC_AUTH` (game info for Discord), `BLOXXIA_DISCORD_APPID` (custom Discord App ID), `DXVK_HUD` / `DXVK_LOG_LEVEL` (passed into the game).

## Known issues

- **Wayland + mouse.** On Wayland the client runs through XWayland, where mouse capture is buggy: the cursor freezes or teleports when you drag the camera. Confirmed to be an XWayland limitation, not Wine or the client. Log into Plasma (X11) and it is fine. No Wine setting works around this, Gamescope included.
- **Proton kicks.** Some servers kick Proton clients (`unexpected client behavior`), likely over packet timing. Doesn't happen with `runner=wine`. `proton_stealth` masks it partially, no guarantees.
- **F11 under Proton on Wayland** can hang the client. Play windowed.
- First launch is slow: Wine prefix, shaders, client downloads. Fast after that.

## Layout

Clients live in `~/.bloxxia/Versions/<version>/<year>/`, Wine prefix is `~/.wine-bloxxia`, Proton stuff in `~/.bloxxia-proton`. Client versions are checked against the CDN every launch and re-downloaded on mismatch. The bootstrapper updates itself through GitHub Releases. HWID is hardware-stable and sent to the server like the original.
