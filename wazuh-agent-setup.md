## Wazuh Agent Installation in Linux
This process will go through the installation of the **Wazuh Agent** in a 1 GB RAM **Ubuntu Server 20.04** node, 2 GB **Kali Linux**.

**Note:** Root user privileges are required to execute all the following commands.

### Adding the Wazuh repository
1. Install the necessary packages for the installation:
```shell
# apt install curl apt-transport-https lsb-release gnupg2
```
2. Install the GPG key:
```shell
# curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | apt-key add -
```
3. Add the repository:
```shell
# echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```
4. Update the package information:
```shell
# apt update
```

### Installing the Wazuh Agents
1. Install the Wazuh Agent package:
```shell
# WAZUH_MANAGER="10.0.2.15" apt-get install wazuh-agent
```
Replace the IP with your **Wazuh Manager's** host IP.
2. Enable and start the Wazuh Agent service:
```shell
# systemctl daemon-reload
# systemctl enable wazuh-agent
# systemctl start wazuh-agent
```
3. Run the following command to check if the Wazuh Agent is active:
```shell
# systemctl status wazuh-agent
```

### Disabling Wazuh Repository
It is recommended to disabling the Wazuh repository to prevent accidental upgrades. To do so, use the following command:
```shell
# sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/wazuh.list
# apt update
```

## Wazuh Agent Installation in Windows
This process will go through the installation of the **Wazuh Agent** in a 512 MB **Windows XP** node. But the same process will be applicable to the later versions.

**Note:** To perform the installation, administrator privileges are required.

1. To start the installation process, download the [Windows installer](https://packages.wazuh.com/4.x/windows/wazuh-agent-4.1.5-1.msi).

2. Open CMD as an administrator and run this command:
```cli
wazuh-agent-4.1.5-1.msi /q WAZUH_MANAGER="10.0.2.15" WAZUH_REGISTRATION_SERVER="10.0.2.15"
```
**OR**, Open Powershell as an administrator and run this command:
```powershell
.\wazuh-agent-4.1.5-1.msi /q WAZUH_MANAGER="10.0.2.15" WAZUH_REGISTRATION_SERVER="10.0.2.15"
```
Replace the IP with your **Wazuh Manager's** host IP.

By default, all agent files are stored in `C:\Program Files (x86)\ossec-agent` after the installation.


### Miscellaneous
Now go to the host where **Wazuh Manager** installed and run this command
1. To check connected agents:
```shell
# sudo /var/ossec/bin/manage_agents -l
```
4. To check agents status:
```shell
# sudo /var/ossec/bin/agent_control -l
```
5. To check logs from agents:
```shell
# sudo tail -f /var/ossec/logs/alerts/alerts.log
```

**NEXT:** [Certificates and Authentication Deployment](../certificates-creation-and-deployment)
