#!/bin/bash

echo "🔧 Masukkan port SSH baru yang ingin digunakan:"
read -p "Port: " PORT_BARU

# Validasi input port
if ! [[ "$PORT_BARU" =~ ^[0-9]+$ ]] || [ "$PORT_BARU" -lt 1 ] || [ "$PORT_BARU" -gt 65535 ]; then
  echo "❌ Port tidak valid! Harus berupa angka 1-65535."
  exit 1
fi

SSHD_CONFIG="/etc/ssh/sshd_config"

echo "📄 Mengubah port SSH ke $PORT_BARU..."
if grep -q "^Port" "$SSHD_CONFIG"; then
    sed -i "s/^Port .*/Port $PORT_BARU/" "$SSHD_CONFIG"
else
    echo "Port $PORT_BARU" >> "$SSHD_CONFIG"
fi

echo "🔄 Reload systemd dan restart ssh socket/service..."
systemctl daemon-reexec
systemctl daemon-reload
systemctl restart ssh.socket
systemctl restart ssh.service

echo "🛡️ Konfigurasi firewall UFW..."
yes | ufw enable

ufw allow "$PORT_BARU"/tcp
ufw delete allow 22/tcp 2>/dev/null || echo "⚠️ Port 22 tidak ditemukan di UFW, dilewati..."

# Optional: open all ports (not recommended for production)
ufw allow 1:65535/tcp
ufw allow 1:65535/udp

ufw reload

echo "🔍 Mengecek apakah port $PORT_BARU sudah listening..."
if ss -tuln | grep -q ":$PORT_BARU "; then
    echo "✅ SSH sekarang berjalan di port $PORT_BARU!"
else
    echo "❌ SSH belum berjalan di port $PORT_BARU. Silakan cek konfigurasi manual."
fi

echo "📌 Login SSH selanjutnya gunakan: ssh -p $PORT_BARU user@ip_address"
