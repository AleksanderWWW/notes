# SSH Hardening Ubuntu/Debian

After every change run

```bash
sudo sshd -t
sudo systemctl reload ssh
sudo systemctl status ssh
```

for the change to take effect.

**WARNING: Don't close your current SSH session until you verify you can log in via a second window**

**Pre-requisite: Ensure your public key is in ~/.ssh/authorized_keys before reloading!**

## Changes to /etc/ssh/sshd_config

### Disable root login

```text
PermitRootLogin no
```

### Disable password authentication

```text
PasswordAuthentication no
```

### Limit amount of auth trials

```text
MaxAuthTries 3
```

### Disable weak encryption algorithms

```text
# Ciphers and keying
# RekeyLimit default none
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group-exchange-sha256
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com,umac-128-etm@openssh.com
```

### [OPTIONAL] Modify log level

```text
LogLevel INFO  # VERBOSE, DEBUG, DEBUG1...
```

Logs are accessible at `/var/log/auth.log`.

### Access Control

```text
AllowUsers <username>
```

Or the entire group

```text
AllowGroups <groupname>
```

### Disable X11 forwarding

```text
X11Forwarding no
```

### [Optional] Disable SSH tunneling 

```text
AllowTcpForwarding no
AllowStreamLocalForwarding no
GatweayPorts no
PermitTunnel no
```

### Automatically logout stale users

```text
ClientAliveInterval 100  # seconds of inactivity before logout
ClientAliveCountMax 3  # three "are you there?" messages before killing a connection
```

### Configure authorized keys permissions

```bash
sudo mkdir -p /etc/ssh/authorized_keys
sudo chown root:root /etc/ssh/authorized_keys
sudo chmod 755 /etc/ssh/authorized_keys
```

then in the ssh config:

```text
# %u is a mini-macro that expands to the current username
AuthorizedKeysFile /etc/ssh/authorized_keys/%u
```

Example for a user `alex`:

```bash
# Lockdown: Only root can read/write; others can only read
sudo chown root:root /etc/ssh/authorized_keys/alex
sudo chmod 644 /etc/ssh/authorized_keys/alex
```

Now `alex` can read his authorized key file, but only the root can change it.
This means that if you want to add another public key you need to explicitly ask admin to do it for you.

### [Optional] Change default port

```text
Port 2222
```

### Banner

```text
Banner /etc/ssh/sshd-banner
```

then in `/etc/ssh/sshd-banner`:

```text
Authorized users only. All others will be prosecuted!
```
