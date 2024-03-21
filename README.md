# Reverse SSH tunnel for Bitcoin Core daemon

If your [Bitcoin Core](https://bitcoin.org/en/full-node) daemon runs behind NAT and your ISP doesn't provide a public
IP address, you may run it as a [Tor hidden service](https://en.bitcoin.it/wiki/Setting_up_a_Tor_hidden_service)
to make it publicly available. However, that might not be sufficient for some use cases and thus a reverse SSH tunnel
might come handy.

## Prerequisites

* Bitcoin Core daemon running on a Linux machine behind NAT.
  * Let's call the machine `node`.
* A server with public IP address you have full access to.
  * Let's call the server `vps`. 

## Instructions

### `vps`

1. Create system user `bitcoin` if not already present.
1. Edit `/etc/ssh/sshd_config`:
   * Allow remote hosts to forwarded ports by setting `GatewayPorts` option to `yes`.
   * Append `bitcoin` user to `AllowUsers` if you already use this option.
1. Restart SSH daemon for changes to take effect:
   ```bash
   $ sudo systemctl restart sshd.service
   ```
1. Open TCP port `8333` in your firewall.

### `node`

1. Install `autossh` if not already present.
1. Use `ssh-keygen` to [generate SSH key](https://www.ssh.com/academy/ssh/keygen) for user `bitcoin`.
1. Append the following snippet to `/home/bitcoin/.ssh/config`:
   ```bash
   Host vps
     HostName #FIXME
     RemoteForward 8333 localhost:8333
     ServerAliveInterval 30
     ServerAliveCountMax 3
   ```
   Replace `#FIXME` with the IP address of your `vps`.
1. Use `ssh-copy-id` to [copy generated SSH key](https://www.ssh.com/academy/ssh/copy-id) to `vps`.
1. Copy [`bitcoind-tunnel.service`](bitcoind-tunnel.service) to `/etc/systemd/system/bitcoind-tunnel.service`
1. Reload systemd services:
   ```bash
   $ sudo systemctl daemon-reload
   ```
1. Start the service and enable it at startup:
   ```bash
   $ sudo systemctl start bitcoind-tunnel.service
   $ sudo systemctl enable bitcoind-tunnel.service
   ```

That's it, your node is publicly available! You may check it via [bitnodes.io](https://bitnodes.io/#join-the-network).

## Resources

- [Directions for setting up a reverse ssh tunnel as a systemd service in Linux](https://github.com/axthosarouris/reverse-ssh-tunnel)
- [SSH tunnelling for fun and profit: AutoSSH](https://www.everythingcli.org/ssh-tunnelling-for-fun-and-profit-autossh/)
