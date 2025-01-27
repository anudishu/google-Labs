

 ![Image Alt](https://github.com/anudishu/google-Labs/blob/master/Screenshot%202025-01-27%20115138.jpg?raw=true)




# Configure Secure RDP using a Windows Bastion Host
# https://www.qwiklabs.com/focuses/1737?parent=catalog

#Task1. Create the VPC network

  #  1 : A new non-default VPC has been created
    gcloud compute networks create securenetwork --subnet-mode=custom

  # 2 : The new VPC contains a new non-default subnet within it
    gcloud compute networks subnets create securenetwork --network=securenetwork --region=us-east4 --range=192.168.1.0/24

 #  3 : A firewall rule exists that allows TCP port 3389 traffic ( for RDP )
    gcloud compute firewall-rules create myfirewalls --network securenetwork --allow=tcp:3389 --target-tags=rdp



#Task 2. Deploy your Windows instances and configure user passwords
#1.Deploy a Windows 2016 server (Server with Desktop Experience) instance called vm-securehost with two network interfaces in the zone zone.
Configure the first network interface with an internal only connection to the newly created VPC subnet.
The second network interface with an internal only connection to the default VPC network. This is the secure server.

 gcloud compute instances create vm-securehost \
  --zone=us-east4-a \
  --machine-type=n1-standard-2 \
  --network-interface=subnet=securenetwork,no-address \
  --network-interface=subnet=default,no-address \
  --maintenance-policy=MIGRATE \
  --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
  --tags=rdp \
  --image=windows-server-2016-dc-v20200211 \
  --image-project=windows-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-standard \
  --boot-disk-device-name=vm-securehost \
  --reservation-affinity=any


#2.Deploy a second Windows 2016 server (Server with Desktop Experience) instance called vm-bastionhost with two network interfaces in the zone zone.
Configure the first network interface to connect to the newly created VPC subnet with an ephemeral public (external NAT) address.
The second network interface with an internal only connection to the default VPC network. This is the jump box or bastion host.

 gcloud compute instances create vm-bastionhost \
  --zone=us-east4-a \
  --machine-type=n1-standard-2 \
  --network-interface=subnet=securenetwork,network-tier=PREMIUM \
  --network-interface=subnet=default,no-address \
  --maintenance-policy=MIGRATE \
  --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append \
  --tags=rdp \
  --image=windows-server-2016-dc-v20200211 \
  --image-project=windows-cloud \
  --boot-disk-size=50GB \
  --boot-disk-type=pd-standard \
  --boot-disk-device-name=vm-bastionhost \
  --reservation-affinity=any


#Connect to the secure host and configure Internet Information Server
#The following gcloud command creates a new user called app-admin and resets the password for a host called vm-bastionhost located in the us-east4-a zone:

gcloud compute reset-windows-password vm-bastionhost --user app-admin --zone us-east4-a


#The following gcloud command creates a new user called app-admin and resets the password for a host called vm-securehost located in the us-east4-a zone:

gcloud compute reset-windows-password vm-securehost --user app_admin --zone us-east4-a


Task 3. Connect to the secure host and configure Internet Information Server
# Install Chrome RDP for Google Cloud Platform (https://chrome.google.com/webstore/detail/chrome-rdp-for-google-clo/mpbbnannobiobpnfblimoapbephgifkm)
# Go to Compute Engine > VM instances
# Click RDP on vm-bastionhost, fill username with app_admin and password with your copied vm-bastionhost's password 
# Click Search, search for Remote Desktop Connection and run it
# Copy and paste the internal ip from vm-securehost, click Connect
# Fill username with app_admin and password with your copied vm-securehost's password 
# Click Search, type Powershell, right click and Run as Administrator
# Run: Install-WindowsFeature -name Web-Server -IncludeManagementTools

