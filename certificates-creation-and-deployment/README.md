# Certificates and Authentication Deployment
Now we will deploy certificate in Elasticsearch, Filebeat and Kibana instances. Then we will setup authentication for secure access.

The native elasticsearch-certutil tool has been used to create certificates, but any other certificates creation method, for example using [OpenSSL](https://www.openssl.org/), can be used. First, we will creat certificates in Elasticsearch instance and then copy Filebeat and Kibana certificates to those instances by using `scp` utility.

## Certificates Creation
This process is for Wazuh single node cluster. For Certificate deployment the instances file `/usr/share/elasticsearch/instances.yml` must be created.

**Note:** Root user privileges are required to execute all the following commands.

1. The instances file can be created `/usr/share/elasticsearch/instances.yml` as follows:
```bash
# nano /usr/share/elasticsearch/instances.yml
```
The file should like:
```bash
instances:
- name: "elasticsearch"
  ip:
  - "10.0.2.11"
- name: "filebeat"
  ip:
  - "10.0.2.15"
- name: "kibana"
  ip:
  - "10.0.2.10"
```
Replace the IPs with the corresponding IP addresses for each instance in your environment.

2. Create the certificates using the elasticsearch-certutil tool:
```shell
# /usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip
```
The resulting file `certs.zip` contains a directory for each instance included in `instances.yml`. Each directory contains a certificate and a private key necessary to secure communications.

3. Unzip the `~/certs.zip` file:
```shell
# unzip ~/certs.zip -d ~/certs
```

### Copying Filebeat and Kibana Certificates to their instances
1. Copy `~/certs/ca` and `~/certs/filebeat` to **Filebeat** deployment:
```shell
# scp -r ~/certs/ca ~/certs/filebeat ubuntu@10.0.2.15:~/
```
Change the credentials for your **Filebeat** host as `<user>@<host-IP>`.

2. Copy `~/certs/ca` and `~/certs/kibana` to **Kibana** deployment:
```shell
# scp -r ~/certs/ca ~certs/kibana kali@10.0.2.11:~/
```
Change the credentials for your **Kibana** host as `<user>@<host-IP>`.

## Elasticsearch Certificates and Authentication Deployment
1. Create the directory `/etc/elasticsearch/certs`, and then copy the certificate authorities, the certificate and key there:
```shell
# mkdir /etc/elasticsearch/certs/ca -p
# cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
# chown -R elasticsearch: /etc/elasticsearch/certs
# chmod -R 500 /etc/elasticsearch/certs
# chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
```
2. Edit the file `/etc/elasticsearch/elasticsearch.yml`:
```shell
# nano /etc/elasticsearch/elasticsearch.yml
```
Uncomment all of `Transport layer`, `HTTP layer` and `Elasticsearch authentication` sections. Updated file should like this:

```bash
network.host: 10.0.2.11
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
3. Restart the Elasticsearch service:
```shell
# systemctl restart elasticsearch
```
4. Run the following command to check if the Elasticsearch is active:
```shell
# systemctl status elasticsearch
```
5. Generate credentials for all the Elastic Stack pre-built roles and users:
```shell
# /usr/share/elasticsearch/bin/elasticsearch-setup-passwords interactive
```
The command above will prompt an output like this. Provide a password for each user and save the password of the `elastic` user for further steps. I use `pA$$w0rd` as a password which I'll use for further steps.

```shell
Changed password for user [apm_system]
Changed password for user [kibana_system]
Changed password for user [kibana]
Changed password for user [logstash_system]
Changed password for user [beats_system]
Changed password for user [remote_monitoring_user]
Changed password for user [elastic]
```

## Filebeat Certificates and Authentication Deployment

**Caution:** Do not enter following commands in root privilege. Use User previlege(`$`).

In [Copying Filebeat and Kibana Certificates to their instances](#copying-filebeat-and-kibana-certificates-to-their-instances) section we copy `~/certs/ca` and `~certs/filebeat` to **Filebeat** deployment. The files must be copied into the **Wazuh Manager's** user home directory(`~/`).

1. Create the directory `/etc/filebeat/certs`, and then copy the certificate authorities, the certificate and key there:
```shell
$ sudo mkdir /etc/filebeat/certs/ca -p
$ sudo cp -R ~/certs/ca/ ~/certs/filebeat/* /etc/filebeat/certs/
$ sudo chmod -R 500 /etc/filebeat/certs
$ sudo chmod 400 /etc/filebeat/certs/ca/ca.* /etc/filebeat/certs/filebeat.*
```
2. Edit the file `/etc/filebeat/filebeat.yml`:
```shell
$ sudo nano /etc/filebeat/filebeat.yml
```
Replace `elasticsearch_password` with the password we generated above and update protocol to 'https' from 'http'. Then uncomment authentication and certification. Updated file should like this:

```shell
# Wazuh - Filebeat configuration file
output.elasticsearch.hosts: 10.0.2.11:9200
output.elasticsearch.password: pA$$w0rd

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
output.elasticsearch.ssl.certificate: /etc/filebeat/certs/filebeat.crt
output.elasticsearch.ssl.key: /etc/filebeat/certs/filebeat.key
output.elasticsearch.ssl.certificate_authorities: /etc/filebeat/certs/ca/ca.crt
output.elasticsearch.username: elastic
```
3. Restart the Filebeat service:
```shell
$ sudo systemctl restart filebeat
```
4. To ensure that Filebeat has been successfully installed, run the following command:
```shell
$ sudo filebeat test output
```
The output should like:

```
elasticsearch: https://10.0.2.11:9200...
  parse url... OK
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 10.0.2.11
    dial up... OK
  TLS...
    security: server's certificate chain verification is enabled
    handshake... OK
    TLS version: TLSv1.3
    dial up... OK
  talk to server... OK
  version: 7.11.2
```

## Kibana Certificates and Authentication Deployment

**Caution:** Do not enter following commands in root privilege. Use User previlege(`$`).

In [Copying Filebeat and Kibana Certificates to their instances](#copying-filebeat-and-kibana-certificates-to-their-instances) section we copy `~/certs/ca` and `~certs/kibana` to **Kibana** deployment. The files must be copied into the **Kibana's** user home directory(`~/`).
  
1. Create the directory `/etc/kibana/certs`, and then copy the certificate authorities, the certificate and key there:
```shell
$ sudo mkdir /etc/kibana/certs/ca -p
$ sudo cp -R ~/certs/ca/ ~/certs/kibana/* /etc/kibana/certs/
$ sudo chmod -R 500 /etc/kibana/certs
$ sudo chmod 400 /etc/kiana/certs/ca/ca.* /etc/kibana/certs/kibana.*
```
2. Edit the file `/etc/kibana/kibana.yml`:
```shell
$ sudo nano /etc/kibana/kibana.yml
```
Update `server.port` with `443`and `elasticsearch.hosts` with `https`. Replace `elasticsearch_password` with the password we generated above. Uncomment all other sections. Updated file should like this:

```bash
server.host: 10.0.2.10
server.port: 443
elasticsearch.hosts: http://10.0.2.11:9200
elasticsearch.password: pA$$w0rd

# Elasticsearch from/to Kibana

elasticsearch.ssl.certificateAuthorities: /etc/kibana/certs/ca/ca.crt
elasticsearch.ssl.certificate: /etc/kibana/certs/kibana.crt
elasticsearch.ssl.key: /etc/kibana/certs/kibana.key

# Browser from/to Kibana
server.ssl.enabled: true
server.ssl.certificate: /etc/kibana/certs/kibana.crt
server.ssl.key: /etc/kibana/certs/kibana.key

# Elasticsearch authentication
xpack.security.enabled: true
elasticsearch.username: elastic
uiSettings.overrides.defaultRoute: "/app/wazuh"
elasticsearch.ssl.verificationMode: certificate
```
3. Restart the Kibana service:
```bash
$ sudo systemctl restart Kibana
```
4. To ensure that Kibana has been successfully installed, run the following command:
```bash
$ sudo systemctl status Kibana
```
4. Access the web interface using the password generated during the Elasticsearch installation process:
```bash
URL: https://10.0.2.10
user: elastic
password: pA$$w0rd
```
Replace the `URL` with your Kibana mechine IP and `password` with your password generated in Elasticsearch Certificate generation process.


