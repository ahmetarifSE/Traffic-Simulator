# Traffic-Simulator
# Unity Trafik Kavşağı Simülasyonu

## Amaç ve Sebepler
Bu projenin temel amacı, gerçek dünyadaki bir kavşak senaryosunu yazılım tasarım desenleriyle modellemek ve esnek, genişletilebilir bir trafik simülasyonu oluşturmaktır.

### Neden Waypoint Sistemi?
- **Kavşağın üstünden geçmek yerine etrafından dolanmak:**
  - Araçların kavşağı gerçekçi şekilde kullanmasını sağlamak için, doğrudan karşıya geçmek yerine kavşağın etrafından, belirli yol noktalarını (waypoint) takip ederek ilerlemeleri hedeflendi.
  - Bu sayede araçlar, kavşak ortasında çakışmadan, kontrollü ve düzenli bir şekilde yön değiştirebiliyor.

- **Genişletilebilirlik:**
  - Farklı yol tipleri, kavşaklar veya çoklu güzergahlar kolayca eklenebilir.

### Waypoint Tasarımında Dikkat Edilenler
- **Her yol için ayrı waypoint dizileri tanımlandı.**
- **Araçlar waypointler arasında sıralı olarak ilerler,** bir noktaya ulaştığında bir sonrakine yönelir.
- **Waypointler, yolun ortasında veya kenarında olacak şekilde sahnede dikkatlice yerleştirildi.**
- **Gerekirse ileride trafik yoğunluğu, şerit değiştirme veya yavaşlama gibi gelişmiş davranışlar için waypoint sistemi kolayca genişletilebilir.**

---

## Kullanılan Tasarım Desenleri ve Uygulama Detayları

### **State Pattern**
- **Nerede Kullanıldı?**
  - Trafik ışıklarının (TrafficLight) durum yönetiminde kullanıldı.
  - Her ışık, GreenState, YellowState, RedState ve FlashingYellowState gibi farklı durum nesneleriyle temsil edilir.
- **Nasıl Çalışır?**
  - Işıklar, mevcut durumlarını bir state nesnesiyle tutar ve durum değişimlerinde ilgili state’in `Enter/Exit` metotları çağrılır.
  - Durumlar arası geçişler, TrafficLightManager tarafından yönetilir.

### **Singleton Pattern**
- **Nerede Kullanıldı?**
  - `TrafficLightManager` ve `VehicleManager` sınıflarında kullanıldı.
- **Nasıl Çalışır?**
  - Bu sınıflardan sahnede yalnızca bir tane bulunur ve `Instance` property’si ile global erişim sağlanır.
  - Tüm trafik ışıkları ve araçlar merkezi olarak bu yöneticiler tarafından kontrol edilir.

### **Strategy Pattern**
- **Nerede Kullanıldı?**
  - Trafik ışıklarının yeşil sürelerini belirlemede kullanıldı.
  - `ITrafficStrategy` arayüzü ile `NormalStrategy` (sabit süre) ve `AdaptiveStrategy` (araç yoğunluğuna göre dinamik süre) uygulanır.
- **Nasıl Çalışır?**
  - Her kavşak, hangi stratejiyi kullanacağını ayarlardan (ScriptableObject) seçer.
  - Strateji, ışık geçişlerinde süre hesaplamasında devreye girer.

### **Command Pattern**
- **Nerede Kullanıldı?**
  - Yaya geçiş taleplerinin yönetiminde kullanıldı.
  - Her yaya geçiş talebi bir `PedestrianRequestCommand` nesnesi olarak kuyruğa alınır.
- **Nasıl Çalışır?**
  - Komutlar, uygun fazda sırayla işlenir ve ilgili ışık grubunda kısa süreli yaya fazı başlatılır.

---

## Nasıl Yaptık?

### **Trafik Işıkları ve Yönetimi**
- Her trafik ışığı bir `TrafficLight` scripti ile yönetiliyor. Durumlar (yeşil, sarı, kırmızı, flashing yellow) State Pattern ile ayrı sınıflar olarak tanımlandı.
- Tüm ışıkların kontrolü ve senkronizasyonu merkezi `TrafficLightManager` (Singleton) tarafından sağlanıyor.
- Işık grupları (kuzey-güney, doğu-batı) Inspector üzerinden atanıyor ve senkronize şekilde yönetiliyor.

### **Konfigürasyon Sistemi**
- Işık süreleri, strateji tipi ve yaya fazı gibi ayarlar bir `IntersectionSettings` ScriptableObject dosyası ile Inspector’dan yapılandırılıyor.
- Her kavşak için farklı ayar dosyaları oluşturulabiliyor.

### **UI Panelleri ve Dashboard**
- UI, Unity’nin Canvas sistemiyle oluşturuldu. Her ışık için aktif renk, kalan süre ve strateji bilgisi gösteriliyor.
- Simülasyonu başlat/durdur, acil durum ve yaya geçişi butonları UIManager scripti ile yönetiliyor.
- Gözlem panelinde toplam geçen araç ve ortalama bekleme süresi anlık olarak güncelleniyor.
- Simülasyon saati ve gece/gündüz durumu da panelde gösteriliyor.

### **Araç Simülasyonu ve Waypoint Sistemi**
- Araçlar, belirli aralıklarla `VehicleManager` tarafından sahneye spawn ediliyor.
- Her araç, yolunu takip etmek için waypoint dizilerini kullanıyor. Waypointler sahnede boş GameObject olarak yerleştirildi.
- Araçlar, waypointler arasında sıralı olarak ilerliyor ve kavşaktan geçerken ışık durumuna göre durup kalkıyor.
- Araç hareketi için Rigidbody ve raycast ile çarpışma kontrolü sağlanıyor.

### **Yaya Geçişi ve Komut Sistemi**
- Her ışık için UI’da bir yaya geçiş butonu var. Butona basıldığında bir `PedestrianRequestCommand` oluşturulup kuyruğa ekleniyor.
- Komutlar, uygun fazda TrafficLightManager tarafından işleniyor ve ilgili yönde kısa süreli yaya fazı başlatılıyor.

### **Adaptif ve Normal Strateji**
- Işıkların yeşil süresi, seçilen stratejiye göre hesaplanıyor. NormalStrategy sabit süre, AdaptiveStrategy ise simüle edilen araç yoğunluğuna göre dinamik süre döndürüyor.
- Strateji seçimi Inspector’daki ScriptableObject üzerinden yapılıyor.

### **Gece Modu**
- Simülasyon saati 22:00 olduğunda tüm ışıklar FlashingYellowState’e geçiyor (yanıp sönen sarı).
- Gece modu aktifken normal trafik döngüsü duruyor, sabah olunca tekrar başlıyor.

### **Trafik Gözlem Paneli**
- Araçlar kavşaktan geçtikçe toplam geçen araç sayısı ve ortalama bekleme süresi VehicleManager tarafından hesaplanıp UI’da gösteriliyor.
- Bu veriler event sistemiyle anlık olarak güncelleniyor.

### **Kod Temizliği ve Genişletilebilirlik**
- Kodda sorumluluklar net ayrıldı, her ana işlev ayrı scriptte toplandı.
- Inspector ve ScriptableObject kullanımı ile parametreler kolayca değiştirilebiliyor.
- Yeni yol, kavşak veya strateji eklemek için mevcut yapıyı genişletmek kolay.

---

## Kullanılan Tasarım Desenleri
- **State Pattern:** Trafik ışığı durumları (Green, Yellow, Red, FlashingYellow)
- **Singleton Pattern:** TrafficLightManager ve VehicleManager
- **Strategy Pattern:** Işık zamanlama stratejileri
- **Command Pattern:** Yaya geçiş talepleri

---

## Kurulum ve Kullanım

1. **Unity ile Açın:**
   - Projeyi Unity Hub veya Unity Editor ile açın (Unity 2021 veya üzeri önerilir).

2. **Sahne Ayarları:**
   - `Assets/Scenes/SampleScene.unity` veya kendi sahnenizi kullanabilirsiniz.

3. **ScriptableObject Ayarları:**
   - `Assets/Settings/` veya ilgili klasördeki `IntersectionSettings` dosyasını düzenleyerek ışık sürelerini ve stratejiyi belirleyin.

4. **Araç ve Yol Ayarları:**
   - Araç prefablarını ve waypoint noktalarını Inspector üzerinden ayarlayın.
   - Spawn/despawn point ve waypoint objelerini sahneye yerleştirin.

5. **Simülasyonu Başlatın:**
   - UI panelinden simülasyonu başlatabilir, durdurabilir, acil durum veya yaya geçişi tetikleyebilirsiniz.

6. **Gece Modu:**
   - Simülasyon saati 22:00 olduğunda tüm ışıklar yanıp sönen sarı moda geçer.

## Kullanılan Assetler:
Evpo Games: Modular Lowpoly Streets (Free)
https://assetstore.unity.com/packages/3d/environments/urban/modular-lowpoly-streets-free-192094

Mena: ARCADE: FREE Racing Car
https://assetstore.unity.com/packages/3d/vehicles/land/arcade-free-racing-car-161085
