
# Yazılım Mühendisliği’nde Over-Engineering ve Lean Thinking Üzerine

“assorted-type hand tool lot” by Ashim D’Silva on Unsplash

**Over-Engineering **(Gereğinden fazla teknik/mühendislik uygulama)** **ve **Lean Thinking **(Yalın Düşünme) kavramları mühendislik gerektiren işler başta olmak üzere, aslında hemen her iş kolunda, hatta belki gündelik yaşantımızda kendine yer edinebilen iki önemli kavram olarak karşımıza çıkmaktalar. Bu yazıda bu iki kavramın yazılım mühendisliği dünyasındaki yerinden bahsetmek istiyorum.

Bir yazılım projesi veya mevcut projedeki bir modül için mimariyi tasarlarken **Over-Engineering’**den kaçarak, Lean Software Development (LSD) kavramının içerdiği prensiplere sadık kalmanın ne gibi artıları olabileceğinden bahsedeceğim.

### Over-Engineering’e Doğru Sürükleniş

Ürün haline gelmiş, nefes alan bir yazılımın değişen müşteri ihtiyaçlarına hızlı ve kaliteli olarak yanıt verebilmesi çok önemli. **Bazen müşteri bile ne istediğini tam olarak bilmediğinden**, bu süreç biraz sancılı ve stresli geçebiliyor.

Biz mühendisler kendimizi Dünya’nın en zeki insanları olarak görüyoruz. (Belki de öyleyiz 😏 ) Bir yazılım mühendisi olarak, yazılımcı ego’su diye bir şeyin var olduğunu kabul ediyorum. İşte bu ego bazen **over-engineering**’e giden yolun ilk adımı olabiliyor. Siz buna zekayı veya bilgiyi ispatlama çabası diyin ben ego tatmini diyeyim, sonuçta ortada çözmemiz gereken bir sorunumuz var.

**Pair-programming** kavramını duymayanınız yoktur. Bu yaklaşımın over-engineering’e doğru gidiş yolunda erken teşhis koyma noktasında fayda sağladığını düşünüyorum. Şöyle ki;

Over-engineering yapmakta olan yazılımcı, yapmakta olduğu yanlışın farkında olmuyor. Yani kimse bile bile yanlış yapmaz değil mi?

**Pair **yaparken over-enginerring’e doğru kaydığınızı hissettiğiniz anda hemen müdahale etmelisiniz. Söz konusu tasarımı ekip arkadaşınızla tartışmaya açmalısınız. (Bu tartışmayı yaparken sormanız gereken soruların ne olacağı ve yaklaşımınızın nasıl olacağıyla alakalı ilerleyen bölümlerde bazı ip uçları vereceğim) Aksi halde kartopu daha da büyüyerek geri dönüşü olmayan bir noktaya varabilir. Yani yol yakınken dönebilmek gerekiyor.

Buraya kadar olan kısımda birçoğumuzun bilip kabul ettiği bir sorundan bahsettik. Peki söz konusu eforun aslında bir over-engineering olduğunu nasıl anlarız? Diğer bir değişle bu yolda yapılan en yaygın hatalar nelerdir? Madde madde anlatmaya çalışayım;

### **Basit Düşünmek Yerine Varsayımlarda Bulunmak**

Aslında temelinde tamamen iyi niyet yatan bu hata çok yaygın olarak yapılmakta. Unutmayalım ki, talep edilmemesine rağmen “İhtiyaç olur” denilerek eklenen her özellik bize development ve bakım maaliyeti olarak geri dönecektir. Üstelik bu özellik hiçbir zaman kullanılmayabilir, harcanan efor tamamen çöp olabilir. Bu hatayı en kısa haliyele “**yanlış veya gereksiz varsayım**” olarak da özetleyebiliriz.

Eskisi kadar olmasa da futbola ilgi duyan birisi olduğumdan ötürü, tam da bu satırları yazarken Hollanda’nın efsane futbolcularında Johan Cruyff’un şu sözü geldi aklıma;

![“Futbol oynamak basittir, zor olan basit futbol oynamaktır.”](https://cdn-images-1.medium.com/max/2000/1*gGDIUr2H4eID4BE0LvZtAA.png)*“Futbol oynamak basittir, zor olan basit futbol oynamaktır.”*

Mesleki bir konuyu açıklarken futboldan alıntı yapacağım hiç aklıma gelmezdi doğrusu. 😄

Bu sözün, yazılım geliştirme yaşam döngüsünün her sürecinde kulağa küpe edilmesi gerektiğini düşünüyorum. Özellikle development sürecinde.Tabi şimdi, “Ne yani, yazılım geliştirme işi basit bir iş mi?” diye düşünebilirsiniz. Cevap vereyim, evet basit :)

**Zor olan şey; bir yazılım problemini çözerken harcanması gereken minimum eforu yakalayabilmek, ve ortaya çıkacak olan yazılımın okunabilir, bakımı kolay, değişime kapalı, gelişime açık olarak çıkması meselesidir.**

Çoğu zaman müşteriden gelen bir isteğin veya iç süreçlerimizi iyileştirmek için geliştirilen bir özelliğin basitçe çözülebilmesi mümkün iken farkında olmadan daha kompleks çözümlere gidebiliyoruz. Hatta bazen sırf teorisini bildiğimizden, ‘**Zamazingo’ Design Pattern**’ ını uyguluyoruz. Sonrada, **“Nasıl uyguladım ama Zamazingo’yu, tam da yerinde kullandım hehe”** diye mutlu oluyoruz, belki de hiç gereği yokken.

Yine bu maddeyle alakalı olan ve literatüre geçen, çoğunuzun malumu**, KISS **(Keep it simple stupid) ve **YAGNI **(You ain’t gonna need it) prensipleri de her zaman aklımızın bir köşesinde tutmamız gereken prensiplerden.

**KISS **için kısaca, gereksiz karmaşıklıktan kaçınma ve basitliğin esas amaç olarak görülmesi prensibi diyebiliriz.

**YAGNI **içinse, bir özelliği/modülü/kontrolü vs. sadece ve sadece gerçekten ihtiyaç duyduğun zaman geliştirmek, dolayısıyla ilerde ihtiyaç duyulur varsayımından kaçınmak olarak tanımlayabiliriz.

### **Yerli Yersiz Generic Type Kullanımı**

**Generic** kullanmanın faydaları üzerinde konuşursak benim ilk aklıma gelenler, bir kodun yeniden kullanılabilirliğini maksimum seviyeye çıkarması, **type-safety** olması ve performansa olan katkısı olur. Tabi her güzel şeyin de bir bedeli oluyor bildiğiniz gibi. **Generic **kullanımı projemizin kod karmaşıklığı artırırken, kod okunabilirliğini de olumsuz yönde etkileyebiliyor.

Kod karmaşıklığı demişken, Go dilini bilen veya gelişmeleri takip edenleriniz bilirler Generic tip desteği yoktur. Go 2 ile birlikte Generic tip konusu gündeme geldiğinde olumlu bakanlar olduğu gibi karşı çıkanlarda da oldu takip ettiğim kadarıyla, ki haksız da sayılmazlar. Yazıyı yazarken Go’nun [**FAQ](https://golang.org/doc/faq) **sayfasına göz atayım dedim. Güncel durum şöyle;
> Generics are convenient but they come at a cost in complexity in the type system and run-time. We haven’t yet found a design that gives value proportionate to the complexity, although we continue to think about it.

Sanırım complexity çok yükseltmeden işi kotarmanın yoluna bakıyorlar, kolay gelsin diyelim 😏

Peki hal böyleyken hiç Generic kullanmayacak mıyız? Elbette kullanacağız. Ancak alet çantamızda ki her alet gibi Generic tipi de yerinde ve verimli kullanmamız gerekiyor. Bazen esas çözmemiz gereken soruna odaklanmak yerine Generic kullanımına kendimizi biraz fazla kaptırıyoruz bence. Nereden mi biliyorum? Mesela kendimden 😅

**Unutmayalım ki; Generic tip ile ekstra efor harcayarak yaptığımız tasarımın ömrünün ne kadar olacağını bilmiyoruz.**

Bir projede gelen analizle birlikte ek bir modül yazmak için kolları sıvadığımda, birbirine benzer ve çok sayıdaki business servis için kod tekrarından kaçınmak adına Generic bir tasarım yapmıştım. Bir süre sonra gelen ek talepler neticesinde tasarladığım Generic yapıyı bozmamak için bazı taklalar atmak zorunda kaldıysam da, en sonunda tasarımı değiştirmekten kaçamadım. Söz konusu modül (ve müşteri ) değişime oldukça yatkın bir yapıya sahip olduğundan başlangıçta bir miktar tekrarlı kodu kabul ederek ilerlemem gerektiğine ikna olmuştum. Modül belirli bir olgunluk seviyesine ulaştığında Generic tip ile refactor edilebilir belki ancak yola çıkarken Generic kullanımı bazı durumlarda **over-engineering** e neden olabiliyor.

Toparlayacak olursak, Generic tasarımı sırf kullanmak için kullanmaktan kaçmamız gerekiyor. Bunun için de hangi yaraya merhem olduğunu, neye çare olduğunu bilmek gerekiyor. İşe başlarken ilk hedefimiz gelen talebi **tam karşılayan,** **en sade** ve **çalışır **versiyonu çıkmak olmalı ki, burada **lean thinking**’e de ufak bir atıfta bulunmuş olduk**. **İlerleyen bölümlerde detaylandıracağız.

### **Frene Basmadan Dolu Dizgin Devam Etmek**

Arada bir frene basmak, hatta kenara çekip tamamen durmak gerekiyor. Son sürat giderken etrafı net göremiyoruz malum.

Kısa aralıklarla **code-review** ve **refactoring **yapmak ve bunu **pair-programmig** ile birleştirmek, yanlış tasarımdan yol yakından dönebilmek için en büyük yardımcımız olacaktır.

Malum her ekipte veya projede pair yapma imkanı olamayabiliyor. Eğer **pair-programming** yapma imkanınız yoksa, yazılımın geliştirme süreci boyunca şu iki soruyu sık sık kendinize sormanızı ve cevap verirken mümkün olduğunda gerçekçi olmaya çalışmanızı öneririm;
> - Bu yazdığım kod ürüne ne kadar değer katıyor?
> - Bunu şimdi yapmaya değer mi?

Gerek kendi verdiğiniz tasarımsal kararlarda gerekse ekip toplantılarında teknik bir tasarım konuşulurken bu iki soruyu tartışmanın, over-engineering’den kaçınma adına faydalı olacağından emin olabilirsiniz.

### Dokunulmaz Kod Yoktur

Bir de over-engineering’e neden olmaktan ziyade mevcut yanlış tasarımın devam etmesine veya daha da kötü bir hal almasına neden olan durumlar vardır.

Uzun süre önce yazılmış olan ve belirli bir süredir de herhangi bir hataya sebebiyet vermemiş kod parçacıkları veya komple bir servis “dokunulmaz kod” muamelesi görebiliyor. “Çalışıyorsa dokunma” kafası yani.

Elbette hatasız ve sistem kaynaklarını olabildiğince az kullanan kod en değerli koddur bizim için. Ancak bu, bu kodların refactor edilemeyeceği anlamına gelmemeli. Şöyle ki;

Söz konusu dokunulmaz kod’umuzun üzerine ek geliştirmeler yapılması gerekti ve biz nasıl olsa çalışıyor diyerek sadece yeni geliştireceğimiz özelliğe odaklandık, geliştirmemizi tamamlayıp yolumuza devam ettik.Tam bu noktada belki de büyük bir fırsatı tepmiş olduk. Bu fırsat, bizim çok uzun zamandır sırf hatasız çalışıyor diye hiç dokunulmayan kodumuzun iyileştirilebilme fırsatı idi. Çünkü zaten geliştirmemizi tamamladıktan sonra komple testten geçirilecek olan ilgili modül, hazır test edilecekken neden öncesinde refactor edilmesin? Belki bir çok gereksiz koddan arındırarak kod okunabilirliğini arttıracak, hatta belki daha az sistem kaynağı kullanır hale getirebilecektik kim bilir? Dokunmadan bilemezsiniz 😏

### **Hype Driven Development**

Bazen gerçekten yetişmekte zorlansakta, her birimiz son teknolojileri yakından takip ediyor, bunlardan popüler olanlarını ise belki çalıştığımız şirketlerde belki de şahsi olarak kullanmaya güncel kalmaya gayret ediyoruz.

Tam bu noktada sırf hype olduğu için bir teknolojiyi kullanma yoluna gidebiliyoruz ki, bu bazı durumlarda sizi over-engineering potasına sokabilir.

Örneğin herkes microservice mimariye geçtiği için monolith uygulamanızı servislere ayırmak zorunda değilsiniz.Gaza gelmeyin, buna gerçekten ihtiyacınız olduğuna kendinizi ve ekibinizi ikna etmelisiniz. Bir teknoloji veya metodolojinin **hype **olması tek başına bir tercih sebebi olmamalıdır.

![image from turnoff.us](https://cdn-images-1.medium.com/max/2600/1*Dz5b4UTUJmwE6aOMFUxBTA.png)*image from turnoff.us*

## Lean Software Development (LSD)

**Lean Thinking** kavramı ilk olarak 1980'li yıllarda otomotiv sanayisinde daha etkin üretim yapılmasını amaçlayan bir iş yapış şekli olarak ortaya çıktı. **Lean Manufacturing Movement (Yalın Üretim Hareketi)**olarak literatüre giren bu kavram, ilerleyen yıllarda yazılım dünyasında **Lean Software Development** (LSD) olarak yer aldı.

LSD Agile community tarafından desteklenmekte. LSD nin birçok prensibi Agile prensiplerle örtüşmekle beraber LSD eşittir Agile gibi bir yaklaşım söz konusu değil. LSD yi hiç bilmeden veya kullanmadan Agile olabileceğiniz gibi tam tersi de mümkündür.

Son olarak** LSD**’ nin en önemli prensiplerine madde madde bakacak olursak;

### **Efor İsrafı**

Lean felsefesine göre müşteri için katma değer sağlamayan her efor ‘israf’ olarak görülmektedir. Önceki bölümlerden birinde sürekli aklınızın bir köşesinde tutmanız gereken iki önemli sorudan bahsetmiştik. Bunlardan ilkini burada hatırlatmakta fayda var.
> Bu yazdığım kod ürüne ne kadar değer katıyor?

### **Kalitenin Sağlanması**

Her ekip yaptıkları işte kaliteyi ön plana çıkarmayı ister elbette. Eğer kalite hedefi bir prensip olarak kabul edilmezse, sadece lafta kalması çok muhtemeldir.
> **LSD yaklaşımında kalite sadece QA ekibinin değil, herkesin işidir.**

Kaliteyi elde etmek için en popüler Lean Development araçları olarak;

* Pair-Programming

* Test-Driven-Development

* Sık sık Refactoring yapılması

* Code-Base’i mümkün olduğunda basit tutmak, olabildiğince az kod yazmak (Bakımı en kolay olan kod, hiç yazılmamış olan koddur.)

* **Automation**. Mümkün olan her manuel işin otomasyona bağlanması, insan hatalarının minimize edilmesi.

* **Context-Switching**’ in mümkün olduğunca azaltılması.

### **Sürekli İletişim ve Öğrenme**

Yazılımı geliştiren ekip ve bu yazılımı talep eden müşteri, iki ana aktör olarak iletişimlerini üst düzeyde tutmalı ve kısa süreli geri bildirim oturumları yaparak mevcut domain hakkındaki bilgi havuzunu genişletmeliler.

Bu kısa süreli ve sık aralıklarla yapılan oturumlarla müşteri ihtiyaç duyduğu şeyleri daha net olarak anlar ve anlatırken, yazılım ekibi de mevcut talepleri nasıl karşılayacağı konusunda daha net fikir sahibi olacaktır.

Bu maddenin yazılım ekibine bakan tarafında ise, düzenli olarak kısa süreli, ekip içi bilgi paylaşım toplantıları yapmak, sürekli güncel tutulan dokümantasyon ve yazılan kodlarda özellikle açıklama ihtiyaç duyulan kısımlar için açıklayıcı yorum satırları yazmak önem arz ediyor.

### **Kritik Kararları Bir Süre Geciktirme**

Her yeni başlanan projede veya mevcut bir projenin yeni bir fazında, tüm ihtiyaçlar %100 oranında kesin ve net olarak **hiç değiştirilmemek üzere **ortaya konsaydı ne güzel olurdu değil mi? Maalesef öyle bir Dünya olmadığını hepimiz çok iyi biliyoruz.

İşte bu yüzden, özellikle nisbeten daha karmaşık sistemlerde, henüz kesinleşmemiş, bazı varsayımlar üzerinden konuşularak ‘el sıkışılan‘ konularda müşteri talepleri kesin olarak netleşene kadar çok önemli diyebileceğimiz bazı tasarımsal kararları **mümkün olabildiğince **geciktirmek gerekiyor. Aksi halde acele edilerek alınan kararlar boşa harcanan yazılım eforu veya gereksiz donanımsal masraf olarak geri dönebilir. Planlamaları kısa vadeli yapıp, ayağımızı yere sağlam basmamız gerekiyor. Aylar sonrası için planlar yaparak bu doğrultuda hemen aksiyonlar almak LSD dinamiklerine aykırıdır.

### **Hızlı ve Atomik Release**

Daha büyük olanın değil daha hızlı olanın hayatta kaldığı bir yazılım ekosistemi içerisindeyiz. Burada hızlıdan kasıt, müşteri isteklerine olabildiğince çevik ve hatasız olarak yanıt verebilme kabiliyetidir.

Major hatalardan arındırılmış olan faz, ne kadar erken tamamlanıp teslim edilirse, o kadar erken geri bildirim alınabilir. Böylece gelen bildirimler hemen bir sonraki faz için planlanarak, kapsama dahil edilebilir.

### **Güçlü Ekip**

LSD yaklaşımının uyguladığı Agile prensiplerden birisi de;
> **Find good people and let them do their own job.**

Bu madde anlaşıldığı üzere yöneticileri ilgilendiriyor. Tabi bunu, “iyi olanları bul, güçlü bir ekip kur ve kenara çekil sadece seyret” olarak anlamamak lazım. Yönetici çalışana işini nasıl yapacağını söylemekten ziyade, süreci tıkayan engeli kaldıran, süreci olumlu yönde teşvik eden, hata tesbiti yapan bir rol üstlenmelidir. **Micro-managing** dediğimiz, kabaca en küçük detaylara bile müdahil olma tarzında ki yönetim şekline girmemelidir.

Bu yazıda, dil ve platform bağımsız olarak her yazılım projesinin geliştirilme sürecini yakından ilgilendiren iki örnemli kavram olan LSD ve over-engineering’den anladıklarımı özetlemeye çalıştım. Özellikle over-engineering için bahsettiğim gerçek hayat örneklerine yeni eklemeler yapmak isterseniz çok memnun olurum. Ek olarak yazım yanlışları, katılmadığnız noktalar vs. için de yine yorumlarınızı beklerim.

### Tavsiye ve Kapanış

Hepimiz gündelik yaşantımızda birçok sorunla uğraşmaktayız. Ailemizle veya dostlarımızla olan ilişkilerimizde de zaman zaman aşmamız gereken sorunlar, yapmamız gereken bazı planlamalar oluyor.

**Lean thinking **prensipleriyle tanıştığımdan beri, bu prensipleri mümkün olduğunca özel yaşantımda da uygulamaya çalışıyorum. Kısa vadeli, nokta atışı planlar yaparak hızlı aksiyonlar almaya çalışıyorum. Evimle veya ailemle alakalı çözülmesi gereken bir sorun olduğunda, basit düşünmeye çalışarak minimum eforla en hızlı çözümü bulmaya çalışıyorum. Önemli bir karar vermem gerekiyorsa ve bazı belirsizlikler söz konusuysa, durumun netleşmesini bekleyerek bu önemli kararı **mümkün olduğunca** bekletiyorum. Bunu denemenizi tavsiye ederim. Lean thinking bir felsefe ve bunu sadece iş hayatıyla sınırlandırmamız gerekmiyor.
