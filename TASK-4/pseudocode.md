BAŞLA

  // --- 1. Giriş ve Kimlik Doğrulama ---
  FONKSIYON KimlikDogrula(OgrenciNo, Sifre) { ... }
  
  DÖNGÜ_BAŞLAT: Giriş
    OgrenciNo = KULLANICIDAN_AL("Öğrenci Numarası:")
    Sifre = KULLANICIDAN_AL("Şifre:")
    
    EĞER KimlikDogrula(OgrenciNo, Sifre) == YANLIŞ İSE
      EKRAN_GÖSTER: "Hatalı giriş. Tekrar deneyin."
      TEKRAR_ET: Giriş
    SON_EĞER
  DÖNGÜ_SONU: Giriş
  
  // Borç Kontrolü (Ders kaydı yapabilmek için)
  EĞER VERI_TABANI_SORGUSU: BorcVarMi(OgrenciNo) İSE
    EKRAN_GÖSTER: "Ders kaydı yapabilmek için borcunuzu ödemeniz gerekmektedir."
    Sistem_SON
  SON_EĞER
  
  // --- 2. Ders Listesi ve Seçimi ---
  
  MevcutDersler = VERI_TABANI_SORGUSU: OgrDersListesiGetir(OgrenciNo) // Tüm potansiyel dersler
  SecilenDersler = BOŞ_LİSTE
  MaksKredi = VERI_TABANI_SORGUSU: MaksimumKrediGetir(OgrenciNo)
  MinKredi = VERI_TABANI_SORGUSU: MinimumKrediGetir(OgrenciNo)
  
  EKRAN_GÖSTER: "Ders Seçim Ekranı"
  
  DÖNGÜ_BAŞLAT: DersSecim
    EKRAN_GÖSTER: MevcutDersler
    Secim = KULLANICIDAN_SEÇ("Ders Kodu girin (veya 'BİTİR'):")
    
    EĞER Secim == "BİTİR" İSE
      DÖNGÜDEN_ÇIK: DersSecim
    SON_EĞER
    
    // Ders Bilgilerini Al
    DersBilgisi = VERI_TABANI_SORGUSU: DersDetayGetir(Secim)
    
    // --- Kontroller (Ders Ekleme Öncesi) ---
    Kontrol_Gecti = DOĞRU

    // KONTROL A: Kontenjan Kontrolü
    EĞER DersBilgisi.MevcutKontenjan <= 0 İSE
      EKRAN_GÖSTER: "HATA: Dersin kontenjanı dolmuştur."
      Kontrol_Gecti = YANLIŞ
    SON_EĞER

    EĞER Kontrol_Gecti == DOĞRU İSE
      // KONTROL B: Ön Koşul Kontrolü
      EĞER DersBilgisi.OnKosulVar İSE
        OnKosulBasarili = VERI_TABANI_SORGUSU: OnKosuluGectiMi(OgrenciNo, DersBilgisi.OnKosulKodu)
        EĞER OnKosulBasarili == YANLIŞ İSE
          EKRAN_GÖSTER: "HATA: Ön koşul dersi olan " + DersBilgisi.OnKosulKodu + " başarıyla tamamlanmamıştır."
          Kontrol_Gecti = YANLIŞ
        SON_EĞER
      SON_EĞER
    SON_EĞER
    
    EĞER Kontrol_Gecti == DOĞRU İSE
      // KONTROL C: Zaman Çakışması Kontrolü (İÇ İÇE DÖNGÜ)
      DÖNGÜ_BAŞLAT: ÇakışmaKontrolu (SecilenDersler listesindeki HerDers İçin)
        EĞER DersBilgisi.Zaman == HerDers.Zaman İSE
          EKRAN_GÖSTER: "HATA: Seçilen ders, " + HerDers.Adı + " dersi ile zaman çakışması yapmaktadır."
          Kontrol_Gecti = YANLIŞ
          DÖNGÜDEN_ÇIK: ÇakışmaKontrolu
        SON_EĞER
      DÖNGÜ_SONU: ÇakışmaKontrolu
    SON_EĞER
    
    // Ders Ekleme
    EĞER Kontrol_Gecti == DOĞRU İSE
      LİSTEYE_EKLE(SecilenDersler, DersBilgisi)
      EKRAN_GÖSTER: DersBilgisi.Adı + " başarıyla eklendi."
      VERI_TABANI_GÜNCELLE: KontenjanAzalt(DersBilgisi.Kodu) // Kontenjanı 1 azalt
    SON_EĞER
    
    KULLANICIDAN_BEKLE()
    
  DÖNGÜ_SONU: DersSecim

  // --- 3. Genel Kontroller (Ders Seçimi Sonrası) ---
  
  ToplamKredi = HESAPLA: SecilenDerslerinKrediToplami(SecilenDersler)
  
  // KONTROL D: Kredi Limiti Kontrolü
  EĞER ToplamKredi < MinKredi VEYA ToplamKredi > MaksKredi İSE
    EKRAN_GÖSTER: "UYARI: Toplam krediniz (" + ToplamKredi + ") izin verilen limitlerin dışındadır. (" + MinKredi + " - " + MaksKredi + ")"
    EKRAN_GÖSTER: "Lütfen ders listenizi düzenleyin."
    // Bu aşamada öğrenciyi tekrar DersSecim döngüsüne yönlendirmek mantıklıdır.
    GIT: DersSecim // Basitlik için doğrudan yönlendirme varsayılmıştır
  SON_EĞER

  // KONTROL E: Zorunlu/Alttan Ders Kontrolü
  ZorunluEksik = VERI_TABANI_SORGUSU: EksikZorunluDersVarMi(OgrenciNo, SecilenDersler)
  EĞER ZorunluEksik İSE
    EKRAN_GÖSTER: "HATA: Almanız zorunlu olan bazı dersleri seçmediniz. Lütfen kontrol edin."
    GIT: DersSecim // Tekrar seçime yönlendir
  SON_EĞER
  
  // --- 4. Onaylama ve Kesinleştirme ---
  
  EKRAN_GÖSTER: "Tebrikler! Ders seçiminiz kurallara uygundur."
  Onay = KULLANICIDAN_ONAY_AL("Ders seçiminizi danışman onayına göndermek istiyor musunuz? (EVET/HAYIR)")
  
  EĞER Onay == EVET İSE
    VERI_TABANI_KAYDET: DersSecimiKaydet(OgrenciNo, SecilenDersler, Durum="Danışman Onayı Bekliyor")
    EKRAN_GÖSTER: "Ders seçiminiz başarıyla danışmanınızın onayına sunulmuştur."
    EKRAN_GÖSTER: "Danışman onayını takip etmeyi unutmayın."
    
    // --- 5. Danışman Onayı Süreci (Pasif Kontrol) ---
    // (Bu süreç, öğrenci arayüzünden bağımsız, danışman arayüzünde gerçekleşir)
    // EĞER DanışmanSistemeGirdi VE DanışmanOnayladı İSE
    //     VERI_TABANI_GÜNCELLE: DersSecimDurum(OgrenciNo, Durum="ONAYLANDI")
    // DEĞİLSE EĞER DanışmanReddetti İSE
    //     VERI_TABANI_GÜNCELLE: DersSecimDurum(OgrenciNo, Durum="REDDEDİLDİ")
    //     OgrenciyeBildirimGonder()
    // SON_EĞER
    
  DEĞİLSE
    EKRAN_GÖSTER: "Ders seçiminiz taslak olarak kaydedildi. Lütfen süre bitmeden onaylayın."
    VERI_TABANI_KAYDET: DersSecimiKaydet(OgrenciNo, SecilenDersler, Durum="Taslak")
  SON_EĞER

Sistem_SON
