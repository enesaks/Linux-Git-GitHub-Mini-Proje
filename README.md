# Linux-Git-GitHub Mini Proje

## Proje Amacı:

Bu proje, bir web sitesinin AWS EC2 instance üzerinde Apache web sunucusu aracılığıyla deploy edilerek internet üzerinden erişilebilir hale getirilmesini hedeflemektedir. Çalışma kapsamında, sunucu kurulumu ve yapılandırması, gerekli ağ ve güvenlik ayarlarının yapılması, web sunucusunun devreye alınması, Git/GitHub üzerinden kod yönetimi ve Bash betikleri ile otomatik dağıtım (CD) sürecinin oluşturulması adımları gerçekleştirilmiştir. Böylece hem bulut tabanlı altyapı yönetimi hem de uçtan uca dağıtım süreçleri uygulamalı olarak pekiştirilmiştir.

Yapılan Adımlar
1. EC2 Instance Oluşturma
	•	Amazon Linux 2023 AMI kullanılarak bir adet t2.micro tipinde EC2 instance açıldı.
	•	Güvenlik grubu ayarlarında SSH (22) ve HTTP (80) portları dış erişime açıldı.

<img width="1521" height="1133" alt="Image" src="https://github.com/user-attachments/assets/e3a8e7d6-534b-40e3-b8e4-80f41c778e9a" />

<img width="1103" height="876" alt="Image" src="https://github.com/user-attachments/assets/f856527d-af59-4163-9c4e-07f686b0102b" />
 
2.	Apache Web Server Kurulumu ve Otomatik Başlatma
	•	httpd paketi sisteme yüklendi.

	•	Bilgisayar yeniden başlatıldığında otomatik olarak çalışması için systemctl enable httpd ayarı yapıldı.
```
	sudo dnf install httpd
	sudo systemctl start httpd   
	sudo systemctl enable httpd  
	
```


4.	Deploy Scripti Oluşturma

	•	/home/ec2-user/script.sh adında bir Bash script hazırlandı.

	•	Script’in işlevleri:

	1.	/tmp/onepage klasörü yoksa git clone ile onepage GitHub reposu indirildi.
    
	3.	Klasör varsa mevcut kodlar git pull ile güncellendi.
    
	5.	onepage klasörü içeriğindeki tüm dosyalar /var/www/html dizinine kopyalandı.
    
	•	Aynı isimde dosyalar varsa üzerine yazıldı.

```
#!/bin/bash
temp='/onepage'
a=0

if [ -d "/tmp/onepage" ]; then
        echo "Proje Bulundu"
        cd /tmp/onepage
        git reset --hard origin/main
        git pull
else
    echo "Proje Bulunamadı"
    cd /tmp
    git clone https://github.com/enesaks/Linux-Git-GitHub-Mini-Proje.git
fi

sudo cp -rf /tmp/onepage/. /var/www/html
```


4.	Otomatik Çalıştırma (systemd)
   
	•	Script’in her 10 dakikada bir otomatik çalışması için zamanlama ayarlandı.

	•	systemd timer kullanılarak periyodik olarak çalışması sağlandı.

```
sudo tee /etc/systemd/system/onepage.service >/dev/null <<'EOF'
[Unit]
Description=Run onepage script
After=network-online.target
Wants=network-online.target

[Service]
Type=oneshot
User=ec2-user
ExecStart=/usr/local/bin/onepage.sh
EOF

# systemd timer (boot'ta ve her 10 dakikada bir)
sudo tee /etc/systemd/system/onepage.timer >/dev/null <<'EOF'
[Unit]
Description=Run onepage script every 10 minutes and at boot

[Timer]
OnBootSec=0min
OnUnitActiveSec=10min
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Etkinleştir ve başlat
sudo systemctl daemon-reload
sudo systemctl enable --now onepage.timer

# Durum kontrolü
systemctl status --no-pager onepage.timer
systemctl list-timers --all | grep onepage || true

```

