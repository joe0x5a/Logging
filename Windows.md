# Windows Logging
Configuring Logging using Group Policy


## Configure DNS Logging
Enable DNS diagnostic logging

Type `eventvwr.msc` at an elevated command prompt and press ENTER to open Event Viewer.
In Event Viewer, navigate to `Applications and Services Logs\Microsoft\Windows\DNS-Server`.

Right-click DNS-Server, point to View, and then click Show Analytic and Debug Logs. The Analytical log will be displayed.

Right-click `Analytical` and then click `Properties`.

Under `When maximum event log size is reached`, choose either: 
`Do not overwrite events (Clear logs manually)`
`Overwrite events as needed`
`Archive the log when full, do not overwrite events`

*note* DNS logging is very verbose and will consume a lot of disk. Be sure to have a method of managing logs if you dont use the overwrite option.  For the purpose of this lab I will set the log to overwrite.

Select the `Enable logging` checkbox, and click `OK` when you are asked if you want to enable this log.

## Configure DHCP Logging




## Create a Logging GPO
Open `Group Policy Management`, right click `Group Policy Objects` and choose  `New` and give the  GPO a name.

![image](https://user-images.githubusercontent.com/53142047/197384282-7068a6bc-bae1-46ed-8438-82042d65fcb6.png)

We now need to edit this GPO to set our Looging configuration. Right Click the  new GPO, called `Logging` in this example and chose `Edit`, this will open the `Group Policy Editor`


### Set the Log size
The defaults are woefully small and will roll.  Configure the max log size until the desired  retention period is achieved (diskspace is cheap).

To set the Application, Security, Setup and system Log size.
`Computer Configuration > Policies > Administrative Templates > Window Components > Event Log Service`
For each  Log  enable `Specify the maximum log file size (KB)`

![image](https://user-images.githubusercontent.com/53142047/197384815-e5f0c26f-213d-44cf-9151-84e5bdfbd357.png)

Sugggested Values:
Application, Setup and System - 4098KB
Security - 1,024,000KB

### Enable the use of Advanced Audit Policies
This step  lets you use the Advanced audit policies.

`Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options > Audit: Force audit policy subcategory settings`

![image](https://user-images.githubusercontent.com/53142047/197385368-ca19b094-676e-47b9-97ca-9e75417b8d92.png)


### Link the GPO to an OU
When the GPO is  complete it has to be linked to an OU. I want the logging policy to be linked to the Domain Controlers only. I also  adjusted the  link order to be 1 so the policy cant be changed by a policy with a better Precedence (lower is better)

![image](https://user-images.githubusercontent.com/53142047/197386263-ca860914-082b-4e5a-9073-ebde887b09c1.png)

Theres more to do with this GPO  but first lets check the settings and been applied to the Domain Controller
For Example: Go to Event Viewer and  select the Secuity Log Properties, check the  log size is correct.
If the settings dont look like they have been applied, use `gpupdate /force` to force the policy to be refreshed.

## Check Current Audit Policy

```
auditPol /get /category:*
```
![image](https://user-images.githubusercontent.com/53142047/197388137-6534cc91-1716-4d4b-8a02-af04107478ee.png)





## System Audit Policies
