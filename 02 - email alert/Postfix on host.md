Postfix on host
Sesuaikan email google agar bisa diakses melalui aplikasi. Yaitu dengan menambahkan App Key.
Ini cara menggunakan notifikasi email jika Wazuh di-install langsung pada host baik itu menggunakan cara quickstart, install script, maupun manual install. Menggunakan postfix sebagai mail transfer agent dan gmail sebagai smtp relay.
Install mailutils
apt install mailutils -y
​
Jika ada prompt, pilih Internet Site, lalu masukkan nama domain, pada langkah ini gunakan local domain saja, seperti wazuh-alert.
Jika perlu memperbaharui konfigurasi bisa jalankan
dpkg-reconfigure postfix
​
Buat file konfigurasi sasl authentication untuk otentikasi akun ke mail server
nano /etc/postfix/sasl_passwd
​
Isikan seperti berikut jika menggunakan gmail.
[smtp.gmail.com]:587 <username>@gmail.com:<app_key>
​
Buat hash database agar akun bisa diakses oleh postfix
sudo postmap /etc/postfix/sasl_passwd

Amankan file password tersebut

```
chown root:root /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
chmod 0600 /etc/postfix/sasl_passwd /etc/postfix/sasl_passwd.db
```

Ubah konfigurasi `/etc/postfix/main.cf`

```
...
smtp_tls_security_level=encrypt
...
relayhost = [smtp.gmail.com]:587
mynetworks = 127.0.0.0/8
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_mechanism_filter = login
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_use_tls = yes
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_tls_wrappermode = yes
```

Restart postfix

```
systemctl restart postfix
```

Test postfix dengan mengirimkan email via command ini:
```
echo "This is the email body." | mail -s "Email subject" -a "From: <sender>@gmail.com" recipient@domain.com
```

Selanjutnya ubah konfigurasi `/var/ossec/etc/ossec.conf`

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

Restart wazuh manager

```
systemctl restart wazuh-manager
```

Test alert dengan lakukan serangan terhadap server.

```
curl -XGET "<http://172.16.1.103/users/?id=SELECT+*+FROM+users>";
```