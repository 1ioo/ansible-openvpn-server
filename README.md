# OpenVPN Server Set Up with Ansible
This Ansible Playbook offers a streamlined solution for automating the setup and management of an OpenVPN server on a Linux system. Leveraging Ansible introduces the powerful principle of idempotency, guaranteeing that the results remain consistent, no matter how frequently the Playbook is executed. This inherent characteristic simplifies updates, especially when fine-tuning specific sections of the Playbook.

This Playbook will be ran on the Control Node (Device controlling Servers).

Before running our Playbook, we would need to set up our environment (Run on Control Node):
```
sudo apt update && sudo apt upgrade -y
sudo apt install ansible sshpass -y
sudo mkdir /etc/ansible
```

# Table of Contents
1. [Part I: Source Files](#part-i-source-files)
2. [Part II: Usage](#part-ii-usage)
3. [Menu](#part-iii-menu)
4. [Roles](#part-iv-roles)
    1. [serverSetUp](#1-serversetup)
    2. [createClientCertificate](#2-createclientcertificate)
    3. [revokeClientCertificate](#3-revokeclientcertificate)

## Part I: Source Files
Files needed in order for the Ansible Script to Work:
```
├── /etc/ansible/                                       # Ansible Folder where all Playbooks are Stored
│   ├── /main.yml                                       # Main Playbook to be Ran
│   ├── /hosts                                          # Host File to Specify servers to be affected by Ansible
│   ├── /ansible.cfg                                    # Ansible Configuration Settings
│   ├── /roles                                          # Roles Folder containing Roles
│   │   ├── /serverSetUp/                               # Folder containing Files related to the serverSetUp role
│   │   |   ├── tasks/                                  # Folder containing Tasks called by the serverSetUp role
│   │   |   |   ├── /ipForward.yml                     # Task File to add PostUp and PostDown IP Forwarding
│   │   |   |   ├── /createServerConfig.yml             # Task File to create Server Configuration
│   │   |   |   ├── /embedCertificates.yml              # Task File to embed cryptographic components in Configuration
│   │   |   |   ├── /generateServerCertificates.yml     # Task File to Generate Server Certificates
│   │   |   |   ├── /main.yml                           # Main Task File for the serverSetUp role
│   │   |   |   ├── /setUpFail2Ban.yml                  # Task File to set up Fail2Ban
│   │   |   |   ├── /setUpIPTables.yml                  # Task File to establish IPTable Rules
│   │   |   ├── vars/                                   # Folder containing files that define variables
│   │   |   |   ├── /main.yml                           # Main vars file to define variables
│   │   |   ├── handlers/                               # Folder to contain handlers
│   │   |   |   ├── /main.yml                           # Main handler file
│   │   ├── /createClientCertificate/                   # Folder containing Files related to the createClientCertificate role
│   │   |   ├── tasks/                                  # Folder containing Tasks called by the serverSetUp role
│   │   |   |   ├── /main.yml                           # Main Task File
│   │   |   |   ├── /embedCertificates.yml              # Task file to embed Client Certificates in Client Configuration
│   │   |   |   ├── /createClientConfig.yml             # Task File to create Client Configuration
│   │   |   ├── vars/                                   # Vars Folder
│   │   |   |   ├── /main.yml                           # Main Vars File
│   │   ├── /revokeClientCertificate/                   # Folder containing Files related to the revokeClientCertificate role
│   │   |   ├── tasks/                                  # Task Folder
│   │   |   |   ├── /main.yml                           # Main Task File
│   │   |   ├── vars/                                   # Vars Folder
│   │   |   |   ├── /main.yml                           # Main Vars File
│   │   |   ├── handlers/                               # Handlers Folder
│   │   |   |   ├── /main.yml                           # Main Handlers File
```

## Part II: Usage
Ansible should be ran from the `/etc/ansible/main.yml` playbook.

Command to Run Ansible Playbook:
```
sudo ansible-playbook main.yml
```

Prior to executing the script, it is highly encouraged to perform a system update and upgrade, ensuring that your system has the latest software versions. This proactive step not only saves time but also optimizes resource utilization for a smoother script execution. It's worth noting that the script itself will handle system update and upgrade if it hasn't been done already, making this step optional.

Run the following command to Update and Upgrade the System:
```
sudo apt update && sudo apt upgrade -y
```

## Part III: Menu
The Playbook incorporates a user-friendly interface, allowing users to make selections tailored to their specific needs or purposes.

```
#################################################################################
#################################################################################
##   ___                __     ______  _   _   ____                            ##
##  / _ \ _ __   ___ _ _\ \   / /  _ \| \ | | / ___|  ___ _ ____   _____ _ __  ##
## | | | | '_ \ / _ \ '_ \ \ / /| |_) |  \| | \___ \ / _ \ '__\ \ / / _ \ '__| ##
## | |_| | |_) |  __/ | | \ V / |  __/| |\  |  ___) |  __/ |   \ V /  __/ |    ##
##  \___/| .__/ \___|_| |_|\_/  |_|   |_| \_| |____/ \___|_|    \_/ \___|_|    ##
##       |_|                                                                   ##
#################################################################################
#################################################################################
=================================================================================
1- Set Up OpenVPNServer
2- Create Client Certificates
3- Revoke Client Certificate
Enter Choice >:
```

Each user selection within the script directs the user to a specific role, each designed to fulfill a unique purpose. These roles encapsulate distinct functionalities, enabling a modular and organized approach to the automation process. Whether it's server setup, client certificate creation, or other tasks, the user is guided to the appropriate role for seamless and purpose-driven execution.

## Part IV: Roles
1. [serverSetUp](#1-serversetup)
2. [createClientCertificate](#2-createclientcertificate)
3. [revokeClientCertificate](#3-revokeclientcertificate)

### 1. serverSetUp
This role performs the following steps to set up the OpenVPN Server:
1. [Updating and Upgrading of System](#1-updating-and-upgrading-of-system)
2. [Installation of Packages](#2-installation-of-packages)
3. [Generation of Server Certificates and Keys](#3-generation-of-server-certificates-and-keys)
4. [Creation of Server Configuration](#4-creation-of-server-configuration)
5. [Server Hardening](#5-server-hardening)

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| serverDirectory | OpenVPN Server Directory |
| serverConfigPath | Full Path of OpenVPN Server Configuration File |
| pkiDirectory | Easy-RSA PKI Directory |
| easyrsaDirectory | Easy-RSA Directory |
| chains | INPUT and OUTPUT chains (edited in Playbook) |
| sshPort | Port to set SSH to Run On |
| openvpnPort | Port to set OpenVPN to Run On |
| dnsServer1 | Primary DNS Server for Server to Push to Client |
| dnsServer2 | Secondary DNS Server for Server to Push to Client |
| ipv4Interface | WAN Interface of System |

<u>_Features (Tasks)_</u>

##### 1. Updating and Upgrading of System
As the first step, the Playbook initiates the updating and upgrading of the system. This essential process ensures that both the system and its package manager are brought up to the latest versions, providing improved security, bug fixes, and access to the latest features and enhancements.

##### 2. Installation of Packages
Necessary packages required for setting up the OpenVPN Server are installed.

List of Packages Installed by Script:
- easy-rsa
- iptables
- iptables-persistent
- openvpn
- fail2ban
- rsyslog
- python3-pexpect

**_NOTE:_**  The installation of `rsyslog` is included to revert back to using `/var/log/auth.log` on Linux systems (such as Debian) that utilize journalctl to store authorization information. If your system already uses `/var/log/auth.log` for authorization information, the installation of this package is not necessary.

##### 3. Generation of Server Certificates and Keys
The server is managed using `Easy-RSA`, which is employed to handle the server's Public Key Infrastructure (PKI). All key and certificate creation and revocation tasks are managed through Easy-RSA. 

##### 4. Creation of Server Configuration
Following the generation of server certificates and keys, the next crucial step involves embedding these cryptographic components into the OpenVPN server's configuration file. This file serves as a blueprint for the server's behavior, detailing the contents of certificates, private keys, CA certificates, and other essential parameters. Once the configuration is set up, initiating the OpenVPN server involves specifying this file's path. The server reads the configuration, loads the embedded certificates and keys, and establishes a secure environment for encrypted communication with clients. This streamlined process ensures that the OpenVPN server operates with the specified security parameters, facilitating robust and protected network communication.

##### 5. Server Hardening
The server undergoes a robust hardening process, incorporating SSH Hardening, the establishment of IPTable Rules, and the implementation of Fail2Ban. These measures collectively enhance the security posture by fortifying the SSH configuration, defining strict IP filtering rules, and providing protection against potential security threats through the proactive monitoring and banning of suspicious activities using Fail2Ban.

### 2. createClientCertificate
This role performs the following steps to create Client Configurations:
1. [Generation of Client Certificate and Keys](#1-generation-of-client-certificate-and-keys)
2. [Creation of Client Configuration](#2-creation-of-client-configuration)

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| serverDirectory | OpenVPN Server Directory |
| pkiDirectory | Easy-RSA PKI Directory |
| openvpnPort | Port to set OpenVPN to Run On |
| openvpnIP | IP Address of OpenVPN Server |)

<u>_Features (Tasks)_</u>

##### 1. Generation of Client Certificate and Keys
The server is managed using `Easy-RSA`, which is employed to handle the server's Public Key Infrastructure (PKI). All key and certificate creation and revocation tasks are managed through Easy-RSA. 

##### 2. Creation of Client Configuration
Following the generation of client's certificates and keys, the next crucial step involves embedding these cryptographic components into the client's configuration file. This file serves as a blueprint for the server's behavior, detailing the contents of certificates, private keys, CA certificates, and other essential parameters. Once the configuration is set up, initiating the connection to the OpenVPN server involves specifying this file's path. The client's system reads the configuration, loads the embedded certificates and keys, and establishes a secure environment for encrypted communication with the OpenVPN server.

**_NOTE_**: Client Configuration File will be named after the Client, followed by either `.conf` or `.ovpn`, depending on what Operating System the Client is using to connect to the OpenVPN Server. After the Client Configuration has been created and configured, it will be relocated to the `/tmp` folder, facilitating easy retrieval by the client through transfer methods like using `scp`.

<u>_Connecting to the OpenVPN Server_</u>

Before connecting to the OpenVPN Server, it is best for the client to Update and Upgrade the System by Running:
```
sudo apt update && sudo apt upgrade -y
```

For Clients who are using Linux Systems, Run the Following:
```
sudo apt install openvpn-systemd-resolved
```

This allows the Client's System to directly update the DNS settings of a link through `systemd-resolved` and use the pushed DNS Servers given by the OpenVPN Server.

Clients can connect to the OpenVPN Server by running:
```
sudo openvpn <openvpn config file>
```

To enable OpenVPN to Connect at StartUp:
```
sudo systemctl enable --now openvpn-client@<clientname>
```

**_NOTE_**: Make sure that Client Configuration is stored within `/etc/openvpn/client`.

### 3. revokeClientCertificate
This role performs the following steps to deny Client's Access to the OpenVPN Server:
1. [Revocation of Client Certificate](#1-revocation-of-client-certificate)
2. [Revocation of Client's Access to OpenVPN Server](#2-revocation-of-clients-access-to-openvpn-server)

<u>_Variables_</u>

Certain Variables can be changed as per your needs. Relevant description on the different variables will be displayed below.

| Variable Name | Description |
| :- | :- |
| serverDirectory | OpenVPN Server Directory |
| pkiDirectory | Easy-RSA PKI Directory |

<u>_Features (Tasks)_</u>

##### 1. Revocation of Client Certificate
Utilizing `Easy-RSA`, the script initiates the revocation of the selected client's certificate, relocating it to the `revoked` folder. This process invalidates the previously signed certificate, rendering it unusable for authentication. Upon certificate revocation, a `crl.pem` file is generated, serving as a certificate revocation list (CRL) that enumerates the revoked certificates. 

##### 2. Revocation of Client's Access to OpenVPN Server
After `crl.pem` has been created, its location will be linked with the `crl-verify` parameter within the server's configuration file. If a client uses a certificate that is found within this file, the client would be denied access to the OpenVPN Server.