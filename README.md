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

![basic-wazuh-workflow](https://user-images.githubusercontent.com/79780921/129677319-ea0f0cd4-cfb9-4c57-a7a8-59c78fdf2a9d.png)


## Workflow
**Wazuh Manager** collects logs from **Wazuh Agents**. **Filebeat** then sends logs from **Wazuh Manager** to **Elasticsearch**. **Kibana Dashboard** visualize the logs from **Elasticsearch**. In this diployment, we'll use following nodes.

| Deployment | OS | RAM |
|------------ | ------------- | ----- |
|Wazuh Manager & Filebeat | Ubuntu Server 20.04 | 1 GB|
|Elasticsearch | Ubuntu Server 20.04 | 1GB|
|Kibana Dashboard & Wazuh Agent-1 | Kali Linux | 2 GB|
|Wazuh Agent-2 | Ubuntu Server 20.04 | 1 GB|
|Wazuh Agent-3 | Windows XP | 512 MB|

In this deployment same Kali Linux used for agent-1 and Kibana.

#### The step-by-step deployment procedures are described below.
- [Wazuh Manager Setup](./wazuh-manager-setup.md)

- [Elasticsearch Setup](./elasticsearch-setup.md)

- [Filebeat Setup](./filebeat-setup.md)

- [Kibana Setup](./kibana-setup.md)

- [Wazuh Agent Setup](./wazuh-agent-setup.md)

- [Certificates Creation and Deployment](./certificates-creation-and-deployment.md)

## Source
https://documentation.wazuh.com/current/index.html
