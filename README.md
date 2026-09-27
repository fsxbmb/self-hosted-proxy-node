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

## Safety boundary

This skill is intended for infrastructure the user owns or is authorized to administer. It must not be used to create an open relay, access third-party infrastructure without authorization, evade provider limits, or conceal abusive activity.

Runtime secrets must stay outside this repository. Do not commit SSH private keys, passwords, panel tokens, UUIDs, certificates, subscription URLs, residential-proxy credentials, server databases, or live configuration files.

## Layout

- SKILL.md — entrypoint and routing guidance
- references/deployment-runbook.md — server and 3x-ui/Xray deployment
- references/clash-routing-and-leak-tests.md — Clash/Mihomo, DNS, TUN, LAN, and WebRTC checks
- references/operations-and-rollback.md — migration, diagnosis, credential rotation, rollback, and teardown
- agents/openai.yaml — Codex UI metadata
