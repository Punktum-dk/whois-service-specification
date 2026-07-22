![Punktum dk Logo](https://punktum.dk/sites/default/files/logo/dk_logo_symbol_1.png)

# Punktum dk WHOIS Service Specification

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Punktum dk WHOIS Service Specification](#punktum-dk-whois-service-specification)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [About this Document](#about-this-document)
    - [License](#license)
    - [Document History](#document-history)
  - [The .dk Registry in Brief](#the-dk-registry-in-brief)
  - [Registrar Collaboration Model](#registrar-collaboration-model)
  - [Features](#features)
  - [Available Environments](#available-environments)
    - [Production Environment](#production-environment)
    - [Sandbox Environment](#sandbox-environment)
  - [Implementation Limitations](#implementation-limitations)
    - [Handle Inquiry Limitation](#handle-inquiry-limitation)
    - [Rate Limiting](#rate-limiting)
  - [Service](#service)
    - [Encoding](#encoding)
    - [Domain name query](#domain-name-query)
      - [Example query for domain name information](#example-query-for-domain-name-information)
        - [Request](#request)
        - [Response](#response)
      - [Example domain name query using punycode](#example-domain-name-query-using-punycode)
        - [Request](#request-1)
        - [Response](#response-1)
      - [Example domain name query using UTF-8](#example-domain-name-query-using-utf-8)
        - [Request](#request-2)
        - [Response](#response-2)
      - [Example domain name query with domain marked for deletion](#example-domain-name-query-with-domain-marked-for-deletion)
        - [Request](#request-3)
        - [Response](#response-3)
      - [Example query for domain name and registrant information](#example-query-for-domain-name-and-registrant-information)
        - [Request](#request-4)
        - [Response](#response-4)
      - [Example domain name query for domain name offered to waiting list](#example-domain-name-query-for-domain-name-offered-to-waiting-list)
        - [Request](#request-5)
        - [Response](#response-5)
    - [Host name query](#host-name-query)
      - [Example query for host information](#example-query-for-host-information)
  - [Test Data](#test-data)
  - [References](#references)
  - [Resources](#resources)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)
  - [Appendices](#appendices)
    - [Domain Status Values](#domain-status-values)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This document describes and specifies the implementation offered by Punktum dk A/S for interaction with the central registry for the ccTLD .dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service is not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by Punktum dk. If you are a registrar in need of structured responses, you might be interested in [the Punktum dk RESTful WHOIS Service][DKHMWHOISREST]. For domain availability lookups, [the Punktum dk Domain Availability Service (DAS)][DKHMDAS] is a lightweight alternative.

<a id="about-this-document"></a>
## About this Document

This specification describes version 6.3.x of the Punktum dk WHOIS Implementation. Future releases will be reflected in updates to this document; please refer to the [Document History](#document-history) below for changes.

The document describes the current Punktum dk WHOIS implementation, for more general documentation on the WHOIS protocol please refer to the RFCs and additional resources in the [References](#references) and [Resources](#resources) chapters below.

Do note that the specification aims to describe the latest release of the service. The service version is listed in the [Document History](#document-history), so changes implemented in the service are reflected in the specification.

This document is not the authoritative source for business and policy rules and possible discrepancies between this and any authoritative sources are regarded as errors in this document. This document is aimed at being the external technical specification and describes the implementation facing the users and is an interpretation of authoritative sources and can therefore be erroneous.

Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

This document is owned and maintained by Punktum dk A/S and must not be distributed without this information.

All examples provided in the document are fabricated/modified from real data to demonstrate commands etc. any resemblance to actual data is coincidental.

<a id="license"></a>
### License

This document is copyright by Punktum dk A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

- 6.3.1 2026-07-22
  - Refreshed examples and cleaned up references, links, and anchors
  - Reviewed the specification in full and applied corrections throughout

- 6.3 2026-06-08
  - Added documentation regarding display of registrant ID validation status
  - Adjusted examples regarding redacted/hidden registrant information

- 5.0 2021-09-09
  - Relabelled to version 5.0 to follow version number

- 4.1 2021-09-05
  - Improved descriptions on available environments and test data
  - Added links to available resources in in sections, which would improve with these

- 4.0 2021-09-02
  - Added documentation on registrar information, introduced in release 5.0.0 of the WHOIS service
  - Updated with information and example on domain names offered to waiting list position, introduced in release
    5.0.0 of the WHOIS service

- 3.4 2021-03-15
  - Added appendix on status values and corrected the explanation on status
  - Updated examples so the dates are contemporary (for now)
  - Updated links to resources, quite a few did not longer work
  - Link to GitPrint removed

- 3.3 2020-01-21
  - Added a few clarifications and corrected some bad formatting

- 3.2 2020-01-21
  - Documenting added DNSSEC status values, introduced in release 4.0.2 of the WHOIS service

- 3.1 2020-01-08
  - Added clause for examples and corrected and extended some of the examples

- 3.0 2019-11-25
  - Major update based on the changes with major release 4.0.0 of the WHOIS service
  - Documenting removal of information on registrant users for domain name inquiries
  - Documenting deprecation of support for handles inquiries

- 2.0 2019-04-30
  - Major update based on the changes with major release 3.0.0 of the WHOIS service
  - Documenting removal of public information on non-registrant users for handle (users) and domain name inquiries
  - Documenting removal of name server contacts for host (name server) inquiries

- 1.0 2016-04-27
  - Initial revision

<a id="the-dk-registry-in-brief"></a>
## The .dk Registry in Brief

Punktum dk is the registry for the ccTLD for Denmark (.dk). The current model used in Denmark is based on a sole registry, with Punktum dk maintaining the central DNS registry.

The WHOIS service offered by Punktum dk A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

<a id="registrar-collaboration-model"></a>
## Registrar Collaboration Model

The registrar collaboration model allows registrars to fully handle administration of domain names.

Punktum dk offers two models of administration:

- "Registrar Management"
- "Registrant Management"

The WHOIS will indicate choice of administrative model for a given domain name, by displaying a `Registrar` field, pointing to the name of the registrar.

If this field is omitted the domain name is under registrant management.

<a id="features"></a>
## Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Waiting list inquiry
- Support for multiple encodings (see: [Encoding](#encoding))
- Support for both IPv4 and IPv6

<a id="available-environments"></a>
## Available Environments

Punktum dk offers the following environments:

| Environment | Role        | Policies                                                                                 |
| ----------- | ----------- | ---------------------------------------------------------------------------------------- |
| production  | production  | This environment is the production environment for the Punktum dk WHOIS Service          |
| sandbox     | development | This environment is intended for client development towards the Punktum dk WHOIS Service |

<a id="production-environment"></a>
### Production Environment

- Requests made to this environment will reflect live production data.

Production is available at: `whois.punktum.dk` port `43`

<a id="sandbox-environment"></a>
### Sandbox Environment

- Queries made to this environment will reflect data only available in the isolated sandbox environment, please see the [sandbox environment specification](https://github.com/Punktum-dk/sandbox-environment-specification) for details.
- The sandbox WHOIS service is publicly available and requires neither IP whitelisting nor a sandbox user.

Sandbox is available at: `whois-sandbox.dk-hostmaster.dk` port `43`

For test data available in the sandbox environment, please see the section on [Test Data](#test-data).

<a id="implementation-limitations"></a>
## Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English.

<a id="handle-inquiry-limitation"></a>
### Handle Inquiry

Punktum dk does not support queries by contact object handles/user-ids. Contacts associated with a domain name can still be listed as part of a domain name query, see [Example query for domain name and registrant information](#example-query-for-domain-name-and-registrant-information) under Service.

<a id="rate-limiting"></a>
### Rate Limiting

To ensure high quality of service, we allow 1 request per second per source IP. Exceeding the rate limit triggers a temporary ban.

We reserve the right to adjust the rate limit and ban abusers for longer periods of time.

<a id="service"></a>
## Service

<a id="encoding"></a>
### Encoding

The service supports the following encodings:

- [ISO-8859-1] (default) can be specified as: `iso-8859-1`, `latin-1` or `latin1`
- Punycode (see also [RFC:5891])
- UTF-8, can be specified as: `utf-8` or `utf8`

See the examples below for how to specify the encoding in a query.

<a id="domain-name-query"></a>
### Domain name query

This is an example of a standard inquiry for a domain name.

The constraints on a domain name in the .dk zone are described in the [Punktum dk Name Service Specification][DKHMNSDOM].

<a id="example-query-for-domain-name-information"></a>
#### Example query for domain name information

##### Request

We inquire about the domain name: `eksempel.dk`

```bash
$ whois eksempel.dk
```

##### Response

The standard response look as follows:

```bash
# Hello xx.xx.xx.xx. Your session has been logged.
#
# Copyright (c) 2002 - 2026 by Punktum dk A/S
#
# Version: 6.3.0
#
# The data in the DK Whois database is provided by Punktum dk A/S
# for information purposes only, and to assist persons in obtaining
# information about or related to a domain name registration record.
# We do not guarantee its accuracy. We will reserve the right to remove
# access for entities abusing the data, without notice.
#
# Any use of this material to target advertising or similar activities
# are explicitly forbidden and will be prosecuted. Punktum dk A/S
# requests to be notified of any such activities or suspicions thereof.

Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2027-06-30
Registration period:  1 year
VID:                  yes
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
Hostname:             auth03.ns.dk-hostmaster.dk

# Use option --show-handles to get handle information.
# whois -h whois.punktum.dk HELP for more help.
```

The response opens with a set of informational lines, all prefixed with `#`:

- A session line, e.g. `# Hello xx.xx.xx.xx. Your session has been logged.` The IP address is masked in this example; as stated, all requests are logged.
- A copyright notice, e.g. `# Copyright (c) 2002 - 2026 by Punktum dk A/S`
- The service version string, e.g. `# Version: 6.3.0`. The service uses [semantic versioning][SEMVER], so this is major release `6`, with no feature or bug releases indicated by the minor release indicator `0` and the patch release indicator `0`.
- A terms of use notice, describing the permitted use of the data.

The remaining lines contain the actual domain name data:

| Field                 | Description                                                                                                                                                                                                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Domain`              | The domain name inquired about                                                                                                                                                                                                                    |
| `DNS`                 | The form of the domain name as used in DNS; punycode (ACE) for IDNA domain names, see [RFC:5891]                                                                                                                                                                             |
| `Registered`          | Date of registration in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark                                                              |
| `Expires`             | Date of expiration in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark                                                                |
| `Registrar`           | This field is available if the inquired domain name is under registrar management; the field is omitted if the domain name is under registrant management. For more information see the chapter on "Registrar Collaboration Model"                                           |
| `Delete date`         | Date indicating deletion in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark. Do note this field is only available if it has been set |
| `Registration period` | Registration period (`1`, `2`, `3` or `5` years)                                                                                                                                                                                                                            |
| `VID`                 | Indication whether the VID service is active, values either `yes` or `no`. See [VID service][VID] for details                                                                                                                                                                |
| `DNSSEC`              | Indication whether DNSSEC service is active, values either `Signed delegation` or `Unsigned delegation`                                                                                                                                                                      |
| `Status`              | Status of the domain name, see the [Domain Status Values](#domain-status-values) appendix                                                                                                                                                                                    |
| `Nameservers`         | List of name servers serving the inquired domain name                                                                                                                                                                                                                       |

<a id="example-domain-name-query-using-punycode"></a>
#### Example domain name query using punycode

This is a way to inquire on IDNA domains using punycode.

##### Request

```bash
$ whois xn--4cabco7dk5a.dk
```

##### Response

Observe the difference between the `Domain` and `DNS` fields

```bash
# Hello 172.16.20.12. Your session has been logged.
#
# Copyright (c) 2002 - 2026 by Punktum dk A/S
#
# Version: 6.3.0
#
# The data in the DK Whois database is provided by Punktum dk A/S
# for information purposes only, and to assist persons in obtaining
# information about or related to a domain name registration record.
# We do not guarantee its accuracy. We will reserve the right to remove
# access for entities abusing the data, without notice.
#
# Any use of this material to target advertising or similar activities
# are explicitly forbidden and will be prosecuted. Punktum dk A/S
# requests to be notified of any such activities or suspicions thereof.

Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2026-03-04
Expires:              2027-03-03
Registrar:            Test A/S
Registration period:  1 year
VID:                  no
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             ns1.test.com
Hostname:             ns2.test.com
Hostname:             ns3.test.com

# Use option --show-handles to get handle information.
# whois -h whois.punktum.dk HELP for more help.
```

<a id="example-domain-name-query-using-utf-8"></a>
#### Example domain name query using UTF-8

The WHOIS service supports responding in UTF-8 by request as opposed to the default of [ISO-8859-1].

##### Request

```bash
$ whois -c dk " --charset=utf8 æøåöäüé.dk"
```

##### Response

Observe the difference between the `Domain` and `DNS` fields.

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2026-03-04
Expires:              2027-03-03
Registrar:            Test A/S
Registration period:  1 year
VID:                  no
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             ns1.test.com
Hostname:             ns2.test.com
Hostname:             ns3.test.com

<a id="example-domain-name-query-with-domain-marked-for-deletion"></a>
#### Example domain name query with domain marked for deletion

If a domain name is marked for deletion prior to its expiration date, a deletion date is calculated and included in the response as an additional `Delete date` field:

`Delete date:          2027-07-14`

<a id="example-query-for-domain-name-and-registrant-information"></a>
#### Example query for domain name and registrant information

A standard domain name query does not include registrant information. To include the registrant associated with the domain name, add the `--show-handles` option to the query.

##### Request

```bash
$ whois -c dk ' --show-handles punktum.dk'
```

##### Response

```
Domain:               punktum.dk
DNS:                  punktum.dk
Registered:           2018-01-25
Expires:              2027-01-31
Registration period:  1 year
VID:                  no
DNSSEC:               Signed delegation
Status:               Active

Registrant
Handle:               DATA REDACTED
Name:                 Punktum dk A/S
Address:              Ørestads Boulevard 108, 11.
Postalcode:           2300
City:                 København S
Country:              DK
Phone:                +45 33646060
Email:                info@punktum.dk
ID status:            ID verified via electronic ID

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
Hostname:             auth03.ns.dk-hostmaster.dk
```

In addition to the standard domain name fields, the response includes a `Registrant` section:

| Field         | Description                                                                    |
| ------------- | ------------------------------------------------------------------------------ |
| `Handle`      | The registrant handle. Handle values are redacted and shown as `DATA REDACTED` |
| `Name`        | Name of the registrant                                                         |
| `Address`     | Address of the registrant                                                      |
| `Postalcode`  | Postal code part of the registrant address                                     |
| `City`        | City part of the registrant address                                            |
| `Country`     | Country part of the registrant address, as an [ISO-3166-1] alpha-2 code        |
| `Phone`       | Phone number of the registrant                                                 |
| `Email`       | Email address of the registrant                                                |
| `ID status`   | The identity verification status of the registrant                             |

Registrant information is displayed for both private individuals and legal entities. Information for a registrant who has name and address protection is withheld and shown as redacted.

The `ID status` field can take the following values:

| Value                                        | Description                                                                                                     |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| `ID verified via electronic ID`              | The registrant has completed identity verification using electronic ID (MitID)                                  |
| `ID verified by submission of documentation` | The registrant has verified their identity by submitting documentation                                          |
| `ID checked by registrar`                    | The registrant has completed the identity check with their registrar                                            |
| `Registrant approved after a risk assessment`| The registrant has been assessed by Punktum dk's risk engine as not required to complete the identity check     |

<a id="example-domain-name-query-for-domain-name-offered-to-waiting-list"></a>
#### Example domain name query for domain name offered to waiting list

The WHOIS service can provide limited information for a domain name offered to a waiting list position, for consistency with EPP and other services.

##### Request

The query resembles a standard domain name query, but the response differs.

```bash
$ whois -c dk waiting-list.dk
```

##### Response

```
Domain:               waiting-list.dk
DNS:                  waiting-list.dk
Registered:           ***N/A***
Expires:              ***N/A***
Registration period:  ***N/A***
VID:                  ***N/A***
DNSSEC:               ***N/A***
Status:               Offered to waiting list
```

👉 Since the domain name is not registered, the standard registration fields are not applicable and are returned as `***N/A***`. The `Status` field is set to `Offered to waiting list`, indicating that the domain name has been offered to the first position on the waiting list. The waiting list expiration is not included in the public data; a waiting list offering expires after 14 days.

<a id="host-name-query"></a>
### Host name query

You can inquire on name server hosts.

<a id="example-query-for-host-information"></a>
#### Example query for host information

##### Request

```bash
$ whois auth02.ns.dk-hostmaster.dk
```

##### Response

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Being spooled
```

If you inquire on a host without a glue record, you get the following response:

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Not being spooled
```

The above examples are relevant for name server hosts ending in `.dk`, since Punktum dk requires glue records for name servers ending in `.dk` that serve their own zone. Glue records are not required for name servers with hostnames served by other TLDs.

See the section on glue records in the [Punktum dk Name Service Specification][DKHMNSGLUE].

Note that the host query no longer discloses name server administrators as part of the response.

<a id="test-data"></a>
## Test Data

The sandbox environment provides a set of predefined test domains, in addition to any domains you register yourself via EPP or RP. For the list of predefined test domains and details on the sandbox environment, see the [sandbox environment specification](https://github.com/Punktum-dk/sandbox-environment-specification).

One predefined test domain is worth noting here: `waiting-list.dk` simulates the transient `Offered to waiting list` state, which is otherwise difficult to reproduce on demand. It can be queried in the sandbox environment to see the response shown under [Example domain name query for domain name offered to waiting list](#example-domain-name-query-for-domain-name-offered-to-waiting-list).

<a id="references"></a>
## References

Here is a list of documents and references used in this document

1. [Punktum dk RESTful WHOIS Service Specification][DKHMWHOISREST]
1. [Punktum dk Domain Availability Service (DAS) Specification][DKHMDAS]
1. [Punktum dk Name Service Specification][DKHMNS]
1. [Punktum dk VID Service][VID]
1. [RFC:3912 WHOIS Protocol Specification][RFC:3912]
1. [RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol][RFC:5891]
1. [ISO-3166-1: Alpha-2 two-letter country code][ISO-3166-1]
1. [ISO-8601: International date format][ISO-8601]
1. [ISO-8859-1: 8-bit single-byte coded graphic character sets][ISO-8859-1]
1. [Semantic Versioning][SEMVER]

<a id="resources"></a>
## Resources

Resources for Punktum dk WHOIS support can be found below.

<a id="issue-reporting"></a>
### Issue Reporting

For issue reporting related to this specification, the WHOIS implementation, or the production environment, please contact us at registrar@punktum.dk or via the [registrar contact form](https://punktum.dk/en/contact-customer-service?lvl1=Registrars&lvl2=CSRegistrarOther).

<a id="additional-information"></a>
### Additional Information

The Punktum dk domain search page:

- `https://punktum.dk/en/search-dk-domain`

<a id="appendices"></a>
## Appendices

<a id="domain-status-values"></a>
### Domain Status Values

| Status                    | Description                                                                          |
| ------------------------- | ------------------------------------------------------------------------------------ |
| `Active`                  | Domain name is published or being published to the zone                              |
| `Deactivated`             | Domain name is not being published to the zone                                       |
| `Reserved`                | Domain name is not being published to the zone (activation required by registrant)   |
| `Offered to waiting list` | Domain name has been offered to a waiting list position (action pending registrant)  |

[DKHMWHOISREST]: https://github.com/Punktum-dk/whois-rest-service-specification
[DKHMDAS]: https://github.com/Punktum-dk/das-service-specification
[DKHMNS]: https://github.com/Punktum-dk/dkhm-name-service-specification
[DKHMNSDOM]: https://github.com/Punktum-dk/dkhm-name-service-specification#domain-names
[DKHMNSGLUE]: https://github.com/Punktum-dk/dkhm-name-service-specification#glue-records
[VID]: https://punktum.dk/en/articles/vid-service
[RFC:3912]: https://www.rfc-editor.org/rfc/rfc3912
[RFC:5891]: https://www.rfc-editor.org/rfc/rfc5891
[ISO-3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601
[ISO-8859-1]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1
[SEMVER]: https://semver.org/
