FONKSIYON AnaATMIslemi():
    
    // --- Başlangıç ve Kart Okuma ---
    EkranaYaz("Lütfen kartınızı yerleştirin...")
    KartVerisi = KartOkuyucudanAl()
    
    // --- PIN Doğrulama Döngüsü (Güvenlik Kontrolü) ---
    PIN_Hata_Sayaci = 0
    MAKS_PIN_DENEMESI = 3
    
    DOGRU_OLDUGU_SURECE: // Sonsuz döngüden çıkış PIN doğruysa veya deneme bittiyse
        Eger IslemZamanAsimiOldu İse CIK // 30 saniye gibi
        PIN = KullanicidanPINAl()
        
        EGER PINDogrula(KartVerisi.HesapNo, PIN) ESIT DEĞİL Doğru İSE:
            PIN_Hata_Sayaci = PIN_Hata_Sayaci + 1
            
            EGER PIN_Hata_Sayaci >= MAKS_PIN_DENEMESI İSE:
                EkranaYaz("PIN hatalı. Kartınız güvenlik nedeniyle alıkonulmuştur.")
                KartAlikoy()
                CIK // Ana İşlemden Çıkış
            SON
            
            EkranaYaz("Hatalı PIN. Kalan deneme hakkı: " + (MAKS_PIN_DENEMESI - PIN_Hata_Sayaci))
        
        DEGİLSE:
            KES // PIN Doğru, döngüden çık
        SON
        
    SON_DOGRU_OLDUGU_SURECE
    
    // Eğer döngüden çıkıldıysa ama PIN hala doğru değilse, yukarıdaki if bloğu işlemi sonlandırmış demektir.
    EGER PIN_Hata_Sayaci >= MAKS_PIN_DENEMESI İSE CIK

    // --- Hesap Seçimi ve Miktar Girişi (UX Kontrolü) ---
    Eger KartVerisindeBirdenFazlaHesapVar İse:
        EkranaYaz("Lütfen hesap tipinizi seçin: 1) Vadesiz 2) Tasarruf")
        SecilenHesap = KullanicidanSecimiAl()
    DEGİLSE:
        SecilenHesap = KartVerisi.VarsayilanHesap
    SON
    
    EkranaYaz("Lütfen çekmek istediğiniz miktarı girin.")
    CekilecekMiktar = KullanicidanMiktarAl()
    
    // --- Ön Kontroller ---
    EGER ATMKasaKontrol(CekilecekMiktar) ESIT DEĞİL Yeterli İSE:
        EkranaYaz("ATM'de istenen miktarda nakit bulunmamaktadır.")
        KartIadeEt()
        CIK
    SON

    EGER BakiyeKontrol(SecilenHesap, CekilecekMiktar) ESIT DEĞİL Yeterli İSE:
        EkranaYaz("Hesap bakiyesi yetersiz.")
        KartIadeEt()
        CIK
    SON
    
    // Ekleme: Günlük Çekim Limiti Kontrolü de buraya entegre edilebilir.
    
    // --- Fon Transferi (Atomik İşlem) ---
    BASLAT_TRANSACTION: 
        TransferDurumu = FonlariTransferEt(SecilenHesap, CekilecekMiktar)
        
        EGER TransferDurumu ESIT DEĞİL Başarılı İSE:
            GERI_AL_TRANSACTION 
            EkranaYaz("Banka sistemi hatası. Lütfen daha sonra tekrar deneyin.")
            KartIadeEt()
            CIK
        SON

        // Başarılı transferi kaydet
        KayitLoguOlustur(SecilenHesap, CekilecekMiktar, "Para Çekme", "Transfer Başarılı")
    ONAYLA_TRANSACTION: 

    // --- Nakit Dağıtımı ve Hata Yönetimi ---
    EkranaYaz("Nakit hazırlanıyor...")
    
    EGER NakitiDagit(CekilecekMiktar) ESIT DEĞİL Başarılı İSE:
        // Hata Yönetimi: Para çekildi ama verilemedi. Acil geri iade!
        EkranaYaz("ATM Donanım Hatası. Nakit verilemedi.")
        
        // Transferi GERİ ALMA GİRİŞİMİ
        GeriAlDurumu = FonlariGeriIadeEt(SecilenHesap, CekilecekMiktar)
        
        EGER GeriAlDurumu ESIT DEĞİL Başarılı İSE:
            HataLoguOlustur("KRİTİK_HATA: Para Geri İadesi Başarısız. Manuel Müdahale Gerekli", SecilenHesap)
            EkranaYaz("Kritik Hata: Hesabınızdan düşülen miktar için bankanızla iletişime geçin.")
        DEGİLSE:
            EkranaYaz("İşlem iptal edildi. Hesabınıza iade yapılmıştır.")
        SON
        
        KartIadeEt()
        CIK // İşlem Sonlandırıldı
    SON
    
    // --- Başarılı Sonuç ---
    EkranaYaz("Lütfen nakitinizi alın.")
    MakbuzBas(SecilenHesap, CekilecekMiktar)
    
    EkranaYaz("İşlem tamamlandı. Kartınızı almayı unutmayınız.")
    KartIadeEt()

SON_FONKSIYON

// --- Ana Fonksiyonlardan Çağrılan Alt Rutinler (Helper Fonksiyonlar) ---

FONKSIYON FonlariGeriIadeEt(HesapNo, Miktar):
    // Banka API: Düşülen miktarı tekrar hesaba yatırır.
    DONDUR Başarılı VEYA Başarısız
SON_FONKSIYON

FONKSIYON KartAlikoy():
    // ATM donanımına kartı yutması için komut gönder.
    // Durumu sistem loguna işle.
    EkranaYaz("Lütfen bankanızla iletişime geçin.")
SON_FONKSIYON

// Diğer alt rutinler (PINDogrula, BakiyeKontrol, ATMKasaKontrol vb.) değişmeden kalır.
