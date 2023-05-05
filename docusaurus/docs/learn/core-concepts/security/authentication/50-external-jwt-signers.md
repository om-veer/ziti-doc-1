# External JWT Signers

External JWT Signers allow external identity providers to facilitate authentication with a Ziti network. External
JWT Signers can be added as a static x509 certificate or via a JWKS endpoint. Authenticating clients can provide
a JWT as a primary authentication mechanism to obtain an [API Session](../sessions.md#api-session). Additionally, the JWT can be required on
all REST API calls if desired by using an Authentication Policy that requires it as a secondary factor.

JWT is described in [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) and on [Wikipedia](https://en.wikipedia.org/wiki/JSON_Web_Token).
X509 PKI is described in [RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) an on [Wikipedia](https://en.wikipedia.org/wiki/X.509)

# Usage

External JWT Signers are used in conjunction with Authentication Policies to allow identities to authenticate using a
JWT. External JWT Signers can be configured to match an identity through the `claimsProperty` and `useExternalId`
fields. `claimsProperty` can change which field in the JWT is matched to the `id` or `externalId` on an identity.
`useExternalId` when true will match the `claimsProperty` to an identity's `externalId` field. If false, it will match
to the `id` field.

_Note: `externalId` values on identities are enforced to be unique._

`claimsProperty` can contain JWT standard claims or private claims. An example usage would be an `email` JWT private
claim and Ziti identities with `externalId` set to email addresses.


## x509 Certificate

If the JWT provider has a static x509 certificate, it is possible define an External JWT Signer using the PEM encoded
public certificate of the signer. If the JWT provider has JWKS endpoint support it is strongly recommended to create
an External JWT Signer using the JWKS endpoint. External JWT Signers configured with a static x509 certificate will
need maintenance during key rotation and certificate expiration. For these operations the Edge Management API 
can be used for external automation.

## JWKS Endpoint

JSON Web Key Sets (JWKS) is defined in [RFC7517 Section 5](https://datatracker.ietf.org/doc/html/rfc7517#section-5)
and is used by many popular IdPs (Auth0, Okta) in order to enable external JWT verification. External JWT Signers
configured with a JWKS endpoint allows an identity provider to rotate keys and bridge signer certificate expiration
windows.

# JWT Validation

External JTW Signers are used to validate JWTs for authentication. This validation requires the following:

- the `iss` field on the JWT must match the `issuer` field on the External JWT Signer
- the `aud` field on the JWT must match the `audience` field on the External JWT Signer
- the `sub` or field defined by `claimsProperty` must match the `id` field on an identity (or `externalid` if `useExternal` is true)
- the JWT must be signed by the x509 certificate or a JWK from the JWKS endpoint defined on the External JWT Signer
- the JWT `kid` must match the External JWT `kid` field for x509 certificates or the `kid` in a JWKS response
- the JWT must not be expired

# Creation 

An External JWT Signer that uses a private `email` claim and matches on `externalId` with a JWKS endpoint can
be created as follows:

`POST /edge/management/v1/ext-jwt-signers`
```json
{
  "name": "My External JWT Signer",
  "enabled": true,
  "issuer": "my-issuer",
  "audience": "my-audience",
  "jwksEndpoint": "https://example.com/.well-known/jwks.json",
  "claimsProperty": "email",
  "useExternalId": true
}
```

An External JWT Signer that uses the `sub` claim and matches on `id` with a x509 certificate can
be created as follows:

`POST /edge/management/v1/ext-jwt-signers`
```json
{
  "name": "My External JWT Signer",
  "enabled": true,
  "issuer": "my-issuer",
  "audience": "my-audience",
  "kid": "2gh80220",
  "certPem": "-----BEGIN CERTIFICATE-----\nMII...\n-----END CERTIFICATE-----\n"
}
```

# External Authentication URL & Client Authentication

Unauthenticated clients can enumerate External JWT Signers that have an `externalAuthUrl` property. Clients will
receive the name of the External JWT Signer and the `externalAuthUrl` only. This information can allow clients
to initiate client authentication to the target identity provider. 

Example Client External JWT Signer response:

`GET /edge/client/v1/external-jwt-signers`
```json
{
  "data": [
    {
      "_links": {
        "self": {
          "href": "./external-jwt-signers/eZjzX3U1u"
        }
      },
      "id": "eZjzX3U1u",
      "name": "my-ext-jwt-signer",
      "externalAuthUrl": "https://my-site/authenticate"
    }
  ],
  "meta": {}
}
```