---
name: self-hosted-proxy-node
description: "Deploy, harden, repair, migrate, and retire authorized self-hosted proxy/VPN nodes on user-owned VPS or cloud servers, including 3x-ui/Xray, residential egress, Clash subscriptions, routing, DNS/WebRTC checks, and rollback."
---

# Self-hosted proxy node

Use this skill when the user asks to create, rebuild, migrate, secure, debug, test, or remove a self-hosted proxy/VPN node on infrastructure they control. It covers the server, the 3x-ui/Xray control plane, an optional authenticated residential/upstream egress, Clash-compatible subscription output, and the user's local routing and leak-prevention configuration.

Do not use it to access infrastructure without authorization, create an open relay, evade provider limits, or hide abusive activity. Keep all credentials, private keys, UUIDs, panel tokens, subscription URLs, and residential-proxy credentials out of the skill files, shell history, logs, screenshots, and final response.

## Operating contract

- Treat the server's public IP and the optional upstream residential exit IP as different addresses. Verify both separately; a static VPS address does not make the exit residential.
- Prefer SSH keys and a non-root administrator. If the provider forbids root login, use the provider-issued account and bootstrap from there. Never ask the user to paste a private key or password into a command.
- Before changing a live server, record the provider, region, OS, public IPv4/IPv6, SSH account, key path, open ports, current node/subscription state, and a rollback point.
- Keep the management panel private: bind it to loopback or protect it with an allowlist, SSH tunnel, or authenticated reverse proxy. Expose only the exact node and TLS ports required by the chosen client configuration.
- Make external mutations in stages: prepare and inspect, apply one change, verify, then continue. Confirm exact targets immediately before destructive actions such as deleting a node, revoking credentials, stopping a server, or replacing a subscription.
- Do not claim that a node is working merely because the panel is reachable. Test SSH, panel health, node handshake, actual exit IP, DNS behavior, client import, routing, and rollback.

## Standard workflow

1. **Build a deployment manifest.** Collect provider/server identity, OS, address family, SSH access method, desired panel and node protocol, domain/TLS status, upstream residential proxy details, client formats, rule requirements, and whether the old node must remain available during migration. Use safe placeholders for secrets.
2. **Access and inventory.** Establish SSH with the supplied key, verify the host and OS, inspect existing listeners/services/routes, and create a dated backup of configuration before editing. Stop if the host identity or ownership is unclear.
3. **Harden first.** Create/test the non-root sudo account, update packages, configure time sync, firewall and SSH policy, close unused listeners, decide the IPv6 policy, and enable BBR only when the running kernel supports it. Keep an already-tested recovery path until the new login is verified.
4. **Install the control plane.** Use the current official 3x-ui/Xray release or documented installation source; inspect scripts before running them and pin a known version when possible. Set unique panel credentials, restrict panel exposure, and store the panel backup securely.
5. **Create the node.** Generate fresh identifiers and keys, choose a transport supported by the server, client, TLS/SNI/domain, and provider, and open only the required port. Do not reuse identifiers from a retired or exposed node.
6. **Attach optional residential egress.** Treat Webshare or another authenticated residential/static proxy as an upstream exit, not as the VPS address. Configure it using the provider's documented protocol and credentials, then verify the exit IP, location, authentication, IPv4/IPv6 behavior, and failure mode before putting it in a client subscription.
7. **Generate the subscription.** Return a complete Clash/Mihomo-compatible configuration when the user expects rules to follow the subscription: proxies, proxy groups, DNS, TUN options, private-network bypasses, domestic direct rules, overseas/AI proxy rules, and a safe final fallback. Validate YAML and import it into a disposable profile before replacing the user's active profile. Read [references/clash-routing-and-leak-tests.md](references/clash-routing-and-leak-tests.md) for the baseline.
8. **Verify end to end.** Check the node from the client, actual external IP, DNS leak behavior, WebRTC candidates, IPv6 consistency, latency and packet loss, domestic direct access, overseas proxy access, private/LAN access, and API endpoints. Record evidence rather than relying on panel status.
9. **Handoff.** Provide the subscription only through a controlled, user-authorized channel; redact tokens in prose and screenshots. State which rules are embedded in the subscription, which are local to a client, what remains untested, and how to revoke the node.
10. **Migrate or retire.** Stage the replacement, switch clients, verify, then revoke old node credentials and remove old subscription entries. For teardown, follow the exact-target checklist in [references/operations-and-rollback.md](references/operations-and-rollback.md).

## Routing and privacy invariants

- Private and link-local networks must remain direct unless the user explicitly requests otherwise: RFC1918 ranges, loopback, link-local, and the user's explicitly named LAN/API addresses.
- Domestic direct routing and overseas proxy routing must be expressed in the subscription if the user expects phone and computer clients to stay synchronized. A client subscribing to a single node URL does not automatically inherit local Clash rules unless the URL serves a full profile or a config provider is configured.
- DNS handling must be checked separately from HTTP proxying. For Mihomo/Clash, use DNS hijacking/fake-IP or an equivalent supported design, keep bootstrap resolution narrowly scoped, and exclude private/local destinations from proxy DNS when required by the LAN topology.
- WebRTC protection is a browser/client concern in addition to DNS. Use the strongest supported browser policy, verify candidate addresses, and explain that an IP reputation/Cloudflare risk score is not itself proof of a WebRTC leak.
- Distinguish a node timeout, a proxy authentication failure, a local route bypass, a DNS failure, an upstream residential failure, and a remote-service block. Fix the narrowest layer first.

## Supporting references

- Read [references/deployment-runbook.md](references/deployment-runbook.md) for the detailed server and 3x-ui/Xray sequence.
- Read [references/clash-routing-and-leak-tests.md](references/clash-routing-and-leak-tests.md) when generating or repairing subscriptions, DNS, TUN, LAN bypasses, or WebRTC protections.
- Read [references/operations-and-rollback.md](references/operations-and-rollback.md) for migration, credential rotation, diagnosis, and deletion.
