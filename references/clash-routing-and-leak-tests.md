# Clash/Mihomo routing and leak tests

This reference describes the intended shape of a full subscription. Adapt syntax to the exact Mihomo/Clash version. Do not copy a rule that the selected client does not support.

## Rule order

Specific rules must precede broad rules. A safe baseline is:

1. loopback, link-local, and private networks -> `DIRECT`;
2. explicitly named LAN/API addresses -> `DIRECT`;
3. explicitly named services that must use the proxy -> proxy group;
4. domestic domain/IP sets -> `DIRECT`;
5. overseas default -> proxy group;
6. an explicit final fallback, never an accidental empty or open relay.

Typical private ranges to consider are `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, and the relevant IPv6 loopback/link-local/ULA ranges. Add the user's actual LAN/API ranges only after confirming their topology. `no-resolve` is useful for IP rules when the client supports it.

For domestic direct routing, use the client's maintained `GEOIP,CN`/`GEOSITE,CN` facilities where available, and keep a small explicit exception list for services the user wants proxied. Do not assume every Chinese-owned domain or CDN IP is safely identified by a country database.

## Common proxy exceptions

If the user asks for the major overseas/AI services to use the proxy, start with narrowly scoped domain suffixes and update them when the service changes. Typical families include OpenAI/ChatGPT, Anthropic/Claude, Google/Gemini, X/Twitter, Bing/Copilot, and xAI/Grok. Include authentication, API, static-content, and upload/download domains only when observed or documented as required. Avoid a blanket `DOMAIN-SUFFIX,com,PROXY` rule.

Keep the rule source in the subscription if phone and computer clients must stay synchronized. A local override in Clash Verge does not automatically travel with a plain node URL.

## DNS baseline

For a TUN profile, choose one coherent DNS design:

- DNS hijacking for port 53 so applications cannot silently bypass the client;
- fake-IP or the client's equivalent when the rule engine expects it;
- encrypted/remote DNS upstreams selected for the user's route;
- narrowly scoped bootstrap resolvers needed to resolve the proxy itself;
- direct resolution/bypass for private and local destinations when the LAN requires it.

Verify both the system resolver and the client resolver. A `dig` query returning a fake-IP range proves interception, not that the final destination route is correct. Test a private hostname and an external hostname separately.

## WebRTC and browser privacy

DNS protection does not automatically prevent WebRTC candidates from exposing a local or real public address. For Chrome, use the supported `disable_non_proxied_udp` IP-handling policy and any already-installed, trusted browser privacy control; verify the result on a WebRTC candidate page. This prevents the common direct-IP leak path but does not remove the WebRTC API. A browser-specific full-disable setting is a separate decision and may break calls.

Do not interpret a Cloudflare/IPPure risk score as a WebRTC test. Check the candidate list and compare it with the intended proxy/upstream exit. A high score can remain because of IP reputation, ASN, behavior, or fingerprint even after the real-IP leak is fixed.

## LAN and local API behavior

TUN mode can intercept more than browser traffic. Preserve explicit direct routes for private networks and the user's API/server address. If a local service returns `502`, inspect in this order:

1. local process is listening and healthy;
2. request target resolves to the intended private address;
3. `NO_PROXY`/bypass rules include the private address;
4. Clash route selects `DIRECT` rather than the proxy group;
5. the upstream service itself is not returning 502;
6. only then inspect TLS, authentication, or application-level errors.

## Verification commands and observations

Use the least invasive checks that answer the question:

```bash
# Local listener and route
ss -lntup
ip route
ip -6 route

# DNS path
dig example.com
dig @127.0.0.1 example.com

# Direct versus upstream exit (use the provider's documented proxy syntax)
curl -4 https://ifconfig.me
curl -4 --proxy <UPSTREAM_PROXY> https://ifconfig.me

# HTTP timing; compare first byte with total time
curl -4 -sS -o /dev/null -w 'connect=%{time_connect} start=%{time_starttransfer} total=%{time_total}\n' https://<target>
```

For client-side checks, record the selected proxy, rule hit, DNS mode, and whether IPv6 was used. Do not include subscription URLs, authorization headers, or proxy credentials in screenshots.
