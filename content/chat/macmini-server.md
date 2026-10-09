On your Mac Mini, open **Terminal** and run:

```bash
ipconfig getifaddr en0
```

- `en0` = **Ethernet** (wired — recommended for a server)
- `en1` = **Wi-Fi** (if you're using wireless instead)

Not sure which one you're on? Run this to see all active interfaces:

```bash
ifconfig | grep "inet " | grep -v 127.0.0.1
```

This lists all active IPs — your local one will look like `192.168.1.x`.

---

Or without Terminal: **System Settings → Network** → click your active connection (Ethernet or Wi-Fi) → the IP address is shown right there.