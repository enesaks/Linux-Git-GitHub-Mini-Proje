# Linux-Git-GitHub Mini Proje

## Proje Amacı:

Bu proje, bir web sitesinin AWS EC2 instance üzerinde Apache web sunucusu aracılığıyla deploy edilerek internet üzerinden erişilebilir hale getirilmesini hedeflemektedir. Çalışma kapsamında, sunucu kurulumu ve yapılandırması, gerekli ağ ve güvenlik ayarlarının yapılması, web sunucusunun devreye alınması, Git/GitHub üzerinden kod yönetimi ve Bash betikleri ile otomatik dağıtım (CD) sürecinin oluşturulması adımları gerçekleştirilmiştir. Böylece hem bulut tabanlı altyapı yönetimi hem de uçtan uca dağıtım süreçleri uygulamalı olarak pekiştirilmiştir.

Yapılan Adımlar
1. EC2 Instance Oluşturma
	•	Amazon Linux 2023 AMI kullanılarak bir adet t2.micro tipinde EC2 instance açıldı.
	•	Güvenlik grubu ayarlarında SSH (22) ve HTTP (80) portları dış erişime açıldı.

 
2.	Apache Web Server Kurulumu ve Otomatik Başlatma
	•	httpd paketi sisteme yüklendi.

	•	Bilgisayar yeniden başlatıldığında otomatik olarak çalışması için systemctl enable httpd ayarı yapıldı.

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


4.	Otomatik Çalıştırma (systemd + cron)
   
	•	Script’in her 10 dakikada bir otomatik çalışması için zamanlama ayarlandı.

	•	systemd timer veya cron kullanılarak periyodik olarak çalışması sağlandı.



