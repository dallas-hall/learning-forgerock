# AM410 Day 1 - Enhancing Intelligent Access

- [AM410 Day 1 - Enhancing Intelligent Access](#am410-day-1---enhancing-intelligent-access)
  - [Lab Environment Setup](#lab-environment-setup)
  - [Lesson 1 - Exploring Authentication Mechanisms](#lesson-1---exploring-authentication-mechanisms)
    - [Authentcation & Authorization](#authentcation--authorization)
    - [Realms](#realms)
    - [Authentication Lifecycle](#authentication-lifecycle)
    - [Intelligent Authentication](#intelligent-authentication)
    - [Lab Notes](#lab-notes)
      - [Setup](#setup)
      - [Realms](#realms-1)
  - [Lesson 2 - Protecting A Website With IG](#lesson-2---protecting-a-website-with-ig)
    - [AM Edge Clients](#am-edge-clients)
    - [IG](#ig)
    - [Authenticate Identities With AM](#authenticate-identities-with-am)
    - [Lab Notes](#lab-notes-1)
      - [IG](#ig-1)
  - [Lesson 3 - Controlling Access](#lesson-3---controlling-access)
    - [Entitlement Management](#entitlement-management)
    - [Lab Notes](#lab-notes-2)
      - [Policies](#policies)

## Lab Environment Setup

Task 1: Examine the AM installation inside of Student workbook > page 17

The VM is available on the internet.

## Lesson 1 - Exploring Authentication Mechanisms

###  Authentcation & Authorization

Businesses offer services and resources to their users. Businesses typically use identity & access management (IdAM) tools to gate access to their services and resources. All access management tools need to:
* Identify who the user is. This is called **Authentication**.
* Identify what the user is allowed to do. This is called **Authorization**.

The **ForgeRock Access Management (FR AM)** tool provides authentication and authorisation. Once a user has successfully authenticated, FR AM maintains information about that user's session a server-side in a structure called an `SSOToken`. Users need to be authenticated to FR AM before they can access any services gated by FR AM.

![images/am-auth-1.png](images/am-auth-1.png)

AM services are written in Java. The layers in this image are:

1. The Protected resources users are trying to access.
2. REST API layer. This is exposed to the world to provide a common way to access FR AM resources. e.g. request user information.
3. AM core services. e.g. authentication and session managment.
4. Persistent storage for FR AM configuration, identities, policies, applications, sessions. e.g. sessions are stored in the **Core Token Service (CTS)** which is a **ForgeRock Directory Services (FR DS)** LDAP instance.

### Realms

![images/am-realm-1.png](images/am-realm-1.png)

Realms:
* Configure AM experiences according to groups of users.
* Organize applications and authorization.
* Manage configuration efficiently and securely:

Realms can configure:
* Authentication
* End user branding
* Identities (users, groups, and privileges)
* Identity stores
* Service configuration
* Authorization
* Applications (policy agents, OAuth 2.0 (OAuth2) clients, and federation)

![images/am-realm-2.png](images/am-realm-2.png)

Realms can be used to create a specific user experience for a group of users. If a solution must be put in place for various sets of users, multiple realms can be created.

Top Level Realm:
* Is the default realm created when AM is installed.
* Should be used by FR AM admins only.
* It is also known as `/` or `root` Realm.

Subrealms:
* Should always be created for other use cases rather than using the Top Level Realm.
  * A single subrealm is appropriate if the user population is a single community and requires a uniform FR AM experience.
  * Multiple subrealms are appropriate is the user population isn't a single community and requires a different FR AM experience per community.
* Application owners can have access to their Subrealm through the FR AM UI to be able to configure it.

![images/am-realm-3.png](images/am-realm-3.png)

The top level in the UI is for global configuration and the left side panel is for the currently selected Realm.

When the FR AM server receives an authentication request, one of the first decisions made is to determine the Realm to which the user is trying to authenticate.

There are 2 ways to access a realm:
1. Use the `realm` URL parameter to access the authentication interface, e.g. [https://am.example.com:9443/login/XUI/?realm=/alpha#login](https://am.example.com:9443/login/XUI/?realm=/alpha#login) would log into the alpha Realm.
2. You could make a better user experience by using a DNS alias for this URL, e.g. [https://subscribers.example.com:9443/login/XUI/#login](https://subscribers.example.com:9443/login/XUI/#login) would redirect the user to the correct page. This can be done in Realm configuration (e.g. click edit on the Realm in the Dashboard).

### Authentication Lifecycle

![images/am-auth-2.png](images/am-auth-2.png)

When AM services are used as part of an access management solution, every protected resource will somehow need to authenticate the end user before they can access it. FR AM authentication is done through a series of REST API calls. This can be done through a user's browser or by an application using the REST API. The user's credentials are checked against the FR DS CTS and if they are valid a session is granted. Users will no longer need to authenticate to FR AM while they have a valid session. The `SSOToken` is a unique ID that stores a user session and is used by all AM systems. A `SSOTokenID` is given to the client. The `SSOTokenID` is either set as a cookie when using a browser, returned to the service in case of a REST call, or included in a JSON Web Token (JWT) when using **ForgeRock Identity Gateway (FR IG)** or a web agent.

![images/am-auth-3.png](images/am-auth-3.png)

Users accessing AM or a resource protected by AM for the first time do not have an existing session and are called unauthenticated users. They will be required to authenticate with AM.

There are 2 session types in FR AM:

**Authentication sessions** are the process of getting user credentials and validating them. They:
* Keep track of the state during authentication.
* Have a limited lifetime which is the authentication process, up to a maximum of 2 minutes. This is stored in the `amlbcookie` clientside cookie.

**Authenticated sessions** are the result of a successful authentication. They:
* Represent an interactive exchange between AM and an identity.
* Created after successful authentication.
* Persisted in the CTS store, by default.
* Referenced using a unique `SSOTokenID` value.
* Contain internal and custom properties.
* Have idle timeout and maximum lifetime. Each time the session is checked for validity by an application or a service, its idle timeout is reset.
* Invalidated by timeout (idle or lifetime), active logout from the user, and administrative invalidation.

Every session has an enforced idle timeout and maximum lifetime:
* Idle timeout: After 30 minutes (default) of inactivity, the session is deactivated.
* Maximum lifetime: After 120 minutes (default), the session is deactivated.

Timeouts can be configured globally, per realm, or per user through the session service.

![images/am-auth-4.png](images/am-auth-4.png)

A session contains information to make access management decisions, such as the user's identity, authentication method, remaining session time, and the authentication realm.

![images/am-auth-5.png](images/am-auth-5.png)

After authentication and AM creates a session, AM needs to communicate a reference about the session to any party that may need it, such as IG, web agents, and applications.

Where no browser is involved, AM returns the `SSOTokenID` directly; for example, as a JSON Response to a REST call. When a browser is used, the browser is instructed to set a cookie. The cookie name is` iPlanetDirectoryPro` by default. You should change the default cookie name for production environments.

`iPlanetDirectoryPro` cookie is the most important clientside cookie as it has the authentication session from ForgeRock AM. This cookie is always sent to ForgeRock AM with each request so ForgeRock AM knows the user is still authenticated. ForgeRock AM will check to see if that session is still valid. This approach is useful when you have full control of the network and want SSO, but isn't desireable for internet facing applications. When you don't have control of the network use another approach like OAuth2 or OIDC.

![images/am-auth-6.png](images/am-auth-6.png)

AM can be configured to use one of the following two methods to keep track of the session:
* A CTS-based session, which is stored in FR DS and is the default approach.
* A client-based session, which is stored in a user's browser using JWT token. You could decode this is https://jwt.io but remember JWT's start with `ey` so delete anything before `ey`.

AM creates an authentication session to track the user or entity progress through an authentication tree.

The session cookie:
* Is an SSOTokenId value for CTS-based sessions.
* Is a JWT containing the SSOTokenId value for client-based sessions.
* Has a domain scope by default, which defines the visibility of the cookie.
  * If the cookie has not expired, it is presented to the server within the scope.
  * If the cookie has expired or not within the scope, it is not presented.

By default, the AM cookie is configured as domain cookie based on the AM **fully qualified domain name (FQDN)**. In this class, the cookie domain is [am.example.com](am.example.com). Therefore, the cookie is not available to [fec.example.com](fec.example.com) or [am.example.org](am.example.org), and would be available to [subdomain.am.example.com](subdomain.am.example.com).

**Note:** By default the session cookie has the scope of a domain name. So in the brower's developer tools your session cookie is not displayed when the browser accesses another domain, because the cookie is configured as a domain-based cookie that is visible to the domain from which it was sent, and subdomains of the cookie domain. Once you return the previous domain your session cookie will be accessible again. When logging out the cookie is removed.

![images/am-auth-7.png](images/am-auth-7.png)

When a cookie is set in the browser and the user returns to the AM login page, AM can read the cookie and validate it. If the validation is successful, AM can proceed with the flow. Otherwise, AM considers that the user is not authenticated and displays the login page again.

![images/am-auth-8.png](images/am-auth-8.png)

Once authentication is successful, the end user may end up on different pages, depending on the context.

Note that changing the Default Success Login URL or the Default Failure Login URL values requires the new URL values to be added to the Validation Service whitelist.

An end user, however, only has access to their user profile page and self-service areas, if configured. Note that a user account may be granted full administrative privileges. If that is the case, the user would then have access to both the AM Admin UI and the user's profile page and self-service areas.

The AM Admin UI can only be accessed by AM administrator users.

An end user, however, only has access to their user profile page and self-service areas, if configured. Note that a user account may be granted full administrative privileges. If that is the case, the user would then have access to both the AM Admin UI and the user's profile page and self-service areas.

In most cases, however, AM is only one step in a process of accessing entirely independent resources, such as access to a website. The user would be redirected to AM and, after successful authentication, would expect to return where they came from. AM login URLs can add a goto URL parameter to specify a redirection URL when authentication is successful. For example:

[https://am.example.com/login/?realm=/alpha#login/&goto=https://fec.example.com](https://am.example.com/login/?realm=/alpha#login/&goto=https://fec.example.com)

After successful authentication or validation of existing `SSOTokenID`, AM sends a redirect response to the browser, and the user is redirected to the `goto` URL.

By default, the `goto` and `gotoOnFail` parameters can only reference:
* URLs that share the same scheme, FQDN, and port as AM.
* URLs that are relative to AM URL.

To access other `goto` and `gotoOnFail` URL values, add the URLs to the realm's Validation Service.

**GOTCHA:** AM requires that all goto redirection URLs are whitelisted in the access management Validation Service for the Realm.

![images/am-auth-9.png](images/am-auth-9.png)

When you are logged into a Realm and try to access aother Realm that you are not logged into, you will see the page `LEAVING SITE`, also called the switchRealm page (check the fragment name in the URL). It is a page displayed when a user is already authenticated to a realm and tries to access another one. This demonstrates that a user can only be logged in to one realm at a time.

### Intelligent Authentication

AM authentication mechanisms use:
* A tree is the primary mechanism for authentication.
* All authentication takes place through a tree, either through:
  * The default tree for a given realm.
  * A tree specified in the URL by adding the `service=<tree-name>` parameter.

![images/am-auth-2.png](images/am-auth-2.png)

When a user accesses the AM login page, AM looks into its configuration and starts a specific authentication process. To decide what that process is, AM takes into account the URL request and the parameters provided in the request.

In the absence of parameters around authentication choices, AM uses the defaults defined for the specific realm being accessed.

AM uses:
* Credentials to authenticate the user and capture credentials:
  * Interactively: LDAP, WebAuthn.
  * Non-interactively: Device profile.
* Authentication nodes within trees to:
  * Control the type of credentials asked of the user.
  * Connect to the credentials store and validate the credentials.
* Trees to control the logical flow of the authentication process.

**Interactive authentication** is when the user is actively involved in the authentication process and is requested for credentials or a touch ID. A **non-interactive authentication** method uses credentials to authenticate the user, but the user does not have to manually supply said credentials. For example, the device profile could be used.

![images/am-trees-1.png](images/am-trees-1.png)

A **tree** is an end-to-end workflow invoked by an end user or device. A tree is a collection of nodes combined to form an authentication process flow. Trees:
* Defines an authentication process.
* Utilizes authentication nodes to build a flexible logical process flow: loops, branches, and recursions.
* Starts at a single entry point. Lets nodes store and retrieve information.
* Completes with one exit point: Either a success or failure ending that determines the authentication result.
* Can be nested or reused as an inner tree within other trees.

Each tree has one entry point and one exit point at runtime. A tree ends with either a success or a failure result.

**Authentication nodes** are the building blocks of a tree authentication flow. Each node corresponds to a single task. Authentication nodes:
* Are building blocks of a tree authentication flow.
* Correspond to a single task in the tree, such as:
  * Starting or finishing the tree.
  * Collecting user data and contextual information
  *  Taking decisions based on configuration and available data.
  *  Locking an account.
* Can have either single or multiple outcomes.
* Can be extensible through JavaScript via the Scripted Decision node or custom nodes written in Java.

Manage trees:
* Use and modify pre-configured templates for common authentication requirements.
* Edit trees with the Tree editor.
* Start from scratch to create a custom tree.
* Configure one tree as the default for a realm.

The AM Admin UI lets you:
* Create, edit, and delete trees.
* Configure the default tree for a realm

AM pre-configured tree templates can be used as is for common end user trees.

The login page presented to the user depends on the URL realm parameter or DNS alias used.
The authentication tree selected by AM is:
* The default tree for the realm.
* The URL parameters `authIndextype` and `authIndexValue;` for example, `authIndexType=service&authIndexValue=MyTree`.
* The End User UI with the tree name added with the service parameter in the URL; for example: https://am.example.com/login/?realm=/alpha&service=MyTree#login

![images/am-trees-2.png](images/am-trees-2.png)

The Tree editor:
* Is a visual interface for creating and editing trees.
* Supports drag-and-drop actions for nodes and their connections.
* Provides a searchable list of nodes in the Nodes panel on the left.
* Displays configurable properties, if any, for a selected node in the configuration panel on the right.

![images/am-trees-3.png](images/am-trees-3.png)

A tree is called an inner tree when it is nested inside another tree by using an Inner Tree Evaluator node, which can be used by any number of other trees. So, if you modify an element in the inner tree, the change affects all the trees using it.

![images/am-nodes-1.png](images/am-nodes-1.png)

AM nodes are grouped into the following categories:
* Basic Authentication Nodes
* Multi-Factor Authentication Nodes
* Risk Authentication Nodes
* Behavioral Authentication Nodes
* Contextual Authentication Nodes
* Federation Authentication Nodes
* Identity Management Authentication Nodes
* Utilities Authentication Nodes
* IoT Authentication Nodes

The basic authentication nodes include:
* The Data Store Decision node: Uses the AM identity store to validate credentials collected.
* The Kerberos Node: Enables desktop SSO such that a user who has already authenticated with a Kerberos Key Distribution Center can authenticate to AM without having to provide the login information again.
* The LDAP Decision node: Validates credentials by using an external directory server using LDAP/LDAPS.
* Username Collector node: Prompts the user for their username.
* Password Collector node: Prompts the user for their password.

![images/am-nodes-2.png](images/am-nodes-2.png)

Collector nodes prompt for user input and device data, such as device profile, location, and certificates. Some collector nodes are also decision nodes. Most collector nodes have a single outcome, except the collectors that are also decision nodes. Most of these collector nodes require some additional configuration.

* OTP Collector Decision node: Prompts for a one-time password to be entered and compares the entered value with a generated value.
* Recovery Code Collection Decision node: Collects a recovery code from the user and compares its hash to a hashed code belonging to the user.
* Choice Collector node: Obtains a choice from the user whose value is one of the node outcomes.

![images/am-nodes-3.png](images/am-nodes-3.png)

Any node outcome can be connected with the input of another node to form a loop mechanism. A node, such as a decision node, can have more than one outcome to provide a branch or alternate path to the flow. Inner Tree Evaluator nodes follow the logic defined in another tree.

![images/am-nodes-4.png](images/am-nodes-4.png)

* The Page Node combines multiple nodes, such as the Username Collector node and Password Collector node, that request input into a single page for display to the user.
* The Message Node displays a custom, localized message on the page, and can provide a localized positive and negative response that the user can select to proceed.
* The Choice Collector node defines two or more options to display to the user when authenticating. Each choice corresponds to a different outcome.
* The Get Session Data, Set Session Properties, and Remove Session Properties nodes can be used to read, write, and delete session properties.
* The Meter node increments a specified metric key each time the node is encountered.
* The Timer Start and Timer Stop nodes can be used to track elapse time between nodes in a tree.
* The Inner Tree Evaluator node allows the nesting and evaluation of authentication trees as a child within a parent tree. There is no limit to the depth of nested trees.
* The Scripted Decision node allows the execution of scripts during authentication. Tree evaluation continues along the outcome path matching the script result. Scripts must first be created in the Script section.
* The Retry Limit Decision node allows the specified number of passes through the Retry outcome path, before continuing along the Reject path.
* The Success URL and Failure URL nodes set the redirection URL to be applied when authentication succeeds or fails, respectively.

![images/am-nodes-5.png](images/am-nodes-5.png)

The process of adding a script to a tree involves:
1. Navigating to SubRealm > Scripts.
2. Creating a Decision node for authentication trees script, and writing the code in JavaScript or Groovy. The example shows the outcome is set to one of two results: "success" or "fail" as a string value, depending on the result of a condition in the logic.
3. Add the Scripted Decision node to the tree and configure the node:
   1. Use your script, myTreeScript, as shown in the example.
   2. Define outcomes that match those returned by the script.

### Lab Notes

#### Setup

Run `st-setlab-env.sh` once to set it up.

To install and configure AM, you need to have an application server installed. In this class, you work with a pre-installed instance of Apache Tomcat (Tomcat).
* A pre-installed instance of Tomcat is located in `/opt/tomcats/am` and is configured to listen on HTTP port `18080`.
* The Tomcat instance has also been configured with SSL/TLS certificates for use in this course and listens to port `9443`.
* The AM binaries have been downloaded and copied into the `/opt/forgerock/software` folder.
* The `AM-7.1.0.war` file has been copied to the `/opt/tomcats/am/webapps` folder. This is using an embedded FR DS instance.
* The name of the file is `login.war`. Note that there is also a sub-folder named `login` that is the expanded version of the `login.war` file, which is created when the application server is started and first discovers the presence of the `.war` file.  The URL is https://am.example.com:9443/login andd URL path `/ login` is derived from the name of the `.war` file located in the Tomcat `webapps` folder.

```
[forgerock@forgerock webapps]$ pwd
/opt/tomcats/am/webapps
[forgerock@forgerock webapps]$ tree -L 1
.
├── docs
├── examples
├── host-manager
├── login <- the expanded login.war.
├── login.war <- the file being used by Tomcat to serve AM. Renaming this would rename the URL path.
├── manager
└── ROOT

6 directories, 1 file
```

* `/home/forgerock/.openamcfg ` contains the `am` configuration file which is `AMConfig_opt_tomcats_am_webapps_login_`
* `/home/forgerock/login` contains files related to this AM instance, such as audit log files .e.g `/home/forgerock/login/var/audit`
* Start AM with `/opt/tomcats/am/bin/startup.sh`
* Look at the AM logs with `tail -f /opt/tomcats/am/logs/catalina.out`
* Shutdown AM with `/opt/tomcats/am/bin/shutdown.sh 30 -force`
  * The command tells Tomcat to try and shut down gracefully for 30 seconds. If the process cannot be closed after that period of time, it will kill the process.
  * For the `shutdown.sh` command to work, a `setenv.sh` file already exists in the `/opt/ tomcats/am/bin` directory, with the `CATALINA_PID` variable defined in the file as `CATALINA_PID="$CATALINA_BASE/bin/catalina.pid"`
  * `ps aux | grep tomcat` to view running Tomcat instances and `kill -9 $PID` to kill Tomcat if the `shutdown.sh` didn't work.

#### Realms

By using a DNS alias, you can authenticate to a realm by simply accessing a different fully qualified domain name (FQDN).

Users reach the AM login page because another portal or application sends them there. Typically, users are redirected back to where they came from. In order to do so, the portal or application can use the `goto` URL parameter.

You can view cookies in Developer Tools > Application > Storage > Cookies.

When a user or the administrator logs in to AM, an internal session is created. That session is represented by a reference value called the SSOTokenID which is set in a cookie name that is:
* Called `iPlanetDirectoryPro` by default. The name should be changed in a production configuration for security reasons.
* Visible to the domain defined for your AM instance.

The presence of the cookie in a browser is required by users to eliminate the need to authenticate again when they return to the AM login page and access:
* Their profile page.
* Any valid goto URL page.

Setting a session cookie is the main technique used to protect access and provide **single sign-on (SSO)** to websites when using AM.

The cookie called `amlbcookie` is used to keep track of the instance accessed in the context of a load-balanced environment.

When you redirect to another site, your original site's session cookie is not displayed when the browser accesses another domain, because the cookie is configured as a domain-based cookie that is visible to the domain from which it was sent, and subdomains of the cookie domain. But the original site's cookie is still available in the browser and can be used when you return to that domain.

Deleting the session cookie in your browser does not terminate a session.

Anyone obtaining the session cookie value can gain access to the user's session. For that reason, all communication with AM must use SSL/TLS.

The `LEAVING SITE` page, also called the `switchRealm` page (check the fragment name in the URL). It is a page displayed when a user is already authenticated to a realm and tries to access another one. This demonstrates that a user can only be logged in to one realm at a time. Most of the time users only belong to one realm and are only able to log in to one realm. If a user belongs to two realms, they have to choose who they wish to log in as.

AM provides a setting that controls whether user profile attributes are displayed or not after successful authentication. The setting is also used to permit authentication in environments that do not define any identities. Set AM UI > Realm > Authentication > Settings > User Profile to `Ignored`.

AM defines several default authentication methods in each realm;
1. For administrative users to access the Admin UI.
2. Foor end users to authenticate with the realm.\

When accessing the AM login page without any service parameter in the URL, AM presents the default tree for authentication with the realm.


## Lesson 2 - Protecting A Website With IG

### AM Edge Clients

Protecting resources or services involves the ability to:
* Intercept requests.
* Instruct end users on how to obtain required access passes.
* Validate a user's identity.
* Evaluate their access rights.
* Relay the request to the backend, when validation is successful.

![images/ig-clients-1.png](images/ig-clients-1.png)

AM provides authentication, SSO, and policy functionality to enterprise applications. There are many different approaches to enterprise application integration. We will use **Forgerock Internet Gateway (FR IG)** to protect a website and demonstrate AM functionality.

### IG

IG is a standalone product used to offer web access management to all web applications, including legacy applications with built-in legacy authentication mechanisms. IG also offers SAML2 support to all applications, and is easy to integrate. IG can also support OIDC operations. IG replaces the old FR Java Web Agent.

Edge clients must be registered with AM before they can participate.  This takes the form of an entry within the AM configuration. The configuration is used by AM to specify how to communicate with the client, how the client authenticates, and the client actions that are allowed.

Integration of applications can take many paths as their context can vary a lot. For example:
* Is the application standalone?
* Is it a mobile application or an SPA?
* Is it deployed in a container for which an agent exists?
* Does the application manage its own authentication and authorization?

FR IG offers:
* Offers web access management to all web applications (including legacy applications).
* Offers easy to integrate SAML2 support.
* Supports OpenID Connect (OIDC) operations.

![images/ig-clients-2.png](images/ig-clients-2.png)

As an authentication guardian, IG intercepts all the communication trying to reach the website. It will try to assess whether the request should be allowed to go through or not. A second functionality is that IG can relay user information to the website/backend application. IG can also verify if access can be granted for that user under current circumstances.

![images/ig-clients-3.png](images/ig-clients-3.png)

IG must intercept all the access requests to protected resources. Typically, the architecture ensures that end users can access IG but cannot access the underlying resources directly. IG listens to the port, intercepts the request, and processes it based on its configuration

![images/ig-clients-4.png](images/ig-clients-4.png)

When a user requests a page from the FEC website, IG intercepts the request and searches for the presence of a cookie, as defined in its configuration. If the cookie is present, IG decodes its value, retrieves the SSO token value, and communicates with AM to find out if the SSO token is valid.
* If it is valid, IG proceeds with the flow.
* If it is not valid, or if the cookie does not exist, IG redirects the user to AM for authentication.

![images/ig-clients-5.png](images/ig-clients-5.png)

If you decode the IG JWT token you will see the information above. One of the most relevant pieces of information is contained in the `ssotoken` key. The value of the field is the `SSOTokenId`, which is a reference to the session created after the user authenticated successfully with AM, and which is persisted in the AM CTS store. The token also contains information about the realm where authentication took place, a reference to the end user, the name of the edge client for whom the token was created, and so on.

This means that the `ssotoken` matches the value of the `iPlanetDirectoryPro` cookie value.

![images/ig-clients-6.png](images/ig-clients-6.png)

When IG finds the IG cookie, it needs to validate the SSO token it contains. To do so, IG accesses the `.../am/json/realms/root/sessions?_action=getSessionInfo` endpoint of AM, providing the SSO token value in the request.
* If the session is not valid, AM responds with a JSON message containing a `"valid":false` key/value pair.
* If the session is valid, AM responds with a JSON message containing information about the session, as shown in the image above.

![images/ig-clients-7.png](images/ig-clients-7.png)

To log out from an application, protected with IG integrated with AM, you must log out of AM. You can either integrate a link in your website, which accesses the AM logout page, or use functionality from IG.

### Authenticate Identities With AM

![](images/am-identities-1.png)

So far, you tested authentication with the `demo` user. The demo user is an identity created, by default, in AM. Users who want to access a resource protected by AM must be able to authenticate successfully with AM, even if they are not in the internal pool of users. This can be done using the LDAP Decision node.

![](images/am-identities-2.png)

The LDAP Decision node configuration contains all the fields needed to set up the communication with the external LDAP Directory and to verify a user's credentials. One important feature of the LDAP Decision node is that it supports LDAP Behera Password Policies which are a feature of modern LDAP servers, such as DS.

![](images/am-identities-3.png)

Because the LDAP Decision node supports Behera Password Policies, it is able to distinguish between more outcome than simply a successful or failed authentication. This means that trees using the LDAP Decision node can provide a better experience to end users. If the profile associated with the username and password is locked, or the password has expired, tree evaluation continues along the respective `Locked` or `Expired` outcome paths.

Different paths can branch out from each outcome. For example, if the `Locked` outcome is triggered, the tree can present end users with a strong authentication alternative.

Data Store Decision node:
* For quick evaluation
* Only available for internal identities
* Generic for all LDAP directory types

LDAP Decision node:
* More specialized
* Supports LDAP Behera Policies
* Supports any LDAP directory: external or embedded

There is a sample tree, called `Example`, that uses the Data Store Decision node.

![](images/am-identities-4.png)

Identity is the heart of access management. When users try to access resources and are redirected to AM for authentication by IG, AM may need to have access to user information.

![](images/am-identities-5.png)

Using a Java identity repository plugin, an AM identity store can do the following:
* Connect to an external or embedded LDAP directory.
* Understand the structure of the directory.
* Search for a user in the directory based on an attribute.
* Retrieve profile information about that user.

Identity stores are required when user account data needs to be read or written. Most functionality you may need will be provided by the default implementation of the Java identity repository plugin. However, should you need further functionality, you can develop your own custom identity repository plugin using Java code.

![](images/am-identities-6.png)

By default AM installs an embedded identity store, however, it is recommended that you use your own external identity store in production. The recommended external identity store is FR DS. Others are supported.

* Identity stores are optional:
  * Only required if you need user profile data.
* Avoid creation of identities using the identity store:
  * Limited functionality.
  * Limited control over the creation flow.
* Avoid having multiple identity stores within a realm:
  * AM considers users with same uid as the same identity.
  * AM creates new users and updates existing users in each identity stor

Mapping identities into the realm is the norm for the majority of deployments, but it is an optional step. If no identity stores are configured, then AM can still authenticate a user, but no user profile data will be available.

### Lab Notes

#### IG

In this course, IG is pre-configured to log the communication between IG and AM.
Note that the logs do not contain the communication happening in the browser, instead the logs contain the direct communication between IG and AM's REST APIs. In a production environment, you should restrict the information collected in logs, for privacy reasons.

Understanding IG in detail is beyond the scope of this course. The basic principle is that IG acts as a powerful reverse proxy in front of the application. IG intercepts requests and uses configuration files, called routes, to decide how to process the request.

When IG receives a request, it goes through each of its routes until it finds a route that applies to the request, based on some conditions defined within the route. In the lab this was at ` /home/forgerock/.openig/config/routes`

`/home/ forgerock/.openig/bin/env.sh` contains the values that IG uses on start up.

`CrossDomainSingleSignOnFilter` is used when a protected application is not on the same domain as the AM domain.


## Lesson 3 - Controlling Access

### Entitlement Management

![](images/am-entitlement-1.png)

**Entitlement management** is the part of access management that deals with permissions. i.e. Can this SUBJECT perform this ACTION on this RESOURCE under these CONDITIONS? AM provides a centralized authorization framework to manage entitlements.

AM implements authorization with one or more policies defined in a policy set. Policy evaluation is a process based on the policies within a policy set, that determines if access to a resource is granted or denied.

![](images/am-pep-1.png)

![](images/am-pep-2.png)

The **policy enforcement points (PEP)** is responsible for enforcing the access decision evaluated by the **policy decision point (PDP)**. AM acts as a PDP. IG acts as a PEP.

![](images/am-policy-sets-1.png)

To evaluate policies, AM must first determine which set of policies should be taken into consideration. When configuring an authorization solution, an administrator defines all the policies required to protect an application or a service, and groups them into what is called a policy set. The policy set is defined within a specific realm.

![](images/am-policy-sets-2.png)

Each policy has a unique name and consists of the following components:
* Subjects: Identify the collection of users to whom the policy applies.
* Actions: Specify operations that are allowed or denied when accessing a resource.
* Resources: Represent the objects that are protected by the policy.
* Conditions (Optional): Apply environmental constraints to the policy.

Policies can also define response attributes to return a set of attributes with the policy decision to the PEP. A policy defines whether a subject can perform an action on a resource depending on some conditions.

![](images/am-policy-sets-3.png)

Authorization is not possible without authentication. Authentication contains information about the user attempting to access a particular resource protected by the policy. AM uses this information to determine if the user is subject to the policy. If the user is subject to the policy, the user is either denied or allowed access.
* Authenticated Users: This subject type implies that any user with a valid `SSOToken` is a member of this subject.
* Users & Groups: User or group as defined in the Identities pages of the realm the policy is created in.
* OpenID Connect/JWT Claim: Validates a claim within a JWT. This condition type only supports string equality comparisons and is case-sensitive.
* Never Match: Doesn't match any subject. This has the effect of disabling the policy.

![](images/am-policy-sets-4.png)

A resource type is a template for resources that policies must adhere to. The template contains patterns and actions:
* Patterns: When defining a resource in a policy, it must match one of the patterns of the resource type.
* Actions: A resource type also contains a set of actions that can be evaluated, and to which a value of Allow or Deny is associated.

The URL resource type is the most commonly used and is shipped with AM. It lets you protect websites and web applications. The HTTP actions, such as GET and POST, are implemented by default. Can user other resource types like OAuth2.

![](images/am-policy-sets-5.png)

When defining a resource within a policy, the resource must match an available pattern from the resource type.

Wildcards can be used to match a group of similar resources. Note that the wildcard does not match the question mark '?' used at the end of a URL to mark the start of the query parameters. The question mark must be added explicitly to a resource to allow parameters.

![](images/am-policy-sets-6.png)

Actions define which action is allowed or not allowed. If an action is absent, it is implicitly denied. When defining an action, you must associate a value to it to decide if it should be allowed or denied.

Most of the time, you should develop solutions that do not need to set up an explicit deny.

![](images/am-policy-sets-7.png)

![](images/am-policy-sets-8.png)

A policy environment condition allows constraints to be defined in the policy. The policy will only be applicable if the set of conditions are met, e.g. the user connects on the weekend.

You can write your own scripted policy environment conditions to extend the prebuilt ones.

![](images/am-policy-sets-9.png)

Subjects and environment conditions can be combined with logical operators. You can also nest logical operators, which gives full flexibility on how to define the pool of users and conditions.

![](images/am-policy-sets-10.png)

The response attributes define information that AM attaches to a response following a policy decision, e.g. name or email address. They are typically used for customizing applications.

![](images/am-policy-sets-11.png)

A PEP accesses AM authorization REST API for a specific realm. A PEP requests policy evaluation by providing AM with:
* The policy set to evaluate
* An `SSOTokenID` or JWT token for a subject
* A resource value
* Environment condition values

![](images/am-policy-sets-12.png)

For every policy considered for evaluation, AM must come to a decision regarding the policy. Either the policy is relevant, and the decision for the specific policy must be taken into account, or the policy is irrelevant.

For each of the policies, there will be a resource and a subject match. If either the subject or the resource does not match, the policy is irrelevant and no result will be recorded.

If both the resource and the subject match, the conditions will be considered. If the condition is met, the policy applies and the value for each defined action will be recorded; either allow or deny.

![](images/am-policy-sets-13.png)

Once all the results have been recorded, a combined decision is put together and sent to IG.

If no policy is applicable (either because there was no resource or subject match, or conditions were not met on any of the policies), the decision will always be empty. Remember that, by default, IG does not let anyone in. If no explicit allow comes back to IG, the result will effectively be to deny access.

![](images/am-policy-sets-14.png)

![](images/am-policy-sets-15.png)

Once a decision is reached, AM must send the response to the PEP. AM sends a response containing:
* Resources to which the decision applies
* Actions and their values: true (allow) or false (deny)
* Attributes: Extra information as defined in attribute response
* Advices

If a policy exists for the subject and the resource, AM responds with a json object containing information about which actions are allowed or denied.

### Lab Notes

#### Policies

The FEC website is protected by IG. IG is configured to evaluate policies with AM before granting access to the Browse and Premium pages. However, no policies are currently configured in AM. As the expected default behavior for a policy enforcement point (PEP) is to deny access, unless explicitly allowed by AM, IG always refuses access to the Browse and Premium pages.

IG could be configured to allow all authenticated users access to everything without a need for policy evaluation, and then reduce access for specific pages by creating policies to deny access. This is not best practice. It is always better to add access permissions to new resources, instead of having to actively restrict access to resources, and potentially forget to do so.

**Gotcha:** IG sends the policy evaluation request based on the backend resource, and not on the resource initially requested by the end user. This is the reason we entered the non-SSL resource on port 80, instead of the requested https resource on port 8443.

**Gotcha:** You need to explicitly allow accessing a URL with query paramtersbecause the wildcard (*) in resource patterns does not match question marks (?).

![](images/labs-ig-1.png)

![](images/labs-ig-2.png)

