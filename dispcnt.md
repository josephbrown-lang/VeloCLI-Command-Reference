# dispcnt

## Description

`dispcnt` is a standalone diagnostic utility at `/opt/vc/bin/dispcnt` that reads internal performance counters directly from shared memory (`/dev/shm`). It is not a VeloCLI `debug --` subcommand — it is run directly from the edge shell.

Counters are maintained by `edged` and `gwd` across named **domains** (e.g., `pkttrace`, `tun`, `ipsec`, `vcedge.com`). Each counter domain maps to a shared-memory file that `dispcnt` reads without requiring edged to be responsive.

By default `dispcnt` prints all counters with their absolute value and a per-second delta, refreshing every 2 seconds indefinitely. Filtering options make it practical for targeted monitoring.

## Syntax

```
dispcnt [OPTIONS] [REGEX...]
```

Trailing positional arguments are treated as additional regex filters (same as `-e`).

## Options

| Flag | Long form | Description |
|---|---|---|
| `-a` | `--no-delta` | Show absolute values only — no delta column. Also sets max repeats to 1 unless `-r` is specified. |
| `-c` | `--clear` | Treat the counter value at startup as zero — output shows change since the command was started. Does not modify the actual counter. |
| `-d STRING` | `--domain=STRING` | Show counters from the specified domain only. Can be repeated for multiple domains. |
| `-D[REGEX]` | `--no-domain[=REGEX]` | Exclude domains matching the regex. Without a regex, excludes domains matching a UUID pattern (flow/peer per-entry counters). |
| `-e REGEX` | `--regex=REGEX` | Show only counters whose name matches the regex. Can be repeated — multiple patterns are OR'd. |
| `-g` | `--all-domains` | Show all domains including UUID-named ones (overrides default UUID exclusion). |
| `-m INT` | `--min=INT` | Skip counters with a value less than INT. |
| `-M INT` | `--max=INT` | Skip counters with a value greater than INT. |
| `-n` | `--domain-names` | Prefix each counter name with its domain name. |
| `-p STRING` | `--prefix=STRING` | Show only counters whose name starts with STRING. Can be repeated. |
| `-r INT` | `--repeat=INT` | Stop after INT iterations. Minimum 1. |
| `-s STRING` | `--substr=STRING` | Show only counters whose name contains STRING. Can be repeated — multiple strings are OR'd. |
| `-t INT` | `--interval=INT` | Seconds between refreshes (default: 2). |
| `-z` | `--no-zero` | Hide counters with a current value of zero. |
| `-Z` | `--no-zero-delta` | Hide counters whose value has not changed since the last interval. |

## Output Format

### With delta (default)
```
Wed May 21 10:42:00 2025

imissed                                  = 0             0      /s
rx_dropped                               = 14            0      /s
tx_dropped                               = 0             0      /s
tot_flow                                 = 2847          0      /s
```

Columns: `<counter_name> = <cumulative_value>  <delta>/s`

### Absolute values only (`-a`)
```
imissed                        = 0
rx_dropped                     = 14
tot_flow                       = 2847
```

## Counter Domains

Named domains exposed by edged/gwd. Use `-d <domain>` to scope to one.

| Domain | Description |
|---|---|
| `vcedge.com` | Global edged counters (flows, drops, forwarding decisions) |
| `pkttrace` | Packet tracker counters (`pkttrace_oor`, `pkttrace_catchall`) |
| `tun` | Tunnel-level counters |
| `ipsec` | IPsec encrypt/decrypt counters |
| `apps` | Application identification counters |
| `enterprise` | Per-enterprise counters |
| `edged-nw-mem` | edged network memory pool usage |
| `natd-mem` | NAT daemon memory pool usage |
| `natd.pinfo` | NAT port-info table counters |
| `natd.shmem` | NAT shared-memory counters |
| `vcgw.com` | Global gateway (gwd) counters |
| `vcgwnat.com` | Gateway NAT counters |

UUID-named domains (e.g., `a1b2c3d4-...`) represent per-flow or per-peer entry counters and are excluded by default. Use `-g` to include them.

## Common Use Cases

### Watch for drops and errors in real time
The most common use during performance testing or troubleshooting:
```bash
dispcnt -s imissed -s error -s drop -r 1 -z 0
```
Prints one snapshot of all counters containing "imissed", "error", or "drop", including zero-value ones.

For continuous monitoring:
```bash
dispcnt -s imissed -s error -s drop -Z
```
Refreshes every 2 seconds, hiding counters that haven't changed.

### Check current flow count
```bash
dispcnt -z -s tot_flow -r 1
```

### Watch a counter increment live (relative to start)
```bash
dispcnt -c -s rx_dropped -Z
```
Starts from zero and shows only counters that are actively incrementing.

### Snapshot all counters to a file
```bash
/opt/vc/bin/dispcnt -a > /tmp/counters_$(date +%Y%m%d_%H%M%S).txt
```
Useful for capturing a baseline or attaching to a support ticket.

### Show counters for a specific domain with domain names prefixed
```bash
dispcnt -a -n -d pkttrace
```

### Filter by regex
```bash
dispcnt -e "^rx_|^tx_" -Z
```

## Relationship to pkt_tracker

`debug --pkt_tracker` arms edged to log per-packet lifecycle events to `/var/log/edged.log`. `dispcnt -d pkttrace` shows the aggregate counters for that tracker (`pkttrace_oor` for out-of-range packets, `pkttrace_catchall` for catch-all matches). Use both together when diagnosing specific flows: `pkt_tracker` gives per-packet detail, `dispcnt` gives the running totals.
