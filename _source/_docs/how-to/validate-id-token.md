---
layout: docs_page
title: Validating ID Tokens on a Client
excerpt: How to validate ID tokens in your client-side application
---

## Overview

If your client application requires authentication and would like to obtain information about the authenticated person, then it should use the OpenID Connect protocol to get an ID token. 

OpenID Connect (OIDC) is an authentication protocol built on top of OAuth 2.0. With OAuth 2.0, a user can authenticate with an authorization server and get you an access token that authorizes access to some server resources. With OIDC, they can also give you a token called an ID token. The ID token contains information about a user and their authentication status. It can be used by your client both for authentication and as a store of information about that user. One OIDC flow can return both access and ID tokens.

> For more information on issuing ID tokens, see here (jakub.todo). 

We will now cover the terms used in this document, and an explanation of why you should use ID tokens. 

- If you'd like to jump straight to the local validation steps, click here: (jakub.todo) 
- If you'd like to see how to validate a token directly with Okta, click here: (jakub.todo)
- If you want to see specifically how to accomplish this using an Okta SDK, click here: (jakub.todo)

### Terms 

Although OpenID Connect is built on top of OAuth 2.0, the specification uses slightly different terms:

- The OpenID Provider (OP), which is the authorization server that issues the ID token. In this case Okta is the OP. For more information about setting-up Okta as your authorization server, go here: [Set Up Authorization Server](https://developer.okta.com/docs/how-to/set-up-auth-server.html).
- The End-User whose information is contained in the ID token.
- A Claim is a piece of information about your End-User. 
- The Relying Party (RP), which is the client application that requests the ID token from Okta.

With these terms in mind, you should be able to parse this sentence from the OpenID Connect spec: `The ID Token is a security token that contains Claims about the Authentication of an End-User by an Authorization Server when using a Client`.

More information about all of this can be found in our high-level discussion of OpenID Connect, which you can find here: (jakub.todo).

The ID tokens are in JSON Web Token (JWT) format, the specification for which can be found here: <https://tools.ietf.org/html/rfc7519>. They are signed using private JSON Web Keys (JWK), the specification for which you can find here: <https://tools.ietf.org/html/rfc7517>.

## ID Tokens vs Access Tokens

The ID Token is a security token granted by the OpenID Provider that contains information about an End-User. This information tells your client application that the user is authenticated, and can also give you information like their username or locale.

You can pass an ID Token around different components of your client, and these components can use the ID Token to confirm that the user is authenticated and also to retrieve information about them.

Access tokens, on the other hand, are not intended to carry information about the user. They simply allow access to certain defined server resources. More discussion about when to use access tokens can be found in How To Validate Access Tokens (jakub.todo).

More information about ID tokens can be found here: (jakub.todo).

## What to Check When Validating an ID Token 

The high-level overview of validating an ID token looks like this:

(jakub.todo link to sections below)

- Retrieve and parse your Okta JSON Web Keys (JWK), which should be checked periodically and cached by your application.
- Decode the ID token, which is in JSON Web Token format.
- Verify the signature used to sign the ID token
- Verify the claims found inside the ID token

### Retrieve The JSON Web Keys

The JSON Web Keys (JWK) need to be retrieved from your [Okta Authorization Server](https://developer.okta.com/docs/how-to/set-up-auth-server.html), though your application should have them cached. Specifically, your Authorization Server's Metadata endpoint contains the `jwks_uri`, which you can use to get the JWK. 

> For more information about retrieving this metadata, see [Retrieve Authorization Server Metadata](https://developer.okta.com/docs/api/resources/oauth2.html#retrieve-authorization-server-metadata).
 
### Decode the ID Token

You will have to decode the ID token, which is in JWT format. Here are a few examples of how to do this:

(jakub.todo. Link to code section below)

### Verify the Token's Signature

You verify the ID token's signature by matching the key that was used to sign in with one of the keys you retrieved from your Okta Authorization Server's JWK endpoint. Specifically, each public key is identified by a `kid` attribute, which corresponds with the `kid` claim in the ID token header.

If the `kid` claim does not match, it is possible that the signing keys have changed. Check the `jwks_uri` value in the Authorization Server metadata and try retrieving the keys again from Okta.

Please note the following:

- For security purposes, Okta automatically rotates keys used to sign the token.
- The current key rotation schedule is four times a year. This schedule can change without notice.
- In case of an emergency, Okta can rotate keys as needed.
- Okta always publishes keys to the `jwks_uri`.
- To save the network round trip, your app should cache the `jwks_uri` response locally. The [standard HTTP caching headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) are used and should be respected.
- The administrator can switch the Authorization Server key rotation mode by updating the Authorization Server's `rotationMode` property. For more information see the API Reference: [Authorization Server Credentials Signing Object](https://developer.okta.com/docs/api/resources/oauth2.html#authorization-server-credentials-signing-object).

### Verify the Claims

You should verify the following:

- The `iss` (issuer) claim matches the identifier of your Okta Authorization Server.
- The `aud` (audience) claim is the value configured in the Authorization Server.
- The `iat` (Client ID) claim is your application's Client ID.
- The `exp` (Expiry Time) claim is the time at which this token will expire. You should make sure that this has not already passed.

## Validating A Token Remotely With Okta

Alternatively, you can also validate an Access or Refresh Token using the Token Introspection endpoint: [Introspection Request](https://developer.okta.com/docs/api/resources/oauth2.html#introspection-request). This endpoint takes your token as a URL query and returns back a simple JSON response with a boolean `active` property. 

This incurs a network request which is slower to do verification, but can be used when you want to guarantee that the access token hasn't been revoked. 

## Okta Libraries to Help You Verify Access Tokens

(jakub.todo) This may not be a great name for this section.

Link out to SDK-specific info on how to validate tokens here.

- <https://okta.github.io/code/php/jwt-validation.html>

Don't see the language you're working in? Get in touch: <developers@okta.com>