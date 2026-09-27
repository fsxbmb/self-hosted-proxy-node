# Operations, migration, diagnosis, and rollback

## Safe migration

Use a staged cutover:

1. build the new server and node while the old node remains available;
2. import the new full subscription into a disposable client profile;
3. test DNS, LAN, domestic, overseas, AI-service, and API paths;
4. publish the new subscription or update the controlled config provider;
5. keep the old node for a short rollback window;
6. revoke old UUIDs/keys and remove old subscription entries only after the user confirms the cutover is stable.

Do not delete the old server as part of a repair unless the user has identified the exact instance and explicitly asked for deletion.

## Common symptoms

| Symptom | First checks | Likely layer |
|---|---|---|
| Only `GLOBAL` appears after import | Subscription response content type, YAML structure, proxy list, client profile mode | Subscription/profile format |
| Node is visible but all requests timeout | Firewall, listener, address family, SNI/TLS, UUID/key, provider status | Server or handshake |
| Client shows 502 | Local API listener, `NO_PROXY`, private-route bypass, upstream logs | LAN route or application/upstream |
| DNS appears domestic or inconsistent | DNS hijack, fake-IP mode, bootstrap resolver, IPv6 | DNS/TUN |
| WebRTC shows local/domestic candidates | Browser policy, direct UDP, TUN route, stale tab/process | Browser privacy |
| Cloudflare risk remains high after leak fix | Exit IP reputation, ASN, behavior, fingerprint, account state | Remote service scoring |
| Phone is unstable but computer works | Mobile profile import, MTU, IPv6, cellular path, node group selection | Client/network path |
| Latency jumps above the old node | Compare `connect`, first-byte, and total time; check upstream exit and route | Egress or provider path |

## Credential rotation

Rotate the smallest affected scope:

- exposed subscription URL -> invalidate token and issue a new one;
- exposed panel password -> change it and review panel sessions;
- exposed UUID/key/certificate -> create a new node credential and remove the old one;
- exposed SSH key -> remove its authorized-key entry and replace it at the provider/server;
- suspected residential credential leak -> revoke at the upstream provider and check usage.

Do not merely rename a node after a secret has leaked. Record the old credential as revoked and confirm old clients fail before distributing the new subscription.

## Backup and rollback

Keep dated, permission-restricted backups of:

- SSH and firewall configuration;
- 3x-ui/Xray database/config and panel version;
- reverse-proxy/TLS configuration;
- generated subscription template without live secrets, plus a separate protected secret mapping;
- local Clash profile before replacement.

Rollback means restoring the last known-good config, restarting only the affected service, and verifying the same end-to-end checks. If access is at risk, use the provider console or snapshot rather than repeatedly editing SSH settings over a failing connection.

## Teardown checklist

Before destructive cleanup, identify exact targets by provider ID, hostname, IP, and subscription entry. Then:

1. remove clients or config-provider references;
2. revoke subscription tokens and node credentials;
3. remove panel users/sessions and upstream credentials;
4. stop/disable services and close firewall rules;
5. take a final audit backup only if the user wants recovery;
6. stop, destroy, or release the exact server/static IP requested;
7. verify that the old subscription URL and node no longer work.

Tell the user exactly what was removed and whether any provider snapshot or backup remains recoverable.
