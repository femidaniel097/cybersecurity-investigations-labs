# OSINT-002 — Source Verification and Digital Footprint Investigation

## 1. Investigation Overview

**Date:** 26 November 2026  
**Learning Day:** Day 10  
**Category:** OSINT and Digital Investigations  
**Target Organisation:** Canonical  
**Methodology:** Passive Open Source Intelligence  
**Status:** Practical completed

This investigation focuses on collecting, evaluating, corroborating and verifying publicly available information about an organisation using advanced search operators, corporate publications, external sources and government-maintained public records.

## 2. Investigation Objectives

- Conduct organisational digital footprint investigations.
- Apply advanced search engine operators.
- Discover publicly indexed documents and subdomains.
- Distinguish primary, secondary and tertiary sources.
- Evaluate source credibility and independence.
- Trace information to its original source.
- Investigate government-maintained public records.
- Identify and resolve false-positive findings.
- Assign appropriate confidence levels to intelligence findings.

## 3. Investigation Question

Is Canonical the publisher of Ubuntu, and can publicly available evidence corroborate its organisational identity and historical trademark registration?

## 4. Tools and Resources

- Google Search
- Web browser
- Canonical corporate website
- Ubuntu website
- Lenovo Press
- Wikipedia
- UK Intellectual Property Office trademark registry

The investigation was conducted using publicly accessible information without intrusive scanning or unauthorised access.

## 5. Advanced Search Operator Investigation

### Query 1 — General Information Discovery

```text
Canonical Ubuntu
```

Purpose:

To establish the organisation's general digital footprint and identify relevant corporate and product information.

Observed results included:

- Corporate website.
- Careers pages.
- Ubuntu operating system information.
- Company information.
- Professional certification resources.

### Query 2 — Domain-Restricted Search

```text
site:canonical.com "Ubuntu"
```

Purpose:

To restrict search results to the organisation's domain and associated subdomains.

Observed results included:

- Ubuntu Blog.
- CVE-related pages.
- Technical documentation.
- Ubuntu on Windows documentation.
- Publicly indexed subdomains.

The investigation demonstrated that the `site:` operator can reveal information hosted on both a parent domain and its subdomains.

The CVE-related results were recorded as investigative leads but were not independently verified.

### Query 3 — Public Document Discovery

```text
site:canonical.com filetype:pdf
```

Purpose:

To discover publicly indexed PDF documents associated with the organisation.

Observed documents included:

| Document | Potential intelligence value |
|---|---|
| MicroK8s with Ubuntu on LinuxONE and IBM Z Intro | Kubernetes and enterprise computing |
| Ubuntu Server for IBM Z and LinuxONE | Enterprise server technologies |
| UbuCon Asia 2023 Tracks | Community activities |
| Opportunity Open Source Conference 4.0 | Open-source engagement |
| How we use Twisted in Launchpad | Software development technologies |

The discovery of these documents provided potential intelligence about Canonical's technical ecosystem.

Their contents and current relevance were not independently assessed.

## 6. Subdomain Identification

The search results revealed the following domains:

| Domain | Classification |
|---|---|
| canonical.com | Parent domain |
| people.canonical.com | Subdomain |
| events.canonical.com | Subdomain |

This demonstrated how passive search techniques can reveal different public-facing components of an organisation's online presence.

No active subdomain enumeration or infrastructure scanning was performed.

## 7. Targeted Source Verification

### Query 4 — Corporate Source Discovery

```text
site:canonical.com "publisher of Ubuntu"
```

The search returned multiple corporate publications describing Canonical as the publisher of Ubuntu.

Observed results included:

- Canonical releases Ubuntu 26.04 LTS Resolute Raccoon.
- Canonical releases Ubuntu 26.10 Stonking Stingray.
- Canonical Releases Ubuntu 24.10 Oracular Oriole.
- Workstations — Ubuntu OS.

These results were classified as first-party corporate information.

Multiple publications from the same organisation were not treated as independent corroboration.

### Query 5 — Product Source Discovery

```text
site:ubuntu.com "Canonical"
```

Observed results included:

- Canonical Academy.
- Careers information.
- Canonical Academy SysAdmin track.
- Canonical Managed Infrastructure.

The search snippets supported the relationship between Canonical and Ubuntu.

Although the corporate and product websites use different domains, their organisational association means they should not automatically be treated as independent sources.

### Query 6 — External Source Discovery

```text
"Canonical" "Ubuntu" -site:canonical.com -site:ubuntu.com
```

Purpose:

To discover relevant information outside Canonical's and Ubuntu's domains.

Observed results included:

| Source | Classification |
|---|---|
| Wikipedia | Tertiary source |
| Lenovo Press | External corporate publication |
| Canonical Ubuntu YouTube | First-party content on an external platform |
| LinkedIn Learning | External commercial platform |
| Reddit | Community-generated information |

This exercise demonstrated that an external hosting platform does not necessarily indicate an independent information source.

## 8. External Evidence Verification

I examined the Lenovo Press document titled:

**Canonical Ubuntu Software Solution Product Guide**

Document details:

- Publisher: Lenovo Press.
- Publication date displayed: 4 May 2026.
- Document reference: LP2426.
- Document length: 12 pages.

Source:

https://lenovopress.lenovo.com/lp2426-canonical-ubuntu-software-solution-product-guide

The document identifies Canonical as the publisher of Ubuntu and describes enterprise solutions involving Ubuntu Server, Ubuntu Pro and Lenovo infrastructure.

This provided external corroboration of the original investigative claim.

However, Lenovo's commercial relationship with Canonical introduces a potential source dependency. The information cannot automatically be considered independently researched.

## 9. Source Tracing

I examined the Wikipedia article titled Canonical (company).

Source:

https://en.wikipedia.org/wiki/Canonical_(company)

The article provided information concerning Canonical's organisational identity and commercial activities.

Rather than relying exclusively on the article, I examined an underlying citation.

Reference 4 identified the following trademark information:

```text
Trademark: CANONICAL
Reference: EU004059218
Filing date: 29 September 2004
```

Following the citation led to the UK Intellectual Property Office trademark search portal.

This demonstrated the process of tracing secondary or tertiary information back towards an authoritative primary source.

## 10. Government Registry Investigation

I conducted a trademark search using the numerical identifier obtained from Wikipedia.

The initial search returned the following record:

| Attribute | Retrieved information |
|---|---|
| Trademark number | UK00004059218 |
| Trademark | PHYTOFILL |
| Status | Registered |
| Filing date | 3 June 2024 |
| Registration date | 30 August 2024 |
| Renewal date | 3 June 2034 |

Although the record originated from an authoritative government registry, it did not correspond to the intended organisation.

The trademark name, identifier prefix and filing date were inconsistent with the original reference.

The record was therefore excluded from the investigation.

## 11. False-Positive Resolution

The initial registry result demonstrated the importance of validating complete identifiers rather than relying on numerical similarity.

Following further investigation, I retrieved the corresponding CANONICAL trademark record.

| Attribute | Verified information |
|---|---|
| Trademark number | UK00904059218 |
| Trademark | CANONICAL |
| Status | Registered |
| Filing date | 29 September 2004 |
| Date of entry in register | 16 December 2005 |
| Renewal date | 29 September 2034 |

Source:

https://trademarks.ipo.gov.uk/ipo-tmcase/page/Results/1/UK00904059218

The trademark name and filing date matched the information obtained from the Wikipedia reference.

The corrected record was accepted as relevant evidence.

The registered proprietor and goods and services classifications were not examined during this investigation.

## 12. Evidence Matrix

| Evidence ID | Source | Source type | Assessment |
|---|---|---|---|
| E001 | Canonical corporate publications | Primary corporate | Search snippets support the publisher claim |
| E002 | Ubuntu website | Primary organisational | Search snippets support organisational association |
| E003 | Lenovo Press | External corporate | Relevant document content examined |
| E004 | Wikipedia | Tertiary | Reference traced to government registry |
| E005 | Canonical YouTube | First-party | External hosting does not establish independence |
| E006 | LinkedIn Learning | External commercial | Discovered but not examined |
| E007 | Reddit | Community-generated | Discovered but not examined |
| E008 | UK Intellectual Property Office | Primary government | Correct trademark record retrieved |
| E009 | PHYTOFILL trademark record | Primary government | Excluded due to identifier mismatch |

## 13. Intelligence Confidence Assessment

| Finding | Confidence |
|---|---|
| Canonical is the publisher of Ubuntu | High |
| Lenovo offers Canonical-based enterprise solutions | High |
| Canonical provides commercial Ubuntu-related services | High |
| CANONICAL trademark filing date | High |
| CANONICAL trademark registration status | High |
| CANONICAL trademark renewal date | High |
| Registered trademark proprietor | Unverified |
| Discovered CVE records | Unverified |

Confidence assessments were based on source relevance, credibility, consistency and available supporting evidence.

They do not represent mathematical probabilities.

## 14. Investigation Conclusion

The collected corporate and external evidence consistently identifies Canonical as the publisher of Ubuntu.

Source tracing also led to a government-maintained trademark record that corroborated the historical filing date obtained from Wikipedia.

An initial trademark search produced an unrelated registration because of an identifier mismatch. This was identified, investigated and corrected before reaching the final conclusion.

The investigation demonstrated that information from a credible source must still be assessed for relevance and correct attribution.

## 15. Investigative Limitations

- Some sources were assessed using search snippets rather than complete documents.
- Search engine results may change over time.
- External corporate sources may have commercial relationships with the target organisation.
- Discovered PDF documents were not comprehensively examined.
- CVE-related search results were not independently verified.
- Trademark proprietor and goods and services classifications were not examined.
- No active infrastructure scanning was conducted.
- The investigation does not constitute a comprehensive corporate due diligence assessment.

## 16. Skills Developed

- Advanced search operators.
- Passive OSINT collection.
- Organisational digital footprint analysis.
- Public document discovery.
- Subdomain identification.
- Source credibility assessment.
- Source independence assessment.
- Intelligence corroboration.
- Source tracing.
- Public registry investigation.
- Entity resolution.
- False-positive identification.
- Evidence validation.
- Intelligence confidence assessment.

## 17. Personal Reflection

This investigation helped me understand that OSINT goes beyond discovering publicly available information.

One of the most valuable experiences was retrieving an incorrect trademark record from an authoritative government registry. Despite the similarity between the numerical identifiers, the retrieved information belonged to a different trademark.

By examining the complete identifiers, trademark names and filing dates, I identified the discrepancy and retrieved the relevant record.

This reinforced the importance of accuracy, source relevance, evidence corroboration and investigative persistence.

These principles are applicable to digital investigations, cybersecurity analysis, threat intelligence and professional intelligence reporting.
