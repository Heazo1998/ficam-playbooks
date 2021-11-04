---
layout: page
collection: fpki
title: Path Discovery and Validation
permalink: fpki/pdval/
sticky_sidenav: true
sidenav: fpki

subnav:
  - text: Key Takeaways
    href: '#key-takeaways'
  - text: What Is PDVAL?
    href: '#what-is-pdval'
  - text: Certification Path Discovery
    href: '#certification-path-discovery'  
  - text: Trust Path Validation
    href: '#trust-path-validation'
  - text: What Is Revocation Checking?
    href: '#what-is-revocation-checking'
  - text: PDVAL Troubleshooting
    href: '#pdval-troubleshooting'   
---

## Key Takeaways
- Path Discovery and Validation (PDVAL) is a two-step process for establishing a certification path and checking each digital certificate (an electronic item that contains information used to verify a person’s identity) in the path to see if it is valid. Together, these steps establish trust so that a certificate can be used for a given purpose.
-	Some of the most common PDVAL purposes include: 
    - Giving a person access to a building, an office, or a computer by verifying their [PIV](https://playbooks.idmanagement.gov/piv/){:target="_blank"}{:rel="noopener noreferrer"} credential. 
    - Encrypting data or information.
    - Verifying a digital signature (for example, in signed email and [digitally signed documents](https://playbooks.idmanagement.gov/signword/){:target="_blank"}{:rel="noopener noreferrer"}). 
-	PDVAL occurs every time a certificate is used. It involves building a path from a certificate to a high-level trusted entity known as a trust anchor. 
-	A PDVAL path can be built with a locally configured [trust store](https://playbooks.idmanagement.gov/fpki/trust-stores){:target="_blank"}{:rel="noopener noreferrer"} or it can be done by using public repositories cited in the certificates themselves. A certificate repository is a system that contains certificates and information about them.
-	The best certification path is typically the shortest path most likely to validate (be confirmed as sound or valid).
-	Certification paths can be built in several different ways.
-	If PDVAL finds that a certificate cannot be used to establish trust, the certificate is revoked (rejected) or considered unknown (the status cannot be determined).

## What Is PDVAL?
Before using a public key contained in a certificate, a relying party (an entity that receives and uses identity and credential data from an identity provider and makes access control decisions based on that data, in accordance with the Federal Trust Framework and established federation governance) first has to determine the authenticity of that certificate and, specifically, the validity of all the certificates leading to a trusted public key, called a trust anchor. To do this, a two-step process called PDVAL, also called path processing, is performed:

1.	Trust Path Discovery – Establishes a certification path, which is an ordered list of certificates starting with a certificate that can be validated by one of the relying party's trust anchors and ending with the certificate to be validated (the target certificate).

2.	Trust Path Validation – Checks each certificate in the certification path to see that it has been properly digitally signed, has not expired, and has not been revoked; also includes other checks on things such as name or path constraints, key usage, and extended key usage.

{% include alert-success.html heading="Note" content="Performing PDVAL helps an application make an informed trust decision (for example, determining whether a certificate is appropriate for use in a particular application context)." %}

The diagram below provides an example of a simple certification path, with three tiers in the hierarchy. The Identity Certificate is the target certificate that represents a human subscriber or a device. An example of a target certificate is an end-entity digital signature certificate on a user’s PIV card.  The root certificate is the trust anchor, which is a certification authority's (CA’s) self-signed certificate trusted by other relying parties. The Intermediate Certificates are from CAs that have signed other CA certificates along the path or the target certificate itself. These CAs are often referred to as issuing CAs. There may be multiple Intermediate Certificates in a certification path between the target and the root.

[![Example of a Certification Path including three certificates in a hierarchy](../../assets/fpki/pivcertificatechain_small.png){:style="float:center"}](../../assets/fpki/pivcertificatechain.png){:target="_blank"}{:rel="noopener noreferrer"}

## Certification Path Discovery
Without valid certification paths, certificates cannot be validated and therefore cannot be trusted.  

Path building implementations can be configured in different ways.  Some implementations may be more efficient (e.g., faster to complete), more successful (e.g., return paths more likely to pass validation), or may yield the longest path with the most validity checks.

Ideally, the path building algorithm (a process or set of rules to be followed) is optimized with priorities to guide path decision-making, is capable of building paths successfully regardless of PKI structure, and can find all possible valid paths. 

{% include alert-warning.html heading="Note" content="Even with an ideal, optimized algorithm, more than one path may need to be built before the best path is identified and provided to the validation step." %}

### What Is Best Path?
The best certification path is typically the shortest path most likely to validate. The longer a path becomes, the greater the potential dilution of trust in the certification path. The longer and more complicated a path, the less likely it is to validate because of basic constraints, policies or policy constraints, name constraints, CRL availability, or even revocation.

{% include alert-warning.html heading="Note" content="All user certificates on a PIV (e.g., PIV authentication, PIV digital signature, PIV key encryption) must chain to the Federal Common Policy Certification Authority (FCPCA G2) as the trust anchor to be considered valid." %}

### How Are Certificates Found?
Path building finds certificates in the relying party’s local repositories and cache. It may also use locations specified in the certificates it inspects during live path construction. For example, the algorithm may use: 

- **Authority Information Access (AIA) Extension** which indicates how to access CA information and services for the issuer of a given certificate, this extension is required of all certificates within Federal PKI that are not roots;  
- **cACertificate attribute** of the issuing CA's directory entry to identify where self-issued certificates are stored;
- **issuedToThisCA element** of the cross-certificate pair attribute of that CA's directory entry to identify where all certificates issued to a CA (except for self-issued certificates) are stored; and
- **issuedByThisCA element** of the cross-certificate pair attribute of the issuing CA's directory entry to identify where all certificates issued by a CA to a non-subordinate or peer CA are stored.

Intermediate Certificates may be retrieved by any means available. This includes LDAP, HTTP, SQL, a local cache or certificate store, or as part of the security protocol itself, as is common practice with signed S/MIME messages and SSL/TLS sessions.

{% include alert-warning.html heading="Note" content="A path may be discovered dynamically each time as needed or it may be constructed once and stored or cached. Products that incorporate PDVAL capabilities may vary in how they choose to implement this operation." %}

### How Is a Certification Path Chained Together?
#### Subject/Issuer Name Chaining
Path construction can be done by following the Subject Name/Issuer Name connection between certificates. Moving from the trust anchor to the target certificate, the Subject Name in one certificate must be the Issuer Name in subsequent certificate in the path, and so on.

#### SKID/AKID Chaining
When name chaining is not sufficient, the Authority Key Identifier (AKID) and Subject Key Identifier (SKID) connection can be used to enhance path construction. 

AKIDs distinguish one public key from another when a given CA has multiple signing keys. SKIDs identify certificates that contain a specific public key. Between a trust anchor and a target certificate, the SKID of the first certificate should be the AKID of the next certificate in the path, and so on.

AKID and SKID values are generally SHA1 hashes of the public key of either the issuing CA or the subject certificate itself.

### Forward vs. Reverse Certification Paths
Certification paths can be constructed in the forward direction (from target certificate to trust anchor), the reverse direction (from trust anchor to target certificate), or a combination of both. The decision is based on the PKI structure/environment.

{% include alert-success.html heading="Note" content="Deciding between forward and reverse direction paths can be an important consideration. For example, forward paths may be better suited for hierarchical PKIs and reverse paths may be better suited for distributed PKIs. Sometimes a combination of both paths may be best suited." %}

### Required Inputs for Path Construction
The following are required inputs to the certification path construction process:

1.	**Target Certificate** - The target certificate itself or information to retrieve the target certificate
2.	**Trust List** – The other endpoint of the path; can consist of either:   
  a.	Trusted CA certificates   
  b.	Trusted keys and DNs; a certificate is not necessarily required

### Optional Inputs for Path Construction
Optional inputs can be used to optimize path construction. Some examples are:

1.	The time for which the certificate is to be validated
2.	Initial user acceptable policy set
3.	The complete certification path for the CA  
4.	Collection of certificates that may be useful in building the path

## Trust Path Validation
Trust path validation inspects each certificate that makes up the certification path, examining policies, constraints, and consulting the issuing CA's CRL or OCSP Responder to determine each certificate's validity status at that moment. What constitutes valid is dictated by each PKI. 

Some path validation steps (e.g., checking certificate revocation status) may be performed during certification path discovery to help find the best path sooner rather than later.

{% include alert-success.html heading="Note" content="Even if a trust path is cached, all certificates in the path should be validated in real time at the beginning of each transaction." %}

### Required Inputs for Path Validation
Path validation requires the following inputs:  
- The certificate path to be evaluated, to include all relevant public keys for signature validation
- The current date/time
- The list of certificate policy object identifiers (OIDs) acceptable to the relying party
- The trust anchor of the certificate path
- Indication of whether policy mapping is allowed and if any policy OID is to be tolerated

### Path Validation Steps
Path validation can perform various checks on each certificate in the path. This includes, for example, signature validation, expiration status, revocation status, policy constraints, key usage, and path length.

The submitted path is not approved if any check fails on any certificate. Otherwise, the submitted path is approved.

## What Is Revocation Checking?
Various circumstances may cause a certificate to become invalid prior to expiration of its validity period. Circumstances include name change, employee termination, and compromise or suspected compromise of the private key. Under such circumstances, the CA needs to revoke the certificate.  

There are two common revocation checking methods:
- Certificate Revocation Lists (CRLs)
- Online Certificate Status Protocol (OCSP)

{% include alert-success.html heading="Note" content="Some IT enterprises manage OCSP services internally, and may not allow individual systems reach out to revocation services provided by external PKI owners. This practice can ensure that revocation statuses are current and can also cut down on external traffic; however, this strategy does require investment in OCSP solutions and requires configurations to be ensure OCSP signatures are trusted." %}

### Certification Revocation Lists
A CRL is a digitally signed and time-stamped list of revoked certificates that is signed by a CA or CRL issuer and made freely available in a public repository. Each revoked certificate is identified by its certificate serial number. 

When a certificate-using system uses a certificate (e.g., for verifying a user's digital signature), that system not only checks the certificate signature and validity but also acquires the most recent CRL and checks that the certificate serial number is not on that CRL.  

A new CRL is issued on a regular periodic basis (e.g., hourly, daily, or weekly), and CRLs have their own validity period as defined by the Next Update extension.  Many PDVAL products may cache a CRL by default until it reaches the date/time contained in the Next Update extension.

{% include alert-success.html heading="Note" content="FPKI CRLs are required to be refreshed at least every 18 hours." %}

### Online Certificate Status Protocol
OCSP can provide more timely revocation information than is possible with CRLs and may also be used to obtain additional status information. Other benefits of OCSP over CRLs include smaller file size and greater efficiency for high-volume transactions.

An OCSP client issues a status request to an OCSP Responder and suspends acceptance of the certificates in question until the Responder provides a response. More than one target certificate may be included in a request. A requestor may choose to sign the OCSP request.

An OCSP request includes:
- **Protocol version** – OCSP protocol version
- **Issuing Certificate identifier** – A hash of the issuing CA certificate subject name and public key
- **Target certificate identifier** – Serial number of each certificate in the service request
- **Optional extensions** – Which may be processed by the OCSP Responder (e.g., serviceLocator extension used to route the request to the OCSP server known to be authoritative for the identified certificate)

OCSP returns a definitive response message composed of:
- Version of the response syntax
- Identifier of the responder
- Time when the response was generated
- Responses for each of the certificates in the request
- Optional extensions
- Signature algorithm OID
- Signature computed across a hash of the response

The response for each of the certificates in a request consists of:
- **Target certificate identifier** – The serial number of the certificate
- **Certificate status value** – Specifies whether a certificate is good, revoked, or unknown (if revoked, includes the time at which the certificate was revoked and, optionally, the reason why it was revoked)   
  o	**_Good_** - Certificate currently within its validity interval is not revoked  
  o	**_Revoked_** - Certificate has been revoked, either temporarily or permanently  
  o	**_Unknown_** - OCSP Responder doesn't know about the certificate, usually because the request indicates an unrecognized issuer that is not served by this responder
- **Validity interval** – The validity interval of the response, after which the response is not considered valid
- **Optional extensions** – Adds additional information to the response (e.g., CRLid extension used to indicate the CRL on which a revoked or onHold certificate is found)

{% include alert-warning.html heading="Note" content="The Revoked status indicates that a certificate with the requested serial number should be rejected and the Unknown status indicates that the status could not be determined by this Responder, thereby allowing the relying party to decide whether it wants to try another source of status information, such as a CRL." %}

## PDVAL Troubleshooting

PDVAL issues often occur because of trust store configuration glitches. You can use this [CA certificate bundle](https://github.com/GSA/ficam-playbooks/blob/staging/_fpki/tools/CACertificatesValidatingToFederalCommonPolicyG2.p7b){:target="_blank"}{:rel="noopener noreferrer"}, which includes all CAs within the FPKI landscape, to address interoperability issues generated through trust store misconfiguration.

The bundle has a filename that ends in "p7b." A .p7b file contains only certificates and chain certificates (intermediate CAs), not the private key. See the playbooks and the documents listed below for more information on working with certificates and .p7b files: 

- [Distribute the certificate to operating systems](https://playbooks.idmanagement.gov/fpki/common/distribute-os/){:target="_blank"}{:rel="noopener noreferrer"} - This playbook page is part of the [Federal Common Policy CA Update](https://playbooks.idmanagement.gov/fpki/common/){:target="_blank"}{:rel="noopener noreferrer"} playbook.
- [Distribute the CA certificates issued by the Federal Common Policy CA G2](https://playbooks.idmanagement.gov/fpki/common/certificates/){:target="_blank"}{:rel="noopener noreferrer"}
- [Frequently Asked Questions](https://playbooks.idmanagement.gov/fpki/common/faq/){:target="_blank"}{:rel="noopener noreferrer"}
- [Certificate Trust](https://playbooks.idmanagement.gov/piv/cert-trust/){:target="_blank"}{:rel="noopener noreferrer"}
- [Network Ports and Protocols](https://playbooks.idmanagement.gov/piv/network/ports/){:target="_blank"}{:rel="noopener noreferrer"} 
- [PACS Test Card and Fault Path User Guide](https://www.idmanagement.gov/docs/pacstest-testuserguide.pdf){:target="_blank"}{:rel="noopener noreferrer"} - See Appendix C: Intermediate Certificate Installation.
- [United States Federal PKI X.509 Certification Practice Statement (CPS) for the Federal Public Key Infrastructure (FPKI) Trust Infrastructure](https://www.idmanagement.gov/docs/fpki-fpkima-cps.pdf){:target="_blank"}{:rel="noopener noreferrer"} - See Section 4.3.1, CA Actions During Certificate Issuance.

To help streamline your work, search for "p7b" in the playbook and document content:
1. Press **Ctrl + F.**
2. Type **"p7b"** in the search box near the upper right corner of the page.
3. Click the down arrow to see each instance of the term on the page. 

**Note:** Not all of the playbooks listed above contain information on .p7b files. Among other topics, the playbooks contain background information on working with certificates.
