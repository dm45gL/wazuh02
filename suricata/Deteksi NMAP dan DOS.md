## Deteksi NMAP dan DOS menggunakan Suricata

Kita akan memetakan `src_ip` dari log suricata ke dalam parameter `srcip` milik Wazuh.
Ubah file `local_decoder.xml` pada wazuh server agar hal tersebut bisa tercapai.

```
  <decoder name="json_child">
    <parent>json</parent>
    <regex type="pcre2">"src_ip":"([^"]+)"</regex>
    <order>srcip</order>
  </decoder>

  <decoder name="json_child">
    <parent>json</parent>
    <plugin_decoder>JSON_Decoder</plugin_decoder>
  </decoder>

```

Selanjutnya buat rules agar wazuh dapat mengenali log dari DOS attack dan NMAP Scanning.

```
<group name="ids,suricata">
  <rule id="900200" level="12">
    <if_sid>86600</if_sid>
    <field name="event_type">^alert$</field>
    <match>ET DOS</match>
    <description>DoS attack has been detected. </description>
    <mitre>
      <id>T1498</id>
    </mitre>
  </rule>

  <rule id="900201" level="12">
    <if_sid>86600</if_sid>
    <field name="event_type">^alert$</field>
    <match>ET SCAN</match>
    <description>Scanning engine detected. </description>
    <mitre>
      <id>T1595</id>
    </mitre>
  </rule>
</group>

```

Aktifkan firewall drop untuk rules yang baru dibuat scanning

```
<ossec_config>
  <active-response>
311
  </active-response>
</ossec_config>

```

Lakukan NMAP, dan perhatikan security alerts.

Pada kali, download Golden Eye.

```
git clone <https://github.com/jseidl/GoldenEye.git>

```

Jalankan Golden Eye

```
./goldeneye.py http://<TARGET_IP>

```

Perhatikan security events.
