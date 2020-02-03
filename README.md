# WebLogic-Cloud-Workshop
Provisioning a Weblogic domain with WebLogic Cloud from OCI Marketplace

# 0. Prerequisites

- [Oracle Cloud Infrastructure](https://cloud.oracle.com/en_US/cloud-infrastructure) enabled account. The tutorial has been tested using [Trial account](https://myservices.us.oraclecloud.com/mycloud/signup) (as of January, 2019).

# 1. Required Keys and OCIDs
Execute the following 3 steps as per [Required Keys and OCIDs](https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/apisigningkey.htm):

1. Create a user in IAM for the person or system who will be calling the API, and put that user in at least one IAM group with any desired permissions. See Adding Users. **You can skip this if the user exists already.**

2. Get these items:

  * RSA key pair in PEM format (minimum 2048 bits).
  * Fingerprint of the public key.
  * Tenancy's OCID and user's OCID.

3. Upload the public key from the key pair in the Console

---

**Note:** Keep a record of: **Fingerptint of the public key**, **Tenancy's OCID**, **user's OCID**, and **Path of the private key on your desktop** for later use in this lab.

---

# 2. Encode the WebLogic administrator password in base64 format

1. Choose a password with the following requirements

2. Encode the password in base64 format

  For example, on Linux:

```
$ echo -n 'Your_Password' | base64
```

---

**Note:** Keep a record of the output of the above **'echo -n 'Your_Password' | base64'** command for later use in this lab.

---

# 3. Create an SSH Key 

Create a secure shell (SSH) key pair so that you can access the compute instances in your Oracle WebLogic Server domains.

A key pair consists of a public key and a corresponding private key. When you create a domain using Oracle WebLogic Cloud, you specify the public key. You then access the compute instances from an SSH client using the private key.

On a UNIX or UNIX-like platform, use the ssh-keygen utility. For example:

```
$ ssh-keygen -b 2048 -t rsa -f mykey
    
$ cat mykey.pub  
```

---

**Note:** Keep a record of the output of the above **'cat mykey.pub'** command for later use in this lab.

---

On a Windows platform, you can use the PuTTY Key Generator utility. See [Creating a Key Pair ](https://www.oracle.com/pls/topic/lookup?ctx=en/cloud/paas/weblogic-cloud/user&id=oci_general_keypair) in the Oracle Cloud Infrastructure documentation.

# 4. Install terraform and terraform OCI provider on your desktop

Download and install terraform and the OCI Terraform Provider as in [Getting Started with the Terraform Provider](https://docs.cloud.oracle.com/en-us/iaas/Content/API/SDKDocs/terraformgetstarted.htm)

# 5. Clone this github on your laptop

Clone the WebLogic Cloud Worshop git repository to your desktop.
```
git clone https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop.git
```

# 6. Update the terraform configuration file with the specifics of your environment

1. Copy the terraform configuration variables example file

```
$ cd ~/WebLogic-Cloud-Workshop/terraform

$ cp terraform.tfvars.example terraform.tfvars
```

2. Open the terraform.tfvars file

Use your preferred editor and open the file terraform.tfvars. it should look like this:

```
# Identity and access parameters

oci_base_identity = {
  api_fingerprint      = "64:8c:3b:..."
  api_private_key_path = "/Path/to/my/private/key/mykey.pem"
  api_private_key_password = "private_key_passphrase"
  compartment_id       = "ocid1.compartment.oc1..aaaaaaaa3l..."
  tenancy_id           = "ocid1.tenancy.oc1..aaaaaaaaznlqfv..."
  user_id              = "ocid1.user.oc1..aaaaaaaajbvljcmjw..."
}

oci_base_general = {
  label_prefix = "base"
  region       = "us-MYREGION-1"
}


# Base 64 password

Base64_Password = "bWlu..."


# Infrastructure parameters (if you change either Compartment_name or Dynamic_Group_name update the statement accordingly in the Create Policy for the Dynamic group in main.tf)

Compartment_name = "WLS_Compartment"
Dynamic_Group_name = "WLS_Dynamic_Group"
Policy_name = "WLS_Policy"
Vault_name = "WLS_Vault"
Key_name = "WLS_Key"
```


3. Update the terraform.tfvars file with the specific of your environment

Update the following variables with the values your recorded earlier in the lab

- api_fingerprint            [(See section #1)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids)
- api_private_key_path       [(See section #1)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids)
- api_private_key_password   [(See section #1)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids) 
- compartment_id             (use the Tenancy OCID as per [section #1](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids))
- tenancy_id                 [(See section #1)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids))           
- user_id                    [(See section #1)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#1-required-keys-and-ocids)

- region (see note below)
---

**Note:** To confirm your home region: Open the Console, open the Region menu, and then click Manage Regions.
A list of the regions offered by Oracle Cloud Infrastructure is displayed. Select your **home region code** e.g. us-ashburn-1, us-phoenix-1.

---

- Base64_Password            [(See section #2)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#2-encode-the-weblogic-administrator-password-in-base64-format)

4. Save terraform.tfvars


# 7. Create the required infrasture to provision a Domain in WebLogic Cloud from the OCI Marketplace

1. Initialize Terraform:

```
$ terraform init
```

2. View what Terraform plans do before actually doing it:

```
$ terraform plan
```

3. Use Terraform to Provision resources:

```
$ terraform apply
```

The result has to be similar:

```
oci_kms_key.WLS_Key: Still creating... [1m10s elapsed]
oci_kms_key.WLS_Key: Still creating... [1m20s elapsed]
oci_kms_key.WLS_Key: Still creating... [1m30s elapsed]
oci_kms_key.WLS_Key: Creation complete after 1m39s [id=ocid1.key.oc1.phx.a5pc75peaafqw.abyhqlj[..........]f5tskaoaa
oci_kms_encrypted_data.WLS_Encrypted_Data: Creating...
oci_kms_encrypted_data.WLS_Encrypted_Data: Creation complete after 2s [id=]

Apply complete! Resources: 3 added, 0 changed, 2 destroyed.

Outputs:

Encrypted_data = IcsoJqtmC[..........]WJcMcUgAAAAA=
compartment_id = ocid1.compartment.oc1..aaaaaaaa[..........]hlybyag3ibeza
cryptographic_endpoint = https://a5[..........]w-crypto.kms.us-phoenix-1.oraclecloud.com
key_OCID = ocid1.key.oc1.phx.a5pc75peaafqw.abyhqlj[..........]f5tskaoaa
```

---

**Note:** Keep a record ofof the output of the above **Encrypted_data**, **cryptographic_endpoint**, and **key_OCI** for later use in this lab.

---


# 8. Provision a Domain in WebLogic Cloud from the OCI Markeplace

**Launch a Stack**

Sign in to Marketplace and specify initial stack information.

1. Sign in to the Oracle Cloud Infrastructure Console.

2. Click the ![alt text](images/main_menu_icon.png) Navigation Menu icon and select Marketplace.

![alt text](images/image040.png)

3. You can choose to browser-search for WebLogic Cloud, or for faster search you can apply the filters:

Type: Stack
Publisher: Oracle
Category: Application Development

![alt text](images/image050.png)

3. Choose **WebLogic Cloud Enterprise Edition BYOL**; This brings you to the Stack Overview page:

![alt text](images/image060.png)

4. Select a version of Oracle WebLogic Server 12c.
If there's more than one 12c patch, the latest patch is the default.

5. Select the compartment **"WLS_Compartment"** in which we will create the stack.

![alt text](images/image061.png)

By default the stack compartment is used to contain the domain compute instances and network resources. If later on you specify a network compartment on the Configure Variables page of the Create Stack wizard, then only the compute instances are created in the stack compartment that you select here.

6. Select the Terms and Restrictions check box, and then click Launch Stack.

![alt text](images/image062.png)

The Create Stack wizard is displayed.

7. Specify Stack Information

![alt text](images/image070.png)

Specify the name, description, and tags for the stack.
a. On the Stack Information page of the Create Stack wizard, enter a name for your stack.

b. Enter a description for the stack (optional).

c. Specify one or more tags for your stack (optional).

d. Click Next.

The Configure Variables page opens.

8. Configure WebLogic Instance Parameters

![alt text](images/image071.png)

Specify the parameters needed to configure the WebLogic instance domain.

a. In the WebLogic Server Instance section, enter the resource name prefix.(The maximum character length is 8.
This prefix is used by all the created resources.)

b. Select the WebLogic Server shape for the compute instances: **VM.Standard2.1**. (Fyi, only the following shapes are supported: VM.Standard2.x, VM.Standard.E2.x, BM.Standard2.x, BM.Standard.E2.x 

c. Enter the SSH public key. [(See section #3)](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/tree/readme#3-create-an-ssh-key)

d. Select the availability domain where you want to create the domain.**Choose one of the displyed ADs**

e. Select the number of managed servers you want to create. **Select 2**
The managed servers will be members of a cluster, unless you selected WebLogic Server Standard Edition.

f. Enter a user name for the WebLogic Server administrator. **Enter weblogic**

g. Enter an encrypted password for the WebLogic Server administrator. **Enter the value of Encrypted-data in the output of the terraform apply command in [section #7](https://github.com/StephaneMoriceau/WebLogic-Cloud-Workshop/blob/readme/README.md#7-create-the-required-infrasture-to-provision-a-domain-in-weblogic-cloud-from-the-oci-marketplace)**

9. Configure Advanced Parameters for a Domain

![alt text](images/image072.png)

a. Don't change / select WebLogic Server Instance Advanced Configuration

b. Network Compartment: Select **WLS_compartment**

c. VCN Strategy: Select **Create New VCN**

d. WLS Network: Enter **WLSCloudVCN** 

e. WLS Network CIDR: Keep the default

f. Subnet Strategy: Select **Create New Subnet**

g. Subnet Type: Keep the default **Use Public Subnet** selection.

![alt text](images/image073.png)

h. Subnet span: Select **Regional Subnet**

i. Select **Provision Load Balancer**

j. LB Network CIDR: Keep the default

k. LB Shape: Select **400Mbps**

l. Do **NOT** select **Prepare Load Balancer for https**

m. Do **NOT** select **Enable authentification using Identity Cloud Service**

![alt text](images/image074.png)

n. Database Strategy: keep the default **No Database**

o. Key Management Service Key ID: Enter the value Key Management Service Configuration section of the Configure Variables page, enter the OCID of your encryption key.

p. Key Management Service Endpoint: Enter the endpoint URL for the vault that contains your encryption key.

q. At the bottom of the Configure Variables page, click **Next**

You are now ready to create the stack.

r. Review the Stack configuration and Click **Create**
















Configure Tags
Oracle WebLogic Cloud can optionally assign tags to the resources (compute, network, and so on) that it creates for your domain.

Tagging allows you to define keys and values and associate them with resources. You can then use the tags to help you organize and find resources based on your business needs. There are separate fields to tag the stack and to tag the resources created within the stack.

To assign an existing tag, enter the Defined Tag Key and Defined Tag Value.
Specify the name of a defined tag using the format <namespace>.<key>. For example, Operations.CostCenter.
To assign a free-form tag, enter the Free-Form Tag Key and Free-Form Tag Value.
Free-form tag keys and values are case sensitive. For example, costcenter and CostCenter are treated as different tags.
Create the Domain Stack
After you have specified the WebLogic instance variables, finish creating the domain stack.

On the Review page of the Create Stack wizard, review the information you have provided, and then click Create.

The Job Details page of the stack in Resource Manager is displayed. A stack creation job name has the format ormjobyyyymmddnnnnnn. (for example, ormjob20190919165004). Periodically monitor the progress of the job until it is finished. If an email address is associated with your user profile, you will receive an email notification.
Note:If you run an Apply job on an existing stack that you created with Oracle WebLogic Cloud, all resources in the stack will be deleted and recreated.
Use Your New Domain
Access and manage your new domain after creating a stack with Oracle WebLogic Cloud.

Typical tasks that you might perform after creating a domain:
View and manage the cloud resources that were created to support your domain. See View the Cloud Resources for a Domain.
Use the WebLogic Server administration console to configure your domain. Create data sources, JMS modules, Coherence clusters, and so on, or deploy applications. See Access the WebLogic Console.
Access the sample application that's deployed to your domain. See Access the Sample Application.
Secure access to your applications using Oracle Identity Cloud Service. See Secure a Domain Using Identity Cloud Service.
If you selected the HTTPS option for the load balancer, you must add your SSL certificate to the load balancer. See Configure SSL for a Domain.
Troubleshoot a problem with your new stack. See Stack Creation Failed.
