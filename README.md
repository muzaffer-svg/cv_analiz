#  AI Destekli CV Analiz Asistanı (Microservice Architecture)

Bu proje, sisteme yüklenen özgeçmişleri (CV) yapay zeka (LLM) ajanları aracılığıyla analiz eden, ATS (Aday Takip Sistemi) standartlarına göre puanlayan ve adaya gelişim odaklı yapılandırılmış JSON verisi sunan otomatik bir n8n iş akışıdır. 

Proje, hem son kullanıcılar için görsel bir arayüz (Form) hem de diğer sistemlerle entegre çalışabilmesi için REST API (Webhook) desteği sunan **Çift Çıkışlı (Dual-Output) Yönlendirme Mimarisi** ile tasarlanmıştır.

---

##  Mimari Özet ve Proje Amacı

**Amaç:** İnsan kaynakları süreçlerindeki manuel CV ön eleme operasyonlarını otomatize etmek ve adaylara STAR (Durum, Görev, Eylem, Sonuç) metodolojisine dayalı, objektif teknik geri bildirimler sunmak.

**Kullanılan Teknolojiler ve Mimari:**
- **Konteynerleştirme (Altyapı):** Docker & Docker Compose
- **Orkestrasyon:** n8n (Node-based Workflow Automation)
- **AI Katmanı:** Ollama (Lokal LLM) / Alternatif API'ler (OpenAI, Groq vb.)
- **Veri İşleme:** Dinamik JSON Parsing ve Koşullu Yönlendirme (If/Switch)
- **Konfigürasyon Yönetimi:** `.env` tabanlı dinamik yapılandırma

---

##  Sistem Gereksinimleri

Projeyi lokal ortamınızda ayağa kaldırmak için aşağıdaki araçların kurulu olması gerekmektedir:
- [Docker](https://www.docker.com/) ve Docker Compose
- [Git](https://git-scm.com/)
- *Opsiyonel (Lokal model kullanılacaksa):* [Ollama](https://ollama.com/)

---

##  Kurulum ve Ayağa Kaldırma (Deployment)

Proje tamamen taşınabilir (portable) olarak tasarlanmıştır. Aşağıdaki adımları takip ederek saniyeler içinde kendi lokalinizde ayağa kaldırabilirsiniz:

**1. Repoyu Klonlayın:**
```bash
git clone [https://github.com/muzaffer-svg/cv-analiz-asistani.git](https://github.com/muzaffer-svg/cv-analiz-asistani.git)
cd cv-analiz-asistani


## 2. Çevre Değişkenlerini (`.env`) Ayarlayın
 
Port çakışmalarını önlemek ve zaman dilimini ayarlamak için örnek konfigürasyon dosyasını oluşturun:
 
```bash
cp .env.example .env
```
 
> **İpucu:** İhtiyaç halinde `.env` dosyasını `nano .env` ile açarak `N8N_PORT` değerini değiştirebilirsiniz.
 
---
 
## 3. Docker Konteynerlerini Başlatın
 
```bash
docker compose up -d
```
 
---
 
**4. İş Akışını (Workflow) Sisteme Dahil Edin:**
- Tarayıcınızdan `http://localhost:5678` adresine gidin. *(Not: Eğer .env dosyasında N8N_PORT değerini değiştirdiyseniz, adrese o port numarasını yazmalısınız, örn: http://localhost:5679).*
- n8n arayüzünde yeni, boş bir iş akışı (workflow) oluşturun.
- Sağ üst köşedeki menüden (üç nokta simgesi) **"Import from File"** seçeneğine tıklayarak proje dizinindeki `workflows/workflow.json` dosyasını içeri aktarın.
- Akışı kaydet (Save) butonuna basın. Eğer farklı bir yapay zeka sağlayıcısı kullanacaksanız ilgili şifreleri (Credentials) girdikten sonra akışı sağ üstten **Active** (Aktif) hale getirin.
---
 
## Sağlayıcı (Provider) Seçenekleri ve Konfigürasyon

Bu sistem n8n içerisindeki `AI Agent` düğümü üzerinden dilediğiniz LLM sağlayıcısına geçiş yapabileceğiniz esnek bir yapıdadır:

- **Ollama (Aynı Bilgisayarda Lokal):** n8n Docker içinden bilgisayarınızdaki Ollama'ya erişecekse Base URL `http://host.docker.internal:11434` olmalıdır.
- **Ollama (Resmi Bulut / Cloud API):** Bilgisayarınızı yormadan doğrudan Ollama'nın bulut sunucularını kullanmak isterseniz:
  1. [https://ollama.com/settings/keys](https://ollama.com/settings/keys) adresine giderek ücretsiz bir API Key oluşturun.
  2. n8n arayüzündeki `Ollama Chat Model` düğümünde **Credentials** kısmından yeni bir bağlantı açın ve bu API Key'i girin.
  3. **Base URL** alanına `https://api.ollama.com` adresini yazarak kaydedin.

> ** Güvenlik Uyarısı:** Güvenlik politikaları gereği `workflow.json` dosyasında hiçbir API anahtarı (Credential) barındırılmaz. Şifreleri ve URL ayarlarını n8n arayüzünden manuel olarak tanımlamanız gerekmektedir.

---

## Sistemin Test Edilmesi (Testability)

Sistem, if/else mantığıyla iki farklı tetikleyiciyi aynı anda dinleyecek şekilde tasarlanmıştır:

### Yöntem 1: Arayüz (UI) Üzerinden İnsan Testi
n8n arayüzünde sol alttaki **Execute Workflow** butonuna tıklayın. Açılan forma `samples/` klasöründe bulunan test CV'lerinden birini yapıştırın. Sistem size görselleştirilmiş bir analiz raporu sunacaktır.

### Yöntem 2: Webhook (API) Üzerinden Makine Testi
Sistemi bir mikroservis gibi test etmek için, iş akışı dinleme modundayken terminalinizden aşağıdaki cURL komutunu çalıştırın:

*(Not: Komuttaki `5678` portunu, `.env` dosyasında belirlediğiniz kendi `N8N_PORT` değerinize göre değiştirmeyi unutmayın!)*

```bash
curl -X POST "http://localhost:5678/webhook-test/cv-analiz" \
     -H "Content-Type: application/json" \
     -d '{"CV_Metni": "AD SOYAD: Emre Demir, UNVAN: Yazılım Geliştirici, YETKİNLİKLER: Java, Python, SQL. DENEYİM: Kütüphane uygulaması geliştirdim."}'
```
 
**Beklenen Çıktı:** İşlem sonucunda terminal ekranınıza yalnızca makine okunabilir (Machine Readable) kusursuz bir JSON veri objesi dönecektir.
 
---
 
##  Sık Karşılaşılan Sorunlar (FAQ)
 
**1. Port Çakışması (Address already in use / Connection Refused)**
- **Sorun:** Sistemi çalıştırdığınızda n8n arayüzüne ulaşamıyorsanız veya Docker kurulumda hata veriyorsa, bilgisayarınızda `5678` portunu kullanan eski bir proje açık kalmış olabilir.
- **Çözüm:** Proje klasöründeki gizli `.env` dosyasını bir metin editörüyle açın (veya terminalden güncelleyin) ve `N8N_PORT=5679` (veya 8080 gibi boş bir sayı) yazarak kaydedin. Ardından `docker compose down` ve `docker compose up -d` komutlarıyla sistemi yeni portta başlatın.

**2. Konteyner İsmi Çakışması (The container name is already in use)**
- **Sorun:** Başlatma komutunu verdiğinizde *"The container name '/n8n_cv_projesi' is already in use"* hatası alıyorsanız, arka planda çalışan veya takılı kalan eski bir proje var demektir.

- **Çözüm A (Eski Sistemi Silip Yeniden Kurmak İsteyenler İçin):**
  Sistemi temizlemek ve takılı kalan konteyneri zorla silmek için terminale şu komutu yazın:
  ```bash
  docker rm -f n8n_cv_projesi
  ```

- **Çözüm B (Aynı Anda 2 Farklı Projeyi Paralel Çalıştırmak İsteyenler İçin):**
Eski sisteminizi kapatmadan veya silmeden, test amaçlı yepyeni bir bağımsız sistem daha açmak istiyorsanız şu 3 değişikliği yapmalısınız:
1. Projeyi bilgisayarınızda **farklı isimli yeni bir klasöre** kopyalayın (Örn: `cv_analiz_test2`).
2. Yeni klasördeki `.env` dosyasını açıp portu değiştirin (Örn: `N8N_PORT=5679`).
3. Yeni klasördeki `docker-compose.yml` dosyasını açıp `container_name: n8n_cv_projesi` satırını benzersiz bir isimle değiştirin (Örn: `container_name: n8n_cv_projesi_2`).

*Not: Eğer sadece adı değiştirip portu ve klasörü ayırmazsanız, konteynerler çakışacak veya arka planda gereksiz RAM tüketecektir.*
 


**Webhook "Invalid JSON" Hatası**
API üzerinden istek attığınızda bu hatayı alıyorsanız, `Respond to Webhook` düğümünde arayüz (form) için hazırlanmış emojili metnin kalıp kalmadığını kontrol edin. Webhook çıkışında Body kısmı yalnızca aşağıdaki formatta olmalıdır:
 
```
{{ JSON.parse($json.output) }}
```
 
**Ollama Connection Refused**
Model çalışmıyorsa:
- Lokal bilgisayarınızda Ollama uygulamasının arka planda açık olduğundan emin olun.
- n8n `AI Agent` ayarlarında Base URL'in `localhost` yerine `host.docker.internal` olarak yazıldığını doğrulayın.
