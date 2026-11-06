# Mikrotik Ovpn Server Setup

## Create Certificate


```bash
/certificate 
add name=ovpn-ca-template common-name=OVPN-CA key-size=2048 days-valid=3650 key-usage=key-cert-sign,crl-sign
sign ovpn-ca-template name=ovpn-ca

/certificate
add name=ovpn-server-template common-name=ovpn-server key-size=2048 days-valid=3650 key-usage=digital-signature,key-encipherment,tls-server
sign ovpn-server-template ca=ovpn-ca name=ovpn-server
set [find name=ovpn-ca] trusted=yes
```

## Create IP Pool
```bash
/ip pool
add name=ovpn-pool ranges=10.10.10.2-10.10.10.254
```
## Create profile
```bash
/ppp profile
add name=ovpn-profile local-address=10.10.10.1 remote-address=ovpn-pool use-encryption=required
```
## Create User
```bash
/ppp secret
add name=vpnuser password="YourStrongPassword123!" service=ovpn profile=ovpn-profile


/interface ovpn-server server
add name=ovpn-server1 disabled=no port=1194 protocol=udp mode=ip certificate=ovpn-server auth=sha256 cipher=aes256-cbc require-client-certificate=
yes
set [find name=ovpn-server1] push-routes="192.168.1.0 255.255.255.0"                 #[local IP Address]
```
## Firewall rull
```bash 
/ip firewall filter
add chain=input action=accept protocol=udp dst-port=1194 in-interface-list=WAN comment="Allow OpenVPN"
add chain=forward action=accept src-address=10.10.10.0/24 comment="VPN to LAN and Internet"
add chain=forward action=accept dst-address=10.10.10.0/24 connection-state=established,related comment="Return traffic to VPN"
```
## Export cert files
```bash
/certificate
add name=ovpn-client1-template \
\... common-name=ovpn-client1 \
\... key-size=2048 \
\... days-valid=3650 \
\... key-usage=digital-signature,key-encipherment,tls-client
sign ovpn-client1-template ca=ovpn-ca name=ovpn-client1
export-certificate ovpn-ca
export-certificate ovpn-client1

/certificate export-certificate ovpn-client1 export-passphrase="MySecureVPN@2026"
```
## Export cert & key files
```bash
/interface/ovpn-server/server/export-client-configuration \
server=ovpn-server1 \
server-address=YOUR_PUBLIC_IP \              #[Replace youre public ip address]
ca-certificate=cert_export_ovpn-ca.crt \
client-certificate=cert_export_ovpn-client1.crt \
client-cert-key=cert_export_ovpn-client1.key
```
