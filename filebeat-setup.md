## Filebeat Installation
This process will go through the installation of the **Filebeat** in a 1 GB RAM **Ubuntu Server 20.04**, same node where the **Wazuh Manager** installed.

**Note:** Root user privileges are required to execute all the following commands.

### Adding the Elastic Stack repository
1. Install the GPG key:
```shell
# curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
```
2. Add the repository:
```shell
# echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```
3. Update the package information:
```shell
# apt update
```

### Filebeat installation and configuration
1. Install the Filebeat package:
```shell
# apt install filebeat=7.11.2
```
2. Download the pre-configured Filebeat config file used to forward Wazuh alerts to Elasticsearch:
```shell
# curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/resources/4.1/elastic-stack/filebeat/7.x/filebeat.yml
```
3. Download the alerts template for Elasticsearch:
```shell
# curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/4.1/extensions/elasticsearch/7.x/wazuh-template.json
# chmod go+r /etc/filebeat/wazuh-template.json
```
4. Download the Wazuh module for Filebeat:
```shell
# curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.1.tar.gz | tar -xvz -C /usr/share/filebeat/module
```
5. Edit the file `/etc/filebeat/filebeat.yml`:
```shell
nano /etc/filebeat/filebeat.yml
```
It will look like this:

```shell
# Wazuh - Filebeat configuration file
output.elasticsearch.hosts: <elasticsearch_ip>:9200
output.elasticsearch.password: <elasticsearch_password>

filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: false

setup.template.json.enabled: true
setup.template.json.path: /etc/filebeat/wazuh-template.json
setup.template.json.name: wazuh
setup.template.overwrite: true
setup.ilm.enabled: false

output.elasticsearch.protocol: https
output.elasticsearch.ssl.certificate: /etc/filebeat/certs/filebeat.crt
output.elasticsearch.ssl.key: /etc/filebeat/certs/filebeat.key
output.elasticsearch.ssl.certificate_authorities: /etc/filebeat/certs/ca/ca.crt
output.elasticsearch.username: elastic
```
Replace elasticsearch_ip with the IP address or the hostname of the Elasticsearch server and update protocol to 'http' from 'https'. Then comment out authentication and certification. We will do it later. Updated file should like this:

```shell
# Wazuh - Filebeat configuration file
output.elasticsearch.hosts: 10.0.2.11:9200
#output.elasticsearch.password: <elasticsearch_password>

filebeat.modules:
  - module: wazuh
    alerts:
      enabled: true
    archives:
      enabled: false

setup.template.json.enabled: true
setup.template.json.path: /etc/filebeat/wazuh-template.json
setup.template.json.name: wazuh
setup.template.overwrite: true
setup.ilm.enabled: false

output.elasticsearch.protocol: http
#output.elasticsearch.ssl.certificate: /etc/filebeat/certs/filebeat.crt
#output.elasticsearch.ssl.key: /etc/filebeat/certs/filebeat.key
#output.elasticsearch.ssl.certificate_authorities: /etc/filebeat/certs/ca/ca.crt
#output.elasticsearch.username: elastic
```

6. Enable and start the Filebeat service:
```shell
# systemctl daemon-reload
# systemctl enable filebeat
# systemctl start filebeat
```
7. To ensure that Filebeat has been successfully installed, run the following command:
```shell
# filebeat test output
```
An example response should look as follows:

```consol
elasticsearch: http://10.0.2.11:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 10.0.2.11
    dial up... OK
  version: 7.11.2
```

### Disabling Filebeat repositories
It is recommended to disabling the Filebeat repository to prevent accidental upgrades. To do so, use the following command:
```shell
# sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-7.x.list
# apt update
```

**NEXT:** [Kibana Installation](./kibana-setup.md)
