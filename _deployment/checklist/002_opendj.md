---
title: OpenDJ Installation Checklist
permalink: "/deployment/checklist/opendj.html"
layout: "document"
categories: ["deployment", "checklist", "shared_services"]
---

## Overview

| Item | Description |
|:-----|:------------|
| Purpose | LDAP server for TDS user accounts |
| Communicates With | OpenAM<br>ART |
| Repository Location | [https://github.com/SmarterApp/IM_OpenDJ](https://github.com/SmarterApp/IM_OpenDJ){:target="_blank"} |
| Additional Documentation | [SBAC OpenDJ Installation](https://github.com/SmarterApp/IM_OpenDJ/blob/master/sbacInstaller/sbacOpenDJ-Installation-12312013.pdf){:target="_blank"}<br>[SBAC SSO Design](https://github.com/SmarterApp/IM_OpenDJ/blob/master/SBAC_SSO_Design-v1.10-03282014.pdf){:target="_blank"} |

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

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>
ubuntu@opendj-deploy:/home/art_userftp$ sudo dpkg --get-selections | grep openssh-sftp-server
openssh-sftp-server       install
</code>
</pre>
</div>


  * If no result was returned, install the `openssh-server` using the following steps:
    * `sudo apt-get update`
    * `sudo apt-get install -y openssh-server`
* Create a new user group for SFTP users:
  * `sudo groupadd `[*meaningful name of group, e.g. **filetransfer** or **sftpusers***{: style="color: #f00;"}]
    * Example: `sudo groupadd `<span class="placeholder-example">sftpusers</span>
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

<div class="highlighter-rouge">
<pre class="highlight">
Match group <span class="placeholder-example">filetransfer</span>
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    PasswordAuthentication yes
</pre>
</div>

  * Example of lines to add to `/etc/ssh/sshd_config` using a different group name (`sftpusers` instead of `filetransfer`):

<div class="highlighter-rouge">
<pre class="highlight">
Match group <span class="placeholder-example">sftpusers</span>
    ChrootDirectory %h
    X11Forwarding no
    AllowTcpForwarding no
    ForceCommand internal-sftp
    PasswordAuthentication yes
</pre>
</div>

* Restart the OpenSSH server:
  * `sudo service ssh restart`

#### Create SFTP User Account
* Create `dropbox` user account and home directory:
  * `sudo adduser dropbox`
* Add the `dropbox` user to the group created earlier:
  * `sudo usermod -G `[*The name of the group created earlier*{: style="color: #f00;"}]` dropbox`
    * Example:  `sudo usermod -G `<span class="placeholder-example">sftpusers</span>` dropbox`
* Change ownership of the `dropbox` user's home directory:
  * `sudo chown root:root /home/dropbox`
* Update permissions on the `dropbox` user's home directory:
  * `sudo chmod 0755 `<span class="placeholder-example">/home/dropbox</span>
* Create a directory where files will be put:
  * `sudo mkdir /home/dropbox/`[*A meaningful directory name*{: style="color: #f00;"}]
    * Example: `sudo mkdir `<span class="placeholder-example">/home/dropbox/sftpfiles</span>
* Update ownership on the directory and contents created above:
  * `sudo chown dropbox:dropbox /home/dropbox/*`
* ***OPTIONAL:*** Create a link to the directory where ART user XML files should be written to:
  * `sudo ln -s /home/drobox/`[*Directory name*{: style="color: #f00;"}]` `[*Logical path where SFTP files should be written*{: style="color: #f00;"}]
    * Example:  `sudo ln -s `<span class="placeholder-example">/home/dropbox/sftpfiles</span>` `<span class="placeholder-example">/opt/dropbox/sftp_root</span>

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
* Clone the `IM_OpenDJ` repository from the Smarter Balanced GitHub to this server:
  * `git clone https://github.com/`[*your GitHub user name*{: style="color: #f00;"}]`/SmarterApp/IM_OpenDJ.git` <br>Example:
    `git clone https://github.com/`<span class="placeholder-example">hansolo</span>`/SmarterApp/IM_OpenDJ.git`
* Copy SBAC OpenDJ installer and content to the /opt directory:
  * `sudo cp -R IM_OpenDJ/sbacInstaller/* /opt`
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

<div class="highlighter-rouge">
<pre class="highlight">
my $inputXMLFileDir    = "<span class="placeholder-example">/opt/dropbox/sftp_root</span>";
</pre>
</div>

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

<div class="highlighter-rouge" style="display:inline-flex;">
<pre class="highlight">
my $inputXMLFileDir    = "<span class="placeholder-example">/opt/dropbox</span>";         # folder where the XML files are uploaded
my $processedFileDir   = "<span class="placeholder-example">/opt/scripts/sbacXMLFiles</span>";      # folder where the XML files are stored after processing
my $httpResponseServer = "<span class="placeholder-example">https://www.example.com/callback/</span>";   # HTTP server for callback response
my $ldapHost           = "<span class="placeholder-example">localhost</span>";                      # host name of the OpenDJ server
my $ldapPort           = "<span class="placeholder-example">1389</span>";                           # port number of the OpenDJ server
my $ldapBindDN         = "<span class="placeholder-example">cn=SBAC Admin</span>";                  # replace with the bindDN of a service account or rootDN with permissions
my $ldapBindPass       = "<span class="placeholder-example">cangetin</span>";                  # replace with password of the OpenDJ service account

my $ldapBaseDN         = "<span class="placeholder-example">ou=People,dc=smarterbalanced,dc=org</span>";   # location where the users may be found
my $ldapTimeout        = "<span class="placeholder-example">10</span>";                             # how long to wait for a connection to the LDAP server before timing out

# Email Variables - these variables are specific to subroutines which generate emails

my $fromAddress       = '<span class="placeholder-example">Smarter-DoNotReply@example.com</span>';  # all email will come from this email address
my $fromPerson        = '<span class="placeholder-example">Smarter-DoNotReply</span>';              # the name of the person sending the email
my $emailAddrOverride = '<span class="placeholder-example">bill.nelson@identityfusion.com</span>';  # when $emailOverride flag is set, send recipient's email to this addr
my $adminEmail        = '<span class="placeholder-example">bill.nelson@identityfusion.com</span>';  # email address of user who is monitoring script results
my $emailServer       = "<span class="placeholder-example">mail.example.com</span>";                # replace with your email server
my $defaultPassword   = "<span class="placeholder-example">password123</span>";                       # default password for test users
</pre>
</div>

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

## Verification
* Connect to the OpenDJ instance with any client (e.g. [Apache Directory Studio](https://directory.apache.org/studio/){:target="_blank"})

### Create Prime User Account
* Create an XML file named `prime_user_testfile_.xml` with the content shown below.  Replace the following:
  * `CREATE-UNIQUE_UUID_HERE` should be a unique identifier (e.g. a GUID from an [Online GUID Generator](https://www.guidgenerator.com/){:target="_blank"})
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

<div class="highlighter-rouge">
<pre class="highlight">
&lt;?xml version='1.0' encoding='UTF-8'?>
&lt;Users&gt;
&lt;User Action="ADD"&gt;
  &lt;UUID&gt;<span class="placeholder-example">2503a564-fde8-11e5-86aa-5e5517507c66</span>&lt;/UUID&gt;
  &lt;FirstName&gt;<span class="placeholder-example">Prime</span>&lt;/FirstName&gt;
  &lt;LastName&gt;<span class="placeholder-example">User</span>&lt;/LastName&gt;
  &lt;Email&gt;<span class="placeholder-example">prime.user@example.com</span>&lt;/Email&gt;
  &lt;Phone/&gt;
  &lt;Role&gt;
    &lt;RoleID&gt;&lt;/RoleID&gt;
    &lt;Name&gt;Administrator&lt;/Name&gt;
    &lt;Level&gt;CLIENT&lt;/Level&gt;
    &lt;ClientID&gt;<span class="placeholder-example">98765</span>&lt;/ClientID&gt;
    &lt;Client&gt;<span class="placeholder-example">PRIME_USER_CLIENT</span>&lt;/Client&gt;
    &lt;GroupOfStatesID/&gt;
    &lt;GroupOfStates/&gt;
    &lt;StateID/&gt;
    &lt;State/&gt;
    &lt;GroupOfDistrictsID/&gt;
    &lt;GroupOfDistricts/&gt;
    &lt;DistrictID/&gt;
    &lt;District/&gt;
    &lt;GroupOfInstitutionsID/&gt;
    &lt;GroupOfInstitutions/&gt;
    &lt;InstitutionID/&gt;
    &lt;Institution/&gt;
  &lt;/Role&gt;
&lt;/User&gt;
&lt;/Users&gt;
</pre>
</div>

* Copy or move the `prime_user_testfile_.xml` file to the dropbox directory that is monitored by the `/opt/scripts/sbacWatchXMLFolder.pl` script
  * **NOTE:** The `opendj` user (or whatever account is running the `sbacWatchXMLFolder.pl` script) must be able to read the `prime_user_testfile_.xml` file
  * **NOTE:** If the `prime_user_testfile_.xml` is created in the `dropbox` directory, run `touch `[*path to dropbox directory*{: style="color: #f00;"}]`/prime_user_testfile_.xml` to update the timestamp on the file.
    * Example: `touch `<span class="placeholder-example">/opt/dropbox</span>`/prime_user_testfile_.xml`
* Connect to OpenDJ with any client (e.g. [Apache Directory Studio](https://directory.apache.org/studio/){:target="_blank"}) and verify the Prime User account was created

###  Verify SFTP Connectivity
* Connect to the SFTP server using an FTP client (e.g. [Cyberduck](https://cyberduck.io/){:target="_blank"} or [FileZilla](https://filezilla-project.org/){:target="_blank"}) using the user account created while following this checklist
* Verify the user can write a file to the desired directory

[back to Deployment Checklists](index.html)