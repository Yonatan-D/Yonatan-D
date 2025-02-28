# 解决每次重启需要重置winsock

win11 wsl 某次更新系统后，每次重启都要在 cmd 用管理员权限运行 `netsh winsock reset` ，否则会报错：

```bash
The stub received bad data.
Error code: Wsl/Service/0x800706f7
Press any key to continue...
```

解决办法：

```powershell
taskkill -IM "wslservice.exe" /F
NoLsp.exe "C:\Program Files\WSL\wsl.exe"
NoLsp.exe "C:\Program Files\WSL\wslservice.exe"
NoLsp.exe "C:\Program Files\WindowsApps\MicrosoftCorporationII.WindowsSubsystemForLinux_2.1.5.0_x64__8wekyb3d8bbwe\wsl.exe"
```

> 参考：https://github.com/microsoft/WSL/issues/4177#issuecomment-1429113508

