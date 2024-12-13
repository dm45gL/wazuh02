Tambahkan pada local_decoder.xml pada Wazuh server
<!-- CPU, memory, disk metric -->
<decoder name="general_health_check">
     <program_name>general_health_check</program_name>
</decoder>

<decoder name="general_health_check1">
  <parent>general_health_check</parent>
  <prematch>ossec: output: 'general_health_metrics':\\.</prematch>
  <regex offset="after_prematch">(\\S+) (\\S+) (\\S+)</regex>
  <order>cpu_usage_%, memory_usage_%, disk_usage_%</order>
</decoder>

<!-- load average metric -->
<decoder name="load_average_check">
     <program_name>load_average_check</program_name>
</decoder>

<decoder name="load_average_check1">
  <parent>load_average_check</parent>
  <prematch>ossec: output: 'load_average_metrics':\\.</prematch>
  <regex offset="after_prematch">(\\S+), (\\S+), (\\S+)</regex>
  <order>1min_loadAverage, 5mins_loadAverage, 15mins_loadAverage</order>
</decoder>

<!-- Memory metric -->
<decoder name="memory_check">
     <program_name>memory_check</program_name>
</decoder>

<decoder name="memory_check1">
  <parent>memory_check</parent>
  <prematch>ossec: output: 'memory_metrics':\\.</prematch>
  <regex offset="after_prematch">(\\S+) (\\S+)</regex>
  <order>memory_used_bytes, memory_available_bytes</order>
</decoder>

<!-- Disk metric -->
<decoder name="disk_check">
     <program_name>disk_check</program_name>
</decoder>

<decoder name="disk_check1">
  <parent>disk_check</parent>
  <prematch>ossec: output: 'disk_metrics':\\.</prematch>
  <regex offset="after_prematch">(\\S+) (\\S+)</regex>
  <order>disk_used_bytes, disk_free_bytes</order>
</decoder>

Tambahkan rule pada `local_rules.xml` milik server

```xml
<group name="performance_metric,">

<!-- CPU, Memory and Disk usage -->
<rule id="100054" level="3">
  <decoded_as>general_health_check</decoded_as>
  <description>CPU | MEMORY | DISK usage metrics</description>
</rule>

<!-- High memory usage -->
<rule id="100055" level="12">
  <if_sid>100054</if_sid>
  <field name="memory_usage_%">^[8-9]\\d|100</field>
  <description>Memory usage is high: $(memory_usage_%)%</description>
  <options>no_full_log</options>
</rule>

<!-- High CPU usage -->
<rule id="100056" level="12">
  <if_sid>100054</if_sid>
  <field name="cpu_usage_%">^[8-9]\\d|100</field>
  <description>CPU usage is high: $(cpu_usage_%)%</description>
  <options>no_full_log</options>
</rule>

<!-- High disk usage -->
<rule id="100057" level="12">
  <if_sid>100054</if_sid>
  <field name="disk_usage_%">^[7-9]\\d|100</field>
  <description>Disk space is running low: $(disk_usage_%)%</description>
  <options>no_full_log</options>
</rule>

<!-- Load average check -->
<rule id="100058" level="3">
  <decoded_as>load_average_check</decoded_as>
  <description>load average metrics</description>
</rule>

<!-- memory check -->
<rule id="100059" level="3">
  <decoded_as>memory_check</decoded_as>
  <description>Memory metrics</description>
</rule>

<!-- Disk check -->
<rule id="100060" level="3">
  <decoded_as>disk_check</decoded_as>
  <description>Disk metrics</description>
</rule>

</group>
```

Restart wazuh-manager
systemctl restart wazuh-manager
​
Cek event yang muncul pada Wazuh dashboard.