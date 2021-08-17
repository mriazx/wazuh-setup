## Kibana Installation
This process will go through the installation of the **Kibana** in a 2 GB RAM **Kali Linux** node.

**Note:** It is recommended to set Static IP address for this node rather than dynamic one.

**Note:** Root user privileges are required to execute all the following commands.

### Adding the Elastic Stack repository
1. Install the necessary packages for the installation:
```shell
# apt-get install lsb-release curl apt-transport-https zip unzip gnupg2
```
2. Install the GPG key:
```shell
# curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | apt-key add -
```
3. Add the repository:
```shell
# echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```
4. Update the package information:
```shell
# apt update
```
### Kibana installation and configuration
1. Install the Kibana package:
```shell
# apt install Kibana=7.11.2
```
2. Download the Kibana configuration file: `kibana.yml`:
```shell
# curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/resources/4.1/elastic-stack/kibana/7.x/kibana.yml
```
3. Edit the file `/etc/kibana/kibana.yml`:
```shell
# nano /etc/kibana/kibana.yml
```
It will look like this:
```shell
server.host: <kibana_ip>
server.port: 443
elasticsearch.hosts: https://<elasticsearch_DN>:9200
elasticsearch.password: <elasticsearch_password>

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
Replace `kibana_ip` with Kibana’s host IP and `elasticsearch_DN` with Elasticsearch's host IP. Update `server.port` with `8080`and `elasticsearch.hosts` with `http`. Comment out all other sections. Updated file should like this:
```shell
server.host: 10.0.2.10
server.port: 8080
elasticsearch.hosts: http://10.0.2.11:9200
#elasticsearch.password: <elasticsearch_password>

# Elasticsearch from/to Kibana

#elasticsearch.ssl.certificateAuthorities: /etc/kibana/certs/ca/ca.crt
#elasticsearch.ssl.certificate: /etc/kibana/certs/kibana.crt
#elasticsearch.ssl.key: /etc/kibana/certs/kibana.key

# Browser from/to Kibana
#server.ssl.enabled: true
#server.ssl.certificate: /etc/kibana/certs/kibana.crt
#server.ssl.key: /etc/kibana/certs/kibana.key

# Elasticsearch authentication
#xpack.security.enabled: true
#elasticsearch.username: elastic
#uiSettings.overrides.defaultRoute: "/app/wazuh"
#elasticsearch.ssl.verificationMode: certificate
```
4. Create the /usr/share/kibana/data directory:
```shell
# mkdir /usr/share/kibana/data
# chown -R kibana:kibana /usr/share/kibana
```
5. Install the Wazuh Kibana plugin:
The installation of the plugin must be done from the Kibana home directory.
```shell
# cd /usr/share/kibana
# sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.1.5_7.11.2-1.zip
```
6. Enable Kibana access to Wazuh API, edit the file `/usr/share/kibana/data/wazuh/config/wazuh.yml` and replace the url with the Wazuh server’s IP address:
```shell
# Scroll down to the bottom of the file to get the configuration.
hosts:
  - default:
     url: https://10.0.2.15
     port: 55000
     username: wazuh-wui
     password: wazuh-wui
     run_as: false
```
7. Enable and start the Kibana service:
```shell
# systemctl daemon-reload
# systemctl enable kibana
# systemctl start kibana
```
8. Run the following command to check if the Kibana is active:
```shell
# systemctl status kibana
```
Access the web interface by url http://10.0.2.0:8080
### Disabling Kibana repositories
It is recommended to disabling the Kibana repository to prevent accidental upgrades. To do so, use the following command:
```shell
# sed -i "s/^deb/#deb/" /etc/apt/sources.list.d/elastic-7.x.list
# apt update
```
