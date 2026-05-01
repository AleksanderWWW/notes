# Kernel Hardening Ubuntu/Debian

These notes outline a few techniques and procedures that increase the security of the kernel
on a Ubuntu machine.

## Applying `Lynis` audit to the kernel configuration

Install `Lynis` from `apt`:

```bash
sudo apt update

sudo apt install lynis
```

Run the `Lynis` audit of the system:

```bash
sudo lynis audit system
```

Scroll to `[+] Kernel Hardening` section. The The output will look similar to this:

```terminaloutput
[+] Kernel Hardening
------------------------------------
  - Comparing sysctl key pairs with scan profile
    - dev.tty.ldisc_autoload (exp: 0)                         [ DIFFERENT ]
    - fs.protected_fifos (exp: 2)                             [ DIFFERENT ]
    - fs.protected_hardlinks (exp: 1)                         [ OK ]
    - fs.protected_regular (exp: 2)                           [ OK ]
    - fs.protected_symlinks (exp: 1)                          [ OK ]
    - fs.suid_dumpable (exp: 0)                               [ DIFFERENT ]
    - kernel.core_uses_pid (exp: 1)                           [ DIFFERENT ]
    - kernel.ctrl-alt-del (exp: 0)                            [ OK ]
    - kernel.dmesg_restrict (exp: 1)                          [ OK ]
    - kernel.kptr_restrict (exp: 2)                           [ DIFFERENT ]
    - kernel.modules_disabled (exp: 1)                        [ DIFFERENT ]
    - kernel.perf_event_paranoid (exp: 3)                     [ DIFFERENT ]
    - kernel.randomize_va_space (exp: 2)                      [ OK ]
    - kernel.sysrq (exp: 0)                                   [ DIFFERENT ]
    - kernel.unprivileged_bpf_disabled (exp: 1)               [ DIFFERENT ]
    - kernel.yama.ptrace_scope (exp: 1 2 3)                   [ OK ]
    - net.core.bpf_jit_harden (exp: 2)                        [ DIFFERENT ]
    - net.ipv4.conf.all.accept_redirects (exp: 0)             [ DIFFERENT ]
    - net.ipv4.conf.all.accept_source_route (exp: 0)          [ OK ]
    - net.ipv4.conf.all.bootp_relay (exp: 0)                  [ OK ]
    - net.ipv4.conf.all.forwarding (exp: 0)                   [ OK ]
    - net.ipv4.conf.all.log_martians (exp: 1)                 [ DIFFERENT ]
    - net.ipv4.conf.all.mc_forwarding (exp: 0)                [ OK ]
    - net.ipv4.conf.all.proxy_arp (exp: 0)                    [ OK ]
    - net.ipv4.conf.all.rp_filter (exp: 1)                    [ DIFFERENT ]
    - net.ipv4.conf.all.send_redirects (exp: 0)               [ DIFFERENT ]
    - net.ipv4.conf.default.accept_redirects (exp: 0)         [ DIFFERENT ]
    - net.ipv4.conf.default.accept_source_route (exp: 0)      [ DIFFERENT ]
    - net.ipv4.conf.default.log_martians (exp: 1)             [ DIFFERENT ]
    - net.ipv4.icmp_echo_ignore_broadcasts (exp: 1)           [ OK ]
    - net.ipv4.icmp_ignore_bogus_error_responses (exp: 1)     [ OK ]
    - net.ipv4.tcp_syncookies (exp: 1)                        [ OK ]
    - net.ipv4.tcp_timestamps (exp: 0 1)                      [ OK ]
    - net.ipv6.conf.all.accept_redirects (exp: 0)             [ DIFFERENT ]
    - net.ipv6.conf.all.accept_source_route (exp: 0)          [ OK ]
    - net.ipv6.conf.default.accept_redirects (exp: 0)         [ DIFFERENT ]
    - net.ipv6.conf.default.accept_source_route (exp: 0)      [ OK ]
```

Paste just the pairs of commands to a file `secure_values.conf`:


```text
# secure_values.conf

- dev.tty.ldisc_autoload (exp: 0)                         [ DIFFERENT ]
- fs.protected_fifos (exp: 2)                             [ DIFFERENT ]
- fs.protected_hardlinks (exp: 1)                         [ OK ]
- fs.protected_regular (exp: 2)                           [ OK ]
- fs.protected_symlinks (exp: 1)                          [ OK ]
...
```

Now create the file that contains only the lines with `DIFFERENT` marker in the structure
that assigns the setting to the value expected by `Lynis`:

```bash
grep "DIFFERENT" secure_values.conf | sed 's/^- \([^ ]*\) (exp: \([^)]*\)).*/\1 = \2/' > 60-kernel-hardening.conf
```

Your `60-kernel-hardening.conf` should look similar to:

```text
dev.tty.ldisc_autoload = 0
fs.protected_fifos = 2
fs.suid_dumpable = 0
kernel.core_uses_pid = 1
kernel.kptr_restrict = 2
kernel.modules_disabled = 1
kernel.perf_event_paranoid = 3
kernel.sysrq = 0
kernel.unprivileged_bpf_disabled = 1
net.core.bpf_jit_harden = 2
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.log_martians = 1
net.ipv6.conf.all.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0
```

Now copy this file to the `/etc/sysctl.d/` catalog:

```bash
sudo cp 60-kernel-hardening.conf /etc/sysctl.d/
```

Test the configuration for potential syntax errors:

```bash
sudo sysctl -p /etc/sysctl.d/60-kernel-hardening.conf
echo $?  # should give 0
```

For the new settings to take effect you need to reboot the machine:

```bash
sudo systemctl reboot
```

Now rerun the `Lynis` audit to see if all the kernel settings are now as expected.


### Note on the filename

The `60-` prefix was chosen because the system reads the setting files alphabetically/numerically.
This way we ensure that your security tweaks are applied after the standard system boot configurations
but before any highly specific late-stage tweaks.

## Securing process visibility

By default any logged-in user can view other users' (including root) processes.
This can be disabled by editing the `/etc/fstab` file:

```text
# /etc/fstab
...

proc /proc proc hidepid=2 0 0

...
```

Now remount the `/proc` catalog:

```bash
sudo systemctl daemon-reload
sudo mount -o remount proc
```

This way only the `sudo` user can view processes others than one's own.

You can verify it by running as a non-root user:

```bash
ps -aux | grep root
```

The only matching output should be the `grep` process itself:

```terminaloutput
... grep --color=auto root
```
