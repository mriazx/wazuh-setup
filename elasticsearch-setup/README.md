## Elasticsearch Installation
This process will go through the installation of the **Elasticsearch** in a 1 GB RAM **Ubuntu Server 20.04** node.

**Note:** It is recommended to set Static IP address for this node rather than dynamic one.

**Note:** Root user privileges are required to execute all the following commands.

### Adding the Elastic Stack repository

1. Install the necessary packages for the installation:
```shell
# apt-get install lsb-release curl apt-transport-https zip unzip gnupg2
```
2. Install the GPG key:
```shell
# # curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
```
3. Add the repository:
```shell
# echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```
4. Update the package information:
```shell
# apt update
```

### Elasticsearch installation and configuration

1. Install the Elasticsearch package:
```shell
# apt install elasticsearch=7.11.2
```
2. Download Elasticsearch configuration file `elasticsearch.yml`:
```shell
# curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/resources/4.1/elastic-stack/elasticsearch/7.x/elasticsearch.yml
```
3. Edit the file `/etc/elasticsearch/elasticsearch.yml`:
```shell
# nano /etc/elasticsearch/elasticsearch.yml
```

It will look like this:

```yml
network.host: 0.0.0.0
node.name: node-1
cluster.initial_master_nodes: node-1

# Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
xpack.security.transport.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
xpack.security.transport.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

# HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
xpack.security.http.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
xpack.security.http.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

# Elasticsearch authentication
xpack.security.enabled: true

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

Replace the `network.host` IP with your mechine IP and comment out all of `Transport layer`, `HTTP layer` and `Elasticsearch authentication` sections. Updated file should like this:

```console
network.host: 10.0.2.11
node.name: node-1
cluster.initial_master_nodes: node-1

# Transport layer
#xpack.security.transport.ssl.enabled: true
#xpack.security.transport.ssl.verification_mode: certificate
#xpack.security.transport.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
#xpack.security.transport.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
#xpack.security.transport.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

# HTTP layer
#xpack.security.http.ssl.enabled: true
#xpack.security.http.ssl.verification_mode: certificate
#xpack.security.http.ssl.key: /etc/elasticsearch/certs/elasticsearch.key
#xpack.security.http.ssl.certificate: /etc/elasticsearch/certs/elasticsearch.crt
#xpack.security.http.ssl.certificate_authorities: /etc/elasticsearch/certs/ca/ca.crt

# Elasticsearch authentication
#xpack.security.enabled: true

path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
```

4. Enable and start the Elasticsearch service:
```shell
# systemctl daemon-reload
# systemctl enable elasticsearch
# systemctl start elasticsearch
```
3. Run the following command to check if the Elasticsearch is active:
```shell
# systemctl status elasticsearch
```

### Disabling Elasticsearch repositories

It is recommended to disabling the Elasticsearch repository to prevent accidental upgrades. To do so, use the following command:

```shell
# sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-7.x.list
# apt update
```

### Miscellaneous

1. Allowing ports in firewall:
```shell
# ufw allow 9200
```
2. To check if the ports are listening:
```shell
# ss -lntp
```

**NEXT:** [Filebeat Installation](../filebeat-setup)
