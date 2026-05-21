# --pkt_tracker [any|sip] [any|sport] [any|dip] [any|dport] [any|proto] [count]

## Description
Tracks the lifecycle of packets matching a specific flow through the VeloCloud Edge dataplane. Once armed, edged logs a `PKT_LOG` message at each processing stage a matching packet passes through, writing to `/var/log/edged.log`. The tracker stays active until the specified packet count is reached or a new invocation resets it.

To view output in real time, tail the log in a separate session and filter by IP:
```bash
tail -f /var/log/edged.log | grep <ip_address>
```

## Arguments
All six arguments are positional and required. Use `any` to match any value for a given field.

| Position | Argument        | Description                                                                              |
|----------|-----------------|------------------------------------------------------------------------------------------|
| 1        | `any` \| `sip`  | Source IP address (IPv4 or IPv6) to match, or `any` to match all source IPs.             |
| 2        | `any` \| `sport`| Source L4 port number to match, or `any` to match all source ports.                     |
| 3        | `any` \| `dip`  | Destination IP address (IPv4 or IPv6) to match, or `any` to match all destination IPs.  |
| 4        | `any` \| `dport`| Destination L4 port number to match, or `any` to match all destination ports.           |
| 5        | `any` \| `proto`| IP protocol number (e.g., `6` for TCP, `17` for UDP, `1` for ICMP), or `any`.          |
| 6        | `count`         | Number of matching packets to track. The tracker disarms after this many packets.        |

## Example Usage

Track the next 50 packets between two specific hosts (any port, any protocol):
```
example_com:velocli> debug --pkt_tracker 192.168.1.10 any 8.8.8.8 any any 50
```

Track TCP port 443 flows from any source to a specific destination:
```
example_com:velocli> debug --pkt_tracker any any 10.0.0.1 443 6 100
```

Then in a second shell session on the edge, watch the log in real time:
```bash
tail -f /var/log/edged.log | grep 192.168.1.10
```

## Output

`pkt_tracker` does not display packet data on the console. All output is written to `/var/log/edged.log` as `PKT_LOG` entries. Each log line represents one processing stage for one matching packet.

### For data flows (with an established flow context)
```
dir: lan_to_wan, 192.168.1.10:12345->8.8.8.8:443 proto 6, seg_id 0, app 108, class 3, fc: 0xffff8a1b policy "Business Policy Rule", reason "vc_queue_link_select", count 49 path "[pkt_path_ether_read, pkt_path_user_flow_fdf, vc_queue_link_select]" pktlen 1400
```

### For flows without a flow context (early in processing or dropped)
```
192.168.1.10:12345->8.8.8.8:443 proto 6, seg_id 0, reason "pkt_path_ether_read", count 49, path "[]"
```

### For VCMP control packets
```
192.168.1.10:0->10.0.0.1:0, "VCMP_PATH_MTU", "pkt_path_vcmp_output_fdf", count 49 path "[pkt_path_vcmp_output_fdf]"
```

## Field Descriptions

| Field | Description |
|---|---|
| `dir` | Traffic direction: `lan_to_wan` (LAN-originated, going to WAN/overlay) or `wan_to_lan` (arriving from WAN/overlay, going to LAN). |
| `<sip>:<sport>-><dip>:<dport> proto <n>` | Five-tuple identifying the flow. `proto` is the IP protocol number. |
| `seg_id` | VeloCloud segment ID the flow belongs to. Segment 0 is the global segment. |
| `app` | Detected application ID (DPI result). `-1` if not yet classified. |
| `class` | Detected application class ID. `-1` if not yet classified. |
| `fc` | Pointer to the flow container (`flow_container`) structure in edged memory. Useful for cross-referencing with other debug output. |
| `policy` | Name of the business policy rule applied to the flow, or `Rule <n>` if the rule has no name. |
| `reason` | The dataplane processing stage where this log was emitted. See path stages below. |
| `count` | Remaining packet count before the tracker disarms. Decrements with each matched packet. |
| `path` | Ordered list of all dataplane stages this packet has visited so far in its lifecycle. |
| `pktlen` | Packet length in bytes at the point of logging. |

## Packet Path Stages

Common values that appear in the `reason` and `path` fields:

| Stage | Description |
|---|---|
| `pkt_path_ether_read` | Packet read from the Ethernet/DPDK interface on ingress. |
| `pkt_path_ipvx_read` | IPv4/IPv6 header parsing. |
| `pkt_path_user_flow_fdf` | User data flow fast-datapath handler — flow lookup and classification. |
| `vc_queue_link_select` | Link/path selection stage — determines which WAN link or tunnel to use. |
| `vc_queue_link_sch` | Link scheduler — applies QoS shaping before sending. |
| `pkt_path_ipsec_encrypt_fdf` | IPsec encryption (fast-datapath). |
| `pkt_path_ipsec_decrypt_fdf` | IPsec decryption (fast-datapath). |
| `pkt_path_vcmp_input_fdf` | VCMP (VeloCloud Multipath Protocol) input fast-datapath handler. |
| `pkt_path_vcmp_output_fdf` | VCMP output fast-datapath handler — overlay encapsulation. |
| `pkt_path_send` | Packet handed off for transmission. |
| `pkt_path_free` | Packet buffer released (end of lifecycle — normal completion or drop). |
| `pkt_path_replicate` | Packet replicated (e.g., multicast or HA sync). |
| `pkt_path_ooo` | Out-of-order packet handling. |
| `pkt_path_late` | Late packet handling (arrived after reorder timeout). |
| `pkt_path_gre_fdf` | GRE tunnel fast-datapath handler. |
| `pkt_path_geneve_fdf` | Geneve tunnel fast-datapath handler. |
| `vc_queue_netif` | Network interface queue processing. |
| `vc_queue_ike` | IKE (IPsec key exchange) processing queue. |
| `pkt_path_frag_data_hdlr` | IP fragmentation handler. |
| `pkt_path_send_local_stack` | Packet sent to the local IP stack (e.g., management traffic). |
