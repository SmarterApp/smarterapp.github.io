---
title: Teacher Hand Scoring System (THSS) Installation Checklist
permalink: "/deployment/checklist/thss.html"
layout: "document"
categories: ["deployment", "checklist", "tds"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | Provide interface for teachers to hand-score assessment items |
| Communicates With | OpenAM, Permissions, ART, TIS |
| Repository Location | [https://bitbucket.org/sbacoss/testintegrationsystem_release](https://bitbucket.org/sbacoss/testintegrationsystem_release) |
| Additional Documentation | [README](https://bitbucket.org/fwsbac/teacherhandscoresys_release), [reportxml_os.xsd](https://bitbucket.org/fwsbac/teacherhandscoresys_release/src/9208d85c5fc1281ff2bb11ba7bad7e07df5fcf23/Docs/reportxml_os.xsd?at=default&fileviewer=file-view-default), [SAML Templates](https://bitbucket.org/fwsbac/teacherhandscoresys_release/src/9208d85c5fc1281ff2bb11ba7bad7e07df5fcf23/Docs/SAML/?at=default) |

***NOTE:***{: style="color: #f00"} *The THSS database can reside on the same MSSQL Server that hosts the TIS databases.*{: style="color: #04384e"}

## Instructions

### Create the THSS Database
* Create the following database on the MSSQL Server:
  * `TSS`

#### Create THSS Databse User Account(s)
* Create a user account (i.e. an MSSQL login or an Active Directory user) that has access to read/write (`db_reader` and `db_writer` roles) for the database cited above.

#### Create the TSS Database Schema
* Run the following scripts in order against the `TSS` database:
  1. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Items.sql`
  2. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Dimensions.sql`
  3. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.ConditionCodes.sql`
  4. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Districts.sql`
  5. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Schools.sql`
  6. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Students.sql`
  7. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Teachers.sql`
  8. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Tests.sql`
  9. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Responses.sql`
  10. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Assignments.sql`
  11. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.Logs.sql`
  12. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo._dbLatency.sql`
  13. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\dbo.fn_SplitDelimitedString.sql`
  14. [*\Path\to\THSS Repository*{: style="color: #f00;"}]`DB\TSS\Tables\*.sql`
    **NOTE:** The intent of item 14 is to describe running all of the stored procedure scripts against the `TSS` database.

### Create AWS Instance
* Create server instance to host Teacher Hand Scoring System components
  * AWS instance must be at least a **t2.small**
  * Select an image with the **Windows Server 2012 R2 Base**{: style="color: #04384e"} 64-bit operating system
* Create an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 1433
  * 80
  * 3389
  * 443

### Configure Windows AWS Instance
{% include checklist/windows_server_config.md %}

## Deploy the Teacher Hand Scoring System

### Build the Test Hand Scoring System
* Clone the THSS repository to a machine that can build .NET applications:
  * `hg clone https://`[*your BitBucket user or team name*{: style="color: #f00;"}]`/sbacoss/testintegrationsystem_release`
  * Example:
    * `hg clone https://`<span class="placeholder-example">jjohnson-fw@bitbucket.org</span>`/sbacoss/testintegrationsystem_release`
* Launch Visual Studio
* Open the `Src/TSS.sln`

{% include checklist/windows_vs_publish.md %}

### Deploy the THSS Component
* Connect to the AWS Windows server via RDP
* Create a directory where the THSS component will be deployed
  * The THSS Component is a web application, thus must be accessible by IIS
  * Example:  `C:\inetpub\`<span class="placeholder-example">thss</span>
* From the machine where the THSS web application was built, copy the *entire contents* of the directory where the **Publish** wizard deployed the component into the directory created in the previous step.

### Configure a THSS Website in IIS
* Launch the **Internet Information Services Manager**
  * Click **Start** button in lower-left corner (leftmost control on toolbar)
  * Click the down arrow (if the **Internet Information Services Manager** does not appear)
  * Search for and launch the **Internet Information Services Manager**
* Right click on the **Default Web Site**
  * Choose **Manage Website...**
  * Choose **Stop**
* Right click on **Sites** and choose **Add Website...**

![Add website within IIS](/res/images/checklist/iis_01_add_site.png){: width="100%"}

* Provide the following details in the dialog that appears:
  * Site name: [*Choose a meaningful name*{: style="color: #f00;"}]
  * Application Pool: [*Choose an exisitng Application Pool, or allow IIS to create a new Application Pool for this site (**recommended**)*{: style="color: #f00;"}]
  * Physical Path: [*The path to where the THSS web application components have been deployed*{: style="color: #f00;"}]
  * Binding:  [*Choose an available port*{: style="color: #f00;"}]
  * Host name:  [***OPTIONAL:*** *Provide a host name*{: style="color: #f00;"}]

* Screenshot showing example values for the TDS Receiver website:

![Add website details](/res/images/checklist/iis_07_thss_site_details.png){: width="100%"}

### Configure the THSS Application
* Navigate to [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data`
  * Example: <span class="placeholder-example">C:\inetpub\thss\App_Data</span>

#### Get IDP Metadata
* Create a new file named ***exactly*** `idp.xml` and save it to the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data` directory
* Navigate to `https://`[*FQDN or IP address of OpenAM server*{: style="color: #f00;"}]`//auth/saml2/jsp/exportmetadata.jsp?realm=/sbac`
  * Example:  `https://sso-deployment.sbtds.org/auth/saml2/jsp/exportmetadata.jsp?realm=/sbac`
* Right-click on the resulting XML and choose **View source**
* Copy the ***entire content*** of the results into [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\idp.xml` and save `idp.xml`.

#### Configure the fedlet.cot File
* Open [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\fedlet.cot` in an editor and update the following:

<div class="highlighter-rouge">
<pre class="highlight">
<code>cot-name=[<span class="placeholder">Name of the Circle of Trust in OpenAM; use <strong>sbac</strong> if following this guide</span>]
sun-fm-cot-status=Active
sun-fm-trusted-providers=https://[<span class="placeholder">FQDN or IP address of OpenAM server</span>]/auth, [<span class="placeholder">Entity ID for THSS in OpenAM, use <strong>thss</strong> if following this guide</span>]
sun-fm-saml2-readerservice-url=
sun-fm-saml2-writerservice-url=</code>
</pre>
</div>

* Example of configured `fedlet.cot`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>cot-name=<span class="placeholder-example">sbac</span>
sun-fm-cot-status=Active
sun-fm-trusted-providers=https://<span class="placeholder-example">sso-deployment.sbtds.org:443</span>/auth, <span class="placeholder-example">thss</strong> </span>
sun-fm-saml2-readerservice-url=
sun-fm-saml2-writerservice-url=</code>
</pre>
</div>

#### Configure the idp-extend.xml File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\idp-extended.xml` in an editor
* Update the following configuration:
  * `entityID=`"https://[*FQDN or IP address of OpenAM server*{: style="color: #f00;"}]/auth"

* Example of configured `idp-extended.xml`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;?xml version="1.0" encoding="UTF-8" standalone="yes"?&gt;
&lt;EntityConfig entityID="https://<span class="placeholder-example">sso-deployment.sbtds.org:443</span>/auth" hosted="0" xmlns="urn:sun:fm:SAML:2.0:entityconfig"&gt;
    &lt;IDPSSOConfig&gt;
        &lt;Attribute name="description"&gt;
            &lt;Value/&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="cotlist"&gt;
            &lt;Value>sbac&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="wantArtifactResolveSigned"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="wantLogoutRequestSigned"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="wantLogoutResponseSigned"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="wantNameIDEncrypted"&gt;
            &lt;Value>&lt;/Value&gt;
        &lt;/Attribute&gt;
    &lt;/IDPSSOConfig&gt;
&lt;/EntityConfig&gt;</code>
</pre>
</div>

#### Configure the sp.xml File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\sp.xml` in an editor
* Update the following configuration:
  * Replace all instances of `FEDLET_DEPLOY_URI` with the [*the protocol/scheme*{: style="color: #f00;"}]`://`[*FQDN or IP address of the THSS website*{: style="color: #f00;"}]

* Example of configured `sp.xml`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;EntityDescriptor entityID="<span class="placeholder-example">thss</span>" xmlns="urn:oasis:names:tc:SAML:2.0:metadata"&gt;
    &lt;SPSSODescriptor AuthnRequestsSigned="false" WantAssertionsSigned="false" protocolSupportEnumeration="urn:oasis:names:tc:SAML:2.0:protocol"&gt;
        &lt;SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Redirect" Location="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogout.aspx" ResponseLocation="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogout.aspx"/&gt;
        &lt;SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogout.aspx" ResponseLocation="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogout.aspx"/&gt;
        &lt;SingleLogoutService Binding="urn:oasis:names:tc:SAML:2.0:bindings:SOAP" Location="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogout.aspx"/&gt;
        &lt;NameIDFormat>urn:oasis:names:tc:SAML:2.0:nameid-format:transient&lt;/NameIDFormat&gt;
        &lt;AssertionConsumerService isDefault="true" index="0" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-POST" Location="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogin.aspx"/&gt;
        &lt;AssertionConsumerService index="1" Binding="urn:oasis:names:tc:SAML:2.0:bindings:HTTP-Artifact" Location="<span class="placeholder-example">http://54.200.42.254</span>/InitiateLogin.aspx"/&gt;
    &lt;/SPSSODescriptor&gt;
&lt;/EntityDescriptor&gt;</code>
</pre>
</div>

#### Configure the sp-extended.xml File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\sp-extended.xml` in an editor
* Update the following configuration:
  * Replace the `FEDLET_ENTITY_ID` with [*Entity ID for THSS in OpenAM, use **thss** if following this guide*{: style="color: #f00;"}]
  * Replace the `CIRCLE_OF_TRUST_NAME` with [*Name of Circle of Trust from OpenAM; use **sbac** if following this guide*{: style="color: #f00;"}]

* Example of configured `sp-extended.xml`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;EntityConfig xmlns="urn:sun:fm:SAML:2.0:entityconfig" xmlns:fm="urn:sun:fm:SAML:2.0:entityconfig" hosted="1" entityID="<span class="placeholder-example">thss</span>"&gt;
    &lt;SPSSOConfig metaAlias="/sp"&gt;
        &lt;Attribute name="description"&gt;
            &lt;Value>&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="signingCertAlias"&gt;
            &lt;Value>&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="encryptionCertAlias"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="basicAuthOn"&gt;
            &lt;Value&gt;false&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="basicAuthUser"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="basicAuthPassword"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="autofedEnabled"&gt;
            &lt;Value&gt;false&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="autofedAttribute"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="transientUser"&gt;
            &lt;Value&gt;anonymous&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAdapter"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAdapterEnv"&gt;
            &lt;Value&gt;&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAccountMapper"&gt;
            &lt;Value&gt;com.sun.identity.saml2.plugins.DefaultLibrarySPAccountMapper&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAttributeMapper"&gt;
            &lt;Value&gt;com.sun.identity.saml2.plugins.DefaultSPAttributeMapper&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAuthncontextMapper"&gt;
            &lt;Value&gt;com.sun.identity.saml2.plugins.DefaultSPAuthnContextMapper&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAuthncontextClassrefMapping"&gt;
            &lt;Value&gt;urn:oasis:names:tc:SAML:2.0:ac:classes:PasswordProtectedTransport|0|default&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="spAuthncontextComparisonType"&gt;
           &lt;Value&gt;exact&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="attributeMap"&gt;
           &lt;Value&gt;*=*&lt;/Value&gt;
        &lt;/Attribute&gt;
        &lt;Attribute name="saml2AuthModuleName"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="localAuthURL"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="intermediateUrl"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="defaultRelayState"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="assertionTimeSkew"&gt;
           &lt;Value&gt;300&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantAttributeEncrypted"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantAssertionEncrypted"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantNameIDEncrypted"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantArtifactResponseSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantPOSTResponseSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantLogoutRequestSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantLogoutResponseSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantMNIRequestSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="wantMNIResponseSigned"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="cotlist"&gt;
           &lt;Value&gt;<span class="placeholder-example">sbac</span>&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="saeAppSecretList"&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="saeSPUrl"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="saeSPLogoutUrl"&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="ECPRequestIDPListFinderImpl"&gt;
           &lt;Value&gt;com.sun.identity.saml2.plugins.ECPIDPFinder&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="ECPRequestIDPList"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="enableIDPProxy"&gt;
           &lt;Value&gt;false&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="idpProxyList"&gt;
           &lt;Value&gt;&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="idpProxyCount"&gt;
           &lt;Value&gt;0&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="useIntroductionForIDPProxy"&gt;
           &lt;Value&gt;false&lt;/Value&gt;
       &lt;/Attribute&gt;
       &lt;Attribute name="relayStateUrlList"&gt;
       &lt;/Attribute&gt;
    &lt;/SPSSOConfig&gt;
&lt;/EntityConfig&gt;</code>
</pre>
</div>

#### Configure the OpenAMSites.xml File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\OpenAMSites.xml` in an editor
* Update the following configuration:
  * `Contents` element:
    * `ForState=`[*Use **SBAC** or **SBAC_PT***{: style="color: #f00;"}]
  * `ClientSite` element:
    * `Url=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the ART server*{: style="color: #f00;"}]

* Example of configured `OpenAMSites.xml`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;?xml version="1.0" encoding="utf-8" ?&gt;
&lt;!-- This configuration file provides links for the drop-down navigation in THSS. --&gt;
&lt;!-- The state name must match the state specified in the settings.config document --&gt;
&lt;Contents&gt;
  &lt;Content ForState="<span class="placeholder-example">SBAC</span>" ContentSectionName="TssSubSites"&gt;
    &lt;ClientSites&gt;
      &lt;ClientSite&gt;
        &lt;ClientID&gt;Tide&lt;/ClientID&gt;
        &lt;Description&gt;ART&lt;/Description&gt;
        &lt;Url&gt;<span class="placeholder-example">http://54.186.87.166:8080</span>&lt;/Url&gt;
      &lt;/ClientSite&gt;
    &lt;/ClientSites&gt;
  &lt;/Content&gt;
&lt;/Contents&gt;</code>
</pre>
</div>

#### Configure the DataDistribution.config File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\DataDistribution.config` in an editor
* Update the following configuration:
  * `ConnectionStrings` element:
    * `DefaultConnection=`[*Connection information to communicate with the MSSQL Server that supports the **TSS** database*{: style="color: #f00;"}]
      * `Data Source=`[*The FQDN or IP address of the MSSQL Server that supports the **TSS** database*{: style="color: #f00;"}]
      * `Initial Catalog=`[*The name of the database that supports the TIS Service; use **TSS** if following this guide*{: style="color: #f00;"}]
      * `User id=`[*The user account that has read/write access to the **TSS** database*{: style="color: #f00;"}]
      * `Password=`[*The password for the user account configured in the **User id** section of the connection string*{: style="color: #f00;"}]
  * **NOTE:** The `Districts` element can be ignored; it is only used when data is partitioned by District (that is, in the case when each District has its own database).

* Example of configured `DataDistribution.config`:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;?xml version="1.0"?&gt;
&lt;TSSDataDistribution&gt;
    &lt;ConnectionStrings&gt;
        &lt;ConnectionString name="DefaultConnection" connectionString="Data Source=<span class="placeholder-example">tis-deployment2.cugsexobhx8t.us-west-2.rds.amazonaws.com</span>;Initial Catalog=<span class="placeholder-example">TSS</span>;User id=<span class="placeholder-example">remoteuser</span>;Password=<span class="placeholder-example">[redacted]</span>" default="true"&gt;
            &lt;Districts&gt;
                &lt;add id="1001"/&gt;
                &lt;add id="1002"/&gt;
                &lt;!-- This section is redundant and added for example only as all districts not explicitly mentioned in another connection string are added to the default configuration. --&gt;
            &lt;/Districts&gt;
        &lt;/ConnectionString&gt;
    &lt;/ConnectionStrings&gt;
&lt;/TSSDataDistribution&gt;</code>
</pre>
</div>

#### Configure the settings.config File
* Open the [*\Path\to\THSS web application*{: style="color: #f00;"}]`\App_Data\settings.config` in an editor
* Update the following configuration:
  * `appSettings` element:
    * `ART_API_REST_API_BASE_URL=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the ART server*{: style="color: #f00;"}]/rest
    * `ART_API_URL=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the ART server*{: style="color: #f00;"}]/rest/user
    * `ART_API_CLIENT=`[*Assessment publisher name; either **SBAC_PT** or **SBAC***{: style="color: #f00;"}]
    * `ART_OAUTH_URL=`https://[*FQDN or IP address of the OpenAM server*{: style="color: #f00;"}]/auth/oauth2/access_token?realm=/sbac
    * `ART_OAUTH_PASSWORD_GRANTTYPE=`[*Boolean for determining if OAuth request should use the **password** grant type*{: style="color: #f00;"}]
    * `ART_OAUTH_USERNAME=`[*User account that exists in OpenDJ*{: style="color: #f00;"}]
    * `ART_OAUTH_REQUIRED=`[*Boolean for determining if OAuth is required*{: style="color: #f00;"}]
    * `ART_OAUTH_PASSWORD=`[*Password for user account defined in **ART_OAUTH_USERNAME***{: style="color: #f00;"}]
    * `ART_OAUTH_SECRET=`[*Client secret for **ART_OAUTH_CLIENTID***{: style="color: #f00;"}]
    * `ART_OAUTH_CLIENTID=`[*The OAuth client name for the Monitoring and Alerting (MnA) component; can use a "common" OAuth client name, e.g. one OAuth client for multiple components*{: style="color: #f00;"}]
    * `IGNORE_TENANCY_CHAINS=`[*Boolean for determining if user tenancy chains should be evaluated*{: style="color: #f00;"}]
    * `SAML_OWNER_PREFIX=`sbac
    * `SAML_SESSIONREFRESH_URL=`https://[*FQDN or IP address of OpenAM server*{: style="color: #f00;"}]/auth/identity/attributes?refresh=true
    * `SAML_REDIRECT=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the THSS website*{: style="color: #f00;"}]
    * `PERMISSIONS_SCHEMA_URL=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the Permissions application*{: style="color: #f00;"}]/rest/role?component=Teacher Hand Scoring System
    * `LOAD_PERMISSIONS_FROM_LOCAL=`[*Boolean for determining if permission sets should be loaded from local files.  Set to **False8**{: style="color: #f00;"}]
    * `ART_ENTITIES_DATA_CACHING_DAYS=`[*Length of time (in days) to cache ART entity, minimizing number of calls to ART*{: style="color: #f00;"}]
    * `ART_SCORER_DATA_CACHING_MINS=`[*Length of time (in minutes) to cache ART scorer data, minimizing number of calls to ART*{: style="color: #f00;"}]
    * `IRIS_OPEN_SOURCE=`[*Set to **True***{: style="color: #f00;"}]
    * `IRIS_VENDOR_ID=`[*GUID representing the IRiS vendor*{: style="color: #f00;"}]
    * `IRIS_ROOT_URL=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the IRiS application*{: style="color: #f00;"}]/IRiS/
    * `IRISBlackbox_ROOT_URL=`[**The scheme/protocol*{: style="color: #f00;"}]://[*The FQDN or IP address of the IRiS blackbox application*{: style="color: #f00;"}]/IRiS/
    * `IRIS_PEM_LOCATION=`[*Leave this field blank/empty*{: style="color: #f00;"}]
    * `IRIS_KEY_EXPIRE_MINUTES=`[*Number of minutes before the IRiS keys expire*{: style="color: #f00;"}]
    * `USER_GUIDE_LOCATION=`[*Path to **TSS_User_Guide.docx***{: style="color: #f00;"}]
    * `SCORE_SUBMITTED_MESSAGE=`[*Message returned when a score is submitted*{: style="color: #f00;"}]
    * `ACCESS_DENIED=`[*Message returned when access is denied*{: style="color: #f00;"}]
    * `SHOW_STATUS=`[*Boolean to determine if status should be displayed*{: style="color: #f00;"}]
    * `MinLogLevel=`[*Minimum level for which logs should be created*{: style="color: #f00;"}]
    * `COOKIE_TIMEOUT_MINS=`[*Number of minutes before THSS cookies are expired*{: style="color: #f00;"}]


* Example of configured `settings.config` file:

<div class="highlighter-rouge">
<pre class="highlight">
<code>&lt;?xml version="1.0"?&gt;
    &lt;appSettings&gt;
        &lt;add key="webpages:Version" value="2.0.0.0" /&gt;
        &lt;add key="webpages:Enabled" value="false" /&gt;
        &lt;add key="PreserveLoginUrl" value="true" /&gt;
        &lt;add key="ClientValidationEnabled" value="false" /&gt;
        &lt;add key="UnobtrusiveJavaScriptEnabled" value="false" /&gt;

        &lt;add key="EMAIL_AS_UUID" value="true"/&gt;
        &lt;add key="ART_API_REST_API_BASE_URL" value="<span class="placeholder-example">http://54.186.87.166:8080</span>/rest" /&gt;
        &lt;add key="ART_API_URL" value="<span class="placeholder-example">http://54.186.87.166:8080</span>/rest/user" /&gt;
        &lt;add key="ART_API_CLIENT" value="<span class="placeholder-example">SBAC_PT</span>"/&gt;
        &lt;add key="ART_OAUTH_URL" value="https://<span class="placeholder-example">sso-deployment.sbtds.org:443</span>/auth/oauth2access_token?realm=/sbac"/&gt;
        &lt;add key="ART_OAUTH_PASSWORD_GRANTTYPE" value="<span class="placeholder-example">true</span>" /&gt;
        &lt;add key="ART_OAUTH_USERNAME" value="<span class="placeholder-example">prime.user@example.com</span>"/&gt;
        &lt;add key="ART_OAUTH_REQUIRED" value="<span class="placeholder-example">true</span>"/&gt;
        &lt;add key="ART_OAUTH_PASSWORD" value="<span class="placeholder-example">[redacted]</span>"/&gt;
        &lt;add key="ART_OAUTH_SECRET" value="<span class="placeholder-example">[redacted]</span>"/&gt;
        &lt;add key="ART_OAUTH_CLIENTID" value="<span class="placeholder-example">pm</span>"/&gt;


        &lt;add key="IGNORE_TENANCY_CHAINS" value="<span class="placeholder-example">False</span>"/&gt;
        &lt;add key="SAML_OWNER_PREFIX" value="<span class="placeholder-example">sbac</span>"/&gt;
        &lt;add key="SAML_SESSIONREFRESH_URL" value="https://<span class="placeholder-example">sso-deployment.sbtds.org</span>/auth/identity/attributes?refresh=true"/&gt;
        &lt;add key="SAML_REDIRECT" value="<span class="placeholder-example">http://54.200.42.254/</span>"/&gt;
        &lt;add key="PERMISSIONS_SCHEMA_URL" value="<span class="placeholder-example">http://54.213.111.234:8080</span>/rest/role?component=Teacher Hand Scoring System"/&gt;

        &lt;!-- It is possible to load a pre-configured set of permissions from below instead of using the permissions API. Set LOAD_PERMISSIONS_FROM_LOCAL to true and load a role.json file in the App_Config folder. See Permissions API for JSON structure. --&gt;
        &lt;add key="LOAD_PERMISSIONS_FROM_LOCAL" value="<span class="placeholder-example">false</span>"/&gt;
        &lt;add key="ART_ENTITIES_DATA_CACHING_DAYS" value="<span class="placeholder-example">1</span>" /&gt;
        &lt;add key="ART_SCORER_DATA_CACHING_MINS" value="<span class="placeholder-example">30</span>"/&gt;
        &lt;add key="IRIS_OPEN_SOURCE" value="<span class="placeholder-example">True</span>"/&gt;
        &lt;add key="IRIS_VENDOR_ID" value="<span class="placeholder-example">2B3C34BF-064C-462A-93EA-41E9E3EB8333</span>" /&gt;
        &lt;add key="IRIS_ROOT_URL" value="<span class="placeholder-example">http://54.186.182.136:8080/IRiS/</span>"/&gt;
        &lt;add key="IRISBlackbox_ROOT_URL" value="<span class="placeholder-example">http://54.186.182.136:8080/IRiS/</span>"/&gt;

        &lt;add key="IRIS_PEM_LOCATION" value="" /&gt;
        &lt;add key="IRIS_KEY_EXPIRE_MINUTES" value="<span class="placeholder-example">30</span>" /&gt;

        &lt;add key="USER_GUIDE_LOCATION" value="<span class="placeholder-example">/content/UserGuide</span>/TSS_User_Guide.docx"/&gt;
        &lt;add key="SCORE_SUBMITTED_MESSAGE" value="<span class="placeholder-example">The score has been saved.</span>" /&gt;
        &lt;add key="ACCESS_DENIED" value="<span class="placeholder-example">You are not Authorized User</span>"/&gt;
        &lt;add key="SHOW_STATUS" value="<span class="placeholder-example">true</span>" /&gt;

        &lt;!-- Minimum level for which logs should be created --&gt;
        &lt;add key="MinLogLevel" value="<span class="placeholder-example">Info</span>" /&gt;
        &lt;add key="COOKIE_TIMEOUT_MINS" value="<span class="placeholder-example">30</span>"/&gt;
    &lt;/appSettings&gt;</code>
</pre>
</div>

[back to Deployment Checklists](index.html)
