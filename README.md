# Active Directory Güvenlik Laboratuvarı

Windows Server 2022 ve Windows 11 Pro kullanarak kurduğum bu laboratuvarda merkezi kullanıcı yönetimi, grup tabanlı erişim kontrolü, Group Policy ve Windows güvenlik logları üzerine çalıştım.

## Laboratuvar Ortamı

| Bileşen | Yapılandırma |
|---|---|
| Sanallaştırma | Hyper-V |
| Sanal ağ | LAB-AG — Internal |
| Domain | lab.test |
| Domain denetleyicisi ve DNS | DC01 — Windows Server 2022 |
| DC01 IP adresi | 192.168.50.10/24 |
| İstemci | CLIENT01 — Windows 11 Pro |
| CLIENT01 IP adresi | 192.168.50.20/24 |
| İstemci DNS adresi | 192.168.50.10 |

Paylaşım ve domain testleri, LAB-AG iç ağı üzerinde gerçekleştirildi. Windows 11 ilk kurulumu ve güncellemeleri için istemci geçici olarak Default Switch ağına bağlandı.

## Gerçekleştirdiğim Çalışmalar

- Active Directory Domain Services ve DNS kurulumu.
- `lab.test` domaininin oluşturulması.
- Şirket ve departman OU yapısının hazırlanması.
- Standart test kullanıcıları ve Global Security gruplarının oluşturulması.
- CLIENT01'in domaine katılması ve Bilgisayarlar OU'suna taşınması.
- GPO ile 300 saniyelik hareketsizlik sonrası ekran kilidi uygulanması.
- Windows güvenlik günlüklerinde başarılı ve başarısız oturum açma kayıtlarının incelenmesi.
- Departman klasörlerinde NTFS ve paylaşım izinlerinin yapılandırılması.
- İzin verilen ve engellenen erişim senaryolarının test edilmesi.

## Kullanıcı ve Grup Yapısı

| Departman OU'su | Kullanıcı | Güvenlik grubu |
|---|---|---|
| BT | mert.test | GG_BT |
| Muhasebe | ayse.test | GG_Muhasebe |
| InsanKaynaklari | deniz.test | GG_InsanKaynaklari |

Departman OU'ları `Sirket` altında oluşturuldu. CLIENT01 için `Sirket/Bilgisayarlar` OU'su kullanıldı; DC01, Domain Controllers OU'sunda bırakıldı.

## GPO: Otomatik Ekran Kilidi

- **GPO adı:** GPO_Istemci_EkranKilidi
- **Bağlandığı OU:** Sirket/Bilgisayarlar
- **Ayar:** Interactive logon: Machine inactivity limit
- **Değer:** 300 saniye

CLIENT01 üzerinde `gpupdate /force` çalıştırıldı. Yeniden başlatma ve kullanıcı girişi sonrasında, hareketsizlik süresi sonunda ekranın kilitlendiği gözlemlendi.

## Güvenlik Loglarının İncelenmesi

Kontrollü bir yanlış parola denemesi yapıldı ve ardından doğru parola ile giriş gerçekleştirildi.

| Tarih ve saat | Olay kimliği | Hesap | Sonuç |
|---|---|---|---|
| 08.11.2026 01:21:02 | 4625 | LAB\mert.test | Başarısız oturum açma |
| 08.11.2026 01:21:09 | 4624 | LAB\mert.test | Başarılı oturum açma |

Başarısız giriş kaydında:

- **Oturum türü:** 2 — Etkileşimli oturum açma
- **Durum:** 0xC000006D
- **Alt durum:** 0xC000006A — Yanlış parola

Aynı hesabın başarısız girişten 7 saniye sonra başarılı giriş yaptığı doğrulandı. Bu kontrollü test, olayları zaman ve hesap bilgileri üzerinden ilişkilendirme çalışmasıdır; tek başına saldırı tespiti olarak değerlendirilmemiştir.

## Departman Paylaşımları

DC01 üzerinde aşağıdaki klasörler oluşturuldu:

- `C:\Paylasimlar\BT`
- `C:\Paylasimlar\Muhasebe`
- `C:\Paylasimlar\InsanKaynaklari`

Her departman klasöründe:

- Üst klasörden izin kalıtımı kapatılarak mevcut izinler açık izinlere dönüştürüldü.
- Genel `Users` erişimi kaldırıldı.
- İlgili departman grubuna NTFS **Modify** izni verildi.
- SYSTEM ve Administrators izinleri korundu.
- Paylaşım izinlerinden Everyone kaldırıldı.
- İlgili departman grubuna **Change** ve **Read** paylaşım izinleri verildi.

### Erişim Testleri

| Kullanıcı | Paylaşım | Gözlemlenen sonuç |
|---|---|---|
| mert.test | BT | Erişim ve klasör oluşturma başarılı |
| mert.test | Muhasebe | Erişim reddedildi |
| ayse.test | Muhasebe | Erişim ve klasör oluşturma başarılı |
| ayse.test | BT | Erişim reddedildi |
| deniz.test | InsanKaynaklari | Erişim ve klasör oluşturma başarılı |
| deniz.test | BT | E
