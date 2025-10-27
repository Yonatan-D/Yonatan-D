# 解决每次重启需要重置winsock

win11 wsl 某次更新系统后，每次重启都要在 cmd 用管理员权限运行 `netsh winsock reset` ，否则会报错：

```bash
参考的对象类型不支持尝试的操作。
Error code: Wsl/Service/0x8007273d
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

