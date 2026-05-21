# ASA Running Configs

Each subdirectory contains the exported running configuration for that firewall.

## How to Export from Packet Tracer

1. Open `packet-tracer/cyb330l-network.pkt` in Cisco Packet Tracer
2. Click on a firewall device → CLI tab
3. Run: `show running-config`
4. Select all output, copy, and paste into the corresponding `running-config.txt` file

## Devices

| Directory | Device | Outside IP |
|-----------|--------|------------|
| `EMAGINE-FW/` | EMAGINE-FW (HQ) | 11.0.0.42 |
| `ORLANDO-FW/` | ORLANDO-FW | 11.0.0.44 |
| `DC-FW/` | DC-FW | 11.0.0.46 |
| `CHICAGO-FW/` | CHICAGO-FW | 11.0.0.48 |
