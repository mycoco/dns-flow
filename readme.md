# dns-flow

English | [中文](README.zh-CN.md)

dns-flow is a website routing tool built around a single browser entry point.

Its goal is simple: configure one local entry in your browser, then let dns-flow decide whether each website should go direct, through a proxy, through a specified DNS / DoH resolver, or be blocked, based on website rules and resolved IP results.

## Features

- Website routing: match websites by domain rules and assign different access strategies.
- Multiple routing profiles: create profiles such as "Default Routing", "Office Network Routing", or "Test Environment Routing".
- Unified browser entry: each routing profile exposes a local listen address and port for the browser to use.
- DNS routing: use a specified DNS server for selected websites.
- DoH routing: use DNS over HTTPS for selected websites when encrypted DNS queries are preferred.
- Proxy routing: forward selected websites to an HTTP / HTTPS proxy.
- System DNS: fall back to the operating system's default DNS resolver.
- Website blocking: block selected websites directly.
- Post-resolve routing: resolve a domain first with DNS, DoH, or system DNS, then route by the resolved IP result, such as country/region, CIDR range, or address scope.
- Physical network adapter binding: when the resolved target is inside the LAN, prefer a selected local physical adapter to avoid global proxies, VPNs, or virtual adapters.
- hosts mapping: pin selected domains to specific IP addresses before normal DNS resolution.
- Request records: review which rule matched, where the request was routed, and whether the configuration works as expected.
- Configuration import/export: export routing profiles and import them on another device.
- System settings: language, auto start, logging, always-on-top, browser extension integration, software updates, and more.

## How Routing Works

dns-flow puts complex access policies behind one browser entry.

Create a routing profile in dns-flow, for example listening on:

```text
127.0.0.1:5353
```

Then configure your browser or browser extension to use this entry. When the browser visits a website, dns-flow evaluates the domain against your rules and chooses the configured strategy.

A website routing rule usually contains:

- Selected sites: domain expressions to match, such as `*google*` or `*.openai.com`.
- Route action: DNS, DoH, proxy, system DNS, or block.
- Route target: the DNS address, DoH endpoint, or proxy address.
- Priority: higher values match first.
- Remark: a note describing why the rule exists.
- Enabled status: temporarily disable a rule without deleting it.

### Basic Routing

Basic routing chooses the access strategy immediately after a domain rule matches.

Common examples:

- Resolve `*.example.com` with `223.5.5.5`.
- Resolve `*.openai.com` with DoH.
- Route `*test*` through a specified HTTP proxy.
- Block `*ads*` directly.
- Let unconfigured websites continue using system DNS.

### Post-Resolve Routing

Some websites cannot be routed reliably by domain alone, because the same domain may resolve to IP addresses in different countries/regions or network ranges. Post-resolve routing handles this case.

The flow is:

1. Resolve the domain with system DNS, a specified DNS server, or DoH.
2. Read the resolved IP result.
3. Continue routing by IP-based conditions.

Post-resolve conditions support:

- Country/region, such as China, United States, Japan, Singapore, and Hong Kong.
- CIDR ranges, such as `8.8.8.0/24`.
- Address scopes, such as private or public addresses.

Post-resolve actions support:

- Direct: access through the local network.
- Proxy: hand off to a specified proxy.
- Block: stop the request.

Typical scenarios:

- Direct for China IPs, proxy for foreign IPs.
- Prefer the local network for private addresses to avoid global proxy or VPN interference.
- Block resolved results from specific countries/regions.

### Physical Adapter Binding for LAN Access

On many machines, several network interfaces may exist at the same time: physical adapters, VPN adapters, virtual adapters, and proxy tool adapters. If a global proxy or VPN takes over the route, access to LAN resources such as routers, NAS devices, printers, or internal services may fail or take the wrong path.

dns-flow provides "Prefer the local network for LAN access" in the advanced settings of a website routing rule. After enabling it, choose a local network interface. When the matched target is resolved to a LAN address, dns-flow prefers the selected local network for access.

This is useful when:

- Accessing router admin pages, NAS devices, internal Git services, or internal test services.
- Keeping LAN access on the real physical adapter while a global proxy or VPN is enabled.
- Fixing specific LAN traffic to a selected adapter on multi-adapter machines.

To reduce accidental selection, dns-flow prioritizes running local interfaces with IPv4 addresses and filters common virtual, loopback, Docker, WSL, TAP/TUN, WireGuard, Tailscale, and ZeroTier interfaces.

## Usage

### 1. Create a Routing Profile

Open dns-flow, go to "Website Routing", and click "New Routing".

Recommended initial fields:

- Routing name: for example, "Default Website Routing".
- Listen address: keep `127.0.0.1` for personal local use.
- Browser entry port: for example, `5353`; make sure it is not already used by another program.
- Enable this routing immediately after saving: recommended, so the profile starts right after saving.

If LAN devices need to connect to this profile, choose `0.0.0.0`, but enable authentication or source IP restrictions when possible.

### 2. Add Website Rules

Inside the routing profile, click "New Website Rule".

Common configurations:

- Use a specified DNS server for selected websites: choose "Use specified DNS", then enter the domain rule and DNS address.
- Route selected websites through a proxy: choose "Use proxy", then enter the domain rule and proxy address.
- Use encrypted DNS for selected websites: choose "Use encrypted DNS (DoH)", then enter the DoH endpoint.
- Direct for domestic websites and proxy for foreign websites: choose the matching scenario, then configure the resolver and foreign proxy.
- Advanced routing rules: manually choose the route action, target, priority, remark, and post-resolve rules.

If a rule is intended for LAN services, expand advanced settings, enable "Prefer the local network for LAN access", and choose the physical adapter to bind.

Separate multiple websites with semicolons:

```text
*google*;*.openai.com;*github*
```

Wildcard matching is supported:

```text
*.example.com
*keyword*
example.com
```

### 3. Configure Fallback Behavior

If a website does not match any rule, keeping "Handle other sites with system defaults" enabled is recommended.

This lets unconfigured websites continue using system DNS instead of failing because no rule matched.

### 4. Start Routing

After saving the profile, click "Start" in the "Website Routing" list.

When the profile starts successfully, the page shows its running status and listen address. Point your browser to that address to use dns-flow routing.

### 5. Configure the Browser Entry

Configure your browser or browser extension to use the dns-flow listen address.

For example, if the profile listens on:

```text
127.0.0.1:5353
```

the browser-side configuration should point to this local entry. After that, dns-flow routes website access automatically by rule.

### 6. Review Request Records

Open "Request Records" to review:

- Request time
- Domain
- Matched routing profile
- Matched rule
- Route target
- Status

If a website does not use the expected proxy, DNS, or DoH path, check request records first to confirm whether it matched the intended rule.

## Configuration Examples

### Use Ali DNS for Selected Websites

```text
Selected sites: *.example.com
Route action: DNS
DNS address: 223.5.5.5
```

### Use DoH for Selected Websites

```text
Selected sites: *.example.com
Route action: DoH
DoH endpoint: https://dns.google/dns-query
```

### Route Selected Websites Through a Proxy

```text
Selected sites: *.example.com
Route action: Proxy
Proxy address: http://127.0.0.1:7890
```

### Direct for China, Proxy for Other Countries/Regions

```text
Selected sites: *
Resolve first with: System DNS / specified DNS / DoH
Post-resolve rules:
  China: Direct
  Other countries/regions: Proxy http://127.0.0.1:7890
```

### Bind LAN Services to a Physical Adapter

```text
Selected sites: nas.local;*.lan
Route action: System DNS
Advanced settings: enable "Prefer the local network for LAN access"
Local network: choose the physical adapter connected to your router
```

This is useful for accessing NAS devices, routers, printers, and internal systems while a global proxy or VPN is enabled.

## Links

- Download: [mycoco/dns-flow/releases](https://github.com/mycoco/dns-flow/releases)
- Feedback: [mycoco/dns-flow/issues](https://github.com/mycoco/dns-flow/issues)
