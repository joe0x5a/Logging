# Windows Logging
Configuring Logging using Group Policy


## Configure DNS Logging
Enable DNS diagnostic logging

Type `eventvwr.msc` at an elevated command prompt and press ENTER to open Event Viewer.
In Event Viewer, navigate to `Applications and Services Logs\Microsoft\Windows\DNS-Server`.

Right-click DNS-Server, point to View, and then click `Show Analytic and Debug Logs`. The Analytical log will be displayed.

Right-click `Analytical` and then click `Properties`.

Under `When maximum event log size is reached`, choose either: 
`Do not overwrite events (Clear logs manually)`
`Overwrite events as needed`
`Archive the log when full, do not overwrite events`

*NOTE*: DNS logging is very verbose and will consume a lot of disk. Be sure to have a method of managing logs if you dont use the overwrite option.  For the purpose of this lab I will set the log to overwrite.

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


## Configure Advanced Audit Policies

`Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Audit Policies`

| __Account Logon__ | __Setting__ |
|-----------------|-----------|
| Credential Validation | Success and Failure |
| Other Account Login Events | Success and Failure |
| __Account Management__ | __Setting__ |
| Application Group Management | Success and Failure |
| Computer Account Management | Success and Failure |
| Distribution Group Management | Success and Failure |
| Other Acct Management Events | Success and Failure |
| Security Group Management | Success and Failure |
| User Account Management | Success and Failure |
|  __Detailed Tracking__ | __Setting__ |
| DPAPI Activity | No Auditing |
| Plug and Play | Success |
| Process Creation | Success and Failure |
| Process Termination | No Auditing |
| RPC Events | Success and Failure |
| Audit Token Right Adj | Success |
| __DS Access__ | __Setting__ |
| Detailed Directory Service Repl | No Auditing |
| Directory Service Access | No Auditing |
| Directory Service Changes | Success and Failure |
| Directory Service Replication | No Auditing |
| __Logon/Logoff__ | __Setting__ |
| Account Lockout | Success |
| User / Device Claims | No Auditing |
| Group Membership | Success |
| IPsec Extended Mode | No Auditing |
| IPsec Main Mode | No Auditing |
| IPsec Quick Mode | No Auditing |
| Logoff | Success |
| Logon | Success and Failure |
| Network Policy Server | Success and Failure |
| Other Logon/Logoff Events | Success and Failure |
| Special Logon | Success and Failure |
| __Object Access__ | __Setting__ |
| Application Generated | Success and Failure |
| Certification Services | Success and Failure |
| Detailed File Share | Success |
| File Share | Success and Failure |
| File System | Success |
| Filtering Platform Connection | Success |
| Filtering Platform Packet Drop | No Auditing |
| Handle Manipulation | No Auditing |
| Kernel Object | No Auditing |
| Other Object Access Events | No Auditing |
| Registry | Success |
| Removable Storage | Success and Failure |
| SAM | Success |
| Central Policy Staging | No Auditing |
| __Policy Change__ | __Setting__ |
| Audit Policy Change | Success and Failure
| Authentication Policy Change | Success and Failure |
| Authorization Policy Change | Success and Failure |
| Filtering Platform Policy Change | Success |
| MPSSVC Rule-Level Policy Change | No Auditing |
| Other Policy Change Events | No Auditing |
| __Privilege Use__ | __Setting__ |
| Non Sensitive Privilege Use | No Auditing |
| Other Privilege Use Events | No Auditing |
| Sensitive Privilege Use | Success and Failure |
| __System__ | __Setting__ |
| IPsec Driver | Success |
| Other System Events | Failure |
| Security State Change | Success and Failure |
| Security System Extension | Success and Failure |
| System Integrity | Success and Failure |

## Log the Commandline of Process Creation Events
Process Creation (sucess and failure) has already been configured. While this will show the process, but it wont show any parameters / switches.

![image](https://user-images.githubusercontent.com/53142047/201486938-e571d394-c335-4d84-a2e8-d44c3a0b0bce.png)

The Process command line section is blank.

In Group Policy Editor, Auditing of Process Creation can be enabled by configuring:
`Computer Configuration > Policies > Administrative Templates > System > Audit Process Creation > Include Commandline in process creation events'

![image](https://user-images.githubusercontent.com/53142047/201487327-6c9e95c9-2d15-4df1-bc26-9ea04f8a1a54.png)

Now the full command line is audited, and the analyst has better visibility

![image](https://user-images.githubusercontent.com/53142047/201487406-caaf29ad-6515-4756-983d-c008d5c881d3.png)


## Audit File Access 
Audit locations which are known to be abused


## Powershell Logging (Version 5)
Notes from https://www.mandiant.com/resources/blog/greater-visibilityt

Logging is configured through Group Policy
` Administrative Templates > Windows Components > Windows POwershell`

![image](https://user-images.githubusercontent.com/53142047/198684111-42a360cc-dc39-42c6-a598-07ac1e0bec3e.png)

Powershell supports 3 types of logging:
- Module Logging
- Script Block Logging
- Transcription

All three write the events to `%SystemRoot%\System32\Winevt\Logs\Microsoft-Windows-PowerShell%4Operational.evtx`
This log can be opened in  Event Viewer byt navigating to `Application and #services Logs > Microsoft > Windows > Powershell > Operational`

### Module Logging
Module logging records: 
- pipeline execution details as PowerShell executes, including variable initialization
- command invocations. 

Module logging will record portions of scripts, some de-obfuscated code, and some data formatted for output. This logging will capture some details missed by other PowerShell logging sources, though it may not reliably capture the commands executed. Module logging has been available since PowerShell 3.0. Module logging events are written to Event ID (EID) 4103.

While module logging generates a large volume of events (the execution of the popular Invoke-Mimikatz script generated 2,285 events resulting in 7 MB of logs during testing), these events record valuable output not captured in other sources.

__To enable Module Logging__
- In the Windows POwershell GPO set `Turn on Module Logging` to `enabled` 
- In the Optons pane, click the button to  show Module Name.
- In the Module Names window enter * to all  modules are in scope.
- Click OK, then OK again.

Alternative method: Configure the setting in the registry.
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ModuleLogging → EnableModuleLogging = 1`
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ModuleLogging \ModuleNames → * = *`


### Script Block Logging
Script block logging records blocks of code as they are executed. It will record the obfuscated and deobfuscated version of the code.
Script block logging events are recorded in EID 4104.
Script blocks exceeding the maximum length of an event log message are fragmented into multiple entries. (32kb)
A script at https://github.com/matthewdunwoody/block-parser can be used to reasemble fragmented blocks.

Powershell 5.0 will log code blocks if they match a suspicious command or scripting techniqiue, even if  Script Block Logging is not enabled. These suspicious blocks are logged at the “warning” level in EID 4104, unless script block logging is explicitly disabled.

Script block logging generates fewer events than module logging (Invoke-Mimikatz generated 116 events totaling 5 MB)

Group Policy also offers the option to  `Log script block execution start / stop events`. his option records the start and stop of script blocks, by script block ID, in EIDs 4105 and 4106. Provides grater forensic evidence but can generate a prohibitively large  number of events. 96,458 events totaling 50 MB per execution of Invoke-Mimikatz.  This option is not generally  recommended for most environments.

__To Enable Script Block Logging__
- In the Windows POwershell GPO set `Turn on Powershell Script Block Logging` to `enabled` and do not configure  the `Log script block execution start / stop events`

Alternative Method: Configure the settings in the registry.
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging → EnableScriptBlockLogging = 1`


### Transcription 
Transcription creates a unique record of every PowerShell session, including all input and output, exactly as it appears in the console session. 
Transcripts are written to text files, broken out by user and session. Transcripts also contain timestamps and metadata for each command in order to aid analysis.
Transcription only shows what appears in the terminal, so will not see the contents of executed scripts, or output which is written to anywhere other than the terminal (eg filesystem)

Transcripts are by default written to  the users Documents folder and will have a name beginning with "Powershell_transcript".  It is better to write the transcripts to a write-only network share, so an attacker cannot easily  delete the log. 

Transcripts are storage efficient, Les than 6kb for  an execution of Invoke-Mimikatz.

__To Enable Transcription__
-  In the “Windows PowerShell” GPO settings, set `Turn on PowerShell Transcription` to enabled.
-  Check the “Include invocation headers” box, in order to record a timestamp for each command executed.
-  Optional, send the logs to a central write-only share

Alternative Method: Configure the settings in the registry.
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\Transcription → EnableTranscripting = 1`
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\Transcription → EnableInvocationHeader = 1`
`HKLM\SOFTWARE\Wow6432Node\Policies\Microsoft\Windows\PowerShell\Transcription → OutputDirectory = “” (Enter path. Empty = default)`


## Log Settings
- Where possible, configure all three settings
- In environments where log sizes cannot be significantly increased, enabling script block logging and transcription will record most activity, while minimizing the amount of log data generated.
- At a minimum, script block logging should be enabled

The size of the  Powershell event log should be increased to 1GB as a minimum.  Powershell logging can generate logs at a rate of 1MB per minute

The Windows Remote Management (WinRM) log, Microsoft-Windows-WinRM%4Operational.evtx, records inbound and outbound WinRM connections, including PowerShell remoting connections. The log captures the source (inbound connections) or destination (outbound connections), along with the username used to authenticate. This connection data can be valuable in tracking lateral movement using PowerShell remoting. Ideally, the WinRM log should be set to a sufficient size to store at least one year of data.

ue to the large number of events generated by PowerShell logging, organizations should carefully consider which events to forward to a log aggregator.

In environments with PowerShell 5.0, organizations should consider, at a minimum, aggregating and monitoring suspicious script block logging events, EID 4104 with level “warning”, in a SIEM or other log monitoring tool. These events provide the best opportunity to identify evidence of compromise while maintaining a minimal dataset.

