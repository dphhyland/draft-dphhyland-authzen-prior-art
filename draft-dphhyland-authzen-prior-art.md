
This document provides a review of the various OAuth and OpenID specifications and how each supports the concept of authorization. 

The purpose of this document is to provide context for the development of new standards within the AuthZen working group.

# Prior Art: OIDC - OAuth 2.x

## Introduction

OAuth describes a set of standards that define the protocols for the authorization of a client to access APIs. OIDC extends the authorization capability of this protocol, by providing a user context to the authorization process, enabling access control to resources based on the identity of the user. Until recently OAuth has provided few mechanisms to describe the access rights of the client; however, recent developments in open ecosystems, such as Open Banking, and evolution of standards to support strong identity using verifiable credentials have driven the development of new standards which provide a more expressive authorization process. This document provides a review of the various OAuth and OpenID standards, with a particular focus on the available mechanisms used by the client to enquire and request authorization for an end-user. 

Conceptually and in practice there are four key activities in OAuth that perform an authorization function:
- **Client Authorization Request ( /authorize | /par | /bc-authorize )**: Invoked by the user agent/client on the Authorization Server (AS) for the purposes of obtaining an access token. This performs an authorization of attributes provided in the request: if successful returns an authorization code (code flow) or access token(other); if unsuccessful returns an error.
- **Token Request (/token)**: The issuance of a token to the client validates the state of the grant, and any specific authorization parameters provided with the authorization request. 
- **Resource Server Authorization (incl /userInfo)**: This is the verification the token presented to the resource server is valid and that the token has the required authorization to access the resource.
- **Grant Management Enquiry (/grants)**: While this api does not act as an authorization control, it does return the authorization state of the client grant.

**User Agent/Client Authorization**: Today there are a number of mechanisms in OAuth and OIDC that enable the client to communicate its Authorization requirement: Within the Authorization Request through Scopes (including finer grained claims defined within OIDC) (REF)[] the authorization_details request parameter defined within the RAR standard, and on the token request through the authorization_details [REF]() parameter. Recently presentation_definition has also been introduced to enable a Verifier to request a Verifiable Presentation from a Holder [REF]() - this attribute type is specific the syntax defined by Section 5 of [DIF.Presentation Exchange](https://specs.identity.foundation/presentation-exchange/spec/v2.0.0/) (**this raises the question on how to consider the authorization_details attribute type**) Each of these attributes enable a user agent/client to describe the authorization it requires ( *from a traditional AS or Wallet* ), which may automatically authorize the request or trigger end user interaction from one or many users that hold ownership over the resources the authorization is being requested for.

**Consent**: Consent is increasingly used within digital activities to capture end user authorization to act on user data. Consent is captured  in many ways and for many different reasons, and has become a key component of privacy regulations such as GDPR. Schemes such as Open Banking have also adopted the concept of consent for the capture and use of data by third parties, which has driven the adoption of OIDC/FAPI use cases which employ the capture of consent over a wide range of scopes beyond that defined in the OIDC standard. Open Banking has also opened a degree of ambiguity in the concept of consent through the authorization requirement over joint accounts, with the capture "consent" of multiple parties for jointly owned Bank Accounts. From an OAuth/OIDC point of view, this user interaction is outside of the standard, and transparent to the client - which presents challenges to the client in understanding the authorization it has been granted. While recent standards such as the (Grant Management API)[] attempt to provide the client with insight into the authorization granted, for reasons of privacy, it is not possible to provide the authorization requirement of jointly held resources. There is a need for further standards in this space to signal to the client that there are outstanding end user activities to be completed - and potentially provide the client information on how to obtain this authorization.

**User Agent/Client Visibility**: Conceptually OAuth and OIDC standards related to authorization are specific to the Authorization requirement for the user agent/client which requires access to user data. This stems from the origins of OAuth which did not hold the concept of identity of the end-user and thus the inferred access that user may have over a resource. Applying this concept consistently through to the Grant Management API and authorization responses such as authorization_details, does provide the framework for transparency to what authorizations are owned by the client and that owned by the user(s). (For example the client may have obtained a consent for a user to make a payment, that is not the same as the user in control of the user agent). 

This concept needs to be considered carefully when looking to extend OAuth to to better support the authorization - the authorization of a client may be different to that if the user - and any standards that are developed need to be clear on the authorization context and the authority the user-agent/client holds.

-- Need to understand the relationship between the AS and RS here and if there 

**Authorization Signalling**:communicating to the client authorization state is poorly defined in OAuth/OIDC standards. Recent standards relating to [Authentication Stepup]() have been defined, the [Grant Management API] now provides insight in the the currently held grant of a client, but otherwise generic HTTP error codes are used to signal authorization outcomes to a client. 

**Authorization Metadata**: 


**Resource Authorization**: 

**OpenID for Verifiable Credentials**: 


## Requests

These request may require the consent or authorization of the end user or resource owner.
In some circumstances the request may be for a resource that is jointly owned and requires the consent/authorization of multiple parties.

I'm being specific with the use of the term authorization here - as the OAuth spec uses the term authorization to describe the process of the client obtaining a token from the authorization server, however consent is also a form of authorization but has developed broader meaning through the development of Privacy regulations worldwide. 


Note: OAuth 0 OIDC ignore unknown scopes and claims (but is dependent upon the AS policy)


### Standard OAuth RFC 6749

Request for the return of an Access Token (and optionally a Refresh Token) from the Authorization Server, which will grant a client access to a resource on behalf of a End-User. 

The scope parameter is used to describe the access that the client is requesting.
scopes


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
- **Rich Authorization Requests (RAR)**: Authorization Details provided as json via the authorization_details parameter in the authorization request. (Note: authorzation_details may also be used on token refresh, however, this only narrows the scope of the returned token, and does not change the authorization of the client grant.) 

Extensions to authorization request:
- resource: Section 2 RFC 8707 Resource Indicators for OAuth 2.0 https://www.rfc-editor.org/rfc/rfc8707
- authorization_details: RAR

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

### Open ID for Verifiable Credentials



### Rich Authorization Requests

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
URL Decoded "authorization_details

- The only mandatory parameter is the type parameter. The type parameter is a string that identifies the type of the authorization details. All other parameters are optional and up to the implementer to define. 

- An Authorization Request may also be placed on the token endpoint; however, this only narrows the scope of the returned token and does not change the authorization of the client grant. When this token is presented to the resource server the narrowed token scope is enforced.  

#### Metadata
https://datatracker.ietf.org/doc/html/rfc9396#name-metadata

the RAR spec provides for the advertsiment of the RAR capability through the metadata endpoint. 

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

The standard allows for APIs to query the status of a grant and to revoke a grant. 

To call the GM API the client requires authorization of the following scopes [Section 6.1](https://openid.net/specs/fapi-grant-management-01.html#name-api-authorization): 
- grant_management_query
- grant_management_revoke


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

OAuth is token based authorization protocol where the response of the authorization is an access token that is then used by a "client" to access a resource. 

OAuth 2.1 requires the more secure decoupled authorization flow where the client is issued an authorization code that is then exchanged for an access token.  The responses listed in this section are the responses which issue a bearer token used by the client to access API resources. (shown in flow (4) below).

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

A standard OAuth 2.x response will return the access token and optionally the refresh token.  These tokens may be opaque or as JWTs. (RFC9068)

The standard will ignore any unknown scopes requested by the client, and depending upon policy may ignore or reject claims.

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

*OAuth 2.0 response with JWT access token*

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

Depending upon the Authorization flow used by the client, there are a range of errors that may be returned. With particular regard to the auhtorization request, responses such as invalid_request, invalid_grant, invalid_Scope are defined which provide some signal to the reason for the error. 

Note: Authentication Stepup as defined in RFC 9470 provides a mechanism for the Resource Server to signal that the client requires an Authentication Stepup by the End User. This is an Authentication and not an Authorization Stepup protocol responding to exceptions relating to the `max_age` and `acr_values`. 

OAuth [2.1](https://www.ietf.org/archive/id/draft-ietf-oauth-v2-1-09.html#name-access-token)provides the following standard for errors relating to failed authorization request on a protected resource: 

`If the protected resource request included an access token and failed authentication, the resource server SHOULD include the error attribute to provide the client with the reason why the access request was declined. The parameter value is described in Section 5.2.4. In addition, the resource server MAY include the error_description attribute to provide developers a human-readable explanation that is not meant to be displayed to end-users. It also MAY include the error_uri attribute with an absolute URI identifying a human-readable web page explaining the error. The error, error_description, and error_uri attributes MUST NOT appear more than once.`


OAuth 2.0: https://datatracker.ietf.org/doc/html/rfc6749#autoid-56
OAuth 2.1: https://www.ietf.org/archive/id/draft-ietf-oauth-v2-1-09.html#name-error-codes
RFC 6750 - The OAuth 2.0 Authorization Framework: Bearer Token Usage: https://datatracker.ietf.org/doc/html/rfc6750#autoid-10 

### Rich Authorization Response

https://datatracker.ietf.org/doc/html/rfc9396#name-token-response 

This returns a token which may be enriched by the Authorization Server.

Policies may be configured to ignore or reject unknown authorization_details types.

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
**debtorAccount**: API-specific field containing the debtor account. In the example, this account was not passed in the authorization_details but was selected by the user during the authorization process. The field user_role conveys the role the user has with respect to this particular account. In this case, they are the owner. This data is used for access control at the payment API (the RS).

#### Errors

The AuthZ response is the same as an OAuth 2.x response - however the authorization code may be a JWT that contains the authorization details. 

The standard is thin on how to deal with errors in the authorization request - it is mostly focused on errors related to the data type (i.e. does not suggest range values) and does not provide any opportunity to communicate the reason for the error. (i.e. is it an authorization issue or a data issue).

Authorization Errors defined by the RAR spec are focused on type excepts - no standards are defined for Authroization based exceptions

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