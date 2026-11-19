# Uygulama açılışı: UIWindow, AppDelegate / SceneDelegate, açılış sırası

Deney: [`deney/AppLaunchLab`](deney/AppLaunchLab) · Lab tarihi: 19 Kasım 2026

## Ne?

**AppDelegate** uygulamanın genel işleriyle ilgilenir: açılışta servisleri başlatmak (Firebase,
bildirimler), yeni sahne için ayar seçmek (`configurationForConnecting`). Uygulamada bir tane vardır.

**SceneDelegate** her sahnenin (scene, uygulamanın bir arayüz örneği; iPad'de birden fazla olabilir)
yaşam döngüsünü yönetir: pencereyi kurmak (`willConnectTo`), ön plana ve arka plana geçmek.

**UIWindow** ekrandaki her şeyin içinde durduğu en dış kutudur. Kendi başına boştur ve bir sahneye
aittir. İçinde gösterilen ilk ekran **`rootViewController`**'dır: storyboard varken onu iOS koyar,
yoksa biz `willConnectTo`'da koyarız.

Açılış sırası:
`didFinishLaunching` → (`configurationForConnecting`) → `willConnectTo` → `viewDidLoad` →
`sceneWillEnterForeground` → `sceneDidBecomeActive`

- `configurationForConnecting` sadece yeni session'da çağrılır; iOS session'ı saklar.
- Pencereyi kendim kurunca `viewDidLoad`, `willConnectTo`'nun içinde (`makeKeyAndVisible()`
  satırında) çalışır.
- Pencereyi `self.window`'a atamazsam metot içindeki geçici sabitte kalır. Metot bitince tutan
  kimse kalmadığı için silinir ve ekran siyah olur.
- App switcher'dan kapatınca hiçbir metot çalışmaz. Kaydedilecek veri `sceneDidEnterBackground`'da
  kaydedilir.

## Neden var, hangi sorunu çözüyor?

iOS 12 ve öncesinde her şey `AppDelegate`'teydi (örneğin `applicationDidEnterBackground`).
Uygulamanın tek arayüzü olduğu için bu yeterliydi.

iOS 13 ile iPad'de aynı uygulamanın birden fazla penceresi açılabilir hâle geldi. Tek bir delege
olsaydı "arka plana geçti" metodu hangi pencere için çalıştığını bilemezdi: bir pencereyi kapatınca
diğeri de arka plana gitmiş gibi olurdu.

Çözüm iş bölümü: **uygulama** seviyesi işler `AppDelegate`'te kaldı (bir tane), **her pencereye**
(scene) ait işler `SceneDelegate`'e taşındı (scene başına bir tane). Böylece her sahnenin kendi ön
plan ve arka plan durumu var.

## Alternatif ve zayıf yan

**Alternatif 1: Storyboard ile başlangıç** (Info.plist'te `Storyboard Name`). Pencereyi iOS kurar,
az kod gerekir.
Zayıf yanı: storyboard büyük bir XML dosyasıdır; ekipte aynı dosyada çalışılınca git conflict'i
çözmek çok zordur. Ekranı storyboard kendisi oluşturduğu için ViewModel'i init'te veremezsin,
sonradan özelliğe atarsın; unutursan derleyici uyarmaz.

**Alternatif 2: Programatik başlangıç** (`willConnectTo`'da `UIWindow` kurmak). Ekranı kimin, nasıl
oluşturduğu tamamen elimde, Coordinator ile uyumlu.
Zayıf yanı: hatayı derleyici yakalamaz. `self.window`'a atamayı unutmak ya da yanlış sınıfı koymak
sessizce siyah ekran verir.

**Alternatif 3: SwiftUI `App` yaşam döngüsü** (`@main struct MyApp: App` + `WindowGroup`). Delege
dosyası olmadan pencere kurulur. UIKit projesinde kullanmıyoruz.

## Projede nerede

_(SubTrack `chore/programmatic-root` görevinden sonra doldurulacak: `SceneDelegate.swift:satır`)_

## Mülakat sorusu ve benim cevabım

**S: AppDelegate ile SceneDelegate arasındaki fark nedir? Neden ikisi birden var?**

**Ne?** `AppDelegate`, uygulama açıldığında ilk çalışan sınıftır ve uygulama seviyesindeki işleri
yönetir: Firebase, veritabanı gibi servislerin kurulumu ve yeni bir sahne için hangi ayarın
kullanılacağı (`configurationForConnecting`). Uygulamada bir tane vardır. `SceneDelegate` ise her
sahnenin yaşam döngüsünü yöneten sınıftır: pencereyi kurar, ön plana / arka plana geçişi, aktif /
pasif olmayı yönetir. Her sahne için bir tane oluşur.

**Neden?** iOS 13'ten önce `SceneDelegate` yoktu, hepsi `AppDelegate`'teydi. iOS 13 ile iPad'de
aynı uygulama iki pencerede açılabilir oldu, mesela ekranı ikiye bölüp iki Notlar penceresi. Tek
delege olsaydı `didEnterBackground`, `didBecomeActive` gibi metotlar iki pencere için ortak
çalışırdı, hangisinin arka plana gittiği ayırt edilemezdi. Bu yüzden ikiye ayrıldı.

**Örnek:** Bunu denedim: her metoda print koyup uygulamayı arka plana aldım. Sadece
`SceneDelegate`'ten print geldi, `AppDelegate`'ten hiçbir şey çıkmadı.

**Bağlantı:** SubTrack'te storyboard'u kaldırıp pencereyi `SceneDelegate`'te kendim kuruyorum. İlk
ekranı Coordinator oluşturup ViewModel'i init'te veriyor; storyboard ekranı kendisi oluşturduğu
için bunu yapamazdım.
