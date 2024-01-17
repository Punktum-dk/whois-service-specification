![Punktum dk Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# Punktum dk WHOIS Service Specification

![Markdownlint Action](https://github.com/DK-Hostmaster/whois-service-specification/workflows/Markdownlint%20Action/badge.svg)
![Spellcheck Action](https://github.com/DK-Hostmaster/whois-service-specification/workflows/Spellcheck%20Action/badge.svg)

2021-09-09
Revision: 5.0

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [DK Hostmaster WHOIS Service Specification](#dk-hostmaster-whois-service-specification)
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
    - [Handle Inquiry](#handle-inquiry)
    - [Encoding](#encoding)
    - [Rate Limiting](#rate-limiting)
  - [Service](#service)
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
      - [Example domain name query including handles](#example-domain-name-query-including-handles)
        - [Request](#request-4)
        - [Response](#response-4)
      - [Example domain name query for domain name offered to waiting list](#example-domain-name-query-for-domain-name-offered-to-waiting-list)
        - [Request](#request-5)
        - [Response](#response-5)
    - [Host name query](#host-name-query)
      - [Example query for host information](#example-query-for-host-information)
        - [Request](#request-6)
        - [Response](#response-6)
    - [Handle inquiry](#handle-inquiry-1)
    - [Additional Help](#additional-help)
      - [Request](#request-7)
      - [Response](#response-7)
  - [Test Data](#test-data)
    - [Domains](#domains)
    - [Waiting List](#waiting-list)
  - [References](#references)
  - [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)
  - [Appendices](#appendices)
    - [Domain Status Values](#domain-status-values)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This document describes and specifies the implementation offered by Punktum dk A/S for interaction with the central registry for the ccTLD dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service in not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by Punktum dk. If you are a registrar, you might be interested in [the Punktum dk Domain Availability Service (DAS)][DKHMDAS] as an alternative.

<a id="about-this-document"></a>
## About this Document

This specification describes version 5 (5.X.X) of the Punktum dk WHOIS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current Punktum dk WHOIS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

:warning: Punktum dk specific features might not be supported by all clients and operating systems.

Do note all command lines examples were created on MacOS version 10.11 using the `whois` command line client shipped with this version, updates to this client and operating system are not automatically reflected in the specification under the clause stated above.

<a id="license"></a>
### License

This document is copyright by Punktum dk A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

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

Punktum dk is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with Punktum dk maintaining the central DNS registry.

The WHOIS service offered by Punktum dk A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

<a id="registrar-collaboration-model"></a>
## Registrar Collaboration Model

The registrar collaboration model allows registrars to fully handle administration of domain names.

Punktum dk offers two models of administration:

- "Registrar Management"
- "Registrant Management"

The WHOIS will indicate choice of administrative model for a given domain name, by displaying a `Registrar` field, pointing to the name of the registrar.

If this field is omitted the domain name is under registrant management.

You can find more information on [the two different models]([models]) on the Punktum dk website, together with more information on [the concept behind the registrar model][concept].

<a id="features"></a>
## Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Support for multiple encodings (see: Encodings below)
- Support for both IPv4 and IPv6

<a id="available-environments"></a>
## Available Environments

Punktum dk offers the following environments:

| Environment | Role | Policies |
| ----------- | ---- | ----------- |
| production  | production | This environment is the production environment for the Punktum dk WHOIS Service |
| sandbox     | development | This environment is intended for client development towards the Punktum dk WHOIS Service |

For information on what service and specification is applicable and available, consult the [Punktum dk WHOIS Service Wiki][WIKI]
For use please see the section on [Test Data](#test-data).

<a id="production-environment"></a>
### Production Environment

- requests made to this environment will reflect live production data

Production is available at: `whois.dk-hostmaster.dk` port `43`

<a id="sandbox-environment"></a>
### Sandbox Environment

- Queries made to this environment will reflect data only available in the isolated sandbox environment, please see the [sandbox environment specification](https://github.com/DK-Hostmaster/sandbox-environment-specification) for details.
- Please additional data please, see the section on [Test Data](#test-data).

<a id="implementation-limitations"></a>
## Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English.

<a id="handle-inquiry-limitation"></a>
### Handle Inquiry

As of service version 4 (4.X.X) Punktum dk does not support inquries for contact object handles/user-ids

<a id="encoding"></a>
### Encoding

The service supports the following encodings:

- [ISO-8859-1] (default) can be specified as: `iso-8859-1`, `latin-1` or `latin1`
- Punycode (see also [RFC:5891])
- UTF-8, can be specified as: `utf-8` or `utf8`

Please see the section on service for more information on how to utilize this.

<a id="rate-limiting"></a>
### Rate Limiting

We only allow a certain number of requests per minute. We reserve the right to adjust the rate limit in order to provide a high quality of service.

Current limit is set to 1 request per second.

In addition the service only allow 1 TCP-connection per. (IPv4)/24.

Meaning that `192.0.2.41` and `192.0.2.52` can not have simultaneous connections, but `192.0.2.41` and `192.0.3.52` can.

<a id="service"></a>
## Service

<a id="domain-name-query"></a>
### Domain name query

This is an example of a standard inquiry for a domain name.

The constraints on a domain name in the .dk zone is described in the [Punktum dk Name Service Specification][DKHMNSDOM].

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
# Hello XX.XX.XX.XX. Your session has been logged.
#
# Copyright (c) 2002 - 2021 by Punktum dk A/S
#
# Version: 5.0.0
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
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk

# Use option --show-handles to get handle information.
# whois -h whois.dk-hostmaster.dk HELP for more help.
```

```
## Hello XX.XX.XX.XX. Your session has been logged.
```

The IP address has been masked for the example, As stated all request are logged.

```
## Copyright (c) 2002 - 2021 by Punktum dk A/S
```

Copyright notice.

```
## Version: 5.0.0
```

This is the version string of the service. The service uses [semantic versioning][SEMVER], so this is major release `3`, No feature or bug releases has been made indicated by the minor release indicator: `0` and the patch release indicator:`0`.

```
## The data in the DK Whois database is provided by Punktum dk A/S
## for information purposes only, and to assist persons in obtaining
## information about or related to a domain name registration record.
## We do not guarantee its accuracy. We will reserve the right to remove
## access for entities abusing the data, without notice.
#
## Any use of this material to target advertising or similar activities
## are explicitly forbidden and will be prosecuted. Punktum dk A/S
## requests to be notified of any such activities or suspicions thereof.
```

Terms of use notice.

```
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registrar:            All Things DK Domains
Registration period:  5 years
VID:                  yes
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

Then we get to the data.

| Field | Description |
| ----- | ----------- |
| `Domain` | The domain name, should match the one enquired about |
| `DNS` | Version of the domain name inquired used in DNS, punycode for IDNA domain names [RFC:5891] |
| `Registered` | Date of registration in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark |
| `Expires` | Date of expiration in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark |
| `Registrar` | This field is available if the inquired domain name is handled by a registrar, the field is omitted if the domain name is registrant handled, for more information see the chapter on "Registrar Collaboration Model" |
| `Delete date` | Date indicating deletion in [ISO-8601] format: `YYYY-MM-DD`, the timezone is not expressed explicitly. The local time of the registry is used, meaning Central European Standard Time (`GMT+1`), Copenhagen/Denmark. Do note this field is only available if is has been set |
| `Registration period` | Registration period (`1`, `2`, `3` or `5` years) |
| `VID` | Indication whether VID service is active, values either `yes` or `no` |
| `DNSSEC` | Indication whether DNSSEC service is active, values either `Signed delegation`, `Unsigned delegation, DNSSEC disabled, no records`, `Unsigned delegation, DNSSEC disabled, keys unpublished`, `Unsigned delegation, DNSSEC disabled`, `Unsigned delegation, no records`, `Unsigned delegation, DNSSEC enabled, keys unpublished` or `Unknown status` |
| `Status` | Status of the domain name, please see the appendix |
| `Nameservers` | List of name servers, serving the inquired domain name |

<a id="example-domain-name-query-using-punycode"></a>
#### Example domain name query using punycode

This is a way to inquire on IDNA domains using punycode.

##### Request

```bash
$ whois xn--4cabco7dk5a.dk
```

##### Response

Observe the difference between the `Domain` and `DNS` fields

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2010-06-14
Expires:              2019-06-30
Registration period:  1 year
VID:                  no
DNSSEC:               Unsigned delegation, no records
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a id="example-domain-name-query-using-utf-8"></a>
#### Example domain name query using UTF-8

The WHOIS service supports responding in UTF-8 by request as opposed to the default of [ISO-8859-1].

##### Request

```bash
$ whois -c dk " --charset=utf8 æøåöäüé.dk"
```

##### Response

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2010-06-14
Expires:              2019-06-30
Registration period:  1 year
VID:                  no
DNSSEC:               Unsigned delegation, no records
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a id="example-domain-name-query-with-domain-marked-for-deletion"></a>
#### Example domain name query with domain marked for deletion

If a domain name is marked for deletion prior to the expiration date a deletion date is calculated.

`Delete date:          2019-07-14`

##### Request

```bash
$ whois eksempel.dk
```

##### Response

```
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
DNSSEC:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a id="example-domain-name-query-including-handles"></a>
#### Example domain name query including handles

Punktum dk's WHOIS support listing handles associated with a given domain.

##### Request

```bash
$ whois -c dk ' --show-handles eksempel.dk'
```

##### Response

```bash
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
DNSSEC:               Signed delegation
Status:               Active

Registrant
Handle:               ***N/A***
Name:                 DK HOSTMASTER A/S
Address:              Ørestads Boulevard 108, 11.
Postalcode:           2300
City:                 København S
Country:              DK

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a id="example-domain-name-query-for-domain-name-offered-to-waiting-list"></a>
#### Example domain name query for domain name offered to waiting list

Punktum dk's WHOIS support listing can provide information, limited though, for domain names offered from to a waiting list position, this is for consistency with EPP, DAS and other services.

##### Request

The query resembles a standard query for a domain name, the response however differs.

```bash
$ whois -c dk eksempel.dk
```

##### Response

```bash
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           ***N/A***
Expires:              ***N/A***
Registration period:  ***N/A***
VID:                  ***N/A***
DNSSEC:               ***N/A***
Status:               Offered to waiting list
```

As can be read from the response.

1. The domain name is not registered: `Registered: ***N/A***`
1. The domain name does not have expiration data: `Expires: ***N/A***`, the waiting list expiration is not included in the public data. An waiting list offering expires after 14 days
1. Since to registration is active, there is not period associated: `Registration period: ***N/A***`
1. VID is not available: `VID: ***N/A***`
1. DNSSEC is not available: `DNSSEC: ***N/A***`
1. Status is `Offered to waiting list`, indicating that the domain name has been offered to the first position on the waiting list

<a id="host-name-query"></a>
### Host name query

You can inquire name server hosts.

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

The above examples are relevant for name server hosts ending in `.dk`, since Punktum dk require glue records for name servers ending in `.dk`, which serve there own zone. Glue records are not required for name servers with hostnames served by other TLDs.

See the section on glue records in the [Punktum dk Name Service Specification][DKHMNSGLUE].

Do note that the host (name server) no longer supports disclosing name server administrators as part of the response.

<a id="handle-inquiry"></a>
### Handle inquiry

As described under Implementation Limitations, Punktum dk does not support queries on handles.

<a id="additional-help"></a>
### Additional Help

Additional help can be obtained on the command line using the following command:

<a id="request"></a>
#### Request

```bash
$ whois -h whois.dk-hostmaster.dk HELP
```

<a id="response"></a>
#### Response

```bash
## Query syntax:
##   [<options>] <query_string>
## Available options:
##   --charset=<charset>
##   --accesscode=<accesscode>[:<accesscode>[:<accesscode>]]
##   --show-handles
## Available charsets:
##   latin-1 also known as iso-8859-1 (default)
##   utf-8
## Example:
##   --charset=latin-1 dk-hostmaster.dk
##   --accesscode=C8850DF92ECB6CF581EF6C8FD31C1CDF dk-hostmaster.dk
## Hint:
##   Most Unix whois clients have problems with these options and tries
##   to parse them themselves. To get around this, do lookups like this:
##     whois " --charset=latin-1 dk-hostmaster.dk"
##   Note the additional space after the first quote.
```

<a id="test-data"></a>
## Test Data

The sandbox uses a combination of a predefined set of test data and data added to the sandbox environment by use.

<a id="domains"></a>
### Domains

| Domain name | Status | Notes |
|-------------|--------|-------|
| `dk-hostmaster.dk` | `Active` | The domain is visible and active |
| `æøåöäüé.dk` | `unavailable` | The domain is visible and active |
| `waiting-list.dk` | `Offered to waiting list` | The domain status is awaiting the designated registrant |
| * | * | Depending on what domains have been registered with the sandbox environment. Please see the [sandbox environment specification](https://github.com/DK-Hostmaster/sandbox-environment-specification) for details. |

<a id="waiting-list"></a>
### Waiting List

Since the `Offered to waiting list` is very transient, the domain name: `waiting-list.dk` simulates this state.

The domain name can be queried via the WHOIS service in the sandbox environment:

```bash
whois -h whois-sandbox.dk-hostmaster.dk waiting-list.dk
# Hello X.X.X.X. Your session has been logged.
#
# Copyright (c) 2002 - 2021 by Punktum dk A/S
#
# Version: 5.0.0
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

Domain:               waiting-list.dk
DNS:                  waiting-list.dk
Registered:           ***N/A***
Expires:              ***N/A***
Registration period:  ***N/A***
VID:                  ***N/A***
DNSSEC:               ***N/A***
Status:               Offered to waiting list
```

Do note this behaviour is reserved for the sandbox environment.

For more details on the sandbox environment, please see the [sandbox environment specification](https://github.com/DK-Hostmaster/sandbox-environment-specification).

<a id="references"></a>
## References

Here is a list of documents and references used in this document

1. [Punktum dk General Terms and Conditions][DKHMTAC]
1. [Punktum dk Name Service Specification][DKHMNS]
1. [RFC:3912 WHOIS Protocol Specification][RFC:3912]
1. [RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol][RFC:5891]
1. [ISO-3166-1: Alpha-2. two-letter country code][ISO-3166-1]
1. [ISO-8601: International date format][ISO-8601]
1. [ISO-8859-1: 8-bit single-byte coded graphic character sets][ISO-8859-1]

<a id="resources"></a>
## Resources

Resources for Punktum dk WHOIS support can be found below.

<a id="mailing-list"></a>
### Mailing list

Punktum dk operates a mailing list for discussion and inquiries about the Punktum dk WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the scope of technical discussion will not be responded to.

Please report any issues via the designated channels and they will be passed on to the appropriate entities within Punktum dk A/S.

- `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a id="issue-reporting"></a>
### Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us. You are of course welcome to post these to the mailing list mentioned above, otherwise use the regular support channels.

<a id="additional-information"></a>
### Additional Information

The Punktum dk website service page

- `https://www.dk-hostmaster.dk/en/whois`

<a id="appendices"></a>
## Appendices

<a id="domain_status_values"></a>
### Domain Status Values

| Status                | Description                                                                        |
| --------------------- | ---------------------------------------------------------------------------------- |
| `Active`              | Domain name is or being published to the zone                                      |
| `Deactivated`         | Domain name is not being published to the zone                                     |
| `Reserved`            | Domain name is not being published to the zone (activation required by registrant) |
| `Offered to waiting list` | Domain name has been offered to a waiting list position (action pending registrant) |

[DKHMTAC]: https://www.dk-hostmaster.dk/en/general-conditions
[DKHMNS]: https://github.com/DK-Hostmaster/dkhm-name-service-specification
[DKHMNSDOM]: https://github.com/DK-Hostmaster/dkhm-name-service-specification#domain-names
[DKHMNSGLUE]: https://github.com/DK-Hostmaster/dkhm-name-service-specification#glue-records
[DKHMDAS]: https://github.com/DK-Hostmaster/das-service-specification
[RFC:3912]: https://tools.ietf.org/html/rfc3912
[RFC:5891]: https://tools.ietf.org/html/rfc5891
[ISO-3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601
[ISO-8859-1]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1
[SEMVER]: https://semver.org/
[concept]: https://www.dk-hostmaster.dk/en/new-basis-collaboration-between-registrars-and-dk-hostmaster
[models]: https://www.dk-hostmaster.dk/en/node/819
[WIKI]: https://github.com/DK-Hostmaster/whois-service-specification/wiki
