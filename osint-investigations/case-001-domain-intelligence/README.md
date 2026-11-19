# Case 001 — OSINT Domain Intelligence Investigation

**Date:** 19 November 2026  
**Target:** example.com  
**Investigation Type:** Passive Domain Intelligence and Infrastructure Analysis  
**Environment:** Ubuntu Linux (VirtualBox)

## 1. Investigation Objective

To conduct a structured open-source intelligence investigation of the domain `example.com`, examining its DNS records, domain registration information, HTTP response headers and publicly observable infrastructure.

## 2. Tools Used

- Ubuntu Linux Terminal
- `dig` — DNS reconnaissance
- `curl` — HTTP header examination
- `resolvectl` — DNS troubleshooting
- ICANN Registration Data Lookup — RDAP investigation
- IANA — Domain reservation verification

## 3. Methodology

The investigation involved:

1. Examining DNS A, AAAA, NS, MX and TXT records.
2. Identifying DNS infrastructure providers.
3. Reviewing registration data through ICANN RDAP.
4. Examining HTTP response headers using `curl -I`.
5. Comparing observations and identifying investigative limitations.

## 4. Findings

### 4.1 DNS Records

**Observed IPv4 addresses:**

- 104.20.23.154
- 172.66.147.243

**Observed nameservers:**

- hera.ns.cloudflare.com
- elliott.ns.cloudflare.com

The nameserver records indicate the use of Cloudflare DNS infrastructure.

### 4.2 Email Configuration

**MX record:** `0 .`

This is a null MX record, indicating that the domain does not accept email.

**SPF record:** `v=spf1 -all`

The published SPF policy does not authorise any IP addresses to send email for the domain.

An additional TXT record was observed, but its purpose could not be established from the available evidence.

### 4.3 Domain Registration

**Registrar:** RESERVED – Internet Assigned Numbers Authority  
**IANA Registrar ID:** 376  
**Creation Date:** 14 August 1995  
**Expiration Date:** 13 August 2027

**Domain Status Codes:**

- clientDeleteProhibited
- clientTransferProhibited
- clientUpdateProhibited

These status codes indicate restrictions on deletion, transfer and modification of the domain registration.

### 4.4 HTTP Infrastructure Analysis

**Command executed:**

`curl -I https://example.com`

**Observed response:**

- HTTP Status: 200
- HTTP Version: HTTP/2
- Content-Type: text/html; charset=utf-8
- Server: cloudflare
- CF-Cache-Status: HIT
- Age: 9803 seconds
- CF-Ray suffix: MAD
- Alternative Service: HTTP/3 on port 443

The HTTP response indicates that Cloudflare handled the request and served cached content.

The MAD suffix is consistent with Cloudflare's Madrid edge location, but it does not establish the location of the origin server.

## 5. Troubleshooting Observations

Initial DNS queries failed with timeout errors involving the local resolver at `127.0.0.53`.

Network diagnostics confirmed:

- Ubuntu IPv4 Address: 192.168.101.8/24
- Default Gateway: 192.168.101.1
- Network Interface: enp0s3
- Address Assignment: DHCP
- NetworkManager: Connected
- systemd-resolved: Running

Subsequent DNS queries succeeded.

The precise cause of the earlier DNS timeouts was not established.

## 6. Analytical Assessment

The investigation established that:

1. The domain uses Cloudflare DNS infrastructure.
2. The domain publishes records indicating that it does not send or receive email.
3. The domain is reserved for documentation purposes.
4. Cloudflare handled the observed HTTP request.
5. The HTTP response was served from cache.

These findings are consistent with the domain's documented special-purpose role.

## 7. Limitations

- DNS records may change over time.
- IP addresses do not independently establish website ownership.
- HTTP headers may describe intermediary infrastructure rather than the origin server.
- Domain registration information does not establish the identity of individual website operators.
- No active vulnerability scanning or exploitation was conducted.

## 8. Conclusion

The investigation successfully demonstrated passive domain reconnaissance using publicly available DNS, registration and HTTP information.

The exercise strengthened practical skills in OSINT collection, technical verification, infrastructure analysis and evidence-based reporting.

## 9. References

- https://www.iana.org/help/example-domains
- https://lookup.icann.org/en
- https://example.com

**Investigation Status:** Completed  
**Case Reference:** OSINT-001
