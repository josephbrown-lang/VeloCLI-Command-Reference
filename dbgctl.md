# dbgctl

## Description

`dbgctl` is a standalone diagnostic utility at `/opt/vc/bin/dbgctl` that reads from edged's shared-memory ring buffers to display, follow, and control runtime debug log output. It is not a VeloCLI `debug --` subcommand — it is run directly from the edge shell.

edged writes all `DBG_LOG()` statements into an in-memory ring buffer. `dbgctl` reads that buffer without requiring edged to write to disk, making it the lowest-overhead way to observe edged internals in real time. Individual log statements can be selectively enabled or disabled at runtime without restarting edged.

Log levels range from `EMERG` (0) to `TRACE` (9). Only statements at or below the active level are recorded.

## Syntax

```
dbgctl [OPTIONS] [PATTERN|LEVEL]
```

The optional positional argument is either a glob/substring pattern (used with `-e`, `-d`, `-p`) or a numeric log level.

## Options

| Flag | Long form | Description |
|---|---|---|
| `-f` | `--follow` | Follow mode — continuously print new log entries as they arrive (like `tail -f`). |
| `-c` | `--clear` | Clear the ring buffer. Combined with `-f`, starts following from a clean slate. |
| `-e [PATTERN\|LEVEL]` | `--enable` | Enable log statements matching the pattern or up to the specified level. Takes effect immediately without restarting edged. |
| `-d [PATTERN\|LEVEL]` | `--disable` | Disable log statements matching the pattern or above the specified level. |
| `-p [PATTERN\|LEVEL]` | `--print` | Print the current enable/disable state of log statements matching the pattern or at the specified level. |
| `-s SIZE` | `--size=SIZE` | Limit dump output to the last SIZE megabytes of the ring buffer. |
| `-l FILE` | `--log=FILE` | Read from a specific log file instead of the default ring buffer. |
| `-m FILE` | `--map=FILE` | Use a specific debug map file. |
| `-i` | `--iso` | Print timestamps in ISO 8601 format instead of the default. |
| `-r` | `--relative` | Print time elapsed since the previous log entry instead of the absolute timestamp. |
| `-t` | `--stats` | Print ring buffer statistics (consumer/producer head/tail positions, drop counts). |

## Pattern Matching

Patterns used with `-e`, `-d`, and `-p` match against log statement metadata using these rules (evaluated in order):

1. **Glob match** on `function:line` — e.g., `vc_pkt*` matches all log statements in functions starting with `vc_pkt`
2. **Exact match** on function name — e.g., `pkt_tracker`
3. **Substring match** on module name or source file — e.g., `NET`, `vc_pkt.c`
4. **Substring match** on the format string — e.g., `"link_detected"`

A numeric value is interpreted as a log level rather than a pattern.

## Log Levels

| Level | Value | Description |
|---|---|---|
| `EMERG` / `PANIC` | 0 | System is unusable / assertion failure |
| `ALERT` | 1 | Immediate action required |
| `CRIT` | 2 | Critical condition |
| `ERR` / `ERROR` | 3–4 | Error conditions |
| `WARNING` | 5 | Warning conditions |
| `NOTICE` | 6 | Normal but significant events |
| `INFO` | 7 | Informational messages |
| `DEBUG` | 8 | Debug-level detail |
| `TRACE` | 9 | Fine-grained tracing |

## Module Names

Patterns can match against module names used in `DBG_LOG(level, MODULE, ...)` calls. Useful module names include:

| Module | Description |
|---|---|
| `NET` | Core network / packet processing |
| `VPN` | IPsec / IKE tunnel management |
| `VCMP` | VeloCloud Multipath Protocol |
| `ROUTING` | Route management |
| `BIZ` | Business policy |
| `LINKSCHED` | Link scheduler / QoS |
| `NETSCHED` | Network scheduler |
| `NAT` | NAT processing |
| `NATD` | NAT daemon |
| `DNS` | DNS handling |
| `FW` | Firewall |
| `FMGR` | Flow manager |
| `MAIN` | edged main / initialization |
| `LINKMGR` | Link manager / WAN link FSM |
| `PATHFSM` | Path state machine |
| `APPS` | Application identification (DPI) |
| `VLAN` | VLAN handling |
| `DHCP` | DHCP |
| `JITTER` | Jitter buffer / reorder |
| `OSPF` | OSPF routing |
| `GENEVE` | Geneve tunnel |
| `HEALTH` | Health monitoring |

## Common Use Cases

### Follow edged logs in real time
```bash
dbgctl -f
```

### Follow from a clean slate (skip buffered history)
```bash
dbgctl -c -f
```

### Enable DEBUG-level logging (level 8) for all modules
```bash
dbgctl -e 8
```

### Enable logging for a specific module
```bash
dbgctl -e NET
dbgctl -e VCMP
```

### Enable logging for a specific function
```bash
dbgctl -e vc_pkt_print_track
```

### Disable verbose logging above INFO level
```bash
dbgctl -d 8
```

### Show what log statements are currently enabled
```bash
dbgctl -p
```

### Show enabled statements for a specific module
```bash
dbgctl -p NET
```

### Dump the current ring buffer contents
```bash
dbgctl
```

### Dump the last 5 MB of the ring buffer
```bash
dbgctl -s 5
```

### Follow with ISO timestamps
```bash
dbgctl -f -i
```

### Check ring buffer health (drop counts)
```bash
dbgctl -t
```

## Relationship to Other Tools

| Tool | What it shows |
|---|---|
| `dbgctl -f` | Live `DBG_LOG` entries from the edged ring buffer — per-function, per-module detail |
| `tail -f /var/log/edged.log` | Persistent on-disk log — lower verbosity, survives reboots |
| `debug --pkt_tracker` | Arms edged to emit `PKT_LOG` entries for a specific flow — output appears in both `dbgctl` and `/var/log/edged.log` |
| `dispcnt` | Aggregate performance counters from shared memory — no per-event detail |

Use `dbgctl` when you need to observe edged behavior at the function level in real time. Use `tail -f /var/log/edged.log` for persistent logging across sessions or when correlating with timestamps after the fact.
