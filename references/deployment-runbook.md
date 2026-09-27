# Deployment runbook

Use this reference after the skill has established that the server and the upstream proxy are authorized. Values in angle brackets are runtime inputs; never replace them with secrets in a reusable file.

## 1. Deployment manifest

Record before making changes:

```text
provider: <GCP/DMIT/DigitalOcean/other>
server_name: <provider name>
region: <region>
os: <distribution and version>
public_ipv4: <address>
public_ipv6: <address or none>
ssh_account: <provider account>
ssh_key_path: <local path; do not copy the key into chat>
management_access: <loopback / SSH tunnel / allowlist / reverse proxy>
node_port(s): <exact ports>
domain_and_tls: <domain, certificate path, or none>
panel: <3x-ui version or current release>
upstream_egress: <none / authenticated HTTP / SOCKS5 / provider-specific>
client_targets: <Mihomo/Clash Verge/mobile/other>
rule_policy: <private direct, domestic direct, overseas proxy, named services>
rollback_point: <backup path or provider snapshot>
```

If the user only supplies an IP and a password, do not infer that the password is safe to use. Prefer asking for the key path or provider console access. If the provider prohibits root login, use the documented account and do not attempt to bypass that control.

## 2. SSH bootstrap and recovery

Use a key-backed connection and verify the host before changing it:

```bash
ssh -i <KEY_PATH> <ACCOUNT>@<SERVER_IP>
id
hostnamectl
cat /etc/os-release
ss -lntup
ip -br address
ip route
```

Create a separate administrator only after confirming the current account has `sudo` access. Test the new account in a second session before changing `sshd` policy. A safe order is:

1. install the public key for the new account;
2. verify a new key-based session;
3. verify non-interactive `sudo` for the intended commands;
4. only then consider disabling password authentication or direct root login;
5. retain the original recovery session until the new path is proven.

Back up SSH, firewall, panel, and node configuration before editing. Never print private keys, passwords, panel tokens, UUIDs, or subscription URLs in diagnostic output.

## 3. Baseline hardening

- Update the OS and security packages using the distribution's supported mechanism.
- Install only needed packages. Prefer the provider's firewall plus a host firewall with an explicit allowlist.
- Allow SSH from the user's trusted source when practical; otherwise use the provider console as recovery.
- Expose only the node/TLS ports required by the client. Keep 3x-ui on loopback, behind an authenticated reverse proxy, through an SSH tunnel, or behind a narrowly scoped allowlist.
- Decide deliberately whether IPv6 is enabled. A working IPv6 path with a different geography can create apparent leaks or inconsistent service behavior; disabling it is safer than leaving it half-configured when the user does not need it.
- Enable BBR only after checking kernel support. Verify the active congestion control rather than treating a sysctl write as proof.
- Add fail2ban or an equivalent SSH abuse control when compatible with the distribution, but do not rely on it instead of keys and a firewall.

## 4. 3x-ui/Xray installation

Use the current official project/release documentation at execution time. Do not blindly paste an old one-line installer from an unverified site. Before running an installer:

- inspect the URL, requested privileges, package changes, services, ports, and persistence;
- pin a release or record the exact version;
- take a backup/snapshot;
- keep the panel off the public Internet during initial configuration if possible.

After installation, verify the service and listener independently:

```bash
systemctl status <panel-service> --no-pager
ss -lntup
journalctl -u <panel-service> -n 100 --no-pager
```

Change default panel credentials immediately. Use a long unique password stored in a password manager, a non-obvious panel path only as a secondary measure, and network restriction as the primary measure. If the panel must be reachable remotely, prefer a domain/TLS/reverse-proxy arrangement or an allowlist over a naked high port.

## 5. Node creation

Select a transport based on the actual client and network requirements, not on a copied screenshot. Generate fresh UUIDs/keys for every node. Keep these values in the panel or a protected secret store; do not put them in shell history or a public repository.

Check all of the following as one set:

- server address versus intended hostname/SNI;
- certificate and TLS mode;
- client network and transport compatibility;
- exact listening port and firewall rule;
- UUID/key/flow/alpn/path consistency;
- whether the node is expected to use the VPS egress or an authenticated upstream residential egress.

A reachable TCP port only proves that something accepted a connection; it does not prove a valid client handshake.

## 6. Residential/upstream egress

When a static residential provider is involved, document the topology explicitly:

```text
client -> VPS node -> authenticated residential/upstream proxy -> destination
```

Verify the upstream separately with a minimal request that returns the public egress IP. Test authentication failure, timeout, IPv4/IPv6 preference, and rotation/expiry behavior. Do not label the VPS public address as residential merely because the final egress is residential. If the upstream is down, fail closed or route to a clearly named fallback; do not silently create an open direct path that violates the user's routing intent.

## 7. Subscription generation

Generate a full Mihomo/Clash profile when the user asks for synchronized rules. At minimum, include:

- named proxies and proxy groups;
- a direct group and a proxy group;
- DNS mode and nameservers appropriate to the TUN design;
- private/LAN direct rules;
- domestic direct rules;
- overseas/AI service proxy rules;
- a final fallback that is explicit and documented;
- a subscription token or access control that is treated as a secret.

Validate YAML syntax, load it in a disposable client profile, and check that the client sees individual proxies rather than only a `GLOBAL` group. A node URL and a full rules profile are different deliverables; state which one was produced.

## 8. Evidence required before handoff

Capture redacted evidence for:

```text
server service active: yes/no
expected listeners only: yes/no
firewall rules loaded: yes/no
node handshake: pass/fail
subscription parse/import: pass/fail
observed VPS IP: <redacted if necessary>
observed upstream exit IP: <redacted if necessary>
DNS leak test: pass/fail
WebRTC candidate test: pass/fail/not applicable
private/LAN access: pass/fail
domestic direct access: pass/fail
overseas proxy access: pass/fail
latency/packet loss: <measurement>
rollback tested: yes/no
```

Do not treat a green panel badge, an IP reputation score, or a single website as sufficient evidence.
