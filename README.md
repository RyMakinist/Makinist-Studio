# Makinist-Studio
Makinist Studio is a Windows IDE and 2D/3D game engine powered by the Makinist RY programming language. Kopyala
# Makinist Studio — Kullanım Mantığı ve Çalışma Yapısı

## 1. Makinist Studio nedir?

Makinist Studio, **Makinist RY** dili ile program yazmak, çalıştırmak ve hata ayıklamak için kullanılan geliştirme ortamıdır.

Temel kullanım zinciri şöyledir:

```text
Makinist Studio
      ↓
.Ry kaynak kodu
      ↓
MakinistRuntime
      ↓
Lexer / Parser / Resolver / Interpreter
      ↓
Native API
      ↓
Engine
      ↓
OpenGL / Win32
```

Kısaca:

- **Studio** kodu yazdığımız ve projeyi yönettiğimiz arayüzdür.
- **MakinistRuntime.exe** `.Ry` kodunu gerçekten çalıştırır.
- **Core** dili çözümler ve yorumlar.
- **Native API** `.Ry` kodunu C++ motor sistemlerine bağlar.
- **Engine** pencere, grafik, input, sahne, kamera, ışık vb. işlerini yürütür.
- **OpenGL + Win32** en alttaki gerçek platform/render katmanıdır.

---

# 2. Makinist Studio ile normal çalışma sırası

Bir projede genel kullanım sırası:

```text
1. Projeyi aç
2. Source/main.Ry dosyasını düzenle
3. Kaydet
4. Çalıştır
5. Çıktıyı konsoldan izle
6. Hata varsa HATALAR / DEBUG bölümlerini kullan
7. Gerekirse breakpoint koy
8. Düzenle ve tekrar çalıştır
```

Studio'nun amacı test scriptlerini sürekli elle çalıştırmak değildir.

Günlük kullanımda kullanıcı doğrudan:

```text
MakinistStudio.exe
```

üzerinden proje açar, `.Ry` kodunu düzenler ve **Çalıştır** düğmesine basar.

---

# 3. Bir Makinist RY projesi nasıl görünür?

Basit proje yapısı:

```text
BenimProjem\
│
├── project.ryproj
│
├── Source\
│   └── main.Ry
│
└── Assets\
    ├── resim.png
    ├── model.obj
    └── diğer dosyalar
```

## project.ryproj

Bu dosya projenin giriş bilgisini taşır.

Örnek:

```ini
[MakinistRY]
format=1
name=Benim Projem
entry=Source/main.Ry
language=tr
sourceRoot=Source
assetRoot=Assets
```

Anlamları:

```text
name        = proje adı
entry       = ilk çalıştırılacak .Ry dosyası
language    = aktif dil paketi
sourceRoot  = kaynak kod klasörü
assetRoot   = asset klasörü
```

---

# 4. main.Ry nedir?

`main.Ry`, çoğu projede programın başlangıç noktasıdır.

En basit örnek:

```ry
import System

System.print("Merhaba Makinist RY")
```

Türkçe dil paketi kullanıldığında Studio bazı anahtar kelimeleri Türkçe gösterebilir:

```ry
içe_aktar System

System.print("Merhaba Makinist RY")
```

Makinist'in temel mantığı şudur:

> Kaynak kod hangi dil paketi ile yazılırsa yazılsın, çalışma sırasında sistem bunu aynı çekirdek komut/anlam yapısına çözer.

---

# 5. Dil nasıl çalışıyor?

`.Ry` dosyası çalıştırıldığında aşağıdaki aşamalar oluşur:

```text
Kaynak kod
   ↓
Lexer
   ↓
Token
   ↓
Parser
   ↓
AST
   ↓
Resolver
   ↓
Interpreter
   ↓
Native API çağrıları
```

## Lexer

Kodun parçalarını ayırır.

Örneğin:

```ry
kare = kare + 1
```

şu tip parçalara ayrılır:

```text
identifier
=
identifier
+
number
```

## Parser

Bu tokenları program yapısına dönüştürür.

Örneğin:

```ry
eğer skor > 10 {
    System.print("Kazandın")
}
```

bir `if` yapısı olarak AST'ye çevrilir.

## Resolver

Değişkenlerin hangi scope'a ait olduğunu belirler.

Örneğin:

```ry
kare = 0

iken kare < 100 {
    kare = kare + 1
}
```

buradaki iç bloktaki `kare`, dışarıdaki gerçek `kare` değişkenini günceller.

## Interpreter

AST'yi gerçekten çalıştırır.

Şimdiki Makinist RY sürümünde ana çalışma sistemi interpreter tabanlıdır.

---

# 6. import / içe_aktar ne yapar?

API modüllerini programa getirir.

Örnek:

```ry
import System
import Graphics3D
import Input
```

veya Türkçe görünümle:

```ry
içe_aktar System
içe_aktar Graphics3D
içe_aktar Input
```

Bundan sonra ilgili modülün fonksiyonları kullanılabilir:

```ry
System.print("Başladı")

Graphics3D.openWindow(...)

Input.isKeyDown("W")
```

---

# 7. Native API mantığı

`.Ry` doğrudan OpenGL veya Win32 çağırmaz.

Doğru mimari:

```text
.Ry
 ↓
NativeRegistry
 ↓
API
 ↓
Engine Backend
 ↓
Engine
 ↓
Renderer / Input / Window
 ↓
OpenGL / Win32
```

Örnek:

```ry
Graphics3D.setPosition(gemi, 0.0, 0.0, 0.0)
```

`.Ry` tarafında bu sadece bir API çağrısıdır.

Gerçekte aşağıdaki katmanlara gider:

```text
Graphics3D.setPosition
       ↓
Graphics3DAPI
       ↓
EngineGraphics3DBackend
       ↓
Scene3D Entity
       ↓
Transform3D
```

Bu nedenle `.Ry` kodu OpenGL ayrıntılarını bilmez.

---

# 8. Graphics3D nasıl çalışıyor?

Şu an kullanılabilen temel 3D hattı:

```text
Graphics3D
 ├── Window
 ├── OBJ Model
 ├── Texture
 ├── Entity
 ├── Transform3D
 ├── Material3D
 ├── Camera3D
 ├── Lighting3D
 ├── Scene3D
 └── Render
```

Basit örnek:

```ry
import System
import Graphics3D

pencere = Graphics3D.openWindow(960, 540, "3D Test")

eğer pencere {
    model = Graphics3D.loadObj("C:/model.obj")
    gemi = Graphics3D.createEntity(model)

    Graphics3D.setPosition(gemi, 0.0, 0.0, 0.0)
    Graphics3D.setScale(gemi, 1.0, 1.0, 1.0)

    Graphics3D.setCamera(0.0, 2.5, 8.0, 0.0, 0.0, 0.0, 58.0)

    kare = 0

    iken Graphics3D.isOpen() && kare < 400 {
        Graphics3D.setRotation(gemi, 0.0, kare * 1.0, 0.0)
        Graphics3D.render()
        Graphics3D.wait(16)

        kare = kare + 1
    }

    Graphics3D.destroyEntity(gemi)
    Graphics3D.unloadModel(model)
    Graphics3D.close()
}
```

---

# 9. Scene3D mantığı

3D dünyada gördüğümüz her nesne bir `Scene3D Entity` olarak düşünülebilir.

Örnek:

```text
Scene3D
│
├── Player
├── Enemy01
├── Enemy02
├── Floor
├── Wall01
├── Wall02
└── Collectible
```

Her entity şunlara bağlanabilir:

```text
Entity
 ├── Transform3D
 ├── Mesh
 ├── Material3D
 ├── active
 ├── visible
 └── layer
```

`Transform3D`:

```text
position
rotation
scale
```

verilerini taşır.

---

# 10. Model ve texture mantığı

## Model

OBJ dosyası:

```ry
model = Graphics3D.loadObj("C:/Assets/ship.obj")
```

şu hattan geçer:

```text
OBJ
 ↓
ObjModelLoader3D
 ↓
MeshData3D
 ↓
OpenGLMesh
 ↓
GPU
```

## Texture

PNG gibi bir dosya:

```ry
doku = Graphics3D.loadTexture("C:/Assets/ship.png")
```

şu hattan geçer:

```text
PNG
 ↓
WIC Image Loader
 ↓
RGBA8
 ↓
Texture2D
 ↓
GPU
```

Sonra entity'ye bağlanır:

```ry
Graphics3D.setTexture(gemi, doku)
```

---

# 11. Material3D mantığı

Bir nesnenin görünüşünü kontrol eder.

Örnek:

```ry
Graphics3D.setColor(gemi, 80, 170, 255)
Graphics3D.setEmissive(gemi, 3, 10, 25)
Graphics3D.setTexture(gemi, doku)
```

Mantıksal olarak:

```text
Mesh      = şekil
Transform = konum
Material  = görünüş
```

---

# 12. Camera3D mantığı

Örnek:

```ry
Graphics3D.setCamera(
```

Mevcut parser kullanımında fonksiyon çağrılarını aynı fiziksel satırda tutmak daha güvenlidir:

```ry
Graphics3D.setCamera(0.0, 2.5, 8.0, 0.0, 0.0, 0.0, 58.0)
```

Parametre mantığı:

```text
eyeX
eyeY
eyeZ

targetX
targetY
targetZ

FOV
```

Yani:

```text
kamera nerede?
kamera nereye bakıyor?
görüş açısı kaç?
```

---

# 13. Lighting3D mantığı

Ortam ışığı:

```ry
Graphics3D.setAmbientLight(0.12)
```

Directional light:

```ry
Graphics3D.setDirectionalLight(-0.4, -1.0, -0.3, 255, 235, 210, 0.9)
```

Point light:

```ry
Graphics3D.setPointLight(2.0, 2.0, 2.0, 80, 140, 255, 3.0, 8.0)
```

Şu anda temel 3D ışık sistemi:

```text
Ambient
Directional
Point
```

üzerinden çalışmaktadır.

---

# 14. Input nasıl kullanılır?

Önce:

```ry
import Input
```

Sonra:

```ry
wBasili = Input.isKeyDown("W")
aBasili = Input.isKeyDown("A")
sBasili = Input.isKeyDown("S")
dBasili = Input.isKeyDown("D")
escBasili = Input.isKeyDown("Escape")
```

Örnek:

```ry
eğer wBasili {
    oyuncuZ = oyuncuZ - 0.08
}
```

Bu değer daha sonra 3D entity'ye uygulanabilir:

```ry
Graphics3D.setPosition(oyuncu, oyuncuX, 0.0, oyuncuZ)
```

---

# 15. Oyun döngüsü mantığı

Tipik bir Makinist RY oyun döngüsü:

```ry
iken Graphics3D.isOpen() {
    # input oku

    # oyun state güncelle

    # collision kontrol et

    # transformları güncelle

    # kamerayı güncelle

    # ışıkları güncelle

    Graphics3D.render()
    Graphics3D.wait(16)
}
```

Mantık:

```text
INPUT
 ↓
GAME LOGIC
 ↓
PHYSICS / COLLISION
 ↓
TRANSFORM
 ↓
CAMERA / LIGHT
 ↓
RENDER
 ↓
WAIT
 ↓
tekrar
```

---

# 16. Neden Graphics3D.wait(16) kullanıyoruz?

Örnek:

```ry
Graphics3D.wait(16)
```

yaklaşık:

```text
1000 / 16 ≈ 60 FPS
```

hedefler.

Bu gerçek bir profesyonel frame limiter değildir; şu anki temel oyun/runtime akışında CPU'yu gereksiz yere tam yükte çalıştırmamak için kullanılır.

---

# 17. Kaynakları neden kapatıyoruz?

3D program sonunda:

```ry
Graphics3D.clearTexture(gemi)
Graphics3D.destroyEntity(gemi)
Graphics3D.unloadTexture(doku)
Graphics3D.unloadModel(model)
Graphics3D.close()
```

yapılması önerilir.

Genel sıra:

```text
Entity'den texture bağlantısını kaldır
 ↓
Entity'yi yok et
 ↓
Texture'ı boşalt
 ↓
Model'i boşalt
 ↓
Pencereyi kapat
```

Böylece kaynak yaşam süresi temiz kalır.

---

# 18. Studio ekranındaki ana bölümler

## Gezgin

Sol taraftaki proje/dosya ağacıdır.

Buradan proje kaynaklarına ulaşılır.

## Editör

Ortadaki Scintilla tabanlı kod alanıdır.

Şu özellikler vardır:

```text
line numbers
syntax highlighting
çoklu sekme
.Ry kaynak düzenleme
```

## Komutlar / Sözlük

Sağ tarafta Makinist dil komutlarını gösterir.

Türkçe / İngilizce gibi dil paketlerinin karşılıklarını incelemek için kullanılabilir.

## Komuta Konsolu

Alt taraftadır.

Çalıştırılan programın `System.print(...)` çıktıları burada görünür.

---

# 19. Studio'da Çalıştır düğmesi ne yapıyor?

Basitleştirilmiş akış:

```text
Çalıştır
 ↓
aktif projeyi belirle
 ↓
project.ryproj oku
 ↓
entry dosyasını bul
 ↓
MakinistRuntime.exe çalıştır
 ↓
Runtime .Ry kodunu yürütür
 ↓
çıktılar Studio konsoluna gelir
```

3D program ise ayrıca Engine tarafından ayrı oyun/render penceresi açılır.

---

# 20. Derle düğmesi ile Çalıştır arasındaki fark

## Çalıştır

`.Ry` programını Runtime üzerinden çalıştırır.

Günlük geliştirmede çoğu zaman kullanacağımız düğme budur.

## Derle

Studio/Runtime C++ kaynaklarını yeniden derlemek ile `.Ry` proje çalıştırmak aynı şey değildir.

Normal kullanıcı:

```text
.Ry yaz
Kaydet
Çalıştır
```

yapar.

Motor geliştiricisi:

```text
C++ Engine/API/Core değiştir
 ↓
MakinistRuntime.exe yeniden derle
 ↓
MakinistStudio.exe gerekirse yeniden derle
 ↓
.Ry test et
```

yapar.

---

# 21. Debugger mantığı

Studio debugger altyapısı:

```text
Breakpoint
Continue
Step Into
Step Over
Step Out
Variables
Watch
Call Stack
Output
```

akışını destekler.

Genel kullanım:

```text
1. İstenen satıra breakpoint koy
2. Debug/Çalıştır
3. Program breakpoint'te durur
4. Değişkenleri kontrol et
5. Step Over / Step Into kullan
6. Continue ile devam et
```

---

# 22. Çok dillilik nasıl çalışıyor?

Makinist'in önemli özelliklerinden biri kaynak dil paketidir.

Örneğin:

```text
Türkçe      İngilizce

eğer        if
değilse     else
iken        while
fonksiyon   function
döndür      return
```

Studio sağ panelde bu karşılıkları gösterebilir.

Ama motorun içinde amaç:

```text
Türkçe kaynak
İngilizce kaynak
Almanca kaynak
       ↓
aynı Core anlamı
```

şeklinde çalışmaktır.

---

# 23. Şu anda özellikle dikkat edilmesi gereken parser kuralı

Mevcut interpreter sürümünde newline bir statement boundary'dir.

Bu nedenle şunu kullanmamak daha güvenlidir:

```ry
eğer a == 1 &&
     b == 2 {
}
```

Bunun yerine:

```ry
eğer a == 1 && b == 2 {
}
```

kullan.

Aynı şekilde API çağrılarını da şimdilik tek satır tutmak en güvenli yöntemdir:

```ry
Graphics3D.setCamera(0.0, 2.5, 8.0, 0.0, 0.0, 0.0, 58.0)
```

---

# 24. Şu anda çalışan temel sistemler

## Dil

```text
Lexer
Parser
AST
Resolver
Interpreter
variables
assignments
functions
if
while
imports
native calls
```

## Studio

```text
Scintilla editor
syntax highlighting
project open/run
console
debugger
breakpoints
step controls
variables
watch
call stack
```

## 2D

```text
OpenGL Renderer2D
Texture2D
image loading
sprites
sprite batch
Camera2D
shapes
fonts/text
Scene2D
Graphics2D API
```

## 3D

```text
OpenGL 3.3 Core
depth
perspective
Matrix4
Mesh
Transform3D
Camera3D
Material3D
Texture
OBJ Loader
Directional Light
Point Light
Scene3D
SceneRenderer3D
Graphics3D API
```

## Input

```text
keyboard input
WASD
arrow keys
Escape
```

---

# 25. Henüz tamamlanmamış büyük sistemler

Bunlar gelecekteki aşamalardır:

```text
Physics / Rigidbody
gerçek collider component sistemi
collision response solver
gravity / forces
character controller
vehicle physics

Audio engine
Animation / Skeleton
glTF / FBX
PBR
Shadows
Normal Mapping
Particles
AI / Navigation
World Streaming

Bytecode + VM üretim hattı
GC'nin üretim sürümü
Package Manager
tam Language Server
tam IntelliSense
Studio 3D viewport
```

Yani mevcut sistem güçlü bir **çalışan temel motor** durumundadır; henüz tam ticari oyun motoru özellik seti değildir.

---

# 26. En önemli mimari kural

Makinist'te katmanlar birbirine karıştırılmamalıdır.

Doğru:

```text
.Ry
 ↓
API
 ↓
Engine
 ↓
Renderer
 ↓
OpenGL
```

Yanlış:

```text
.Ry
 ↓
direkt glDraw...
```

veya:

```text
Graphics3DAPI.cpp
 ↓
doğrudan OpenGL çağrısı
```

API kullanıcı dilini temsil eder.

Engine motor davranışını temsil eder.

OpenGL ise sadece renderer backend'dir.

Bu ayrım korunmalıdır.

---

# 27. Bir özellik eklerken izlenecek yöntem

Örneğin ileride Rigidbody eklemek istiyoruz.

Doğru sıra:

```text
1. Engine\Physics\Rigidbody oluştur
2. Engine içinde fizik davranışını doğrula
3. Backend-neutral API contract yaz
4. NativeRegistry'ye bağla
5. Physics .Ry API ekle
6. .Ry test yaz
7. Studio'dan fiziksel acceptance yap
8. PASS + LOCK
```

Bu yöntem 2D ve 3D renderer geliştirmesinde kullandığımız yaklaşımın devamıdır.

---

# 28. Makinist Studio'nun kullanım özeti

Normal kullanıcı açısından:

```text
Projeyi aç
 ↓
.Ry kodunu yaz
 ↓
Kaydet
 ↓
Çalıştır
 ↓
Console'u izle
 ↓
Program penceresini kullan
```

Motor geliştiricisi açısından:

```text
Core / API / Engine C++ değiştir
 ↓
Build
 ↓
Regression test
 ↓
.Ry acceptance
 ↓
Studio physical test
 ↓
PASS + LOCK
```

---

# 29. Şu anki Makinist'in kısa tanımı

Bugünkü Makinist RY:

> Çok dilli kaynak yapısını destekleyen, kendi lexer/parser/resolver/interpreter çekirdeği bulunan; Native API üzerinden bağımsız C++ Engine'e bağlanan; Studio içinde düzenlenip çalıştırılabilen; OpenGL tabanlı gerçek 2D ve 3D program/oyun çalıştırabilen bir programlama dili + oyun motoru + IDE temelidir.

---

# 30. Hızlı başlangıç

## Konsol programı

```ry
import System

System.print("Merhaba Makinist")
```

## 3D pencere

```ry
import Graphics3D

pencere = Graphics3D.openWindow(960, 540, "Makinist 3D")

eğer pencere {
    Graphics3D.setClearColor(5, 10, 25)

    kare = 0

    iken Graphics3D.isOpen() && kare < 300 {
        Graphics3D.render()
        Graphics3D.wait(16)

        kare = kare + 1
    }

    Graphics3D.close()
}
```

---

# Son söz

Makinist Studio'da temel çalışma mantığı şudur:

```text
KODU STUDIO'DA YAZ
        ↓
.Ry OLARAK KAYDET
        ↓
RUNTIME ÇALIŞTIRSIN
        ↓
API MOTORLA KONUŞSUN
        ↓
ENGINE İŞİ YAPSIN
        ↓
OPENGL / WINDOWS SONUCU EKRANA GETİRSİN
```

Studio motor değildir.

Runtime dil değildir.

API renderer değildir.

Her katman kendi işini yapar.

Makinist'in büyürken korunması gereken en önemli yapı budur.
