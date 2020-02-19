# WoT Discovery

Two-phase directory-based discovery.

## Authors
* Michael McCool, Intel Corporation, michael.mccool@intel.com
* Carsten Bormann, Universität Bremen TZI, cabo@tzi.org
* Kunihiko Toumura (東村邦彦), Hitachi Ltd., kunihiko.toumura.yv@hitachi.com

## Abstract

W3C Web of Things (WoT) Thing Descriptions (TD) provide metadata that
describe the affordances and semantics of IoT devices and services.
However, before a TD can be interpreted and used, it needs to be found.
This proposals describes a two-phase, directory-based discovery process 
for WoT TDs that also 
provides appropriate security and privacy access controls, supports 
multiple "first contact" mechanisms, supports advanced querying
capabilities, and supports both local and global discovery use cases.
This discovery process is based on CoRE Resource Directory {{CoRE-RD}}
with authentication following ACE {{ACE}} and can be accessed over either
CoAP {{CoAP}} or HTTP {{HTTP1.1}}.  To preserve privacy, the system
separates the discovery process into two stages, open
introduction protocols for first contact and a directory service that
provides querying capabilities and access to metadata, but only after
authentication and authorization.  Any introduction mechanism may be
used that can provide the address of a directory service but we will
specify several useful ones.  

## Introduction

This MAY {{?RFC2119}} be useful.

## Architecture
Two-phase architecture: open and lightweight "first contact" introduction 
mechanism, following by an authenticated and authorization step that gates
access to an "exploration" directory mechanism that provides detailed metadata.

The first contact mechanism should be designed to preserve privacy, and will
essentially be designed just to provide links to directory services without
revealing what devices are indexed by those directories.

The exploration mechanism should be designed to be scalable, and implementable
both on small devices (with a small amount of data, such as a single TD) and on
larger systems managing millions of devices.   It should support searches of
various types.

### Introduction Phase: First Contact Mechanism
* Obtain address of directory service - only
* Address should not leak any other metadata, eg type of devices
* Can have multiple mechanisms for introduction
  - Local: QR code. mDNS, DNS-SD, DHCP, QR code, Bluetooth beacon, etc.
  - Global: search engine
  - Self: Well-known addresses, eg “.well-known/core?rt=wot.directory”

### Exploration Phase: Directory 
* Mutual authentication and secure connection required, and then...
* Queryable Directory service
* Multiple query type supported
  - Location (may be necessary to use for introduction)
  - Keywords
  - Templates
  - SPARQL semantic query
  - GraphQL
  - By-Example (templates)
* External service: 
  - Need registration sub-API
  - Timeouts
  - Access control
  - Self (peer-to-peer): same query API, but no public registration API
  - Mutable IDs need way to notify registered users of changes
  
## Design Options

### Use an Existing Mechanism for Discovery
TODO

### Use Existing Mechanisms for Introductions and a New Mechanism for Exploration
TODO

### Design a new Mechanism for both Introduction and Exploration
TODO

## CoRE Discovery and Directories
CoRE proposes several discovery and directory mechanisms.
* Things by CoAP multicast on well-known URI, for example, `/.well-known/core?rt=w3c.wot.thing-description`
* Discovery of Resource Directories use a similar approach: `/.well-known/core/?rt=core.rd*`
These, however, only support discovery in local network segments via a (non-scalable) broadcast mechanism.
This gives a rough localization as well, but delimited by the topology of the network segmentation.

## Registration of TDs to CoRE Resource Directories
```
POST coap://rd.example.com/rd?ep=(Thing-ID)
Content-Format: application/td+json
```
* Content-format ID is t.b.d., but needs to be in the 256-9999 range.
* Registration with a directory will also support a time-to-live.
  The Thing will have to periodically refresh the TD and/or send a keep-alive.
* Updates to existing TDs should be supported to allow changes to TDs, 
  including mutable IDs.
* HTTP should also be supported.

## Retrieve TD from CoRE Resource Directory
```
GET coap://rd.example.com/rd-lookup?rt=MyLampThing
Accept: application/td+json
```
* Technically CoRE-RD only stores links to resources, not the resources
  themselves.  A "CoRE Data Hub" combined with the RD may be required to actually
  allow storage and retrieval of TDs themselves.
* HTTP should also be supported

## Authentication
* Authentication should be through either OAuth2 (for HTTP) or ACE-OAuth2 for CoAP.
* Mutual authentication and confidentialityt should be supported via either TLS or DTLS.
* The secure connection should be established before query parameters are set to avoid
  inferencing of PII from query parameters.
* OAuth2 scopes should be pre-defined for a suitable set of roles and tasks associated with those roles

## Other Introduction Mechanisms
The IPv6 Authoritative Border Router Option, DHCP, DNS-SRV, QR codes, Bluetooth beacons
can also be used for discovery of Things and Directory Services.  
All such mechanisms are acceptable as long at they satisfy the following properties:
* No metadata about the Things are distributed; only an address of a directory service should be returned.
* If a link directly to a Thing is returned, the address should not have any implied metadata.
   - For example, a URL should not be used with a human-readable name.
* Generally, access to full directory services should only be provided after mutual authentication and authorization
   - Privacy: both TDs and query parameters for searches can be used to infer PII information
     even if the TDs do not explicitly include such information
   - Security: directory queries can be expensive and so should only be invoked for authorized users to mitigate
     denial-of-service attacks.  Mutual authentication also mitigates man-in-the-middle attacks based on manipulation
     of TD metadata.
Introduction services that are global in nature (eg DNS) may allow additional query parameters to identify
the location of interest.  Other forms of metadata should however be avoided and the granularity of location accuracy
should be limited to avoid the inference of PII.

