### Set Up a Keystore
* Create resources directory and child directories:
  * `sudo mkdir -p /var/lib/tomcat7/resources/{progman,security}`
  * `sudo chown -R tomcat7:tomcat7 /var/lib/tomcat7/resources/`
* Create the wildcard SSL cert public key (*.sbtds.org):
  * `sudo vi /var/lib/tomcat7/resources/security/sbtds_org.cer`
* Copy the certificate contents (including the BEGIN CERTIFICATE and END CERTIFICATE lines) into `/var/lib/tomcat7/resources/security/sbtds_org.cer`
  * Example:

~~~~
-----BEGIN CERTIFICATE-----
// This is where the certificate content is
cTwSq+bBCCAPWCvsaJg=
-----END CERTIFICATE-----
~~~~

* Create the keystore (***NOTE:*** the keystore file must be named **samlKeystore.jks**):
  * `cd /var/lib/tomcat7`
  * `sudo keytool -importcert -alias sbtdsorg -keystore ./resources/security/samlKeystore.jks -file ./resources/security/`[*name of certificate file*{: style="color: #f00;"}]
    * Example: `sudo keytool -importcert -alias sbtdsorg -keystore ./resources/security/samlKeystore.jks -file ./resources/security/sbtds_org.cer`
    * provide password (been using _password123_)
    * Type `yes` when prompted to trust the certificate
* Generate the private key:
  * `sudo keytool -genkey -alias `[*choose a meaningful alias*{: style="color: #f00;"}]` -keyalg RSA -keystore `[*path/to/keystore*{: style="color: #f00;"}]`  -keysize 2048`
    * Example: `sudo keytool -genkey -alias proctor-saml-sp -keyalg RSA -keystore ./resources/security/samlKeystore.jks  -keysize 2048`
  * Provide the password to the keystore created previously.
  * Answer the prompts.  Example of the command and prompts shown below:

~~~~
sudo keytool -genkey -alias progman-saml-sp -keyalg RSA -keystore ./resources/security/samlKeystore.jks  -keysize 2048
Enter keystore password:
What is your first and last name?
  [Unknown]:  ProgMan Component
What is the name of your organizational unit?
  [Unknown]:  sbac
What is the name of your organization?
  [Unknown]:  SBAC
What is the name of your City or Locality?
  [Unknown]:  San Diego
What is the name of your State or Province?
  [Unknown]:  California
What is the two-letter country code for this unit?
  [Unknown]:  US
Is CN=ProgMan Component, OU=sbac, O=SBAC, L=San Diego, ST=California, C=US correct?
  [no]:  yes
~~~~

#### Verify Keystore Contents
* To view the keystore contnets, use the following command:
  * `sudo keytool -list -keystore `[*path/to/samlKeystore.jks*{: style="color: #f00;"}]
    * Example:  `sudo keytool -list -keystore /var/lib/tomcat7/resources/security/samlKeystore.jks`
  * Output will be similar to the following (after providing the correct password):

~~~~
Enter keystore password:

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 2 entries

sbtdsorg, Apr 6, 2016, trustedCertEntry,
Certificate fingerprint (SHA1): D6:06:FA:33:AB:E4:27:26:D5:E1:B2:AB:1E:1D:FF:1E:7E:C0:21:4F
progman-saml-sp, Apr 6, 2016, PrivateKeyEntry,
Certificate fingerprint (SHA1): 8D:3A:66:1D:0C:7B:0A:40:96:B7:A6:8F:13:27:AB:E8:05:7D:8D:3A
~~~~

#### Additional Notes
* Common keystore commands can be found [here](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html)