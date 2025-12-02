Why Cloudflare would return 421 for this request

SNI vs Host/:authority mismatch

During the TLS handshake the client sends an SNI hostname. Cloudflare uses that SNI to pick which certificate and which origin routing to apply.

Your HTTP/2 request shows Host: evil.com (which maps to the :authority pseudo-header in HTTP/2). If the TLS SNI sent during the handshake is not evil.com (for example the client used an IP, another domain, or no SNI), Cloudflare may decide the request targeted the wrong origin and reply 421.

HTTP/2 :authority

In HTTP/2 the Host: header is represented as :authority. Well-behaved HTTP/2 clients set that automatically. If there's a mismatch between SNI and :authority, edge will treat it as misdirected.

Hitting the wrong Cloudflare edge / origin IP

If you sent the request to a Cloudflare IP that isn’t configured for evil.com (scanning by IP, or using an old/incorrect /etc/hosts mapping), Cloudflare will reject it with 421 instead of routing to the intended origin.

HTTP version / ALPN differences

If ALPN negotiation chooses a protocol the backend doesn’t expect (rare), it can also cause routing oddities. Usually the root cause is SNI/Host mismatch though.

How to reproduce (safe) and confirm it’s client-side

Correct (normal) request — this should not trigger 421 if DNS/SNI match:

curl -I --http2 https://evil.com/v2/event/trigger-events/tomorrowland/web


Simulate a bad client that connects to a Cloudflare edge IP but sends a Host header for evil.com while SNI is different (this often produces 421). Example using openssl s_client to show SNI vs direct IP behavior:

# connect by IP (no SNI) — this often produces a mismatch at Cloudflare
openssl s_client -connect <CLOUDFLARE_EDGE_IP>:443

# connect by IP but set the SNI to evil.com (this sets the SNI properly)
openssl s_client -connect <CLOUDFLARE_EDGE_IP>:443 -servername evil.com


If -servername evil.com fixes the behavior, you just demonstrated an SNI problem.

Another practical curl trick (map domain -> IP while preserving SNI):

# map evil.com to a specific IP but keep SNI as evil.com
curl -I --http2 --resolve evil.com:443:<CLOUDFLARE_EDGE_IP> https://evil.com/v2/event/trigger-events/tomorrowland/web


If the above works but curl -I https://<CLOUDFLARE_EDGE_IP>/v2/... -H "Host: evil.com" returns 421, that proves Cloudflare requires SNI/:authority alignment.

How to fix it (when you control the client)

Always set SNI to the hostname you intend (most modern clients do this automatically).

Use the domain in the request URL rather than the raw IP, or use --resolve in curl so SNI is preserved.

Ensure the HTTP/2 client populates :authority (most libraries do; custom HTTP/2 stacks can miss it).

If you’re scanning/automating, make your scanner pass correct SNI and :authority values per-target rather than sending raw IP requests with a Host: header.

Is it reportable as a security issue?

No. A Cloudflare 421 is expected behavior and not a leakage of sensitive information. It’s a routing protection. Only if you observe internal data leaked (stack traces, private headers, backend IPs) tied to different hostnames would it become interesting. The generic 421 itself is not a vuln.

Short checklist you can run now

Run the working curl command:

curl -I --http2 https://evil.com/v2/event/trigger-events/tomorrowland/web


If you were scanning by IP, reproduce the 421 with:

curl -I --http2 https://<CF_EDGE_IP>/v2/event/trigger-events/tomorrowland/web -H "Host: evil.com"


Then try again with --resolve to preserve SNI:

curl -I --http2 --resolve evil.com:443:<CF_EDGE_IP> https://evil.com/v2/event/trigger-events/tomorrowland/web


Use openssl s_client -connect <IP>:443 -servername evil.com to confirm SNI fixes it.
