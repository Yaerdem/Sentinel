This solution is for the customers who are using Palo Alto IOT solution and want to send alerts to Microsoft Sentinel.

As a solution to this, I've created a logic app to connect one of the
Palo Alto IOT with a customer ID and key, then get the Palo Alto IOT alerts to Microsoft Sentinel table named PaloAltoIOTAlerts

**Deployment Steps**

**Prerequisites**
1. Get log analytics workspace ID an key [this link](https://learn.microsoft.com/en-us/answers/questions/1154380/where-is-azure-is-the-primary-key-and-workspace-id).
2. Get your API key details from Palo Alto IOT [this link](https://pan.dev/iot/api/iot-public-api-headers/).

## Setup

### Deploy MISP Orchestrator

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FYaerdem%2FSentinel%2Frefs%2Fheads%2Fmain%2FPalo%20Alto%20IOT%2Fazuredeploy.json)

1. Deploy the `Palo Alto IOT Playbook` 
2. Fill out the information needed:
    * `Playbook Name` (this is Playbook Name whatever you want)
    * `Customer ID` (this is the Customer ID you can get from Palo Alto IOT Solution)
    * `Base URL` (this is the Base URL you can get from Palo Alto IOT Solution-It should be something like this: blalbabla.iot.paloaltonetworks.com)
    * `X-Key-ID` (this is the X-Key-ID which you get from the Prerequisites)
    * `X-Access-Key` (this is the X-Access-Key which you get from the Prerequisites)
3. Update the `Create new event MISP` step with the correct `event_creator_mail` 
    * **NOTE**: This should be the same user that you created the MISP key for and not an admin role
4. Make a note of the `HTTP POST URL` from the 'When a HTTP request is received' step

In the Basics menu, I'm adding credentials (it will be used to gather
privileged users from AD, since all users can read domain users
properties, standard domain user is enough.)

![Graphical user interface Description automatically generated with
medium confidence](./media/image5.png)

In the Hybrid Workers menu, I'll pick one of the windows servers which
is already reporting to Sentinel enabled Log analytics workspace.

![Graphical user interface, text, application Description automatically
generated](./media/image6.png)

This server will be the responsible to gather privileged users from AD.
To do this operation, first of all, you need to connect this server via
RDP, then run the following powershell cmdlet to run AD related cmdlets.

![Text Description automatically
generated](./media/image7.png)

To create a new PowerShell Runbook navigate to your Automation Account
and select the Runbooks blade. Then create a runbook,

![Graphical user interface, text, application Description automatically
generated](./media/image8.png)

![Graphical user interface Description automatically
generated](./media/image9.png)

Select PowerShell from the Runbook type menu and paste the script below
in the resulting window. Click save then publish to activate the
Runbook.

Import-Module ActiveDirectory

Get-ADUser -Filter \"admincount -eq \'1\'\" -properties sid, cn \| select sid, cn \| convertto-json

![Text Description automatically generated with medium
confidence](./media/image10.png)

**Test Runbook**

You can test your runbook to check everything is working properly,

![Text Description automatically generated with low
confidence](./media/image11.png)

When task completes, you should see all your privileged accounts in the
output. As you see in the script, you can change the script according to
your need, e.g just to gather domain admins, or custom groups (SAP
users, VIP users, HR users\...)

![Graphical user interface, text Description automatically
generated](./media/image12.png)

**Logic Apps Deployment**

First of all, you need to download logic app template file as a .json
format from the github URL:
[Sentinel/Watchlist-Add-OnPremADPrivUsersToWatchList.json at main ·
Yaerdem/Sentinel
(github.com)](https://github.com/Yaerdem/Sentinel/blob/main/Watchlist-Add-OnPremADPrivUsersToWatchList.json)

Go to Azure Portal and search for template

![Graphical user interface Description automatically
generated](./media/image13.png)

![Graphical user interface, text, application, email Description
automatically
generated](./media/image14.png)

Load Watchlist-Add-OnPremADPrivUsersToWatchList.json file which you
downloaded from Github.

![Graphical user interface, application Description automatically
generated](./media/image15.png)

Fill the parameters according to your Azure environment:

![Graphical user interface, text, application, email Description
automatically
generated](./media/image16.png)

**Resource Group**: Name of the Resource group you want to deploy this
logic app

**Region**: Region name for this logic app

**Playbook Name**: Name of the logic app

**Automation Account Name**: Automation account name which you created
before the logic app

**Log Analytics Workspace ID**: Workspace ID of Sentinel Enabled Log
analytics workspace

**Resource Group for Automation**: RG name which contains your
Automation Account you created

**Resource Group for Log Analytics WS**: RG name which contains your
Microsoft Sentinel enabled Log Analytics Workspace

**Runbook Name**: Runbook name which you created in Automation Account

**Subscription ID:** Your Subscription ID which contains your Log
Analytics Workspace and automation account.

**Watchlist Alias**: Watchlist Alias that will be created by this logic
app

**Worker Group Name**: Worker Group name which you created in Automation
Account

After creation, you should see deployment is complete.

![Graphical user interface, text, application, Teams Description
automatically
generated](./media/image17.png)

Fix all the connections based on your Azure environment, shown below.

![Graphical user interface, application Description automatically
generated](./media/image18.png)

![Graphical user interface, application Description automatically
generated](./media/image19.png)

Run logic app

![Graphical user interface, text, application Description automatically
generated](./media/image20.png)

Go to Microsoft Sentinel and see your newly created watchlist

![Graphical user interface, text, application, email Description
automatically generated](./media/image21.png)

![Graphical user interface, text, application, email Description
automatically generated](./media/image22.png)

You can use this watchlist to create your own analytic rule. I'm sharing
one example to query against this watchlist to gather only privileged
users' successful logon activities.

![Graphical user interface, text, application, email Description
automatically generated](./media/image23.png)
