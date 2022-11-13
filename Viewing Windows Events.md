# Viewing Windows Events
Methods to view Windows Events
EVTX Samples are from https://github.com/sbousseaden/EVTX-ATTACK-SAMPLES

## Event Viewer

## wevutil.exe


## Get-WinEvent

### Search for events involving a specific Executable

![image](https://user-images.githubusercontent.com/53142047/201525225-f60cabf0-df22-4f25-91b8-40f22b629edc.png)

``` Powershell
Get-WinEvent -FilterHashTable @{Path='.\merged.evtx'; Data='C:\Windows\System32\net1.exe' }
```


## XPath Queries


## Event IDs

