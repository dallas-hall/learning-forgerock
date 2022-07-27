# AM410 Day 3 - Extending Services Using OAuth2-Based Protocols

- [AM410 Day 3 - Extending Services Using OAuth2-Based Protocols](#am410-day-3---extending-services-using-oauth2-based-protocols)
  - [Lesson 1 - Integrating Applications With OAuth2](#lesson-1---integrating-applications-with-oauth2)
    - [OAuth2](#oauth2)
      - [Overview](#overview)
      - [Concepts](#concepts)
    - [OAuth2 In AM](#oauth2-in-am)
      - [Authorisation Server](#authorisation-server)
      - [OAuth2 Client](#oauth2-client)
    - [Labs](#labs)
      - [OAuth2](#oauth2-1)
  - [Lesson 2 - Integrating Applications With OIDC](#lesson-2---integrating-applications-with-oidc)
    - [ODIC Overview](#odic-overview)
  - [Lesson 3 - Authenticating OAuth2 Clients and using mTLS in OAuth2 for PoP](#lesson-3---authenticating-oauth2-clients-and-using-mtls-in-oauth2-for-pop)
    - [4 Authentication Modes](#4-authentication-modes)
- [PoP & mTLS](#pop--mtls)
    - [Labs](#labs-1)
  - [Lesson 4 - Transforming OAuth2 Tokens](#lesson-4---transforming-oauth2-tokens)
    - [Labs](#labs-2)
  - [Lesson 5 - Implementing Social authentication](#lesson-5---implementing-social-authentication)
    - [Social Access](#social-access)
    - [Labs](#labs-3)

## Lesson 1 - Integrating Applications With OAuth2

### OAuth2

#### Overview

![](images/am401/am-oauth2-18.png)

https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc - OAuth 2.0 is a security standard where you give one application permission to access your data in another application. The steps to grant permission, or consent, are often referred to as authorization or even delegated authorization. You authorize one application to access your data, or use features in another application on your behalf, without giving them your password. Sweet!

![](images/am401/am-oauth2-19.png)

![](images/am401/am-oauth2-20.png)

OAuth2 is a framework that defines a flexible agreement for how applications interact. Provides a method for an application to access resources with the resource owner consent, respecting consent withdrawal. With OAuth2, the resource owner credentials do not need to be shared. The result of an OAuth2 handshake is a security token, called an access token. JWTs are commonly used because it is an easy way to encode and pass around JSON data without having to escape it for different contexts.

Some OAuth2 usages are:
* Consumer applications:
  * Mobile applications
  * IoT/low-level devices
* REST API protection
* Authorization in applications without identity data, or use OIDC
* Open Banking solutions

**Note:** OAuth2 is an authorization protocol, not an authentication protocol.

![](images/am401/am-oauth2-1.png)

Four main actors take part in OAuth2 processes:
1. Resource owner (or end user): An entity able to grant access to a protected resource.
2. Resource server (an API server): Hosts protected resources; able to accept and respond to requests with access tokens for accessing a protected resource.
3. Client: An application that requests access to protected resources on behalf of the resource owner and with its authorization.
4. Authorization server: Issues access tokens to the client after the resource owner successfully authenticates and provides authorization.

![](images/am401/am-oauth2-4.png)

https://www.digitalocean.com/community/tutorials/an-introduction-to-oauth-2

![](images/am401/am-oauth2-5.png)

The access token gives access to the bearer of the token to specified resources for a limited period of time. Once the client receives the access token, it can make a request to the resource server

In most use cases, the authorization server and the resource server are controlled by the same organization. The resource server must be able to process (validate) the access token issued by the authorization server.

#### Concepts

* The **Resource Owner (RO)**:
  * Typically wants to access a service related to some resources they own.
  * Must give consent to the application by authenticating with the **Authorization Server (AS)**, and allow consent using a consent form.
* The **Resource Server (RS)**
  * Often runs from the same organization as the AS.
  * Stores resources held by the RO.
  * Allow access or perform actions on the resource. This is usually implemented using a REST API.
  * Must validate the access token with the AS and may also be able to retrieve further information from the AS.
* The **OAuth2 clients (OC)**:
  * Receive a request from the RO.
  * Obtain a mandatory access token.
  * Request resources from the RS API by presenting an access token that must be validated by the RS.
* The AS:
  * Authenticates RO and requests their consent.
  * Validates client configuration at runtime.
  * Issues access tokens, and possibly an authorization code and refresh tokens.
  * Exposes REST API endpoints to validate and retrieve access token information.
* **Access Tokens (AT)**:
  * Are commonly either an opaque token or a JW
  * Are the result of all OAuth2 grant type flows.
  * Are bearer tokens, which means anyone presenting it can obtain access, and it needs to be protected with TLS and **Proof-of-Possession (PoP)**.
  * May include key-value pairs that can be modified by using a script.

![](images/am401/am-oauth2-2.png)

* AT Validation:
  * When clients present the access token to the RS API to obtain a resource, the RS must validate it to AM before returning resources.
  * Validation requires:
    * Verifying the access token's existence.
    * Checking the access token has not expired; they are typically short lived.
* AT PoP
  * Follows Proof-of-Possession Key Semantics for **JSON Web Tokens (JWT)** internet-draft.
  * Associates PoP key with a JWT access token.
  * PoP is not configured by default.
* **Authorization Code (AC):**
  * Are requested in the OAuth2 Authorization Code flow:
    * User logs in to AM and gives consent.
    * Client receives the authorization code.
    * Client exchanges the authorization code for an access token.
  * Are credentials representing the RO's authorization.
  * Provide security benefits: Authenticate to the client and send the access token directly to client, not via the browser.
* **Refresh Tokens (RT):**
  * Are used to obtain a new access token, which are short lived for security reasons.
  * Are opaque, not JWTs, and cannot be decoded and verified.
  * Can be used in one place, the token endpoint of an authorization server.
  * Can be obtained when receiving an access token.
  * Are not allowed nor recommended for all grant type flows.

![](images/am401/am-oauth2-3.png)

* **Macaroons**:
  * Are a type of bearer token issued with a broad scope to a client, in place of regular OAuth2 access or refresh tokens.
  * Are designed to share a single token with multiple clients and resource servers without compromising security.
  * Can be created by the client, as needed, from the single macaroon access token.
  * Offer distributed token modification restricting scopes by using caveats.
  * Are useful in microservices: A single client delegates tasks to services with restrictions.

Using an OAuth2 Access Token Modification Script, you can modify the key-value pairs contained within an OAuth2 access token. Token modification scripts work for client-based and CTS-based access tokens. Modification works when macaroons are used in place of regular tokens.

* Obtaining Authorization and Access Tokens:
  * OAuth2 grant types are interaction flows designed for different clients to acquire access tokens.
    * There are a lot of OAuth2 grant types.
    * Each grant type was developed with specific use cases in mind.
  * All grant type flow communication must be performed through a secure channel.

![](images/am401/am-oauth2-6.png)

Authorization Code grant type is a two-step interactive process used when the client, running on a server, requires access to protected resources.

The Authorization Code grant type flow is intended for server-side web applications that can keep the client secret secure, and can be used when a web browser is available to redirect the end user to the authorization server.

![](images/am401/am-oauth2-7.png)

After the access token is acquired as the result of the end of the Authorization Code grant type flow, or any grant type flow, it is best practice that the access token is validated before access to the resource is given. If the RO refuses to grant permission, the OAuth2 process stops.

![](images/am401/am-oauth2-8.png)

In this case, the access token is retrieved by the client through communicating directly with AM. Because the client must authenticate to request and obtain an access token, extra security is offered only if the client can securely store the client secret.

The Authorization Code and Device Code grant types, when combined with the **Proof Key Code Exchange (PKCE)** standard RFC7636: Proof Key for Code Exchange by OAuth Public Clients, are used when the client, usually a mobile or a JavaScript application, requires access to protected resources. The client must generate a code that must be part of the communication between the client and the authorization provider. This code mitigates against interception attacks performed by malicious users.

**Note:** The example would actually use OIDC and not just OAuth2. OIDC is discussed in the next lesson.

![](images/am401/am-oauth2-9.png)

It is best practice that the client sends that access token to the resource server when requesting access to a resource. The resource server should always validate the access token with the authorization server.

![](images/am401/am-oauth2-10.png)

Client Credentials grant type is a suitable choice when:
* The protected resources are controlled by the client or public.
* The resources are public.
* The flow does not involve an end user.

![](images/am401/am-oauth2-11.png)

The graphic shows a real-life use case example where the client credentials grant could be used in a machine-to-machine (M2M) interaction. It includes:
* The client as the survey application.
* The resource server as the survey service that provides a REST API to view survey data.
* The OAuth2 authorization server, which is hosted by the organization that provides the survey resource server.

![](images/am401/am-oauth2-12.png)

Device Code grant type flow is intended for devices with a limited user interface and no web browser.

**Note:** AM does not display the confirmation nor the consent pages if the user has a valid session when entering the code, and the client is allowed to skip consent.

![](images/am401/am-oauth2-13.png)

Device Code with PKCE example.

Implicit grant type:
* Has been deprecated because it is hard to secure; use Authorization Code with PKCE instead.
* Used inside a web browser as a simple all-in-one request.

Resource Owner Password Credentials grant type:
* Is a last resort for legacy applications; not for new applications.
* Is used for single request; no need to store the credentials

Recommended OAuth2 Guidelines:
* Always use secure communications (e.g. TLS) between OAuth2 components (players).
* The resource server should validate every access token it receives.
* Confirm the access token grants the authorizations (scopes) that match the action requested.
* Choose the appropriate grant type flow for the application.

OAuth2 grant type flows require scopes to limit the client's access to resources. Scopes are:
* A method to restrict client access to the RO's resources.
* Not associated with data.
* Concepts specified as strings, that the resource server interprets, to provide access or resources to the client.
* Dependent on the architecture of your environment.

A client can request one or more scopes, which AM may display in the consent screen. If the RO agrees to share access to their resources, scopes are included in the access token.

OAuth2 Scopes:
* Help the resource server to determine if a resource should be returned.
* Are specific to the application and resource server requirements.

![](images/am401/am-oauth2-14.png)

Scopes can be granted statically or dynamically. When AM receives a request for an access token, it decides which scope to grant. AM decides which scope to grant statically or dynamically when:
* The RO authenticates successfully.
* The client is registered with scopes. For example, scope A and B; and there is a request for scope A and B.

AM grants the scopes:
* Statically: The RO is redirected to a client registered with requested scopes; then scope A and B is granted for each user.
* Dynamically: The RO is redirected to a client with a list of scopes and authorization policies evaluated; then each user may get different scopes based on policy conditions.

Many OAuth2 and OIDC flows require the user to explicitly consent (agree) to provide the client with access to their resources. One of the pillars of OAuth2 and OIDC is the act of trust by giving consent. OAuth2 Consent:
* Is required by many OAuth2 and OIDC flows to access a resource.
* Is an act of trust and one of the pillars of OAuth2 and OIDC.
* Is granted by users based on scopes, which can limit information shared with, and actions performed by the client.
* Can be stored to minimise user interaction.
* Can be revoked by users at any time.
* Can be skipped when no user interaction is needed for trusted applications or services.

### OAuth2 In AM

OAuth2 client functionality is implemented through the use of AM authentication nodes in a tree. Therefore, you do not need to deploy your own resource server implementation when AM is an OAuth2 client. In this case, you use web or Java agents or IG to protect resources.

AM can be configured to take the role of:
* An authorization server (or authentication provider)
  * To authenticate ROs and obtain authorization before returning an access token to a client.
  * By registering confidential or public clients statically or dynamically.
* An OAuth2 client
  * To provide a session after successfully authenticating the RO and obtaining authorization.
  * To access resources protected by agents.
  * By using OAuth2/OIDC nodes in authentication trees.

#### Authorisation Server

1. In the AM Admin UI, select the desired realm.
2. On the realm page, select Service, then select Add a Service.
3. From the drop-down menu, select the OAuth2 Provider service, then select Create.
   1. If the OAuth2 Provider service is already configured in the realm, select OAuth2 Provider to edit the configuration.

![](images/am401/am-oauth2-15.png)

For clients to communicate with AM, configured as an OAuth2 Provider authorization server, they must be registered with AM. Registering the client configures AM to become aware of them, and provides the means for clients to both connect (authenticate) and communicate with AM to obtain an access token.

#### OAuth2 Client

2 types of client types:
1. Public:
   1. Can't be trusted to keep a secret. e.g. Native applications and Single Page Applications (SPAs).
   2. Not required to authenticate.
2. Confidential:
   1. Can be trusted to keep a secret. e.g. Web applications and services.
   2. Required to authenticate. Can use mTLS.

3 types of client profiles:
1. Native/SPA, Native apps are developed for specific platforms or devices. e.g. Mobile apps.
2. Web apps, are clients that run on a web server. End users (ROs) access web apps using a web browser. The web app makes API calls using a server-side programming language.
3. Service Machine to Machine (M2M) or service apps interact with an API, without user involvement. The M2M app acts on behalf of itself, and not a user, and can ask for an access token directly without involving a user at all.

The client type registered can determine the grant type flow used for communication with AM.

Changing the client profile configuration depends on your working knowledge of the OAuth2: the grant types and components.

![](images/am401/am-oauth2-16.png)

There are a variety of OAuth2 endpoints. e.g. `/oauth2/authorize` and `/oauth2/access_token` which provide OAuth2 related data.

AM OAuth2 services are stateless unless otherwise indicated; they do not hold any token information locally. Instead, the OAuth2 service either stores the OAuth2/OIDC tokens in:
* CTS token store
* Clientside

![](images/am401/am-oauth2-17.png)

Macaroons are layered on top of existing CTS-based and client-based OAuth2 tokens. When macaroons are enabled, AM issues:
* CTS-based macaroon tokens, where:
  * The access token is stored in the CTS.
  * The macaroons are issued to clients as tokens, and the macaroon ID is a reference to the access token in the CTS.
* Client-based macaroon tokens, such that:
  * The access token is a signed and/or encrypted JWT wrapped in a macaroon.
  * The resulting token size may impact deployment, is limited by browser token size, and may impact network performance.

### Labs

**Note:** You can download an FR API collection for Postman and it will install common FR API end points.

#### OAuth2

Before OAuth2 client applications can communicate with AM as an authorization server, AM must be configured with an OAuth2 Provider service, which is also an OIDC Provider.

* Subscribers of the ForgeRock Entertainment Company (FEC) website may use a Smart TV (the OAuth2 client) to stream content. To be able to stream content, the Smart TV needs to obtain an access token for a specific subscriber.
* The Smart TV is an OAuth2 device-type client that can use the Device Code OAuth2 grant type flow to obtain the access token needed. After the Smart TV gets the access token, it presents the access token to the FEC application (the resource server), and requests to access a specific resource (a movie to stream).
* To decide whether access to the resource should be allowed or not, the FEC application validates the access token and checks if it contains the `streaming` scope.
* Only the scopes defined for, and requested by a specific client, can be added to the access token.
* You register an OAuth2 client profile, in AM, for the Smart TV OAuth2 client to communicate with AM as an OAuth2 Provider (authorization server) by using the Device Code grant type flow and the `streaming` scope.

An OAuth2 client can only request an access token from AM, provided that the client is registered and can successfully authenticate itself with AM.

Depending on the type of OAuth2 client, you create a client profile configured with the client credentials and scopes that it can request

Emulate the SmartTV being authorized to stream movies by using:
* Postman REST API requests to initiate and test the Device Code grant type flow.
* A web browser to enter the unique user code and authorize the SmartTV for the streaming scope.

## Lesson 2 - Integrating Applications With OIDC

### ODIC Overview

![](images/am401/am-oidc-2.png)

https://developer.okta.com/blog/2019/10/21/illustrated-guide-to-oauth-and-oidc - OAuth 2.0 is designed only for authorization, for granting access to data and features from one application to another. OpenID Connect (OIDC) is a thin layer that sits on top of OAuth 2.0 that adds login and profile information about the person who is logged in. Establishing a login session is often referred to as authentication, and information about the person logged in (i.e. the Resource Owner) is called identity. When an Authorization Server supports OIDC, it is sometimes called an identity provider, since it provides information about the Resource Owner back to the Client.

![](images/am401/am-oidc-3.png)

![](images/am401/am-oidc-4.png)

**OpenID Connect (OIDC)** is an identity layer built on top of the OAuth 2.0 framework. It allows third-party applications to verify the identity of the end-user and to obtain basic user profile information. OIDC uses **JSON web tokens (JWTs)**, which you can obtain using flows conforming to the OAuth 2.0 specifications.

While OAuth 2.0 is about resource access and sharing, OIDC is about user authentication. Its purpose is to give you one login for multiple sites. Each time you need to log in to a website using OIDC, you are redirected to your OpenID site where you log in, and then taken back to the website.

JWTs contain claims, which are statements (such as name or email address) about an entity (typically, the user) and additional metadata.

![](images/am401/am-oidc-1.png)

Full-featured applications must be available both on the backend and on the front end; for example, on native applications, in order to support personalization.

Personalization is key, and because personal data is usually kept with the server, a technology needs to be used to ensure the identity of the app user. After the user identity has been ensured, the relevant user personal data can be retrieved from the server and made available to the application.

The ID token is a JWT that is:
* The primary extension that OIDC makes to OAuth2 to enable end users to be authenticated.
* A security token that contains claims about the authentication of an end user, and potentially other requested claims.
* Signed, encoded, and optionally encrypted.
* Asserts the identity of the end user so that it can be used for authentication.

![](images/am401/am-oidc-5.png)

The ID token contains three parts separated by a period (.) character:
1. The first part is a base 64-encoded representation of the algorithm used to encode the signature.
2. The second part is a base 64-encoded representation of the token content itself. It holds information such as the issuer (iss), the user id (sub), the audience (aud), the expiry (exp), and the nonce value.
3. The third part is the signature, which is signed the authorization server.

The client must validate the signature to ensure that the expected authorization server emitted the token.

OIDC can provide information about the user through the use of scopes and claims.
* A claim corresponds to a user profile attribute.
* A scope is a set of claims of the same nature.

![](images/am401/am-oidc-6.png)

The scope is mapped to its claims, and the claims are mapped to their individual user attributes from the user profile. The scope and claim mappings are defined in an OIDC script.

AM currently supports 5 OIDC grant types.

![](images/am401/am-oidc-7.png)

At the start, the end user wants to modify a profile setting, such as a phone number in an application. The application needs the identity of the user to be asserted by the OIDC provider before they allow the user to sign in and change their profile settings.

![](images/am401/am-oidc-8.png)

AM provides an OIDC claims script, called `OIDC Claims Script`, that includes the profile scope and related claims in the ID token. This is written in Groovy and you can write them in JavaScript also.

![](images/am401/am-oidc-9.png)


## Lesson 3 - Authenticating OAuth2 Clients and using mTLS in OAuth2 for PoP

### 4 Authentication Modes

![](images/am401/am-mtls-1.png)

Only confidential clients are required to authenticate. Public clients, such as clients for native applications or SPAs, are not able to store credentials securely. This is why they are not required to authenticate. Instead, they should mitigate the security risk by using OAuth2 grants with PKCE.

There are four different methods to authenticate an OAuth2 client.
1. Form parameters
2. Authorization headers
3. JWT profiles
4. mTLS

![](images/am401/am-mtls-2.png)

When using form parameters to authenticate OAuth2 clients, a secret must be shared between the client and AM.

![](images/am401/am-mtls-3.png)

When using authorization headers to authenticate OAuth2 clients, a secret must be shared between the client and AM.

![](images/am401/am-mtls-4.png)

![](images/am401/am-mtls-5.png)

When authenticating an OAuth2 client using form parameters or authorization headers, the client secret is sent together with the request, which presents a security risk.

With the JWT profiles approach, the request does not contain a secret key. Instead, a JWT assertion is provided to AM when the client accesses an endpoint requiring authentication.

The JWT assertion contains information that identifies the client, and is signed. AM verifies the content of the assertion and the validity of its signature before sending a response.

The JWT token must be digitally signed and be a valid JWT. The JWT Client Assertions must be signed. AM supports JWT Client Assertions signed using a private key. The corresponding public key, needed to verify the signature, must be provided by the client and is added to the client profile in AM.
* As an X.509 certificate
* As a JWK set
* As a public URL containing keys
* As a JSON web key set

![](images/am401/am-mtls-6.png)

mTLS is an extension of the TLS handshake to authenticate clients.
* In a TLS handshake, the server provides a certificate to the client, such as a browser, and the browser validates the certificate.
* Mutual TLS is the process of how a client presents its X.509 certificate and proves possession of the corresponding private key to a server when negotiating a TLS session.

![](images/am401/am-mtls-7.png)

To utilize TLS for OAuth client authentication, the TLS connection between the client and the authorization server MUST have been established or reestablished with mTLS X.509 certificate authentication. You must configure the web container in which AM runs to use TLS connections, and to request and accept client certificates.

![](images/am401/am-mtls-8.png)

![](images/am401/am-mtls-9.png)

Clients can authenticate themselves with mTLS using either the PKI method or using a self-signed certificate. With the PKI method, the certificate contains a subject DN and a validated certificate chain to identify the client. The certificate is valid if the subject DN in the certificate matches the expected DN configured or registered for that particular client.

In the context of X.509 certificate spoofing:
* Problem: If the PKI method of client authentication is used, an attacker could try to impersonate a client using a certificate with the same subject DN but issued by a different CA, which the authorization server trusts.
* Solution: To cope with that threat, the authorization server should only accept, as trust anchors, a limited number of CAs whose certificate issuance policy meets its security requirements.

There is an assumption then that the client and server agree on the set of trust anchors that the server uses to create and validate the certificate chain. Without this assumption the use of a Subject DN to identify the client certificate would open the server up to certificate spoofing attacks.

# PoP & mTLS

mTLS certificate-bound access tokens ensure that only the party in possession of the private key corresponding to the certificate can utilize the token to access the associated resources. The process is sometimes referred to as key confirmation, **proof-of-possession (PoP)**, or holder-of-key.

Binding an access token to the client's certificate prevents the use of stolen access tokens or replay of access tokens by unauthorized parties. It also has the benefit of decoupling that binding from the client authentication with the authorization server, which enables mTLS during protected resource access to serve purely as a PoP mechanism.

To obtain a certificate-bound access token, OAuth2 clients authenticate with an authorization server using mTLS. Upon successful client authentication, the OAuth2 authorization server binds the access token to the client's TLS certificate.

![](images/am401/am-mtls-10.png)

* Ensure the environment enforces TLS:
  * Between client and AM
  * Between client and resource server
* Enable the TLS Certificate-Bound Access Tokens switch in the OAuth2 provider service.
* Enable the Use Certificate-Bound Access Tokens switch in the client profile.
* Configure the client for mTLS authentication.

![](images/am401/am-mtls-11.png)

The request to obtain a certificate-bound access token is identical to the request for a bearer access token using mTLS authentication for the client, which was described in the previous section. The difference is in the content of the access token returned.

For an mTLS client certificate-bound access token, the resource server compares that certificate hash to a hash of the client certificate used for mTLS authentication and rejects the request, if they do not match.

### Labs

In this exercise, you use curl to simulate an OAuth2 client requesting an access token with the client credentials OAuth2 grant. The authentication method chosen for client authentication is mTLS. To implement mTLS client authentication, configure the Tomcat server to request client certificates, create a client certificate, add it to the AM truststore, then create and configure a client profile in AM.

## Lesson 4 - Transforming OAuth2 Tokens

A security token is a set of data that supports sharing of identity and security information in heterogeneous environments or across security domains. Examples of security tokens include JWTs and SAML2 assertions.

A **security token service (STS)** is a service that can validate security tokens it receives, and can issue new security tokens in response. The new tokens can be used by clients to obtain appropriate access credentials for requested resources in heterogeneous environments or across security domains.

While the goal of both the STS and OAuth2 Token Exchange service implementations are to transform tokens, they differ completely in implementation and capabilities.

The STS service is used to establish cross-domain trust federation relationships. This is done through a REST STS framework loosely based on the SOAP WS-Trust specification that you need to build, deploy, and maintain yourself.

The REST STS service supports username/password, SSO tokens, X.509 certificates, and ID tokens as input tokens, and SAML2 assertions and ID tokens as output tokens. Due to its transformation capabilities, the STS service is more suited to helping federate legacy platforms.

AM configured as an OAuth2/OIDC authorization server uses the OAuth2 Token Exchange specification to transform OAuth2-related tokens. The AM OAuth2 Token Exchange is used only with OAuth2/OIDC platforms.

![](images/am401/am-token-exchange-1.png)

The graphic illustrates a client application, such as a mobile app, interacting with an application service (API1) in the cloud. The cloud application service communicates with several microservices (API2, API3). The microservices may interact with other microservices, such as AP2 communicating with API4.

Exchanging tokens requires a subject token, which is the original token to be exchanged. In the ForgeRock documentation and this lesson, the new token resulting from token exchange is referred to as an exchanged token.
* The subject token: A required token representing the identity of the party on behalf of whom the request is being made.
* The actor token: An optional token representing the identity of the acting party.

A client may want to exchange tokens for:
* Impersonation:
  * Used by a client to act as the subject on another client.
  * Has a subject token.
* Delegation:
  * Used by a subject to act on behalf of another subject.
  * Has a subject token and an actor token. The actor identity is stored in an act claim.

To impersonate means to pretend you are another person when performing a job or duty. Impersonation is a token exchange use case whereby a client performs an action on behalf of the user where there is no need to separate the user and the client.

To delegate means giving a job or duty to someone else to perform it on your behalf. Delegation is a token exchange whereby a client performs an action on behalf of the user when it is important that the user and client must be kept as different entities.

AM performs token exchange as if it were issuing a normal token of that type, with the following differences:
* Copies claims and values that must stay the same, from the subject token, in the new token.
* Derives scopes from the scope implementation used in the OAuth2/OIDC grant type flows.
* Adds the act and may_act claims.
* Adds an expiry time independent of the original token.

The `act` claim is part of the exchanged token that:
* Identifies the party acting on behalf of the token's subject.
* Expresses that delegation has occurred.
* Can be nested to have a chain of delegation.

The `may_act` claim is part of the original subject token that:
* Specifies who may act on behalf of the user.
* Acts as a condition for the authorization server issuing exchanged tokens where:
  * The client making the exchange must be authorized in the claim (impersonation or delegation).
  * The subject of the actor token must also be authorized in the claim of the subject token (delegation).

Exchanged tokens do not need the exact same scopes/claims that are in the subject token. Exchanged tokens can have their scopes/claims restricted or expanded.

![](images/am401/am-token-exchange-2.png)

1. Service A receives a subject (access) token from a client application.
2. Service A sends the subject token to AM for exchange.
3. AM responds with a new (exchanged) access token.
4. Service A sends a new access token to Service B.

Impersonation tokens cannot be told apart from normal tokens: no extra claims. Due to the risk of identity theft, implement token impersonation across trusted systems only.

![](images/am401/am-token-exchange-3.png)

1. The user opens the bank app.
2. The application requests authentication using an authorization service.
3. The user authenticates.
4. The bank app receives an access token with multiple scopes.
5. The app presents the banking page and services.
6. The user selects the transfer service.
7. The app requests an access token exchange impersonating the user with transfer scope.
8. The new impersonation access token is returned.
9. The app sends the new access token with the transfer scope request.

10 to 12 The transfer service processes the request and returns the result to the app, which displays the result information.

![](images/am401/am-token-exchange-4.png)

Here is the same sequence of steps shown for the impersonation with a restriction flow with the following differences:
* This example uses an ID token instead of an access token.
* After logging in, the user receives the subject token, which is an ID token that represents proof of their identity.
* When the user makes a transaction, the client exchanges the user's ID token with an impersonation access token with the transfer scope, and sends it to the transfer service.

The impersonation access token is an expanded token since it now has the `transfer` scope, which wasn't present in the ID token.

![](images/am401/am-token-exchange-5.png)

1. Service A receives a subject token and an actor token.
2. Service A sends both the subject token and actor token to AM.
3. AM responds with a new (exchanged) access token with the identity of the actor token attached as additional data.
4. Service A sends a new access token, with the identity of the actor token, to Service B. In this case, the actor token is acting on behalf of the subject token.

An example of the delegation pattern is:
* A user phones a call center because of a problem with their water supply.
* The operator (client) answers the call, verifies the identity of the user, and creates an ID token (the Subject token) on behalf of the user.
* The operator also creates an access token (the actor token) for themselves.
* The operator then exchanges both tokens for a delegated access token for booking a repair on behalf of the user. Both the operator (actor token) and the user (subject token) are reflected on the repair request.

The delegated token is a restricted token, since it can only be used for booking a repair on behalf of the user, and not for any other action, such as ending the user's contract.

![](images/am401/am-token-exchange-6.png)

The flow illustrates a user who phones a call center because their water supply has a problem. The operator (client) answers the call, verifies the identity of the user, and creates an ID token (the subject token) on behalf of the user. The operator also creates an access token (the actor token) for themselves.

Then, the operator exchanges both tokens for a delegated access token to book a repair for the user, where both the operator and the user are reflected on the repair request.

This delegated token flow is an example of a restricted token, because the token can only be used for booking a repair on behalf of the user, and not for any other action, such as ending the user's contract.

![](images/am401/am-token-exchange-7.png)

The scenario illustrated here is where both the subject and the actor tokens are ID tokens. To authorize a repair, the operator could request a delegated access token with the scope of `repair`.

In this case, the delegated token is an expanded token because it now has the `repair` scope, which neither of the ID tokens had before.

### Labs

## Lesson 5 - Implementing Social authentication

Optional.

### Social Access

In many contexts, it makes sense to simplify the user flow to improve the user's experience.

Website registration and login issues:
* Time consuming
* Forgotten login information
* Multiple accounts, e.g. password management and password reuse.

Social registration and login unifies and simplifies the process:
* Fewer accounts for users to manage
* Data sharing and authorization control with OAuth2
* Make both social and traditional login available.

Some users are reluctant to use social login as it comes with its own risks: lack of trust in the social media, what if the social account itself is compromised, and similar concerns. For that reason, most organization propose both social and traditional login to their users.

Social access has two components:
1. Social registration - basic social registration is available for AM, it belongs to the identity management sphere, and the Identity Platform, which integrates ForgeRock Identity Management (IDM), should be used for more complex solutions.
2. Social authentication - is an IAM task. It is implemented in AM by adding the social identity provider service to the realm, and defining the supported social identity providers

![](images/am401/am-social-1.png)

Using a third-party OIDC provider means that AM is now filling the role of the client or relying party in an OAuth2/OIDC flow. AM provides a set of social identity provider nodes that effectively transform AM into an OAuth2/OIDC client.

The flow then becomes fairly simple; the social identity provider nodes are included in a tree that is called when the user tries to log in.

![](images/am401/am-social-2.png)

To implement social registration and authentication in AM, you must add the social identity provider service to the realm and ensure it is enabled. You can then add all the social providers supported by your company.

If you support a social identity provider that is not part of the predefined list, find out if the flow used by the social identity provider is OAuth2 or OIDC and use the corresponding generic choice in the drop-down menu.

![](images/am401/am-social-3.png)

To make a social identity provider available to your solution, you must first access the provider site and register a client representing your application. Check the social identity provider documentation to find out how to do that. At the end of the process you should have obtained client credentials, and defined the various elements for the OAuth2/OIDC flow to be successful.

![](images/am401/am-social-4.png)

To delegate social registration and social authentication to a social identity provider, the authentication tree can be modified to include a Select Identity Provider node, a Social Provider Handler Node, and a Provision Dynamic Account node.

### Labs