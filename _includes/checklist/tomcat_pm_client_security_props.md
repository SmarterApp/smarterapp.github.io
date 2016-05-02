* Create a `pm-client-security.properties` file in `/var/lib/tomcat7/resources/progman`
* Copy the following into `/var/lib/tomcat7/resources/progman/pm-client-security.properties`:

~~~~
oauth.access.url=https://[FQDN or IP address of OpenAM server]/auth/oauth2/access_token?realm=/sbac
pm.oauth.client.id=[OAuth client id from OpenAM]
pm.oauth.client.secret=[OAuth client secret from OpenAM]
pm.oauth.batch.account=[User account in OpenDJ]
pm.oauth.batch.password=[Password for OpenDJ user account]
~~~~

* Example:

~~~~
oauth.access.url=https://sso-dev.sbtds.org/auth/oauth2/access_token?realm=/sbac
pm.oauth.client.id=pm
pm.oauth.client.secret=[redacted]
pm.oauth.batch.account=prime.user@example.com
pm.oauth.batch.password=[redacted]
~~~~

* Update ownership for directories and files in the `/var/lib/tomcat7/resources/` directory:
  * `sudo chown -R tomcat7:tomcat7 /var/lib/tomcat7/resources`