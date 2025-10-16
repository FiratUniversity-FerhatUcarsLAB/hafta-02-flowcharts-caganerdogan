BAŞLA

  // --- Yardımcı Fonksiyonlar (Önceki Adımlardan Alınmıştır) ---

  FONKSIYON KimlikDogrula(TC_Kimlik, Sifre)
    // Kullanıcı doğrulaması yapılır
    VERI_TABANI_SORGUSU: KullaniciVarMi(TC_Kimlik, Sifre)
    EĞER SorguSonucu == BAŞARILI İSE
      GeriDön: DOĞRU
    DEĞİLSE
      GeriDön: YANLIŞ
    SON_EĞER
  SON_FONKSIYON

  FONKSIYON HastaneRandevuSistemi(KullaniciID)
    // Randevu Modülü Akışı
    EKRAN_GÖSTER: "--- Randevu Modülü ---"
    // Adımlar: Poliklinik Seçimi, Doktor Seçimi, Uygun Saatleri Getirme, Randevu Onaylama, Kaydetme ve SMS Gönderme
    // ... (Önceki Randevu Sistemi algoritmasının detayları buraya gelir) ...
    EKRAN_GÖSTER: "Randevu Modülü İşlemi Tamamlandı."
  SON_FONKSIYON

  FONKSIYON TahlilSonucuGoruntulemeSistemi(HastaID)
    // Tahlil Modülü Akışı
    EKRAN_GÖSTER: "--- Tahlil Sonucu Modülü ---"
    // Adımlar: Tahlil Varlığı Kontrolü, Sonuç Hazır Kontrolü, Görüntüleme/Bekleme Mesajı, PDF İndirme
    // ... (Önceki Tahlil Sistemi algoritmasının detayları buraya gelir) ...
    EKRAN_GÖSTER: "Tahlil Sonucu Modülü İşlemi Tamamlandı."
  SON_FONKSIYON

  // -----------------------------------------------------------
  
  // 1. GİRİŞ VE KİMLİK DOĞRULAMA
  
  DÖNGÜ_BAŞLAT: Giris
    EKRAN_GÖSTER: "Hastane Bilgi Sistemi - Giriş Ekranı"
    KullaniciAdi = KULLANICIDAN_AL("T.C. Kimlik No:")
    Sifre = KULLANICIDAN_AL("Şifre:")
    
    GirisBasarili = KimlikDogrula(KullaniciAdi, Sifre)
    
    EĞER GirisBasarili == YANLIŞ İSE
      EKRAN_GÖSTER: "Hatalı Giriş. Lütfen tekrar deneyin."
      TEKRAR_ET: Giris
    DEĞİLSE
      DÖNGÜDEN_ÇIK: Giris
    SON_EĞER
  DÖNGÜ_SONU: Giris
  
  HastaID = KullaniciAdi // Başarılı girişten sonra TC Kimlik No'yu Hasta ID olarak kullan
  IslemDevamEdiyor = DOĞRU
  
  // 2. ANA MENÜ VE MODÜL YÖNLENDİRMESİ
  
  DÖNGÜ_BAŞLAT: AnaMenu
    
    EĞER IslemDevamEdiyor == YANLIŞ İSE
      DÖNGÜDEN_ÇIK: AnaMenu
    SON_EĞER
    
    EKRAN_GÖSTER: "--- ANA MENÜ ---"
    EKRAN_GÖSTER: "1. Randevu Al/Görüntüle"
    EKRAN_GÖSTER: "2. Tahlil Sonucu Görüntüle"
    EKRAN_GÖSTER: "3. Çıkış"
    
    Secim = KULLANICIDAN_AL("Lütfen yapmak istediğiniz işlemi seçin (1, 2 veya 3):")
    
    EĞER Secim == 1 İSE
      // Randevu Modülü Çağrısı
      CAGIR_FONKSIYON: HastaneRandevuSistemi(HastaID)
      
    DEĞİLSE EĞER Secim == 2 İSE
      // Tahlil Modülü Çağrısı
      CAGIR_FONKSIYON: TahlilSonucuGoruntulemeSistemi(HastaID)
      
    DEĞİLSE EĞER Secim == 3 İSE
      // Çıkış İşlemi
      EKRAN_GÖSTER: "Sistemden çıkış yapılıyor. İyi günler dileriz."
      IslemDevamEdiyor = YANLIŞ
      
    DEĞİLSE
      EKRAN_GÖSTER: "Geçersiz seçim. Lütfen 1, 2 veya 3 girin."
    SON_EĞER
    
    EĞER IslemDevamEdiyor == DOĞRU İSE
        KULLANICIDAN_BEKLE("Ana Menüye dönmek için herhangi bir tuşa basın...")
    SON_EĞER
    
  DÖNGÜ_SONU: AnaMenu

SON
