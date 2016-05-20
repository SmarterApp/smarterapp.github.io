## SAML Service Provider Registration
* Launch OpenAM
* Log in with appropriate credentials
* Click on **Register Remote Service Provider**
* On the **Create a SAMLv2 Remote Service Provider** page:
  * Select the **/sbac** realm
  * Verify the **URL** option button is checked/selected
  * Enter the `/saml/metadata` endpoint for the desired component in the **URL** field
    * Example:  enter `http://`<span class="placeholder-example">54.213.81.243:8080</span>`/saml/metadata` in the **URL** field
  * Under the **Circle of Trust**
    * Verify the **Add to existing** option button is checked/selected
    * Verify **sbac** is the selected value for the **Existing Circle of Trust** dropdown list
  * Click the **Configure** button (upper righthand corner, across from the **Create a SAMLv2 Remote Service Provider** header)

#### Verify the Service Provider is Configured
* Click on the **Federation** tab
* Observe the following:
  * The **Circle of Trust** table contains a record that represents the component that was added
  * The **Entity Providers** table conains a record with a **Name** equal to the **entityId** set in the component's SAML metadata file