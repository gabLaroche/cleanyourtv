# Chromecast with Google TV 4K (sabrina), Google TV 14, UTTC.250917.004

Cleaned 2026-10-25, debloat only: the stock Google TV home, phone casting, Assistant and YouTube Music were kept.
Backup taken before any change: `backup/20261025-110257/`. Undo records: `~/.cleanyourtv/recovery-<tv ip>-*.json`.

## What was disabled (19 packages, nothing uninstalled)

- `1-telemetry.txt`: Google feedback, remote logging, netoscope, federated compute, on-device personalisation, ad services.
- `2-preloaded.txt`: Play Games, Google TV / Play Movies, print spooler, TalkBack.
- `3-streaming.txt`: Disney+, Apple TV, Tou.tv, CBC Gem, Paramount+, Discovery+, Crunchyroll.

All 19 were still disabled after a reboot. Private DNS (AdGuard) broke all name resolution here, probably
because it conflicts with the Tailscale VPN, so it was reverted. Set up ad blocking on the router instead.

## Undo

Turn on Wireless debugging and connect (`adb connect <tailscale ip>:<port>`, then `adb tcpip 5555`), then:

    TV_HOST=<tv ip> tools/restore.sh work/3-streaming.txt
    TV_HOST=<tv ip> tools/restore.sh work/2-preloaded.txt
    TV_HOST=<tv ip> tools/restore.sh work/1-telemetry.txt

To bring back one app only: `tools/tv.py pm enable <package>`, e.g. `com.disney.disneyplus`.
Or from the TV itself: Settings > Apps > See all apps > Show system apps > the app > Enable.

## Launcher (Home redirect)

Installed `com.gathin.tvhome` with launcherx and setupwraith left enabled, so casting and the Assistant keep working.
`launcher/app/src/main/res/values/config.xml`: dock YouTube, Netflix, Prime Video, Crave; `live_tv_package` empty;
`stock_home_redirect` `com.google.android.apps.tv.launcherx`; `app_keys` `190:com.google.android.youtube.tv`,
`191:com.netflix.ninja`. Built and installed with
`STOCK_LAUNCHER=none HOME_KEY_SERVICE=1 launcher/build-and-install.sh`. The script stops at its Home check
(expected, launcherx stays Home), so HomeKeyService was enabled afterwards the way the script does it.
On the real remote, Home, YouTube, Netflix, mic, holding Home and holding Volume Up all work. After a reboot the
TV came up on the new home.

To change the dock order: edit `dock_apps` in config.xml and run the same install command again (it re-enables
HomeKeyService only if the script gets that far, so enable it again by hand as before).
To undo: uninstall Clean Your TV in Settings > Apps; the Google home takes over again.

## Notes

- AdGuard Private DNS: port 853 is reachable on this Wi-Fi, so the failure points at Tailscale's DNS
  (100.100.100.100). With Tailscale disconnected it should work. Untested: turning off "Use Tailscale DNS" in the
  Tailscale app, which might allow both.

- After a reboot, the Tailscale app had to be opened by hand before the TV was reachable again.
- Port 5555 from `adb tcpip` and Wireless debugging both turn off at every reboot.
