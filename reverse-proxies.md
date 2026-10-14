# Reverse Proxies

**The one-sentence version:** A reverse proxy is one front door
for many services: it listens on the standard web ports, reads
the *name* each visitor asked for, and forwards the request to
the right machine inside. It also gives you one place to handle
TLS instead of configuring certificates on every service.

## The problem it solves

A home server accumulates services, each on its own port:

| Service | Address you type today |
|---|---|
| Files | `http://192.168.1.10:8000` |
| Media | `http://192.168.1.10:8096` |
| Notes | `http://192.168.1.10:3000` |

Ports are hard to remember, hard to bookmark cleanly, and —
most importantly — **TLS certificates attach to names, not
ports**. You can't practically get and renew a certificate for
every `ip:port` combo. A reverse proxy collapses this to:

| Service | Address you type with a proxy |
|---|---|
| Files | `https://files.home.arpa` |
| Media | `https://media.home.arpa` |
| Notes | `https://notes.home.arpa` |

The proxy listens on ports 80 and 443 (the standard web
ports), looks at the hostname in each request, and forwards it
to the right internal `address:port`. The browser never sees
the ports. Combine with [local DNS records](names-on-your-lan.md)
and the names resolve inside your house.

**Terminology check:** a *forward* proxy sits in front of
*clients* (your browser asking the proxy to fetch the web); a
*reverse* proxy sits in front of *servers* (the web asking the
proxy, which fetches from your services). This chapter is only
about the reverse kind.

## Caddy: the simple one

**Caddy** is a web server whose headline feature is **automatic
HTTPS**: for a public domain name, it obtains and renews TLS
certificates from Let's Encrypt on its own, and redirects HTTP
to HTTPS without being asked. Its config file (a `Caddyfile`)
reads almost like the table above.

This Caddyfile was validated with the real Caddy binary
(`caddy validate` exited 0 — adapted cleanly, no errors):

```text
{
	admin off
}

files.home.arpa {
	reverse_proxy 192.168.1.10:8000
}

media.home.arpa {
	reverse_proxy 192.168.1.10:8096
}
```

That's the whole thing: two named sites, each forwarded to the
service's internal address and port. `admin off` disables
Caddy's admin API (a remote-control port you don't need on a
home box — leaving it on is a needless open door).

**The honest TLS caveat:** automatic public certificates only
work for names the public internet can resolve — Let's Encrypt
has to reach your server to verify you own the name. For
internal-only names like `*.home.arpa`, there's nothing public
to verify, so Caddy instead offers an **internal CA**
(`caddy trust` installs it on your machines) or DNS-based
verification for names you own publicly. Either way, the
concept is the same — the proxy terminates TLS — but
\"automatic\" has limits inside a LAN. Caddy's docs cover both
paths (not verified in this environment).

## Nginx: the ubiquitous one

**Nginx** is the web server half the internet runs on. More
powerful, more verbose, and the config language shows it. The
unit of config is the **server block**: one per name you serve.
This server block was validated with the real nginx binary —
wrapped in a minimal config, `nginx -t` reported \"syntax is
ok\" and \"test is successful\":

```text
server {
	listen 80;
	server_name files.home.arpa;

	location / {
		proxy_pass http://192.168.1.10:8000;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
	}
}
```

Line by line:

- `listen 80` — take HTTP traffic on the standard port.
- `server_name` — this block handles requests *for this name*.
  A second block with `server_name media.home.arpa;` and a
  different `proxy_pass` gives you the second service.
- `proxy_pass` — where requests actually go.
- The two `proxy_set_header` lines — pass the original hostname
  and the visitor's real address through; without them, the
  backend service sees every visitor as \"the proxy\" and
  can't tell which name was requested.

**Check:** after reloading nginx, a request for the name should
reach the backend. From the proxy machine:

```bash
curl -s -o /dev/null -w "%{http_code}\n" -H "Host: files.home.arpa" http://localhost/
```

```text
200
```

(Example — `200` means the backend answered through the proxy.
Any `5xx` means the proxy is up but the backend isn't; `connection
refused` means nginx itself isn't listening. The `-H` flag fakes
the hostname so you can test before DNS is set up.)

Nginx doesn't do automatic certificates itself; the usual
pairing is **certbot** (Let's Encrypt's client) for public
names. Same caveat as Caddy applies for internal-only names.

## Which to pick

| | Caddy | Nginx |
|---|---|---|
| Config style | A few lines per site | A server block per site |
| TLS | Automatic for public names | Via certbot, you wire it up |
| Best for | \"I want HTTPS with minimal fuss\" | \"I already know nginx\" or need its advanced features |
| Footprint | Single static binary | Package install, more knobs |

Both solve the same problem. If you're starting from zero,
Caddy's brevity is the kinder teacher; the concepts (listen,
match the name, forward to the backend) transfer directly to
nginx later.

## What a reverse proxy is not

- **Not a security boundary by itself.** It doesn't
  authenticate anyone — anyone who can reach it can ask it for
  your services. Put authentication on the services or the
  proxy, and think hard before exposing the proxy to the
  internet ([Port Forwarding](port-forwarding.md)).
- **Not a VPN.** It doesn't encrypt your whole connection or
  hide your traffic — it serves specific named services.
- **Not required.** If you have one service, bookmark the
  `ip:port` and skip this chapter. The proxy earns its keep at
  two-plus services, or the moment you want real HTTPS.

**Next:** [Port Forwarding](port-forwarding.md)
