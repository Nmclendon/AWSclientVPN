# AWSclientVPN
CNE 430 final where we configure a client VPN deployment allowing workstations to connect to a AWS VPC in a safe and secure way. Contributors to this project are Theodora Kochel, Daothid Tern, and Nicholas McLendon

### Step 1: Create Environment

1. Navigate to the CloudFormation dashboard and select "Create stack" in the top right of the window.

2. Select "Choose an existing template".

3. Under "Template source", select "Upload template file" and upload the provided `.yaml` file to configure the stack.

4. Click "Next".

5. Name your stack whatever you like on the top of the next page, we recommend keeping the name similar to the naming convention in the config.yaml file, then click "Next".

6. Leave all settings on the following page at their defaults and click "Next" again.

7. On the final page, click "Submit".

**.yaml file**

- Please see the yaml directory for the configuration file, and information on the modifications you will need to make to it.

### Step 2: Create Simple AD Service

1. Navigate to the "Directory Service" dashboard and select "Set up a directory"
2. Choose "Simple AD" as the directory type then click next.
3. Select "small" for the directory size, give it a Directory DNS name we recommend "corp.<project_name>.com"
4. Set the Administrator password and make sure you either write down or save that password in some way as you will need it later. Click "Next"
5. for the VPC, make sure you select the VPC that was created by the CloudFront stack deployment
6. Under "subnet" pick either "<project_name>-SN-PRIV-A" or "<project_name>-SN-PRIV-B" and click "Next"
7. Click "Create directory"

### Step 3: Generate Certificates

**For this next step you will want to make sure you have git is installed on whatever machine you'll be using to generate these certificates. We did this on an Ubuntu EC2.**

1. Open a terminal window and navigate to the /tmp directory.

   - cd /tmp

2. Clone the easy-rsa repository from

   - git clonehttps://github.com/OpenVPN/easy-rsa.git

3. Navigate to the easyrsa3 directory and creake your keys, certificate authority, and certificate.

- cd easy-rsa/easyrsa3
- ./easyrsa init-pki
- ./easyrsa build-ca <project_name>

  <Project_name>VPN

- ./easyrsa build-server-full server <project_name>.com

4. Open the AWS "Certificate manager" and click on "Import at the top right of the screen. 

5. Back on your EC2, navigate to the "issued" subdirectory inside the pki "directory" copy the body of the <project_name>.com.crt file and paste it into the "Certificate body" box in the AWS Certificate Manager".

- cd pki/issued
- nano <project_name>.com.crt

6. Navigate to the "private" subdirectory inside the "pki" directory. Copy the body (large jumble of seemingly random characters at the bottom) of the <project_name>.key file and paste it into the "Certificate private key" box in the AWS Certificate Manager, and then click "Import Certificate"

- cd ..
- cd private
- nano <project_name>.key

### Step 4: Create a VPN Endpoint

1. Navigate to the "VPC" dashboard in AWS.

2. On the left-hand side find the "Virtual Private Network" section and select "Client VPN endpoints"

3. Click on "Create client VPN endpoint" name your VPN end point, for "Name" enter <project_name> "Client VPN, under Client IPv4 CIDR" enter 192.168.12.0/22,

4. Click the "Server certificate ARN" dropdown and select the server certificate you imported in step 3, then under Authentication Options check "Use user-based authentication"

6. Check "Active Directory authentication", then under "Directory ID" chose the directory you created in step 2

7. Under "Connection Logging", "Do you want to log the details on client connections?" select no

8. For DNS Server 1 IP address and DNS Server 2 IP address we need to enter the IP addresses of the directory service instance. Go back to the tab with the directory service console open, click the directory service instance ID , locate the DNS address area and copy one IP into each of the DNS Server IP boxes.

9. Check Enable split-tunnel, the in the VPC ID dropdown select the VPC that was created in the step 1 CloudFormation stack deployment

10. Ensure that both the Default Security Group and the "<project_name>-GeneralSG" Security Group are selected

11. Check Enable self-service portal, then click Create Client VPN Endpoint

### Step 5: Configure and Associate VPN Endpoint

1. From the Client VPN Endpoints area of the VPC console, select the VPN endpoint your created in step 4.

2. Select the "Target Network Associations" tab and click "Associate Target Network"

3. Click the VPC* dropdown and click the VPC you created in step 1 with the CloudFormation stack deployment

4. Open in a new tab, navigate to the VPC dashboard, click on "Subnets" and select your VPC from the "Filter by VPC" dropdown menu. 

5. Locate the subnet ID for the 3 private subnets in the project VPC

6. Back in the "Associate Target Network" tab, under the "choose a subnet to associate" dropdown menu, pick the first available PRIV subnet from the list (PRIV-A, PRIV-B, PRIV-C)

7. Click Associate then wait for the VPN Endpoint to finish associating with the target network, this can take a few minutes

8. In the Client VPN Dashboard select your VPN endpoint and select the "Authorization rules" tab, then click "Add Authorization rule"

9. For "Destination network to enable" enter 10.16.0.0/16, for "Grant access to" check Allow access to all users, then click Add Authorization Rule and wait for it to load.

## Usage

1. Select the Endpoint and click "Download Client Configuration" at the top right of the window.

2. Go to https://aws.amazon.com/vpn/client-vpn-download/ and download the VPN client for your operating system, and install the VPN Application

3. Start the client VPN application, click on "File" then "manage profiles" then "add a profile"

4. Make a display name and load the profile you downloaded (client configuration)

5. Select the profile you just created and click "Connect"

6. Enter "Administrator" for the username and enter the password you set in step 2 for the Directory Service

7. Open a terminal on your machine and ping one of the IP addresses associated witht eh directory servive (The same IP addresses you used for "DNS Server 1 IP" and "DNS Server 1 IP" in step 4). If this works then you are now connected to your VPN. Congradulations!

## Conclusion

In this project, we successfully set up a secure client VPN deployment allowing workstations to connect to an AWS VPC. By following the steps outlined, we created the necessary infrastructure, including a VPC, Simple AD service, a VPN endpoint, and ensured proper configuration and association. The deployment facilitates secure and authenticated connections to AWS resources, enhancing the security and accessibility of the environment. By leveraging AWS services, we achieved a robust and scalable VPN solution, ensuring that users can safely connect to the network and access critical resources.

Thank you for following along, and we hope this guide has been helpful in understanding and deploying an AWS client VPN.

## Acknowledgement

Special thanks to acantril and their "learn-cantrill-io-labs" repository where we found the guide and configuration file we used to complete this project. Please check out their repository if you'd like to learn more about working with AWS.
