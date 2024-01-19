
This document provides a review of the various OAuth and OpenID specifications and how each supports the concept of authorization. 

The purpose of this document is to provide context for the development of new standards within the AuthZen working group.

# Prior Art: OIDC - OAuth 2.x - draft 

## Introduction

OAuth describes a set of standards that define the protocols for the authorization of a client to access APIs. OIDC extends the authorization capability of this protocol, by providing a user context to the authorization process, enabling access control to resources based on the identity of the user. Until recently OAuth has provided few mechanisms to describe the access rights of the client; however, recent developments in open ecosystems, such as Open Banking, and evolution of standards to support strong identity using verifiable credentials have driven the development of new standards which provide a more expressive authorization process. This document provides a review of the various OAuth and OpenID standards, with a particular focus on the available mechanisms used by the client to enquire and request authorization for an end-user. 

Conceptually and in practice there are four key activities in OAuth that perform an authorization function:
- **Client Authorization Request ( /authorize | /par | /bc-authorize )**: Invoked by the user agent/client on the Authorization Server (AS) for the purposes of obtaining an access token. This performs an authorization of attributes provided in the request: if successful returns an authorization code (code flow) or access token (other); if unsuccessful returns an error.
- **Token Request (/token)**: The issuance of a token to the client validates the state of the grant, and any specific authorization parameters provided with the authorization request. 
- **Resource Server Authorization (incl /userInfo)**: This is the verification the token presented to the resource server is valid and that the token has the required authorization to access the resource.
- **Grant Management Enquiry (/grants)**: While this api does not act as an authorization control, it does return the authorization state of the client grant.

**User Agent/Client Authorization**: Today there are a number of mechanisms in OAuth and OIDC that enable the client to communicate its Authorization requirement, which may be performed in a number of different ways beyond standard OAuth flows providing support for many different use cases (e.g. [CIBA](https://openid.net/specs/openid-client-initiated-backchannel-authentication-core-1_0.html), [Device Authorization Grant](https://www.rfc-editor.org/rfc/rfc8628) etc.). Despite this diversity the mechanism to communicate the authorization requirement is standardized through the use of Scopes (including finer grained claims defined within OIDC) [OIDC Section 5](https://openid.net/specs/openid-connect-core-1_0.html#Claims) the <code>authorization_details</code> request parameter defined within the RAR standard [RFC 9396 Section 2](https://datatracker.ietf.org/doc/html/rfc9396#name-request-parameter-authoriza), and on the token request through the <code>authorization_details</code> [RFC 9396 Section 6](https://datatracker.ietf.org/doc/html/rfc9396#name-token-request) parameter. Each of these authorization request attributes enable a user agent/client to describe the authorization it requires, which may automatically authorize the request or trigger end-user Consent based interaction from one or many users that hold ownership over the resources the authorization is being requested for.

**Consent**: Consent is increasingly used within digital activities to capture end user authorization to act on user data. Consent is captured  in many ways and for many different reasons, and has become a key component of privacy regulations such as GDPR. Schemes such as Open Banking have also adopted the concept of consent for use of apis by third parties, which has driven the development of the FAPI standard and Rich Authorization Requests to accommodate a more secure expressive way of communicating the authorization requirement of the API client. Open Banking has also opened a degree of ambiguity in the concept of consent due to the authorization requirement over joint accounts, with the capture "consent" of multiple parties for jointly owned Bank Accounts. From an OAuth/OIDC point of view, the Consent interaction is outside of the standard and transparent to the client - this presents challenges to the client in understanding the state authorization it has been granted if multiple parties are required to perform separate consent activities. While recent standards such as the [Grant Management API](https://openid.net/specs/oauth-v2-grant-management-ID1.html) attempt to provide the client with insight into the authorization granted, for reasons of privacy, it is not possible to provide the authorization requirement of jointly held resources. 

**User Agent/Client Visibility**: Conceptually OAuth and OIDC standards related to authorization are specific to the  requirement of the user agent/client request for access to user data. This stems from the origins of OAuth which did not include the concept of identity of the end-user and thus the inferred access that user may have over a resource OAuth left user authorization up to implementer to address outside of the API client connection. The introduction of end-user identity through OIDC addressed authorization of end-user attributes (Claims) but did not extend to the authorization of other resources the end-user may own, or have access to. The development of the [Grant Management API](https://openid.net/specs/oauth-v2-grant-management-ID1.html) standard provides visibility of the authorization state of a client request, and although its intent was not to provide a conduit through to the end-user resource ownership, revealing resource ownership through this API does provide opportunities for the client to enquire on authorization potential held by the end-user.

**Authorization Signalling**:communicating to the client authorization state is poorly defined in OAuth/OIDC standards. Recent standards relating to (Authentication Stepup)[] (note this is an authentication stepup not AuthN) have been defined, and the [Grant Management API](https://openid.net/specs/oauth-v2-grant-management-ID1.html) now potentially provides insight into resource ownership of the client, but otherwise generic HTTP codes are used to signal authorization outcomes to a client. 

**Resource Authorization**: [RFC 8707: Resource Indicators for OAuth 2.0](https://www.rfc-editor.org/rfc/rfc8707.html) provides a mechanism for the client to signal to the authorization server the protected resources it requires access to. While the standard does not allow the inclusion of a [fragment component](https://www.rfc-editor.org/rfc/rfc8707#name-resource-parameter) it does provide opportunity for a query component (although discouraged). The resource parameter may also be used on the <code>./token</code> endpoint. The [RFC 9396: OAuth 2.0 Rich Authorization Request](https://datatracker.ietf.org/doc/html/rfc9396#name-common-data-fields) standard also provides opportunity communicate resource endpoint authorization requirements through the <code>locations</code> attribute. 

**Authorization Metadata**: There are a couple of OAuth standards which describe mechanisms for the communication of authorization metadata to the client. These include:
- [RFC 8414: OAuth 2.0 Authorization Server Metadata](https://www.rfc-editor.org/rfc/rfc8414.html): Providing the client with information regarding scopes - claims - <code>authorization_details</code> type supported by the Authorization Server.
- [Oauth 2.0 Protected Resource Metadata](https://www.ietf.org/archive/id/draft-ietf-oauth-resource-metadata-01.html) (DRAFT:Published 20 Oct 2023): Offers a mechanism for the Resource Server to advertise its capabilities to the client through a a <code>.well_known</code> endpoint. This can signal to the client the authorization servers, scopes that the resource server supports, documentation and other values required by the client to invoke.

**OpenID for Verifiable Credentials**: OID4VC builds on existing OpenID Connect standards to enable a client to perform activities related to the issuance, verification and presentation of Verifiable Credentials. These standards leverage authorization and token flows to request authorize and return credential data, of which there are authorization activities performed. These flows have not been covered in this document but will be addressed in a subsequent version. 

**Other Standards**: This document covers the main standards that describe the authorization process within OAuth and OIDC, and provide sufficient context for framing new Authorization Standards of the AuthZen working group; however, there are a number of standards that are worth mentioning and review by the reader - including [OAuth-PoA Grant Type](https://www.ietf.org/archive/id/draft-vattaparambil-oauth-poa-grant-type-01.html), [RFC 8693 Token Exchange](https://www.rfc-editor.org/rfc/rfc8693#name-token-exchange-request-and-)

## Requests

This section describes the various methods for requesting authorization defined by OAuth and OIDC standards. 

NOTE: AS policy determines the behavior of an authorization request relating to unknown or scopes - authorization types that a client has not been registered for.


### Standard OAuth RFC 6749

Request for the return of an Access Token (and optionally a Refresh Token) from the Authorization Server, which will grant a client access to a resource on behalf of a End-User. 

The scope parameter is used to describe the access that the client is requesting.

```http
 
GET /authorize?
    response_type=code&
    client_id=s6BhdRkqt3&state=xyz&
    redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb&
    **scope=openid%20profile&**
    
    HTTP/1.1

```

There are a number of extension standards that provide alternative methods for requesting authorization: 
- **Pushed Authorization Requests (PAR)**: Requests provided via a Pushed Authorization Request (PAR) endpoint.
- **JWT-Secured Authorization Request (JAR)**: Requests encoded in a JWT and provided either within the request or via uri. 
- **Rich Authorization Requests (RAR)**: Authorization Details provided as json via the <code>authorization_details</code> parameter in the authorization request. (Note: <code>authorization_details</code> may also be used on token refresh, however, this only narrows the scope of the returned token, and does not change the authorization of the client grant.) 

Extensions to authorization request:
- <code>resource</code> parameter: [Section 2 RFC 8707 Resource Indicators for OAuth 2.0](https://www.rfc-editor.org/rfc/rfc8707)
- <code>authorization_details</code> parameter: [Section 2 RFC 9396 RAR](https://datatracker.ietf.org/doc/html/rfc9396#name-request-parameter-authoriza)
- [Pushed Authorization Requests](https://www.rfc-editor.org/rfc/rfc9126): for back channel lodgment of authorization request.

###  OIDC Claims Request

https://openid.net/specs/openid-connect-core-1_0.html#ClaimsParameter

```http
GET /authorize?
    response_type=code&
    client_id=s6BhdRkqt3&
    scope=openid%20profile%20email&
    redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb&
    state=xyz&
    nonce=n-0S6_WzA2Mj&
    claims=
    {
        "userinfo":
        {
            "given_name": {"essential": true},
            "nickname": null,
            "email": {"essential": true},
            "email_verified": {"essential": true},
            "picture": null
        },
        "id_token":
        {
            "auth_time": {"essential": true},
            "acr": {"values": ["urn:mace:incommon:iap:silver"]}
        }
    }
HTTP/1.1
Host: server.example.com
```

### Rich Authorization Requests

As defined in [Section 3](https://datatracker.ietf.org/doc/html/rfc9396#name-authorization-request) for request on <code>/authorize</code> endpoint, and for a token request in [Section 7](https://datatracker.ietf.org/doc/html/rfc9396#name-token-response).

``` http
GET /authorize?response_type=code
   &client_id=s6BhdRkqt3
   &state=af0ifjsldkj
   &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
   &code_challenge_method=S256
   &code_challenge=K2-ltc83acc4h0c9w6ESC_rEMTJ3bwc-uCHaoeK1t8U
   &authorization_details=%5B%7B%22type%22%3A%22account%5Finfo
   rmation%22%2C%22actions%22%3A%5B%22list%5Faccounts%22%2C%22
   read%5Fbalances%22%2C%22read%5Ftransactions%22%5D%2C%22loca
   tions%22%3A%5B%22https%3A%2F%2Fexample%2Ecom%2Faccounts%22%
   5D%7D%2C%7B%22type%22%3A%22payment%5Finitiation%22%2C%22act
   ions%22%3A%5B%22initiate%22%2C%22status%22%2C%22cancel%22%5
   D%2C%22locations%22%3A%5B%22https%3A%2F%2Fexample%2Ecom%2Fp
   ayments%22%5D%2C%22instructedAmount%22%3A%7B%22currency%22%
   3A%22EUR%22%2C%22amount%22%3A%22123%2E50%22%7D%2C%22credito
   rName%22%3A%22Merchant%20A%22%2C%22creditorAccount%22%3A%7B
   %22iban%22%3A%22DE02100100109307118603%22%7D%2C%22remittanc
   eInformationUnstructured%22%3A%22Ref%20Number%20Merchant%22
   %7D%5D HTTP/1.1
Host: server.example.com

```

``` json 
[
   {
      "type": "account_information",
      "actions": [
         "list_accounts",
         "read_balances",
         "read_transactions"
      ],
      "locations": [
         "https://example.com/accounts"
      ]
   },
   {
      "type": "payment_initiation",
      "actions": [
         "initiate",
         "status",
         "cancel"
      ],
      "locations": [
         "https://example.com/payments"
      ],
      "instructedAmount": {
         "currency": "EUR",
         "amount": "123.50"
      },
      "creditorName": "Merchant A",
      "creditorAccount": {
         "iban": "DE02100100109307118603"
      },
      "remittanceInformationUnstructured": "Ref Number Merchant"
   }
]
```
URL Decoded <code>authorization_details</code>

**NOTES**: 
- The only mandatory parameter is the type parameter. The type parameter is a string that identifies the type of the authorization details. All other parameters are optional and up to the implementer to define. 
- An Authorization Request may also be placed on the token endpoint; however, this only narrows the scope of the returned token and does not change the authorization of the client grant. When this token is presented to the resource server the narrowed token scope is enforced.  

#### Metadata
https://datatracker.ietf.org/doc/html/rfc9396#name-metadata

the RAR spec provides for the advertisement of the RAR capability through the metadata endpoint. 

```http
   ...
   "authorization_details_types_supported":[
      "payment_initiation",
      "account_information"
   ]
}
```


### Grant Management API

The Grant Management API allows the client to manage the grants that it has been issued. This includes the ability to revoke or extend a clients grant. 
As part of the Authorization request [Section 5.2](https://openid.net/specs/fapi-grant-management-01.html#name-authorization-request)

To enquire on the status of a grant [Section 6.4](https://openid.net/specs/fapi-grant-management-01.html#name-query-status-of-a-grant)

To call the GM API the client requires authorization of the following scopes [Section 6.1](https://openid.net/specs/fapi-grant-management-01.html#name-api-authorization): 
- <code>grant_management_query</code>
- <code>grant_management_revoke</code>


#### Update a Grant

```http

GET /authorize?response_type=code&
     client_id=s6BhdRkqt3
     &grant_management_action=update
     &grant_id=TSdqirmAxDa0_-DB_1bASQ
     &scope=write
     &redirect_uri=https%3A%2F%2Fclient.example.org%2Fcb
     &code_challenge_method=S256
     &code_challenge=K2-ltc83acc4h... HTTP/1.1
Host: as.example.com

```

#### Grant Enquiry 

```http 
GET /grants/TSdqirmAxDa0_-DB_1bASQ
Host: as.example.com
Authorization: Bearer 2YotnFZFEjr1zCsicMWpAA
```

## Responses

This section of the document describes the various responses that may be returned from an authorization request. 

OAuth 2.1 requires the more secure decoupled authorization flow where the client is issued an authorization code that is then exchanged for an access token.  The responses listed in this section assume this flow. (shown in flow (4) below). 

``` http 


     +--------+                               +---------------+
     |        |--(1)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(2)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(3)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(4)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(5)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(6)--- Protected Resource ---|               |
     +--------+                               +---------------+


```

### OIDC - OAuth2.x Response

A standard OAuth 2.x response will return the access token and optionally the refresh token.  These tokens may be opaque or as JWTs. [RFC9068](https://datatracker.ietf.org/doc/rfc9068/)

The authorization server MAY fully or partially ignore the scope requested by the client, based on the authorization server policy or the resource owner's instructions.  If the issued access token scope is different from the one requested by the client, the authorization server MUST include the "scope" response parameter to inform the client of the actual scope granted.

```http

HTTP/1.1 200 OK
Content-Type: application/json

{
   "access_token": "SlAV32hkKG",
   "token_type": "Bearer",
   "expires_in": 3600,
   "refresh_token": "8xLOxBtZp8",
   "scope": "cheese"
}

```

**OAuth 2.0 response with JWT access token**

``` http 
HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store
Pragma: no-cache

{
    "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJodHRwczovL2F1dGhvcml6YXRpb24tc2VydmVyLmV4YW1wbGUuY29tLyIsInN1YiI6IjViYTU1MmQ2NyIsImF1ZCI6Imh0dHBzOi8vcnMuZXhhbXBsZS5jb20vIiwiZXhwIjoxNjM5NTI4OTEyLCJpYXQiOjE2MTgzNTQwOTAsImp0aSI6ImRiZTM5YmYzYTNiYTQyMzhhNTEzZjUxZDZlMTY5MWM0IiwiY2xpZW50X2lkIjoiczZCaGRSa3F0MyIsInNjb3BlIjoib3BlbmlkIHByb2ZpbGUgcmVhZGVtYWlsIn0.XXXXX",
    "token_type": "Bearer",
    "expires_in": 3600,
    "refresh_token": "XXXXX",
    "scope": "openid profile reademail"
}
```

``` json
 {
     "iss": "https://authorization-server.example.com/",
     "sub": "5ba552d67",
     "aud":   "https://rs.example.com/",
     "exp": 1639528912,
     "iat": 1618354090,
     "jti" : "dbe39bf3a3ba4238a513f51d6e1691c4",
     "client_id": "s6BhdRkqt3",
     "scope": "openid profile reademail"
   }
```

**Note**: Authorization Server Policy may be configured to ignore or reject unknown scopes and claims.

#### Errors

Depending upon the Authorization flow used by the client, there are a range of errors that may be returned. With particular regard to the authorization request, responses such as invalid_request, invalid_grant, invalid_Scope are defined which provide some signal to the reason for the error. 

Note: Authentication Stepup as defined in RFC 9470 provides a mechanism for the Resource Server to signal that the client requires an Authentication Stepup by the End User. This is an Authentication and not an Authorization Stepup protocol responding to exceptions relating to the `max_age` and `acr_values`. 

OAuth [2.1](https://www.ietf.org/archive/id/draft-ietf-oauth-v2-1-09.html#name-access-token)provides the following standard for errors relating to failed authorization request on a protected resource: 

`If the protected resource request included an access token and failed authentication, the resource server SHOULD include the error attribute to provide the client with the reason why the access request was declined. The parameter value is described in Section 5.2.4. In addition, the resource server MAY include the error_description attribute to provide developers a human-readable explanation that is not meant to be displayed to end-users. It also MAY include the error_uri attribute with an absolute URI identifying a human-readable web page explaining the error. The error, error_description, and error_uri attributes MUST NOT appear more than once.`


OAuth 2.0: https://datatracker.ietf.org/doc/html/rfc6749#autoid-56
OAuth 2.1: https://www.ietf.org/archive/id/draft-ietf-oauth-v2-1-09.html#name-error-codes
RFC 6750 - The OAuth 2.0 Authorization Framework: Bearer Token Usage: https://datatracker.ietf.org/doc/html/rfc6750#autoid-10 

### Rich Authorization Response

https://datatracker.ietf.org/doc/html/rfc9396#name-token-response 

This returns a token which may be enriched by the Authorization Server.

Policies may be configured to ignore or reject unknown <code>authorization_details</code> types.

If the access token is a JWT then it may contain the authorization details.

```http

HTTP/1.1 200 OK
Content-Type: application/json
Cache-Control: no-store

{
   "access_token": "2YotnFZFEjr1zCsicMWpAA",
   "token_type": "example",
   "expires_in": 3600,
   "refresh_token": "tGzv3JOkF0XG5Qx2TlKWIA",
   "authorization_details": [
      {
         "type": "payment_initiation",
         "actions": [
            "initiate",
            "status",
            "cancel"
         ],
         "locations": [
            "https://example.com/payments"
         ],
         "instructedAmount": {
            "currency": "EUR",
            "amount": "123.50"
         },
         "creditorName": "Merchant A",
         "creditorAccount": {
            "iban": "DE02100100109307118603"
         },
         "remittanceInformationUnstructured": "Ref Number Merchant"
      }
   ]
}

```

RAR also allows an AS to add further claims to the JWT presented to  the RS to assist in the RS processing (e.g. "debtorAccount" below)

Taken from : https://datatracker.ietf.org/doc/html/rfc9396#name-jwt-based-access-tokens

```http

{
   "iss": "https://as.example.com",
   "sub": "24400320",
   "aud": "a7AfcPcsl2",
   "exp": 1311281970,
   "acr": "psd2_sca",
   "txn": "8b4729cc-32e4-4370-8cf0-5796154d1296",
   "authorization_details": [
      {
         "type": "https://scheme.example.com/payment_initiation",
         "actions": [
            "initiate",
            "status",
            "cancel"
         ],
         "locations": [
            "https://example.com/payments"
         ],
         "instructedAmount": {
            "currency": "EUR",
            "amount": "123.50"
         },
         "creditorName": "Merchant A",
         "creditorAccount": {
            "iban": "DE02100100109307118603"
         },
         "remittanceInformationUnstructured": "Ref Number Merchant"
      }
   ],
   "debtorAccount": {
      "iban": "DE40100100103307118608",
      "user_role": "owner"
   }
}

```

In this case, the AS added the following example claims to the JWT-based access token:

**sub**: indicates the user for which the client is asking for payment initiation.
**txn**: transaction id used to trace the transaction across the services of provider example.com
**debtorAccount**: API-specific field containing the debtor account. In the example, this account was not passed in the <code>authorization_details</code> but was selected by the user during the authorization process. The field user_role conveys the role the user has with respect to this particular account. In this case, they are the owner. This data is used for access control at the payment API (the RS).

#### Errors

The AuthZ response is the same as an OAuth 2.x response - however the authorization code may be a JWT that contains the authorization details. 

The standard is thin on how to deal with errors in the authorization request - it is mostly focused on errors related to the data type (i.e. does not suggest range values) and does not provide any opportunity to communicate the reason for the error. (i.e. is it an authorization issue or a data issue).

Authorization Errors defined by the RAR spec are focused on type excepts - no standards are defined for Authorization based exceptions

https://datatracker.ietf.org/doc/html/rfc9396#name-authorization-error-response

### Grant Management API Response

The token response of an AS supporting the Grant Management API will include the grant_id that enables the client to manage and enquire on the grant. 

The /grants API returns a JSON-formatted response that contains the grant_id and the grant status.  The provides the client insight into grant is holds, enabling monitoring-discovery of any latent grants that may be provided to the client through the consent process - e.g. multi-party consent flows. 

Note this response is not an access token. 

```http
HTTP/1.1 200 OK
Cache-Control: no-cache, no-store
Content-Type: application/json

{
   "scopes":[
      {
         "scope":"contacts read write",
         "resources":[
            "https://rs.example.com/api"
         ]
      },
      {
         "scope":"openid"
      }
   ],
   "claims":[
      "given_name",
      "nickname",
      "email",
      "email_verified"
   ],
   "authorization_details":[
      {
         "type":"account_information",
         "actions":[
            "list_accounts",
            "read_balances",
            "read_transactions"
         ],
         "locations":[
            "https://example.com/accounts"
         ]
      }
   ]
}
```


## Specifications

### General Method for Requesting Authorization over http
OAuth 2.0: https://datatracker.ietf.org/doc/html/rfc6749
The OAuth 2.0 Authorization Framework: Bearer Token Usage: https://www.rfc-editor.org/rfc/rfc6750
Assertion Framework for OAuth 2.0 Client Authentication and Authorization Grants: https://www.rfc-editor.org/rfc/rfc7521.html

### Methods for Requesting Authorization to specialized Data Types  
OpenID Connect Core: https://openid.net/specs/openid-connect-core-1_0.html  - for Authorization to identity data in via id_token and /userInfo API
Self Issued OIDC https://openid.net/specs/openid-connect-self-issued-v2-1_0.html - 
OpenID for Verifiable Presentations: https://openid.net/specs/openid-4-verifiable-presentations-1_0.html - for Authorization to vp_token
OpenID for Verifiable Credential Issuance: https://openid.net/specs/openid-4-verifiable-credential-issuance-1_0.html - for Authorization at a Credential Endpoint.

### Alternate Methods for Requesting Authorization
Rich Authorization Requests:  https://datatracker.ietf.org/doc/html/rfc9396
JWT-Secured Authorization Request (JAR): https://datatracker.ietf.org/doc/html/rfc9101
Pushed Authorization Requests (PAR): https://datatracker.ietf.org/doc/html/rfc9126

### Methods for Inspecting and Managing Authorization Grants
Grant Management API: https://openid.net/specs/fapi-grant-management-01.html

### Authorization Semantics
Resource Indictors for OAuth: https://www.rfc-editor.org/rfc/rfc8707.html
JSON Web Token (JWT) Profile for OAuth 2.0 Access Tokens: https://datatracker.ietf.org/doc/html/rfc9068 
OAuth 2.0 Step Up Authentication Challenge Protocol https://datatracker.ietf.org/doc/rfc9470/