# Attackers Leveraging Google SaaS services for Command and Control infrastructure

Attackers come up with creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Notion online workbooks to provide a command and control infrastructure to interact with victims via the OffensiveNotion agent. The attack flow of an attack that leverages Notion cloud infrastructure is illustrated below:

![image](https://user-images.githubusercontent.com/16122365/235897423-28a8e75e-591d-4d53-b0f5-7261d27625e4.png)

Utilising popular cloud infrastructure provides several benefits to the attacker: 

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious

# Install pre-requistes and compile the rust program

TBD - Not required for this blog.

# Configure the Notion components

1. Create a Notion account at https://www.notion.so/signup

![image](https://user-images.githubusercontent.com/16122365/235878558-5eb7fe30-a0df-4724-88f0-e822600e9680.png)

2. Create a page on Notion that will be used to produce sub pages for each victim/C2 connection, interact with the OffensiveNotion agent installed on victim devices, and to receive results/outputs of interactions with the target.

![image](https://user-images.githubusercontent.com/16122365/235880026-2dfa8cc8-cd1a-4973-b7f9-8f1c654fe9db.png)

3. Head over to https://developers.notion.com/ and sign-in with a new or existing account and head to your integrations

![image](https://user-images.githubusercontent.com/16122365/235880353-8b02d54f-9bec-4e83-9d7a-2de3f34fa988.png)

4. Create a new integration

![image](https://user-images.githubusercontent.com/16122365/235880696-2e656014-e438-4ec6-b2a1-ceeb6016f80d.png)

5. Configure capabilities and permissions

![image](https://user-images.githubusercontent.com/16122365/235880955-e661bc8c-3beb-469c-9494-a7bc57076ef5.png)

6. Make a note of the internal integration token secret that will be used to authenticate and authorise against the Notion API

![image](https://user-images.githubusercontent.com/16122365/235881183-b197cf67-0743-4766-a4e0-0d14de2be1ce.png)

7. Head over to your newly created Notion page click the three dots at the top right and add a new connection

![image](https://user-images.githubusercontent.com/16122365/235882256-9bcede44-0195-4de9-a675-a07658f452cb.png)

8. Grab the parent page ID of your Notion page by hitting the share button and copying the link into an editor of choice

![image](https://user-images.githubusercontent.com/16122365/235883290-086ad4f6-77d4-48e4-b485-e2aad1f72f06.png)

9. Grab the id portion of the URL which will be used in the agent configuration

![image](https://user-images.githubusercontent.com/16122365/235883605-33e05857-61b2-4f7b-add9-9a3b58317c28.png)


# Testing the C2 Server - Initial Access

For the purposes of the blog I will not be delivering the payload but you can assume that any common initial access vector can be used and then the agent itself can be downloaded as a subsequent stage malware as is common with many intrusions.

1. Download the agent from the releases section of the Offensive Notion GitHub: https://github.com/mttaggart/OffensiveNotion/releases. For the test we will use the Windows debug agent to keep things simple.

![image](https://user-images.githubusercontent.com/16122365/235884546-d7ff729a-176b-4d7b-8a2f-c3f94e8fbba3.png)

2. Transfer the agent to the victim host

![image](https://user-images.githubusercontent.com/16122365/235885306-c7423bc0-5e77-41b9-9343-69ba0b2f527a.png)

3. Run the agent with the -d parameter

`./offensive_notion -d`

5. Enter the following configuration parameters

```
[*] Starting!
Getting config options!
[*] Enter agent sleep interval > 5
[*] Enter agent jitter time > 0
[*] Enter parent page id > [insert page id - STEP 8 in previous section]
[*] Enter API Key > [insert integration token secret - STEP 6 in previous section]
[*] Enter Config File Path > [leave blank]
[*] Enter Log Level (1-4) > 2
```
6. Once run, you should see a new sub page created in your Notion parent page that is representative of the hostname of the victim device.

![image](https://user-images.githubusercontent.com/16122365/235894877-9276b54d-8f82-4c44-b64c-d93ad7fbbd58.png)

7. By clicking on the hostname entry we can enter a sub-page and start to interact with the agent.

![image](https://user-images.githubusercontent.com/16122365/235903428-abcc7ae1-5462-41fc-a59f-7433468d2048.png)

8. When in the subpage select Empty page and you are ready to start interacting with the OffensiveNotion agent

![image](https://user-images.githubusercontent.com/16122365/235906588-c1c950d0-072b-4940-810f-59f44a5d73b9.png)

## Example commands and output:

A full list of the tools capabilities and commands are listed here: https://github.com/mttaggart/OffensiveNotion/wiki/6.-Agent-Interaction. For the purposes of the blog I will provide some examples and then provide advice how we could go about huning for and detecting related behaviours.

To start issuing commands to the agent you first have to create a todo list

![image](https://user-images.githubusercontent.com/16122365/235906801-dd20c0b7-0750-4888-8299-c4d9f0452d4e.png)

The command syntax looks something like the below. Complete commands require the target emoji before they will be executed by the agent.

![image](https://user-images.githubusercontent.com/16122365/235907555-b7b5f399-0e74-49c2-b198-2e6758561cca.png)


### Discovery

Once the bad guys have established a command and control connection to the victim it is common place for commands to be executed to perform discovery to understand more about the system, users, domain, software installed, networks its connected to and so on. To do this they will often use living off the land techniques such as PowerShell, WMI, CMD etc. or in some instances they will automate the process by downloading scripts that can be run on the host providing an output of the results to the console or to a file that can be retrieved.

The below images provide an illustration of the commands that can be run and subsequent output recorded in the Notion page.

#### Host discovery

![image](https://user-images.githubusercontent.com/16122365/236190674-1dd0d601-800c-4b0f-81f6-7e7d95e4efbd.png)


#### Network discovery

Gather the system network configuration.

![image](https://user-images.githubusercontent.com/16122365/236192492-05ef67cf-7fde-4a7e-ae11-fe0dff34791c.png)

Perform a portscan to discover other hosts on the local subnet.

![image](https://user-images.githubusercontent.com/16122365/236191046-b82506fc-6c1b-4cfb-a85c-027e6ecd7f0e.png)


### Persistence

Once access to the victim is established the attackers will install persistence mechanisms to facilitate ongoing access to the victim(s) and associated networks. Often this is done through malicious services, registry run kys or autostart locations, scheduled tasks, or more sneakily via creation of Windows Management Instrumentation (WMI) event subscriptions.

To demonstrate this we have selected the WMI event subscription method that will create an event subscription that will trigger on reboot and restore the C2 connectivity via the Notion API.

![image](https://user-images.githubusercontent.com/16122365/236193167-fc90cd67-e1b8-4df6-b581-cc01d210b3bd.png)

### Data ingress

An example of using Github to pull an enumeration script down to the host.

![image](https://user-images.githubusercontent.com/16122365/236192774-bc2cfdf2-b1d1-48a2-b9cb-a91924fb3fac.png)

### Collection and Exfiltration

An example of downloading a tool or using PowerShell to perform collection and exfiltration of data to an Azure storage account.

![image](https://user-images.githubusercontent.com/16122365/236194819-05e234a8-e95e-430b-b8ca-556851500f84.png)

# Detection

In this scenario we understand that the OffensiveNotion C2 framework will programmatically interact with the endpoint at api.notion.com.

However, its worth noting that calls to api.notion.com will be quite noisy if we don't include some process exclusions. For example browsers and some desktop apps may cause false positives. To start with we will want to exclude common browser process file names or paths, and legitimate notion related applications.

## Kusto query to detect this network behaviour

The DeviceNetworkEvents table in MDE advanced hunting would seem an obvious choice to start with. First I will execute a query to identify connections to the api.notion.com.

![image](https://user-images.githubusercontent.com/16122365/236221296-01d8fbc1-b058-4a41-aafa-178c98617f4b.png)

Based on what we see above we can build out the start of our query. The below line of code can be used to exclude common browser process filenames but their are likely many more that need to be added based on evaluation of the operating environment. Next we can make sure were only looking at connections to the endpoints we care about. The query below works well as it is and can be used to detect the behaviour related to the OffensiveNotion tool.

![image](https://user-images.githubusercontent.com/16122365/236220558-9eb5f41b-2efd-4b5a-9eb0-7e21d09e7159.png)

## Final KQL query

```
let excludedProcesses = datatable(name:string)["browser1.exe","browser2.exe"]; //examples but check your environment first to remove false positives and use the filename and file path to reduce risk of false negative or evasion from the bad guys
DeviceNetworkEvents
| where RemoteUrl has "api.notion.com" and not (InitiatingProcessFileName has_any (excludedProcesses)) and InitiatingProcessVersionInfoCompanyName != "Notion Labs, Inc"
```

As you can see the activity has been picked up based on the telemetry forwarded from the victim host

**IMAGE**

## Identifying files created by the suspicious process making connections to Notion APIs

Below I will cover off the ability to download files (T1544: Ingress Tool Transfer) from the C2. In this instance, we have downloaded an enumeration script that can be used by the attacker to perform discovery of the local system and connected network.

The below query is an example of how we can use our initial query to identify suspicious process filenames that are communicating with the Notion APIs that subsequently create files on the host which could indicate a tool being transferred from the C2 to the victim.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
DeviceNetworkEvents
| where RemoteUrl has "api.notion.com" and not(InitiatingProcessFileName has_any (excludedProcessFileNames)) and InitiatingProcessVersionInfoCompanyName != "Notion Labs, Inc"
| join DeviceFileEvents on InitiatingProcessFileName | where ActionType1 =~ "FileCreated"
| project TimeGenerated1, ActionType, DeviceName, InitiatingProcessAccountDomain1, InitiatingProcessAccountName1, InitiatingProcessAccountSid1, InitiatingProcessCommandLine1, InitiatingProcessParentFileName1, InitiatingProcessFileName1, FileName
```

An example output from the query is shown below

![image](https://user-images.githubusercontent.com/16122365/236261298-f6a2c2f6-855d-42a9-aa71-1964d0cfff22.png)

In addition to the above the below query can be used to pivot for processes and commandlines associated with the processes that had performed the initial network connections.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
DeviceNetworkEvents
| where RemoteUrl contains "notion.com"
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames)) and InitiatingProcessVersionInfoCompanyName != "Notion Labs, Inc"
| extend joinkey = strcat(InitiatingProcessFileName, DeviceName, InitiatingProcessAccountName)
| join kind=leftouter (DeviceProcessEvents | extend joinkey = strcat(InitiatingProcessParentFileName, DeviceName, InitiatingProcessAccountName) | summarize ProcessesRanByParent = make_set(InitiatingProcessCommandLine) by joinkey) on joinkey
| join kind=leftouter (DeviceFileEvents | where ActionType == "FileCreated" | extend joinkey = strcat(InitiatingProcessParentFileName, DeviceName, InitiatingProcessAccountName) | summarize FilesCreated = make_set(FileName) by joinkey) on joinkey
| project TimeGenerated, DeviceName, InitiatingProcessAccountName, InitiatingProcessCommandLine, InitiatingProcessFolderPath, FilesCreated, ProcessesRanByParent, LocalIP, RemoteIP, RemoteUrl
```

An example output from the query is displayed below

![image](https://user-images.githubusercontent.com/16122365/236263327-1602c230-f44b-474c-9510-b03fc1a40922.png)


## Identifying data exfiltration

GC2 provides the ability to exfiltrate data (T1567.002:Exfiltration Over Web Service: Exfiltration to Cloud Storage) from the endpoint to Azure storage or AWS S3. The image below illustrates how the data appears in the specified Azure storage blob.

**IMAGE**

The image below illustrates how data can also be exfiltrated to an S3 bucket in AWS:

**IMAGE**

Defender for Cloud Apps is pretty good at this and could be used to identify and alert on a high volume of data uploaded to Google. This could prove tricky if Notion is being used widely across the organisation. 

**IMAGE**

# Conclusion

In conclusion, attackers are constantly adapting and finding new ways to leverage legitimate services for malicious purposes. In this case, we've demonstrated how Google Cloud Services can be used as a command and control infrastructure. By understanding the tools and procedures used by current threat groups we can perform micro emulation and understand what telemetry is available to improve our ability to detect, prevent, and respond to adversarial operations, improving overall security posture and awareness of the threat.
