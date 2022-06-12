---
coding: utf-8

title: Ephemeral Diffie-Hellman Over COSE (EDHOC) and Object Security for Constrained Environments (OSCORE) Profile for Authentication and Authorization for Constrained Environments (ACE)
abbrev: CoAP-EDHOC-OSCORE
docname: draft-selander-ace-edhoc-oscore-profile-latest
category: std

ipr: trust200902
area: Security
workgroup: ACE Working Group
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
-
    ins: G. Selander
    name: Göran Selander
    org: Ericsson
    email: goran.selander@ericsson.com
-
    ins: J. Preuß Mattsson
    name: John Preuß Mattsson
    org: Ericsson
    email: john.mattsson@ericsson.com

-
    ins: M. Tiloca
    name: Marco Tiloca
    org: RISE
    email: marco.tiloca@ri.se

-
    ins: R. Höglund
    name: Rikard Höglund
    org: RISE
    email: rikard.hoglund@ri.se

normative:
  RFC2119:
  RFC8174:
  RFC6749:
  RFC7252:
  RFC7519:
  RFC7800:
  RFC8126:
  RFC8392:
  RFC8610:
  RFC8613:
  RFC8742:
  RFC8747:
  RFC8949:
  I-D.ietf-ace-oauth-authz:
  I-D.ietf-ace-oauth-params:
  I-D.ietf-lake-edhoc:
  I-D.ietf-core-oscore-edhoc:
  I-D.ietf-cose-x509:
  I-D.ietf-cose-cbor-encoded-cert:

informative:
  RFC4949:
  RFC7231:
  RFC8446:
  RFC9147:
  I-D.ietf-ace-oscore-profile:
  I-D.ietf-core-oscore-key-update:

entity:
  SELF: "[RFC-XXXX]"

--- abstract

This document specifies a profile for the Authentication and Authorization for Constrained Environments (ACE) framework.
It utilizes Ephemeral Diffie-Hellman Over COSE (EDHOC) for achieving mutual authentication between an OAuth 2.0 Client and Resource Server, and it binds an authentication credential of the Client to an OAuth 2.0 access token.
EDHOC also establishes an Object Security for Constrained RESTful Environments (OSCORE) Security Context, which is used to secure communications when accessing protected resources according to the authorization information indicated in the access token.
A resource-constrained server can use this profile to delegate management of authorization information to a trusted host with less severe limitations regarding processing power and memory.

--- middle


# Introduction

This document defines the "coap_edhoc_oscore" profile of the ACE framework {{I-D.ietf-ace-oauth-authz}}. This profile addresses a "zero-touch" constrained setting where trusted operations can be performed with low overhead without endpoint specific configurations.

Like in the "coap_oscore" profile {{I-D.ietf-ace-oscore-profile}}, also in this profile a client (C) and a resource server (RS) use the Constrained Application Protocol (CoAP) {{RFC7252}} to communicate, and Object Security for Constrained RESTful Environments (OSCORE) {{RFC8613}} to protect their communications. Besides, the processing of requests for specific protected resources is identical to what is defined in the "coap_oscore" profile.

When using this profile, C accesses protected resources hosted at the RS with the use of an access token issued by a trusted authorization server (AS) and bound to an authentication credential of C. This differs from the "coap_oscore" profile, where the access token is bound to a symmetric key used to derive OSCORE keying material. As recommended in {{I-D.ietf-ace-oauth-authz}}, this document recommends the use of CBOR Web Tokens (CWTs) {{RFC8392}} as access tokens.

The authentication and authorization workflow requires C and the RS to have access to each other's authentication credentials. In particular, C obtains both an access token and an authentication credential of the RS from the AS, while the RS can obtain the access token for C (and the authentication credential of C included therin) through different possible options. If the RS successfully verifies the access token, then C and the RS run the Ephemeral Diffie-Hellman Over COSE (EDHOC) protocol {{I-D.ietf-lake-edhoc}}, with each peer using its own and the other peer's authentication credential.

Once completed the EDHOC execution, C and the RS are mutually authenticated and establish an OSCORE Security Context to protect following communications, while the RS achieves proof of possession of the private key of C. Instead, C achieves proof of possession of the private key of the RS either once completed the EDHOC execution (if an optional, fourth EDHOC message is sent by the RS) or after the first message exchange using OSCORE.

Authentication credentials can be a raw public key, e.g., encoded as a CWT Claims Set (CCS, {{RFC8392}}); or a public key certificate, e.g., encoded as an X.509 certificate or as a CBOR encoded X.509 certificate (C509, {{I-D.ietf-cose-cbor-encoded-cert}}); or a different type of data structure containing (or uniquely referring to) the public key of the peer in question.

The ACE protocol establishes what those authentication credentials are, and may transport the actual authentication credentials by value or rather uniquely refer to them. If an actual authentication credential is pre-provisioned or can be obtained over less constrained links, then it suffices that ACE provides a unique reference such as a certificate hash (e.g., by using the COSE header parameter "x5t", see {{I-D.ietf-cose-x509}}). This is in the same spirit as EDHOC, where the authentication credentials may be transported or referenced in the ID_CRED_x message fields (see Section 3.5.3 of {{I-D.ietf-lake-edhoc}}).

Generally, the AS and RS are likely to have trusted access to each other's authentication credentials, since the AS acts on behalf of the RS as per the trust model of ACE. Also, the AS needs to have some information about C, including the respective authentication credential, in order to identify C when it requests an access token and to determine what access rights it can be granted. However, the authentication credential of C may potentially be conveyed (or uniquely referred to) within the request sent to the AS.

## Terminology # {#terminology}

{::boilerplate bcp14}

Certain security-related terms such as "authentication", "authorization", "confidentiality", "(data) integrity", "Message Authentication Code (MAC)", "Hash-based Message Authentication Code (HMAC)", and "verify" are taken from {{RFC4949}}.

RESTful terminology follows HTTP {{RFC7231}}.

Readers are expected to be familiar with the terms and concepts defined in CoAP {{RFC7252}}, OSCORE {{RFC8613}} and EDHOC {{I-D.ietf-lake-edhoc}}.

Readers are also expected to be familiar with the terms and concepts of the ACE framework described in {{I-D.ietf-ace-oauth-authz}} and in {{I-D.ietf-ace-oauth-params}}.

Terminology for entities in the architecture is defined in OAuth 2.0 {{RFC6749}}, such as client (C), resource server (RS), and authorization server (AS).  It is assumed in this document that a given resource on a specific RS is associated to a unique AS.

Note that the term "endpoint" is used here, as in {{I-D.ietf-ace-oauth-authz}}, following its OAuth definition, which is to denote resources such as token and introspect at the AS and authz-info at the RS. The CoAP {{RFC7252}} definition, which is "An entity participating in the CoAP protocol" is not used in this document.

The authorization information (authz-info) resource refers to the authorization information endpoint as specified in {{I-D.ietf-ace-oauth-authz}}. The term "claim" is used in this document with the same semantics as in {{I-D.ietf-ace-oauth-authz}}, i.e., it denotes information carried in the access token or returned from introspection.

Furthermore, this document refers to the following term.

* Token series - A series of access tokens sorted in chronological order of release and such that: they are all issued by the same AS; they are all issued to the same C and for the same RS; they are all issued together with the same authentication credential of the RS; and they are all bound to the same authentication credential of C used as proof-of-possession key. When an access token becomes invalid (e.g., due to its expiration or revocation), the token series it belongs to ends.

Concise Binary Object Representation (CBOR) {{RFC8949}}{{RFC8742}} and Concise Data Definition Language (CDDL) {{RFC8610}} are used in this document. CDDL predefined type names, especially bstr for CBOR byte strings and tstr for CBOR text strings, are used extensively in this document.

Examples throughout this document are expressed in CBOR diagnostic notation without the tag and value abbreviations.

# Protocol Overview {#overview}

This section gives an overview of how to use the ACE framework {{I-D.ietf-ace-oauth-authz}} together with the authenticated key establishment protocol EDHOC {{I-D.ietf-lake-edhoc}}. By doing so, a client (C) and a resource server (RS) generate an OSCORE Security Context {{RFC8613}} associated with authorization information, and use that Security Context to protect their communications. The parameters needed by C to negotiate the use of this profile with the authorization server (AS), as well as the OSCORE setup process, are described in detail in the following sections.

The RS maintains a collection of authentication credentials. These are related to OSCORE Security Contexts associated with authorization information for all the clients that the RS is communicating with. The authorization information is maintained as policy that is used as input to the processing of requests from those clients.

This profile requires C to retrieve an access token from the AS for the resources it wants to access on an RS, by sending an access token request to the token endpoint, as specified in Section 5.8 of {{I-D.ietf-ace-oauth-authz}}. The access token request and response MUST be confidentiality protected and ensure authenticity. The use of EDHOC and OSCORE between C and the AS is RECOMMENDED in this profile, in order to reduce the number of libraries that C has to support. However, other protocols fulfilling the security requirements defined in Section 5 of {{I-D.ietf-ace-oauth-authz}} MAY alternatively be used, such as TLS {{RFC8446}} or DTLS {{RFC9147}}.

Once C has retrieved the access token, C uploads it to the RS. To this end, there are two different options, as further detailed in this document.

* C posts the access token to the authz-info endpoint by using the mechanisms specified in Section 5.8 of {{I-D.ietf-ace-oauth-authz}}. If the access token is valid, the RS responds to the request with a 2.01 (Created) response, after which C initiates the EDHOC protocol by sending EDHOC message_1 to the RS. When using this profile, the communication with the authz-info endpoint is not protected, except for the update of access rights.

* C initiates the EDHOC protocol by sending EDHOC message_1 to the RS, specifying the access token as External Authorization Data (EAD) in the EAD_1 field of EDHOC message_1 (see Section 3.8 {{I-D.ietf-lake-edhoc}}). If the access token is valid and the processing of EDHOC message_1 is successful, the RS responds with EDHOC message_2, thus continuing the EDHOC protocol. This alternative cannot be used for the update of access rights.

When running the EDHOC protocol, C uses the authentication credential of the RS specified by the AS together with the access token, while the RS uses the authentication credential of C bound to and specified within the access token. If C and the RS complete the EDHOC execution successfully, they are mutually authenticated and they derive an OSCORE Security Context as per {{Section A.1 of I-D.ietf-lake-edhoc}}. Also, the RS associates the two used authentication credentials and the completed EDHOC execution with the derived Security Context. The latter is in turn associated with the access token and the access rights of C specified therein.

From then on, C effectively gains authorized and secure access to protected resources on the RS, for as long as the access token is valid. Until then, C can communicate with the RS by sending a request protected with the established OSCORE Security Context above. The Security Context is discarded when an access token (whether the same or a different one) is used to successfully derive a new Security Context for C, either by re-running EDHOC or by exchanging nonces and using the EDHOC-KeyUpdate function (see {{edhoc-key-update}}).

After the whole message exchange has taken place, C can contact the AS to request an update of its access rights, by sending a similar request to the token endpoint. This request also includes an identifier, which allows the AS to find the correct data it has previously shared with C. This specific identifier, encoded as a byte string, is assigned by the AS to a "token series" (see {{terminology}}). Upon a successful update of access rights, the new issued access token becomes the latest one of its token series. When the latest access token of a token series becomes invalid (e.g., when it expires or gets revoked), that token series ends.

An overview of the profile flow for the "coap_edhoc_oscore" profile is given in {{protocol-overview}}. The names of messages coincide with those of {{I-D.ietf-ace-oauth-authz}} when applicable.

~~~~~~~~~~~~~~~~~~~~~~~

   C                            RS                       AS
   |                            |                         |
   | <==== Mutual authentication and secure channel ====> |
   |                            |                         |
   | ------- POST /token  ------------------------------> |
   |                            |                         |
   | <-------------------------------- Access Token ----- |
   |                               + Access Information   |
   |                            |                         |
   | ---- POST /authz-info ---> |                         |
   |       (access_token)       |                         |
   |                            |                         |
   | <----- 2.01 Created ------ |                         |
   |                            |                         |
   | <========= EDHOC ========> |                         |
   |  Mutual authentication     |                         |
   |  and derivation of an      |                         |
   |  OSCORE Security Context   |                         |
   |                            |                         |
   |                /Proof-of-possession and              |
   |                Security Context storage/             |
   |                            |                         |
   | ---- OSCORE Request -----> |                         |
   |                            |                         |
   | <--- OSCORE Response ----- |                         |
   |                            |                         |
/Proof-of-possession            |                         |
and Security Context            |                         |
storage (latest)/               |                         |
   |                            |                         |
   | ---- OSCORE Request -----> |                         |
   |                            |                         |
   | <--- OSCORE Response ----- |                         |
   |                            |                         |
   |           ...              |                         |

~~~~~~~~~~~~~~~~~~~~~~~
{: #protocol-overview title="Protocol Overview"}


# Client-AS Communication # {#c-as-comm}

The following subsections describe the details of the POST request and response to the token endpoint between C and the AS.

In this exchange, the AS provides C with the access token, together with a set of parameters that enable C to run EDHOC with the RS. These especially include the authorization credential of the RS, namely AUTH\_CRED\_RS, as transported by value or uniquely referred to.

The proof-of-possession key (pop-key) bound to the access token and specified therein, as transported by value or uniquely referred to, MUST be the authentication credential of C, namely AUTH\_CRED\_C. This is provided by C to the AS in the POST request to the token endpoint, as transported by value or uniquely referred to, by means of the "req_cnf" parameter defined in {{I-D.ietf-ace-oauth-params}}.

The request to the token endpoint and the corresponding response can include EDHOC\_Information, which is a CBOR map object defined in {{edhoc-parameters-object}}. This object is transported in the "edhoc\_info" parameter registered in {{iana-oauth-params}} and {{iana-oauth-cbor-mappings}}.

## C-to-AS: POST to token endpoint # {#c-as}

The client-to-AS request is specified in {{Section 5.8.1 of I-D.ietf-ace-oauth-authz}}.

The client must send this POST request to the token endpoint over a secure channel that guarantees authentication, message integrity and confidentiality (see {{secure-comm-as}}).

An example of such a request is shown in {{token-request}}. In this example, C specifies its own authentication credential by reference, as the hash of an X.509 certificate carried in the "x5t" field of the "req\_cnf" parameter. In fact, it is expected that C can typically specify its own authentication credential by reference, since the AS is expected to obtain the actual authentication credential during an early client registration process or during a previous secure association establishment with C.

~~~~~~~~~~~~~~~~~~~~~~~
   Header: POST (Code=0.02)
   Uri-Host: "as.example.com"
   Uri-Path: "token"
   Content-Format: "application/ace+cbor"
   Payload:
   {
     "audience" : "tempSensor4711",
     "scope" : "read",
     "req_cnf" : {
       "x5t" : h'822E4879F2A41B510C1F9B'
     }
   }
~~~~~~~~~~~~~~~~~~~~~~~
{: #token-request title="Example of C-to-AS POST /token request for an access token."}

If C wants to update its access rights without changing an existing OSCORE Security Context, it MUST include EDHOC\_Information in its POST request to the token endpoint. In turn, EDHOC\_Information MUST include the "id" field, carrying a CBOR byte string containing the identifier of the token series to which the current, still valid access token shared with the RS belongs to. This POST request MUST omit the "req_cnf" parameter.

This identifier is assigned by the AS as discussed in {{as-c}}, and, together with other information such as audience (see Section 5.8.1 of {{I-D.ietf-ace-oauth-authz}}), can be used by the AS to determine the token series to which the new requested access token has to be added. Therefore, the identifier MUST identify the pair (AUTH\_CRED\_C, AUTH\_CRED\_RS) associated with a still valid access token previously issued for C and the RS by the AS.

The AS MUST verify that the received value identifies a token series to which a still valid access token issued for C and the RS belongs to. If that is not the case, the Client-to-AS request MUST be declined with the error code "invalid_request" as defined in {{Section 5.8.3 of I-D.ietf-ace-oauth-authz}}.

An example of such a request is shown in {{token-request-update}}.

~~~~~~~~~~~~~~~~~~~~~~~
   Header: POST (Code=0.02)
   Uri-Host: "as.example.com"
   Uri-Path: "token"
   Content-Format: "application/ace+cbor"
   Payload:
   {
     "audience" : "tempSensor4711",
     "scope" : "write",
     "edhoc_info" : {
        "id" : h'01'
     }
   }
~~~~~~~~~~~~~~~~~~~~~~~
{: #token-request-update title="Example of C-to-AS POST /token request for updating access rights to an access token."}

## AS-to-C: Access Token Response # {#as-c}

After verifying the POST request to the token endpoint and that C is authorized to obtain an access token corresponding to its access token request, the AS responds as defined in {{Section 5.8.2 of I-D.ietf-ace-oauth-authz}}. If the request from C was invalid, or not authorized, the AS returns an error response as described in {{Section 5.8.3 of I-D.ietf-ace-oauth-authz}}.

The AS can signal that the use of EDHOC and OSCORE as per this profile is REQUIRED for a specific access token, by including the "ace_profile" parameter with the value "coap_edhoc_oscore" in the access token response. This means that C MUST use EDHOC with the RS and derive an OSCORE Security Context, as specified in {{edhoc-exec}}. After that, C MUST use the established OSCORE Security Context to protect communications with the RS, when accessing protected resources at the RS according to the authorization information indicated in the access token. Usually, it is assumed that constrained devices will be pre-configured with the necessary profile, so that this kind of profile signaling can be omitted.

When issuing any access token of a token series, the AS MUST send the following data in the response to C.

* The identifier of the token series to which the issued access token belongs to. This is specified in the "id" field of EDHOC\_Information.

   All the access tokens belonging to the same token series are associated with the same identifier, which does not change throughout the series lifetime. A token series terminates when the latest issued access token in the series becomes invalid (e.g., when it expires or gets revoked).

   The AS assigns an identifier to a token series when issuing the first access token T of that series. When assigning the identifier, the AS MUST ensure that this was never used in a previous series of access tokens such that: i) they were issued for the same RS for which the access token T is being issued; and ii) they were bound to the same authentication credential AUTH\_CRED\_C of the requesting client to which the access token T is being issued (irrespectively of the exact way AUTH\_CRED\_C is specified in such access tokens).

When issuing the first access token of a token series, the AS MUST send the following data in the response to C.

* The authentication credential of the RS, namely AUTH\_CRED\_RS. This is specified in the "rs\_cnf" parameter defined in {{I-D.ietf-ace-oauth-params}}. AUTH\_CRED\_RS can be transported by value or referred to by means of an appropriate identifier.

   When issuing the first access token ever to a pair (C, RS) using a pair of corresponding authentication credentials (AUTH\_CRED\_C, AUTH\_CRED\_RS), it is typically expected that the response to C specifies AUTH\_CRED\_RS by value.

   When later issuing further access tokens to the same pair (C, RS) using the same AUTH\_CRED\_RS, it is typically expected that the response to C specifies AUTH\_CRED\_RS by reference.

When issuing the first access token of a token series, the AS MAY send the following data in the response to C. If present, these data MUST be included in the corresponding fields of EDHOC\_Information. Some of this information takes advantage of the knowledge that the AS may have about C and RS since their early registration process, with particular reference to what they support as EDHOC peers.

* The EDHOC methods supported by both C and the RS (see {{Section 3.2 of I-D.ietf-lake-edhoc}}). This is specified in the "methods" field of EDHOC\_Information.

* The EDHOC cipher suite (see {{Section 3.6 of I-D.ietf-lake-edhoc}}) to be used by C and the RS as selected cipher suite when running EDHOC. This is specified in the "cipher\_suites" field of EDHOC\_Information. If present, this MUST specify the EDHOC cipher suite which is most preferred by C and at the same time supported by both C and the RS.

* Whether the RS supports or not the function EDHOC-KeyUpdate (see {{Section 4.2.2 of I-D.ietf-lake-edhoc}}). This is specified in the "key\_update" field of EDHOC\_Information.

* Whether the RS supports or not EDHOC message\_4 (see {{Section 5.5 of I-D.ietf-lake-edhoc}}). This is specified in the "message\_4" field of EDHOC\_Information.

* Whether the RS supports or not the combined EDHOC + OSCORE request defined in {{I-D.ietf-core-oscore-edhoc}}. This is specified in the "comb\_req" field of EDHOC\_Information.

* The path component of the URI of the EDHOC resource at the RS, where C is expected to send EDHOC messages as CoAP requests. This is specified in the "uri\_path" field of EDHOC\_Information. If not specified, the URI path "/.well-known/edhoc" defined in {{Section 9.7 of I-D.ietf-lake-edhoc}}) is assumed.

* The size in bytes of the OSCORE Master Secret to derive after the EDHOC execution (see {{Section A.1 of I-D.ietf-lake-edhoc}}) and to use for establishing an OSCORE Security Context. This is specified in the "osc\_ms\_len" field of EDHOC\_Information. If not specified, the default value from {{Section A.1 of I-D.ietf-lake-edhoc}} is assumed.

* The size in bytes of the OSCORE Master Salt to derive after the EDHOC execution (see {{Section A.1 of I-D.ietf-lake-edhoc}}) and to use for establishing an OSCORE Security Context. This is specified in the "osc\_salt\_len" field of EDHOC\_Information. If not specified, the default value from {{Section A.1 of I-D.ietf-lake-edhoc}} is assumed.

* The OSCORE version to use (see {{Section 5.4 of RFC8613}}). This is specified in the "osc\_version" field of EDHOC\_Information. If specified, the AS MUST indicate the highest OSCORE version supported by both C and the RS. If not specified, the default value of 1 (see {{Section 5.4 of RFC8613}}) is assumed.

When issuing any access token of a token series, the AS MUST specify the following data in the claims associated with the access token.

* The identifier of the token series, specified in the "id" field of EDHOC\_Information, and with the same value specified in the response to C from the token endpoint.

* The same authentication credential of C that C specified in its POST request to the token endpoint (see {{c-as}}), namely AUTH\_CRED\_C. If the access token is a CWT, this information MUST be specified in the "cnf" claim.

   In the access token, AUTH\_CRED\_C can be transported by value or referred to by means of an appropriate identifier, regardless of how C specified it in the request to the token endpoint. Thus, the specific field carried in the access token claim and specifying AUTH\_CRED\_C depends on the specific way used by the AS.

   When issuing the first access token ever to a pair (C, RS) using a pair of corresponding authentication credentials (AUTH\_CRED\_C, AUTH\_CRED\_RS), it is typically expected that AUTH\_CRED\_C is specified by value.

   When later issuing further access tokens to the same pair (C, RS) using the same AUTH\_CRED\_C, it is typically expected that AUTH\_CRED\_C is specified by reference.

When issuing the first access token of a token series, the AS MAY specify the following data in the claims associated with the access token. If these data are specified in the response to C from the token endpoint, they MUST be included in the access token and specify the same values that they have in the response from the token endpoint.

* The size in bytes of the OSCORE Master Secret to derive after the EDHOC execution and to use for establishing an OSCORE Security Context. If it is included, it is specified in the "osc\_ms\_len" field of EDHOC\_Information, and it has the same value that the "osc\_ms\_len" field has in the response to C. If it is not included, the default value from {{Section A.1 of I-D.ietf-lake-edhoc}} is assumed.

* The size in bytes of the OSCORE Master Salt to derive after the EDHOC execution (see {{Section A.1 of I-D.ietf-lake-edhoc}}) and to use for establishing an OSCORE Security Context. If it is included, it is specified in the "osc\_salt\_len" field of EDHOC\_Information, and it has the same value that the "osc\_salt\_len" field has in the response to C. If it is not included, the default value from {{Section A.1 of I-D.ietf-lake-edhoc}} is assumed.

* The OSCORE version to use (see {{Section 5.4 of RFC8613}}). This is specified in the "osc\_version" field of the "edhoc\_info" parameter. If it is included, it is specified in the "osc\_version" field of EDHOC\_Information, and it has the same value that the "osc\_version" field has in the response to C. If it is not included, the default value of 1 (see {{Section 5.4 of RFC8613}}) is assumed.

When issuing the first access token of a token series, the AS can take either of the two possible options.

* The AS provides the access token to C, by specifying it in the "access\_token" parameter of the access token response. In such a case, the access token response MAY include the parameter "access\_token", which MUST encode the CBOR simple value "false" (0xf4).

* The AS does not provide the access token to C. Rather, the AS uploads the access token to the authz-info endpoint at the RS, exactly like C would do, and as defined in {{c-rs}} and {{rs-c}}. Then, when replying to C with the access token response as defined above, the response MUST NOT include the parameter "access\_token", and MUST include the parameter "token\_uploaded" encoding the CBOR simple value "true" (0xf5). This is shown by the example in {{example-without-optimization-as-posting}}.

   Note that, in case C and the RS have already completed an EDHOC execution leveraging a previous access token series, using this approach implies that C and the RS have to re-run the EDHOC protocol. That is, they cannot more efficiently make use of the EDHOC-KeyUpdate function, as defined in {{edhoc-key-update}}.

   Also note that this approach is not applicable when issuing access tokens following the first one in the same token series, i.e., when updating access rights.

When CWTs are used as access tokens, EDHOC\_Information MUST be transported in the "edhoc\_info" claim, defined in {{iana-token-cwt-claims}}.

Since the access token does not contain secret information, only its integrity and source authentication are strictly necessary to ensure. Therefore, the AS can protect the access token with either of the means discussed in {{Section 6.1 of I-D.ietf-ace-oauth-authz}}. Nevertheless, when using this profile, it is RECOMMENDED that the access token is a CBOR web token (CWT) protected with COSE_Encrypt/COSE_Encrypt0 as specified in {{RFC8392}}.

{{fig-token-response}} shows an example of an AS response. The "rs_cnf" parameter specifies the authentication credential of the RS, as an X.509 certificate tranported by value in the "x5chain" field. The access token and the authentication credential of the RS have been truncated for readability.

~~~~~~~~~~~~~~~~~~~~~~~
   Header: Created (Code=2.01)
      Content-Type: "application/ace+cbor"
      Payload:
      {
        "access_token" : h'8343a1010aa2044c53 ...
         (remainder of access token (CWT) omitted for brevity)',
        "ace_profile" : "coap_edhoc_oscore",
        "expires_in" : "3600",
        "rs_cnf" : {
          "x5chain" : h'3081ee3081a1a00302 ...'
          (remainder of the access credential omitted for brevity)'
        }
        "edhoc_info" : {
          "id" : h'01',
          "methods" : [0, 1, 2, 3],
          "cipher_suites": 0
        }
      }
~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-token-response title="Example of AS-to-C Access Token response with EDHOC and OSCORE profile."}

{{fig-token}} shows an example CWT Claims Set, including the relevant EDHOC parameters in the "edhoc\_info" claim. The "cnf" claim specifies the authentication credential of C, as an X.509 certificate tranported by value in the "x5chain" field. The authentication credential of C has been truncated for readability.

~~~~~~~~~~~~~~~~~~~~~~~
   {
    "aud" : "tempSensorInLivingRoom",
    "iat" : "1360189224",
    "exp" : "1360289224",
    "scope" :  "temperature_g firmware_p",
    "cnf" : {
      "x5chain" : h'3081ee3081a1a00302 ...'
    }
    "edhoc_info" : {
      "id" : h'01',
      "methods" : [0, 1, 2, 3],
      "cipher_suites": 0
    }
  }
~~~~~~~~~~~~~~~~~~~~~~~
{: #fig-token title="Example of CWT Claims Set with EDHOC parameters."}

If C has requested an update to its access rights using the same OSCORE Security Context, which is valid and authorized, then:

* The response MUST NOT include the "rs\_cnf" parameter.

* The EDHOC\_Information in the response MUST include only the "id" field, specifying the identifier of the token series.

* The EDHOC\_Information in the access token MUST include only the "id" field, specifying the identifier of the token series. In particular, if the access token is a CWT, the "edhoc\_info" claim MUST include only the "id" field.

This identifier of the token series needs to be included in the new access token in order for the RS to identify the old access token to supersede, as well as the OSCORE Security Context already shared between C and the RS and to be associated with the new access token.

## The EDHOC_Information # {#edhoc-parameters-object}

An EDHOC\_Information is an object including information that guides two peers towards executing the EDHOC protocol. In particular, the EDHOC\_Information is defined to be serialized and transported between nodes, as specified by this document, but it can also be used by other specifications if needed.

The EDHOC\_Information can either be encoded as a JSON object or as a CBOR map.  The set of common fields that can appear in an EDHOC\_Information can be found in the IANA "EDHOC Information" registry (see {{iana-edhoc-parameters}}), defined for extensibility, and the initial set of parameters defined in this document is specified below. All parameters are optional.

{{fig-cbor-key-edhoc-params}} provides a summary of the EDHOC\_Information parameters defined in this section.

~~~~~~~~~~~
+---------------+------+--------------+----------+--------------------+
| Name          | CBOR | CBOR value   | Registry | Description        |
|               | Type |              |          |                    |
+---------------+------+--------------+----------+--------------------+
| id            | 0    | bstr         |          | Identifier of      |
|               |      |              |          | EDHOC execution    |
+---------------+------+--------------+----------+--------------------+
| methods       | 1    | int /        | EDHOC    | Set of supported   |
|               |      | array of int | Method   | EDHOC methods      |
|               |      |              | Type     |                    |
|               |      |              | Registry |                    |
+---------------+------+--------------+----------+--------------------+
| cipher_suites | 2    | int /        | EDHOC    | Set of supported   |
|               |      | array of int | Cipher   | EDHOC cipher       |
|               |      |              | Suites   | suites             |
|               |      |              | Registry |                    |
+---------------+------+--------------+----------+--------------------+
| key_update    | 3    | simple value |          | Support for the    |
|               |      | "true" /     |          | EDHOC-KeyUpdate    |
|               |      | simple value |          | function           |
|               |      | "false"      |          |                    |
+---------------+------+--------------+----------+--------------------+
| message_4     | 4    | simple value |          | Support for EDHOC  |
|               |      | "true" /     |          | message_4          |
|               |      | simple value |          |                    |
+---------------+------+--------------+----------+--------------------+
| comb_req      | 5    | simple value |          | Support for the    |
|               |      | "true" /     |          | EDHOC + OSCORE     |
|               |      | simple value |          | combined request   |
+---------------+------+--------------+----------+--------------------+
| uri_path      | 6    | tstr         |          | URI-path of the    |
|               |      |              |          | EDHOC resource     |
+---------------+------+--------------+----------+--------------------+
| osc_ms_len    | 7    | uint         |          | Length in bytes of |
|               |      |              |          | the OSCORE Master  |
|               |      |              |          | Secret to derive   |
+---------------+------+--------------+----------+--------------------+
| osc_salt_len  | 8    | uint         |          | Length in bytes of |
|               |      |              |          | the OSCORE Master  |
|               |      |              |          | Salt to derive     |
+---------------+------+--------------+----------+--------------------+
| osc_version   | 9    | uint         |          | OSCORE version     |
|               |      |              |          | number to use      |
+---------------+------+--------------+----------+--------------------+
~~~~~~~~~~~
{: #fig-cbor-key-edhoc-params title="EDHOC_Information Parameters" artwork-align="center"}

* id: This parameter identifies an EDHOC execution and is encoded as a byte string. In JSON, the "id" value is a Base64 encoded byte string. In CBOR, the "id" type is a byte string, and has label 0.

* methods: This parameter specifies a set of supported EDHOC methods (see {{Section 3.2 of I-D.ietf-lake-edhoc}}). If the set is composed of a single EDHOC method, this is encoded as an integer. Otherwise, the set is encoded as an array of integers, where each array element encodes one EDHOC method. In JSON, the "methods" value is an integer or an array of integers. In CBOR, the "methods" is an integer or an array of integers, and has label 1.

* cipher\_suites: This parameter specifies a set of supported EDHOC cipher suites (see {{Section 3.6 of I-D.ietf-lake-edhoc}}). If the set is composed of a single EDHOC cipher suite, this is encoded as an integer. Otherwise, the set is encoded as an array of integers, where each array element encodes one EDHOC cipher suite. In JSON, the "cipher\_suites" value is an integer or an array of integers. In CBOR, the "cipher\_suites" is an integer or an array of integers, and has label 2.

* key\_update: This parameter indicates whether the EDHOC-KeyUpdate function (see {{Section 4.2.2 of I-D.ietf-lake-edhoc}}) is supported. In JSON, the "key\_update" value is a boolean. In CBOR, "key\_update" is the simple value "true" or "false", and has label 3.

* message\_4: This parameter indicates whether the EDHOC message\_4 (see {{Section 5.5 of I-D.ietf-lake-edhoc}}) is supported. In JSON, the "message\_4" value is a boolean. In CBOR, "message\_4" is the simple value "true" or "false", and has label 4.

* comb\_req: This parameter indicates whether the combined EDHOC + OSCORE request defined in {{I-D.ietf-core-oscore-edhoc}}) is supported. In JSON, the "comb\_req" value is a boolean. In CBOR, "comb\_req" is the simple value "true" or "false", and has label 5.

* uri\_path: This parameter specifies the path component of the URI of the EDHOC resource where EDHOC messages have to be sent as requests. In JSON, the "uri\_path" value is a string. In CBOR, "uri\_path" is text string, and has label 6.

* osc\_ms\_len: This parameter specifies the size in bytes of the OSCORE Master Secret to derive after the EDHOC execution, as per {{Section A.1 of I-D.ietf-lake-edhoc}}. In JSON, the "osc\_ms\_len" value is an integer. In CBOR, the "osc\_ms\_len" type is unsigned integer, and has label 7.

* osc\_salt\_len: This parameter specifies the size in bytes of the OSCORE Master Salt to derive after the EDHOC execution, as per {{Section A.1 of I-D.ietf-lake-edhoc}}. In JSON, the "osc\_salt\_len" value is an integer. In CBOR, the "osc\_salt\_len" type is unsigned integer, and has label 8.

* osc\_version: This parameter specifies the OSCORE Version number that the two EDHOC peers have to use when using OSCORE. For more information about this parameter, see {{Section 5.4 of RFC8613}}. In JSON, the "osc\_version" value is an integer. In CBOR, the "osc\_version" type is unsigned integer, and has label 9.

An example of JSON EDHOC_Information is given in {{fig-edhoc-info-json}}.

~~~~~~~~~~~
   "edhoc_info" : {
       "id" : b64'AQ==',
       "methods" : 1,
       "cipher_suites" : 0
   }
~~~~~~~~~~~
{: #fig-edhoc-info-json title="Example of JSON EDHOC\_Information"}

The CDDL grammar describing the CBOR EDHOC_Information is:

~~~~~~~~~~~
EDHOC_Information = {
   ? 0 => bstr,             ; id
   ? 1 => int / array,      ; methods
   ? 2 => int / array,      ; cipher_suites
   ? 3 => true / false,     ; key_update
   ? 4 => true / false,     ; message_4
   ? 5 => true / false,     ; comb_req
   ? 6 => tstr,             ; uri_path
   ? 7 => uint,             ; osc_ms_len
   ? 8 => uint,             ; osc_salt_len
   ? 9 => uint,             ; osc_version
   * int / tstr => any
}
~~~~~~~~~~~

# Client-RS Communication # {#c-rs-comm}

The following subsections describe the exchanges between C and the RS, which comprise the token uploading to the RS, and the execution of the EDHOC protocol. Note that, as defined in {{as-c}}, the AS may not have provided C with the access token, and have rather uploaded the access token to the authz-info endpoint at the RS on behalf of C.

In order to upload the access token to the RS, C can send a POST request to the authz-info endpoint at the RS. This is detailed in {{c-rs}} and {{rs-c}}, and it is shown by the example in {{example-without-optimization}}.

Alternatively, C can upload the access token while executing the EDHOC protocol, by transporting the access token in the EAD_1 field of the first EDHOC message sent to the RS. This is further discussed in {{edhoc-exec}}, and it is shown by the example in {{example-with-optimization}}.

In either case, following the uploading of the access token, C and the RS run the EDHOC protocol to completion, by exchanging POST requests and related responses to a dedicated EDHOC resource at the RS (see {{edhoc-exec}}). Once completed the EDHOC execution, C and the RS have agreed on a common secret PRK\_OUT (see {{Section 4.1.3 of I-D.ietf-lake-edhoc}}), from which they establish an OSCORE Security Context (see {{edhoc-exec}}). After that, C and the RS use the established OSCORE Security Context to protect their communications when accessing protected resources at the RS, as per the access rights specified in the access token (see {{access-rights-verif}}).

Note that, by means of the respective authentication credentials, C and the RS are mutually authenticated once they have successfully completed the execution of the EDHOC protocol.

As to proof-of-possession, the RS always gains knowledge that C has PRK\_OUT at the end of the successful EDHOC execution. Instead, C gains knowledge that the RS has PRK\_OUT either when receiving and successfully verifying the optional EDHOC message\_4 from the RS, or when successully verifying a response from the RS protected with the generated OSCORE Security Context.

## C-to-RS: POST to authz-info endpoint # {#c-rs}

The access token can be uploaded to the RS by using the authz-info endpoint at the RS. To this end, C MUST use CoAP and the Authorization Information endpoint described in {{Section 5.10.1 of I-D.ietf-ace-oauth-authz}} in order to transport the access token.

That is, C sends a POST request to the authz-info endpoint at the RS, with the request payload conveying the access token without any CBOR wrapping. As per {{Section 5.10.1 of I-D.ietf-ace-oauth-authz}}, the Content-Format of the POST request has to reflect the format of the transported access token. In particular, if the access token is a CWT, the content-format MUST be "application/cwt".

The communication with the authz-info endpoint does not have to be protected, except for the update of access rights case described below.

Note that C may be required to re-POST the access token in order to complete an access request to a protected resource, since the RS may delete a stored access token (and associated OSCORE Security Context) at any time, for example due to all storage space being consumed. This situation is detected by C when it receives an AS Request Creation Hints response from the RS (see {{Section 5.3 of I-D.ietf-ace-oauth-authz}}). After reposting the same access token, C and the RS run the EDHOC protocol again, or instead rely on the more efficient method leveraging the EDHOC-KeyUpdate function (see {{edhoc-key-update}}). In either case, C and the RS will derive a new OSCORE Security Context to be used for protecting their communications from then on.

Also, C may want to update the OSCORE Security Context currently shared with the RS. To this end, C can rely on either of the following options.

* Re-POST the access token, after which C and the RS run the EDHOC protocol again, or instead rely on the more efficient method leveraging the EDHOC-KeyUpdate function (see {{edhoc-key-update}}). After that, C and the RS derive a new OSCORE Security Context.

* Run the OSCORE key update protocol KUDOS defined in {{I-D.ietf-core-oscore-key-update}}, which yields a new OSCORE Security Context.

In either case, C and the RS have established a new OSCORE Security Context that replaces the current one and will be used for protecting their communications from then on. In particular, the RS MUST associate the new OSCORE Security Context with the current (just re-posted) access token. Note that, unless C and the RS re-run the EDHOC protocol, they preserve their same OSCORE identifiers, i.e., their OSCORE Sender/Recipient IDs.

If C has already posted a valid access token, has already established an OSCORE Security Context with the RS, and wants to update its access rights, then C can do so by posting a new access token to the /authz-info endpoint, as described above. The new access token contains the updated access rights for C to access protected resources at the RS, and C has to obtain it from the AS as a new access token in the same token series of the current one (see {{c-as}} and {{as-c}}). When posting the new access token to the authz-info endpoint, C MUST protect the POST request using the current OSCORE Security Context shared with the RS. After successful verification (see {{rs-c}}), the RS will replace the old access token with the new one, while preserving the same OSCORE Security Context. In particular, C and the RS do not re-run the EDHOC protocol and they do not establish a new OSCORE Security Context.

## RS-to-C: 2.01 (Created) # {#rs-c}

Upon receiving an access token from C, the RS MUST follow the procedures defined in {{Section 5.10.1 of I-D.ietf-ace-oauth-authz}}. That is, the RS must verify the validity of the access token. The RS may make an introspection request (see {{Section 5.9.1 of I-D.ietf-ace-oauth-authz}}) to validate the access token.

If the access token is valid, the RS proceeds as follows.

The RS checks whether it is already storing the authentication credential of C, namely AUTH\_CRED\_C, specified as pop-key in the access token by value or reference. In such a case, the RS stores the access token and MUST reply to the POST request with a 2.01 (Created) response.

Otherwise, the RS retrieves AUTH\_CRED\_C, e.g., from the access token if the authentication credential is specified therein by value, or from a further trusted source pointed to by the AUTH\_CRED\_C identifier included in the access token. After that, the RS validates the actual AUTH\_CRED\_C. In case of successful validation, the RS stores AUTH\_CRED\_C as a valid authentication credential. Then, the RS stores the access token and MUST reply to the POST request with a 2.01 (Created) response.

If the RS does not find an already stored AUTH\_CRED\_C, or fails to retrieve it or to validate it, then the RS MUST respond with an error response code equivalent to the CoAP code 4.00 (Bad Request). The RS may provide additional information in the payload of the error response, in order to clarify what went wrong.

Instead, if the access token is valid but it is associated with claims that the RS cannot process (e.g., an unknown scope), or if any of the expected parameters is missing (e.g., any of the mandatory parameters from the AS or the identifier "id"), or if any parameters received in the EDHOC\_Information is unrecognized, then the RS MUST respond with an error response code equivalent to the CoAP code 4.00 (Bad Request). In the latter two cases, the RS may provide additional information in the payload of the error response, in order to clarify what went wrong.

When an access token becomes invalid (e.g., due to its expiration or revocation), the RS MUST delete the access token and the associated OSCORE Security Context, and MUST notify C with an error response with code 4.01 (Unauthorized) for any long running request, as specified in {{Section 5.8.3 of I-D.ietf-ace-oauth-authz}}.

If the RS receives an access token in an OSCORE protected request, it means that C is requesting an update of access rights. In such a case, the RS MUST check that both the following conditions hold.

* The RS checks whether it stores an access token T\_OLD, such that the "id" field of EDHOC\_Identifier matches the "id" field of EDHOC\_Identifier in the new access token T\_NEW.

* The RS checks whether the OSCORE Security Context CTX used to protect the request matches the OSCORE Security Context associated with the stored access token T\_OLD.

If both the conditions above hold, the RS MUST replace the old access token T\_OLD with the new access token T\_NEW, and associate T\_NEW with the OSCORE Security Context CTX. Then, the RS MUST respond with a 2.01 (Created) response protected with the same OSCORE Security Context, with no payload.

Otherwise, the RS MUST respond with a 4.01 (Unauthorized) error response. The RS may provide additional information in the payload of the error response, in order to clarify what went wrong.

As specified in {{Section 5.10.1 of I-D.ietf-ace-oauth-authz}}, when receiving an updated access token with updated authorization information from C (see {{c-rs}}), it is recommended that the RS overwrites the previous access token. That is, only the latest authorization information in the access token received by the RS is valid. This simplifies the process needed by the RS to keep track of authorization information for a given client.

## EDHOC Execution and Setup of OSCORE Security Context # {#edhoc-exec}

In order to mutually authenticate and establish a long-term secret PRK\_OUT with forward secrecy, C and the RS run the EDHOC protocol {{I-D.ietf-lake-edhoc}}. In particular, C acts as EDHOC Initiator thus sending EDHOC message_1, while the RS acts as EDHOC Responder.

As per {{Section A.2 of I-D.ietf-lake-edhoc}}, C sends EDHOC message_1 and EDHOC message_3 to an EDHOC resource at the RS, as CoAP POST requests. Also the RS sends EDHOC message_2 and (optionally) EDHOC message_4 as 2.04 (Changed) CoAP responses. If, in the access token response received from the AS (see {{c-as}}), the "uri_path" field of the EDHOC\_Information was included, then C MUST target the EDHOC resource at the RS with the URI path specified in the "uri_path" field.

In order to seamlessly run EDHOC, clients do not have to first upload to the RS an access token whose scope explicitly indicates authorized access to the EDHOC resource. At the same time, the RS has to ensure that attackers cannot perform requests on the EDHOC resource, other than sending EDHOC messages. Specifically, it SHOULD NOT be possible to perform anything else than POST on an EDHOC resource.

When preparing EDHOC message\_1, C performs the following steps, in additions to those defined in {{Section 5.2.1 of I-D.ietf-lake-edhoc}}.

* If, in the access token response received from the AS (see {{c-as}}), the "methods" field of the EDHOC\_Information was included, then C MUST specify one of those EDHOC methods in the METHOD field of EDHOC message\_1. That is, one of the EDHOC methods specified in the "methods" field of EDHOC\_Information MUST be the EDHOC method used when running EDHOC with the RS.

* If, in the access token response received from the AS (see {{c-as}}), the "cipher\_suites" field of the EDHOC\_Information was included, then C MUST specify the EDHOC cipher suite therein in the SUITES\_I field of EDHOC message\_1. That is, the EDHOC cipher suite specified in the "cipher\_suites" field of EDHOC\_Information MUST be the selected cipher suite when running EDHOC with the RS.

* Rather than first uploading the access token to the authz-info endpoint at the RS as described in {{c-rs}}, C MAY include the access token in the EAD\_1 field of EDHOC message\_1 (see {{Section 3.8 of I-D.ietf-lake-edhoc}}). This is shown by the example in {{example-with-optimization}}.

   In such a case, as per {{Section 3.8 of I-D.ietf-lake-edhoc}}, C adds the pair EAD\_ACCESS\_TOKEN = (ead\_label, ead\_value) to the EAD\_1 field. In particular, ead\_label is the integer value TBD registered in {{iana-edhoc-ead}} of this document, while ead\_value is a CBOR byte string with value the access token. That is, the value of the CBOR byte string is the content of the "access_token" field of the access token response from the AS (see {{as-c}}).

   If EDHOC message\_1 includes the pair EAD\_ACCESS\_TOKEN within the field EAD\_1, then the RS MUST process the access token carried out in ead\_value as specified in {{rs-c}}. If such a process fails, the RS MUST reply to C as specified in {{rs-c}}, it MUST discontinue the EDHOC protocol, and it MUST NOT send an EDHOC error message (see {{Section 6 of I-D.ietf-lake-edhoc}}). The RS MUST have successfully completed the processing of the access token before continuing the EDHOC execution by sending EDHOC message\_2.

   Note that the EAD\_1 field of EDHOC message\_1 cannot carry an access token for the update of access rights, but rather only an access token issued as the first of a token series.

In EDHOC message_2, the authentication credential CRED\_R indicated by the message field ID\_CRED\_R is the authentication credential of the RS, namely AUTH\_CRED\_RS, that C obtained from the AS. The processing of EDHOC message_2 is defined in detail in {{Section 5.3 of I-D.ietf-lake-edhoc}}.

In EDHOC message_3, the authentication credential CRED\_I indicated by the message field ID\_CRED\_I is the authentication credential of C, namely AUTH\_CRED\_C, i.e., the PoP key bound to the access token and specified therein. The processing of EDHOC message_3 is defined in detail in {{Section 5.4 of I-D.ietf-lake-edhoc}}.

Once successfully completed the EDHOC execution, C and RS have both derived the long-term secret PRK\_OUT (see {{Section 4.1.3 of I-D.ietf-lake-edhoc}}), from which they both derive the key PRK\_Exporter (see {{Section 4.2.1 of I-D.ietf-lake-edhoc}}). Then, C and the RS derive an OSCORE Security Context, as defined in {{Section A.1 of I-D.ietf-lake-edhoc}}. In addition, the following applies.

* If, in the access token response received from the AS (see {{c-as}}) and in the access token, the "osc\_ms\_size" field of the EDHOC\_Information was included, then C and the RS MUST use the value specified in the "osc\_ms\_size" field as length in bytes of the OSCORE Master Secret. That is, the value of the "osc\_ms\_size" field MUST be used as value for the oscore\_key\_length parameter of the EDHOC-Exporter function when deriving the OSCORE Master Secret (see {{Section A.1 of I-D.ietf-lake-edhoc}}).

* If, in the access token response received from the AS (see {{c-as}}) and in the access token, the "osc\_salt\_size" field of the EDHOC\_Information was included, then C and the RS MUST use the value specified in the "osc\_salt\_size" field as length in bytes of the OSCORE Master Salt. That is, the value of the "osc\_salt\_size" field MUST be used as value for the oscore\_salt\_length parameter of the EDHOC-Exporter function when deriving the OSCORE Master Salt (see {{Section A.1 of I-D.ietf-lake-edhoc}}).

* If, in the access token response received from the AS (see {{c-as}}) and in the access token, the "osc\_version" field of the EDHOC\_Information was included, then C and the RS MUST derive the OSCORE Security Context, and later use it to protect their communications, consistently with the OSCORE version specified in the "osc\_version" field.

* Given AUTH\_CRED\_C the authentication credential of C used as CRED\_I in the completed EDHOC execution, the RS associates the derived OSCORE Security Context with the stored access token bound to AUTH\_CRED\_I as pop-key (regardless of whether AUTH\_CRED\_I is specified by value or by reference in the access token claims).

If C supports it, C MAY use the EDHOC + OSCORE combined request defined in {{I-D.ietf-core-oscore-edhoc}}, as also shown by the example in {{example-with-optimization}}. In such a case, both EDHOC message\_3 and the first OSCORE-protected application request to a protected resource are sent to the RS as combined together in a single OSCORE-protected CoAP request, thus saving one round trip. This requires C to derive the OSCORE Security Context with RS already after having successfully processed the received EDHOC message\_2. If, in the access token response received from the AS (see {{c-as}}), the "comb\_req" field of the EDHOC\_Information was included and specified the CBOR simple value "false" (0xf4), then C MUST NOT use the EDHOC + OSCORE combined request with the RS.

## Access Rights Verification # {#access-rights-verif}

The RS MUST follow the procedures defined in {{Section 5.10.2 of I-D.ietf-ace-oauth-authz}}. That is, if the RS receives an OSCORE-protected request targeting a protected resource from C, then the RS processes the request according to {{RFC8613}}, when Version 1 of OSCORE is used. Future specifications may define new versions of OSCORE, that the AS can indicate C and the RS to use by means of the "osc\_version" field of EDHOC\_Information (see {{c-as-comm}}).

If OSCORE verification succeeds and the target resource requires authorization, the RS retrieves the authorization information using the access token associated with the OSCORE Security Context. Then, the RS must verify that the authorization information covers the target resource and the action intended by C on it.

# Secure Communication with the AS # {#secure-comm-as}

TBD

# Discarding the Security Context # {#discard-context}

TBD

# Use of EDHOC-KeyUpdate # {#edhoc-key-update}

TBD

# Security Considerations

TBD

# Privacy Considerations

TBD

# IANA Considerations

This document has the following actions for IANA.

Note to RFC Editor: Please replace all occurrences of "{{&SELF}}" with
the RFC number of this specification and delete this paragraph.

## ACE OAuth Profile Registry ## {#iana-ace-oauth-profile}

IANA is asked to add the following entry to the "ACE OAuth Profile"
Registry following the procedure specified in {{I-D.ietf-ace-oauth-authz}}.

* Profile name: coap_edhoc_oscore
* Profile Description: Profile for delegating client authentication and
authorization in a constrained environment by establishing an OSCORE Security Context {{RFC8613}} between resource-constrained nodes, through the execution of the authenticated key establishment protocol EDHOC {{I-D.ietf-lake-edhoc}}.
* Profile ID:  TBD (value between 1 and 255)
* Change Controller: IESG
* Reference:  {{&SELF}}

## OAuth Parameters Registry ## {#iana-oauth-params}

IANA is asked to add the following entries to the "OAuth Parameters" registry.

* Name: "edhoc_info"
* Parameter Usage Location: token request, token response
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Name: "token_uploaded"
* Parameter Usage Location: token response
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

## OAuth Parameters CBOR Mappings Registry ## {#iana-oauth-cbor-mappings}

IANA is asked to add the following entries to the "OAuth Parameters CBOR Mappings" following the procedure specified in {{I-D.ietf-ace-oauth-authz}}.

* Name: "edhoc_info"
* CBOR Key: TBD
* Value Type: map
* Specification Document(s): {{&SELF}}

&nbsp;

* Name: "token_uploaded"
* CBOR Key: TBD
* Value Type: simple value "true" / simple type "false"
* Specification Document(s): {{&SELF}}

## JSON Web Token Claims Registry ## {#iana-token-json-claims}

IANA is asked to add the following entries to the "JSON Web Token Claims" registry following the procedure specified in {{RFC7519}}.

*  Claim Name: "edhoc_info"
*  Claim Description: Information for EDHOC execution
*  Change Controller: IETF
*  Reference: {{&SELF}}

## CBOR Web Token Claims Registry ## {#iana-token-cwt-claims}

IANA is asked to add the following entries to the "CBOR Web Token Claims" registry following the procedure specified in {{RFC8392}}.

* Claim Name: "edhoc_info"
* Claim Description: Information for EDHOC execution
* JWT Claim Name: "edhoc_info"
* Claim Key: TBD
* Claim Value Type(s): map
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

## JWT Confirmation Methods Registry ## {#iana-jwt-confirmation-methods}

IANA is asked to add the following entries to the "JWT Confirmation Methods" registry following the procedure specified in {{RFC7800}}.

* Confirmation Method Value: "x5bag"
* Confirmation Method Description: An unordered bag of X.509 certificates
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "x5chain"
* Confirmation Method Description: An ordered chain of X.509 certificates
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "x5t"
* Confirmation Method Description: Hash of an X.509 certificate
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "x5u"
* Confirmation Method Description: URI pointing to an X.509 certificate
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "c5b"
* Confirmation Method Description: An unordered bag of C509 certificates
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "c5c"
* Confirmation Method Description: An ordered chain of C509 certificates
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "c5t"
* Confirmation Method Description: Hash of an C509 certificate
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "c5u"
* Confirmation Method Description: URI pointing to a COSE_C509 containing an ordere chain of certificates
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "kcwt"
* Confirmation Method Description: A CBOR Web Token (CWT) containing a COSE_Key in a 'cnf' claim
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Value: "kccs"
* Confirmation Method Description: A CWT Claims Set (CCS) containing a COSE_Key in a 'cnf' claim
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

## CWT Confirmation Methods Registry ## {#iana-cwt-confirmation-methods}

IANA is asked to add the following entries to the "CWT Confirmation Methods" registry following the procedure specified in {{RFC8747}}.

* Confirmation Method Name: x5bag
* Confirmation Method Description: An unordered bag of X.509 certificates
* JWT Confirmation Method Name: "x5bag"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_X509
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: x5chain
* Confirmation Method Description: An ordered chain of X.509 certificates
* JWT Confirmation Method Name: "x5chain"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_X509
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: x5t
* Confirmation Method Description: Hash of an X.509 certificate
* JWT Confirmation Method Name: "x5t"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_CertHash
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: x5u
* Confirmation Method Description: URI pointing to an X.509 certificate
* JWT Confirmation Method Name: "x5u"
* Confirmation Key: TBD
* Confirmation Value Type(s): uri
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: c5b
* Confirmation Method Description: An unordered bag of C509 certificates
* JWT Confirmation Method Name: "c5b"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_C509
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: c5c
* Confirmation Method Description: An ordered chain of C509 certificates
* JWT Confirmation Method Name: "c5c"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_C509
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: c5t
* Confirmation Method Description: Hash of an C509 certificate
* JWT Confirmation Method Name: "c5t"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_CertHash
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: c5u
* Confirmation Method Description: URI pointing to a COSE_C509 containing an ordere chain of certificates
* JWT Confirmation Method Name: "c5u"
* Confirmation Key: TBD
* Confirmation Value Type(s): uri
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: kcwt
* Confirmation Method Description: A CBOR Web Token (CWT) containing a COSE_Key in a 'cnf' claim
* JWT Confirmation Method Name: "kcwt"
* Confirmation Key: TBD
* Confirmation Value Type(s): COSE_Messages
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

&nbsp;

* Confirmation Method Name: kccs
* Confirmation Method Description: A CWT Claims Set (CCS) containing a COSE_Key in a 'cnf' claim
* JWT Confirmation Method Name: "kccs"
* Confirmation Key: TBD
* Confirmation Value Type(s): map / #6(map)
* Change Controller: IESG
* Specification Document(s): {{&SELF}}

## EDHOC External Authorization Data Registry # {#iana-edhoc-ead}

IANA is asked to add the following entry to the "EDHOC External Authorization Data" defined in {{Section 9.5 of I-D.ietf-lake-edhoc}}.

* Label: TBD
* Message: EDHOC message\_1
* Description: "ead\_value" specifies an access token
* Reference: {{&SELF}}

## EDHOC Information Registry # {#iana-edhoc-parameters}

It is requested that IANA create a new registry entitled "EDHOC Information" registry. The registry is to be created with registration policy Expert Review {{RFC8126}}. Guidelines for the experts are provided in {{iana-expert-review}}. It should be noted that in addition to the expert review, some portions of the registry require a specification, potentially on Standards Track, be supplied as well.

The columns of the registry are:

* Name: A descriptive name that enables easier reference to this item. Because a core goal of this document is for the resulting representations to be compact, it is RECOMMENDED that the name be short.

   This name is case sensitive. Names may not match other registered names in a case-insensitive manner unless the Designated Experts determine that there is a compelling reason to allow an exception. The name is not used in the CBOR encoding.

* CBOR Value: The value to be used as CBOR abbreviation of the item.

   The value MUST be unique. The value can be a positive integer, a negative integer or a string. Integer values between -256 and 255 and strings of length 1 are to be registered by Standards Track documents (Standards Action). Integer values from -65536 to -257 and from 256 to 65535 and strings of maximum length 2 are to be registered by public specifications (Specification Required). Integer values greater than 65535 and strings of length greater than 2 are subject to the Expert Review policy. Integer values less than -65536 are marked as private use.

* CBOR Type: The CBOR type of the item, or a pointer to the registry that defines its type, when that depends on another item.

* Registry: The registry that values of the item may come from, if one exists.

* Description: A brief description of this item.

* Specification: A pointer to the public specification for the item, if one exists.

This registry will be initially populated by the values in {{fig-cbor-key-edhoc-params}}. The "Specification" column for all of these entries will be this document and {{I-D.ietf-lake-edhoc}}.

## Expert Review Instructions # {#iana-expert-review}

The IANA registry established in this document is defined to use the registration policy Expert Review. This section gives some general guidelines for what the experts should be looking for, but they are being designated as experts for a reason so they should be given substantial latitude.

Expert reviewers should take into consideration the following points:

* Point squatting should be discouraged. Reviewers are encouraged to get sufficient information for registration requests to ensure that the usage is not going to duplicate one that is already registered and that the point is likely to be used in deployments. The zones tagged as private use are intended for testing purposes and closed environments; code points in other ranges should not be assigned for testing.

* Specifications are required for the Standards Action range of point assignment. Specifications should exist for Specification Required ranges, but early assignment before a specification is available is considered to be permissible. Specifications are needed for the first-come, first-serve range if they are expected to be used outside of closed environments in an interoperable way. When specifications are not provided, the description provided needs to have sufficient information to identify what the point is being used for.

* Experts should take into account the expected usage of fields when approving point assignment. The fact that there is a range for Standards Track documents does not mean that a Standards Track document cannot have points assigned outside of that range. The length of the encoded value should be weighed against how many code points of that length are left, the size of device it will be used on, and the number of code points left that encode to that size.

--- back

# Examples # {#examples}

This appendix provides examples where this profile of ACE is used. In particular:

* {{example-without-optimization}} does not make use of use of any optimization.

* {{example-with-optimization}} makes use of the optimizations defined in this specification, hence reducing the roundtrips of the interactions between the Client and RS.

* {{example-without-optimization-as-posting}} does not make use of any optimization, but consider an alternative workflow where the AS uploads the access token to the RS.

All these examples build on the following assumptions, as relying on expected early procedures performed at the AS. These include the registration of RSs by the respective Resource Owners as well as the registrations of Clients authorized to request access token for those RSs.

* The AS knows the authentication credential AUTH_CRED_C of the Client C.

* The Client knows the authentication credential AUTH_CRED_AS of the AS.

* The AS knows the authentication credential AUTH_CRED_RS of RS.

* The RS knows the authentication credential AUTH_CRED_AS of the AS.

   This is relevant in case the AS and RS actually require a secure association (e.g., for the RS to perform token introspection at the AS, or for the AS to upload an access token to the RS on behalf of the Client).

As a result of the assumptions above, it is possible to limit the transport of AUTH_CRED_C and AUTH_CRED_RS by value only to the following two cases, and only when the Client requests an access token for the RS in question for the first time when considering the pair (AUTH_CRED_C, AUTH_CRED_RS).

* In the Token Response from the AS to the Client, where AUTH_CRED_RS is specified by the 'rs_cnf' parameter.

* In the access token, where AUTH_CRED_C is specified by the 'cnf' claim.

   Note that, even under the circumstances mentioned above, AUTH_CRED_C might rather be indicated by reference. This is possible if the RS can effectively use such a reference from the access token to retrieve AUTH_CRED_C (e.g., from a trusted repository of authentication credentials reachable through a non-constrained link), and if the AS is in turn aware of that.

In any other case, it is otherwise possible to indicate both AUTH_CRED_C and AUTH_CRED_RS by reference, when performing the ACE access control workflow as well as later on when the Client and RS run EDHOC.

## Workflow without Optimizations # {#example-without-optimization}

The example below considers the simplest (though least efficient) interaction between the Client and RS. That is: first the Client uploads the access token to the RS; then the Client and RS run EDHOC; and, finally, the Client accesses the protected resource at the RS.

~~~~~~~~~~~~~~~~~~~~~~~
    C                                 AS                             RS
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
M01 |--------------------------------->|                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
M02 |<---------------------------------|                              |
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_AS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
M03 |--------------------------------->|                              |
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token request to /token         |                              |
    |  (OSCORE-protected message)      |                              |
M04 |--------------------------------->|                              |
    |  'req_cnf' identifies            |                              |
    |     AUTH_CRED_C by reference     |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M05 |<---------------------------------|                              |
    |  'rs_cnf' specifies              |                              |
    |     AUTH_CRED_RS by value        |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'01',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |
    |  In the access token:            |                              |
    |     * the 'cnf' claim specifies  |                              |
    |       AUTH_CRED_C by value       |                              |
    |     * the 'edhoc_info' claim     |                              |
    |       specifies the same as      |                              |
    |       'edhoc_info' above         |                              |
    |                                  |                              |

 // Possibly after chain verification, the Client adds AUTH_CRED_RS
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |  Token upload to /authz-info     |                              |
    |  (unprotected message)           |                              |
M06 |---------------------------------------------------------------->|
    |                                  |                              |

 // Possibly after chain verification, the RS adds AUTH_CRED_C
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |   2.01 (Created)                 |                              |
    |   (unprotected message)          |                              |
M07 |<----------------------------------------------------------------|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M08 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
M09 |<----------------------------------------------------------------|
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_RS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M10 |---------------------------------------------------------------->|
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Access to protected resource    |                              |
    |  (OSCORE-protected message)      |                              |
    |  (access control is enforced)    |                              |
M11 |---------------------------------------------------------------->|
    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M12 |<----------------------------------------------------------------|
    |                                  |                              |

 // Later on, the access token expires ...
 //  - The Client and RS delete their OSCORE Security Context and
 //    terminate the EDHOC session used to derive it (unless the same
 //    session is also used for other reasons).
 //  - The RS retains AUTH_CRED_C as still valid,
 //    and the AS knows about it.
 //  - The Client retains AUTH_CRED_RS as still valid,
 //    and the AS knows about it.

    |                                  |                              |
    |                                  |                              |

 // Time passes ...

    |                                  |                              |
    |                                  |                              |

 // The Client asks for a new access token; now all the
 // authentication credentials can be indicated by reference

 // The price to pay is on the AS, about remembering that at least
 // one access token has been issued for the pair (Client, RS)
 // and considering the pair (AUTH_CRED_C, AUTH_CRED_RS)

    |                                  |                              |
    |  Token request to /token         |                              |
    |  (OSCORE-protected message)      |                              |
M13 |--------------------------------->|                              |
    |  'req_cnf' identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M14 |<---------------------------------|                              |
    |  'rs_cnf' identifies             |                              |
    |     AUTH_CRED_RS by reference    |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'05',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |
    |  In the access token:            |                              |
    |     * the 'cnf' claim specifies  |                              |
    |       AUTH_CRED_C by reference   |                              |
    |     * the 'edhoc_info' claim     |                              |
    |       specifies the same as      |                              |
    |       'edhoc_info' above         |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token upload to /authz-info     |                              |
    |  (unprotected message)           |                              |
M15 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  2.01 (Created)                  |                              |
    |  (unprotected message)           |                              |
M16 |<----------------------------------------------------------------|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M17 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
    |  (no access control is enforced) |                              |
M18 |<----------------------------------------------------------------|
    |  ID_CRED_R specifies             |                              |
    |     CRED_R = AUTH_CRED_RS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M19 |---------------------------------------------------------------->|
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Access to protected resource /r |                              |
    |  (OSCORE-protected message)      |                              |
    |  (access control is enforced)    |                              |
M20 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M21 |<----------------------------------------------------------------|
    |                                  |                              |
~~~~~~~~~~~~~~~~~~~~~~~

## Workflow with Optimizations # {#example-with-optimization}

The example below builds on the example in {{example-without-optimization}}, while additionally relying on the two following optimizations.

* The access token is not separately uploaded to the /authz-info endpoint at the RS, but rather included in the EAD_1 field of EDHOC message_1 sent by the Client to the RS.

* The Client uses the EDHOC+OSCORE request defined in {{I-D.ietf-core-oscore-edhoc}} is used, when running EDHOC both with the AS and with the RS.

These two optimizations used together result in the most efficient interaction between the Client and RS, as consisting of only two roundtrips to upload the access token, run EDHOC and access the protected resource at the RS.

Also, a further optimization is used upon uploading a second access token to the RS, following the expiration of the first one. That is, after posting the second access token, the Client and RS do not run EDHOC again, but rather EDHOC-KeyUpdate() and EDHOC-Exporter() building on the same EDHOC session established before.

~~~~~~~~~~~~~~~~~~~~~~~
    C                                 AS                             RS
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
M01 |--------------------------------->|                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
M02 |<---------------------------------|                              |
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_AS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC+OSCORE request to /token  |                              |
M03 |--------------------------------->|                              |
    |  * EDHOC message_3               |                              |
    |      ID_CRED_I identifies        |                              |
    |         CRED_I = AUTH_CRED_C     |                              |
    |         by reference             |                              |
    |  --- --- ---                     |                              |
    |  * OSCORE-protected part         |                              |
    |      Token request               |                              |
    |         'req_cnf' identifies     |                              |
    |         AUTH_CRED_C by reference |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M04 |<---------------------------------|                              |
    |  'rs_cnf' specifies              |                              |
    |     AUTH_CRED_RS by value        |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'01',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |
    |  In the access token:            |                              |
    |     * the 'cnf' claim specifies  |                              |
    |       AUTH_CRED_C by value       |                              |
    |     * the 'edhoc_info' claim     |                              |
    |       specifies the same as      |                              |
    |       'edhoc_info' above         |                              |
    |                                  |                              |

 // Possibly after chain verification, the Client adds AUTH_CRED_RS
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M05 |---------------------------------------------------------------->|
    |  Access token specified in EAD_1 |                              |
    |                                  |                              |

 // Possibly after chain verification, the RS adds AUTH_CRED_C
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |  EDHOC message_2                 |                              |
M06 |<----------------------------------------------------------------|
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_RS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC+OSCORE request to /r      |                              |
M07 |---------------------------------------------------------------->|
    |  * EDHOC message_3               |                              |
    |      ID_CRED_I identifies        |                              |
    |         CRED_I = AUTH_CRED_C     |                              |
    |         by reference             |                              |
    |  --- --- ---                     |                              |
    |  * OSCORE-protected part         |                              |
    |      Application request to /r   |                              |
    |                                  |                              |

 // After the EDHOC processing is completed, access control
 // is enforced on the rebuilt OSCORE-protected request,
 // like if it had been sent stand-alone

    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M08 |<----------------------------------------------------------------|
    |                                  |                              |

 // Later on, the access token expires ...
 //  - The Client and RS delete their OSCORE Security Context and
 //    terminate the EDHOC session used to derive it (unless the same
 //    session is also used for other reasons).
 //  - The RS retains AUTH_CRED_C as still valid,
 //    and the AS knows about it.
 //  - The Client retains AUTH_CRED_RS as still valid,
 //    and the AS knows about it.

    |                                  |                              |
    |                                  |                              |

 // Time passes ...

    |                                  |                              |
    |                                  |                              |

 // The Client asks for a new access token; now all the
 // authentication credentials can be indicated by reference

 // The price to pay is on the AS, about remembering that at least
 // one access token has been issued for the pair (Client, RS)
 // and considering the pair (AUTH_CRED_C, AUTH_CRED_RS)

    |                                  |                              |
    |  Token request to /token         |                              |
    |  (OSCORE-protected message)      |                              |
M09 |--------------------------------->|                              |
    |  'req_cnf' identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M10 |<---------------------------------|                              |
    |  'rs_cnf' identifies             |                              |
    |     AUTH_CRED_RS by reference    |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'05',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |
    |  In the access token:            |                              |
    |     * the 'cnf' claim specifies  |                              |
    |       AUTH_CRED_C by reference   |                              |
    |     * the 'edhoc_info' claim     |                              |
    |       specifies the same as      |                              |
    |       'edhoc_info' above         |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token upload to /authz-info     |                              |
    |  (unprotected message)           |                              |
M11 |---------------------------------------------------------------->|
    |   Payload {                      |                              |
    |     access_token: access token   |                              |
    |     nonce_1: N1  // nonce        |                              |
    |   }                              |                              |
    |                                  |                              |
    |                                  |                              |
    |  2.01 (Created)                  |                              |
    |  (unprotected message)           |                              |
M12 |<----------------------------------------------------------------|
    |   Payload {                      |                              |
    |     nonce_2: N2  // nonce        |                              |
    |   }                              |                              |
    |                                  |                              |

 // The Client and RS first run EDHOC-KeyUpdate(N1 | N2), and
 // then EDHOC-Exporter() to derive a new OSCORE Master Secret and
 // OSCORE Master Salt, from which a new OSCORE Security Context is
 // derived. The Sender/Recipiend IDs are the same C_I and C_R from
 // the previous EDHOC execution

    |                                  |                              |
    |  Access to protected resource /r |                              |
    |  (OSCORE-protected message)      |                              |
    |  (access control is enforced)    |                              |
M13 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M14 |<----------------------------------------------------------------|
    |                                  |                              |
~~~~~~~~~~~~~~~~~~~~~~~

## Workflow without Optimizations (AS token posting) # {#example-without-optimization-as-posting}

The example below builds on the example in {{example-without-optimization}}, but assumes that the AS is uploading the access token to the RS on behalf of C.

In order to save roundtrips between the Client and RS, further, more efficient interactions can be seamlessly considered, e.g., as per the example in {{example-with-optimization}}.

~~~~~~~~~~~~~~~~~~~~~~~
    C                                 AS                             RS
    |                                  |                              |
    |                                  | Establish secure association |
    |                                  | (e.g., OSCORE using EDHOC)   |
    |                                  |<---------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
M01 |--------------------------------->|                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
M02 |<---------------------------------|                              |
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_AS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
M03 |--------------------------------->|                              |
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Token request to /token         |                              |
    |  (OSCORE-protected message)      |                              |
M04 |--------------------------------->|                              |
    |  'req_cnf' identifies            |                              |
    |     AUTH_CRED_C by reference     |                              |
    |                                  |                              |
    |                                  |                              |
    |                                  |  Token upload to /authz-info |
    |                                  |  (OSCORE-protected message)  |
M05 |                                  |----------------------------->|
    |                                  |  In the access token:        |
    |                                  |     * the 'cnf' claim        |
    |                                  |       specifies AUTH_CRED_C  |
    |                                  |       by value               |
    |                                  |     * the 'edhoc_info'       |
    |                                  |       claim specifies        |
    |                                  |         {                    |
    |                                  |           id     : h'01',    |
    |                                  |           suite  : 2,        |
    |                                  |           methods: 3         |
    |                                  |         }                    |
    |                                  |                              |

 // Possibly after chain verification, the RS adds AUTH_CRED_C
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |                                  |  2.01 (Created)              |
    |                                  |  (OSCORE-protected message)  |
M06 |                                  |<-----------------------------|
    |                                  |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M07 |<---------------------------------|                              |
    |  'rs_cnf' specifies              |                              |
    |     AUTH_CRED_RS by value        |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'token_uploaded' = true         |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'01',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |


 // Possibly after chain verification, the Client adds AUTH_CRED_RS
 // to the set of its trusted peer authentication credentials,
 // relying on the AS as trusted provider

    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M08 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
M09 |<----------------------------------------------------------------|
    |  ID_CRED_R identifies            |                              |
    |     CRED_R = AUTH_CRED_RS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M10 |---------------------------------------------------------------->|
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Access to protected resource    |                              |
    |  (OSCORE-protected message)      |                              |
    |  (access control is enforced)    |                              |
M11 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M12 |<----------------------------------------------------------------|
    |                                  |                              |

 // Later on, the access token expires ...
 //  - The Client and RS delete their OSCORE Security Context and
 //    terminate the EDHOC session used to derive it (unless the same
 //    session is also used for other reasons).
 //  - The RS retains AUTH_CRED_C as still valid,
 //    and the AS knows about it.
 //  - The Client retains AUTH_CRED_RS as still valid,
 //    and the AS knows about it.

    |                                  |                              |
    |                                  |                              |

 // Time passes ...

    |                                  |                              |
    |                                  |                              |

 // The Client asks for a new access token; now all the
 // authentication credentials can be indicated by reference

 // The price to pay is on the AS, about remembering that at least
 // one access token has been issued for the pair (Client, RS)
 // and considering the pair (AUTH_CRED_C, AUTH_CRED_RS)

    |                                  |                              |
    |  Token request to /token         |                              |
    |  (OSCORE-protected message)      |                              |
M13 |--------------------------------->|                              |
    |  'req_cnf' identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |                                  |  Token upload to /authz-info |
    |                                  |  (OSCORE-protected message)  |
M14 |                                  |----------------------------->|
    |                                  |  In the access token:        |
    |                                  |     * the 'cnf' claim        |
    |                                  |       specifies AUTH_CRED_C  |
    |                                  |       by reference           |
    |                                  |     * the 'edhoc_info'       |
    |                                  |       claim specifies        |
    |                                  |         {                    |
    |                                  |           id     : h'05',    |
    |                                  |           suite  : 2,        |
    |                                  |           methods: 3         |
    |                                  |         }                    |
    |                                  |                              |
    |                                  |                              |
    |                                  |  2.01 (Created)              |
    |                                  |  (OSCORE-protected message)  |
M15 |                                  |<-----------------------------|
    |                                  |                              |
    |                                  |                              |
    |  Token response                  |                              |
    |  (OSCORE-protected message)      |                              |
M16 |<---------------------------------|                              |
    |  'rs_cnf' specifies              |                              |
    |     AUTH_CRED_RS by reference    |                              |
    |                                  |                              |
    |  'ace_profile' =                 |                              |
    |             coap_edhoc_oscore    |                              |
    |                                  |                              |
    |  'token_uploaded' = true         |                              |
    |                                  |                              |
    |  'edhoc_info' specifies:         |                              |
    |     {                            |                              |
    |       id     : h'05',            |                              |
    |       suite  : 2,                |                              |
    |       methods: 3                 |                              |
    |     }                            |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_1 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M17 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_2                 |                              |
    |  (no access control is enforced) |                              |
M18 |<----------------------------------------------------------------|
    |  ID_CRED_R specifies             |                              |
    |     CRED_R = AUTH_CRED_RS        |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  EDHOC message_3 to /edhoc       |                              |
    |  (no access control is enforced) |                              |
M19 |---------------------------------------------------------------->|
    |  ID_CRED_I identifies            |                              |
    |     CRED_I = AUTH_CRED_C         |                              |
    |     by reference                 |                              |
    |                                  |                              |
    |                                  |                              |
    |  Access to protected resource /r |                              |
    |  (OSCORE-protected message)      |                              |
    |  (access control is enforced)    |                              |
M20 |---------------------------------------------------------------->|
    |                                  |                              |
    |                                  |                              |
    |  Response                        |                              |
    |  (OSCORE-protected message)      |                              |
M21 |<----------------------------------------------------------------|
    |                                  |                              |
~~~~~~~~~~~~~~~~~~~~~~~

# Acknowledgments # {#acknowldegment}
{: numbered="no"}

Work on this document has in part been supported by the H2020 project SIFIS-Home (grant agreement 952652).