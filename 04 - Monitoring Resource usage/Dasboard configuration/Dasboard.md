
Dashboard configuration
Agar data dapat ditampilkan ke dalam visualisasi, data harus dalam tipa data double, bukan string, oleh karena itu, modifikasi tipe data pada Wazuh Indexer.
Tambahkan custom field pada Wazuh template, custom field ini akan mendefinisikan data dengan tipe data baru (double)
nano /etc/filebeat/wazuh-template.json
​
Tambahkan parameter dalam comment
```
{
 "order": 0,
 "index_patterns": [
    "wazuh-alerts-4.x-*",
    "wazuh-archives-4.x-*"
  ],
 "settings": {
   ...
  },
 "mappings": {
    "dynamic_templates": [
      {
   ...
 "data": {
   "properties": {
####start-line
         "1min_loadAverage": {
                 "type": "double"
               },
         "5mins_loadAverage": {
                 "type": "double"
               },
         "15mins_loadAverage": {
                 "type": "double"
               },
         "cpu_usage_%": {
                 "type": "double"
               },
         "memory_usage_%": {
                 "type": "double"
               },
         "memory_available_bytes": {
                 "type": "double"
               },
         "memory_used_bytes": {
                  "type":  "double"
               },
         "disk_used_bytes": {
                 "type": "double"
               },
         "disk_free_bytes": {
                 "type": "double"
               },
         "disk_usage_%": {
                 "type": "double"
               }
###End-line
     "audit": {
       "properties": {
                 "acct": {
                     "type": "keyword"

```

Apply wazuh templates

```
filebeat setup -index-management
```

Re-index data pada Wazuh.
Masuk ke menu Management > Dev Tools
Ketik perintah ini

```
GET _cat/indices
```

output yang diharapkan seperti ini

```
green open wazuh-monitoring-2023.44w   qI6SpkKJR5iKqfAYn2-jTA 1 0   251 0   366kb   366kb
green open .opensearch-observability   QRiT8cxWRWOOdONa4Qj9lQ 1 0     0 0    208b    208b
green open wazuh-statistics-2023.44w   BVeeZTZZSQCivSoFEh8aTA 1 0  1094 0 966.3kb 966.3kb
green open wazuh-alerts-4.x-2023.11.02 _7d0DfqMSl29zu3XcJkClg 3 0 25518 0  10.8mb  10.8mb
green open wazuh-alerts-4.x-2023.11.03 F2gWVP9IR8GoqxNtlLLUAg 3 0  7690 0   4.8mb   4.8mb
```

Index alerts `wazuh-alerts-4.x-2023.11.02` ini akan dimodifikasi agar menggunakan template baru.
Kembali ke Dev Tools lalu backup alerts terakhir lalu cek tipe data pada paremeter yang akan diubah

```
GET /wazuh-alerts-4.x-2023.04.24
```

Output yang terlihat

```
            "memory_available_bytes": {
              "type": "keyword"
            },
            "memory_usage_%": {
              "type": "keyword"
            },
            "memory_used_bytes": {
              "type": "keyword"
```

tipe data keyword akan diubah menjadi double agar bisa diterjemahkan ke dalam grafik.
Pindahkan data alerts ke dalam index baru, nama bebas namun saya menggunakan 'backup'

```
POST _reindex
{
  "source": {
    "index": "wazuh-alerts-4.x-2023.11.02"
  },
  "dest": {
    "index": "wazuh-alerts-4.x-backup"
  }
}
```

output yang keluar adalah statu 200 Ok dan

```
{
  "took": 1728,
  "timed_out": false,
  "total": 11599,
  "updated": 0,
  "created": 11599,
  "deleted": 0,
  "batches": 12,
  "version_conflicts": 0,
  "noops": 0,
  "retries": {
    "bulk": 0,
    "search": 0
  },
  "throttled_millis": 0,
  "requests_per_second": -1,
  "throttled_until_millis": 0,
  "failures": []
}
```

lalu hapus index lama

```
DELETE /wazuh-alerts-4.x-2023.11.02
```

output yang diharapkan adalah 200 OK dan

```
{
  "acknowledged": true
}
```

pindahkan backup tadi ke index yang seharusnya

```
{
  "source": {
    "index": "wazuh-alerts-4.x-backup"
  },
  "dest": {
    "index": "wazuh-alerts-4.x-2023.11.02"
  }
}
```
lalu hapus index backup tadi

```
DELETE /wazuh-alerts-4.x-backup
```

Cek kembali tipe data pada index

```
GET /wazuh-alerts-4.x-2023.11.02
```

Search memory dan pastikan berubah menjadi tipe data double

```
            },
            "memory_available_bytes": {
              "type": "double"
            },
            "memory_usage_%": {
              "type": "double"
            },
            "memory_used_bytes": {
              "type": "double"
            },
```

Selanjutnya buat dashboard Resource Monitoring.