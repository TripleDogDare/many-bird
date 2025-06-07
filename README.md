# many-bird
Configuration instructions for multiple netbird instances on linux

# More netbird configs!
I add more netbird configs to a `config.d` directory.

```bash
sudo mkdir /etc/netbird/config.d
# Login with whatever method you want but set the config file
netbird login --config /etc/netbird/config.d/wt1.config
sudo chmod 600 /etc/netbird/config.d/wt1.config
```
Set the new config file to be `rw` only by root with `chmod 600` like the default config file.

Modify the interface name and port to not conflict with other services.
I generally have the interface name match the file name.
- `"WgIface": "wt1"`
- `"WgPort": 51821`

I also add all netbird interfaces to the blacklist.
I'm not sure if this is strictly necessary but seems like a good idea.
```json
      "IFaceBlackList": [
        "wt0",
        "wt1",
        "wt",
        "utun",
        "tun0",
        "zt",
        "ZeroTier",
        "wg",
        "ts",
        "Tailscale",
        "tailscale",
        "docker",
        "veth",
        "br-",
        "lo"
    ],
```

# Systemd
Copy `netbird@.service` to `/etc/systemd/system`

This service is configured to load services from `/etc/netbird/config.d/*.json`.

The file name designates the service name.
```bash
sudo systemctl enable netbird@wt1
sudo systemctl start netbird@wt1
sudo systemctl status netbird@wt1
```

## Status
You can poll the status by specifying the particular service socket.
```
$ netbird status --daemon-addr unix:///var/run/netbird-wt1.sock
OS: linux/amd64
Daemon version: 0.44.0
CLI version: 0.44.0
Management: Connected
Signal: Connected
Relays: 3/3 Available
Nameservers: 0/0 Available
FQDN: my-device.netbird.cloud
NetBird IP: 100.121.5.251/16
Interface type: Kernel
Quantum resistance: false
Networks: -
Forwarding rules: 0
Peers count: 1/1 Connected
```

# nbstat
Dependences:
- [jq](https://jqlang.org/)

This is a custom script that will output a nicely formatted status of peers.
This will look more like `tailscale status` and pokes each netbird socket individually.
```
  $ nbstat
=== Status for socket: /run/netbird.sock ===
FQDN                     NetBird IP      Status        Relay  Last Seen
device1.netbird.cloud    100.80.50.94    Disconnected         0000-12-31T18:09:24-05:50
device2.netbird.cloud  100.80.148.153  Disconnected         0000-12-31T18:09:24-05:50
device3.netbird.cloud         100.80.201.142  Disconnected         0000-12-31T18:09:24-05:50

=== Status for socket: /run/netbird-wt1.sock ===
FQDN                  NetBird IP       Status     Relay                                             Last Seen
peer1.netbird.cloud  100.121.150.251  Connected  rels://streamline-us-chi1-0.relay.netbird.io:443  2025-06-07T14:32:26.350704022-05:00
```
