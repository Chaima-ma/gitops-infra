````md
# Postmortem: `mlthrive.com` DNS and TLS setup

## Summary

The new domain `mlthrive.com` has been connected to the UpCloud Managed Object Storage static website and is now working over HTTPS.

Final result:

- `https://mlthrive.com/` → **200 OK**
- UpCloud Object Storage state → **running**
- TLS certificate issued successfully
- Main website is accessible on the new domain

The remaining work is to migrate the other services, subdomains and application links that still reference `nightingaleheart.com`.

---

## Problem

The DNS records for `mlthrive.com` were configured in Cloudflare according to the UpCloud static website setup.

The main records were:

- `mlthrive.com` → UpCloud Object Storage endpoint
- `*.mlthrive.com` → UpCloud Object Storage endpoint
- `_acme-challenge.mlthrive.com` → UpCloud ACME validation endpoint

UpCloud successfully verified the DNS configuration, but TLS certificate provisioning repeatedly failed with:

```text
Failed custom domain verification
Failed domain verification for TLS certificate: mlthrive.com, *.mlthrive.com
````

The Object Storage service remained in:

```text
operational_state: setup-checkup
waiting_certificate_issuing
```

---

## Investigation

### 1. Verified domain registration and delegation

Confirmed that `mlthrive.com` was registered through Cloudflare and delegated to the correct Cloudflare authoritative nameservers:

```text
melody.ns.cloudflare.com
sid.ns.cloudflare.com
```

---

### 2. Verified the static website DNS configuration

The new static website was configured against the existing UpCloud Object Storage service and bucket.

UpCloud provided the following ACME validation target:

```text
_acme-challenge.mlthrive.com
    CNAME
_acme-challenge.4b36777d05f59ddfa9167617932ccaefafee33fd4525c9f6.upcloudlb.com
```

The UpCloud authoritative DNS correctly published the current TXT challenge:

```text
XamNeSMdArta1NjK_HaDED19HfEyr8BlJYocZEz_15E
```

---

### 3. Found inconsistent ACME responses

Different public resolvers were returning different TXT values.

The current UpCloud challenge was:

```text
XamNeSMdArta1NjK_HaDED19HfEyr8BlJYocZEz_15E
```

However, Cloudflare/Google resolvers were initially returning two different values:

```text
ET6GleENdmefrtWnhsZDlJ7iCux9byRJZrBGag6O_ms
-GRx5qAbPg6P8CIYEP6SZoeEsEPWmsUIPyPtOqJKTyI
```

Quad9 eventually resolved the expected chain correctly:

```text
_acme-challenge.mlthrive.com
    -> CNAME to UpCloud
    -> TXT XamNe...
```

This showed that the UpCloud side itself was publishing the expected challenge.

---

### 4. Verified UpCloud ACME DNS stability

The UpCloud authoritative nameserver was queried repeatedly over several minutes.

The TXT challenge remained stable throughout the test:

```text
XamNeSMdArta1NjK_HaDED19HfEyr8BlJYocZEz_15E
```

So the issue was not caused by UpCloud continuously changing the active token.

---

### 5. Ruled out CAA and DNSSEC

Checked the new domain for certificate restrictions.

No CAA records were configured, so certificate issuance was not restricted to a specific CA.

No DS record existed at the `.com` delegation level, so broken DNSSEC was not involved.

---

### 6. Identified Cloudflare Universal SSL interference

A direct non-recursive TXT query against the Cloudflare authoritative nameservers showed something important.

Instead of returning only the configured CNAME, Cloudflare was publishing additional TXT validation tokens at:

```text
_acme-challenge.mlthrive.com
```

Example:

```text
ET6GleENdmefrtWnhsZDlJ7iCux9byRJZrBGag6O_ms
-GRx5qAbPg6P8CIYEP6SZoeEsEPWmsUIPyPtOqJKTyI
```

At the same time, a CNAME query still returned:

```text
_acme-challenge.mlthrive.com
    -> _acme-challenge.4b36777d05f59ddfa9167617932ccaefafee33fd4525c9f6.upcloudlb.com
```

Cloudflare Universal SSL was enabled for:

```text
mlthrive.com
*.mlthrive.com
```

The extra TXT records were related to Cloudflare's own certificate validation and were interfering with the external ACME validation used by UpCloud.

---

## Resolution

Cloudflare Universal SSL was disabled for `mlthrive.com`.

After that, the Cloudflare authoritative nameservers stopped returning the additional TXT validation records.

Both authoritative nameservers started returning only the expected CNAME:

```text
_acme-challenge.mlthrive.com
    -> _acme-challenge.4b36777d05f59ddfa9167617932ccaefafee33fd4525c9f6.upcloudlb.com
```

Public resolvers then resolved the complete validation chain correctly:

```text
_acme-challenge.mlthrive.com
    -> UpCloud CNAME
    -> TXT XamNeSMdArta1NjK_HaDED19HfEyr8BlJYocZEz_15E
```

The custom domain was then retriggered in UpCloud.

UpCloud initially reported:

```text
waiting_certificate_issuing
```

and subsequently completed successfully.

Final Object Storage state:

```text
operational_state: running
configured_status: started
```

No remaining state errors were reported.

---

## Verification

HTTPS was tested directly:

```bash
curl -Iv https://mlthrive.com/
```

Result:

```text
HTTP/1.1 200 OK
```

The response was served successfully from UpCloud Object Storage.

---

## Support

An urgent support request was also sent to UpCloud with the domain, Object Storage UUID, ACME validation details and DNS test results.

The issue was resolved before a support response was received.

---

## Root cause

The root cause was a conflict between:

* Cloudflare Universal SSL certificate validation
* UpCloud ACME DNS validation for the Managed Object Storage static website

Cloudflare was publishing its own TXT validation records at `_acme-challenge.mlthrive.com`, while UpCloud expected validation to follow the configured CNAME to its own ACME hostname.

Disabling Cloudflare Universal SSL removed the conflicting validation records and allowed the UpCloud certificate issuance to complete.

---

## Current status

The main website is now available at:

[https://mlthrive.com/](https://mlthrive.com/)

The next step is to audit and migrate the remaining services, DNS records, endpoints and application links still using the old `nightingaleheart.com` domain.

For example:

```text
https://aiwhatif.nightingaleheart.com/
```

currently returns:

```text
DNS_PROBE_FINISHED_NXDOMAIN
```

because the old domain is no longer active.

Next I’ll audit the remaining DNS records, service endpoints and application links, and migrate them to the new domain.

```
```
