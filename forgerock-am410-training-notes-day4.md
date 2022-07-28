# AM410 Day 4 - Federating Across Entities Using SAML2

- [AM410 Day 4 - Federating Across Entities Using SAML2](#am410-day-4---federating-across-entities-using-saml2)
  - [Lesson 1 - Implementing SSO Using SAML2](#lesson-1---implementing-sso-using-saml2)
    - [Okta & Duo SAML Explanation](#okta--duo-saml-explanation)
    - [SAML In AM](#saml-in-am)
    - [Labs](#labs)
      - [Set Up Federation](#set-up-federation)
      - [Federation](#federation)
  - [Lesson 2 - Delegating Authentication Using SAML2](#lesson-2---delegating-authentication-using-saml2)
    - [AM As A SAML2 SP](#am-as-a-saml2-sp)
    - [SAML2 Metadata](#saml2-metadata)
    - [Labs](#labs-1)

## Lesson 1 - Implementing SSO Using SAML2

### Okta & Duo SAML Explanation

https://support.okta.com/help/s/article/Beginners-Guide-to-SAML?language=en_US

https://duo.com/blog/the-beer-drinkers-guide-to-saml

**SAML (Security Assertion Markup Language)** is an XML-based standard for exchanging authentication and authorization data between an identity provider (IdP) such as Okta, and a service provider (SP) such as Box, Salesforce, G Suite, Workday, etc, allowing for a **Single Sign-On (SSO)** experience.
* **Identity Provider (IdP)** is the software tool or service that performs the authentication; checking usernames and passwords, verifying account status, invoking two-factor, etc. This is often visualized by a login page and/or dashboard.
* **Service Provider (SP)** is the web application where user is trying to gain access.
* **SAML Assertion** is a message asserting a user’s identity and often other attributes, sent over HTTP via browser redirects.
* **Federating identities** is a common practice where user identities are stored across disparate applications and organizations. SAML allows these federated apps and organizations to communicate and trust one another’s users.

SAML provides a way to authenticate users to SP provided third-party web apps (e.g. Gmail for Business, Office 365, etc.) by redirecting the user’s browser to a company login page. Then after successful authentication on that login page, redirecting the user’s browser back to that third-party web app where they are granted access. **The key to SAML is browser redirects!**

SAML is the common underlying protocol that makes web-based SSO possible. A company maintains a single login page, single or multiple identity stores, and various authentication rules. Then they can easily configure any web app that supports SAML to allow their users to log in all web apps from the same login screen with a single password. It also has the security benefit of neither forcing users to maintain and potentially reuse passwords for every web app they need access to, nor exposing passwords to those web apps.

![](images/am401/saml-1.png)

1. Bob walks to the Wristband Tent, where his ID is checked and a wristband is provided.
   1. The Wristband Tent is the IdP, it verifies Bob's identity to make sure he meets the criteria to get a wristband.
2. Bob walks over the Beer Tent and uses his wirstband to get a beer.
   1. The Beer Teen is the SP, it provides the service that Bob wants.

There are two different sign-in flows for which authentication can be handled by SAML:
1. SP-initiated flow.
2. IdP-initiated flow.

![](images/am401/saml-2.png)

SP-initiated flow occurs when the user first navigates to the SP, getting redirected to the IdP with a SAML request, and then redirected back to the SP with a SAML assertion.

![](images/am401/saml-3.png)

IdP-initiated flow occurs when the user first navigates to the IdP (typically a login page or dashboard), and then going to the SP with a SAML assertion.


**Note:** **SAML Tracer**, a free add-on available for Firefox, is an extremely useful tool in helping understand and troubleshoot SAML assertions. It allows you to extrapolate SAML assertions from your browsing session and examine them line-by-line.

### SAML In AM

* Standard for exchanging data between organizations and across security domains.
* XML-based protocol.
* Uses assertions to pass information:
  * About a principal (usually an end user)
  * From an Identity Provider (holds the identities, manages authentication)
  * To a Service Provider (protects and provides a service)
* AM can be configured as an IdP or an SP.

SPs and IdPs participating in a federation need a common understanding about:
* **Circle of Trust (CoT)**: The entities that are part of the system.
* Metadata: Configuration parameters of the participating entities.
* SAML2 protocol objects:
  * Requests, responses, and assertions
  * Facilitate communication at runtime

![](images/am401/am-saml-1.png)

The CoT is a group of federated entities, normally the federation partners, who have come together to form a trusted federation. Entities in the federation share security and connection information with each other, and create a trust relationship by which they can exchange and utilize federated identity information between them.

![](images/am401/am-saml-2.png)

SP-initiated means that the federation is initiated and controlled by the SP.

1. User tries to access a service.
2. SP sends a SAML2 authentication request.
3. IdP processes the request and authenticates the user.
4. IdP creates an assertion.
5. IdP sends a SAML2 response containing an assertion to the SP.
6. SP processes the response.
7. SP proceeds with the flow.

![](images/am401/am-saml-3.png)

**Artifact binding** defines the way the assertion created by the IdP is transmitted to the SP. It ensures that the assertion is never exposed to the client; the client is given an artifact (a unique semi-random number) that can be used by the SP to retrieve the assertion from the IdP. The artifact binding does require that the SP can reach the IdP directly. It uses Simple Object Access Protocol (SOAP) to communicate with the IdP.

![](images/am401/am-saml-4.png)

![](images/am401/am-saml-5.png)

IdP-initiated means that the federation is initiated and controlled by the IdP.

![](images/am401/am-saml-6.png)

The process of associating the IdP and SP accounts correctly is called **linking accounts.** To be able to perform account linking, something must be present in the assertion in order to link the authenticated user with a local account on the SP. That can take two forms:
1. Common key, a common key is used to link the accounts.
   1. Must be established before federation can take place:
      1. Out-of-band (Microsoft requirement for Azure integration).
      2. Established during first access.
      3. Requires user to log in to both IdP and SP.
2. Common attribute, a common attribute is used to link the accounts, e.g. an email address.
   1. Requires user to log in to IdP only.
   2. Called auto-federation.

![](images/am401/am-saml-7.png)

* AM as an IdP is called the hosted IdP.
  * Monitoring the SSO service endpoint.
  * Validating authentication request.
  * Authenticating end user.
  * Creating an assertion.
  * Sending the response.
* Third-party SP is called remote SP.

![](images/am401/am-saml-8.png)

![](images/am401/am-saml-9.png)

![](images/am401/am-saml-10.png)

![](images/am401/am-saml-11.png)

The IdP can return profile attributes of the principal. It checks its configuration to find out which attributes need to be returned. That process is governed by the IdP attribute mapper.

![](images/am401/am-saml-12.png)

Trust and SSO are two separate concepts. In SAML2, trust is established between an SP and an IdP. This trust relationship allows things such as data sharing. There is no trust relationship across SPs, but there is SSO functionality for all of them.

![](images/am401/am-saml-13.png)

Entities within the same realm are able to use SSO even if they belong to separate CoTs.

### Labs

Two applications are installed on the CloudShare CentOS VM. They represent Software as a service (SaaS) services used by a company, let's call it Infocrew, for daily operations. In this exercise, you will observe how AM as an IdP can integrate with the two SAML2-compliant service providers.

#### Set Up Federation

1. Configure AM as a hosted IdP
   1. Create a hosted IdP in AM
2. Configure remote SPs in AM
   1. Import the SP metadata into AM.
3. Create a Circle of Trust (CoT) which contains the IdP and SPs, so that they are able to communicate with each other in a SAML2 context.
4. Integrate AM metadata in the third-party SAML2 entities environments
   1. Download the AM metadata and provide to the SPs.

#### Federation

When a user federates an account for the first time, they need to create a link between the account on the IdP and the account on the SP. In our example, FakeCRM has no reason to know that user.40 is the same person as their user called a.adamski.

There are different ways to address this question. The solution adopted in this exercise is a two-step process:
1. Initial federation access: The user enters their credentials in the IdP, is redirected to the SP, and then enters their local credentials for the SP. After this step, the accounts are linked and both SP and IdP will be able to process the next step. SP sends a `samlp:AuthnRequest` to IdP
2. Following federation accesses: The user only needs to enter credentials in the IdP, and the SP will know the user. IdP sends a `samlp:Response` containing the SAML assertion to SP.

Request

```xml
<samlp:AuthnRequest
	xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="s2b95037fbadda642f4a995a0d00a51a81022b4e39" Version="2.0" IssueInstant="2022-07-28T01:37:17Z" Destination="https://am.example.com:9443/login/SSORedirect/metaAlias/bravo/idp" ForceAuthn="false" IsPassive="false" ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" AssertionConsumerServiceURL="https://fakecrm.example.org:10443/sp1/AuthConsumer/metaAlias/sp">
	<saml:Issuer
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">https://fakecrm.example.org:10443/sp1
	</saml:Issuer>
	<samlp:NameIDPolicy
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent" SPNameQualifier="https://fakecrm.example.org:10443/sp1" AllowCreate="true">
	</samlp:NameIDPolicy>
	<samlp:RequestedAuthnContext
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Comparison="minimum">
		<saml:AuthnContextClassRef
			xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
		</saml:AuthnContextClassRef>
	</samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

![](images/am401/am-saml-14.png)

![](images/am401/am-saml-15.png)

Response

```xml
<samlp:AuthnRequest
	xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="s2b95037fbadda642f4a995a0d00a51a81022b4e39" Version="2.0" IssueInstant="2022-07-28T01:37:17Z" Destination="https://am.example.com:9443/login/SSORedirect/metaAlias/bravo/idp" ForceAuthn="false" IsPassive="false" ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" AssertionConsumerServiceURL="https://fakecrm.example.org:10443/sp1/AuthConsumer/metaAlias/sp">
	<saml:Issuer
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">https://fakecrm.example.org:10443/sp1
	</saml:Issuer>
	<samlp:NameIDPolicy
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent" SPNameQualifier="https://fakecrm.example.org:10443/sp1" AllowCreate="true">
	</samlp:NameIDPolicy>
	<samlp:RequestedAuthnContext
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Comparison="minimum">
		<saml:AuthnContextClassRef
			xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
		</saml:AuthnContextClassRef>
	</samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
<samlp:Response
	xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="s29220908c78d8691694fdbcddc7d1d35a516ed6a6" InResponseTo="s2b95037fbadda642f4a995a0d00a51a81022b4e39" Version="2.0" IssueInstant="2022-07-28T01:37:31Z" Destination="https://fakecrm.example.org:10443/sp1/AuthConsumer/metaAlias/sp">
	<saml:Issuer
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">https://am.example.com:9443/login
	</saml:Issuer>
	<samlp:Status
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
		<samlp:StatusCode
			xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Value="urn:oasis:names:tc:SAML:2.0:status:Success">
		</samlp:StatusCode>
	</samlp:Status>
	<saml:Assertion
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="s29829c867c4e2b81118dd0fc0c780f46dc4b8dadd" IssueInstant="2022-07-28T01:37:31Z" Version="2.0">
		<saml:Issuer>https://am.example.com:9443/login</saml:Issuer>
		<ds:Signature
			xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
			<ds:SignedInfo>
				<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></ds:CanonicalizationMethod>
				<ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"></ds:SignatureMethod>
				<ds:Reference URI="#s29829c867c4e2b81118dd0fc0c780f46dc4b8dadd">
					<ds:Transforms>
						<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"></ds:Transform>
						<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></ds:Transform>
					</ds:Transforms>
					<ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"></ds:DigestMethod>
					<ds:DigestValue>ia1SxYbQkphIzfpCJJpOH5mSUeFQWF5DNbrv+2MKeQ4=</ds:DigestValue>
				</ds:Reference>
			</ds:SignedInfo>
			<ds:SignatureValue>lLqJJ3CWhysHFAkA3QsrwTl1VvrHO810Sk9Ny73aQwOQ89/at2WxD6ENfoH/AtIU9sgRM+/BLefAlrtys5M4AG0fO0g5EQ0gYXWtTVXv7fgMq2EVmAL4z1GhafmYi067mZAV/Hrh9fY/yIilMFyHpBfuOW/5OqCMmSDHhae9qq0t/CzpmR1sPgyGyMHomFXXLtrZeycxBh7i9r50HPb8hzd3jgxONTSlHNDjDl6kTwsb+1YooAHDTIaaFOOarFHZkynap/NFfelXuHr4KA7S7Te/rpEzd4GTKjm8vRsKUwQa7xAXsaqYNFrjJoI7dRurzUzaTXve7jC2wqe7eNN85A==</ds:SignatureValue>
			<ds:KeyInfo>
				<ds:X509Data>
					<ds:X509Certificate>MIIDdzCCAl+gAwIBAgIES3eb+zANBgkqhkiG9w0BAQsFADBsMRAwDgYDVQQGEwdVbmtub3duMRAwDgYDVQQIEwdVbmtub3duMRAwDgYDVQQHEwdVbmtub3duMRAwDgYDVQQKEwdVbmtub3duMRAwDgYDVQQLEwdVbmtub3duMRAwDgYDVQQDEwdVbmtub3duMB4XDTE2MDUyNDEzNDEzN1oXDTI2MDUyMjEzNDEzN1owbDEQMA4GA1UEBhMHVW5rbm93bjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5rbm93bjEQMA4GA1UEChMHVW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEQMA4GA1UEAxMHVW5rbm93bjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANdIhkOZeSHagT9ZecG+QQwWaUsi7OMv1JvpBr/7HtAZEZMDGWrxg/zao6vMd/nyjSOOZ1OxOwjgIfII5+iwl37oOexEH4tIDoCoToVXC5iqiBFz5qnmoLzJ3bF1iMupPFjz8Ac0pDeTwyygVyhv19QcFbzhPdu+p68epSatwoDW5ohIoaLzbf+oOaQsYkmqyJNrmht091XuoVCazNFt+UJqqzTPay95Wj4F7Qrs+LCSTd6xp0Kv9uWG1GsFvS9TE1W6isVosjeVm16FlIPLaNQ4aEJ18w8piDIRWuOTUy4cbXR/Qg6a11l1gWls6PJiBXrOciOACVuGUoNTzztlCUkCAwEAAaMhMB8wHQYDVR0OBBYEFMm4/1hF4WEPYS5gMXRmmH0gs6XjMA0GCSqGSIb3DQEBCwUAA4IBAQDVH/Md9lCQWxbSbie5lPdPLB72F4831glHlaqms7kzAM6IhRjXmd0QTYq3Ey1J88KSDf8A0HUZefhudnFaHmtxFv0SF5VdMUY14bJ9UsxJ5f4oP4CVh57fHK0w+EaKGGIw6TQEkL5L/+5QZZAywKgPz67A3o+uk45aKpF3GaNWjGRWEPqcGkyQ0sIC2o7FUTV+MV1KHDRuBgreRCEpqMoY5XGXe/IJc1EJLFDnsjIOQU1rrUzfM+WP/DigEQTPpkKWHJpouP+LLrGRj2ziYVbBDveP8KtHvLFsnexA/TidjOOxChKSLT9LYFyQqsvUyCagBb4aLs009kbW6inN8zA6</ds:X509Certificate>
				</ds:X509Data>
			</ds:KeyInfo>
		</ds:Signature>
		<saml:Subject>
			<saml:NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent" NameQualifier="https://am.example.com:9443/login" SPNameQualifier="https://fakecrm.example.org:10443/sp1">xrlMTAYdoBqeydNcu0/50a5Esz6f</saml:NameID>
			<saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
				<saml:SubjectConfirmationData InResponseTo="s2b95037fbadda642f4a995a0d00a51a81022b4e39" NotOnOrAfter="2022-07-28T01:47:31Z" Recipient="https://fakecrm.example.org:10443/sp1/AuthConsumer/metaAlias/sp"></saml:SubjectConfirmationData>
			</saml:SubjectConfirmation>
		</saml:Subject>
		<saml:Conditions NotBefore="2022-07-28T01:27:31Z" NotOnOrAfter="2022-07-28T01:47:31Z">
			<saml:AudienceRestriction>
				<saml:Audience>https://fakecrm.example.org:10443/sp1</saml:Audience>
			</saml:AudienceRestriction>
		</saml:Conditions>
		<saml:AuthnStatement AuthnInstant="2022-07-28T01:37:31Z" SessionIndex="s256df4f69d90609e9f2963b3a8447c98b00bbaf01">
			<saml:AuthnContext>
				<saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml:AuthnContextClassRef>
			</saml:AuthnContext>
		</saml:AuthnStatement>
	</saml:Assertion>
</samlp:Response>
```

![](images/am401/am-saml-16.png)

![](images/am401/am-saml-17.png)

![](images/am401/am-saml-18.png)

So far, you have observed how users can get access to a service automatically, simply because they are authenticated to the IdP. If you add a new service to the CoT and set up federation between the IdP and that new SP, you will be able to achieve SSO across SP1 and SP2. So when you log into the IdP and have an active session all subsequent logins to the SP don't require you to login.

## Lesson 2 - Delegating Authentication Using SAML2

### AM As A SAML2 SP

If AM is the SP then it is called a hosted SP. The third-party IdPs are called remote IdPs.

![](images/am401/am-saml-19.png)

An can SP can start a federation process. Typically, federation will start with a user trying to access a web page or a service using a browser.
* End user wants to access a resource.
* End user selects a login link that starts SAML2 flow in:
  * Integrated mode: Uses a SAML2 authentication node in a tree.
  * Standalone mode: Uses a JavaScript file with parameters.

Integrated mode:
* Integrates SAML2 authentication into the normal AM authentication process.
* Uses the SAML2 authentication node in a tree:
  * The SAML2 authentication node acts as the SP and handles the SAML2 protocol details.
  * Does not support SLO with trees.
* Only supports SP-initiated SSO; you cannot trigger IdP-initiated SSO.

Standalone mode:
* Provides specific JavaScript pages to initiate SSO and SLO:
  * `spSSOInit.jsp` and `spSingleLogoutInit.jsp`
  * Uses parameters to indicate the entities taking part and potentially overwrite defaults. e.g. `https://am.example.com/login/saml2/jsp/spSSOInit.jsp?idpEntityID=https%3A%2F%2Fgovidp.example.net%2Fidp&metaAlias=/sp`
* Supports SP-initiated and IdP-initiated SSO and SLO.

![](images/am401/am-saml-20.png)

Once the SAML2 request has been sent, the SP must wait until it receives a response from the IdP. That response must be sent to the assertion consumer service.

To validate the response, the SP must compare the content of the response with information it holds in its configuration or cache. It must validate the response itself, and then the assertion contained in the response.

![](images/am401/am-saml-21.png)

The SP retrieves the information held in the assertion and links the principal with a local user. It is possible to federate users who do not have an account with the SP specifically. In such cases, you could link the user to an anonymous account on the SP. e.g. uni library allowing students access to their website using their student account, yet the website doesn't care who the student is.

To link the principal to a local account, the SP verifies the `NameID` format. If it is persistent, it retrieves the user associated with the provided key. If no such key exists, it means it is a first access for the user, and the user is asked to authenticate locally. If attributes are used, the SP will try to find the user with the corresponding attribute.

![](images/am401/am-saml-22.png)

What happens once the account is linked locally will depend on the business requirements and your implementation. In AM, a local session must always be created. Once the local user is retrieved, AM as an SP creates a local session and injects all the attributes from the assertion in the session.

### SAML2 Metadata

`https://am.example.com:9443/login/saml2/jsp/exportmetadata.jsp?&realm=/bravo&entityid=https://am.example.com:9443/login`

Federation between two entities must be negotiated and configured before federation can take place. OASIS standard defines SAML2 metadata, an XML document that contains configuration data. It lets a third-party set up federation automatically with no need for negotiation. An entity can publish their metadata document, and any entity wishing to federate with them has enough information to do so automatically. If the basic information is not enough, or if specific behavior is expected, some negotiation may be needed.

Metadata includes information about the supported identifiers, binding supported service endpoints, certificates, and keys, as well as cryptographic capabilities and security and privacy policies.

Metadata can be shared informally (e.g. email) and does not have to follow the standard SAML2 metadata format. In such cases, setting up the federation would be a manual process.

![](images/am401/am-saml-23.png)

This is an example of SP metadata. It contains blocks defining how this specific entity will federate with other entities. It contains information such as:
* `EntityDescriptor`: Defines the entity ID which corresponds to the outside world name.
* `SPSSODescriptor`: Shows that this metadata belongs to an SP.
* Service endpoints: Information, such as where assertions should be sent to.
* `NameID` format: Defines which identifiers are supported and can be sent in the response.

![](images/am401/am-saml-24.png)

This is an example of IdP metadata. It contains blocks defining how this specific entity will federate with other entities. It contains information, such as:
* `EntityDescriptor`: Defines the entity ID which corresponds to the "outside world" name.
* Signing and encryption information.
* `IDPSSODescriptor`: Shows that this metadata belongs to an IdP.
* Service endpoints: Information, such as where SAML2 authentication requests should be sent to.
* `NameID` format: Defines which identifiers are supported and can be sent in the response.

 Metadata key building blocks
* `EntityDescriptor`: Describes system entity such as IdP or SP: `<EntityDescriptor entityID="https://am.example.com/login">`
* `IDPSSODescriptor` or `SPSSODescriptor`: Determines the role of the entity.
* `KeyDescriptor`: Defines information related to security, such as signing and encryption algorithms or certificates.
* `NameIDFormat`: Defines the supported ways to provide a `NameID` (for an IdP) and to understand NameID (for an SP).

One difference between IdP and SP metadata is the descriptor block that defines the role of the entity. Another difference is within the list of services endpoints that must be provided. **Service endpoints** describe where SAML2 protocol objects, such as SAML2 requests, SAML2 responses, and SAML2 assertions, should be sent. Another difference between metadata for SPs and IdPs is the service endpoints that can be defined. \

![](images/am401/am-saml-25.png)

AM is able to automatically upload SAML2 metadata XML files and generate the corresponding configuration in the Admin UI.

Standard metadata only provides minimal information about the federation. Anything more complex must be configured manually in the AM Admin UI. For example, attribute mapping or auto-federation settings are not part of the standard metadata.

The meta alias uniquely identifies entities participating in a SAML2 federation within an AM deployment:
* Must be unique across the entire AM deployment.
* Contains the subrealm in the name. For example, `/bravo/idp`.
* Used extensively within AM. For example, SSO service endpoint may look like: `https://am.example.com/login/SSORedirect/metaAlias/bravo/idp`

The Entity ID is a unique identifier for the outside world:
* Often uses URL. For example, Entity ID may be: `https://am.example.com/login.`

### Labs

In this exercise, you will use AM as a SAML2 SP, by delegating authentication to a third-party SAML2 IdP called Govidp

A default SP (or IdP) contains all the configuration required to perform a successful federation flow. In this example, however, you add a non-default behavior, using a feature called auto-federation. When an account exists in both the SP and the IdP, auto-federation is a way to automatically link a user by matching the content of the assertion and the associated value for the user in the SP. In this exercise, we use the email attribute of a user. If the user email does not exist in the SP, they will log in to their account in the SP as shown in the previous lesson.

You need to make AM aware of which entities it can work with. To do so, you will use the third-party IdP metadata, as it contains all the information needed to set up a basic, default federation flow.



* This XML file does not appear to have any style information associated with it. The document tree is shown below. It is the metadata file.

```xml
<EntityDescriptor
	xmlns="urn:oasis:names:tc:SAML:2.0:metadata"
	xmlns:query="urn:oasis:names:tc:SAML:metadata:ext:query"
	xmlns:mdattr="urn:oasis:names:tc:SAML:metadata:attribute"
	xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion"
	xmlns:xenc="http://www.w3.org/2001/04/xmlenc#"
	xmlns:xenc11="http://www.w3.org/2009/xmlenc11#"
	xmlns:alg="urn:oasis:names:tc:SAML:metadata:algsupport"
	xmlns:x509qry="urn:oasis:names:tc:SAML:metadata:X509:query"
	xmlns:ds="http://www.w3.org/2000/09/xmldsig#" entityID="https://govidp.example.net:10443/idp">
	<IDPSSODescriptor protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol">
		<KeyDescriptor use="signing">
			<ds:KeyInfo>
				<ds:X509Data>
					<ds:X509Certificate> MIIDdzCCAl+gAwIBAgIES3eb+zANBgkqhkiG9w0BAQsFADBsMRAwDgYDVQQGEwdVbmtub3duMRAw DgYDVQQIEwdVbmtub3duMRAwDgYDVQQHEwdVbmtub3duMRAwDgYDVQQKEwdVbmtub3duMRAwDgYD VQQLEwdVbmtub3duMRAwDgYDVQQDEwdVbmtub3duMB4XDTE2MDUyNDEzNDEzN1oXDTI2MDUyMjEz NDEzN1owbDEQMA4GA1UEBhMHVW5rbm93bjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5r bm93bjEQMA4GA1UEChMHVW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEQMA4GA1UEAxMHVW5rbm93 bjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANdIhkOZeSHagT9ZecG+QQwWaUsi7OMv 1JvpBr/7HtAZEZMDGWrxg/zao6vMd/nyjSOOZ1OxOwjgIfII5+iwl37oOexEH4tIDoCoToVXC5iq iBFz5qnmoLzJ3bF1iMupPFjz8Ac0pDeTwyygVyhv19QcFbzhPdu+p68epSatwoDW5ohIoaLzbf+o OaQsYkmqyJNrmht091XuoVCazNFt+UJqqzTPay95Wj4F7Qrs+LCSTd6xp0Kv9uWG1GsFvS9TE1W6 isVosjeVm16FlIPLaNQ4aEJ18w8piDIRWuOTUy4cbXR/Qg6a11l1gWls6PJiBXrOciOACVuGUoNT zztlCUkCAwEAAaMhMB8wHQYDVR0OBBYEFMm4/1hF4WEPYS5gMXRmmH0gs6XjMA0GCSqGSIb3DQEB CwUAA4IBAQDVH/Md9lCQWxbSbie5lPdPLB72F4831glHlaqms7kzAM6IhRjXmd0QTYq3Ey1J88KS Df8A0HUZefhudnFaHmtxFv0SF5VdMUY14bJ9UsxJ5f4oP4CVh57fHK0w+EaKGGIw6TQEkL5L/+5Q ZZAywKgPz67A3o+uk45aKpF3GaNWjGRWEPqcGkyQ0sIC2o7FUTV+MV1KHDRuBgreRCEpqMoY5XGX e/IJc1EJLFDnsjIOQU1rrUzfM+WP/DigEQTPpkKWHJpouP+LLrGRj2ziYVbBDveP8KtHvLFsnexA /TidjOOxChKSLT9LYFyQqsvUyCagBb4aLs009kbW6inN8zA6 </ds:X509Certificate>
				</ds:X509Data>
			</ds:KeyInfo>
		</KeyDescriptor>
		<KeyDescriptor use="encryption">
			<ds:KeyInfo>
				<ds:X509Data>
					<ds:X509Certificate> MIIDYTCCAkmgAwIBAgIEFt4OQjANBgkqhkiG9w0BAQsFADBhMQswCQYDVQQGEwJVSzEQMA4GA1UE CBMHQnJpc3RvbDEQMA4GA1UEBxMHQnJpc3RvbDESMBAGA1UEChMJRm9yZ2VSb2NrMQswCQYDVQQL EwJBTTENMAsGA1UEAxMEdGVzdDAeFw0xODA0MDMxNDIwNThaFw0yODAzMzExNDIwNThaMGExCzAJ BgNVBAYTAlVLMRAwDgYDVQQIEwdCcmlzdG9sMRAwDgYDVQQHEwdCcmlzdG9sMRIwEAYDVQQKEwlG b3JnZVJvY2sxCzAJBgNVBAsTAkFNMQ0wCwYDVQQDEwR0ZXN0MIIBIjANBgkqhkiG9w0BAQEFAAOC AQ8AMIIBCgKCAQEAi7t6m4d/02dZ8dOe+DFcuUYiOWueHlNkFwdUfOs06eUETOV6Y9WCXu3D71db F0Fhou69ez5c3HAZrSVS2qC1Htw9NkVlLDeED7qwQQMmSr7RFYNQ6BYekAtn/ScFHpq8Tx4BzhcD b6P0+PHCo+bkQedxwhbMD412KSM2UAVQaZ+TW+ngdaaVEs1Cgl4b8xxZ9ZuApXZfpddNdgvjBeeY QbZnaqU3b0P5YE0s0YvIQqYmTjxh4RyLfkt6s/BS1obWUOC+0ChRWlpWE7QTEVEWJP5yt8hgZ5Me cTmBi3yZ/0ts3NsL83413NdbWYh+ChtP696mZbJozflF8jR9pewTbQIDAQABoyEwHzAdBgNVHQ4E FgQUDAvAglxsoXuEwI2NT1hFtVww2SUwDQYJKoZIhvcNAQELBQADggEBADiHqUwRlq1xdHP7S387 vMLOr+/OUgNvDUogeyrpdj5vFve/CBxSFlcoY215eE0xzj2+bQoe5To3s8CWkP9hqB3EdhaRBfCr d8Vpvu8xBZcxQzmqwNjmeDrxNpKes717t05fDGgygUM8xIBs29JwRzHzf7e0ByJjn9fvlUjDAGZ7 emCTN382F2iOeLC2ibVl7dpmsWZTINhQRbmq5L4ztOcjITk5WZnBF439oRRn68fWZVkOv2UqaKbk uMjgotNuot+ebHtOchEiwKz8VAK7O3/IgD6rfNBfz+c/WeoPcrfQBR4zfizw/ioR115RSywifzlw q5yziqyU04eP4wLr3cM= </ds:X509Certificate>
				</ds:X509Data>
			</ds:KeyInfo>
			<EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#rsa-oaep-mgf1p">
				<ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"/>
			</EncryptionMethod>
			<EncryptionMethod Algorithm="http://www.w3.org/2001/04/xmlenc#aes128-cbc">
				<xenc:KeySize>128</xenc:KeySize>
			</EncryptionMethod>
		</KeyDescriptor>
		<ArtifactResolutionService index="0" Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/ArtifactResolver/metaAlias/idp"/>
		<SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://govidp.example.net:10443/idp/IDPSloRedirect/metaAlias/idp" ResponseLocation="https://govidp.example.net:10443/idp/IDPSloRedirect/metaAlias/idp"/>
		<SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://govidp.example.net:10443/idp/IDPSloPOST/metaAlias/idp" ResponseLocation="https://govidp.example.net:10443/idp/IDPSloPOST/metaAlias/idp"/>
		<SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/IDPSloSoap/metaAlias/idp"/>
		<ManageNameIDService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://govidp.example.net:10443/idp/IDPMniRedirect/metaAlias/idp" ResponseLocation="https://govidp.example.net:10443/idp/IDPMniRedirect/metaAlias/idp"/>
		<ManageNameIDService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://govidp.example.net:10443/idp/IDPMniPOST/metaAlias/idp" ResponseLocation="https://govidp.example.net:10443/idp/IDPMniPOST/metaAlias/idp"/>
		<ManageNameIDService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/IDPMniSoap/metaAlias/idp"/>
		<NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:persistent</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:WindowsDomainQualifiedName</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:kerberos</NameIDFormat>
		<NameIDFormat>urn:oasis:names:tc:SAML:1.1:nameid-format:X509SubjectName</NameIDFormat>
		<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="https://govidp.example.net:10443/idp/SSORedirect/metaAlias/idp"/>
		<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="https://govidp.example.net:10443/idp/SSOPOST/metaAlias/idp"/>
		<SingleSignOnService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/SSOSoap/metaAlias/idp"/>
		<NameIDMappingService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/NIMSoap/metaAlias/idp"/>
		<AssertionIDRequestService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="https://govidp.example.net:10443/idp/AIDReqSoap/IDPRole/metaAlias/idp"/>
		<AssertionIDRequestService Binding="urn:oasis:names:tc:SAML:2.0:bindings:URI" Location="https://govidp.example.net:10443/idp/AIDReqUri/IDPRole/metaAlias/idp"/>
	</IDPSSODescriptor>
</EntityDescriptor>
```
![](images/am401/am-saml-26.png)

![](images/am401/am-saml-27.png)

AM lets you create a remote entity either by uploading a metadata XML file, or by making a REST call and providing the metadata in JSON format.

This is the SAML authentication request from AM as SP to remote IdP.

```xml
<samlp:AuthnRequest
	xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="s27845c3b947bab365dd2972df4ac2794a4e0f0a65" Version="2.0" IssueInstant="2022-07-28T04:25:55Z" Destination="https://govidp.example.net:10443/idp/SSORedirect/metaAlias/idp" ForceAuthn="false" IsPassive="false" ProtocolBinding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" AssertionConsumerServiceURL="https://am.example.com:9443/login/AuthConsumer/metaAlias/alpha/sp">
	<saml:Issuer
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">https://am.example.com:9443/login
	</saml:Issuer>
	<samlp:NameIDPolicy
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent" SPNameQualifier="https://am.example.com:9443/login" AllowCreate="true">
	</samlp:NameIDPolicy>
	<samlp:RequestedAuthnContext
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Comparison="minimum">
		<saml:AuthnContextClassRef
			xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport
		</saml:AuthnContextClassRef>
	</samlp:RequestedAuthnContext>
</samlp:AuthnRequest>
```

This is the SAML authentication response from remote IdP to AM as SP.

```xml
<samlp:Response
	xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" ID="s27802805d49eb7f633514947e78f5dc097eb61a58" InResponseTo="s27845c3b947bab365dd2972df4ac2794a4e0f0a65" Version="2.0" IssueInstant="2022-07-28T04:25:59Z" Destination="https://am.example.com:9443/login/AuthConsumer/metaAlias/alpha/sp">
	<saml:Issuer
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion">https://govidp.example.net:10443/idp
	</saml:Issuer>
	<samlp:Status
		xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol">
		<samlp:StatusCode
			xmlns:samlp="urn:oasis:names:tc:SAML:2.0:protocol" Value="urn:oasis:names:tc:SAML:2.0:status:Success">
		</samlp:StatusCode>
	</samlp:Status>
	<saml:Assertion
		xmlns:saml="urn:oasis:names:tc:SAML:2.0:assertion" ID="s2c10ad8a9ddadfc27aad69ec646670c3b31b85bc4" IssueInstant="2022-07-28T04:25:59Z" Version="2.0">
		<saml:Issuer>https://govidp.example.net:10443/idp</saml:Issuer>
		<ds:Signature
			xmlns:ds="http://www.w3.org/2000/09/xmldsig#">
			<ds:SignedInfo>
				<ds:CanonicalizationMethod Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></ds:CanonicalizationMethod>
				<ds:SignatureMethod Algorithm="http://www.w3.org/2001/04/xmldsig-more#rsa-sha256"></ds:SignatureMethod>
				<ds:Reference URI="#s2c10ad8a9ddadfc27aad69ec646670c3b31b85bc4">
					<ds:Transforms>
						<ds:Transform Algorithm="http://www.w3.org/2000/09/xmldsig#enveloped-signature"></ds:Transform>
						<ds:Transform Algorithm="http://www.w3.org/2001/10/xml-exc-c14n#"></ds:Transform>
					</ds:Transforms>
					<ds:DigestMethod Algorithm="http://www.w3.org/2001/04/xmlenc#sha256"></ds:DigestMethod>
					<ds:DigestValue>GEEjC7+78bk7Mudp5kMbPmLlT5LzTQWnFg0dQgcJrGY=</ds:DigestValue>
				</ds:Reference>
			</ds:SignedInfo>
			<ds:SignatureValue>JKfwYWzUkGtKY9zXqhSLE/NmXlSdlNeDhUoT1gXjdlc952ryQEXhJoaH6CwfTWWUyY1SBLb+iWqq/8WADRm2cUbq8DhFp5Ozy0u4/rUa8pkTlByKNGDRtdZN7K4eIl8T5oj0fQu0drFv3DoVu/7Mow2l2Cb0xZNJNso9NFYP8EXYr4oI5O5MemqUI/pUwM3eue/FvxdTKwgDH7LU7ZxNNBMJy6L5fz/ooT/hOQ18nR2O3MblcconNpo8Q74NfKYU07WeuuAA2ugMJA+uHmlsVaPw6USIH/OGjxudlTR9OpKDj9sI1F7n+TF8yB3cZgWIDIyeRRQSmZYwKfCXTc56EQ==</ds:SignatureValue>
			<ds:KeyInfo>
				<ds:X509Data>
					<ds:X509Certificate>MIIDdzCCAl+gAwIBAgIES3eb+zANBgkqhkiG9w0BAQsFADBsMRAwDgYDVQQGEwdVbmtub3duMRAwDgYDVQQIEwdVbmtub3duMRAwDgYDVQQHEwdVbmtub3duMRAwDgYDVQQKEwdVbmtub3duMRAwDgYDVQQLEwdVbmtub3duMRAwDgYDVQQDEwdVbmtub3duMB4XDTE2MDUyNDEzNDEzN1oXDTI2MDUyMjEzNDEzN1owbDEQMA4GA1UEBhMHVW5rbm93bjEQMA4GA1UECBMHVW5rbm93bjEQMA4GA1UEBxMHVW5rbm93bjEQMA4GA1UEChMHVW5rbm93bjEQMA4GA1UECxMHVW5rbm93bjEQMA4GA1UEAxMHVW5rbm93bjCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEBANdIhkOZeSHagT9ZecG+QQwWaUsi7OMv1JvpBr/7HtAZEZMDGWrxg/zao6vMd/nyjSOOZ1OxOwjgIfII5+iwl37oOexEH4tIDoCoToVXC5iqiBFz5qnmoLzJ3bF1iMupPFjz8Ac0pDeTwyygVyhv19QcFbzhPdu+p68epSatwoDW5ohIoaLzbf+oOaQsYkmqyJNrmht091XuoVCazNFt+UJqqzTPay95Wj4F7Qrs+LCSTd6xp0Kv9uWG1GsFvS9TE1W6isVosjeVm16FlIPLaNQ4aEJ18w8piDIRWuOTUy4cbXR/Qg6a11l1gWls6PJiBXrOciOACVuGUoNTzztlCUkCAwEAAaMhMB8wHQYDVR0OBBYEFMm4/1hF4WEPYS5gMXRmmH0gs6XjMA0GCSqGSIb3DQEBCwUAA4IBAQDVH/Md9lCQWxbSbie5lPdPLB72F4831glHlaqms7kzAM6IhRjXmd0QTYq3Ey1J88KSDf8A0HUZefhudnFaHmtxFv0SF5VdMUY14bJ9UsxJ5f4oP4CVh57fHK0w+EaKGGIw6TQEkL5L/+5QZZAywKgPz67A3o+uk45aKpF3GaNWjGRWEPqcGkyQ0sIC2o7FUTV+MV1KHDRuBgreRCEpqMoY5XGXe/IJc1EJLFDnsjIOQU1rrUzfM+WP/DigEQTPpkKWHJpouP+LLrGRj2ziYVbBDveP8KtHvLFsnexA/TidjOOxChKSLT9LYFyQqsvUyCagBb4aLs009kbW6inN8zA6</ds:X509Certificate>
				</ds:X509Data>
			</ds:KeyInfo>
		</ds:Signature>
		<saml:Subject>
			<saml:NameID Format="urn:oasis:names:tc:SAML:2.0:nameid-format:persistent" NameQualifier="https://govidp.example.net:10443/idp" SPNameQualifier="https://am.example.com:9443/login">n0xFKsQyzOpE1w7IoCdhd0u2ef9N</saml:NameID>
			<saml:SubjectConfirmation Method="urn:oasis:names:tc:SAML:2.0:cm:bearer">
				<saml:SubjectConfirmationData InResponseTo="s27845c3b947bab365dd2972df4ac2794a4e0f0a65" NotOnOrAfter="2022-07-28T04:35:59Z" Recipient="https://am.example.com:9443/login/AuthConsumer/metaAlias/alpha/sp"></saml:SubjectConfirmationData>
			</saml:SubjectConfirmation>
		</saml:Subject>
		<saml:Conditions NotBefore="2022-07-28T04:15:59Z" NotOnOrAfter="2022-07-28T04:35:59Z">
			<saml:AudienceRestriction>
				<saml:Audience>https://am.example.com:9443/login</saml:Audience>
			</saml:AudienceRestriction>
		</saml:Conditions>
		<saml:AuthnStatement AuthnInstant="2022-07-28T04:25:59Z" SessionIndex="s2b9b200f22346b95a0041b23a405717d817d6b201">
			<saml:AuthnContext>
				<saml:AuthnContextClassRef>urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport</saml:AuthnContextClassRef>
			</saml:AuthnContext>
		</saml:AuthnStatement>
		<saml:AttributeStatement>
			<saml:Attribute Name="govidp_mail">
				<saml:AttributeValue
					xmlns:xs="http://www.w3.org/2001/XMLSchema"
					xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:type="xs:string">user.41@example.com
				</saml:AttributeValue>
			</saml:Attribute>
		</saml:AttributeStatement>
	</saml:Assertion>
</samlp:Response>

```