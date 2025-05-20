# Wazuh - Elastic Stack

- Accédez [ici](https://documentation.wazuh.com/4.5/deployment-options/elastic-stack/all-in-one-deployment/index.html)

#### Installation des dépendances de base

```sh
apt-get install apt-transport-https zip unzip lsb-release curl gnupg
```

#### Ajout du dépôt officiel Elasticsearch

```sh
curl -s https://artifacts.elastic.co/GPG-KEY-elasticsearch | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/elasticsearch.gpg --import && chmod 644 /usr/share/keyrings/elasticsearch.gpg
```

- Télécharge la clé GPG d’Elasticsearch et la sauvegarde dans /usr/share/keyrings pour signature des paquets.

```sh
echo "deb [signed-by=/usr/share/keyrings/elasticsearch.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | tee /etc/apt/sources.list.d/elastic-7.x.list
```

#### Installation d’Elasticsearch

```sh
apt-get update
apt-get install elasticsearch=7.17.13
```

#### Configuration d’Elasticsearch

```sh
curl -so /etc/elasticsearch/elasticsearch.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/elasticsearch_all_in_one.yml

curl -so /usr/share/elasticsearch/instances.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/instances_aio.yml
```

#### Génération des certificats SSL pour Elasticsearch

```sh
/usr/share/elasticsearch/bin/elasticsearch-certutil cert ca --pem --in instances.yml --keep-ca-key --out ~/certs.zip

unzip ~/certs.zip -d ~/certs
```

#### Déploiement des certificats

```sh
mkdir /etc/elasticsearch/certs/ca -p
cp -R ~/certs/ca/ ~/certs/elasticsearch/_ /etc/elasticsearch/certs/
chown -R elasticsearch: /etc/elasticsearch/certs
chmod -R 500 /etc/elasticsearch/certs
chmod 400 /etc/elasticsearch/certs/ca/ca._ /etc/elasticsearch/certs/elasticsearch.\*
rm -rf ~/certs/ ~/certs.zip
```

#### Activation d’Elasticsearch

```sh
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
```

#### Génération des mots de passe

```sh
/usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

#### Test de connexion à Elasticsearch

```sh
curl -XGET https://localhost:9200 -u elastic:<elastic_password> -k
```

#### Ajout du dépôt Wazuh

```sh
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg

echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list
```

#### Installation du Wazuh Manager

```sh
apt-get update
apt-get install wazuh-manager=4.5.4-1
systemctl daemon-reload
systemctl enable wazuh-manager
systemctl start wazuh-manager
systemctl status wazuh-manager
```

#### Installation de Filebeat (agent log → Elasticsearch)

```sh
apt-get install filebeat=7.17.13
```

#### Configuration de Filebeat

```sh
curl -so /etc/filebeat/filebeat.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/filebeat_all_in_one.yml

curl -so /etc/filebeat/wazuh-template.json https://raw.githubusercontent.com/wazuh/wazuh/v4.5.4/extensions/elasticsearch/7.x/wazuh-template.json
chmod go+r /etc/filebeat/wazuh-template.json
```

- Configure Filebeat pour parser les logs de Wazuh et les envoyer vers Elasticsearch.

```sh
curl -s https://packages.wazuh.com/4.x/filebeat/wazuh-filebeat-0.2.tar.gz | tar -xvz -C /usr/share/filebeat/module
```

#### Saisie du mot de passe Elasticsearch dans Filebeat

```sh
nano /etc/filebeat/filebeat.yml

# Modifier:
output.elasticsearch.password: <elasticsearch_password>
```

#### Copie des certificats dans Filebeat

```sh
cp -r /etc/elasticsearch/certs/ca/ /etc/filebeat/certs/
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/filebeat/certs/filebeat.crt
cp /etc/elasticsearch/certs/elasticsearch.key /etc/filebeat/certs/filebeat.key
```

#### Activation de Filebeat

- Active Filebeat et teste la connexion à Elasticsearch.

```sh
systemctl daemon-reload
systemctl enable filebeat
systemctl start filebeat

filebeat test output
```

#### Installation de Kibana (interface graphique)

```sh
apt-get install kibana=7.17.13
```

#### Ajout des certificats à Kibana

```sh
mkdir /etc/kibana/certs/ca -p
cp -R /etc/elasticsearch/certs/ca/ /etc/kibana/certs/
cp /etc/elasticsearch/certs/elasticsearch.key /etc/kibana/certs/kibana.key
cp /etc/elasticsearch/certs/elasticsearch.crt /etc/kibana/certs/kibana.crt
chown -R kibana:kibana /etc/kibana/
chmod -R 500 /etc/kibana/certs
chmod 440 /etc/kibana/certs/ca/ca.* /etc/kibana/certs/kibana.*
```

#### Configuration de Kibana

```sh
curl -so /etc/kibana/kibana.yml https://packages.wazuh.com/4.5/tpl/elastic-basic/kibana_all_in_one.yml

nano /etc/kibana/kibana.yml

# Modifier:
elasticsearch.password: <elasticsearch_password>
```

#### Préparation du dossier de données Kibana

```sh
mkdir /usr/share/kibana/data
chown -R kibana:kibana /usr/share/kibana
```

#### Installation du plugin Wazuh pour Kibana

```sh
cd /usr/share/kibana
sudo -u kibana /usr/share/kibana/bin/kibana-plugin install https://packages.wazuh.com/4.x/ui/kibana/wazuh_kibana-4.5.4_7.17.13-1.zip
```

#### Permettre à Kibana d’écouter sur le port 443

```sh
setcap 'cap_net_bind_service=+ep' /usr/share/kibana/node/bin/node
```

#### Démarrage de Kibana

```sh
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
```

#### Accès à l'interface

```sh
URL: https://<wazuh_server_ip>
user: elastic
password: <PASSWORD_elastic>
```
