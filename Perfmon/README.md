# PerfMon: BlackBox performance collection

[Configure-BlackBox.ps1](Configure-BlackBox.ps1) configures the BlackBox Performance Monitor
collector for English-language Windows 11. Run the following commands from the repository root in an elevated **Windows
PowerShell 5.1** session:

```powershell
.\Perfmon\Configure-BlackBox.ps1 -Action Install
```

If your shell is already in the `Perfmon` directory, use
`.\Configure-BlackBox.ps1` instead of `.\Perfmon\Configure-BlackBox.ps1`.
Use `powershell.exe`, not PowerShell 7 (`pwsh.exe`).

The collector samples every 15 seconds and registers the `Start BlackBox`
startup task, which runs as SYSTEM. Logs are
stored under `%systemdrive%\PerfLogs\Admin`, with a 300 MB limit per segment;
this is not a total storage or retention limit.

## Counter groups

Both groups are defined near the top of the script and are always requested:

- `$systemCounters`: the existing 16 system and process counter paths.
- `$claudeCounters`: five additional paths useful for Claude Desktop and local
  Cowork workloads:

| Counter path | Purpose |
| --- | --- |
| `\Hyper-V Hypervisor Logical Processor(_Total)\% Total Run Time` | Host CPU utilization including hypervisor activity |
| `\Hyper-V Hypervisor Virtual Processor(*)\% Guest Run Time` | Guest CPU utilization per virtual processor |
| `\Hyper-V Virtual Storage Device(*)\Latency` | Virtual storage latency |
| `\Hyper-V Virtual Switch(*)\Dropped Packets Incoming/sec` | Incoming virtual-switch packet drops |
| `\Hyper-V Virtual Switch(*)\Dropped Packets Outgoing/sec` | Outgoing virtual-switch packet drops |

The script combines and deduplicates both groups before checking availability.
Unsupported paths are skipped with warnings. Wildcard paths remain eligible
when their counter exists but no instances are currently active.

These Hyper-V counters include other virtualized workloads. Inspect individual
instances to identify Cowork: observed names include `cowork-vm-...` virtual
processors and storage paths containing Claude VHDX files. Names can vary;
do not hard-code a particular machine's instance names. Switch drops alone do
not establish a Claude-specific network failure. Existing `Process(*)`
counters already capture Claude processes; use `ID Process` to identify PIDs.

## Adding counters and reviewing logs

Add each new path as a quoted entry in the appropriate array, then rerun
`-Action Install`. This applies the list and briefly restarts collection.
No separate configuration file is needed.

Open `perfmon.exe` and inspect **Data Collector Sets > User Defined > BlackBox**
to review its configured counters. In **Performance Monitor**, open Properties,
select **Source > Log files**, and add a collected performance log. Select the
desired counters and instances on the **Data** tab. To validate Cowork coverage,
review a recording spanning Cowork startup, activity, shutdown, and restart.

## Removal

To remove the collector and its startup task while preserving collected logs:

```powershell
.\Perfmon\Configure-BlackBox.ps1 -Action Remove
```

Alerting is deferred; this script collects troubleshooting data only.

## Verification and troubleshooting

- After installation, confirm the script reports that `BlackBox` is running.
  Check **Data Collector Sets > User Defined > BlackBox** in `perfmon.exe`
  and the `Start BlackBox` task in Task Scheduler.
- The 21 requested paths can expand to many instances. The enabled path count
  can be lower if optional counters are unavailable on that endpoint.
- A `Skipping` warning identifies an unavailable counter or instance. Check
  the named counter set locally, for example:

  ```powershell
  Get-Counter -ListSet 'Hyper-V Virtual Storage Device'
  ```

- Claude-related VM counters depend on the installed Windows virtualization
  components. Cowork instances may be absent while its local VM is stopped.
- If no requested counters are available, installation fails before changing
  the existing collector. Later failures report the stage and any changes
  already made; review that output before retrying.
- Review log disk usage separately. The script does not enforce a total
  retention cap or automatically delete old logs by age.
