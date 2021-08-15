# What is Wazuh?
Wazuh is a free, open source and enterprise-ready security monitoring solution for threat detection, integrity monitoring, incident response and regulatory compliance. It can be used to monitor endpoints, cloud services and containers, and to aggregate and analyze data from external sources. Wazuh is a comprehensive **SIEM** solution provides the following capabilities:
* Security Analytics
* Intrusion Detection
* Log Data Analysis
* File Integrity Monitoring
* Vulnerability Detection
* Configuration Assessment
* Incident Response
* Regulatory Compliance
* Cloud Security Monitoring
* Containers Security

We will deploy a basic wazuh **SIEM** solution in VirtualBox as bleow.

![Basic Wazuh Workflow](https://github.com/mriazx/wazuh-setup/blob/main/basic-wazuh-workflow.png)

## Workflow
**Wazuh Manager** collects logs from **Wazuh Agents**. **Filebeat** then sends logs from **Wazuh Manager** to **Elasticsearch**. **Kibana Dashboard** visualize the logs from **Elasticsearch**. In this diployment, we'll use following nodes.
Deployment | OS | RAM
------------ | ------------- | -----
Wazuh Agent-1 | Kali Linux | 2 GB
Wazuh Agent-2 | Ubuntu Server 20.04 | 1 GB
Wazuh Agent-3 | Windows XP | 512 MB
Wazuh Manager & Filebeat | Ubuntu Server 20.04 | 1 GB
Elasticsearch | Ubuntu Server 20.04 | 1GB
Kibana Dashboard | Kali Linux | 2 GB

In this deployment same Kali Linux used for agent-1 and Kibana.

#### The setep-by-step deployment procedures are described below.
[Wazuh Manager Setup](https://github.com/mriazx/wazuh-setup/blob/main/wazuh-manager-setup.md)

[Elasticsearch Setup](https://github.com/mriazx/wazuh-setup/blob/main/elasticsearch-setup.md)

[Filebeat Setup](https://github.com/mriazx/wazuh-setup/blob/main/filebeat-setup.md)

[Kibana Setup](https://github.com/mriazx/wazuh-setup/blob/main/kibana-setup.md)

[Wazuh Agent Setup]()

[Security Deployment]()
