
# Dokumentasi Instalasi & Konfigurasi WARP WireProxy (SOCKS5 Publik)

## 1. Instalasi WARP via fscarmen script
Jalankan perintah berikut:
```bash
wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh && bash menu.sh a g4m062oD-Z65gmo31-Ux49K5a6
```
Keterangan:
- `a` → otomatis install
- `g4m062oD-Z65gmo31-Ux49K5a6` → lisensi atau ID khusus (gantikan sesuai kebutuhan)

Tunggu proses sampai selesai. Setelah instalasi, status WARP akan tampil, biasanya di akhir akan ada info **Local Socks5: 127.0.0.1:40001**.

---

## 2. Edit konfigurasi WireProxy
File konfigurasi ada di:
```
/etc/wireguard/proxy.conf
```
Buka file:
```bash
nano /etc/wireguard/proxy.conf
```
Cari bagian:
```ini
[Socks5]
BindAddress = 127.0.0.1:40001
```
Ubah jadi:
```ini
[Socks5]
BindAddress = [::]:40001
Username = userku
Password = passku123
```
Keterangan:
- `[::]` → bind ke semua interface IPv4 & IPv6 (dual-stack)
- Tambahkan `Username` dan `Password` untuk keamanan
- Gunakan password kuat
- atau bisa tanpa username dan password

---

## 3. Restart layanan WireProxy
```bash
systemctl restart wireproxy
```

---

## 4. Buka port SOCKS5 di firewall
Jika menggunakan `ufw`:
```bash
ufw allow 40001/tcp
```

---

## 5. Verifikasi port sudah listen di publik
```bash
ss -ltnp | grep 40001
```
Harus muncul:
```
LISTEN 0      128    *:40001     *:*    users:(("wireproxy",pid=xxxx,fd=xx))
```

---

## 6. Tes SOCKS5 dari luar VPS
**Cek IPv4 via SOCKS5**
```bash
curl --socks5-hostname 103.127.156.78:40001 -4 https://api.ipify.org?format=json
```
**Cek IPv6 via SOCKS5**
```bash
curl --socks5-hostname 103.127.156.78:40001 -6 https://api64.ipify.org?format=json
```
**Cek status WARP**
```bash
curl --socks5-hostname 103.127.156.78:40001 https://www.cloudflare.com/cdn-cgi/trace
```
Pastikan ada `warp=on` pada hasilnya.

---

## 7. Gunakan di server lain (contoh Xray)
Contoh outbound SOCKS5 di Xray:
```json
{
  "protocol": "socks",
  "settings": {
    "servers": [
      {
        "address": "103.127.132.192",
        "port": 40001,
        "users": [
          {
            "user": "userku",
            "pass": "passku123"
          }
        ]
      }
    ]
  }
}
```

---

## 8. Monitoring koneksi SOCKS5
- **Lihat koneksi aktif:**
```bash
ss -ntp sport = :40001
```
- **Pantau log service:**
```bash
journalctl -u wireproxy -f
```
- **Pantau koneksi real-time:**
```bash
tcpdump -i any tcp port 40001
```

---

📌 **Catatan Keamanan**:
- Selalu gunakan autentikasi pada SOCKS5 publik
- Batasi IP yang boleh akses
- Jangan biarkan open proxy tanpa kontrol, rawan disalahgunakan
