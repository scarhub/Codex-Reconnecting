# Codex & OpenClaw Proxy Installer for Windows

[简体中文](README.zh-CN.md) | **English**

Beginner-friendly Windows migration packages for configuring a local HTTP proxy independently for Codex GUI and OpenClaw Gateway.

> **Package availability:** At the time this README was prepared, the GitHub default branch contained no installer folders and the Releases page had no package archive. This README is documentation, not an installer. Do not expect to install by cloning this repository until the maintainer publishes the package folders or a Release archive containing them.

## What this project does

Some Windows applications do not automatically use the local proxy configured in a proxy app or in Windows. This project provides two independent ways to pass a local HTTP proxy address to the relevant processes:

| Package | Intended use | How proxy settings are applied |
|---|---|---|
| **Codex Proxy Migration** | Codex GUI | Stores HTTP_PROXY, HTTPS_PROXY, and NO_PROXY as persistent current-user environment variables. Fully restart Codex GUI for it to inherit changes. |
| **OpenClaw Proxy Migration** | OpenClaw Gateway and its child processes, including Codex processes started by OpenClaw | Starts Gateway through a wrapper that sets proxy variables only for that Gateway process tree. |

You can install either package or both. They do not depend on each other: stopping the Codex user-level proxy does not remove the proxy settings supplied to OpenClaw by its dedicated launcher.

### What this project does not do

- It does not include or install a proxy application. You must provide and start your own local HTTP proxy.
- It does not change Windows Settings → Network & Internet → Proxy, WinHTTP, or system-wide environment variables.
- It does not install, sign in to, or modify Codex or OpenClaw.
- It does not change Codex/OpenClaw source code, provider settings, or WebSocket transport.
- A working local proxy does not guarantee that every network or service issue will disappear.

## Before you begin

1. Use a Windows account that can run the applications and scheduled task involved.
2. Install and sign in to the applications you intend to use:
   - For the Codex package, install and sign in to Codex GUI.
   - For the OpenClaw package, install and sign in to OpenClaw first. Its gateway.cmd launcher must exist.
3. Install your own proxy application, configure a local HTTP proxy listener, and start it before running an installer. The address is commonly 127.0.0.1 and the port depends on your proxy application; do not assume the example port is correct.
4. Once a Release archive is available, download it from the repository Releases page and extract it to a permanent folder, for example C:\Tools\Codex_OpenClaw_Proxy_Installer. Do not run an installer from inside a ZIP. Do not move or rename the extracted folder after installation: PATH entries or the scheduled-task action may refer to that location.
5. Do not copy another computer's generated config.ps1, install_state.json, Backup, or Logs into a new deployment. Those files contain machine-specific configuration, rollback state, or diagnostics.

The proxy application, Codex, OpenClaw, and their sign-in credentials are not included. Sign in on each computer yourself; do not copy authentication directories or tokens from another computer.

## Choose and install a package

### A. Codex GUI Proxy Migration

Use this package when Codex GUI needs to inherit your local proxy.

1. Open the extracted Codex_Proxy_Migration folder.
2. Double-click install_codex.bat.
3. Follow the prompts:
   - If one proxy is detected, confirm whether to use it.
   - If several candidates are detected, choose the correct address and port.
   - If none is detected, enter the local proxy host and port shown by your proxy application.
4. The installer checks that the selected port is listening. If it is not, start the proxy application or correct the address; do not continue with a guessed port.
5. The installer creates a machine-specific config.ps1, adds the package's CodexGUI folder to your user PATH, and asks whether to enable user-level proxy variables now.
6. Open a **new** Command Prompt and run:

~~~cmd
proxy status
~~~

If you chose not to enable the variables during installation, run proxy start when you are ready.

Commands:

~~~cmd
proxy status
proxy start
proxy stop
~~~

- proxy start checks the local proxy, then sets the current user's HTTP_PROXY, HTTPS_PROXY, and NO_PROXY.
- proxy stop removes those three current-user variables.
- proxy status displays their persistent user-level values and a read-only view of Windows proxy settings.

These are user-level environment variables, not the Windows system proxy. They persist until stopped or removed. Codex GUI processes that are already running do not receive a new environment: after proxy start **or** proxy stop, completely exit Codex GUI and its background processes, then reopen it.

If proxy start detects a new port automatically, it may update the user variables while config.ps1 still contains the previous port. If proxy status reports that the configured listener is unavailable, compare the displayed HTTP_PROXY/HTTPS_PROXY values with your proxy app. If needed, update ProxyHost and ProxyPort in this package's config.ps1 to the actual listening address.

### B. OpenClaw Proxy Migration

Use this package when OpenClaw Gateway and its child processes need the local proxy.

1. Open the extracted OpenClaw_Proxy_Migration folder.
2. Double-click install_openclaw.bat.
3. Follow the prompts:
   - Confirm a detected proxy or choose from the candidates.
   - If no proxy is detected, enter the host and port shown by your proxy application.
   - Confirm the Gateway port and the path to gateway.cmd. The default path is under your user profile's .openclaw folder; change it only if your installation uses a different location.
4. The installer verifies the proxy port and gateway.cmd, then creates this computer's config.ps1.
5. If an existing Task Scheduler task named **OpenClaw Gateway** is found, the package changes its Action to call the package's PowerShell launcher and backs up the original task XML. It does not create a task if none exists.
6. If asked, allow the installer to start Gateway. Otherwise, use the existing OpenClaw Gateway task in Task Scheduler. If no task was updated, start it manually by running OpenClaw\start_openclaw_gateway.ps1 from the extracted package folder.

The launcher checks the proxy before starting Gateway. It sets HTTP_PROXY, HTTPS_PROXY, and NO_PROXY for the Gateway process and its children only; it does not write these variables to the Windows user or system environment. Keep the package folder at the same path while the scheduled task uses it.

For a basic check, run OpenClaw\test_openclaw_proxy.ps1 from the package folder. Startup logs are written under Logs. A successful local port check confirms only that something is listening at that address; it does not by itself prove that a remote service is reachable.

## Changing the proxy port

The port is controlled by your proxy application. Start that application first and confirm its current local HTTP address.

- **Codex:** run proxy start. The command can try to discover a replacement port when the configured one is unavailable. If discovery is ambiguous or does not find your proxy, update ProxyHost and ProxyPort in Codex_Proxy_Migration\config.ps1, then run proxy start again. Fully restart Codex GUI afterward.
- **OpenClaw:** the Gateway launcher can use a uniquely identified proxy from available environment/system-proxy information. If it cannot confidently identify one, it stops rather than guessing. Update ProxyHost and ProxyPort in OpenClaw_Proxy_Migration\config.ps1 to the address shown by your proxy application, then start Gateway again.
- Do not use setx, change system-wide variables, or change Windows system proxy settings just to configure these packages.

## Uninstalling

Keep the package folder in place until its uninstaller finishes. Each package has its own uninstaller and can be removed independently.

### Codex package

1. If you ran proxy start manually after installation, note that the uninstaller only restores the saved prior environment when the installer itself enabled the proxy. If you enabled it later by hand, run proxy stop first; this removes the three variables and does not restore any custom values that existed before installation. Record and restore those values yourself if needed.
2. Double-click uninstall_codex.bat.
3. Type YES when asked to confirm.
4. Completely exit and reopen Codex GUI after removal.

When the installer enabled the proxy, the uninstaller uses its installation record to restore the saved prior user environment. It also removes only the user PATH entry added by this package. If proxy start was run manually after declining proxy activation during installation, first run proxy stop; the uninstaller does not automatically restore the earlier environment in that case. It does not uninstall Codex GUI or delete authentication files.

### OpenClaw package

1. Double-click uninstall_openclaw.bat.
2. Type YES when asked.
3. If prompted about package-generated logs, choose whether to delete or keep them.

The uninstaller attempts to restore the backed-up OpenClaw Gateway task action and prior package configuration. It keeps the Backup folder and does not uninstall OpenClaw or delete authentication. If there is no valid installation record or rollback backup, the script may refuse to make changes rather than guess.

## Troubleshooting

- **Proxy unavailable:** Start your proxy app and confirm its local HTTP host and port. Do not use a SOCKS-only port unless your application explicitly exposes it as an HTTP proxy.
- **No proxy detected:** Enter the exact address shown by your proxy app. Common ports vary by application and configuration.
- **Codex GUI still reconnects:** Check proxy status, fully exit every Codex GUI/background process, and reopen it. The package cannot fix unrelated server, account, or network failures.
- **OpenClaw does not start:** Confirm gateway.cmd exists at the configured path, the proxy is running, and the OpenClaw Gateway task points to the current package folder. Review Logs\openclaw-gateway-proxy.log.
- **Port change not picked up:** Update the relevant package's config.ps1 as described above. Do not change the other package; the two proxy controls are independent.
- **Scheduled task rollback fails:** Keep the Backup folder intact. Do not delete the package before rollback completes.

## Package contents and safe handling

A published Release should provide two folders: Codex_Proxy_Migration and OpenClaw_Proxy_Migration. Each includes a double-click installer and uninstaller, scripts, and an example configuration. The installer generates machine-specific files such as config.ps1 and install_state.json; task backups and logs may also be generated during use.

Do not publish or share generated config.ps1, install_state.json, task XML backups, or diagnostic logs unless you have reviewed and intentionally sanitized them. They are not substitutes for credentials, but can disclose local paths and machine configuration.

## License and responsibility

Review the license and release notes included with each published package. Choose a proxy application you trust and review scripts before running them. This project does not collect credentials and does not guarantee uninterrupted access to any service.
