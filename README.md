# pi-notifier

Local fork of `@steven-rothwell/pi-notifier`.

Windows desktop notifications for Pi. Get notified when the agent finishes, when API errors occur, or when tools fail.

## What changed vs upstream

- **HiDPI / DPI-aware rendering.** The upstream PowerShell toast is not DPI-aware, so on high-DPI (scaling >100%) displays the popup is bitmap-stretched and looks blurry. This fork calls `SetProcessDPIAware()` before creating the WinForms `Form`, so the toast renders at native DPI — crisp text and rounded corners.
  - Change is one line in `notifyWindows()`: `[void][NotifWin32]::SetProcessDPIAware()` added before `$f = New-Object System.Windows.Forms.Form`, plus the matching `[DllImport("User32.dll")]` on the `NotifWin32` type.
  - Uses `SetProcessDPIAware` (system-DPI-aware). Sufficient for a fixed 5s toast on a single high-DPI display; per-monitor v2 would help only if you move the toast across mixed-DPI monitors.

Everything else is unchanged from upstream v1.0.3.

## Features

- **Agent Finished** - Know when Pi is ready for your next input
- **Provider Error** - Instant alerts for HTTP 4xx/5xx errors (rate limits, auth failures, etc.)
- **Tool Error** - Get notified when any tool execution fails
- **Focus Detection** - Notifications are suppressed when Pi's terminal is the foreground window
- **Custom Sounds** - Choose from 13 Windows system sounds or disable sounds per notification type
- **Dark/Light Theme** - Stylish toast popups with a modern look

## Install (local path)

```bash
pi install ./packages/pi-notifier
```

Or load without installing, for a single session:

```bash
pi -e ./packages/pi-notifier/notifier.ts
```

Restart Pi after installing.

## Configure

Run `/notifier` in Pi's TUI to toggle per type, change sounds, switch theme, and test.

Settings are stored in `~/.pi/notifier.json`:

```json
{
  "theme": "dark",
  "notifications": {
    "agentFinished": { "enabled": true, "sound": "C:\\Windows\\Media\\Windows Notify Messaging.wav" },
    "providerError": { "enabled": true, "sound": "C:\\Windows\\Media\\Windows Error.wav" },
    "toolError":     { "enabled": true, "sound": "C:\\Windows\\Media\\Windows Exclamation.wav" }
  }
}
```

## Requirements

- **Windows** - Uses PowerShell and WinForms for native notifications
- **Pi** - Requires `@earendil-works/pi-coding-agent` and `@earendil-works/pi-tui`

## How It Works

Hooks three Pi events:

1. `agent_end` - agent finished, waiting for input
2. `after_provider_response` - HTTP 4xx/5xx from AI providers
3. `tool_execution_end` - tool execution failed (`isError: true`)

Before showing, checks if Pi's terminal is the foreground window; if so, the notification is skipped.

## License

MIT — upstream © steven-rothwell.
