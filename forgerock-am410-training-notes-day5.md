# AM410 Day 5 - Installing and Deploying AM

- [AM410 Day 5 - Installing and Deploying AM](#am410-day-5---installing-and-deploying-am)
  - [Lesson 1 - Installing AM](#lesson-1---installing-am)
    - [Deployment Planning](#deployment-planning)
    - [DS Configurations](#ds-configurations)
    - [Deployment Topology](#deployment-topology)
    - [Deployment Preparation](#deployment-preparation)
    - [Releases](#releases)
    - [Support Tickets](#support-tickets)
    - [Environment Preparation](#environment-preparation)
    - [Tomcat](#tomcat)
    - [Downloads](#downloads)
    - [Deploying AM](#deploying-am)
    - [Install Tools](#install-tools)
    - [Preparing DS](#preparing-ds)
    - [Preparing Truststore](#preparing-truststore)
    - [Installing](#installing)
      - [Web Wizard](#web-wizard)
      - [Common Deployment Problems](#common-deployment-problems)
      - [Amster](#amster)
    - [Bootstrap Process](#bootstrap-process)
    - [Labs](#labs)
  - [Lesson 1 - Upgrading AM](#lesson-1---upgrading-am)
    - [Versions](#versions)
    - [Upgrade Overview](#upgrade-overview)
    - [Backup Steps](#backup-steps)
    - [Upgrade Steps](#upgrade-steps)
    - [Restore From Backup](#restore-from-backup)
    - [Labs](#labs-1)
  - [Lesson 2 - Hardening AM Security](#lesson-2---hardening-am-security)
    - [Labs](#labs-2)
  - [Lesson 3 -  Clustering AM](#lesson-3----clustering-am)
    - [Labs](#labs-3)
  - [Lesson 4 - Deploying the Identity Platform to the Cloud](#lesson-4---deploying-the-identity-platform-to-the-cloud)
    - [Labs](#labs-4)

## Lesson 1 - Installing AM

### Deployment Planning

Read https://backstage.forgerock.com/docs/am/7.1/deployment-planning-guide/ for full details about:
* Planning the deployment architecture
* Deployment configuration locations
* An example deployment topology
* Sizing hardware and services for deployment
* Deployment requirements for product disk storage requirements and recommendations

Consider some important questions regarding your system:
* What are you protecting?
  * It can highlight plans for additional services that also require protected access.
* How many users are supported?
  * Is based on system usage and helps to project future growth and capacity needed to support growth.
* What are your product service-level agreements?
  * The SLAs help define your scaling and high availability requirements. You can scale your deployment using:
    * **Vertical scaling:** This involves increasing resources to a single host server.
    * **Horizontal scaling:** This involves adding additional host servers to form a cluster, typically behind a load balancer, so that the servers can function as a single unit.
* What are your high availability requirements?
  * **High availability** refers to your system's ability to operate continuously for a specified length of time. High availability systems design aims to prevent single points of failure and for continuous availability.
* Which type of clients will be supported?
  * Web applications may require a web agent or IG to protect them.
  * Java applications may require a Java agent or IG to protect them.
  * An AJAX application can use AM's RESTful API.
  * Legacy or custom applications can use ForgeRock Identity Gateway (IG).
  * Third-party applications can use SAML federation or a fedlet, OpenID Connect (OIDC), or OAuth 2.0 (OAuth2).
* What are your Secure Sockets Layer (SSL)/ Transport Layer Security (TLS) requirements?
  * Two common approaches to handling SSL are:
    * **SSL pass-through:** Using SSL through to the application servers containers themselves.
    * **SSL offloading:** Using a load balancer (or proxy) network device or software configured for SSL and using non-SSL communication between the load balancer and internal servers.
* What are your other security requirements? e.g. firewalls, DNS, etc.

Deployment plan tasks include:
* Project initiation, defines the overall scope and requirements of the deployment. In addition, it determines roles and responsibilities of stakeholders and resources required for the deployment.
* Architecting involves designing the deployment elements.
* Implementation is deploying the infrastructure and software.
* Automation and continuous integration for DevOps.
* Functional testing:
  * Test all functionality to deliver the solution without any failures.
  * Ensure customizations and configurations are covered.
* Non-functional testing:
  * Testing failover and disaster recovery procedures
  * Monitoring
  * Load testing to determine system demand and measure its responses to help anticipate peak load conditions.
* Supportability:
  * Creating the runbook for system administrators including procedures for backup and restoration, debugging, change control, and other processes.

![](images/am401/am-install-1.png)

### DS Configurations

For demo or evaluation purposes, you can install AM with an embedded **ForgeRock Directory Services (FR DS)** that supports the data for the CTS, identity data, and the configuration. The embedded DS is not supported for production purposes.

External DS instances are recommended for production environments. It is more common to create a separate DS:
* The CTS store
* The identity store
* The configuration store

![](images/am401/am-install-2.png)

The graphic shows three different ways for storing and managing the AM configuration data stores:
1. Embedded DS configuration: Stores all the data in the single embedded DS server that is included with an AM installation.
2. External DS configuration: Storing configuration data in an external DS data store(s).
  1. Is shared between the AM instances in your deployment.
  2. Can be replicated between multiple DS instances in a cluster.
  3. Can be made available to AM instances in different regions; improving availability and data integrity.
3. File-based configuration: Used specifically for automated cloud deployments, and is only available and supported when used within Docker images.
  1. Saved as files in the file system.
  2. Checked into a source control system, such as Git.

### Deployment Topology

![](images/am401/am-install-3.png)

The example topology is a basic starting point for a highly available and scalable AM configuration. The goal is to ensure there is no single point of failure. In this case, the load balancers represent more than one load balancer, possibly in an active-passive configuration, or other forms supported by the load balancer software or hardware used. The load balancer is configured to distribute requests among the AM instances within the site.

The AM instances have the same configuration so that they can all communicate with the same DS infrastructure, which is configured with replicated external DS instances for the CTS store, the configuration store, and the identity (user) store.

AM relies on the DS SDK for load balancing, failover, and heartbeat capabilities to spread the load across the directory servers or to throttle performance. Therefore, it is not recommended to place a load balancer in front of the external DS instances.

https://backstage.forgerock.com/docs/am/7.1/deployment-planning-guide/deploy-topologies-onprem.html has more details.

### Deployment Preparation

Before deploying and installing an AM instance, you should consider the following areas:
* Access the AM Release Notes and consult the information provided. Many of the links in the list above reference the release notes.
* Read the Before You Install section.
* Revisit the Deployment Planning Guide.
* Read the Installation Guide as part of any preparation.
* Consult the Setup Guide for key AM concepts.

### Releases

* Initial major release: Latest is 7.0.0.
* Minor release: Limited feature release that includes maintenance fixes, such as AM 7.1.0.
* Maintenance release: Collection of fixes and minor RFEs, such as AM 7.0.1.
* Patch release, such as AM 7.0.1.1:
  * Contains a small collection of fixes for customer issues.
  * Contains updates with non-breaking functionality in terms of API compatibility.
  * Excludes Requests for Enhancements (RFEs).
* All software fully supported for three years, with an extra year for all critical fixes and high security fixes.

A security advisory is a patch bundle with security issues ranging from minor to critical. ForgeRock uses the following three key areas when assessing any potential security issues, assessed in this order of precedence:
* Criticality: Where does the exploit sit on the severity line?
* Customer Impact: What is the potential impact to the customer?
* Publicity: Has the security issue been made public? Is there an exploit that has also been made public?

Security advisory contains all previous security advisory fixes for the version so that you only need to apply the latest to your environment. Note that patches are different for different versions.

### Support Tickets

If you are a customer, you can raise a ticket with ForgeRock Support in case of problems. Note that ForgeRock Support covers suspected bugs or problems with the software and some requests for assistance.

See https://backstage.forgerock.com/knowledge/kb/article/a60648400

### Environment Preparation

AM relies on browser cookies, which are returned based on the FQDN hostname or its domain name. See https://backstage.forgerock.com/docs/am/7.1/install-guide/prepare-networking.html

AM needs to be configured with a fully resolvable FQDN. For evaluation purposes, AM can use any FQDN by adapting your /etc/hosts file accordingly.

AM is a Java application that requires a Java Development Kit (JDK) to run. Ensure that you configure the recommended settings to avoid performance issues. See https://backstage.forgerock.com/docs/am/7.1/install-guide/prepare-java.html

The default number of file descriptors in your environment will not be enough; mainly if you are working with policy agents. Consider increasing the file descriptors value to 64k+. See https://backstage.forgerock.com/docs/am/7.1/install-guide/prerequisites-file-descriptors.html

### Tomcat

For Apache Tomcat containers, it is typical to set JVM parameters, such as the `CATALINA_OPTS` environment variable in the `$CATALINA_BASE/bin/setenv.sh`. See https://backstage.forgerock.com/docs/am/7.1/install-guide/prepare-containers.html

### Downloads

The AM and Amster software distribution files can be downloaded for evaluation purposes and licensed versions can be obtained via a subscription. Use https://backstage.forgerock.com/downloads

AM is distributed as a `.zip` file containing the following artifacts:
* Admin tools.
* The AM `.war` file that can be deployed to a supported Java web application container.
* Configuration tools, which are deprecated from 7.0.
* Monitoring dashboard samples.

Amster and agents are not included in the AM `.zip` distribution file, and must be downloaded separately. The Amster `.zip` file can be extracted into any empty folder. After the AM `.war` file has been deployed, Amster can be used to:
* Install AM instances.
* Connect to AM instances to manage AM configuration.
* Back up and restore AM configuration.

### Deploying AM

Extract the AM .war file and deploy with an appropriate file name in the Java web application container. Examples are:
* `login.war` results in AM being accessed with the https://am.example.com/login URL.
* `am.war` results in AM being accessed with the https://am.example.com/am URL.

Deployment of AM extracts the distribution files into the Java application container, before installation and configuration.

AM requires a deployment URI with a non-empty string after the `/`.
Do not deploy AM in the Java container root context. For example, in Tomcat, do not rename the `.war` file to the name `ROOT.war`.

After deploying the AM `.war` file, before installation, when you access the AM URL the first time, your application container displays the AM installer (or upgrader, if performing an upgrade) user interface. ForgeRock recommends that any external network access to the application container is suspended until the installation, or upgrade, is complete.

Customers deploying AM in a production environment most commonly need to create a custom `.war` file, combining the core AM services and any customizations that have been developed. Customizations can include custom authentication nodes, custom AM services, and custom UI look and feel.

### Install Tools

3 types of installation process:
1. Web wizard:
  1. An AJAX-driven web configuration tool.
  2. Automatically checks answers to reduce deployment errors.
2. Amster:
  1. A command-line interface built upon the AM REST interface.
  2. Used in DevOps processes, such as continuous integration, command-line installations, and scripted cloud deployments.
3. `forgeops` repo - https://github.com/ForgeRock/forgeops.git
  1. Scripts that use DevOps tools, such as kustomize, docker, and Kubernetes.
  2. Used to customize and deploy the Identity Platform.

All tools trigger the same internal process that controls the deployment of AM.

### Preparing DS

![](images/am401/am-install-4.png)

See https://backstage.forgerock.com/docs/ds/7.1/install-guide/preface.html

### Preparing Truststore

AM requires access to the self-signed certificate that DS generates. AM may also require access to CA certificates for secure connections to other sites.

* Use a (JDK default) truststore that contains the necessary CA certificates, and change the default truststore password.
* Export the DS certificate (for each external DS server).
* Add (or import) the DS certificate to the truststore.
* Configure the AM JVM container to use that truststore when starting up.

See https://backstage.forgerock.com/docs/am/7.1/install-guide/prepare-trust-store.html

![](images/am401/am-install-5.png)

### Installing

#### Web Wizard

Both the web wizard and Amster methods collect input in different ways, and perform the same installation and configuration tasks. Both methods need you to accept the terms and conditions when starting installation. Amster provides a command-line driven approach for installation of AM.

![](images/am401/am-install-6.png)

![](images/am401/am-install-7.png)

* Server URL: The server URL is the FQDN of the server. The server name should be resolvable through DNS services or through the `/etc/hosts` file.
* Cookie Domain: The cookie domain is optional. An empty cookie domain creates a host-only cookie. Otherwise, a cookie domain value must be a DNS domain or subdomain under the AM server FQDN. Setting a domain cookie allows any host in the specified domain, and any of its subdomains, to receive a cookie. If the cookie domain is different to the domain used to access AM, then the AM administrator is unable to log in to the AM console.
* Platform Locale: The default value is `en_US`.
* Configuration Directory: The path to a configuration directory that AM uses to store its persistent information. If you use the embedded DS as a configuration store, then the embedded DS instance is created within the specified folder. The configuration directory specified also contains the AM log and debug files, and must be writable by the user's account under which the container is running.

![](images/am401/am-install-8.png)

On the Configuration Store settings page, choose one of the following two options before entering the connection details:
* Embedded DS, the default option.
* External DS, select for production configurations and for creating clusters, with or without a site configuration. The external DS instance should have been created with the am-config profile and started prior to entering the connection details. When using an external configuration store, you must set up replication as AM will not do it for you.

See https://backstage.forgerock.com/docs/am/7.1/install-guide/configure-sites.html#add-servers-to-site

![](images/am401/am-install-9.png)

When you select the New deployment option in the Configuration Store page (the previous step), the User Store page is displayed to configure where AM looks for user identities. AM must have write access to the directory service chosen, and adds to the directory schema needed for AM to manage access for users. For production implementations it is common for the administrator to select the Embedded User Data Store option, and then perform the following additional actions:
* Create a sub realm in AM, and add the external user store to the sub realm.
* Remove the embedded user store from the sub realm.
* Either remove:
  * The default `demo` user account from the embedded user store configured in the top-level realm, or
  * The embedded data store from the top-level realm as well.

![](images/am401/am-install-10.png)

The Site Configuration page configures a site name and a URL for a load balancer that is used to balance requests across multiple AM servers. When you deploy multiple servers, AM automatically enables session high availability. By default, AM is configured to store session data in a CTS DS service that is shared by multiple AM servers. If an AM server fails, the shared storage means that other AM servers in the deployment:
* Have access to the user's session data.
* Can serve requests about that user.
* Does not ask the user to log in again.

On the Summary page, you can:
1. Choose to edit any of the settings for the configuration store, user store, and site before starting the installation.
2. Select Create Configuration to start the installation process.

After starting the installation, the progress of the installation and configuration process is displayed. The information is written to a log file in the `/path/to/amconfig/install.log`, where `/path/top/amconfig` is the configuration directory you selected in the Server Settings page.

#### Common Deployment Problems

Installing and configuring AM is usually straightforward. However, occasionally the issues listed in the slide are causes of common problems.

* Memory: Can fail if the heap size < 1024 MB.
* Disk space: Will fail if free space < 5% + 5GB.
* No permissions on the configuration folder:
  * Which is created for the DS instance (if the Embedded DS options is selected) and bootstrap files.
  * Must be writable by the user process under which the AM container is running.
* Cookie domain misconfigured: The wrong cookie domain prevents the administrator from logging in to the AM Admin UI.

#### Amster

* A command-line interface built upon REST APIs, available since version 5.
* Provides remote access, scripted deployments, configuration export and import, and encryption of sensitive data.
* Downloaded from backstage.forgerock.com.
* Installed by unzipping the Amster .zip archive file into an empty directory.

![](images/am401/am-install-11.png)

![](images/am401/am-install-12.png)

![](images/am401/am-install-13.png)

There are a heap of `cfg` options that can also be specified, e.g. `cfgStoreDirMgrPwd` and `cfgStoreHost` to connect to an external FR DS.

![](images/am401/am-install-14.png)

See https://backstage.forgerock.com/docs/amster/7.1/entity-reference/preface.html

![](images/am401/am-install-15.png)

![](images/am401/am-install-16.png)

See https://backstage.forgerock.com/docs/amster/7.1/entity-reference/preface.html

Amster can export all the configuration related to an AM instance, and import it back to the same instance, or a different one. Note that Amster only manages configuration data. User information in data stores is not imported or exported, or modified in any way by Amster.

![](images/am401/am-install-17.png)

### Bootstrap Process

![](images/am401/am-install-18.png)

The AM bootstrap process steps are:
1. AM looks for the .openamcfg folder, located in the `$HOME` folder of the user running the AM container.
2. In the `.openamcfg` folder, AM reads a file named `AMConfig_<deploy-path>_`, where:
   1. The `<deploy-path>` is the name of the container path where AM is deployed; joined with underscore (_) characters.
   2. For example; if the AM deployment path is `/opt/tomcats/am/webapps/login`, the `<deploy-path>` is the `AMConfig_opt_tomcats_am_webapps_login_` name.
   3. The `AMConfig_<deploy-path>_` file contains the AM configuration folder path (specified during install time).
3. AM uses the configuration folder and looks for the bootstrap `boot.json` file in the `config` subfolder, as shown.

In the home directory of the user running the Java EE container process, a folder called `.openamcfg` is created when AM is installed the first time. For each AM instance installed, the `.openamcfg` folder contains a deployment file that contains the configuration folder specified when the AM instance was installed.

![](images/am401/am-install-19.png)

![](images/am401/am-install-20.png)

The AM configuration folder structure contains the following folders and their contents:
* `config`: Contains the boot.json file, and a subfolder structure with property files, such as the log4j.properties file.
* `security`: Contains the following important security related folders:
  * `keys/amster`: Contains key files for Amster to connect to AM without a password.
  * `keystores`: Contains the default AM keystore files.
  * `secrets/encrypted`: Contains files with the keystore passwords. If you decide to change the amadmin password by using a secret store, then the default location of the special secret store is in the `userpasswords` folder. The folder for the special secret store can be changed by configuring the `org.forgerock.openam.secrets.special.user.passwords.dir` advanced server property.
* `var`: Contains the following folders and files:
  * `audit`: The audit log files, such as access.audit.json, among others.
  * `debug`: The AM service and component log files, such as the Authentication log file.
  * `stats`: Statistics related to different AM functions.
  * `install.log`: The log file created when the AM instance is installed.

Other folders that may optionally be present are:
* `opends`: Contains the embedded DS folder structure, which is only present if the AM configuration is created with the embedded DS.
* `backups`: Contains backup .zip files of the embedded DS server (if used).
* `upgrade`: Contains log files related to the AM upgrade process.

AM also requires a DS for the configuration store to storage AM configuration. The DS configuration store can be an external or an embedded DS. Therefore, the AM administrator should check the DS configuration store log files to ensure the DS server is operating normally. When using embedded DS as a configuration store, the DS logs are located in the `/path/to/am-config/opends/logs` folder.

### Labs

## Lesson 1 - Upgrading AM

### Versions

![](images/am401/am-upgrade-1.png)

You can migrate AM from `5.5.x` versions and later to AM `7.1`.

See https://backstage.forgerock.com/knowledge/kb/article/a18529200

Backing the deployment is essential before any upgrade and at a minimum includes backup of:
* The DS servers in the deployment
* The AM configuration and bootstrap files
* The AM deployment folders in the Java container

Have a read of https://backstage.forgerock.com/docs/am/7.1/upgrade-guide/upgrade-planning.html and https://backstage.forgerock.com/docs/ds/7.1/maintenance-guide/backup-restore.html

All customizations need to be developed and tested on the new software before an upgrade is done. AM customization can be applied at different levels. Prepare an AM .war file that contains any tested customizations you require and include changes to AM.

### Upgrade Overview

General upgrade steps are:
* Back up an AM deployment.
* Prepare the new AM .war file.
* Copy the new AM .war file to the AM container and start the container.
* Upgrade AM with one of the following methods:
  * Access and follow the web wizard instructions.
  * Install and use the command-line configuration tool.
* Access and test the AM services and customizations.

### Backup Steps

General backup steps:
* Stop AM.
* For an embedded and external DS configuration: Archive the AM configuration folder and the `.openamcfg` folder; excluding log and lock files. For external DS servers, ensure the configuration DS is backed up using the DS backup operations.
* Start AM, if you require it to be running before an upgrade.

See https://backstage.forgerock.com/docs/ds/7.1/maintenance-guide/backup-restore.html

### Upgrade Steps

Upgrade steps for single AM instances:
* Stop AM.
* Deploy new customized .war file in the container.
* Start AM.
* Access AM, such as https://am.example.com/am and follow the upgrade instructions in the web wizard, or use the command-line configuration tool.
* Restart AM.

![](images/am401/am-upgrade-2.png)

![](images/am401/am-upgrade-3.png)

You can upgrade with the GUI or a CLI tool.

### Restore From Backup

It is not enough to back up a deployment. You must be able to restore it. That seems obvious, but actually learning to restore an environment is as important as backing it up every day.

* Do not mix versions: Structure of the configuration changes from release to release.
* In replicated environments, directory replication mechanically applies new changes, and data restored from the backup must not be older than the replication purge delay.
* To restore AM:
  * Stop the AM server.
  * Restore the AM container and configuration folders from the last backup.
  * Restart AM.



### Labs

## Lesson 2 - Hardening AM Security

###

### Labs

## Lesson 3 -  Clustering AM


###

### Labs

## Lesson 4 - Deploying the Identity Platform to the Cloud


###

### Labs

