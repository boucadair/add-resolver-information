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
   information to identify the capabilities of DNS resolvers. How such an information is then used by DNS clients is out of the scope of the document.

--- middle

#  Introduction

   Historically, DNS stub resolvers communicated with upstream
   resolvers without needing to know anything about the features
   supported by these recursive resolvers.  As more and more recursive
   resolvers expose different features that may impact delivered DNS
   services, means to help stub resolvers to identify the capabilities of
   resolvers are valuable.  Typically, stub resolvers can discover
   and authenticate encrypted DNS resolvers provided by a local network,
   for example, using the techniques specified in {{?I-D.ietf-add-dnr}} and
   {{?I-D.ietf-add-ddr}}.  However, these stub resolvers need a mechanism to
   retrieve information from the discovered recursive resolvers about
   their capabilities.

   This document fills that void by specifying a method for stub
   resolvers to retrieve such information.  To that aim, a new resource record (RR) type
   is defined for stub resolvers to query the recursive resolvers.  The
   information that a resolver might want to expose is defined in
   {{key-val}}.

   Retrieved information can be used to feed the server selection
   procedure. However, that selection procedure is out of the scope of this document.

#  Terminology

{::boilerplate bcp14-tagged}

   This document makes use of the terms defined in {{?RFC8499}}.

   'Encrypted DNS' refers to a DNS scheme where DNS exchanges are
   transported over an encrypted channel between a DNS client and server (e.g.,
   DNS over HTTPS (DoH) {{?RFC8484}}, DNS over TLS (DoT) {{?RFC7858}}, or
   DNS over QUIC (DoQ) {{?RFC9250}}).

# Retrieving Resolver Information

   A stub resolver that wants to retrieve the resolver information may
   use the RR type "RESINFO" defined in this document.

   The content of the RDATA in a response to a RESINFO RR type query is defined in
   {{key-val}}.  If the resolver understands the RESINFO RR type, the
   RRSet in the Answer section MUST have exactly one record.

   A DNS client can retrieve the resolver information using the RESINFO
   RR type and the QNAME of the domain name that is used to authenticate the
   DNS resolver (referred to as the Authentication Domain Name (ADN) in {{?I-D.ietf-add-dnr}}).

   If the Special-Use Domain Name "resolver.arpa", defined in
   {{?I-D.ietf-add-ddr}}, is used to discover an encrypted DNS resolver, the
   client can retrieve the resolver information using the RESINFO RR
   type and QNAME of "resolver.arpa".  For DDR, the client has to
   validate a designated resolver using a secure connection (Section 4.2
   of {{?I-D.ietf-add-ddr}}) and then issue a DNS request to retrieve the
   resolver information.

#  Format of the Resolver Information {#format}

   The resolver information uses the same format as DNS TXT records.
   The motivation for using the same format as TXT records is to convey a
   small amount of useful information about a DNS resolver.  As a
   reminder, the format rules for TXT records are defined in
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

   Keys MUST either be defined in the IANA registry ({{key-reg}}) or begin
   with the substring "temp-" for names defined for local use only.

#  Resolver Information Keys/Values {#key-val}

   The following resolver information keys are defined:

   qnamemin:
   : If the DNS resolver supports QNAME minimisation {{!RFC9156}}
      to improve DNS privacy, the key is present.  Note that, as per the
      rules for the keys defined in {{Section 6.4 of !RFC6763}}, if there
      is no '=' in a key, then it is a boolean attribute, simply
      identified as being present, with no value.

      This is an optional attribute.

   exterr:
   : If the DNS resolver supports extended DNS errors (EDE) option
      {{!RFC8914}} to return additional information about the cause of DNS
      errors, the value of this key lists the possible extended DNS
      error codes that can be returned by this DNS resolver.  When
      multiple values are present, these values MUST be comma-separated.

      This is an optional attribute.

   infourl:
   : An URL that points to the generic unstructured resolver
      information (e.g., DoH APIs supported, possible HTTP status codes
      returned by the DoH server, how to report a problem) for
      troubleshooting purposes.

      The server MUST support the content-type 'text/html'.  The DNS
      client MUST reject the URL if the scheme is not "https".  The URL
      SHOULD be treated only as diagnostic information for IT staff.  It
      is not intended for end user consumption as the URL can possibily
      provide misleading information.  A DNS client MAY choose to display
      the URL to the end user, if and only if the encrypted resolver has
      sufficient reputation, according to some local policy (e.g., user
      configuration, administrative configuration, or a built-in list of
      respectable resolvers).

      This is a an optional attribute.  For example, a DoT server may
      not want to host an HTTPS server.

   sig:
   : It contains the signature of the RESINFO RR excluding the "sig"	
	   attribute.  The signature is generated by the encrypted DNS server	
		using all of the attributes in the RESINFO RR except the "sig"	
		attribute.  The signature algorithm MUST be compatible with the	
		public key in the DNS server's end-entity certificate.  As a	
		reminder, the server's end-entity certificate's public key will be	
		compatible with the selected authentication algorithm from the	
		client's "signature_algorithms" TLS extension (Section 4.4.2.2 of
      {{!RFC8446}}).

      This is a an optional attribute.

   New keys can be defined as per the procedure defined in {{key-reg}}.

   {{ex-1}} shows an example of a published resolver information record:

~~~~
resolver.example.net. 7200 IN RESINFO qnamemin exterr=15,16,17
                      infourl=https://resolver.example.com/guide
~~~~
{: #ex-1 title='An Example of Resolver Information Record' artwork-align="center"}


#  Security Considerations

   In order to prevent DNS response forgery attacks, DNS clients MUST either use an authenticated secure
   connection to the DNS resolver or use local DNSSEC validation ({{Section 10 of ?RFC8499}}) to retrieve the resolver information.

   If a resolver supports DDR but does not support RESINFO, the client	
	can receive a positive RESINFO response from an upstream DNS server	
	or an attacker.  To prevent such an attack, resolvers supporting DDR	
	MUST convey the "sig" attribute.  DNS clients using DDR for encrypted	
	resolver discovery querying for a RESINFO RR MUST validate the	
	signature in the "sig" attribute for data origin authentication.  If	
	the signature validation fails, the DNS client MUST reject the	
	RESINFO RR.


#  IANA Considerations

> Note to the RFC Editor: Please update "RFCXXXX" occurences with the RFC number to be assigned to this document.

##  RESINFO RR Type

   This document requests IANA to register a new value from the
   "Resource Record (RR) TYPEs" registry of the "Domain Name System
   (DNS) Parameters" registry group available at {{RRTYPE}}:

~~~~
Type: RESINFO
Value: TBD
Meaning: Resolver Information as Key/Value Pairs
Reference: RFCXXXX
~~~~

## DNS Resolver Information Key Registration {#key-reg}

   This document requests IANA to create a new registry entitled "DNS
   Resolver Information Keys" under the "Domain Name System (DNS) Parameters" registry group ({{IANA-DNS}}).  This new registry contains definitions of
   the keys that can be used to provide the resolver information.

   The registration procedure is Specification Required ({{Section 4.6 of !RFC8126}}).

   The structure of the registry is as follows:

   Name:
   : The key name.  The name MUST conform to the definition in
      {{format}} of this document.  The IANA registry MUST NOT register names that begin
      with "temp-", so these names can be used freely by any
      implementer.

   Value Type:
   : The type of the value to be used in the key.

   Description:
   : A description of the registered key.

   Specification:
   : The reference specification for the registered
      element.

   The initial content of this registry is provided in {{initial}}.


| Name   | Value Type | Description | Specification |
|:------:|:----------:|:------------|:-------------:|
| qnamemin | None | The presence of the key name indicates that 'qnameminimization' is enabled | RFCXXXX |
| exterr   | 16-bit unsigned integer  | Lists the set of supported extended DNS errors. It must be a value in the "Extended DNS Error Codes" registry.  | RFCXXXX   |
| infourl  | string  | Provides an URL that points to an unstructured resolver information that is used for troubleshooting | RFCXXXX     |
| sig      | binary  | Includes a signature of RESINFO RR for data origin authentication | RFCXXXX     |
{: #initial title='Initial RESINFO Registry'}

--- back

# Acknowledgments
{:numbered="false"}

   This specification leverages the work that has been documented in
   {{?I-D.pp-add-resinfo}}.

   Thanks to Tommy Jensen, Vittorio Bertola, Vinny Parla, Chris Box, Ben
   Schwartz, Tony Finch, Daniel Kahn Gillmor, Eric Rescorla, Shashank
   Jain, Florian Obser, and Richard Baldry for the discussion and
   comments.

   Thanks to Mark Andrews, Joe Abley, Paul Wouters, Tim
   Wicinski, and Steffen Nurpmeso for the discussion on the RR
   formatting rules.

   Thanks to Tommy Jensen for the Shepherd review.

   Thanks to Johan Stenstam for the dns-dir review.
