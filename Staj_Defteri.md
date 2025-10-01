# IoT Güvenlik Veri Seti Projesi - Staj Defteri

Bu defter, 30 günlük staj süresi boyunca "Nesnelerin İnterneti (IoT) için Saldırı Tespiti Veri Seti Oluşturma" projesinin adımlarını, öğrenilenleri ve karşılaşılan zorlukları günlük olarak belgelemektedir.

---

### **1. Hafta: Proje Planlama ve Temel Teknolojileri Öğrenme**

**Gün 1: Proje Başlangıcı ve Planlama**
Bugün stajımın ilk günü. Danışman hocamla bir toplantı yaparak projenin amacını ve hedeflerini netleştirdik. Projenin ana hedefi, makine öğrenmesi tabanlı saldırı tespit sistemleri (IDS) eğitmek için kullanılabilecek, etiketlenmiş bir ağ trafiği veri seti oluşturmak. Proje kapsamında Docker ile sanal bir IoT laboratuvarı kuracağımızı, normal ve saldırı trafiği üreteceğimizi ve bu trafiği etiketleyeceğimizi öğrendim. Gün sonunda proje takvimini ve yapılacaklar listesini oluşturdum.

**Gün 2: Docker Temelleri**
Bugün Docker öğrenmeye başladım. Docker'ın ne olduğunu, neden sanal makinelere göre daha avantajlı olduğunu araştırdım. "Image", "container", "volume", "network" gibi temel kavramları öğrendim. Docker Desktop'ı bilgisayarıma kurdum ve `docker run hello-world` komutuyla ilk container'ımı çalıştırdım. Bu, teknolojinin ne kadar hızlı ve kolay olduğunu görmemi sağladı.
_[Önerilen Ekran Görüntüsü: `hello-world` container'ının terminal çıktısı]_

**Gün 3: Docker Compose'a Giriş**
Birden fazla container'ı yönetmek için Docker Compose'un gerekli olduğunu öğrendim. `docker-compose.yml` dosyasının yapısını inceledim. Servislerin, ağların ve volümlerin nasıl tanımlandığını anladım. Basit bir Nginx ve Redis servisi içeren örnek bir `docker-compose.yml` dosyası yazarak `docker-compose up` komutunu denedim. İki servisin tek bir komutla ayağa kalkması ve birbiriyle iletişim kurabilmesi çok etkileyiciydi.
_[Önerilen Ekran Görüntüsü: Örnek Nginx ve Redis servislerinin `docker-compose ps` çıktısı]_

**Gün 4: Ağ Temelleri ve Wireshark**
Projenin temelini oluşturan ağ trafiğini anlamak için TCP/IP, portlar, protokoller (TCP, UDP) gibi temel ağ kavramlarını tekrar ettim. Ağ trafiğini analiz etmek için kullanılan Wireshark programını kurdum. Kendi bilgisayarımın ağ trafiğini yakalamaya başlayarak HTTP ve DNS paketlerini inceledim. Paketlerin yapısını, kaynak/hedef IP adreslerini ve port bilgilerini nasıl okuyacağımı öğrendim.
_[Önerilen Ekran Görüntüsü: Wireshark arayüzünde yakalanmış bir HTTP GET isteği paketi]_

**Gün 5: Kali Linux ve Güvenlik Araçları**
Saldırı senaryoları için Kali Linux kullanacağımızı öğrendim. VirtualBox üzerine Kali Linux kurdum. Nmap, Hydra ve hping3 gibi temel güvenlik araçlarının ne işe yaradığını araştırdım. `nmap localhost` komutuyla kendi makinemde basit bir port taraması yaparak Nmap'in temel kullanımını öğrendim. Bu araçların ne kadar güçlü olduğunu ve sorumlu bir şekilde kullanılması gerektiğini anladım.

---

### **2. Hafta: Laboratuvar Ortamının Kurulumu**

**Gün 6: Proje Yapısını Oluşturma**
Bugün proje için gerekli klasör yapısını oluşturdum (`http_api`, `traffic_generators` vb.). `docker-compose.yml` dosyasını yazmaya başladım. İlk olarak Mosquitto (MQTT broker) servisini tanımladım. Resmi `eclipse-mosquitto` imajını kullandım ve konfigürasyon dosyası için bir volume mount ekledim. `mosquitto.conf` dosyasını oluşturarak anonim erişime izin verdim.

**Gün 7: Node-RED ve MQTT Entegrasyonu**
`docker-compose.yml` dosyasına Node-RED servisini ekledim. Verilerin kalıcı olması için bir volume tanımladım. `docker-compose up` ile Mosquitto ve Node-RED'i çalıştırdım. Tarayıcıdan Node-RED arayüzüne (`http://localhost:1880`) eriştikten sonra, bir "MQTT in" nodu ekleyerek Mosquitto broker'ına başarıyla bağlandım. Bu iki servisin birbiriyle konuştuğunu görmek projenin ilk somut başarısıydı.
_[Önerilen Ekran Görüntüsü: Node-RED arayüzünde "connected" durumunu gösteren MQTT nodu]_

**Gün 8: Flask ile HTTP API Geliştirme (Bölüm 1)**
Akıllı priz simülasyonu için Flask tabanlı bir web API yazmaya başladım. `http_api` klasörü içinde `app.py`, `Dockerfile` ve `requirements.txt` dosyalarını oluşturdum. `/status` endpoint'ini yazarak prizin mevcut durumunu JSON formatında döndüren temel bir fonksiyon oluşturdum.

**Gün 9: Flask ile HTTP API Geliştirme (Bölüm 2)**
Bugün API'ye `/toggle` ve `/login` endpoint'lerini ekledim. `/login` endpoint'ini, kaba kuvvet saldırısı (brute force) için hedef olacak şekilde tasarladım ve basit bir "admin/password123" kullanıcı adı/şifresi belirledim. Ayrıca, tehlikeli komutları engellemek için `LAB_SAFE` ortam değişkeni kontrolünü ekledim. Bu, güvenli bir laboratuvar ortamı oluşturmanın önemini anlamamı sağladı.
_[Önerilen Ekran Görüntüsü: Postman veya `curl` ile `/status` endpoint'ine yapılan başarılı bir istek]_

**Gün 10: RTSP Kamera Servisini Ekleme**
`docker-compose.yml` dosyasına son servis olan RTSP kamera simülatörünü ekledim. Hazır bir imaj olan `aler9/rtsp-simple-server` kullandım. Tüm servisleri (`mosquitto`, `nodered`, `http_api`, `rtsp_camera`) `docker-compose up -d` komutuyla çalıştırdım. VLC Player kullanarak `rtsp://localhost:8554/mystream` adresinden test yayınına erişebildiğimi doğruladım. Laboratuvar ortamı artık tamamdı.
_[Önerilen Ekran Görüntüsü: Docker Desktop'ta çalışan 4 container'ın listesi]_

---

### **3. Hafta: Trafik Üretme ve Saldırı Simülasyonları**

**Gün 11: Normal Trafik Jeneratörü - MQTT**
`traffic_generators/mqtt_publisher.py` script'ini yazdım. Bu script, `paho-mqtt` kütüphanesini kullanarak her 2 saniyede bir rastgele sıcaklık ve nem verisi üretip Mosquitto broker'ına gönderiyor. Script'i çalıştırdığımda, Node-RED arayüzündeki "debug" panelinde MQTT mesajlarının başarıyla geldiğini gördüm. Bu, normal sensör trafiğini simüle etmenin ilk adımıydı.
_[Önerilen Ekran Görüntüsü: Node-RED debug penceresinde akan MQTT verileri]_

**Gün 12: Normal Trafik Jeneratörü - HTTP**
`traffic_generators/http_client.py` script'ini yazdım. Bu script ise `requests` kütüphanesini kullanarak periyodik olarak Flask API'nin `/status` ve `/toggle` endpoint'lerine istek atıyor. Bu, bir kullanıcının akıllı prizi kontrol etmesini simüle ediyor. İki jeneratör script'ini aynı anda çalıştırarak laboratuvarda sürekli bir normal trafik akışı sağladım.

**Gün 13: Trafik Yakalamaya Hazırlık**
Bugün `tcpdump` komutunu detaylıca öğrendim. Docker'ın oluşturduğu sanal ağ arayüzündeki trafiği nasıl yakalayacağımı araştırdım. Sadece proje servislerine ait portları (`1883`, `5000`, `1880`, `8554`) dinleyerek gereksiz trafikten kaçınmak için filtreler kullanmayı öğrendim. Yakalanan verileri daha sonra işlemek üzere `.pcap` formatında kaydetmenin en doğru yöntem olduğuna karar verdim.

**Gün 14: Saldırı Senaryosu 1 - Nmap Port Taraması**
Normal trafik jeneratörleri çalışırken ve `tcpdump` ile trafik kaydı yaparken, Kali Linux makinemden ilk saldırıyı gerçekleştirdim. `attack_commands.md` dosyasında hazırladığım Nmap komutunu kullanarak Docker host makinesinin IP adresini taradım. Nmap, açık olan 4 portu (1883, 1880, 5000, 8554) başarıyla tespit etti. Saldırının başlangıç ve bitiş zamanını dikkatlice not aldım.
_[Önerilen Ekran Görüntüsü: Nmap'in açık portları listelediği terminal çıktısı]_

**Gün 15: Saldırı Senaryosu 2 - Hydra Brute Force**
İkinci saldırı olarak, HTTP API'nin `/login` endpoint'ine yönelik bir kaba kuvvet saldırısı gerçekleştirdim. Kali üzerinde küçük bir şifre listesi (`pass.txt`) oluşturdum ve Hydra'yı bu listeyle çalıştırdım. Hydra, deneme yanılma yoluyla doğru şifrenin "password123" olduğunu kısa sürede buldu. Bu saldırının zaman damgalarını da kaydettim.
_[Önerilen Ekran Görüntüsü: Hydra'nın başarılı şifreyi bulduğu anın terminal çıktısı]_

**Gün 16: Saldırı Senaryosu 3 - hping3 DoS Saldırısı**
Son saldırı olarak, `hping3` aracıyla HTTP API'ye yönelik bir TCP SYN Flood (Denial of Service) saldırısı düzenledim. Saldırıyı yaklaşık 2 dakika boyunca sürdürdüm. Bu sırada, `http_client.py` script'inin API'ye bağlanmada zorlandığını veya hatalar verdiğini gözlemledim. Bu, DoS saldırısının servisi nasıl etkilediğini gösteren önemli bir gözlemdi. Saldırıyı `Ctrl+C` ile durdurup zaman bilgilerini not aldım ve `tcpdump` kaydını sonlandırdım.

---

### **4. Hafta: Veri İşleme, Etiketleme ve Raporlama**

**Gün 17: PCAP Dosyasını CSV'ye Dönüştürme**
Bugün, yakaladığım `capture.pcap` dosyasını işlemeye odaklandım. `scapy` Python kütüphanesini kullanarak paketleri okuyacak bir script (`pcap_to_csv.py`) yazdım. Script, her paketten zaman damgası, kaynak/hedef IP, portlar, protokol ve paket uzunluğu gibi temel özellikleri çıkarıp bir CSV dosyasına yazıyor. Script'i çalıştırarak ilk ham veri setimi (`unlabeled_traffic.csv`) oluşturdum.

**Gün 18: Otomatik Etiketleme Script'ini Geliştirme**
Veri setini etiketlemek için `auto_labeler.py` script'ini yazdım. Bu script, `pandas` kütüphanesini kullanarak CSV dosyasını okuyor ve kullanıcı tarafından sağlanan saldırgan IP'si, hedef IP'si ve zaman aralığı bilgilerine göre paketleri "normal" veya "attack" olarak etiketliyor. Bu yaklaşım, kontrollü bir laboratuvar ortamında son derece etkili bir etiketleme yöntemi sunuyor.

**Gün 19: Veri Setini Etiketleme**
`auto_labeler.py` script'ini, saldırı simülasyonları sırasında kaydettiğim notlarla (saldırgan/hedef IP'leri ve zaman aralıkları) çalıştırdım. Her bir saldırı tipi için script'i ayrı ayrı çalıştırarak `unlabeled_traffic.csv` dosyasındaki ilgili satırları etiketledim ve sonuçları `labeled_traffic.csv` dosyasına kaydettim. Sonunda, makine öğrenmesi modelleri için hazır, etiketlenmiş bir veri setim oldu.
_[Önerilen Ekran Görüntüsü: Excel'de veya bir metin düzenleyicide açılmış, "label" sütunu eklenmiş son CSV dosyası]_

**Gün 20: Proje Dokümantasyonu - README**
Projenin tekrar edilebilirliğini sağlamak için detaylı bir `README.md` dosyası hazırladım. Projenin amacını, gereksinimleri, kurulum adımlarını, trafik jeneratörlerinin ve saldırıların nasıl çalıştırılacağını adım adım anlattım. Veri yakalama ve etiketleme sürecini de detaylandırdım. İyi bir dokümantasyonun, projenin kendisi kadar önemli olduğunu anladım.

**Gün 21-30: Rapor Yazımı ve Sunum Hazırlığı**
Stajın geri kalanını, staj raporunu yazarak ve proje sunumunu hazırlayarak geçirdim. Projenin tüm aşamalarını, elde edilen sonuçları ve oluşturulan veri setinin potansiyel kullanım alanlarını detaylı bir şekilde raporladım. Bu projenin, siber güvenlik alanında pratik deneyim kazanmam için harika bir fırsat olduğunu düşünüyorum. Özellikle kontrollü bir ortamda saldırıları gerçekleştirip sonuçlarını analiz etmek çok öğreticiydi.