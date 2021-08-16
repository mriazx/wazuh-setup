### Configure Elasticsearch certificate
1. The instances file can be created `/usr/share/elasticsearch/instances.yml` as follows:
```shell
# nano /usr/share/elasticsearch/instances.yml
```
  The file should like:
```yml
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
3. Unzip the `~/certs.zip` file:
```shell
# unzip ~/certs.zip -d ~/certs
```
4. Create the directory `/etc/elasticsearch/certs`, and then copy the certificate authorities, the certificate and key there:
```shell
# mkdir /etc/elasticsearch/certs/ca -p
# cp -R ~/certs/ca/ ~/certs/elasticsearch/* /etc/elasticsearch/certs/
# chown -R elasticsearch: /etc/elasticsearch/certs
# chmod -R 500 /etc/elasticsearch/certs
# chmod 400 /etc/elasticsearch/certs/ca/ca.* /etc/elasticsearch/certs/elasticsearch.*
```
5. Edit the file `/etc/elasticsearch/elasticsearch.yml`:
```shell
# nano /etc/elasticsearch/elasticsearch.yml
```
  Uncomment all of `Transport layer`, `HTTP layer` and `Elasticsearch authentication` sections. Updated file should like this:
```yml
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
6. Restart the Elasticsearch service:
```shell
# systemctl restart elasticsearch
```
7. Run the following command to check if the Elasticsearch is active:
```shell
# systemctl status elasticsearch
```
8. Generate credentials for all the Elastic Stack pre-built roles and users:
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
9. Copy `~/certs/ca` and `~certs/filebeat` to **Filebeat** deployment:
```shell
# scp -r ~/certs/ca ~/certs/filebeat ubuntu@10.0.2.15:~/
```
10. Copy `~/certs/ca` and `~/certs/kibana` to **Kibana** deployment:
```shell
# scp -r ~/certs/ca ~certs/kibana kali@10.0.2.11:~/
```

### Configure Filebeat certificate
  In previous section we copy `~/certs/ca` and `~certs/filebeat` to **Filebeat** deployment. The files must be copied into the **Wazuh Manager's** user home directory(`/home/<user>`).
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
```yml
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
# sudo filebeat test output
```
  The output should like:
```shell
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
