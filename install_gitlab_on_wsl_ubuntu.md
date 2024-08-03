# 1. install shadowsocks on ubuntu
In china, some website are blocked by the great fire wall. So you need a proxy.
```shell
sudo apt update
sudo apt install shadowsocks-libev
```
github: https://github.com/shadowsocks/shadowsocks-libev
you can compile and install on your own hand.

# 2. config shadowsocks client as system service
create config file for ss-local(shadowsocks client):
```shell
vim /etc/shadowsocks.json
```
write the following configs:
```python
{
    "server": "c61s2.portablesubmarines.com",
    "server_port": 7326,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "wcJokwbrY66YTpZa",
    "timeout": 300,
    "method": "aes-256-gcm"
}
```
make sure your have a shadowsocks server.
now, create a Systemd service for ss-local
```shell
sudo nano /etc/systemd/system/sslocal.service
```
write the following configs:
```shell
[Unit]
Description=Shadowsocks Client

[Service]
ExecStart=/usr/bin/ss-local -c /etc/shadowsocks.json
Restart=always

[Install]
WantedBy=multi-user.target
```
Enable the new service:
```shell
sudo systemctl daemon-reload
sudo systemctl enable sslocal
sudo systemctl start sslocal
```
Check if the service is started:
```shell
sudo systemctl status sslocal
```
if success, you can see:
```shell
● shadowsocks.service - Shadowsocks Client
     Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled; vendor preset: enabled)
     Active: active (running) since Sat 2024-08-03 13:44:18 CST; 3h 27min ago
   Main PID: 1134 (ss-local)
      Tasks: 1 (limit: 9359)
     Memory: 1.4M
     CGroup: /system.slice/shadowsocks.service
             └─1134 /usr/bin/ss-local -c /etc/shadowsocks.json

Aug 03 13:44:18 LAPTOP-4B9EAO75 systemd[1]: Started Shadowsocks Client.
Aug 03 13:44:18 LAPTOP-4B9EAO75 ss-local[1134]:  2024-08-03 13:44:18 INFO: initializing ciphers... aes-256-gcm
Aug 03 13:44:20 LAPTOP-4B9EAO75 ss-local[1134]:  2024-08-03 13:44:20 INFO: listening at 127.0.0.1:1080
Aug 03 13:44:20 LAPTOP-4B9EAO75 ss-local[1134]:  2024-08-03 13:44:20 INFO: running from root user
```

# 3. install and config proxychains
when apt install from gitlab repo, you need proxy.
```shell
sudo apt install proxychains
```
then add this line to /etc/proxychains.conf
```shell
socks5  127.0.0.1 1080
```

# 4. install and configure gitlab-ce
install some install dependence:
```shell
sudo apt update
sudo apt install -y curl openssh-server ca-certificates tzdata perl
```
add gitlab repo to apt-get:
```shell
curl -x sock5h://127.0.0.1:1080 -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh > script.deb.sh
sudo proxychains bash script.deb.sh
sudo proxychains apt-get install gitlab-ce
```
external_url 和其他配置还待研究


