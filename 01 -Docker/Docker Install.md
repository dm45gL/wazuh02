Docker Install
Command untuk instalasi docker
Wazuh membutuhkan banyak memory map untuk berjalan, oleh karena itu, jumlah memory map pada host harus ditambahkan dengan cara
Ubah max_map_count pada host
sysctl -w vm.max_map_count=262144
​
Ubah secara permanen dengan mengubah file konfigurasi /etc/sysctl.conf dan tambahkan line
vm.max_map_count=262144
​
Untuk verifikasi jalankan perintah
sysctl vm.max_map_count
​
Instal docker sesuai sumbernya (berlaku untuk semua OS berbasis debian)
curl -sSL <https://get.docker.com/> | sh
​
Start docker
systemctl start docker

Download binary docker-compose

```
curl -L "<https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$>(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Ubah properties upaya bisa di-execute

```
chmod +x /usr/local/bin/docker-compose
```

Test docker compose menggunakan command

```
docker-compose --version
```

Atau bisa menggunakan packages dari repository Ubuntu

```
apt install docker.io
```

2. Command untuk instalasi Wazuh (single node)
Clone repository Wazuh on Docker
    
    ```
    git clone <https://github.com/wazuh/wazuh-docker.git> -b v4.5.3
    ```
    
    Masuk ke directory single node
```
cd wazuh-docker/single-node
```

Generate certificate untuk digunakan oleh wazuh

```
docker-compose -f generate-indexer-certs.yml run --rm generator
```

Deploy wazuh

```
docker compose up
```

atau

```
docker compose up -d
```