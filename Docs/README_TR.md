# SL-CustomObjects Geliştirici Dokümantasyonu (Türkçe)

**SL-CustomObjects** projesinin resmi geliştirici dokümanına hoş geldiniz. Bu araç, *SCP: Secret Laboratory* oyunu için hazırlanan **MapEditorReborn (MER)** (LabAPI ve EXILED sürümleri) eklentisine Unity tarafında özel harita şablonları tasarlamanızı ve dışa aktarmanızı sağlar.

---

## 📖 Mimari ve Tasarım Genel Bakış

SL-CustomObjects; Unity içinde oluşturduğunuz odaları, yapıları ve etkileşimli nesne gruplarını sunucu tarafının okuyabileceği JSON şablonlarına dönüştürür.

```
+---------------------------------------+
|              Unity Editor             |
|    (Hiyerarşik Tasarımların Çizimi)    |
+---------------------------------------+
                    |
                    v (Compile Schematic - Şablon Derleme)
+---------------------------------------+
|         JSON Serileştirme Dosyaları    |
|    - [Ad].json (Genel Nesneler)       |
|    - [Ad]-Rigidbodies.json (Fizik)    |
|    - [Ad]-Teleports.json (Işınlanma)  |
+---------------------------------------+
                    |
                    v (Sunucuya Aktarma)
+---------------------------------------+
|          SCP: SL Oyun Sunucusu        |
|     (MapEditorReborn ile Oluşturma)   |
+---------------------------------------+
```

---

## 🛠️ Kod Yapısı ve Dosya Açıklamaları

### 1. Temel Sınıflar

#### 🔹 [Schematic.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Schematic.cs)
Her şablon yapısının kök (root) nesnesinde bulunması gereken ana bileşendir. Derleme süreçlerini yönetir.
* **`CompileSchematic()`**: Altındaki tüm `SchematicBlock` bileşenlerini tarar, nesne hiyerarşisini listeler, animasyonları AssetBundle olarak derler, fizik/ışınlanma verilerini ayırır ve JSON çıktısı üretir. Ayarlarda aktifse tüm bunları `.zip` dosyası haline getirir.
* **`Update()`**: Kök nesnenin boyutunun (scale) değiştirilmesini engeller ve klasör isminde boşluk olmamasını denetler.

#### 🔹 [SchematicBlock.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/SchematicBlock.cs)
Diğer tüm nesnelerin türediği soyut (abstract) temel sınıftır. Unity'nin `MonoBehaviour` sınıfından miras alır.
* **`Compile(SchematicBlockData block)`**: Nesnenin yerel pozisyon, yönelim (Euler açıları), boyut (Scale) ve `Static` olup olmadığı gibi temel nitelikleri JSON formatına hazırlar.
* **`Decompile(...)`**: Kaydedilmiş bir JSON dosyasını okuyarak Unity sahnesinde hiyerarşiyi otomatik olarak geri inşa eder.

---

### 2. Nesne Bileşenleri (`Assets/DONT TOUCH/Scripts/BlockComponents/`)

Unity sahnenizde oluşturduğunuz nesnelere atanan ve onların oyundaki işlevlerini belirleyen sınıflar:

* **[PrimitiveComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/PrimitiveComponent.cs)**: Unity temel şekillerini (Küp, Küre, Silindir, Düzlem vb.) sunucu tarafı ilkel nesnelerine dönüştürür. Çarpışma (collision) ve renk/materyal verilerini saklar.
* **[LightComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/LightComponent.cs)**: Oyunda ışık (Directional, Point, Spot, Area) oluşturulmasını sağlar. Yoğunluk, renk ve gölge özelliklerini yönetir.
* **[TeleportComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/TeleportComponent.cs)**: Oyuncuları veya nesneleri belirli koordinatlara ya da ID'lere ışınlayan mekanikleri tanımlar.
* **[TextComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/TextComponent.cs)**: Oyun içinde havada asılı duran 3D yazılar (text toys) oluşturur. Boyut, içerik ve renk ayarlarını barındırır.
* **[InteractableComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/InteractableComponent.cs)**: Haritada oyuncuların etkileşime girebileceği buton veya tetikleyiciler oluşturur. E tuşuyla tetiklenen animasyon geçişleri sunar (bir hedef `Animator` bileşeni bağlayıp oynatılacak durum adlarını belirleyerek, ardışık etkileşimlerde iki farklı animasyon durumu arasında geçiş yapabilir).
* **[WaypointComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/WaypointComponent.cs)**: Rotalama, harita işaretleri veya devriyeler için yönlendirme düğümleri tanımlar.
* **[WorkstationComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/WorkstationComponent.cs)**: Silah eklentilerini değiştirmeye yarayan çalışma masalarını oluşturur.
* **[PickupComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/PickupComponent.cs)**: Kart, silah, cephane gibi eşyaların (pickups) doğma noktalarını belirler. Sınırsız alma, eklentili silah doğurma gibi gelişmiş ayarları vardır.
* **[LockerComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/Locker/LockerComponent.cs)**: Haritadaki dolap ve sandıkların içerdiği ganimet havuzlarını ve fiziksel yapılarını yönetir (`LockerChamber.cs` ve `LockerItem.cs` ile birlikte).
* **[CameraComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Scripts/CameraComponent.cs)**: SCP-079 kameralarını konumlandırmanızı sağlar.
* **[EmptyComponent.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/BlockComponents/EmptyComponent.cs)**: Nesneleri gruplandırmaya yarayan boş oyun objeleridir.

---

### 3. Editör Eklentileri (`Assets/DONT TOUCH/Scripts/Editors/`)

* **[RightClickMenuExtended.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Editors/RightClickMenuExtended.cs)**: Unity'de hiyerarşi kısmına sağ tıkladığınızda çıkan **"🛠️ MER Blocks"** menüsünü oluşturur. Objeleri bu menü aracılığıyla oluşturmak en doğru yöntemdir.
* **[SchematicEditor.cs](file:///home/flaouve/Projects/scp%20sl/Map/Fla-SL-CustomObjects/Assets/DONT%20TOUCH/Scripts/Editors/SchematicEditor.cs)**: `Schematic` bileşeninin arayüzünü (Inspector) özelleştirerek "Compile Schematic" butonunu ekler.

---

## ⚙️ Şablon Derleme Adımları

1. Unity projenizi açın.
2. Sahnenizde hiyerarşide sağ tıklayarak **🛠️ MER Blocks** menüsünden nesneler oluşturup haritanızı tasarlayın.
3. Tasarladığınız tüm objelerin tek bir üst nesnenin (kök) altında olmasına dikkat edin. Bu kök nesnede `Schematic` bileşeni yer almalıdır.
4. Kök nesneyi seçip Inspector panelinden **Compile Schematic** butonuna tıklayın.
5. Derlenen çıktıları (JSON ve varsa AssetBundle dosyalarını) dışa aktarma klasörünüzde (varsayılan olarak Masaüstünde) bulabilirsiniz.
6. Bu dosyaları oyun sunucunuzda aşağıdaki klasöre kopyalayın:
   `LabAPI/configs/ProjectMER/Schematics/[ŞablonAdınız]/`
