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


# Testing the C2 Server

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
Once run, you should see a new sub page created in your Notion parent page that is representative of the hostname of the victim device.

![image](https://user-images.githubusercontent.com/16122365/235894877-9276b54d-8f82-4c44-b64c-d93ad7fbbd58.png)

## Example commands and output:

### Discovery



### Persistence



### Data ingress



### Exfiltration



# Detection

In this scenario we understand that the C2 framework will programmatically interact with the endpoints on the *.googleapis.com domain. In particular we will want to monitor for oauth2.googleapis.com, sheets.googleapis.com, and drive.googleapis.com.

However, its worth noting that calls to .googleapis.com will be quite noisy if we don't include some process exclusions. For example browsers and some desktop apps may cause false positives. To start with we will want to exclude common browser process filenames, updaters for chrome or legitimate other google related applications.

## Kusto query to detect this network behaviour

The DeviceNetworkEvents table in MDE advanced hunting would seem an obvious choice to start with. First I will execute a query to identify connections to the *.googleapis.com domain.

**IMAGE**

Based on what we see above we can build out the start of our query. The below line of code can be used to exclude common browser process filenames but their are likely many more that need to be added based on evaluation of the operating environment.

**IMAGE**

Next we can make sure were only looking at connections to the endpoints we care about. The query below works well as it is and can be used to detect the behaviour related to the GC2 offensive tool.

**IMAGE**

In the end I decided to create an additional field called visitedURLs that would hold a dynamic set of the RemoteUrl's visited in the time period of the query. This can be used to do analysis and filtering in a noisy environment.

**IMAGE**

I then used project to produce a more clear field output for the analyst and to add a count of the number of connections identified in the array.

**IMAGE**

## Final KQL query

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com") and isnotempty(InitiatingProcessFileName)
| summarize visitedURLs=make_list(RemoteUrl) by ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName
| project ActionType, DeviceName, InitiatingProcessAccountName, InitiatingProcessParentFileName, InitiatingProcessFileName, visitedURLs, Connections=array_length(visitedURLs)
//| where visitedURLs contains "oauth2.googleapis.com" and visitedURLs has_any ("sheets.googleapis.com","drive.googleapis.com") // may allow for higher fidelity as the GC2 go application communicates to both the google drive folder and sheets API.
```

As you can see the activity has been picked up based on the telemetry forwarded from the victim host

**IMAGE**

## Identifying files created by the suspicious process making connections to Google APIs

Below I will cover off the ability to download files (T1544: Ingress Tool Transfer) from the C2. In this instance, we have downloaded an enumeration script that can be used by the attacker to perform discovery of the local system and connected network.

The below query is an example of how we can use our initial query to identify suspicious process filenames that are communicating with Google APIs and to use the distinct list of names as a filter to search for files created on the host which could indicate a tool being transferred from the C2 to the victim.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
let processComWithGoogleAPI = DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com") and isnotempty(InitiatingProcessFileName)
| distinct InitiatingProcessFileName;
DeviceFileEvents
| where ActionType == "FileCreated" and InitiatingProcessFileName in~ (processComWithGoogleAPI)
```

An example output from the query is shown below

**IMAGE**

The below query can be used to pivot for processes and commandlines associated with the processes that had performed the initial network connections.

```
let excludedProcessFileNames = datatable (browser:string)["teams.exe","GoogleUpdate.exe","outlook.exe","msedge.exe","chrome.exe","iexplorer.exe","brave.exe","firefox.exe"]; //add more browsers or mail clients where needed for exclusion 
let processComWithGoogleAPI = DeviceNetworkEvents 
| where not(InitiatingProcessFileName has_any (excludedProcessFileNames))
| where RemoteUrl has_any ("oauth2.googleapis.com","sheets.googleapis.com","drive.googleapis.com","www.googleapis.com") and isnotempty(InitiatingProcessFileName)
| distinct InitiatingProcessFileName;
DeviceFileEvents
| where ActionType == "FileCreated" and InitiatingProcessFileName in~ (processComWithGoogleAPI)
```

An example output from the query is shown below

**IMAGE**


## Identifying data exfiltration

GC2 provides the ability to exfiltrate data (T1567.002:Exfiltration Over Web Service: Exfiltration to Cloud Storage) from the endpoint to Azure storage or AWS S3. The image below illustrates how the data appears in the specified Azure storage blob.

**IMAGE**

The image below illustrates how data can also be exfiltrated to an S3 bucket in AWS:

**IMAGE**

Defender for Cloud Apps is pretty good at this and could be used to identify and alert on a high volume of data uploaded to Google. This could prove tricky if Notion is being used widely across the organisation.

**IMAGE**

# Conclusion

In conclusion, attackers are constantly adapting and finding new ways to leverage legitimate services for malicious purposes. In this case, we've demonstrated how Google Cloud Services can be used as a command and control infrastructure. By understanding the tools and procedures used by current threat groups we can perform micro emulation and understand what telemetry is available to improve our ability to detect, prevent, and respond to adversarial operations, improving overall security posture and awareness of the threat.
