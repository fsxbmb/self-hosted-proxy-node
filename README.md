# Self-hosted Proxy Node

Codex skill for deploying, hardening, repairing, migrating, testing, and retiring an authorized self-hosted proxy/VPN node.

The workflow supports both:

- a VPS-only exit;
- a VPS node with an authenticated residential/upstream egress.

It covers 3x-ui/Xray, Clash/Mihomo subscriptions, routing, DNS/TUN, LAN bypasses, WebRTC checks, end-to-end validation, migration, credential rotation, and rollback.

## Install locally

Copy this directory into the Codex skills directory:

~~~bash
mkdir -p ~/.codex/skills
cp -R self-hosted-proxy-node ~/.codex/skills/
~~~

Then invoke it explicitly with:

~~~text
Use $self-hosted-proxy-node to deploy and validate an authorized self-hosted proxy node.
~~~

It can also be selected automatically when the request clearly matches the skill description.

## Required input

The skill does not require one fixed cloud provider or one fixed node protocol. Before changing a server, provide at least:

1. Authorization and server identity: provider, server name or ID, region, OS, and public IPv4/IPv6.
2. Access method: provider console or SSH account plus a local private-key path. Do not paste private-key contents into chat.
3. Node and client requirements: protocol, domain/TLS/SNI status, ports, Clash/Mihomo or other client targets.
4. Egress mode: none for VPS-only exit, or an authenticated HTTP/SOCKS5/provider-specific residential upstream.
5. Routing policy: private/LAN addresses that must stay direct, domestic direct rules, overseas/AI services that must use the proxy, and rollback requirements.

For example:

~~~text
Use $self-hosted-proxy-node for an authorized server.
provider: <provider>
server: <name or ID>
region: <region>
os: <distribution and version>
ssh_account: <account>
ssh_key_path: <local path, or provider console>
node: <protocol and TLS/domain requirements>
upstream_egress: none | authenticated HTTP | SOCKS5 | provider-specific
client_targets: <Clash Verge / mobile Mihomo / other>
rules: <private direct, domestic direct, named overseas/AI services>
rollback: <keep old node / replace after testing / other>
~~~

If there is no residential exit, use upstream_egress: none; the VPS public IP is then the destination's observed exit. If a residential exit is supplied, the skill verifies the VPS and final residential egress separately.

Expected output is a staged deployment record, protected backups, the generated client configuration or subscription, end-to-end test evidence, and rollback instructions. Actual passwords, private keys, UUIDs, panel tokens, subscription URLs, and residential credentials stay outside this repository.

## Safety boundary

This skill is intended for infrastructure the user owns or is authorized to administer. It must not be used to create an open relay, access third-party infrastructure without authorization, evade provider limits, or conceal abusive activity.

Runtime secrets must stay outside this repository. Do not commit SSH private keys, passwords, panel tokens, UUIDs, certificates, subscription URLs, residential-proxy credentials, server databases, or live configuration files.

## Layout

- SKILL.md — entrypoint and routing guidance
- references/deployment-runbook.md — server and 3x-ui/Xray deployment
- references/clash-routing-and-leak-tests.md — Clash/Mihomo, DNS, TUN, LAN, and WebRTC checks
- references/operations-and-rollback.md — migration, diagnosis, credential rotation, rollback, and teardown
- agents/openai.yaml — Codex UI metadata
