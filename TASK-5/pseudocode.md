BAŞLA

  // --- Sistem Değişkenleri ve İlk Durum ---
  SistemAktif = DOĞRU
  AlarmDurumu = KAPALI
  TehlikeSeviyesi = 0 // 0: Güvenli, 1: Uyarı, 2: Alarm
  
  FONKSIYON SensörleriOku()
    // Evdeki tüm sensörlerden (Hareket, Kapı/Pencere, Duman vb.) veriyi al
    SensorVerileri = VERI_TOPLA(Tüm_Sensorler)
    GeriDön: SensorVerileri
  SON_FONKSIYON
  
  FONKSIYON TehditAlgıla(Veriler)
    // Sensör verilerini analiz et ve tehdit seviyesini belirle
    
    EĞER Veriler.DumanAlgılandı == DOĞRU İSE
      GeriDön: 2 // Yüksek Tehlike (Yangın)
    DEĞİLSE EĞER Veriler.KapıAçıldı_Yetkisiz == DOĞRU VEYA Veriler.HareketAlgılandı_Gece == DOĞRU İSE
      GeriDön: 2 // Yüksek Tehlike (Hırsızlık)
    DEĞİLSE EĞER Veriler.BilinmeyenSesAlgılandı == DOĞRU İSE
      GeriDön: 1 // Düşük Tehlike (Uyarı)
    DEĞİLSE
      GeriDön: 0 // Güvenli
    SON_EĞER
  SON_FONKSIYON
  
  // --- ANA SİSTEM DÖNGÜSÜ (7/24 Çalışır) ---
  DÖNGÜ_SÜREKLİ_ÇALIŞIR (KOŞUL: SistemAktif == DOĞRU)
    
    // 1. Sensör Okuma
    MevcutVeriler = SensörleriOku()
    
    // 2. Tehdit Algılama
    YeniTehlikeSeviyesi = TehditAlgıla(MevcutVeriler)
    
    // 3. Tehdit Seviyesine Göre Aksiyon Alma
    
    EĞER YeniTehlikeSeviyesi == 2 İSE // Yüksek Alarm Seviyesi
      
      EĞER AlarmDurumu == KAPALI İSE
        AlarmDurumu = AÇIK
        
        // Alarm Aksiyonları
        SİREN_ÇAL(SESLİ_ALARM)
        
        // Bildirim Gönderme
        TELEFON_BİLDİRİMİ_GÖNDER("Acil Durum! Evinizde hırsızlık/yangın riski algılandı.")
        EPOSTA_GÖNDER("Alarm Durumu: YÜKSEK, Adres: [Adres]")
        
        // Gerekirse Güvenlik Birimini Ara (Opsiyonel)
        CAGRI_MERKEZİ_ARA("Polis/İtfaiye") 
      SON_EĞER
      
    DEĞİLSE EĞER YeniTehlikeSeviyesi == 1 İSE // Uyarı Seviyesi
      
      EĞER AlarmDurumu == KAPALI İSE
        // Sessiz veya düşük seviyeli uyarılar
        TELEFON_BİLDİRİMİ_GÖNDER("Uyarı! Evde sıra dışı bir durum (örn: bilinmeyen ses) algılandı.")
        YEREL_IŞIK_YAK(KIRMIZI)
        
        // AlarmDurumu'nu Yüksek seviyede tutmaya gerek yok, sadece uyarı verilir
      SON_EĞER
      
    DEĞİLSE // Tehlike Seviyesi 0 (Güvenli)
      
      // Sistem Alarmdayken Güvenli Duruma Geçilmez, önce manuel sıfırlama beklenir
      EĞER AlarmDurumu == KAPALI İSE 
        // Normal çalışma durumu, herhangi bir aksiyon yok
      SON_EĞER
      
    SON_EĞER // Tehdit seviyesi kontrolü sonu

    // 4. Alarm Sıfırlama Mekanizması
    // Alarm aktifken, sıfırlama komutu kontrol edilir (Bu komut harici bir kaynaktan, örn: mobil uygulama)
    EĞER AlarmDurumu == AÇIK İSE
      
      SifirlamaKomutu = HARICI_KOMUT_AL("Sifirla") 
      
      EĞER SifirlamaKomutu == ALINDI İSE
        AlarmDurumu = KAPALI
        SİREN_DURDUR()
        TELEFON_BİLDİRİMİ_GÖNDER("Alarm manuel olarak sıfırlandı.")
        EKRAN_GÖSTER("Sistem normale döndü.")
      SON_EĞER
    SON_EĞER
    
    // Enerji tasarrufu veya sistem kararlılığı için kısa bekleme süresi
    BEKLE(5_SANİYE)
    
  DÖNGÜ_SONU // Sistem Aktif olduğu sürece tekrar eder

SON
