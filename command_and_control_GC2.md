# Attackers Leveraging Google Cloud Services for Command and Control

Attackers come up with creative ways to abuse cloud services for their own purposes. This blog will cover how attackers have leveraged Google cloud services in the form of the Google API, Drive and Sheets services to provide a command and control infrastructure to interact with the victim. The attack flow of an attack that leverages Google cloud infrastructure is illustrated below:

![image](https://user-images.githubusercontent.com/16122365/235530695-58e2b2cc-1550-480f-867d-186380492599.png)

Utilising popular cloud infrastructure provides several benefits to the attacker:

- Low cost infrastructure
- Traffic tends to blend in with normal personal / business internet usage
- Not many organisations can/will block Google cloud services 
- Out of the box not many security tools pick up the network communication as malicious



## Setting Up the Environment

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

`export GOOS=windows`
`export GOARCH=amd64`

Then we build the Windows binary using:

`go build -o gc2-sheet.exe`

Before restoring the operating system and architecture variables:

`unset GOOS`
`unset GOARCH`


