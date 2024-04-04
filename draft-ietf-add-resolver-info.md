---
title: "DNS Resolver Information"
abbrev: "DNS Resolver Information"
category: std

docname: draft-ietf-add-resolver-info-latest
submissiontype: IETF
number:
date:
consensus: true
v: 3
area: "Internet"
workgroup: "ADD"
keyword:
 - Transparency
 - User Experience
 - DNS server selection

author:
 -
    fullname: Tirumaleswar Reddy
    organization: Nokia
    country: India
    email: kondtir@gmail.com
 -
    fullname: Mohamed Boucadair
    organization: Orange
    country: France
    city: Rennes
    code: 35000
    email: mohamed.boucadair@orange.com

normative:

informative:
  RRTYPE:
    title: Resource Record (RR) TYPEs
    author:
      org: IANA
    target: https://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml
    date: false

  IANA-DNS:
    title: Domain Name System (DNS) Parameters
    author:
      org: IANA
    target: http://www.iana.org/assignments/dns-parameters/dns-parameters.xhtml#dns-parameters-4
    date: false

--- abstract

   This document specifies a method for DNS resolvers to publish
   information about themselves.  DNS clients can use the resolver
   information to identify the capabilities of DNS resolvers. How DNS clients use such an information is beyond the scope of this document.

--- middle

#  Introduction

   Historically, DNS clients selected and communicated with recursive resolvers without needing to know anything about the features
   enabled by these resolvers. However, recent developments (e.g., Extended Error Reporting {{!RFC8914}} or encrypted DNS) and practices (e.g., filtering behaviors) imply that earlier assumptions no longer generally apply. Typically, DNS clients can discover and authenticate encrypted DNS resolvers provided by a local network (e.g., using the Discovery of Network-designated Resolvers (DNR) {{!RFC9463}} and the Discovery of Designated Resolvers (DDR) {{!RFC9462}}), however, these DNS clients can't retrieve
   information from the discovered recursive resolvers about their capabilities to feed the resolver selection process. Instead of depending on opportunistic approaches, DNS clients need a more reliable mechanism to discover the features that are configured on these resolvers.

   This document fills that void by specifying a method for stub
   resolvers to retrieve such information.  To that aim, a new resource record (RR) type
   is defined for DNS clients to query the recursive resolvers.  The
   initial information that a resolver might want to expose is defined in
   {{key-val}}. That information is scoped to cover properties that are used to infer privacy and transparency policies of a resolver. Other information can be registered in the future per the guidance in {{key-reg}}.

   Retrieved information can be used to feed the resolver selection procedure. For example, the resolver selection procedure may use the retrieved information to prioritize privacy-preserving resolvers over those that don't enable QNAME minimization {{!RFC9156}}. However, that selection procedure is out of the scope of this document. Once a resolver is selected, this document does not interfere with DNS operations with that resolver.



#  Terminology

{::boilerplate bcp14-tagged}

This document makes use of the terms defined in {{?RFC8499}}. The following additional terms are used:

Encrypted DNS:
: Refers to a DNS scheme where DNS exchanges are
   transported over an encrypted channel between a DNS client and server (e.g.,
   DNS over HTTPS (DoH) {{?RFC8484}}, DNS over TLS (DoT) {{?RFC7858}}, or
   DNS over QUIC (DoQ) {{?RFC9250}}).

Encrypted DNS resolver:
: Refers to a DNS resolver that supports any encrypted DNS scheme.

Reputation:
: "The estimation in which an identifiable actor is held, especially by the
   community or the Internet public generally" ({{Section 1 of ?RFC7070}}).


# Retrieving Resolver Information {#retreive}

   A DNS client that wants to retrieve the resolver information may
   use the RR type "RESINFO" defined in this document. The content of the RDATA in a
   response to a query for RESINFO RR QTYPE is defined in {{key-val}}. If the resolver understands the
   RESINFO RR type, the RRSet MUST have exactly one record. Invalid records MUST be silently ignored by DNS clients. RESINFO is a property of the resolver and is not subject to recursive resolution.

   A DNS client can retrieve the resolver information using the RESINFO
   RR type and the QNAME of the domain name that is used to authenticate the
   DNS resolver (referred to as the Authentication Domain Name (ADN) in DNR {{!RFC9463}}).

   If the Special-Use Domain Name "resolver.arpa", defined in {{!RFC9462}}, is used to
   discover an encrypted DNS resolver, the client can retrieve the resolver information
   using the RESINFO RR type and QNAME of "resolver.arpa". In this case, a client has to contend
   with the risk that a resolver does not support RESINFO. The resolver might
   pass the query upstream, and then the client can receive a positive RESINFO response either
   from a legitimate DNS resolver or an attacker.

   The DNS client MUST set the Recursion Desired (RD) bit of
   the query to 0. The DNS client MUST discard the response if the
   AA flag in the response is set to 0, indicating that the DNS resolver is not
   authoritative for the response.

   If a group of resolvers is sharing the same ADN and/or anycast address, then these instances SHOULD expose a consistent RESINFO.

#  Format of the Resolver Information {#format}

   The resolver information record uses the same format as DNS TXT records.
   The format rules for TXT records are defined in
   the base DNS specification ({{Section 3.3.14 of !RFC1035}}) and further
   elaborated in the DNS-based Service Discovery (DNS-SD) specification
   ({{Section 6.1 of !RFC6763}}). The recommendations to limit the TXT record size are
   discussed in {{Section 6.1 of !RFC6763}}.

   Similar to DNS-SD, the RESINFO RR type uses "key/value" pairs to
   convey the resolver information.  Each "key/value" pair is encoded
   using the format rules defined in {{Section 6.3 of !RFC6763}}.  Using
   standardized "key/value" syntax within the RESINFO RR type makes it
   easier for future keys to be defined.  If a DNS client sees unknown
   keys in a RESINFO RR type, it MUST silently ignore them.  The same
   rules for the keys as those defined in {{Section 6.4 of !RFC6763}} MUST
   be followed for RESINFO.

   Resolver information keys MUST either be defined in the IANA registry ({{key-reg}}) or begin
   with the substring "temp-" for names defined for local use only.

#  Resolver Information Keys/Values {#key-val}

   The following resolver information keys are defined:

   qnamemin:
   : The presence of this key indicates that the DNS resolver supports QNAME minimisation {{!RFC9156}}
      to improve DNS privacy.  Note that, per the
      rules for the keys defined in {{Section 6.4 of !RFC6763}}, if there
      is no '=' in a key, then it is a boolean attribute, simply
      identified as being present, with no value.

     The presence of this key indicates that the DNS resolver is configured to minimise the amount of privacy-sensitive data sent to an authoritative name server.

     This is an optional attribute.

   exterr:
   : If the DNS resolver supports extended DNS errors (EDE) option
      {{!RFC8914}} to return additional information about the cause of DNS
      errors, the value of this key lists the possible extended DNS
      error codes that can be returned by this DNS resolver. A value can be an individual EDE or a range of EDEs. Range values MUST be identified by "-".  When
      multiple non-contiguous values are present, these values MUST be comma-separated.

      Returned EDEs (e.g., Blocked (15), Censored (16), and Filtered (17)) indicate whether the DNS resolver is configured to reveal the reason why a query was filtered/blocked, when such event happens.

      This is an optional attribute.

   infourl:
   : An URL that points to the generic unstructured resolver
      information (e.g., DoH APIs supported, possible HTTP status codes
      returned by the DoH server, or how to report a problem) for
      troubleshooting purposes. The server that exposes such information is called "resolver information server".

      The resolver information server MUST support the content-type 'text/html'.  The DNS
      client MUST reject invalid the URL if the scheme is not "https". Invalid URLs MUST be ignored.  The URL
      SHOULD be treated only as diagnostic information for IT staff.  It
      is not intended for end user consumption as the URL can possibly
      provide misleading information. A DNS client MAY choose to display
      the URL to the end user, if and only if the encrypted resolver has
      sufficient reputation, according to some local policy (e.g., user
      configuration, administrative configuration, or a built-in list of
      respectable resolvers).

      This key can be used by IT staff to retrieve other useful information about the resolver and also the procedure to report problems (e.g., invalid filtering).

      This is an optional attribute.

   New keys can be defined as per the procedure defined in {{key-reg}}.

# An Example

{{ex-1}} shows an example of a published resolver information record.

~~~~
resolver.example.net. 7200 IN RESINFO qnamemin exterr=15-17
                      infourl=https://resolver.example.com/guide
~~~~
{: #ex-1 title='An Example of Resolver Information Record' artwork-align="center"}

As mentioned in {{retreive}}, a DNS client that discovers the ADN "resolver.example.net"
of its resolver using DNR will issue a query for RESINFO RR QTYPE for that ADN
and will learn that the resolver supports:

* QNAME minimisation,
* Blocked (15), Censored (16), and Filtered (17) EDEs, and
* that more information can be retrieved from https://resolver.example.com/guide.

#  Security Considerations

DNS clients communicating with discovered DNS resolvers MUST use one of the following measures
to prevent DNS response forgery attacks:

1. Establish an authenticated secure connection to the DNS resolver.
2. Implement local DNSSEC validation ({{Section 10 of ?RFC8499}}) to verify the authenticity of the resolver information.

It is important to note that, of these two measures, only the first one can apply to queries for 'resolver.arpa'.

An encrypted resolver may return incorrect information in RESINFO. If the client cannot validate the attributes received from the resolver, that will be used for resolver selection or displayed to the end-user, the client should process those attributes only if the encrypted resolver has sufficient reputation according to local policy (e.g., user configuration, administrative configuration, or a built-in list of reputable resolvers). This approach limits the ability of a malicious encrypted resolver to cause harm with false claims.

#  IANA Considerations

> Note to the RFC Editor: Please update "RFCXXXX" occurrences with the RFC number to be assigned to this document.

##  RESINFO RR Type

   This document requests IANA to update this entry from the
   "Resource Record (RR) TYPEs" registry of the "Domain Name System
   (DNS) Parameters" registry group available at {{RRTYPE}}:

~~~~
Type: RESINFO
Value: 261
Meaning: Resolver Information as Key/Value Pairs
Reference: RFCXXXX
~~~~

## DNS Resolver Information Key Registration {#key-reg}

   This document requests IANA to create a new registry entitled "DNS
   Resolver Information Keys" under the "Domain Name System (DNS) Parameters" registry group ({{IANA-DNS}}).  This new registry contains definitions of
   the keys that can be used to provide the resolver information.

   The registration procedure is Specification Required ({{Section 4.6 of !RFC8126}}). Designated experts should carefully
   consider the security implications of allowing a resolver to include new keys in this registry. Additional
   considerations are provided in {{sec-de}}.

   The structure of the registry is as follows:

   Name:
   : The key name.  The name MUST conform to the definition in
      {{format}} of this document.  The IANA registry MUST NOT register names that begin
      with "temp-", so these names can be used freely by any
      implementer.

   Description:
   : A description of the registered key.

   Specification:
   : The reference specification for the registered
      element.

   The initial content of this registry is provided in {{initial}}.


| Name   |  Description | Specification |
|:------:|:------------|:-------------:|
| qnamemin | The presence of the key name indicates that QNAME minimization is enabled | RFCXXXX |
| exterr   | Lists the set of enabled extended DNS errors. It must be an INFO-CODE decimal value in the "Extended DNS Error Codes" registry.  | RFCXXXX   |
| infourl  | Provides an URL that points to an unstructured resolver information that is used for troubleshooting | RFCXXXX     |
{: #initial title='Initial RESINFO Registry'}

##  Guidelines for the Designated Experts {#sec-de}

   It is suggested that multiple designated experts be appointed for
   registry change requests.

   Criteria that should be applied by the designated experts include
   determining whether the proposed registration duplicates existing
   entries and whether the registration description is clear and fits
   the purpose of this registry.

   Registration requests are evaluated within a three-week review period on the advice of
   one or more designated experts.  Within the review period, the
   designated experts will either approve or deny the registration
   request, communicating this decision to IANA.
   Denials should include an explanation and, if applicable, suggestions
   as to how to make the request successful.

--- back

# Acknowledgments
{:numbered="false"}

   This specification leverages the work that has been documented in
   {{?I-D.pp-add-resinfo}}.

   Thanks to Tommy Jensen, Vittorio Bertola, Vinny Parla, Chris Box, Ben
   Schwartz, Tony Finch, Daniel Kahn Gillmor, Eric Rescorla, Shashank
   Jain, Florian Obser, Richard Baldry, and Martin Thomson for the discussion and
   comments.

   Thanks to Mark Andrews, Joe Abley, Paul Wouters, and Tim
   Wicinski for the discussion on the RR formatting rules.

   Special thanks to Tommy Jensen for the careful and thoughtful Shepherd review.

   Thanks to Johan Stenstam and Jim Reid for the dns-dir reviews, Ray Bellis for the RRTYPE allocation review, Arnt Gulbrandsen for the ART review,
   and Mallory Knodel for the gen-art review.

   Thanks to Eric Vyncke for the AD review.

   Thanks to Gunter Van de Velde, Erik Kline, Paul Wouters, Orie Steele, Warren Kumari, and Murray Kucherawy for the IESG review.
