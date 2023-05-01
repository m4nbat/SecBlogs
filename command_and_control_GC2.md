# Attackers Leveraging Google Cloud Services for Command and Control

Attackers come up with creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Google cloud services in the form of the Google API, Drive and Sheets services to provide a command and control infrastructure to interact with the victim. The attack flow of an attack that leverages Google cloud infrastructure is illustrated below:

![image](https://user-images.githubusercontent.com/16122365/235530695-58e2b2cc-1550-480f-867d-186380492599.png)

Utilising popular cloud infrastructure provides several benefits to the attacker:

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious



## Install pre-requistes and compile the go program

First we need to grab the project from Github:

`git clone https://github.com/looCiprian/GC2-sheet`

Make sure go is installed, if it isn't download from: 

`https://go.dev/doc/install`

On Linux you can download the package and then install using:

`sudo tar -C /usr/local -xzf go1.20.3.linux-amd64.tar.gz`

Then add go to your PATH:

`export PATH=$PATH:/usr/local/go/bin`

Check your version and were ready to go (excuse the pun :))

`go version`

Next we need to compile the code using build.

`go build gc2-sheet.go`

To generate a windows compatible executable we ensure we are in the project folder and set OS and architecture variables:

```
export GOOS=windows
export GOARCH=amd64
```

Then we build the Windows binary using:

`go build -o gc2-sheet.exe`

Before restoring the operating system and architecture variables:

```
unset GOOS
unset GOARCH
```

## Configure the Google cloud components

1. Create a folder on Google Drive that can be used to upload and download data from/to the victim.

![image](https://user-images.githubusercontent.com/16122365/235531473-a5b1f0e7-1269-4f39-943c-7e5abf292e83.png)

2. Create a Google Sheet that will be used to send commands to the victim and to receive and record their output in the Google Sheet.

images.githubusercontent.com/16122365/235531490-d27d8dc9-17c4-4560-aaa1-4468971d9d8d.png)

3. Create a service account at [https://console.cloud.google.com/](https://console.cloud.google.com/)

![image](https://user-images.githubusercontent.com/16122365/235531719-f275eca6-7d95-4854-a4b8-e7586b695deb.png)

![image](https://user-images.githubusercontent.com/16122365/235531746-38a527db-eb21-4209-8a91-e5ebdc147244.png)

4. Once created, create and download a JSON key file for the service account.

![image](https://user-images.githubusercontent.com/16122365/235531768-3d943c72-dd1a-4bf3-bc5c-85df14607297.png)

5. Add the service principal as an editor on the google drive folder and google sheets document.

![image](https://user-images.githubusercontent.com/16122365/235531871-59d6aa73-27a4-47a4-a667-81b27b6c8b9a.png)

## Testing the C2 Server

Transfer the windows binary created initial and the key file to the test victim host. It's possible to hardcode these to the binary however as its bank holiday weekend and we are just demonstrating capability we will use the command line to specify our keys and credentials.

![image](https://user-images.githubusercontent.com/16122365/235531953-562ddc40-ea6e-4991-8e0e-bffa1f324cf8.png)

Once run, you should see a new tab created in your google sheets.

![image](https://user-images.githubusercontent.com/16122365/235531991-e72cc064-fa68-44d8-9f6f-e101ce9bc65f.png)

### Example commands and output:

The image illustrates the commands that can be run from the Google sheet. These are processed by the agent running on the victim, executed on the host, before the result/output is written back to the google sheet.

![image](https://user-images.githubusercontent.com/16122365/235532039-96d9a6ec-2cdc-4da1-be68-e2a19b581679.png)




