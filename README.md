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
- [Implementation Limitations](#implementation-limitations)
    - [Encoding](#encoding)
    - [Rate Limiting](#rate-limiting)
- [Service](#service)
    - [Domain name request](#domain-name-request)
        - [Example query using punycode](#example-query-using-punycode)
        - [Example query with domain marked for deletion](#example-query-with-domain-marked-for-deletion)
        - [Example query extracting handles](#example-query-extracting-handles)
        - [Example query extracting anonymous handles](#example-query-extracting-anonymous-handles)
    - [Host name request](#host-name-request)
        - [Example query for host information](#example-query-for-host-information)
        - [Example query for host and handle information](#example-query-for-host-and-handle-information)
    - [Handle request](#handle-request)
        - [Example query for public handle](#example-query-for-public-handle)
        - [Example query for anonymous handle](#example-query-for-anonymous-handle)
    - [Additional Help](#additional-help)
- [References](#references)
- [Resources](#resources)
    - [Mailing list](#mailing-list)
    - [Issue Reporting](#issue-reporting)
    - [Additional Information](#additional-information)

<!-- /MarkdownTOC -->

# Introduction

This document describes and specifies the implementation offered by DK Hostmaster A/S for interaction with the central registry for the ccTLD dk using the WHOIS Service. It is primarily aimed at a technical audience, and the reader is required to have prior knowledge of the WHOIS protocol and possibly DNS registration.

The WHOIS service in not optimal for structured querying, both due to the lack of structure in the protocol specification and due to the constraints on the public service offered by DK Hostmaster. If you are a registrar, you might be interested in [the DK Hostmaster Domain Availability Service (DAS)](https://github.com/DK-Hostmaster/das-service-specification) as an alternative.

# About this Document

This specification describes version 2 (2.0.x) of the DK Hostmaster WHOIS Implementation. Future releases will be reflected in updates to this specification, please see the document history section below.
The document describes the current DK Hostmaster WHOIS implementation, for more general documentation on the used protocols and additional information please refer to the RFCs and additional resources in the References and Resources chapters below.
Any future extensions and possible additions and changes to the implementation are not within the scope of this document and will not be discussed or mentioned throughout this document.

Printable version can be obtained via [this link](https://gitprint.com/DK-Hostmaster/whois-service-specification/blob/master/README.md), using the gitprint service.

## License

This document is copyright by DK Hostmaster A/S and is licensed under the MIT License, please see the separate LICENSE file for details.

## Document History

* 1.0 2016-04-27
  * Initial revision

# The .dk Registry in Brief

DK Hostmaster is the registry for the ccTLD for Denmark (dk). The current model used in Denmark is based on a sole registry, with DK Hostmaster maintaining the central DNS registry.

The WHOIS service offered by DK Hostmaster A/S aims to adhere to the WHOIS standard (see also [RFC:3912]).

# Implementation Limitations

In general the service is not localized and all WHOIS information is provided in English. 

## Encoding

The service supports the following encodings:

- ISO8859-1 (default)
- Punycode (see also [RFC:3492])
- UTF-8

Please see the section on service for more information on how to utilize this.

## Rate Limiting

We only allow a certain number of requests per minute. We reserve the right to adjust the rate limit in order to provide a high quality of service. 

Current limit is set to 1 request per second.

In addition the service only allow 1 TCP-connection per. (IPv4)/24.

Meaning that `192.0.2.41` and `192.0.2.52` can not have simultanous connections, but `192.0.2.41` and `192.0.3.52` can.

# Service

## Domain name request

This is an example of a standard inquiry for a domain name

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
| DNS | Version of the domain name inquired used in DNS, punycode for IDNA domain names [RFC:3492] |
| Registered | Date of registration [ISO-8601] |
| Expires | Date of expiration [ISO-8601] |
| Registration period | Registration period (`1`, `2`, `3` or `5` years) |
| VID | Indication whether VID service is active, values either `yes` or `no` |
| Dnssec | Indication whether DNSSEC service is active, values either `Signed delegation` or `Unsigned delegation` |
| Status | Status of the domain name |
| Nameservers | List of nameservers, serving the inquired domain name |

### Example query using punycode

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

### Example query with domain marked for deletion

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

### Example query extracting handles

DK Hostmaster's WHOIS support extracting handles associated with a given domain.

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

### Example query extracting anonymous handles

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

## Host name request

You can inquire nameserver hosts.

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

## Handle request

In addition to domain and hostname inquiries, you can inquire handles (contact-ids).

### Example query for public handle

#### Request

```bash
$ whois -c dk DKHM1-DK
```

Please note the specification on the country code.

#### Response

```
Handle:               DKHM1-DK
Name:                 DK HOSTMASTER A/S
Address:              Kalvebod Brygge 45, 3.
Postalcode:           1560
City:                 København V
Country:              DK
```

### Example query for anonymous handle

#### Request

```bash
$ whois -c dk ANON-DK
```

#### Response

```
Handle:               ***N/A***
```

## Additional Help 

Additional help can be obtainted on the command line using the following command:

```bash
$ whois -h whois.dk-hostmaster.dk HELP
```

# References

Here is a list of documents and references used in this document

* General Terms and Conditions: https://www.dk-hostmaster.dk/fileadmin/filer/pdf/generelle_vilkaar/general-conditions.pdf
* RFC: 3492 Punycode: A Bootstring encoding of Unicode for Internationalized Domain Names in Applications (IDNA): https://tools.ietf.org/html/rfc3492
* RFC: 3912 WHOIS Protocol Specification: https://tools.ietf.org/html/rfc3912
* Documentation on the format of a domain name with the DK Hostmaster A/S registry: https://www.dk-hostmaster.dk/english/technical-administration/forms/register-domainname/

# Resources

Resources for DK Hostmaster WHOIS support can be found below.

## Mailing list

DK Hostmaster operates a mailing list for discussion and inquiries  about the DK Hostmaster WHOIS service. To subscribe to this list, write to the address below and follow the instructions. Please note that the list is for technical discussion only, any issues beyond the technical scope will not be responded to, please send these to the contact issue reporting address below and they will be passed on to the appropriate entities within DK Hostmaster A/S.

* `whois-discuss+subscribe@liste.dk-hostmaster.dk`

## Issue Reporting

For issue reporting related to this specification, the WHOIS implementation or the production environment, please contact us.  You are of course welcome to post these to the mailing list mentioned above, otherwise use the address specified below:

 * `tech@dk-hostmaster.dk`

## Additional Information

The DK Hostmaster website:

  * `https://www.dk-hostmaster.dk/english/technical-administration/tech-notes/whois-service/`

[RFC:3492]: https://tools.ietf.org/html/rfc3492

[RFC:3912]: https://tools.ietf.org/html/rfc3912

[ISO-8601]: https://en.wikipedia.org/wiki/ISO_8601
