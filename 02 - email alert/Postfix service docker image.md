### Postfix service docker image

Sesuaikan email google agar bisa diakses melalui aplikasi. Yaitu dengan menambahkan App Key.
Ubah `docker-compose.yml` dan masukkan image `juanluisbaptiste/postfix` sebagai smtp-relay service

```
services:
  smtp-relay:
    image: juanluisbaptiste/postfix
    environment:
      SMTP_SERVER: smtp.gmail.com
      SMTP_PORT: 587
      SMTP_USERNAME: <your username>@gmail.com
      SMTP_PASSWORD: <your app key>
      SERVER_HOSTNAME: wazuh-alert
    ports:
      - 25:25/tcp
  wazuh.manager:
    ...
```

Deploy wazuh-docker:

```
docker compose up
```

Cari tau nama container dengan perintah

```
docker ps
```

output

```
CONTAINER ID   IMAGE                         COMMAND                  CREATED       STATUS       PORTS                                                                                                                                                           NAMES
47d2695446f1   wazuh/wazuh-dashboard:4.5.3   "/entrypoint.sh"         2 hours ago   Up 2 hours   443/tcp, 0.0.0.0:443->5601/tcp, :::443->5601/tcp                                                                                                                single-node-wazuh.dashboard-1
3fb95285423d   wazuh/wazuh-manager:4.5.3     "/init"                  2 hours ago   Up 2 hours   0.0.0.0:1514-1515->1514-1515/tcp, :::1514-1515-      >1514-1515/tcp, 0.0.0.0:514->514/udp, :::514->514/udp, 0.0.0.0:55000->55000/tcp, :::55000->55000/tcp, 1516/tcp   single-node-wazuh.manager-1
6b847c853684   wazuh/wazuh-indexer:4.5.3     "/entrypoint.sh open…"   2 hours ago   Up 2 hours   0.0.0.0:9200->9200/tcp, :::9200->9200/tcp                                                                                                                       single-node-wazuh.indexer-1
58ec295c9150   juanluisbaptiste/postfix      "/run.sh"                2 hours ago   Up 2 hours   0.0.0.0:25->25/tcp, :::25->25/tcp                                                                                                                               single-node-smtp-relay-1
```

dari output, bisa diketahui, nama containernya adalah `single-node-smtp-relay-1`
Cari tau IP Address container postfix dengan command

```
docker inspect single-node-smtp-relay-1 | grep IPAddress
```

output
```
"SecondaryIPAddresses": null,
         "IPAddress": "",
                 "IPAddress": "192.168.16.4",
```

IP Address ini akan dimasukkan ke dalam ossec.conf Wazuh agar bisa digunakan sebagai smtp server. Perubahan dilakukan melalui Menu Edit
Configuration pada Wazuh Manager GUI.

```
<global>
  ....
  <email_notification>yes</email_notification>
  <smtp_server>(postfix_container_ip)</smtp_server>
  <email_from>(alert_email)@gmail</email_from>
  <email_to>recipient@domain.com</email_to>
  <email_maxperhour>12</email_maxperhour>
  <email_log_source>alerts.log</email_log_source>
  ...
</global>

<alerts>
  ...
  <email_alert_level>5</email_alert_level>
</alerts>
```

Klik Save lalu Restart Manager. Test notifikasi email dengan melakukan serangan ke agent.

```
curl -XGET "<http://172.16.1.103/users/?id=SELECT+*+FROM+users>";
```