# Virtual Printer Setup Guide

## Overview

The Virtual Printer feature allows Bambuddy to emulate one or more Bambu Lab printers on your network. This enables you to:

- Send prints directly from Bambu Studio or OrcaSlicer to Bambuddy without a physical printer
- Queue prints for later
- Archive prints without printing immediately
- Use multiple virtual printers

For detailed documentation, see: https://wiki.bambuddy.cool/features/virtual-printer/

## Important Warning ⚠️

Enabling Virtual Printer support adds **many network ports** to your Bambuddy container:

| Service | Ports | Protocol | Purpose |
|---------|-------|----------|---------|
| Bind/Detect | 3000, 3002 | TCP | Required for slicer discovery |
| SSDP | 2021 | UDP | Printer discovery on same LAN |
| MQTT | 8883 | TCP/TLS | Printer communication |
| File Transfer | 6000 | TCP/TLS | Job verification & upload |
| RTSP Camera | 322 | TCP/TLS | Camera streaming (X1/H2/P2) |
| FTPS | 990 | TCP/TLS | File transfer control (privileged port) |
| FTP Data | 50000-50100 | TCP | Passive file transfer data |
| Proprietary | 2024-2026 | TCP/TLS | A1/P1S slicer communication |

**Total: ~12+ ports will be exposed**

Make sure these ports are not in use on your Home Assistant host before enabling.

## How to Enable Virtual Printer

1. **Go to Home Assistant Add-ons**
   - Settings → Add-ons, Backups & Repositories → Bambuddy

2. **Edit the Configuration**
   - Click on Bambuddy → Configuration tab
   - Find `virtual_printer_enabled` option

3. **Enable Virtual Printer**
   - Set `virtual_printer_enabled` to `true`
   - Leave other port settings as default unless you have conflicts

4. **Save and Restart**
   - Click "Save"
   - The add-on will restart with Virtual Printer support enabled
   - Check the logs to confirm ports are configured correctly

## Port Configuration

If you have port conflicts with other services:

1. **In the Configuration UI**, change individual ports:
   - `bind_port_1` - Primary bind port (default: 3000)
   - `bind_port_2` - Secondary bind port (default: 3002)
   - `ssdp_port` - SSDP discovery (default: 2021)
   - `mqtt_port` - MQTT communication (default: 8883)
   - `file_transfer_port` - File transfer tunnel (default: 6000)
   - `rtsp_camera_port` - Camera streaming (default: 322)
   - `ftps_port` - FTPS control (default: 990, privileged port)
   - `ftp_data_port_start/end` - FTP data range (default: 50000-50100)
   - `proprietary_port_start/end` - Proprietary protocol (default: 2024-2026)

⚠️ **Important Notes on Ports:**
- Ports 1-1023 are privileged ports and may require special permissions
- Port 990 (FTPS) is restricted due to being a privileged port - changing it may break FTP functionality
- After changing ports, you must re-add the printer in your slicer

2. **Restart the add-on after port changes**

## Next Steps: Certificate Setup

After enabling Virtual Printer, you need to add the Bambuddy CA certificate to your slicer.

### For Bambu Studio & OrcaSlicer:

1. **Extract the CA Certificate**

   From the Bambuddy web UI:
   - Go to Settings → Virtual Printer → download or copy the CA certificate

   Or via Docker:
   ```bash
   docker exec addon_0f8c2bfe_bambuddy cat /app/data/virtual_printer/certs/bbl_ca.crt > ~/bambuddy-ca.crt
   ```

2. **Locate Your Slicer's Certificate File**

   | OS | Bambu Studio | OrcaSlicer |
   |---|---|---|
   | Windows | `C:\Users\<username>\AppData\Local\BambuStudio\cert\printer.cer` | `C:\Users\<username>\AppData\Local\OrcaSlicer\cert\printer.cer` |
   | macOS | `/Applications/BambuStudio.app/Contents/Resources/cert/printer.cer` | `/Applications/OrcaSlicer.app/Contents/Resources/cert/printer.cer` |
   | Linux | `~/.local/share/BambuStudio/cert/printer.cer` | `~/.local/share/OrcaSlicer/cert/printer.cer` |

3. **Append the CA Certificate**

   - Open `printer.cer` in a text editor
   - Go to the end of the file
   - Paste the entire contents of `bambuddy-ca.crt` below the last `-----END CERTIFICATE-----`
   - Save the file

   ✅ **Don't replace** — append to preserve physical printer support

4. **Restart Your Slicer Completely**

   - **Windows**: End Task and reopen
   - **macOS**: Cmd+Q (not just close) and reopen
   - **Linux**: Fully quit and restart

## Creating Your First Virtual Printer

Once the certificate is installed:

1. **In the Bambuddy Web UI**
   - Settings → Virtual Printer → Add Virtual Printer

2. **Configure the Virtual Printer**
   - **Name**: e.g., "Archive Printer" or "Review Queue"
   - **Mode**: Choose one:
     - `Immediate` - Files auto-archive
     - `Review` - Files go to pending for manual review
     - `Print Queue` - Files archive and queue for printing
     - `Proxy` - Forward to a real printer (advanced)
   - **Printer Model**: Select which Bambu printer to emulate (e.g., X1C, P1S, A1)
   - **Access Code**: 8-character code (same as your real printer)
   - **Bind IP**: A unique IP on your network (e.g., 192.168.1.101)
   - Click **Create** → toggle **Enabled**

3. **Add to Your Slicer**

   In Bambu Studio/OrcaSlicer:
   - Device → Add Printer
   - The virtual printer should appear in discovery
   - Or manually add by IP: enter the Bind IP + access code
   - Select the printer model you defined
   - Click Add

4. **Test with a Print**

   - Slice a model
   - Select the virtual printer from the dropdown
   - Click the **Send** button (NOT Print)
   - The file should appear in Bambuddy

   ⚠️ **Use Send, not Print** — in server modes, Print won't work since there's no real printer

## Modes Explained

| Mode | Archives Files | Queues for Print | Use Case |
|------|---|---|---|
| **Immediate** | ✅ Auto | ❌ No | Quick archiving without review |
| **Review** | ✅ Manual | ❌ No | Inspect files before archiving |
| **Print Queue** | ✅ Yes | ✅ Yes | Build a queue to dispatch later |
| **Proxy** | ❌ No | ❌ No | Forward to a real printer (remote printing) |

## Firewall Configuration

If you have local firewall rules, open these ports:

**UFW (Ubuntu/Debian):**
```bash
sudo ufw allow 3000/tcp       # Bind/detect
sudo ufw allow 3002/tcp       # Bind/detect (alt)
sudo ufw allow 2021/udp       # SSDP
sudo ufw allow 8883/tcp       # MQTT
sudo ufw allow 990/tcp        # FTPS
sudo ufw allow 6000/tcp       # File transfer
sudo ufw allow 322/tcp        # RTSP camera
sudo ufw allow 2024:2026/tcp  # Proprietary ports
sudo ufw allow 50000:50100/tcp  # FTP data
```

**Firewalld (Fedora/RHEL):**
```bash
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=3002/tcp
sudo firewall-cmd --permanent --add-port=2021/udp
sudo firewall-cmd --permanent --add-port=8883/tcp
sudo firewall-cmd --permanent --add-port=990/tcp
sudo firewall-cmd --permanent --add-port=6000/tcp
sudo firewall-cmd --permanent --add-port=322/tcp
sudo firewall-cmd --permanent --add-port=2024-2026/tcp
sudo firewall-cmd --permanent --add-port=50000-50100/tcp
sudo firewall-cmd --reload
```

## Troubleshooting

### Slicer can't find the virtual printer

- ✅ Confirm `virtual_printer_enabled` is `true` in Bambuddy settings
- ✅ Check the Bambuddy logs for errors (Settings → Logs)
- ✅ Verify ports 3000/3002 are reachable: `nc -zv <BAMBUDDY_IP> 3000`
- ✅ For same-LAN discovery, verify 2021/UDP is open
- ✅ If on VPN/remote, manually add by IP instead of discovery

### TLS certificate error in slicer

- ✅ Restart the slicer completely (not just close)
- ✅ Verify the certificate was appended correctly to `printer.cer`
- ✅ Check that it's a valid PEM certificate (starts with `-----BEGIN CERTIFICATE-----`)
- ✅ If switching between Bambuddy instances, regenerate and re-append the certificate

### FTP errors / "Connection refused"

- ✅ Check that port 990 (FTPS) is not in use: `telnet localhost 990`
- ✅ Ensure `CAP_NET_BIND_SERVICE` is enabled (should be automatic in config)
- ✅ Check Bambuddy logs for "Permission denied" errors
- ✅ Verify FTP data port range (50000-50100) is available

### Files don't appear in Bambuddy

- ✅ Use the **Send** button, not Print (in server modes)
- ✅ Check the Bambuddy web UI under Archive or Print Queue
- ✅ Verify the printer model in Bambuddy matches the printer selected in the slicer

### Port conflicts

- In the add-on Configuration, change the conflicting port(s)
- Avoid using ports 1-1023 unless necessary (privileged ports)
- After changing, restart the add-on and update the slicer

## Disabling Virtual Printer

1. Set `virtual_printer_enabled` to `false` in add-on Configuration
2. Click Save
3. The add-on restarts without the extra ports
4. Existing virtual printers in Bambuddy remain but won't be accessible

## Additional Resources

- **Official Documentation**: https://wiki.bambuddy.cool/features/virtual-printer/
- **Docker Configuration**: https://wiki.bambuddy.cool/features/virtual-printer/#docker-linux
- **Troubleshooting Guide**: https://wiki.bambuddy.cool/features/virtual-printer/#troubleshooting

---

**Last Updated**: 2025-01-01  
**Based on Bambuddy Docs**: https://wiki.bambuddy.cool/features/virtual-printer/
