* Create a `pm-client-security.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/pm-client-security.properties`:

<div class="highlighter-rouge" style="display: inline-flex;">
<pre class="highlight">
<code>oauth.access.url=https://[<span class="placeholder">FQDN or IP address of OpenAM server</span>]/auth/oauth2/access_token?realm=/sbac
pm.oauth.client.id=[<span class="placeholder">OAuth client id from OpenAM</span>]
pm.oauth.client.secret=[<span class="placeholder">OAuth client secret from OpenAM</span>]
pm.oauth.batch.account=[<span class="placeholder">User account in OpenDJ</span>]
pm.oauth.batch.password=[<span class="placeholder">Password for OpenDJ user account</span>]</code>
</pre>
</div>

* Example:

<div class="highlighter-rouge">
<pre class="highlight">
<code>oauth.access.url=https://<span class="placeholder-example">sso-dev.sbtds.org</span>/auth/oauth2/access_token?realm=/sbac
pm.oauth.client.id=<span class="placeholder-example">pm</span>
pm.oauth.client.secret=<span class="placeholder-example">[redacted]</span>
pm.oauth.batch.account=<span class="placeholder-example">prime.user@example.com</span>
pm.oauth.batch.password=<span class="placeholder-example">[redacted]</span></code>
</pre>
</div>

* Update ownership for directories and files in the `/var/lib/tomcat7/resources/` directory:
  * `sudo chown -R tomcat7:tomcat7 /var/lib/tomcat7/resources`