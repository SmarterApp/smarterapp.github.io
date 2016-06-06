### Set Up a Keystore
* Create resources directory and child directories:
  * `sudo mkdir -p /var/lib/tomcat7/resources/{progman,security}`
  * `sudo chown -R tomcat7:tomcat7 /var/lib/tomcat7/resources/`
* Create the wildcard SSL cert public key (*.sbtds.org):
  * `sudo vi /var/lib/tomcat7/resources/security/sbtds_org.cer`
* Copy the certificate contents (including the BEGIN CERTIFICATE and END CERTIFICATE lines) into `/var/lib/tomcat7/resources/security/sbtds_org.cer`
  * Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>-----BEGIN CERTIFICATE-----
<span class="placeholder-example">// This is where the certificate content is</span>
-----END CERTIFICATE-----</code>
</pre>
</div>

* Create the keystore (***NOTE:*** the keystore file must be named **samlKeystore.jks**):
  * `cd /var/lib/tomcat7`
  * `sudo keytool -importcert -alias `[<*A meaningful alias*{: style="color: #f00;"}]` -keystore ./resources/security/samlKeystore.jks -file ./resources/security/`[*name of certificate file*{: style="color: #f00;"}]
    * Example: `sudo keytool -importcert -alias `<span class="placeholder-example">sbtdsorg</span>` -keystore ./resources/security/samlKeystore.jks -file ./resources/security/sbtds_org.cer`
    * provide password
    * Type `yes` when prompted to trust the certificate
* Generate the private key:
  * `sudo keytool -genkey -alias `[*choose a meaningful alias*{: style="color: #f00;"}]` -keyalg RSA -keystore `[*path/to/keystore*{: style="color: #f00;"}]`  -keysize 2048`
    * Example: `sudo keytool -genkey -alias `<span class="placeholder-example">proctor-saml-sp</span>` -keyalg RSA -keystore ./resources/security/samlKeystore.jks  -keysize 2048`
  * Provide the password to the keystore created previously.
  * Answer the prompts.  Example of the command and prompts shown below:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>sudo keytool -genkey -alias progman-saml-sp -keyalg RSA -keystore ./resources/security/samlKeystore.jks  -keysize 2048
Enter keystore password:
What is your first and last name?
  [Unknown]:  <span class="placeholder-example">ProgMan Component</span>
What is the name of your organizational unit?
  [Unknown]:  <span class="placeholder-example">sbac</span>
What is the name of your organization?
  [Unknown]:  <span class="placeholder-example">SBAC</span>
What is the name of your City or Locality?
  [Unknown]:  <span class="placeholder-example">San Diego</span>
What is the name of your State or Province?
  [Unknown]:  <span class="placeholder-example">California</span>
What is the two-letter country code for this unit?
  [Unknown]:  <span class="placeholder-example">US</span>
Is CN=ProgMan Component, OU=sbac, O=SBAC, L=San Diego, ST=California, C=US correct?
  [no]:  <span class="placeholder-example">yes</span></code>
</pre>
</div>

#### Verify Keystore Contents
* To view the keystore contnets, use the following command:
  * `sudo keytool -list -keystore `[*path/to/samlKeystore.jks*{: style="color: #f00;"}]
    * Example:  `sudo keytool -list -keystore `<span class="placeholder-example">/var/lib/tomcat7/resources/security</span>`/samlKeystore.jks`
  * Output will be similar to the following (after providing the correct password):

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>Enter keystore password:

Keystore type: JKS
Keystore provider: SUN

Your keystore contains 2 entries

<span class="placeholder-example">sbtdsorg</span>, Apr 6, 2016, trustedCertEntry,
Certificate fingerprint (SHA1): D6:06:FA:33:AB:E4:27:26:D5:E1:B2:AB:1E:1D:FF:1E:7E:C0:21:4F
<span class="placeholder-example">progman-saml-sp</span>, Apr 6, 2016, PrivateKeyEntry,
Certificate fingerprint (SHA1): 8D:3A:66:1D:0C:7B:0A:40:96:B7:A6:8F:13:27:AB:E8:05:7D:8D:3A</code>
</pre>
</div>

#### Additional Notes
* Common keystore commands can be found [here](https://www.sslshopper.com/article-most-common-java-keytool-keystore-commands.html)