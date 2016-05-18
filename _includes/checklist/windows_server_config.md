* Connect to the Windows server via Remote Desktop
  * [Instructions for connecting to an AWS Windows server](http://docs.aws.amazon.com/AWSEC2/latest/WindowsGuide/connecting_to_windows_instance.html)
* Click on **Server Manager** (in toolbar next to **Start Button**)
* Click on **Add Roles and Features**
* Follow the wizard prompts:
* Choose **Role-based**:

  ![Choose Role-based option](/res/images/checklist/add-roles-features-01-role-based.png){: width="80%" }

* Leave **select a server from the server pool** selected (should show the name of the current server):

  ![Choose server from pool](/res/images/checklist/add-roles-features-02-choose-server.png){: width="80%" }

* Scroll down and check the **Web Server (IIS)** role:
  * A dialog will appear (shown to the right in the screenshot) to confirm adding the feature.  Click **Add Features** to confirm
  * Check the **Application Developmen** checkbox beneath the **Web Server (IIS)** role
  * Choose to install **ASP.NET 4.5**
  * <span style="background-color: #ff0;">Get New SCREENSHOT</span>

  ![Choose Web Server Role](/res/images/checklist/add-roles-features-03-web-server-role.png){: width="85%" }

* Expand the **.NET Framework 4.5 Features** and check the **ASP.NET 4.5** checkbox:

  ![Add ASP.NET 4.5](/res/images/checklist/add-roles-features-04-dotnet-45.png){: width="80%" }

* Click **Next** on the information screen to continue with the server's configuration:

  ![Information about web server role](/res/images/checklist/add-roles-features-05-notification.png){: width="80%" }

* Review the selected features and click **Next** once satisfied:

  ![Show selected features](/res/images/checklist/add-roles-features-06-show-selected-features.png){: width="80%" }

* Click **Install** to add the required features to this server:

  ![Install selected features](/res/images/checklist/add-roles-features-07-install.png){: width="80%" }

* After the features have been installed, click **Close** to exit the wizard
