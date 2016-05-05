---
title: OpenDJ Installation Checklist
permalink: "/deployment/checklist/opendj.html"
layout: "document"
categories: ["deployment", "checklist"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | LDAP server for TDS user accounts |
| Communicates With | OpenAM, ART |
| Repository Location | [https://bitbucket.org/sbacoss/opendj_release](https://bitbucket.org/sbacoss/opendj_release) |
| Additional Documentation | [SBAC OpenDJ Installation](https://bitbucket.org/sbacoss/opendj_release/src/6f5a11b091f5304189c4f1eb54e03aeb98c44162/sbacInstaller/sbacOpenDJ-Installation-12312013.pdf?at=default), [SBAC SSO Design](https://bitbucket.org/sbacoss/opendj_release/src/6f5a11b091f5304189c4f1eb54e03aeb98c44162/SBAC_SSO_Design-v1.10-03282014.pdf?at=default) |

## Instructions

### Create AWS Instance
* Create server instance to host OpenDJ software
  * AWS instance type must be at least **t2.medium**
  * Select an image with the **Ubuntu 14.04 LTS 64-bit**{: style="color: #04384e"} operating system
* Create an AWS security group with the following ports for inbound TCP traffic (can be done during instance creation):
  * 22
  * 1389
  * 4444
  * 4989
  * 8989

### Install OpenDJ on AWS Instance
* Update package manager:
  * `sudo apt-get update`
  * `sudo apt-get upgrade -y`
* Install packages to satisfy dependencies:
  * `sudo apt-get install -y make perl liblinux-inotify2-perl libwww-perl cpanminus unzip mercurial software-properties-common ntp`
* Add repository and install Java 6 JDK using Oracle Java Installer:
  * `sudo add-apt-repository ppa:webupd8team/java -y`
  * `sudo apt-get update`
  * `sudo apt-get install -y oracle-java6-installer`
* Install Perl modules to satisfy dependencies:
  * `sudo cpanm Net::LDAP Net::SMTP File::Copy LWP::UserAgent HTTP::Request`
* Create `dropbox` user account and home directory:
  * `sudo useradd -d [path to where user XML files are uploaded] -m dropbox -p [choose a password] -s /bin/bash`
* Clone the `opendj_release` repository from the Smarter Balanced BitBucket to this server:
  * `hg clone https://[your BitBucket user or team name]/sbacoss/opendj_release`
  * Example:
    * `hg clone https://jjohnson-fw@bitbucket.org/sbacoss/opendj_release`
* Copy SBAC OpenDJ installer and content to the /opt directory:
  * `sudo cp -R opendj_release/sbacInstaller/* /opt`
* Execute the SBAC OpenDJ installer:
  * `cd /opt`
  * `sudo ./installOpenDJ.sh`
* Set up OpenDJ to run when server starts up:
  * `cd /opt/opendj/bin/`
  * `sudo ./create-rc-script -f /etc/init.d/opendj -u opendj`
  * `cd /etc/init.d/`
  * `sudo update-rc.d opendj defaults`
* Update `/opt/scripts/sbacWatchXMLFolder.pl` to monitor the correct dropbox user directory:
  * `my $inputXMLFileDir    = "`[*path to where user XML files are uploaded*{: style="color: red;"}]`";`
* Update `/opt/scripts/sbacProcessXML.pl` with appropriate configuration values:
  * `my $inputXMLFileDir   = "`[*path to where user XML files are uploaded*{: style="color: red;"}]`";`
  * `my $processedFileDir  = "`[*path where user XML files are stored after they are processed*{: style="color: red;"}]`";`
  * `my $httpResponseServer = "`[*HTTP server for a callback response, does not need to be set*{: style="color: red;"}]`";`
  * `my $ldapHost           = "`[*Name of the OpenDJ server.  Can be set to **localhost***{: style="color: red;"}]`";`
  * `my $ldapPort           = "`[*Port the OpenDJ server listens on.  The standard port is **1389***{: style="color: red;"}]`";`                           # port number of the OpenDJ server
  * `my $ldapBindDN        = "`[*OpenDJ service account or rootDN with sufficient permission, valid value after OpenDJ installation: **cn=SBAC Admin** (be sure to include the **cn=** as part of the value; see example below)*{: style="color: red;"}]`";`
  * `my $ldapBindPass      = "`[*password for OpenDJ service account or rootDN, valid value after OpenDJ installation: **cangetin***{: style="color: red;"}]`";`
  * `my $fromAddress       = '`[*email address that user notification messages should be from*{: style="color: red;"}]`';`
  * `my $fromPerson        = `[*display name that user notification messages should be from*{: style="color: red;"}]`';`
  * `my $emailAddrOverride = '`[*when $emailOverride flag is set, send recipient's email to this address*{: style="color: red;"}]`';`
  * `my $adminEmail        = '`[*email address of user who is monitoring script results*{: style="color: red;"}]`';`
  * `my $emailServer       = "`[*name of email server*{: style="color: red;"}]`";`
  * `my $defaultPassword   = "`[*desired default password for test users*{: style="color: red;"}]`";`

* Example of configured `/opt/scripts/sbacWatchXMLFolder.pl`:

~~~~
my $inputXMLFileDir    = "/opt/dropbox";         # folder where the XML files are uploaded
my $processedFileDir   = "/opt/scripts/sbacXMLFiles";      # folder where the XML files are stored after processing
my $httpResponseServer = "https://www.example.com/callback/";   # HTTP server for callback response
my $ldapHost           = "localhost";                      # host name of the OpenDJ server
my $ldapPort           = "1389";                           # port number of the OpenDJ server
my $ldapBindDN         = "cn=SBAC Admin";                  # replace with the bindDN of a service account or rootDN with permissions
my $ldapBindPass       = "cangetin";                  # replace with password of the OpenDJ service account

my $ldapBaseDN         = "ou=People,dc=smarterbalanced,dc=org";   # location where the users may be found
my $ldapTimeout        = "10";                             # how long to wait for a connection to the LDAP server before timing out

# Email Variables - these variables are specific to subroutines which generate emails

my $fromAddress       = 'Smarter-DoNotReply@example.com';  # all email will come from this email address
my $fromPerson        = 'Smarter-DoNotReply';              # the name of the person sending the email
my $emailAddrOverride = 'bill.nelson@identityfusion.com';  # when $emailOverride flag is set, send recipient's email to this addr
my $adminEmail        = 'bill.nelson@identityfusion.com';  # email address of user who is monitoring script results
my $emailServer       = "mail.example.com";                # replace with your email server
my $defaultPassword   = "password123";                       # default password for test users
~~~~

* Start `/opt/scripts/sbacWatchXMLFolder.pl` as a background process:
  * `sudo perl sbacWatchXMLFolder.pl &`
* Create script that will run `/opt/scripts/sbacWatchXMLFolder.pl` as a background process as opendj user on startup:
  * `cd /etc/init.d`
  * `sudo vi sbac-userwatch.sh`
  * Copy the following code into `/etc/init.d/sbac-userwatch.sh`:

    ~~~~ shell
    #!/bin/sh
    su -c "perl /opt/scripts/sbacWatchXMLFolder.pl" opendj &
    ~~~~

  * Make the script executable:
    * `sudo chmod +x /etc/init.d/sbac-userwatch.sh`
* Set the `sbac-userwatch.sh` script to run when the server starts up:
  * `cd /etc/init.d`
  * `sudo update-rc.d sbac-userwatch.sh defaults`

### Install/Configure SFTP Server
* `sudo apt-get install -y openssh-server`
* Add an SFTP access group:
  * `sudo addgroup sftpusers`


## Verification
* Connect to the OpenDJ instance with any client (e.g. [Apache Directory Studio](https://directory.apache.org/studio/))

### Create Prime User Account
* Create an XML file named `prime_user_testfile_.xml` with the following contents:

~~~~ xml
<?xml version='1.0' encoding='UTF-8'?>
<Users>
<User Action="ADD">
  <UUID>CREATE-UNIQUE_UUID_HERE</UUID>
  <FirstName>CHOOSE-FIRST_NAME</FirstName>
  <LastName>CHOOSE-LAST_NAME</LastName>
  <Email>temp-bootstrap@example.com</Email>
  <Phone/>
  <Role>
    <RoleID></RoleID>
    <Name>Administrator</Name>
    <Level>CLIENT</Level>
    <ClientID>CHOOSE-CLIENT_IDENTIFIER_NUMBER</ClientID>
    <Client>CHOOSE-UNIQUE_CLIENT_NAME</Client>
    <GroupOfStatesID/>
    <GroupOfStates/>
    <StateID/>
    <State/>
    <GroupOfDistrictsID/>
    <GroupOfDistricts/>
    <DistrictID/>
    <District/>
    <GroupOfInstitutionsID/>
    <GroupOfInstitutions/>
    <InstitutionID/>
    <Institution/>
  </Role>
</User>
</Users>
~~~~

An example of the `prime_user_testfile_.xml` file:

~~~~ xml
<?xml version='1.0' encoding='UTF-8'?>
<Users>
<User Action="ADD">
  <UUID>2503a564-fde8-11e5-86aa-5e5517507c66</UUID>
  <FirstName>Prime</FirstName>
  <LastName>User</LastName>
  <Email>prime.user@example.com</Email>
  <Phone/>
  <Role>
    <RoleID></RoleID>
    <Name>Administrator</Name>
    <Level>CLIENT</Level>
    <ClientID>98765</ClientID>
    <Client>PRIME_USER_CLIENT</Client>
    <GroupOfStatesID/>
    <GroupOfStates/>
    <StateID/>
    <State/>
    <GroupOfDistrictsID/>
    <GroupOfDistricts/>
    <DistrictID/>
    <District/>
    <GroupOfInstitutionsID/>
    <GroupOfInstitutions/>
    <InstitutionID/>
    <Institution/>
  </Role>
</User>
</Users>
~~~~

* Copy or move the `prime_user_testfile_.xml` file to the dropbox directory that is monitored by the `/opt/scripts/sbacWatchXMLFolder.pl` script
  * **NOTE:** The `opendj` user (or whatever account is running the `sbacWatchXMLFolder.pl` script) must be able to read the `prime_user_testfile_.xml` file
  * **NOTE:** If the `prime_user_testfile_.xml` is created in the dropbox directory, run `touch /opt/dropbox/prime_user_testfile_.xml` to update the timestamp on the file.
* Connect to OpenDJ with any client (e.g. [Apache Directory Studio](https://directory.apache.org/studio/)) and verify the Prime User account was created

[back to Deployment Checklists](index.html)