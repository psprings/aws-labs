# aws-labs
## Description
This tool is for use with **AWS Training Labs**. If you want to run through labs multiple times, or just hate all of the manual steps involved in initial setup, then try this out. Only a select few commands work with Windows at present.

---
For full sync features:
* [Atom](https://atom.io/) as text editor/sync tool
 * Plugin: [remote-sync](https://atom.io/packages/remote-sync)

---

Open up the labs that you are registered for via https://aws.qwiklab.com

Select your lab

When you click `Start Lab` you should see the following options under the `Connect` tab:

---

#### AWS Console Details
AWS Account:
`<AWS ACCOUNT>`

Username:
`<USERNAME>`

Password:
`<PASSWORD>`
Open Console
#### Access Key Details
Show Access Keys
#### Key Pair Details
EC2 Key Pair Private Key: Download PEM/PPK  

---

You will need:
* `Access Key ID`
  * Click `Show Access Keys`
* `Secret Access Key`
  * Click `Show Access Keys`
* Downloaded private key `.pem`/`.ppk` (only support `.pem` for now)
  * Click `Download PEM/PPK`
* The **lab number** that you are working on
* Which operating system you want to use in AWS
  * LINUX (default)
    * Faster to spin up
    * Easier log into the box, or transfer files in a bidirectional manner
  * WINDOWS
    * Can run full featured IDE in the cloud

## How to use
### Initial Ruby setup
Pull down this repo
```bash
$ cd $HOME && git clone https://github.com/psprings/aws-labs.git && cd aws-labs
```
Make sure that you have the `bundler` gem installed
```bash
$ gem install bundler
```
Install all dependencies
```bash
$ bundle install
```
### AWS lab setup
**TL;DR** version:
```bash
$ thor help setup:easy
Usage:
  thor setup:easy

Options:
  -i, [--aws-access-key-id=AWS_ACCESS_KEY_ID]          # AWS Access Key ID
  -k, [--aws-secret-access-key=AWS_SECRET_ACCESS_KEY]  # AWS Secret Access Key
  -r, [--region=REGION]                                # AWS Region
  -o, [--os=OS]                                        # Operating system to use
  -l, [--lab-number=N]                                 # Lab number

Full setup of all services
```
```bash
$ thor setup:easy --aws-access-key-id=<AWS_ACCESS_KEY_ID> --aws-secret-access-key=<AWS_SECRET_ACCESS_KEY> --os=<LINUX or WINDOWS> --lab-number=<LABNUMBER>
```

### Full usage
To see what commands are available, run `thor list`
```bash
$ thor list
interact
--------
thor interact:atom  # Open project in Atom
thor interact:ssh   # SSH Command for given OS

setup
-----
thor setup:configure_lab_key                                # Configure key for the new lab
thor setup:easy                                             # Full setup of all services
thor setup:get_instances                                    # Get instances matching parameter
thor setup:new_lab AWS_ACCESS_KEY_ID AWS_SECRET_ACCESS_KEY  # Configure your ~/.aws/credentials file with new lab creds
thor setup:remote_sync                                      # Configure remote-sync for Atom

```
