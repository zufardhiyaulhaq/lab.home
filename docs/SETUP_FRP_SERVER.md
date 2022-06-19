# FRP server
1. spin instance in cloud provider
2. download FRP server
```
wget https://github.com/fatedier/frp/releases/download/v0.43.0/frp_0.43.0_linux_amd64.tar.gz
```
3. Install FRP server
```
tar xzfv frp_0.43.0_linux_amd64.tar.gz
cp frp_0.43.0_linux_amd64/frpc /usr/bin/frpc
cp frp_0.43.0_linux_amd64/frps /usr/bin/frps
```
4. create configuration for FRP server
```
mkdir /etc/frps
vi /etc/frps/config.ini

[common]
server_addr = 0.0.0.0
server_port = 7000
token = 12345678

dashboard_addr = 0.0.0.0
dashboard_port = 7500

dashboard_user = admin
dashboard_pwd = admin


vi /etc/systemd/system/frps.service

[Unit]
Description=FRP Server Daemon

[Service]
Type=simple
ExecStart=/usr/bin/frps -c /etc/frps/config.ini
Restart=always
RestartSec=2s
LimitNOFILE=infinity

[Install]
WantedBy=multi-user.target

systemctl daemon-reload
systemctl start frps
systemctl enable frps
systemctl status frps
```
