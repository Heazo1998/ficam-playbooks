---
layout: page
collection: piv
title: Network Ports and Protocols
permalink: piv/network/ports/
sticky_sidenav: true
sidenav: pivnetwork

subnav:
  - text: Verifying and Troubleshooting
    href: '#verifying-and-troubleshooting'
  - text: Web Services for Validating PIV Certificates
    href: '#web-services-for-validating-piv-certificates'
  - text: Web Services for the Federal Public Key Infrastructure
    href: '#web-services-for-the-federal-public-key-infrastructure'
---

Your workstations, servers, network domain controllers, and applications need to validate the [revocation](../../cert-trust/#revocation) status of the PIV certificates and all intermediate certificate authority (CA) certificates.  In addition, the [certificate chain](../../cert-trust/#certificate-chains) path building may retrieve and download the intermediate CA certificates.

The validation occurs in real time (with some caching) and requires ensuring network traffic is open and available to the destination web services, ports, and protocols.  Many U.S. federal agencies implement a layered network security model with demilitarized zones (DMZs), proxies, and Trusted Internet Connections (TICs) to monitor, defend, and protect the networks, applications and users.

## Verifying and Troubleshooting
Non-accessible endpoints for the web services due to firewalls blocking access is a very common root cause for errors.  If you encounter user errors including "Cannot validate" and similar domain controller errors, your first troubleshooting step should be to verify your network and access.

{% include alert-info.html heading = "nslookup and certutil are your friendly tools" content="Restricted or denied access to Internet web services including the OCSP and CRL web services used in the certificate validations lead to common errors and issues.  Collaborate with your Network Engineers to review the web services, IP addresses, ports and protocols, and verify access from all local and wide-area network segments." %}

It is simple to begin troubleshooting if the web services endpoints are accessible or blocked by firewall rules.  You have the basic four utility tools for troubleshooting:

- certutil (Microsoft)
- openssl
- nslookup
- tracert


For the typical network domain, _certutil_ will be your best option to identify a number of possible root causes.  There are many options available in the _certutil_ utility tool, and two are covered here.

Export your _public_ key and certificate for PIV Authentication to a .cer file (mypiv_auth.cer), and run the following command in a command line from workstation(s) *and* domain controller(s):

```
  certutil -verify -urlfetch mypiv_auth.cer >>verify_piv.txt
```

The text file output will include a *full* check against all options for CRLs, OCSP, intermediate certificates to verify a trust chain, and the root (COMMON).  Review all items and ensure at least one successful verification message is included for _each check_.  You may see errors for the LDAP verifications and these can be ignored if a CRL or OCSP check is successful.

{% include alert-warning.html heading = "Time is important" content="When reviewing the verification messages, you should pay careful attention to the time.  For example, if a CRL file is not downloaded in under 15 seconds, it is very likely that you will encounter network authentication errors and will need to perform some tuning." %} 

There is also a graphical user interface to help perform these verification checks.

```
  certutil -v -url mypiv_auth.cer
```
The graphical user interface allows you to check OCSP, CRL, and AIA (intermediate certificate retrievals).

## Web Services for Validating PIV Certificates

[Revocation](../../cert-trust/#revocation) status is validated using using either Online Certificate Status Protocol (OCSP) or Certificate Revocation Lists (CRLs). To meet your initial network requirements, you should ensure the OCSP and CRL URLs included in *your agency* users' [PIV Authentication certificates](../../details/#view-your-piv-credential-certificates) are accessible from all workstations and domain controllers.

| Type | Certificate Extension | Protocol (Port) | Considerations|
| ----- | -------| -------| ------|
| OCSP | Authority Information Access | HTTP (80) | All PIV certificates have OCSP references and OCSP URLs which are Internet accessible and provided by the issuing CA. Intermediate CAs are **not** required to have OCSP available for the _intermediate_ certificates.|
| CRL  | CRL Distribution Point (CDP) | HTTP (80) | All PIV certificates have CRL capabilities provided by the issuing CA.  All intermediate CA certificates have CRL capabilities.  CRL files have an expiration time that varies between 6 to 18 hours. CRL file sizes range from a few kilobytes to more than 30 megabytes (MB).

Lightweight Directory Application Protocol (LDAP) for retrieving information is not preferred and has been increasingly deprecated; therefore, LDAP is not included.

There are dozens of OCSP and CRL URLs for *all* issued PIV credentials.  If you have users with PIV credentials from other agencies or partners, identifying all the URLs to verify against your network configurations will be more complex.

## Web Services for the Federal Public Key Infrastructure

The Federal Common Policy Certificate Authority (COMMON) is the root certificate authority and has web services to publish both [certificate chains](../../cert-trust/#certificate-chains) (p7b files) and [CRLs](../../cert-trust/#revocation) for all intermediate certificate authorities which the root signs.

To enable communications with these Federal Common Policy Certificate Authority services, including those currently operational and any expansion, you should verify outbound communications to the base domain of http.fpki.gov. For example, a successful connection to http://http.fpki.gov/fcpca/fcpca.crt will download a copy of the Federal Common Policy CA certificate.

You should consider allowing two protocols (port): HTTP (80) and DNS (53).  Although the web services for publishing CRLs are not currently served over HTTPS (443), you may want to allow HTTPS (443) to future proof for any expansion.
