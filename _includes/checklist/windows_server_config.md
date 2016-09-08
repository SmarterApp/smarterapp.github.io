
#### Configure the Web Server Components

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
  * Check the **Application Development** checkbox beneath the **Web Server (IIS)** role
  * Choose to install **ASP.NET 4.5**

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

#### Configure IIS Permissions for the Log Directory
***IMPORTANT:***{: style="color: #f00;"} *Repeat the steps below for each log file directory.  If you want applications to write to different log file directories, the **IIS_IUSRS** must be able to write to the desired directory/directories.*{: style="background-color: #ff0;"}

* Launch the **File Explorer**
* Navigate to the directory where the application(s) is/are configured to write log entires
  * Refer to the `LogFilePath value` in the `web.config`
  * Example: <span class="placeholder-example">C:\inetpub\logs</span>
* Right-click on the directory and choose **Properties**
* Click on the **Security** tab

![Edit log file directory security](/res/images/checklist/file_explorer_01_security_tab.png){: width="100%"}

* Click the **Edit** button
* Click the **Add...** button
* Enter **IIS_IUSRS** in the **Enter object names to select** field and click **Check Names**
  * The **IIS_IUSRS** should be updated to include the server name, indicating the user group was found.
* An example of the result after the **Check Names** button is clicked:

![Add IIS_IUSRS](/res/images/checklist/file_explorer_02_check_names.png){: width="100%"}

* Click **OK**
* Give the **IIS_IUSRS** **Write** permission to the log directory:
  * Scroll down in the list of available permissions
  * Check the box next to **Write**
* An example of granting **Write** permission to the **IIS_IUSRS** group:

![Add IIS_IUSRS](/res/images/checklist/file_explorer_03_add_write_perms.png){: width="75%"}

* Click **OK**
* Click **OK**

#### Add Inbound and Outbound Firewall Rules to Allow Communicating with SQL Server
* Launch the **Windows Firewall with Advanced Security** pane
* While viewing the **Inbound Rules**, click on **New Rule...**
* Set the **Rule Type** to **Port**:

  ![Install selected features](/res/images/checklist/win_fw_01_new_rule.png){: width="80%" }

* Choose **TCP** and provide the port number (MSSQL Server default port is **1433**)

  ![Install selected features](/res/images/checklist/win_fw_02_define_port.png){: width="80%" }

* Leave **Allow the Connection** selected:

  ![Install selected features](/res/images/checklist/win_fw_03_allow.png){: width="80%" }

* Leave all checkboxes checked on the **Apply Rule** screen:

  ![Install selected features](/res/images/checklist/win_fw_04_apply_rule.png){: width="80%" }

* Provide a name for the rule and click the **Finish** button:

  ![Install selected features](/res/images/checklist/win_fw_05_name_rule.png){: width="80%" }

* Repeat the steps above, creating an **Outbound Rule** for the same port.
