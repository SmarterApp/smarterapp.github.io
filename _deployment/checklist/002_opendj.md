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
* Add a record set to AWS [Route 53](http://docs.aws.amazon.com/Route53/latest/DeveloperGuide/resource-record-sets-creating.html?console_help=true):
    * Choose a meaningful name
    * Type: CNAME
    * TTL: 300 seconds (default value)
    * Value: [*DNS Name of the AWS instance*{: style="color: #f00;"}]
    * Routing Policy: Simple

### Configure SFTP Server for ART -> OpenDJ Integration on AWS Instance
* Verify the `openssh-sftp-server` is installed:
  * `sudo dpkg --get-selections | grep openssh-sftp-server`
  * Example output:

~~~~
ubuntu@opendj-deploy:/home/art_userftp$ sudo dpkg --get-selections | grep openssh-sftp-server
openssh-sftp-server       install
~~~~

  * If no result was returned, install the `openssh-server` using the following steps:
    * `sudo apt-get update`
    * `sudo apt-get install -y openssh-server`
* Create a new user group for SFTP users:
  * `sudo groupadd `[*meaningful name of group, e.g. **filetransfer** or **sftpusers***{: style="color: #f00;"}]
    * Example: `sudo groupadd sftpusers`
* Configure the SFTP server by editing `/etc/ssh/sshd_config`:
  * Add the following lines to the end of the file, taking care to preserve the indentation (i.e. add the lines *exactly* as they appear)

<div class="highlighter-rouge">
<pre class="highlight">
<code>Match group [<span class="placeholder">name of user group added previously</span>]
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    PasswordAuthentication yes</code>
</pre>
</div>

  * Update the `Match group` line, replacing `filetransfer` with the name of the group used when executing the `sudo groupadd` command

  * Example of lines to add to `/etc/ssh/sshd_config` using a group name of `filetransfer`:

~~~~
Match group filetransfer
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    PasswordAuthentication yes
~~~~

  * Example of lines to add to `/etc/ssh/sshd_config` using a different group name (`sftpusers` instead of `filetransfer`):

~~~~
Match group sftpusers
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    PasswordAuthentication yes
~~~~

* Restart the OpenSSH server:
  * `sudo service ssh restart`

#### Create SFTP User Account
* Create `dropbox` user account and home directory:
  * `sudo adduser dropbox`
* Add the `dropbox` user to the group created earlier:
  * `sudo usermod -G `[*The name of the group created earlier*{: style="color: #f00;"}]` dropbox`
    * Example:  `sudo usermod -G sftpusers dropbox`
* Change ownership of the `dropbox` user's home directory:
  * `sudo chown root:root /home/dropbox`
* Update permissions on the `dropbox` user's home directory:
  * `sudo chmod 0755 /home/dropbox`
* Create a directory where files will be put:
  * `sudo mkdir /home/dropbox/`[*A meaningful directory name*{: style="color: #f00;"}]
    * Example: `sudo mkdir /home/dropbox/sftpfiles`
* Update ownership on the directory and contents created above:
  * `sudo chown dropbox:dropbox /home/dropbox/*`
* ***OPTIONAL:*** Create a link to the directory where ART user XML files should be written to:
  * `sudo ln -s /home/drobox/`[*Directory name*{: style="color: #f00;"}]` `[*Logical path where SFTP files should be written*{: style="color: #f00;"}]
    * Example:  `sudo ln -s /home/dropbox/sftpfiles /opt/dropbox/sftp_root`

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
* Clone the `opendj_release` repository from the Smarter Balanced BitBucket to this server:
  * `hg clone https://`[*your BitBucket user or team name*{: style="color: #f00;"}]`/sbacoss/opendj_release`
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

### Update Perl Scripts That Process User Data

#### Update sbacWatchXMLFolder.pl
* Update `/opt/scripts/sbacWatchXMLFolder.pl` to monitor the correct dropbox user directory:
  * `my $inputXMLFileDir    =  "`[*path to where user XML files are uploaded.  This is the **dropbox** user's directory or the link to that directory created earlier*{: style="color: red;"}]`";`
* Example of configured `/opt/scripts/sbacWatchXMLFolder.pl`:

~~~~
my $inputXMLFileDir    = "/opt/dropbox/sftp_root";
~~~~

#### Update sbacProcessXML.pl
* Update `/opt/scripts/sbacProcessXML.pl` with appropriate configuration values:
  * `my $inputXMLFileDir   = "`[*path to where user XML files are uploaded.  This is the **dropbox** user's directory or the link to that directory created earlier*{: style="color: red;"}]`";`
  * `my $processedFileDir  = "`[*path where user XML files are stored after they are processed*{: style="color: red;"}]`";`
  * `my $httpResponseServer = "`[*HTTP server for a callback response, does not need to be set*{: style="color: red;"}]`";`
  * `my $ldapHost           = "`[*Name of the OpenDJ server.  Can be set to **localhost***{: style="color: red;"}]`";`
  * `my $ldapPort           = "`[*Port the OpenDJ server listens on.  The standard port for this installtation is **1389***{: style="color: red;"}]`";`                           # port number of the OpenDJ server
  * `my $ldapBindDN        = "`[*OpenDJ service account or rootDN with sufficient permission, valid value after OpenDJ installation: **cn=SBAC Admin** (be sure to include the **cn=** as part of the value; see example below)*{: style="color: red;"}]`";`
  * `my $ldapBindPass      = "`[*password for OpenDJ service account or rootDN, valid value after OpenDJ installation: **cangetin***{: style="color: red;"}]`";`
  * `my $fromAddress       = '`[*email address that user notification messages should be from*{: style="color: red;"}]`';`
  * `my $fromPerson        = `[*display name that user notification messages should be from*{: style="color: red;"}]`';`
  * `my $emailAddrOverride = '`[*when $emailOverride flag is set, send recipient's email to this address*{: style="color: red;"}]`';`
  * `my $adminEmail        = '`[*email address of user who is monitoring script results*{: style="color: red;"}]`';`
  * `my $emailServer       = "`[*name of email server*{: style="color: red;"}]`";`
  * `my $defaultPassword   = "`[*desired default password for test users*{: style="color: red;"}]`";`

* Example of configured `/opt/scripts/sbacProcessXML.pl`:

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
* Create an XML file named `prime_user_testfile_.xml` with the content shown below.  Replace the following:
  * `CREATE-UNIQUE_UUID_HERE` should be a unique identifier (e.g. a GUID from an [Online GUID Generator](https://www.guidgenerator.com/))
  * `CHOOSE-FIRST_NAME` should be a meaningful first name
  * `CHOOSE-LAST_NAME` should be a meaningful last name
  * `CHOOSE-EMAIL` should be replaced with an easy to remember email adddress
  * `CHOOSE-CLIENT_IDENTIFIER_NUMBER` should be replaced with a meaningful Client ID.
    * This can be a string value, but is typically a number
  * `CHOOSE-UNIQUE_CLIENT_NAME` should be replaced with a meaningful
    * This can be a string value and can be the same as the value chosen for `CHOOSE-CLIENT_IDENTIFIER_NUMBER`

* The content of the `prime_user_testfile_.xml`:

<div class="highlighter-rouge">
<pre class="highlight"><code><span class="cp">&lt;?xml version='1.0' encoding='UTF-8'?&gt;</span>
<span class="nt">&lt;Users&gt;</span>
<span class="nt">&lt;User</span> <span class="na">Action=</span><span class="s">"ADD"</span><span class="nt">&gt;</span>
  <span class="nt">&lt;UUID&gt;</span>[<span class="placeholder">CREATE-UNIQUE_UUID_HERE</span>]<span class="nt">&lt;/UUID&gt;</span>
  <span class="nt">&lt;FirstName&gt;</span>[<span class="placeholder">CHOOSE-FIRST_NAME</span>]<span class="nt">&lt;/FirstName&gt;</span>
  <span class="nt">&lt;LastName&gt;</span>[<span class="placeholder">CHOOSE-LAST_NAME</span>]<span class="nt">&lt;/LastName&gt;</span>
  <span class="nt">&lt;Email&gt;</span>[<span class="placeholder">CHOOSE-EMAIL</span>]<span class="nt">&lt;/Email&gt;</span>
  <span class="nt">&lt;Phone/&gt;</span>
  <span class="nt">&lt;Role&gt;</span>
    <span class="nt">&lt;RoleID&gt;&lt;/RoleID&gt;</span>
    <span class="nt">&lt;Name&gt;</span>Administrator<span class="nt">&lt;/Name&gt;</span>
    <span class="nt">&lt;Level&gt;</span>CLIENT<span class="nt">&lt;/Level&gt;</span>
    <span class="nt">&lt;ClientID&gt;</span>[<span class="placeholder">CHOOSE-CLIENT_IDENTIFIER_NUMBER</span>]<span class="nt">&lt;/ClientID&gt;</span>
    <span class="nt">&lt;Client&gt;</span>[<span class="placeholder">CHOOSE-UNIQUE_CLIENT_NAME</span>]<span class="nt">&lt;/Client&gt;</span>
    <span class="nt">&lt;GroupOfStatesID/&gt;</span>
    <span class="nt">&lt;GroupOfStates/&gt;</span>
    <span class="nt">&lt;StateID/&gt;</span>
    <span class="nt">&lt;State/&gt;</span>
    <span class="nt">&lt;GroupOfDistrictsID/&gt;</span>
    <span class="nt">&lt;GroupOfDistricts/&gt;</span>
    <span class="nt">&lt;DistrictID/&gt;</span>
    <span class="nt">&lt;District/&gt;</span>
    <span class="nt">&lt;GroupOfInstitutionsID/&gt;</span>
    <span class="nt">&lt;GroupOfInstitutions/&gt;</span>
    <span class="nt">&lt;InstitutionID/&gt;</span>
    <span class="nt">&lt;Institution/&gt;</span>
  <span class="nt">&lt;/Role&gt;</span>
<span class="nt">&lt;/User&gt;</span>
<span class="nt">&lt;/Users&gt;</span></code>
</pre>
</div>

An example of the `prime_user_testfile_.xml` file with placeholders replaced by example values:

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

###  Verify SFTP Connectivity
* Connect to the SFTP server using an FTP client (e.g. [Cyberduck](https://cyberduck.io/) or [FileZilla](https://filezilla-project.org/)) using the user account created while following this checklist
* Verify the user can write a file to the desired directory

[back to Deployment Checklists](index.html)