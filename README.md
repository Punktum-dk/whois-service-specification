![DK Hostmaster Logo](https://www.dk-hostmaster.dk/sites/default/files/dk-logo_0.png)

# DK Hostmaster WHOIS Service Specification

![GitHub Workflow build status badge markdownlint](https://github.com/DK-Hostmaster/whois-service-specification/workflows/Markdownlint%20Workflow/badge.svg)

2019-11-25
Revision: 3.0

## Table of Contents

<!-- MarkdownTOC bracket=round levels="1,2,3,4" indent="  " autolink="true" autoanchor="true" -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
  - [License](#license)
  - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
  - [Handle Inquiry](#handle-inquiry-limitation)
  - [Encoding](#encoding)
  - [Rate Limiting](#rate-limiting)
- [Service](#service)
  - [Domain name query](#domain-name-query)
    - [Example query for domain name information](#example-query-for-domain-name-information)
    - [Example domain name query using punycode](#example-domain-name-query-using-punycode)
    - [Example domain name query using UTF-8](#example-domain-name-query-using-utf-8)
    - [Example domain name query with domain marked for deletion](#example-domain-name-query-with-domain-marked-for-deletion)
    - [Example domain name query including registrant](#example-domain-name-query-including-registrant)
  - [Host name query](#host-name-query)
    - [Example query for host information](#example-query-for-host-information)
  - [Handle inquiry](#handle-inquiry)
  - [Additional Help](#additional-help)
    - [Request](#request)
    - [Response](#response)
- [References](#references)
- [Resources](#resources)
  - [Mailing list](#mailing-list)
  - [Issue Reporting](#issue-reporting)
  - [Additional Information](#additional-information)

<!-- /MarkdownTOC -->

<a id="introduction"></a>
## Introduction

This document describes and specifies the implementation offered by DK Hostmaster A/S for interaction with the central registry for the ccTLD dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service in not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by DK Hostmaster. If you are a registrar, you might be interested in [the DK Hostmaster Domain Availability Service (DAS)](https://github.com/DK-Hostmaster/das-service-specification) as an alternative.

<a id="about-this-document"></a>
## About this Document

This specification describes version 4 (4.X.X) of the DK Hostmaster WHOIS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/whois-service-specification/blob/master/README.md), using the **gitprint** service.

<a id="license"></a>
### License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a id="document-history"></a>
### Document History

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

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

The WHOIS service offered by DK Hostmaster A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

<a id="features"></a>
## Features

The service implements the following features.

- Domain name inquiry
- Host name inquiry
- Support for multiple encodings (see: Encodings below)
- Support for both IPv4 and IPv6

<a id="implementation-limitations"></a>
## Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English.

<a id="handle-inquiry-limitation"></a>
### Handle Inquiry

As of service version 4 (4.X.X) DK Hostmaster does not support inquries for contact object handles/user-ids

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

This is an example of a standard inquiry for a domain name

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
# Copyright (c) 2002 - 2019 by DK Hostmaster A/S
#
# Version: 4.0.0
#
# The data in the DK Whois database is provided by DK Hostmaster A/S
# for information purposes only, and to assist persons in obtaining
# information about or related to a domain name registration record.
# We do not guarantee its accuracy. We will reserve the right to remove
# access for entities abusing the data, without notice.
#
# Any use of this material to target advertising or similar activities
# are explicitly forbidden and will be prosecuted. DK Hostmaster A/S
# requests to be notified of any such activities or suspicions thereof.

Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
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
## Copyright (c) 2002 - 2019 by DK Hostmaster A/S
```

Copyright notice.

```
## Version: 4.0.0
```

This is the version string of the service. The service uses [semantic versioning](semver.org), so this is major release `3`, No feature or bug releases has been made indicated by the minor release indicator: `0` and the patch release indicator:`0`.

```
## The data in the DK Whois database is provided by DK Hostmaster A/S
## for information purposes only, and to assist persons in obtaining
## information about or related to a domain name registration record.
## We do not guarantee its accuracy. We will reserve the right to remove
## access for entities abusing the data, without notice.
#
## Any use of this material to target advertising or similar activities
## are explicitly forbidden and will be prosecuted. DK Hostmaster A/S
## requests to be notified of any such activities or suspicions thereof.
```

Terms of use notice.

```
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

Then we get to the data.

| Field | Description |
| ----- | ----------- |
| ´Domain´ | The domain name, should match the one enquired about |
| ´DNS´ | Version of the domain name inquired used in DNS, punycode for IDNA domain names [RFC:5891] |
| ´Registered´ | Date of registration [ISO-8601] |
| ´Expires´ | Date of expiration [ISO-8601] |
| ´Registration period´ | Registration period (`1`, `2`, `3` or `5` years) |
| ´VID´ | Indication whether VID service is active, values either `yes` or `no` |
| ´Dnssec´ | Indication whether DNSSEC service is active, values either `Signed delegation` or `Unsigned delegation` |
| ´Status´ | Status of the domain name: 'A' for active, 'S' marked for deletion and 'H' on hold if deletion date has been surpassed |
| ´Nameservers´ | List of name servers, serving the inquired domain name |

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
Dnssec:               Unsigned delegation
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
$ whois " --charset=utf8 æøåöäüé.dk"
```

##### Response

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2010-06-14
Expires:              2019-06-30
Delete date:          2019-07-14`
Registration period:  1 year
VID:                  no
Dnssec:               Unsigned delegation
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
Dnssec:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a id="example-domain-name-query-including-handles"></a>
#### Example domain name query including handles

DK Hostmaster's WHOIS support listing handles associated with a given domain.

##### Request

```bash
$ whois ' --show-handles eksempel.dk'
```

##### Response

```bash
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2022-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
Status:               Active

Registrant
Handle:               ***N/A***
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

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

The above example is relevant for name server hosts not ending in `.dk`, since DK Hostmaster require glue records for name servers ending in `.dk` and glue records are not required for name servers with hostnames hosted with other TLDs.

Do note that the host (name server) no longer supports disclosing name server administrators as part of the response.

<a id="handle-inquiry"></a>
### Handle inquiry

As described under Implementation Limitations, DK Hostmaster does not support queries on handles.

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

<a id="references"></a>
## References

Here is a list of documents and references used in this document

- General Terms and Conditions: https://www.dk-hostmaster.dk/fileadmin/filer/pdf/generelle_vilkaar/general-conditions.pdf
- RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol: https://tools.ietf.org/html/rfc5891
- RFC:3912 WHOIS Protocol Specification: https://tools.ietf.org/html/rfc3912
- Documentation on the format of a domain name with the DK Hostmaster A/S registry: https://www.dk-hostmaster.dk/english/technical-administration/forms/register-domainname/
- ISO-8601: International date format: https://en.wikipedia.org/wiki/ISO_8601
- ISO-3166-1: Alpha-2. two-letter country code: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
- ISO-8859-1: 8-bit single-byte coded graphic character sets: https://en.wikipedia.org/wiki/ISO/IEC_8859-1

<a id="resources"></a>
## Resources

Resources for DK Hostmaster WHOIS support can be found below.

<a id="mailing-list"></a>
### Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

- `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a id="issue-reporting"></a>
### Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

- `info@dk-hostmaster.dk`

<a id="additional-information"></a>
### Additional Information

The DK Hostmaster website service page

- `https://www.dk-hostmaster.dk/en/whois`

[RFC:5891]: https://tools.ietf.org/html/rfc5891

[RFC:3912]: https://tools.ietf.org/html/rfc3912

[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601

[ISO-3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2

[ISO-8859-1]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1
