Watchlists in Microsoft Sentinel allow you to correlate data from a data
source you provide with the events in your Microsoft Sentinel
environment. For example, you might create a watchlist with a list of
high-value assets, terminated employees, or service accounts in your
environment.

Microsoft Sentinel customers often ask if there is a chance to create
analytic rules just for important(privileged) users such as Domain
admins, Enterprise admins. "Excessive Windows logon failures", "Multiple
RDP connections from Single System" or "RDP Nesting" built-in rules can
be some use-cases to track anomalies for privileged accounts.

By using watchlists, you can import on-premises AD privileged users to
Microsoft Sentinel and create analytics rules based on your needs. As
this operation is manual and you need to make watchlist up to date, you
must add or remove watchlist items when a specific user is added or
removed to specific privileged group such as domain admins.

As a solution to this, I've created a logic app to connect one of the
on-premises server (not domain controller) with standard read-only user
to gather privileged users, then update a watchlist based on this list.

**Deployment Steps**

**Create an Automation Account from the Azure Portal**

[*Before you begin review the pre-requisites of deploying a Hybrid
Runbook Worker
here:*](https://docs.microsoft.com/azure/automation/automation-windows-hrw-install)

![Graphical user interface, text, application, chat or text message
Description automatically
generated](./media/image1.png){width="2.5235804899387575in"
height="1.3109514435695537in"}

![Graphical user interface, application Description automatically
generated](./media/image2.png){width="3.919403980752406in"
height="2.309834864391951in"}

Deploy the Automation Hybrid Worker solution from the Azure Market place

![Graphical user interface, application Description automatically
generated](./media/image3.png){width="4.344724409448819in"
height="3.0813145231846018in"}

Go to Automation Account menu, create a Hybrid Worker Group

![Graphical user interface, text, application, email Description
automatically
generated](./media/image4.png){width="3.8222222222222224in"
height="3.1015179352580926in"}

In the Basics menu, I'm adding credentials (it will be used to gather
privileged users from AD, since all users can read domain users
properties, standard domain user is enough.)

![Graphical user interface Description automatically generated with
medium confidence](./media/image5.png){width="6.471983814523185in"
height="1.704798775153106in"}

In the Hybrid Workers menu, I'll pick one of the windows servers which
is already reporting to Sentinel enabled Log analytics workspace.

![Graphical user interface, text, application Description automatically
generated](./media/image6.png){width="4.461641513560805in"
height="1.2673140857392826in"}

This server will be the responsible to gather privileged users from AD.
To do this operation, first of all, you need to connect this server via
RDP, then run the following powershell cmdlet to run AD related cmdlets.

![Text Description automatically
generated](./media/image7.png){width="6.5in"
height="1.0368055555555555in"}

To create a new PowerShell Runbook navigate to your Automation Account
and select the Runbooks blade. Then create a runbook,

![Graphical user interface, text, application Description automatically
generated](./media/image8.png){width="3.281129702537183in"
height="1.9443733595800525in"}

![Graphical user interface Description automatically
generated](./media/image9.png){width="3.501784776902887in"
height="1.8238462379702538in"}

Select PowerShell from the Runbook type menu and paste the script below
in the resulting window. Click save then publish to activate the
Runbook.

Import-Module ActiveDirectory

Get-ADUser -Filter \"admincount -eq \'1\'\" -properties sid, cn \| select sid, cn \| convertto-json

![Text Description automatically generated with medium
confidence](./media/image10.png){width="6.5in"
height="1.0784722222222223in"}

**Test Runbook**

You can test your runbook to check everything is working properly,

![Text Description automatically generated with low
confidence](./media/image11.png){width="6.482400481189852in"
height="1.7221587926509185in"}

When task completes, you should see all your privileged accounts in the
output. As you see in the script, you can change the script according to
your need, e.g just to gather domain admins, or custom groups (SAP
users, VIP users, HR users\...)

![Graphical user interface, text Description automatically
generated](./media/image12.png){width="4.538601268591426in"
height="2.1932250656167978in"}

**Logic Apps Deployment**

First of all, you need to download logic app template file as a .json
format from the github URL:
[Sentinel/Watchlist-Add-OnPremADPrivUsersToWatchList.json at main ·
Yaerdem/Sentinel
(github.com)](https://github.com/Yaerdem/Sentinel/blob/main/Watchlist-Add-OnPremADPrivUsersToWatchList.json)

Go to Azure Portal and search for template

![Graphical user interface Description automatically
generated](./media/image13.png){width="4.843571741032371in"
height="1.1110706474190726in"}

![Graphical user interface, text, application, email Description
automatically
generated](./media/image14.png){width="2.7811472003499564in"
height="2.239501312335958in"}

Load Watchlist-Add-OnPremADPrivUsersToWatchList.json file which you
downloaded from Github.

![Graphical user interface, application Description automatically
generated](./media/image15.png){width="3.6343318022747155in"
height="1.568439413823272in"}

Fill the parameters according to your Azure environment:

![Graphical user interface, text, application, email Description
automatically
generated](./media/image16.png){width="2.645736001749781in"
height="2.8644783464566927in"}

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
generated](./media/image17.png){width="2.631848206474191in"
height="1.4964730971128608in"}

Fix all the connections based on your Azure environment, shown below.

![Graphical user interface, application Description automatically
generated](./media/image18.png){width="4.468585958005249in"
height="2.8540616797900262in"}

![Graphical user interface, application Description automatically
generated](./media/image19.png){width="3.826247812773403in"
height="2.7568427384076992in"}

Run logic app

![Graphical user interface, text, application Description automatically
generated](./media/image20.png){width="5.292630139982502in"
height="3.4175918635170603in"}

Go to Microsoft Sentinel and see your newly created watchlist

![Graphical user interface, text, application, email Description
automatically generated](./media/image21.png){width="6.5in"
height="6.711111111111111in"}

![Graphical user interface, text, application, email Description
automatically generated](./media/image22.png){width="6.5in"
height="2.490972222222222in"}

You can use this watchlist to create your own analytic rule. I'm sharing
one example to query against this watchlist to gather only privileged
users' successful logon activities.

![Graphical user interface, text, application, email Description
automatically generated](./media/image23.png){width="6.5in"
height="3.5722222222222224in"}
