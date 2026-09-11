---
title: "Requirements and Gaps for Post-Quantum Certificate Rotation in Multi-Tenant Public Key Infrastructure Environments"
abbrev: "Multi-Tenant PKI PQC Requirements"
docname: draft-vicente-pquip-multitenant-pki-requirements-01
category: info
submissiontype: independent
ipr: trust200902
area: Security
workgroup: Post-Quantum Use In Protocols
keyword:
  - post-quantum
  - PQC
  - PKI
  - multi-tenant
  - certificate rotation
  - ACME
  - CNSA 2.0

author:
  -
    ins: B. Vicente
    name: Brian Vicente
    organization: Sanctum SecOps LLC
    email: bvicente@sanctumsecops.com
    uri: https://orcid.org/0009-0006-6395-5308
    city: Pine City
    region: NY
    country: United States of America

normative:
  RFC2119:
  RFC8174:
  RFC5280:
  RFC8555:
  RFC7696:
  RFC9763:
  RFC9794:
  RFC5272:

informative:
  MOSCA:
    title: "Cybersecurity in an Era with Quantum Computers: Will We Be Ready?"
    author:
      - ins: M. Mosca
    seriesinfo:
      IEEE Security and Privacy: "16(5):38-41"
    date: 2018
  CNSA20:
    title: "Commercial National Security Algorithm Suite 2.0"
    author:
      - org: NSA
    seriesinfo:
      NSA: CNSA 2.0
    date: September 2022
  NIST-PQC:
    title: "Post-Quantum Cryptography Standards: FIPS 203, 204, 205"
    author:
      - org: NIST
    seriesinfo:
      NIST: FIPS 203/204/205
    date: August 2024

--- abstract

Organizations operating Public Key Infrastructure (PKI) across multiple isolated tenant environments face a critical gap: existing PKI management protocols and standards do not address the coordination requirements for post-quantum cryptographic (PQC) algorithm migration in shared, multi-tenant certificate authority deployments. This document identifies the functional requirements and open protocol gaps that must be addressed to enable safe, consistent, and auditable PQC certificate rotation across multi-tenant PKI environments. No new protocol mechanisms are specified; this is an informational requirements document intended to motivate future standards work.

--- middle

# Introduction

The publication of NIST FIPS 203 (ML-KEM), FIPS 204 (ML-DSA), and
FIPS 205 (SLH-DSA) in 2024 has initiated a global transition away
from quantum-vulnerable cryptographic algorithms toward post-quantum
alternatives.  For organizations operating certificate authority (CA)
infrastructure, this transition requires replacing classical key
exchange and signature algorithms across all issued certificates,
OCSP responders, CRL signing keys, and TLS endpoints before
applicable regulatory deadlines.

The NSA Commercial National Security Algorithm Suite 2.0 (CNSA 2.0)
establishes concrete migration deadlines: PQC algorithms are required
for software and firmware signing by 2027, for TLS and certificate
infrastructure by 2029, and classical algorithm use is to be retired
by 2033.

Multi-tenant PKI deployments — where a single CA platform issues
certificates for multiple independent organizational tenants with
isolated trust anchors and policy domains — present unique
coordination challenges not addressed by existing IETF protocols.
This document describes those challenges and derives functional
requirements for a compliant solution.

## Requirements Language

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and
"OPTIONAL" in this document are to be interpreted as described in BCP
14 [RFC2119] [RFC8174] when, and only when, they appear in all
capitals.

# Problem Statement

Existing PKI management protocols — including the Automatic
Certificate Management Environment (ACME) [RFC8555], Certificate
Management over CMS (CMC) [RFC5272], and the base X.509 profile
[RFC5280] — were designed for single-tenant or hierarchically-managed
CA environments.  They do not specify:

*  Mechanisms to detect and report algorithm configuration divergence
   between the CA policy and the algorithms in use across active
   tenant certificate populations (configuration drift).

*  Procedures to ensure that a certificate issuance transaction
   maintains semantic consistency with the issuing tenant's declared
   cryptographic policy at the time of issuance.

*  Protocols for coordinating the sequencing of certificate rotation
   actions across tenants in a manner that accounts for network
   topology and service dependency constraints.

*  Audit mechanisms that provide tenant-isolated, per-transaction
   evidence of algorithm compliance at issuance time.

Without addressing these gaps, a multi-tenant PKI operator has no
standardized mechanism to guarantee that all tenants have completed
PQC migration in a coordinated, consistent, and auditable manner
before regulatory deadlines.

# Terminology

Multi-Tenant PKI:  A PKI deployment in which a single platform

   instance hosts certificate authority services for multiple
   independent organizational tenants, each with isolated certificate
   policies, trust anchors, and subscriber populations.

PQC Migration:  The process of replacing quantum-vulnerable
   cryptographic algorithms (e.g., RSA, ECDSA, ECDH) with post-
   quantum cryptographic algorithms standardized by NIST (e.g., ML-
   KEM, ML-DSA, SLH-DSA).

Configuration Drift:  A condition in which the cryptographic
   algorithm configuration of one or more active certificates or CA
   components diverges from the declared cryptographic policy of the
   tenant or platform operator.

Hybrid Transitional Configuration:  A cryptographic configuration
   that combines classical and post-quantum algorithms (e.g., X25519
   with ML-KEM-768, ECDSA-P256 with ML-DSA-65) as defined in
   [RFC9794].

CRQC:  Cryptographically Relevant Quantum Computer.  A quantum
   computer capable of running Shor's algorithm at sufficient scale
   to break RSA-2048 and ECDH-P256 in practical time.

# Scope and Limitations of Existing Standards

## ACME (RFC 8555)

ACME [RFC8555] automates the issuance, renewal, and revocation of
certificates by specifying challenge-response domain ownership
verification.  ACME does not specify:

*  Enforcement of algorithm policy at the per-issuance-transaction
   level.

*  Detection of divergence between a certificate's algorithm and the
   issuing CA's current declared policy.

*  Coordination protocols for rotating certificates across multiple
   tenants in dependency order.

## X.509 and RFC 5280

[RFC5280] defines the X.509 certificate and CRL profile.  It
specifies certificate structure and validation, but does not address:

*  Real-time detection of certificates whose algorithms are
   inconsistent with a CA's current policy.

*  Mechanisms to gate certificate issuance based on a policy-
   compliance precondition.

## Cryptographic Algorithm Agility (RFC 7696)

[RFC7696] provides guidelines for implementing algorithm agility in
IETF protocols — specifically, the ability to select and negotiate
cryptographic algorithms without hard-coded dependencies.  It does
not specify:

*  How a CA system should enforce algorithm agility requirements
   uniformly across all tenants in a multi-tenant deployment.

*  How consistency of algorithm selections across a distributed
   certificate population should be monitored or enforced.

## Related Certificates (RFC 9763)

[RFC9763] defines the RelatedCertificate X.509 extension, which
allows two certificates with different algorithms to be
cryptographically linked.  This supports dual-algorithm operation
during PQC transition but does not address the coordination and
scheduling concerns identified in this document.

## Hybrid Scheme Terminology (RFC 9794)

[RFC9794] establishes terminology for post-quantum and traditional
hybrid cryptographic schemes.  This document uses that terminology
but addresses a separate problem: the operational management gap in
migrating multi-tenant PKI systems to PQC.

# Functional Requirements

A solution addressing the gaps identified in Section 3 SHOULD satisfy
the following functional requirements:

## Algorithm Policy Consistency

REQ-1: The system MUST be capable of detecting, for each active
certificate in a tenant's issued certificate population, whether the
certificate's algorithms are consistent with the tenant's current
cryptographic policy.

REQ-2: The system MUST provide a per-tenant view of algorithm
consistency across the entire active certificate population.

## Issuance Integrity

REQ-3: The system SHOULD support a mechanism by which a certificate
issuance request can be evaluated for compliance with the issuing
tenant's current algorithm policy before the certificate is issued.

REQ-4: The system SHOULD maintain per-issuance-transaction audit
records sufficient to demonstrate that each issued certificate was
algorithm-compliant at the time of issuance.

## Rotation Coordination

REQ-5: The system MUST provide a mechanism for ordering certificate
rotation actions across a multi-tenant environment to avoid service
disruption caused by rotating certificates in an order that violates
trust chain or service dependency constraints.

REQ-6: The system SHOULD support awareness of the network topology
context in which certificates are deployed to inform the sequencing
of rotation operations.

## Compliance Reporting

REQ-7: The system MUST support mapping of each tenant's algorithm
posture against applicable compliance deadline frameworks (e.g., CNSA
2.0) and provide gap reports identifying certificates and CA
components that require migration before specific deadlines.

REQ-8: The system SHOULD provide aggregate and per-tenant compliance
progress metrics suitable for regulatory reporting.

# IPR Considerations

The author and Sanctum SecOps LLC hold or have applied for patents covering
subject matter related to this document. Specifically, the subject matter
herein overlaps in part with the disclosure of U.S. Provisional Patent
Application No. 64/080,137 ("Multi-Tenant Public Key Infrastructure with
Drift-Gated Issuance, Per-Transaction Semantic Consistency Verification, and
Topology-Aware Post-Quantum Certificate Rotation", filed 2026-06-01) and U.S.
Patent Application No. 19/698,870 ("System and Method for Post-Quantum
Cryptographic Readiness Observability and Feedback in Networked Environments",
attorney docket SANC-2026-002, filed 2026-06-05).

Disclosure of any such patents will be made in accordance with the procedures
defined in BCP 79. Publication of this Internet-Draft does not constitute any
patent license, express or implied, from the author or Sanctum SecOps LLC.
License terms, if any, are not yet known.

This work product is the original work of the named author and is offered to
the IETF community as an Independent Submission. No portion of this document
is offered as a trade secret. All technical disclosures herein are intended
as public prior art as of the publication date of the initial -00 revision.

# Patent-Pending Claim Scope (Informational)

The following claim families are subject to the pending United States patent
applications identified above. This summary is provided to inform implementers
of the protected scope; it is not an enabling disclosure and does not waive
any patent right. The specific algorithms, parameter values, weights,
thresholds, predicates, sequencing rules, and binding logic that practice
these claims are not disclosed in this Internet-Draft.

Family A — Policy-to-CA Compiler: apparatus, method, and non-transitory
computer-readable medium claims directed to compiling a declarative
multi-tenant cryptographic policy into per-tenant certificate-authority
configuration in a manner that preserves tenant isolation and produces
auditable evidence of compilation correctness.

Family B — Drift-Gated Certificate Issuance with Per-Transaction Semantic
Consistency: apparatus, method, and computer-readable medium claims directed
to gating certificate issuance on detection of configuration drift between a
tenant's declared cryptographic policy and the cryptographic material proposed
for issuance.

Family C — Crypto-Agile Post-Quantum Migration with Topology-Aware Rotation:
apparatus, method, and computer-readable medium claims directed to
coordinating post-quantum certificate rotation across multi-tenant PKI
deployments in a manner that accounts for service-dependency topology.

Family D — Policy-Linked Audit Engine: apparatus, method, and
computer-readable medium claims directed to producing tenant-isolated,
per-transaction audit records that cryptographically bind certificate
issuance, renewal, and rotation events to the policy version in force.

Nothing in this Internet-Draft is to be construed as disclosing the manner
in which any of the above claims is reduced to practice.

# Author Credentials

The author holds the following industry certifications, independently
verifiable through Pearson Credly at <https://www.credly.com/users/brian-vicente>:

* CompTIA Security+ ce (2026-03-29 / valid through 2029-03-29)
* CompTIA Secure Infrastructure Specialist - CSIS Stackable (2026-03-29 / valid through 2029-03-29)
* CompTIA Secure Cloud Professional - CSCP Stackable (2026-03-29 / valid through 2027-06-29)
* CompTIA Cloud Admin Professional - CCAP Stackable (2025-10-10 / valid through 2027-06-29)
* CompTIA IT Operations Specialist - CIOS Stackable (2025-10-10 / valid through 2029-03-29)
* CompTIA Network+ ce (2025-10-10 / valid through 2029-03-29)
* CompTIA Cloud+ ce (2024-06-29 / valid through 2027-06-29)
* Linux Professional Institute Linux Essentials (2024-01-16)
* CompTIA A+ ce (2023-10-16 / valid through 2029-10-16)

# Security Considerations

The primary threat model motivating this document is the harvest-now,
decrypt-later (HNDL) attack, in which an adversary captures
ciphertext protected by quantum-vulnerable algorithms today with the
intention of decrypting it once a CRQC becomes available.  Long-lived
certificates and CA signing keys that remain in use beyond the
anticipated CRQC arrival window are particularly exposed.

Mosca's inequality [MOSCA] provides a practical framework for urgency
assessment: if the sum of the time required to complete PQC migration
and the remaining confidentiality lifetime of sensitive data exceeds
the estimated time to CRQC availability, then migration is overdue.
A multi-tenant PKI environment amplifies this risk because a single
unrotated CA signing key may protect the entire trust anchor for
multiple tenants.

Any mechanism that gates certificate issuance based on policy
compliance introduces a denial-of-service vector: a misconfigured or
overly restrictive policy could block legitimate certificate
issuance.  Implementations MUST provide auditable override mechanisms
and alerting to prevent silent issuance failures.

Multi-tenant isolation MUST be preserved at the algorithm policy
layer: a drift condition in one tenant MUST NOT trigger issuance
blocks in other tenants.

# IANA Considerations

This document has no IANA actions.  Future work specifying protocol
extensions to address the requirements in Section 5 may require IANA
registration of new ACME extensions, X.509 extensions, or CMS
attributes.

# References

## Normative References

[RFC2119]  Bradner, S., "Key words for use in RFCs to Indicate
           Requirement Levels", BCP 14, RFC 2119, March 1997,
           <https://www.rfc-editor.org/rfc/rfc2119>.

[RFC8174]  Leiba, B., "Ambiguity of Uppercase vs Lowercase in RFC
           2119 Key Words", RFC 8174, May 2017,
           <https://www.rfc-editor.org/rfc/rfc8174>.

[RFC5280]  Cooper, D., "Internet X.509 PKI Certificate and
           Certificate Revocation List (CRL) Profile", RFC 5280, May
           2008, <https://www.rfc-editor.org/rfc/rfc5280>.

[RFC8555]  Barnes, R., "Automatic Certificate Management Environment
           (ACME)", RFC 8555, March 2019,
           <https://www.rfc-editor.org/rfc/rfc8555>.

[RFC7696]  Housley, R., "Guidelines for Cryptographic Algorithm
           Agility and Selecting Mandatory-to-Implement Algorithms",
           RFC 7696, November 2015,
           <https://www.rfc-editor.org/rfc/rfc7696>.

[RFC9763]  Ounsworth, M., "Related Certificates for Use in Multiple
           Authentications within a Protocol", RFC 9763, April 2025,
           <https://www.rfc-editor.org/rfc/rfc9763>.

[RFC9794]  Hale, N., "Terminology for Post-Quantum Traditional Hybrid
           Schemes", RFC 9794, June 2025,
           <https://www.rfc-editor.org/rfc/rfc9794>.

[RFC5272]  Schaad, J., "Certificate Management over CMS (CMC)",
           RFC 5272, June 2008,
           <https://www.rfc-editor.org/rfc/rfc5272>.

## Informative References

[MOSCA]    Mosca, M., "Cybersecurity in an Era with Quantum
           Computers: Will We Be Ready?", IEEE Security and
           Privacy 16(5):38-41, 2018.

[CNSA20]   NSA, "Commercial National Security Algorithm Suite 2.0",
           NSA CNSA 2.0, September 2022.

[NIST-PQC] NIST, "Post-Quantum Cryptography Standards: FIPS 203, 204,
           205", NIST FIPS 203/204/205, August 2024.

--- back
