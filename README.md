# This is Concert installation guide and how to integrate with watsonx
This following guide derived from [CE-Concert2.0-installation-guide](https://pages.github.ibm.com/skol/pe-bootcamp/concert/labsv2/labs/Lab0-setup/)


## 1 - Provision VM on techzone - [link](https://techzone.ibm.com/my/reservations/create/68a831993211144a53c49722)
using 1TB for disk/ 16CPU, 64GiB

![vm-spec-information](/images/1.png)

## 2 - Provision watsonx.ai on techzone - [link](https://techzone.ibm.com/my/reservations/create/64b8490a564e190017b8f4eb)
Preferred Region must be **AMERICAS (US-south)**

because Concert has a Ai recommendation that why we use watsonx ai as assistant

![watsonx.ai-spec-information](/images/2.png)

after provisioning, join invitation to ibm-cloud 

in my case i got invitation to **itz-watsonx-034**

### Create watsonx project and get project ID

0.At watsonx reservation details page, store **IBM Cloud Service ID** and **IBM Cloud API key** in safe place

![keep cloud api key, cloud service id](/images/8.png)

1.Select ***watsonx*** from left side bar

![Select watsonx from left side menu](/images/3.png)

2.Click ***Launch*** from the watsonx.ai tile

![Click Launch from watsonx.ai tile](/images/4.png)

3.Make sure you are already in correct cloud account (in my case it's itz-watonx-34)
Then click **Create a sandbox project**

![Click Create sandbox project](/images/5.png)

after that click your sandbox account that just got created

![Select own sandbox account](/images/6.png)

---

4.In **Manage** tab, copy ***Project ID*** and store in safe place

![Store project ID](/images/7.png)

---
### API Key - import the Service ID as part of the project

1. From watsonx.ai screen, in the **manage** tab, select **Access control** in the left side menu

![Select Access control](/images/9.png)

Click **Add collaborator** and select **Add access groups**

2. Enter your Access Group name. The access Group Name is the **IBM Cloud Service ID** on your watsonx.ai reservation page.
in my case it's

![my service id](/images/10.png)

give the access level as **Admin**, click **Add**

![my service id](/images/11.png)

3. Retrieve the **IBM entitlement API key** - [entitlement API key](https://www.ibm.com/docs/en/concert?topic=concert-obtaining-entitlement-api-key)

Follow the instruction

![go on entitlement website from link](/images/12.png)

Store the **Entitlement keys** in a safe place

![entitilement api key page](/images/13.png)

---

### Check point - right now you will have 4 secrets
- **IBM Cloud Service ID** (from watsonx.ai reservation page)
- **IBM Cloud API Key** (from watsonx.ai reservation page)
- **IBM Cloud Project ID** (from watsonx.ai manage tab section)
- **Entitlement key** (from entitlement api key page)

Save it as .txt or whatever you will ahve to use it again

![Checkpoint secret key](/images/secret_1.png)
---

---

### Access into VM

1 Click **Download SSH Key** from vm resaervation details page, we need it to connect to instance using **SSH**

also remember **Public IP**, **Root password** we will use it again in following command

![vm reservation details](/images/14.png)

We need to modify the key permission
`chmod 600 <your_key_path>/pem_ibmcloudvsi_download.pem`

in my case it will be

> `chmod 600 ./pem_ibmcloudvsi_download.pem`

2 Access to your VM instance

 `ssh -i <your_key_path>/pem_ibmcloudvsi_download.pem itzuser@<your_public_ip> -p <your_ssh_port>`

for instance

> ```ssh -i ./pem_ibmcloudvsi_download.pem itzuser@149.81.7.19 -p 2223```


3 Modify techzone machine hostname

In version 2.0.0, concert can be joined only if the machine has a FQDN known by a DNS. This is not the case of techzone VMs. You will then change the vm hostname so that it can be resolvable.
Replace **YOUR_VM_PUBLIC_IP** with the **public IP** defined in your Techzone reservation.

``sudo hostnamectl set-hostname <YOUR_VM_PUBLIC_IP>.nip.io``

> ```sudo hostnamectl set-hostname 149.81.7.19.nip.io```

Then reboot the VM

``sudo reboot``
![change hostname and reboot](/images/15.png)

---

### Prepare VM disk
1 Login to vm again

``
ssh -i <your_key_path>/pem_ibmcloudvsi_download.pem itzuser@<your_public_ip> -p <your_ssh_port>
``

2 toprepare the disk, execute the following commands:

```
sudo -i
lsblk
mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/vdd
blkid | grep /dev/vdd
```
![after running following command](/images/16.png)

3 Copy the **UUID** value and store it in safe place, run following command


```
mkdir -p /mnt/concert

cp /etc/fstab /etc/fstab.orig
vi /etc/fstab
```

- Insert this line at the end of the file:

``UUID=YOUR_UUID /mnt/concert ext4 discard,defaults,nofail 0 0``

in my case uuid is, the result will be
> UUID="f678c74d-2edc-416b-9d0c-697389658d8e"

![insert new uuid](/images/17.png)

- Save the file and continue with following command:
```
mount -a
systemctl daemon-reload
lsblk
chmod 777 /mnt/concert
```

---

## Installing IBM Concert ()
- #### make sure you are login with itzuser(not root)
1 Change the umask in .bashrc file

```
echo "umask 022" >> $HOME/.bashrc
source $HOME/.bashrc
umask
```

output should be: 0022

![umask output](/images/18.png)

2 Start installation (we will install v2.0.0.1)

```
loginctl enable-linger itzuser
cd /mnt/concert
wget https://github.com/IBM/Concert/releases/download/v2.0.0.1/ibm-concert.tar.gz
tar xfz ibm-concert.tar.gz
```

> Noted (Optional): you can check the ibm concert version from this website  [concert version release](https://github.com/IBM/Concert/releases) 

> Noted (Optional): example of change concert version: wget https://github.com/IBM/Concert/releases/download/v2.1.0/ibm-concert-x86.tar.gz

3 Create a $HOME/env.sh file
```
vi $HOME/env.sh
```

Paste the content of [download env.sh](https://pages.github.ibm.com/skol/pe-bootcamp/concert/labsv2/files/env.sh) in  **$HOME/env.sh** file 

![env.sh](/images/19.png)

> ![real example env.sh](/images/20.png)

**Save the file (:wq)**
- source the $HOME/env.sh file to set environment variables

```
source $HOME/env.sh
```

testing with 
```
echo $INSTALL_DIR
```

> output: /mnt/concert/ibm-concert

4 Configure Concert parameter file

```
cd $INSTALL_DIR
cp $INSTALL_DIR/etc/sample-params/concert-vm-quick-start-params.ini $INSTALL_DIR/etc/params.ini
```

5 Edit $INSTALL_DIR/etc/params.ini with required parameters

```
vi $INSTALL_DIR/etc/params.ini
```

```
DOCKER_EXE=podman

INSTALL_VM=true
INSTALL_CONCERT=true
IMAGE_REGISTRY_PREFIX=cp.icr.io/cp
HUB_IMAGE_REGISTRY_SUFFIX=/solis-hub
CONCERT_IMAGE_REGISTRY_SUFFIX=/concert
```

- save the file with (:wq)

6 Login to IBM Registry

```
${DOCKER_EXE} login ${IBM_REGISTRY} --username=${IBM_REGISTRY_USER} --password=${IBM_REGISTRY_PASSWORD}
```

![Login to ibm registry](/images/21.png)

7 Install Concert -> Replace ***CONCERT_USER*** and ***CONCERT_PASSWORD*** by values of your choice

in this case I will use **ibmconcert** for both value

```
$INSTALL_DIR/bin/setup --license_acceptance=y --username=<CONCERT_USER> --password=<CONCERT_PASSWORD>
```

> $INSTALL_DIR/bin/setup --license_acceptance=y --username=ibmconcert --password=ibmconcert

- The installation process take 5-7 miniutes

![Concert install successfull image](/images/22.png)

- If the installation process finished, **Concert url** is : [https://YOUR_VM_PUBLIC_IP.nip.io:12443](https://YOUR_VM_PUBLIC_IP.nip.io:12443)

> example concert url : https://149.81.7.19.nip.io:12443

- Login with username/password which you provided

![Concert login](/images/23.png)

---

### Get IBM Concert API Key

Click Key icon at the top right corner, Click **Genearte API key**

![Click key icon at top right cornner](/images/24.png)

![Generate Concert API key](/images/25.png)

![Result api key](/images/26.png)

---

## Watsonx.ai integration

1 You need to update the $HOME/env.sh file

```
vi $HOME/env.sh
```

based on the secret that we save as example:
![Secret1](/images/secret_1.png)

update following variables:
- WATSONX_API_KEY=< IBM Cloud API key at watsonx.ai reservation deatail page>
- WATSONX_API_PROJECT_ID=<watsonx.ai project id>
- WATSONX_API_URL=https://us-south.ml.cloud.ibm.com

![PROJECT_ID, API_URL](/images/28.png)

![update $HOME/env.sh](/images/27.png)

- Save the file (:wq) and source to set environment variables

```
:wq
source $HOME/env.sh
echo $WATSONX_API_KEY
```

> output: 4arAg0TUJlcOVe3U9I2QdmOyg8erZAX8abP098KNjcrD Or your watsonx.ai api key

2 Apply the watsonx.ai configuration

```
cd $INSTALL_DIR
echo "WATSONX_API_KEY=$WATSONX_API_KEY" >> ibm-concert-std/etc/local_config.env
echo "WATSONX_API_PROJECT_ID=$WATSONX_API_PROJECT_ID" >> ibm-concert-std/etc/local_config.env
echo "WATSONX_API_URL=$WATSONX_API_URL" >> ibm-concert-std/etc/local_config.env

ibm-concert-std/bin/start_service ibm-roja-py-utils
```

Test the integration

To verify that the integration with watsonx.ai is successfull, you can look at the ibm-roja-py-utils pod logs:

```
podman logs ibm-roja-py-utils
```

Last output should be something like this with mesasge : **Client successfully initialized**:
```
[itzuser@149 ibm-concert]$ podman logs ibm-roja-py-utils
{'timestamp': 2025-10-26:09:16:03, 'logLevel': info, 'callerMethod': auth.py:L48, 'message': FIPS mode is NOT enabled for cryptography.}
{'timestamp': 2025-10-26:09:16:03, 'logLevel': info, 'callerMethod': auth.py:L53, 'message': FIPS mode is NOT enabled for python.}
{'timestamp': 2025-10-26:09:16:09, 'logLevel': info, 'callerMethod': client.py:L529, 'message': Client successfully initialized}
{'timestamp': 2025-10-26:09:16:09, 'logLevel': info, 'callerMethod': genai.py:L191, 'message': Connection to watsonx.ai successful!}
info:     Started server process [4]
info:     Waiting for application startup.
info:     Application startup complete.
info:     Uvicorn running on https://0.0.0.0:20443 (Press CTRL+C to quit)
```

---

### Testing watsonx.ai already integrated

- At concert website click **question mark icon** at top navigation bar, then Click ***Load Sample data***

![Load sample data](/images/29.png)

Go on side navigation bar select 
```
Dimensions > Vulnerability > (select any cve)

on the right side you will see datails about cve, this one got generated by watsonx
```

![Load sample data](/images/30.png)

---

# For concert workflow installation guide look at ./concnert-workflow-install folder