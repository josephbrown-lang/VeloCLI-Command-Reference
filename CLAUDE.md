# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository is a comprehensive reference documentation for VeloCloud CLI (VeloCLI) debug commands. It contains 312 markdown files documenting individual debug commands for VeloCloud SD-WAN Edge devices. This is a documentation-only repository with no build, test, or runtime components.

## Repository Structure

```
/debug/               # All VeloCLI debug command documentation (312 .md files)
  *.md               # Individual command reference files
LICENSE              # MIT License
```

## Documentation Format

Each command documentation file follows a consistent structure:

1. **Command syntax** - Header with full command syntax including arguments
2. **Description** - Overview of what the command does and when to use it
3. **Arguments** - Table describing parameters (if applicable)
4. **Example usage** - Command examples with sample output (often JSON or tabular format)
5. **Field descriptions** - Table explaining output fields

### Common Patterns

- Commands use double-dash prefix: `debug --command_name`
- Arguments are typically enclosed in brackets: `{required}`, `[optional]`
- Output formats include JSON arrays, tabular data, and key-value pairs
- Many commands support filtering by:
  - IP version (`v4`, `v6`, `all`)
  - Segment ID (network segmentation)
  - Logical ID (peer Edge/Gateway identifiers)
  - IP addresses or prefixes

## Key Command Categories

The debug commands cover these major SD-WAN functional areas:

### Routing & Overlay
- **Routes**: `routes.md`, `overlay_routes.md`, `static_routes.md`, `remote_routes.md`, etc.
- **BGP**: `bgp.md`, `bgp_view.md`, `bgpd_dump.md`, etc.
- **Tunnels**: `tunnel_counts.md`, `ike_tunnel_debug.md`

### Traffic & Flows
- **Flow Analysis**: `flow_dump.md` - comprehensive flow table with QoS, NAT, application details
- **Applications**: `applications.md`, `app_*` series for application identification

### Network Services
- **DNS**: `dns_ip_cache.md`, `dns_name_lookup.md`, etc.
- **ARP/ND**: `arp_dump.md`, `clear_arp_cache.md`, `clear_nd6_cache.md`
- **BFD**: `bfd_info.md`, `bfdd_dump.md`

### Performance & Diagnostics
- **Bandwidth**: `bw_test_link.md`, `debug_bw_test.md`
- **QoS**: `qos_dump.md`, `qos_stats.md`
- **Statistics**: Various `*_stats.md` files

### Platform & Hardware
- **DPDK**: `dpdk_ports_dump.md`, `dpdk_bond_dump.md`
- **Interfaces**: `interface_dump.md`, `physical_interfaces.md`

## Working with Documentation

### Finding Commands

Commands are named descriptively. Common prefixes include:
- Route-related: `*routes*.md`
- Stats/metrics: `*stats*.md`
- Dumps: `*dump*.md`
- Configuration: `*config*.md`

### Updating Documentation

When modifying command documentation:

1. **Maintain consistent structure** - Follow the format: syntax → description → arguments → examples → field descriptions
2. **Include complete examples** - Show actual command output (JSON/tables) when possible
3. **Document all fields** - Provide comprehensive field descriptions for command output
4. **Cross-reference related commands** - Link to VeloCloud documentation URLs where relevant
5. **Note deprecations** - Mark deprecated commands clearly (e.g., `client_connector.md`)

### Common VeloCloud Concepts Referenced

- **Logical ID**: UUID identifying VeloCloud Edges and Gateways
- **Segment ID**: Network segmentation (segment 0 is global, others increment from 1)
- **VCRP**: VeloCloud Routing Protocol (overlay routing)
- **Edge2Edge**: Direct tunnels between VeloCloud Edges
- **Cloud routes**: Routes to VeloCloud Gateways
- **Overlay vs. Underlay**: SD-WAN overlay network vs. physical transport network
- **DPI**: Deep Packet Inspection for application identification
- **QoE**: Quality of Experience metrics
- **DMPO/DEC**: Dynamic Multipath Optimization / Dynamic Error Correction

## No Build/Test Commands

This is a pure documentation repository. There are no:
- Build processes
- Test suites
- Linting tools
- Package dependencies
- Runtime environments

All work involves markdown file editing only.
