# DK Hostmaster WHOIS Service Specification

2016/04/27
Revision: 1.0

# Table of Contents

<!-- MarkdownTOC bracket=round depth=3 -->

- [Introduction](#introduction)
- [About this Document](#about-this-document)
    - [License](#license)
    - [Document History](#document-history)
- [The .dk Registry in Brief](#the-dk-registry-in-brief)
- [Features](#features)
- [Implementation Limitations](#implementation-limitations)
    - [Encoding](#encoding)
    - [Rate Limiting](#rate-limiting)
- [Service](#service)
    - [Domain name query](#domain-name-query)
        - [Example query for domain name information](#example-query-for-domain-name-information)
        - [Example domain name query using punycode](#example-domain-name-query-using-punycode)
        - [Example domain name query using UTF-8](#example-domain-name-query-using-utf-8)
        - [Example domain name query with domain marked for deletion](#example-domain-name-query-with-domain-marked-for-deletion)
        - [Example domain name query including handles](#example-domain-name-query-including-handles)
        - [Example domain name query extracting anonymous handles](#example-domain-name-query-extracting-anonymous-handles)
    - [Host name query](#host-name-query)
        - [Example query for host information](#example-query-for-host-information)
        - [Example query for host and handle information](#example-query-for-host-and-handle-information)
        - [Example query for host and handle information using UTF-8](#example-query-for-host-and-handle-information-using-utf-8)
    - [Handle inquiry](#handle-inquiry)
        - [Example query for public handle](#example-query-for-public-handle)
        - [Example query for public handle using UTF-8](#example-query-for-public-handle-using-utf-8)
        - [Example query for anonymous handle](#example-query-for-anonymous-handle)
    - [Additional Help](#additional-help)
- [References](#references)
- [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)

<!-- /MarkdownTOC -->

<a name="introduction"></a>
# Introduction

This document describes and specifies the implementation offered by DK Hostmaster A/S for interaction with the central registry for the ccTLD dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service in not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by DK Hostmaster. If you are a registrar, you might be interested in [the DK Hostmaster Domain Availability Service (DAS)](https://github.com/DK-Hostmaster/das-service-specification) as an alternative.

<a name="about-this-document"></a>
# About this Document

This specification describes version 2 (2.0.x) of the DK Hostmaster WHOIS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/whois-service-specification/blob/master/README.md), using the gitprint service.

<a name="license"></a>
## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

<a name="document-history"></a>
## Document History

* 1.0 2016-04-27
  * Initial revision

<a name="the-dk-registry-in-brief"></a>
# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

The WHOIS service offered by DK Hostmaster A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

<a name="features"></a>
# Features

The service implements the following features.

- Domain name inquiry  also with extended information on handles
- Host name inquiry also with extended on handles
- Handle inquiry
- Support for multiple encodings (see: Encodings below)
- Support for both IPv6 and IPv6

<a name="implementation-limitations"></a>
# Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English. 

<a name="encoding"></a>
## Encoding

The service supports the following encodings:

- [ISO-8859-1] (default) can be specified as: `iso-8859-1`, `latin-1` or `latin1`
- Punycode (see also [RFC:5891])
- UTF-8, can be specified as: `utf-8` or `utf8`

Please see the section on service for more information on how to utilize this.

<a name="rate-limiting"></a>
## Rate Limiting

We only allow a certain number of requests per minute. We reserve the right to adjust the rate limit in order to provide a high quality of service. 

Current limit is set to 1 request per second.

In addition the service only allow 1 TCP-connection per. (IPv4)/24.

Meaning that `192.0.2.41` and `192.0.2.52` can not have simultanous connections, but `192.0.2.41` and `192.0.3.52` can.

<a name="service"></a>
# Service

<a name="domain-name-query"></a>
## Domain name query

This is an example of a standard inquiry for a domain name

<a name="example-query-for-domain-name-information"></a>
### Example query for domain name information

#### Request

We inquire about the domain name: `eksempel.dk`

```bash
$ whois eksempel.dk
```

#### Response

The standard response look as follows:

```bash
# Hello XX.XX.XX.XX. Your session has been logged.
#
# Copyright (c) 2002 - 2016 by DK Hostmaster A/S
#
# Version: 2.0.2
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
Expires:              2017-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk

# Use option --show-handles to get handle information.
# Whois HELP for more help.
```

```
# Hello XX.XX.XX.XX. Your session has been logged.
```

The IP address has been masked for the example, As stated all request are logged.

```
# Copyright (c) 2002 - 2016 by DK Hostmaster A/S
```

Copyright notice.

```
# Version: 2.0.2
```

This is the version string of the service. The service uses (semantic versioning)[semver.org], so this is major release `2`, bug fix release `2`. No feature releases has been made indicated by the minor release indicator: `0`.

```
# The data in the DK Whois database is provided by DK Hostmaster A/S
# for information purposes only, and to assist persons in obtaining
# information about or related to a domain name registration record.
# We do not guarantee its accuracy. We will reserve the right to remove
# access for entities abusing the data, without notice.
#
# Any use of this material to target advertising or similar activities
# are explicitly forbidden and will be prosecuted. DK Hostmaster A/S
# requests to be notified of any such activities or suspicions thereof.
```

Terms of use notice.

```
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2017-06-30
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
| Domain | The domain name, should match the one enquired about |
| DNS | Version of the domain name inquired used in DNS, punycode for IDNA domain names [RFC:5891] |
| Registered | Date of registration [ISO-8601] |
| Expires | Date of expiration [ISO-8601] |
| Registration period | Registration period (`1`, `2`, `3` or `5` years) |
| VID | Indication whether VID service is active, values either `yes` or `no` |
| Dnssec | Indication whether DNSSEC service is active, values either `Signed delegation` or `Unsigned delegation` |
| Status | Status of the domain name: 'A' for active, 'S' marked for deletion and 'H' on hold if deletion date has been surpassed |
| Nameservers | List of nameservers, serving the inquired domain name |

<a name="example-domain-name-query-using-punycode"></a>
### Example domain name query using punycode

This is a way to inquire on IDNA domains using punycode.

#### Request

```bash
$ whois xn--4cabco7dk5a.dk
```

#### Response

Observe the difference between the `Domain` and `DNS` fields

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2010-06-14
Expires:              2016-06-30
Registration period:  1 year
VID:                  no
Dnssec:               Unsigned delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a name="example-domain-name-query-using-utf-8"></a>
### Example domain name query using UTF-8

The WHOIS service supports responding in UTF-8 by request as opposed to the default of [ISO-8859-1].

#### Request

```bash
$ whois " --charset=utf8 æøåöäüé.dk"
```

#### Response

```
Domain:               æøåöäüé.dk
DNS:                  xn--4cabco7dk5a.dk
Registered:           2010-06-14
Expires:              2016-06-30
Registration period:  1 year
VID:                  no
Dnssec:               Unsigned delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a name="example-domain-name-query-with-domain-marked-for-deletion"></a>
### Example domain name query with domain marked for deletion

If a domain name is marked for deletion prior to the expiration date a deletion date is calculated.

`Delete date:          2016-04-14`

#### Request

```bash
$ whois eksempel.dk
```

#### Response

```
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           2010-06-14
Expires:              2016-06-30
Delete date:          2016-04-14
Registration period:  1 year
VID:                  no
Dnssec:               Unsigned delegation
Status:               Active

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Hostname:             auth02.ns.dk-hostmaster.dk
```

<a name="example-domain-name-query-including-handles"></a>
### Example domain name query including handles

DK Hostmaster's WHOIS support listing handles associated with a given domain.

#### Request

```bash
$ whois ' --show-handles eksempel.dk'
```

#### Response

```bash
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2017-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
Status:               Active

Registrant
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK

Administrator
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Handle:               DKHM1-DK
Hostname:             auth02.ns.dk-hostmaster.dk
Handle:               DKHM1-DK
```

<a name="example-domain-name-query-extracting-anonymous-handles"></a>
### Example domain name query extracting anonymous handles

If you make a inquiry asking for handle information and the users are maked anonymous in the WHOIS service: `***N/A***` is returned.

#### Request

```bash
$ whois ' --show-handles eksempel.dk'
```

#### Response

```bash
Domain:               eksempel.dk
DNS:                  eksempel.dk
Registered:           1999-05-17
Expires:              2017-06-30
Registration period:  5 years
VID:                  yes
Dnssec:               Signed delegation
Status:               Active

Registrant
Handle:               ***N/A***

Administrator
Handle:               ***N/A***

Nameservers
Hostname:             auth01.ns.dk-hostmaster.dk
Handle:               DKHM1-DK
Hostname:             auth02.ns.dk-hostmaster.dk
Handle:               DKHM1-DK
```

<a name="host-name-query"></a>
## Host name query

You can inquire nameserver hosts.

<a name="example-query-for-host-information"></a>
### Example query for host information

#### Request 

```bash
$ whois auth02.ns.dk-hostmaster.dk
```

#### Response

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Being spooled
```

If you inquire on a host without a glue record, you get the following response:

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Not being spooled
```

The above example is relevant for nameserver hosts not ending in `.dk`, since DK Hostmaster require glue records for nameservers ending in `.dk` and glue records are not required for nameservers with hostnames hosted with other TLDs.

<a name="example-query-for-host-and-handle-information"></a>
### Example query for host and handle information

#### Request

```bash
$ whois " --show-handles auth02.ns.dk-hostmaster.dk"
```

#### Response

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Being spooled

Contact
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK
```

<a name="example-query-for-host-and-handle-information-using-utf-8"></a>
### Example query for host and handle information using UTF-8

As described earlier [ISO-8859-1] is the default encoding, so in order to retrieve information encoded as UTF-8, you have to use the `--charset` parameter.

#### Request

```bash
$ whois " --show-handles --charset=utf8 auth02.ns.dk-hostmaster.dk"
```

#### Response

```
Nameserver:           auth02.ns.dk-hostmaster.dk
Glue:                 Being spooled

Contact
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK
```

<a name="handle-inquiry"></a>
## Handle inquiry

In addition to domain and hostname inquiries, you can inquire handles (contact-ids).

<a name="example-query-for-public-handle"></a>
### Example query for public handle

#### Request

```bash
$ whois -c dk DKHM1-DK
```

Please note the `-c` flag for specifying country code, this is parameter is specific to your `whois` and might vary.

#### Response

```
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK
```

| Field | Description |
| ----- | ----------- |
| Handle | The handle, should match the one enquired about |
| Name | Name associated with the entity referenced by the above handle |
| Address | Address associated with the entity referenced by the above handle |
| Postalcode | Postalcode associated with the above address |
| City | Postalcode associated with the above address |
| Country | 2-letter country code associated with the above address, specified in [ISO-3166-1] alpha-2 format |

Please note that due to the representation in DK Hostmasters system for handling contacts the following rules are applied to postal information.

For Denmark the local representation is chosen and the international representation is discarded. For other countries the international representation is chosen and the local representation is discarded. Please see the table below.

| Denmark | Other country |
| ----------- | ----------- |
| **Local representation** | Local representation |
| International representation | **International representation** |

Please refer to [the EPP Service specification](https://github.com/DK-Hostmaster/epp-service-specification#create-contact) for more information on creation of contact objects in the DK Hostmaster system.

<a name="example-query-for-public-handle-using-utf-8"></a>
### Example query for public handle using UTF-8

As described earlier [ISO-8859-1] is the default encoding, so in order to retrieve information encoded as UTF-8, you have to use the `--charset` parameter.

#### Request

```bash
$ whois -c dk " --charset=utf8 DKHM1-DK"
```

Please note the `-c` flag for specifying country code, this is parameter is specific to your `whois` and might vary.

```
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK
```

<a name="example-query-for-anonymous-handle"></a>
### Example query for anonymous handle

#### Request

```bash
$ whois -c dk ANON-DK
```

Please note the `-c` flag for specifying country code, this is parameter is specific to your `whois` and might vary.

#### Response

```
Handle:               ***N/A***
```

<a name="additional-help"></a>
## Additional Help 

Additional help can be obtainted on the command line using the following command:

#### Request

```bash
$ whois -h whois.dk-hostmaster.dk HELP
```

#### Response

```bash
# Query syntax:
#   [<options>] <query_string>
# Available options:
#   --charset=<charset>
#   --accesscode=<accesscode>[:<accesscode>[:<accesscode>]]
#   --show-handles
# Available charsets:
#   latin-1 also known as iso-8859-1 (default)
#   utf-8
# Example:
#   --charset=latin-1 dk-hostmaster.dk
#   --accesscode=C8850DF92ECB6CF581EF6C8FD31C1CDF dk-hostmaster.dk
# Hint:
#   Most Unix whois clients have problems with these options and tries
#   to parse them themselves. To get around this, do lookups like this:
#     whois " --charset=latin-1 dk-hostmaster.dk"
#   Note the additional space after the first quote.
```

<a name="references"></a>
# References

Here is a list of documents and references used in this document

* General Terms and Conditions: https://www.dk-hostmaster.dk/fileadmin/filer/pdf/generelle_vilkaar/general-conditions.pdf
* RFC:5891 Internationalized Domain Names in Applications (IDNA): Protocol: https://tools.ietf.org/html/rfc5891
* RFC:3912 WHOIS Protocol Specification: https://tools.ietf.org/html/rfc3912
* Documentation on the format of a domain name with the DK Hostmaster A/S registry: https://www.dk-hostmaster.dk/english/technical-administration/forms/register-domainname/
* ISO-8601: International date format: https://en.wikipedia.org/wiki/ISO_8601
* ISO-3166-1: Alpha-2. two-letter country code: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2
* ISO-8859-1: 8-bit single-byte coded graphic character sets: https://en.wikipedia.org/wiki/ISO/IEC_8859-1

<a name="resources"></a>
# Resources

Resources for DK Hostmaster WHOIS support can be found below.

<a name="mailing-list"></a>
## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `tech-discuss+subscribe@liste.dk-hostmaster.dk`

<a name="issue-reporting"></a>
## Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `info@dk-hostmaster.dk`

<a name="additional-information"></a>
## Additional Information

The DK Hostmaster website service page

  * `https://www.dk-hostmaster.dk/en/whois`

[RFC:5891]: https://tools.ietf.org/html/rfc5891

[RFC:3912]: https://tools.ietf.org/html/rfc3912

[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601

[ISO-3166-1]: https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2

[ISO-8859-1]: https://en.wikipedia.org/wiki/ISO/IEC_8859-1

