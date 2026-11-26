# 03: Closing the VNC server the client pack left open

*2026-11-26*

The EVE-NG Client Pack installs UltraVNC so the web UI can open node consoles. EVE-NG only needs the viewer, so I installed that alone. Afterwards I checked what was listening on the laptop anyway.

## Finding it

```
Get-NetTCPConnection -LocalPort 5900 -State Listen -ErrorAction SilentlyContinue
LocalAddress  LocalPort  State
0.0.0.0       5900       Listen

Get-Process -Id (Get-NetTCPConnection -LocalPort 5900 -State Listen).OwningProcess | Select-Object Id, ProcessName, Path
27844  winvnc  C:\Program Files\uvnc bvba\UltraVNC\winvnc.exe
```

The UltraVNC server was running anyway, listening on every interface.

## Shutting it down

Stopped it, then checked for anything that could bring it back:

```
Stop-Process -Name winvnc -Force

Get-CimInstance Win32_StartupCommand | Where-Object Command -like '*winvnc*'
(nothing)

Get-NetFirewallRule | Where-Object DisplayName -like '*vnc*' | Select-Object DisplayName, Direction, Action, Enabled
winvnc.exe     Inbound  Allow  True
winvnc.exe     Inbound  Allow  True
vncviewer.exe  Inbound  Allow  True
vncviewer.exe  Inbound  Allow  True
```

No startup entry, but four inbound allow rules. The viewer only connects out, so none of them are needed. I renamed the server so it can't run but the viewer stays, and disabled the rules:

```
Rename-Item "C:\Program Files\uvnc bvba\UltraVNC\winvnc.exe" "winvnc.exe.disabled"
Get-NetFirewallRule | Where-Object DisplayName -like '*vnc*' | Disable-NetFirewallRule
```

## Result

```
Get-NetTCPConnection -LocalPort 5900 -State Listen -ErrorAction SilentlyContinue
(nothing)

Test-Path "C:\Program Files\uvnc bvba\UltraVNC\vncviewer.exe"
True
```

Nothing listening on 5900, the viewer still works, and the four rules are disabled.
