Haproxy log

```
Oct 26 15:10:14 webdev haproxy[2247896]: 210.210.184.38:55963 [26/Oct/2023:15:10:14.580] localhost~ glog_sv/glogsv1 0/0/0/9/9 200 476 - - ---- 32/32/0/0/0 0/0 {glog.bayuskylabs.demo} "POST /api/cluster/metrics/multiple HTTP/1.1" glog.bayuskylabs.demo/api/cluster/metrics/multiple 26/Oct/2023:15:10:14.573 "glogsv1" POST /api/cluster/metrics/multiple HTTP/1.1
Oct 26 15:10:14 webdev haproxy[2247896]: 192.168.78.19:53796 [26/Oct/2023:15:10:14.917] localhost~ webgitdev/apisv1 0/0/2/2/4 204 200 - - ---- 32/32/9/9/0 0/0 {gitlab.bayuskylabs.demo} "POST /api/v4/jobs/request HTTP/1.1" gitlab.bayuskylabs.demo/api/v4/jobs/request 26/Oct/2023:15:10:11.923 "apisv1" POST /api/v4/jobs/request HTTP/1.1
Oct 26 15:10:14 webdev haproxy[2247896]: 192.168.182.7:34464 [26/Oct/2023:15:10:14.958] localhost~ webgitdev/apisv1 0/0/2/1/3 204 200 - - ---- 32/32/9/9/0 0/0 {gitlab.bayuskylabs.demo} "POST /api/v4/jobs/request HTTP/1.1" gitlab.bayuskylabs.demo/api/v4/jobs/request 26/Oct/2023:15:10:11.954 "apisv1" POST /api/v4/jobs/request HTTP/1.1
Oct 26 15:10:15 webdev haproxy[2247896]: 210.210.184.38:62336 [26/Oct/2023:15:10:15.055] localhost~ webgitdev/apisv1 0/0/2/13/15 304 388 - - ---- 35/35/11/11/0 0/0 {gitlab.bayuskylabs.demo} "GET /bayuskylabs/users/?id=SELECT+*+FROM+users HTTP/1.1" gitlab.bayuskylabs.demo/bayuskylabs/users/?id=SELECT+*+FROM+users 26/Oct/2023:15:10:15.046 "apisv1" GET /bayuskylabs/users/?id=SELECT+*+FROM+users HTTP/1.1
Oct 26 15:10:15 webdev haproxy[2247896]: 210.210.184.38:62335 [26/Oct/2023:15:10:15.052] localhost~ webgitdev/apisv1 0/0/2/168/170 200 1611 - - ---- 35/35/10/10/0 0/0 {gitlab.bayuskylabs.demo} "POST /api/graphql HTTP/1.1" gitlab.bayuskylabs.demo/api/graphql 26/Oct/2023:15:10:15.044 "apisv1" POST /api/graphql HTTP/1.1
Oct 26 15:10:15 webdev haproxy[2247896]: 210.210.184.38:62337 [26/Oct/2023:15:10:15.053] localhost~ webgitdev/apisv1 0/0/1/213/214 304 628 - - ---- 35/35/9/9/0 0/0 {gitlab.bayuskylabs.demo} "GET /bayuskylabs/<script>alert("TEST");</script>" gitlab.bayuskylabs.demo/bayuskylabs/<script>alert("TEST");</script> 26/Oct/2023:15:10:15.045 "apisv1" GET /bayuskylabs/<script>alert("TEST");</script> HTTP/1.1
Oct 26 15:10:15 webdev haproxy[2247896]: 192.168.182.18:51748 [26/Oct/2023:15:10:15.370] localhost~ webgitdev/apisv1 0/0/2/26/28 403 397 - - ---- 36/36/9/9/0 0/0 {gitlab.bayuskylabs.demo} "POST /api/v4/jobs/request HTTP/1.1" gitlab.bayuskylabs.demo/api/v4/jobs/request 26/Oct/2023:15:10:15.366 "apisv1" POST /api/v4/jobs/request HTTP/1.1
```

local decoder

```xml
<decoder name="haproxy">
     <program_name>haproxy</program_name>
     <prematch>\d+.\d+.\d+.\d+:\d+ \S+ \S+</prematch>
</decoder>

<decoder name="haproxy1">
    <parent>haproxy</parent>
    <regex>(\d+.\d+.\d+.\d+):(\d+) [(\S+)] (\S+) (\S+)/(\S+) (\d+/\d+/\d+/\d+/\d+) (\S+) (\S+)</regex>
    <order>client_ip, client_port, accept_date, frontend_name, backend_name, server_name, timer, status_code, response_lenght</order>
</decoder>

<decoder name="haproxy1">
    <parent>haproxy</parent>
    <regex>- - (\S+) (\d+/\d+/\d+/\d+/\d+) (\d+)/(\d+)</regex>
    <order>termination_state, connections, server_queue, backend_queue</order>
</decoder>

<decoder name="haproxy1">
    <parent>haproxy</parent>
    <type>web-log</type>
    <regex>{(\.*)} "(\.*)</regex>
    <order>headers, url</order>
</decoder>

<decoder name="haproxy">
     <prematch>\d+.\d+.\d+.\d+:\d+ \S+ \S+</prematch>
</decoder>

<decoder name="haproxy2">
    <parent>haproxy</parent>
    <regex>(\d+.\d+.\d+.\d+):(\d+) [(\S+)] (\S+) (\S+)/(\S+) (\d+/\d+/\d+/\d+/\d+) (\S+) (\S+)</regex>
    <order>client_ip, client_port, accept_date, frontend_name, backend_name, server_name, timer, status_code, response_lenght</order>
</decoder>

<decoder name="haproxy2">
    <parent>haproxy</parent>
    <regex>- - (\S+) (\d+/\d+/\d+/\d+/\d+) (\d+)/(\d+)</regex>
    <order>termination_state, connections, server_queue, backend_queue</order>
</decoder>

<decoder name="haproxy2">
    <parent>haproxy</parent>
    <regex>{(\.*)} "(\.*)</regex>
    <order>headers, url</order>
</decoder>
```

local rules
<rule id="100002" level="3">
    <decoded_as>haproxy</decoded_as>
    <description>Haproxy logs</description>
  </rule>
  
  <rule id="100004" level="7">
    <!--<if_sid>31100,31108</if_sid>-->
    <decoded_as>haproxy</decoded_as>
    <url>=select%20|select+|insert%20|%20from%20|%20where%20|union%20|</url>
    <url>union+|where+|null,null|xp_cmdshell</url>
    <description>SQL injection attempt.</description>
    <mitre>
      <id>T1190</id>
    </mitre>
    <group>attack,sql_injection,pci_dss_6.5,pci_dss_11.4,pci_dss_6.5.1,gdpr_IV_35.7.d,nist_800_53_SA.11,nist_800_53_SI.4,tsc_CC6.6,tsc_CC7.1,tsc_CC8.1,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>
  
  <rule id="100005" level="6">
    <decoded_as>haproxy</decoded_as>
    <url>%3Cscript|%3C%2Fscript|script>|script%3E|SRC=javascript|IMG%20|</url>
    <url>%20ONLOAD=|INPUT%20|iframe%20</url>
    <description>XSS (Cross Site Scripting) attempt.</description>
    <mitre>
      <id>T1059.007</id>
    </mitre>
    <group>attack,pci_dss_6.5,pci_dss_11.4,pci_dss_6.5.7,gdpr_IV_35.7.d,nist_800_53_SA.11,nist_800_53_SI.4,tsc_CC6.6,tsc_CC7.1,tsc_CC8.1,tsc_CC6.1,tsc_CC6.8,tsc_CC7.2,tsc_CC7.3,</group>
  </rule>