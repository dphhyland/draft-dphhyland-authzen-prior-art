
This document provides a review of the various OAuth and OpenID specifications and how each supports the concept of authorization. 

The purpose of this document is to provide context for the development of new standards within the AuthZen working group.

# Prior Art: OIDC - OAuth 2.x

Provides a protocol for the authorization of a client to access a resource on behalf of an End-User. 

Standard OAuth 2.x spec provides for the authorization of the scopes requested by the cilent

Creates a Grant on the Authorization Server that describes the authorization of the access granted to the client. 

ODIC extends the authorization process to require the Authorization Server Authenticate the user to the client through the inclusion of the openid scope. 
OIDC defines the ability to request identity information through the claims parameter,and allows the request to target the API the claim is return through i.e.userInfo and id_token endpoints.

The Authentication Request typically includes a consent flow through which the End-User consents to the claims and scopes being requested by the client. 

Note: OAuth 0 OIDC ignore unknown scopes and claims (but is dependent upon the AS policy)

I've covered here the core authorization flows of OIDC and OAuth - other standard such as CIBA do have further characteristics related to the authorization process. 


## Requests

These request may require the consent or authorization of the end user or resource owner.
In some circumstances the request may be for a resource that is jointly owned and requires the consent/authorization of multiple parties.

I'm being specific with the use of the term authorization here - as the OAuth spec uses the term authorization to describe the process of the client obtaining a token from the authorization server, however consent is also a form of authorization but has developed broader meaning through the development of Privacy regulations worldwide. 

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
_Pushed Authorization Requests (PAR): Requests provided via a Pushed Authorization Request (PAR) endpoint.
_JWT-Secured Authorization Request (JAR): Requests encoded in a JWT and provided either within the request or via uri. 
_Rich Authorization Requests (RAR): Authorization Details provided as json via the authorization_details parameter in the authorization request. (Note: authorzation_details may also be used on token refresh, however, this only narrows the scope of the returned token, and does not change the authorization of the client grant.) 

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

The only mandatory parameter is the type parameter. The type parameter is a string that identifies the type of the authorization details. All other parameters are optional and up to the implementer to define. 

An Authorization Request may also be placed on the token endpoint; however, this only narrows the scope of the returned token and does not change the authorization of the client grant. When this token is presented to the resource server the narrowed token scope is enforced.  

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

```http

```

## Responses




### OIDC - OAuth2.x Response

A successful client authorization request will result in a redirect to the client redirect_uri with the authorization code or token. 

```http

  HTTP/1.1 302 Found
  Location: https://client.example.org/cb?
    code=SplxlOBeZQQYbYS6WxSbIA
    &state=af0ifjsldkj

```


**Note**: Authorization Server Policy may be configured to ignore or reject unknown scopes and claims.

#### Errors



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
RAR also allows an AS to add further claims to the JWT presented to  the RS to assist in the RS processing 

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

Authorization Errors defined by theRAR spec are focused on type excepts - no standards are defined for Authroization based exceptions

https://datatracker.ietf.org/doc/html/rfc9396#name-authorization-error-response

### Grant Management API Response



#### Errors



## Specifications

### General Method for Requesting Authorization over http
OAuth 2.0: https://datatracker.ietf.org/doc/html/rfc6749

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
