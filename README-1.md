# Dokumentasi Instalasi dan Konfigurasi WARP (wgcf)

Panduan ini menjelaskan langkah-langkah instalasi dan konfigurasi **Cloudflare WARP via WireGuard (wgcf)** pada Linux server.

---

## 1. Instalasi `wgcf`

```bash
wget https://github.com/ViRb3/wgcf/releases/download/v2.2.28/wgcf_2.2.28_linux_amd64
mv wgcf_2.2.28_linux_amd64 wgcf
chmod +x wgcf
mv wgcf /usr/local/bin/
```

---

## 2. Registrasi Akun & Generate Profile

```bash
wgcf register
wgcf generate
mv wgcf-profile.conf /etc/wireguard/wgcf.conf
```

---

## 3. Edit Konfigurasi `/etc/wireguard/wgcf.conf`

```ini
[Interface]
PrivateKey = <isi-private-key-dari-wgcf-generate>
Address = 172.16.0.2/32, 2606:4700:110:86d3:fad7:5210:88a5:e4b5/128
DNS = 1.1.1.1, 1.0.0.1
MTU = 1280
# KUNCI: jangan biarkan wg-quick utak-atik main routing table
Table = off

[Peer]
PublicKey = bmXOC+F1FxEMF9dyiK2H5/1SUtzH0JuVo51h2wPfgyo=
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = 162.159.192.1:2408
PersistentKeepalive = 25
```

Cek permission & validasi konfigurasi:

```bash
chmod 600 /etc/wireguard/wgcf.conf
wg-quick strip wgcf >/dev/null && echo "Config OK"
```

---

## 4. Buat Drop-in Systemd

```bash
sudo mkdir -p /etc/systemd/system/wg-quick@wgcf.service.d
sudo tee /etc/systemd/system/wg-quick@wgcf.service.d/override.conf >/dev/null <<'EOF'
[Unit]
After=network-online.target
Wants=network-online.target

[Service]
Environment=PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
EOF

sudo systemctl daemon-reload
```

---

## 5. (Opsional) Hapus DNS di Config

Jika ada error `resolvconf`, edit config:

```bash
nano /etc/wireguard/wgcf.conf
```

Hapus baris:
```
DNS = 1.1.1.1, 1.0.0.1
```

Atur DNS manual di `/etc/resolv.conf` bila perlu.

---

## 6. Start dan Enable Service

```bash
sudo systemctl stop wg-quick@wgcf 2>/dev/null || true
sudo wg-quick down wgcf 2>/dev/null || true

sudo systemctl start wg-quick@wgcf
sleep 1
systemctl status wg-quick@wgcf -n 50

# Enable auto-start saat boot
sudo systemctl enable wg-quick@wgcf
```

---

## 7. Verifikasi Koneksi

```bash
wg show
ip route | head -n 3
ip -6 route | head -n 3

curl --interface wgcf https://www.cloudflare.com/cdn-cgi/trace | grep warp
```

Jika berhasil, akan muncul output:
```
warp=on
```

---

## 8. Debugging

Jika masih gagal, ambil log untuk dicek:

```bash
journalctl -u wg-quick@wgcf -n 100 --no-pager
```

---

## 9. konfigurasi xray
```json
{
  "log" : {
    "access": "/var/log/xray/access.log",
    "error": "/var/log/xray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
      {
      "listen": "127.0.0.1",
      "port": 10000,
      "protocol": "dokodemo-door",
      "settings": {
        "address": "127.0.0.1"
      },
      "tag": "api"
    },
   {
      "listen": "127.0.0.1",
      "port": "10003",
      "protocol": "trojan",
      "settings": {
          "decryption":"none",
           "clients": [
              {
                 "password": "36326ea2-6994-4d53-bcd5-f43bbdaec562"
  #trojanws
  #tr# yui 25-Aug-2024
  },{"password": "948989f2-f25f-4b98-8d9b-b61441415564","email": "yui@gmail.com"
  #tr# rhh 25-Jun-2025
  },{"password": "ffd02688-3eba-4eea-bb8a-b2e032b9e968","email": "rhh@gmail.com"
              }
          ],
         "udp": true
       },
       "streamSettings":{
           "network": "ws",
           "wsSettings": {
               "path": "/trojan-ws"
            }
         }
     },
    {
        "listen": "127.0.0.1",
        "port": "10007",
        "protocol": "trojan",
        "settings": {
          "decryption":"none",
             "clients": [
               {
                 "password": "36326ea2-6994-4d53-bcd5-f43bbdaec562"
  #trojanws
  #tr# yui 25-Aug-2024
  },{"password": "948989f2-f25f-4b98-8d9b-b61441415564","email": "yui@gmail.com"
  #tr# rhh 25-Jun-2025
  },{"password": "ffd02688-3eba-4eea-bb8a-b2e032b9e968","email": "rhh@gmail.com"
               }
           ]
        },
         "streamSettings":{
         "network": "grpc",
           "grpcSettings": {
               "serviceName": "trojan-grpc"
         }
      }
   }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "ip": [
          "0.0.0.0/8",
          "10.0.0.0/8",
          "100.64.0.0/10",
          "169.254.0.0/16",
          "172.16.0.0/12",
          "192.0.0.0/24",
          "192.0.2.0/24",
          "192.168.0.0/16",
          "198.18.0.0/15",
          "198.51.100.0/24",
          "203.0.113.0/24",
          "::1/128",
          "fc00::/7",
          "fe80::/10"
        ],
        "outboundTag": "blocked"
      },
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "type": "field",
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ]
      }
    ]
  },
  "stats": {},
  "api": {
    "services": [
      "StatsService"
    ],
    "tag": "api"
  },
  "policy": {
    "levels": {
      "0": {
        "statsUserDownlink": true,
        "statsUserUplink": true
      }
    },
    "system": {
      "statsInboundUplink": true,
      "statsInboundDownlink": true,
      "statsOutboundUplink" : true,
      "statsOutboundDownlink" : true
    }
  }
}
{
  "outbounds": [
    {
      "protocol": "dns",
      "tag": "dns-out"
    },
    {
      "streamSettings": {
        "sockopt": {
          "mark": 255
        }
      },
      "settings": {
        "domainStrategy": "UseIPv4"
      },
      "protocol": "freedom",
      "tag": "direct"
    },
    {
      "protocol": "blackhole",
      "tag": "blackhole"
    },
    {
      "settings": {
        "servers": [
          {
            "port": 10000,
            "address": "127.0.0.1"
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none"
      },
      "protocol": "socks",
      "tag": "out"
    }
  ],
  "log": {
    "loglevel": "warning"
  },
  "dns": {
    "hosts": {
      "dns.tiarap": "174.138.21.128",
      "dns.1host": "76.76.2.38",
      "geosite:category-ads-all": "127.0.0.1"
    },
    "servers": [
      {
        "address": "https://doh.tiarap.org/dns-query",
        "domains": ["geosite:geolocation-!cn"],
        "expectIPs": ["geoip:!cn"]
      },
      "174.138.21.128",
      {
        "address": "174.138.21.128",
        "port": 53,
        "domains": ["geosite:id", "geosite:category-games@id"],
        "expectIPs": ["geoip:id"],
        "skipFallback": true
      },
      {
        "address": "localhost",
        "skipFallback": true
      }
    ]
  },
  "routing": {
    "rules": [
      {
        "type": "field",
        "outboundTag": "Reject",
        "domain": ["geosite:category-ads-all"]
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "domain": [
          "geosite:private",
          "geosite:apple-id",
          "geosite:google-id",
          "geosite:tld-id",
          "geosite:category-games@id"
        ]
      },
      {
        "type": "field",
        "outboundTag": "Proxy",
        "domain": ["geosite:geolocation-!cn"]
      },
      {
        "type": "field",
        "outboundTag": "Direct",
        "domain": ["geosite:id"]
      },
      {
        "type": "field",
        "outboundTag": "Proxy",
        "network": "tcp,udp"
      }
    ]
  },
  "inbounds": [
    {
      "port": 10000,
      "protocol": "dokodemo-door",
      "settings": {
        "port": 53,
        "network": "udp",
        "address": "174.138.21.128"
      },
      "tag": "dns-in",
      "listen": "127.0.0.1"
    }
  ]
}
```

## Selesai 🎉

Sekarang WARP via wgcf sudah berjalan di VPS Anda.
