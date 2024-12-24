## Wazuh NIDS using Suricata

Pada endpoint, install suricata, jalankan command ini line per line

```bash
add-apt-repository ppa:oisf/suricata-stable
apt-get update
apt-get install suricata -y
```

Download dan extract rules untuk suricata

```bash
cd /tmp/ && curl -LO <https://rules.emergingthreats.net/open/suricata-7.0.2/emerging.rules.tar.gz>
sudo tar -xvzf emerging.rules.tar.gz && sudo mv rules/*.rules /etc/suricata/rules/
sudo chmod 640 /etc/suricata/rules/*.rules
```

Ubah konfigurasi suricata `nano /etc/suricata/suricata.yaml`

```
HOME_NET: "<ENDPOINT_IP>"
EXTERNAL_NET: "any"

default-rule-path: /etc/suricata/rules
rule-files:
- "*.rules"

# Global stats configuration
stats:
enabled: yes

# Linux high speed capture support
af-packet:
  - interface: enp0s3
```

Restart suricata

```
systemctl restart suricata
```

Edit konfigurasi `nano /var/ossec/etc/ossec.conf` untuk memonitor log suricata

```xml
<ossec_config>
  <localfile>
    <log_format>json</log_format>
    <location>/var/log/suricata/eve.json</location>
  </localfile>
</ossec_config>
```

Pada Wazuh server, install Wazuh advanced rules dari SOCFortress

```bash
curl -so ~/wazuh_socfortress_rules.sh https://raw.githubusercontent.com/socfortress/Wazuh-Rules/main/wazuh_socfortress_rules.sh && bash ~/wazuh_socfortress_rules.sh
```

Restart wazuh-manager

```bash
systemctl restart wazuh-manager
```

Jalankan ping berulang ke endpoint

```
ping -c 40 <ENDPOINT_IP>
```

Perhatikan security events.