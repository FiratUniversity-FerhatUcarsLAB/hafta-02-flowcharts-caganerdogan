1. Veri Yapıları

// Temel Nesneler
YAPI Urun {
    ID: Tamsayı
    Ad: Metin
    Fiyat: Ondalık Sayı
    MevcutStok: Tamsayı
    YenidenStoklanmaTarihi: Tarih?
    BenzerUrunIDleri: Tamsayı listesi
}

YAPI SepetKalemi {
    UrunID: Tamsayı
    Miktar: Tamsayı
}

YAPI Indirim {
    Kod: Metin
    Tip: ('Yüzde', 'Sabit Miktar')
    Deger: Ondalık Sayı
    MinimumSepetTutari: Ondalık Sayı
}

YAPI AlisverisSepeti {
    KullaniciID: Tamsayı
    Kalemler: SepetKalemi listesi
    UygulananIndirimTutari: Ondalık Sayı
    UygulananIndirimKodu: Metin?
}

2. Yardımcı Fonksiyonlar

// Veritabanından verileri çektiğimizi varsayan yer tutucu fonksiyonlar
FONKSIYON VeritabanindanUrunAl(UrunID): Urun
FONKSIYON KargoUcretiHesapla(Sepet): Ondalık Sayı
FONKSIYON IndirimVeritabanindanBul(Kod): Indirim
FONKSIYON VeritabaninaKaydet(KullaniciID, Veri)
FONKSIYON VeritabanindanCek(KullaniciID): SerilestirilmisVeri

// A: Stok Dışı Bilgilendirme ve Öneri Üretme
FONKSIYON UrunBilgisiUret(UrunID: Tamsayı): Metin
    Urun = VeritabanindanUrunAl(UrunID)
    Cikti = ""

    EĞER Urun.MevcutStok <= 0:
        Cikti.Ekle(Urun.Ad + " tükenmiştir. ")

        EĞER Urun.YenidenStoklanmaTarihi VARSA:
            Cikti.Ekle("Tahmini stok yenilenme: " + Urun.YenidenStoklanmaTarihi + ". ")
        
        EĞER Urun.BenzerUrunIDleri BOŞ DEĞİLSE:
            Cikti.Ekle("Alternatifler: ")
            HER BIR BenzerID IÇIN Urun.BenzerUrunIDleri:
                BenzerUrun = VeritabanindanUrunAl(BenzerID)
                Cikti.Ekle(BenzerUrun.Ad + " - " + BenzerUrun.Fiyat + " | ")
                
    DÖN Cikti
BITIR FONKSIYON

3. Kalıcılık (Persistence) İşlemleri

FONKSIYON SepetiKaydet(Sepet: AlisverisSepeti):
    VeritabaninaKaydet(Sepet.KullaniciID, Sepet.Serilestir())
BITIR FONKSIYON

FONKSIYON SepetiYukle(KullaniciID: Tamsayı): AlisverisSepeti
    Veri = VeritabanindanCek(KullaniciID)
    EĞER Veri VARSA:
        DÖN Veri.GeriCoz()
    DEĞİLSE:
        DÖN Yeni AlisverisSepeti(KullaniciID: KullaniciID)
BITIR FONKSIYON

4. Çekirdek Sepet ve Hesaplama İşlemleri

// B: Ürün Ekleme (Stok Kontrollü)
FONKSIYON UrunEkle(Sepet: AlisverisSepeti, UrunID: Tamsayı, Miktar: Tamsayı):
    Urun = VeritabanindanUrunAl(UrunID)

    // Mevcut sepet miktarını bul ve yeni miktarla topla
    BULUNAN_KALEM = Sepet.Kalemler.Bul(UrunID = UrunID)
    MevcutMiktar = EĞER BULUNAN_KALEM VARSA: BULUNAN_KALEM.Miktar DEĞİLSE: 0
    GerekliToplamMiktar = Miktar + MevcutMiktar

    // Kritik Stok Kontrolü
    EĞER GerekliToplamMiktar > Urun.MevcutStok:
        YAZDIR HATA("Yetersiz Stok. İstek: " + GerekliToplamMiktar + ", Mevcut: " + Urun.MevcutStok)
        YAZDIR UrunBilgisiUret(UrunID) // Bilgilendirme ve öneri göster
        DÖN Sepet
        
    // Sepet Güncelleme
    EĞER BULUNAN_KALEM VARSA:
        BULUNAN_KALEM.Miktar = GerekliToplamMiktar
    DEĞİLSE:
        YENI_KALEM = Yeni SepetKalemi(UrunID: UrunID, Miktar: Miktar)
        Sepet.Kalemler.Ekle(YENI_KALEM)

    SepetiKaydet(Sepet)
    DÖN Sepet
BITIR FONKSIYON

FONKSIYON SepetToplaminiHesapla(Sepet: AlisverisSepeti): Ondalık Sayı
    ToplamFiyat = 0.0
    HER BIR Kalem IÇIN Sepet.Kalemler:
        Urun = VeritabanindanUrunAl(Kalem.UrunID)
        ToplamFiyat = ToplamFiyat + (Urun.Fiyat * Kalem.Miktar)
    DÖN ToplamFiyat
BITIR FONKSIYON

// C: İndirim Uygulama
FONKSIYON IndirimUygula(Sepet: AlisverisSepeti, IndirimKodu: Metin): BASARI/HATA
    Indirim = IndirimVeritabanindanBul(IndirimKodu)
    EĞER Indirim YOKSA DÖN HATA("Geçersiz Kupon Kodu")

    SepetTutari = SepetToplaminiHesapla(Sepet)
    EĞER SepetTutari < Indirim.MinimumSepetTutari DÖN HATA("Minimum sepet tutarı karşılanmadı")

    IndirimTutari = EĞER Indirim.Tip == 'Yüzde':
        SepetTutari * (Indirim.Deger / 100.0)
    DEĞİLSE:
        Indirim.Deger
    
    Sepet.UygulananIndirimTutari = IndirimTutari
    Sepet.UygulananIndirimKodu = IndirimKodu
    SepetiKaydet(Sepet)
    DÖN BASARI
BITIR FONKSIYON

// D: Nihai Fiyat Hesaplama
FONKSIYON NihaiToplamHesapla(Sepet: AlisverisSepeti): Ondalık Sayı
    AraToplam = SepetToplaminiHesapla(Sepet)
    KargoUcreti = KargoUcretiHesapla(Sepet)
    
    NihaiToplam = AraToplam - Sepet.UygulananIndirimTutari + KargoUcreti
    
    DÖN NihaiToplam
BITIR FONKSIYON
