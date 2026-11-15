# 🏦 Kredi Ön Değerlendirme Botu (n8n + Google Gemini AI)

Müşterilerden web formu aracılığıyla alınan finansal verileri analiz eden, bankacılık risk kurallarına (Borç/Gelir Oranı) göre kredi ön değerlendirme kararı üreten, sonuçları Google Sheets veritabanına kaydeden ve Google Gemini AI kullanarak müşteriye kişiselleştirilmiş bilgilendirme e-postası gönderen uçtan uca otomasyon iş akışı.

---

## 🚀 İş Akışı Mimarisi (Workflow)

```text
[Form Trigger] ➔ [JavaScript Finansal Hesaplama & Karar] ➔ [Google Sheets Log] ➔ [Google Gemini AI Metin Üretimi] ➔ [E-posta Bildirimi]
```

1. **Form Trigger (`On form submission`):** Müşteriden *Ad Soyad, Aylık Gelir, İstenen Kredi Tutarı, Vade (Ay), E-posta* ve *Mevcut Aylık Borç* bilgilerini toplar.
2. **Finansal Hesaplama & Karar Motoru (`Code in JavaScript`):**
   - Basit faiz formülü (`%3.5` aylık baz oran) ile toplam geri ödeme ve yeni aylık taksit tutarını hesaplar.
   - Toplam aylık borcun aylık gelire oranını (**DTI - Debt-to-Income Ratio**) bulur.
   - **Karar Kuralı:** Borç/Gelir oranı `%50` ve altındaysa **`ÖN ONAY`**, üzerindeyse **`RED`** statüsü atar.
3. **Veri Kaydı (`Google Sheets`):** Başvuru verilerini, hesaplanan taksit/oran metriklerini ve karar sonucunu merkezi tabloya ekler.
4. **Yapay Zeka ile Kurumsal İletişim (`Basic LLM Chain` + `Google Gemini`):** Başvuru sahibinin adını, finansal rakamlarını ve değerlendirme sonucunu (`ÖN ONAY` / `RED`) kullanarak kurumsal bir banka kredi uzmanı dilinde kişiselleştirilmiş e-posta metni oluşturur.
5. **Otomatik Gönderim (`Send an Email`):** Üretilen metni başvuru sahibinin e-posta adresine otomatik olarak iletir.

---

## 🛠️ Kullanılan Teknolojiler

- **n8n** (Workflow Otomasyonu)
- **JavaScript** (Finansal Algoritma & Karar Mantığı)
- **Google Sheets API** (Kayıt & Loglama)
- **Google Gemini (`gemini-2.5-flash`)** (LangChain LLM Entegrasyonu)
- **SMTP / Gmail** (Otomatik E-posta Gönderimi)

---

## ⚙️ Kurulum Adımları

Bu projeyi kendi n8n ortamınızda çalıştırmak için aşağıdaki adımları izleyin:

### 1. İş Akışını İçe Aktarın
1. Bu repodaki `Kredi_On_Degerlendirme_Public.json` dosyasını bilgisayarınıza indirin.
2. n8n arayüzünde yeni bir workflow açın, sağ üst menüden (**`...` ➔ Import from File**) dosyayı içeri aktarın.

### 2. Google Sheets Tablosunu Hazırlayın
1. Google Drive üzerinde yeni bir Google Sheets dosyası oluşturun.
2. İlk satıra (`A1:E1`) aşağıdaki sütun başlıklarını birebir ekleyin:

| Ad_Soyad | Aylik_Gelir | Kredi_Tutari | Borc_Gelir_Orani | Durum |
| :--- | :--- | :--- | :--- | :--- |

3. n8n içindeki **Append row in sheet** düğümünü açın, kendi Google Sheets hesabınızı bağlayın ve `BURAYA_GOOGLE_SHEET_ID_GIRIN` alanına oluşturduğunuz tablonun ID'sini girin.

### 3. Yapay Zeka ve E-posta Kimlik Bilgilerini Tanımlayın
1. **Google Gemini Chat Model** düğümüne tıklayın ve Google AI Studio'dan aldığınız kendi Gemini API anahtarınızı tanımlayın.
2. **Send an Email** düğümüne tıklayın, kendi SMTP/Gmail hesabınızı seçin ve `From Email` alanına gönderici adresinizi yazın.

### 4. Test Edin
1. **On form submission** düğümündeki **Test URL** bağlantısını açın.
2. Örnek bir başvuru formu doldurup gönderin (`Borc_Gelir_Orani <= 50` ise **ÖN ONAY**, `> 50` ise **RED** senaryosunu test edebilirsiniz).
3. Google Sheets tablonuza satır eklendiğini ve müşteriye kişiselleştirilmiş karar e-postasının ulaştığını doğrulayın.

---

## 🔒 Güvenlik ve Gizlilik Notu
Bu repoda paylaşılan JSON şablonu hiçbir kişisel veri, e-posta adresi veya API anahtarı içermez. Tüm kimlik bilgileri (Credentials) kullanıcı tarafından kendi n8n ortamında yerel olarak tanımlanır.
