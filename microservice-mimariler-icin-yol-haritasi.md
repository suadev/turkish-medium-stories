
# Microservice Mimari’ler İçin Yol Haritası

image from en.dopl3r.com

**Microservice Mimari, **2011 yılındaki ilk telafuzundan bugüne kadar popülerliğini gün geçtikçe artırdı**. **Özellikle** **son 2–3 yıldır kendi arkadaş çevremde de bir Microservice Mimari muhabbetidir ki almış başını gidiyor. Birçok ekip mevcut **monolith **uygulamalarının yenilenme sürecinde veya yeni başlanacak projelerde Microservice Mimari’yi tercih etmeye başladı.

Bu arada yukarıdaki görselden dolayı yanlış anlaşılmamak adına belirteyim, Microservice Mimari**’**ye karşı değilim tabi. Amacım bu mimariyi uygulamadan önceki ön hazırlıklar ve uygularken dikkat edilmesi gereken noktalara vurgu yapmak. Konuyu aşağıdaki 4 başlık altında inceleyeceğiz;

* **Bir Uygulamanın Microservice Mimari’ye Olan Uygunluğu**

* **Microservice Mimari’ye Geçiş İçin Hazır mıyız?**

* **Microservice Mimari’yi Uygularken Yapılan Hatalar**

* **Kişisel Tavsiyelerim**

## Microservice Mimari’yi Uygulamalı mıyız?

Bu çok önemli soruyu sormaya ve üzerinde belki de günlerce düşünüp konuşmak gerekirken, buna lüzum dahi görmeden, “**Hadi beyler Microservice Mimari’ye geçiyoruz**” diyerek mevzuya bodoslama dalan bir çok ekip olduğuna eminim. Aksi halde bu kadar “**başarısızlık hikayes**i” ortaya çıkmazdı diye düşünüyorum.

Bu bölümde madde madde “**Microservice Mimari’yi Uygulamalı mıyız?**”sorusuna nasıl cevap aramamız gerektiğine bakalım. Eğer burada bahsedilen sorunlar veya iyileştirmeler sizin için geçerli değilse Microservice Mimari’yi uygulamak için geçerli bir sebebiniz yoktur diyebiliriz.

Buna rağmen siz, “Yok benim de microservice’ lerim olacak, binecem sırtlarına vuracam kırbacı” diyorsanız siz bilirsiniz tabi :)

![“Bizim Monolith uygulamayı servislere bölelim” diye sürekli darlanan Tech Lead](https://cdn-images-1.medium.com/max/2000/1*VTHpho9riuroAf1tfNOtFw.png)*“Bizim Monolith uygulamayı servislere bölelim” diye sürekli darlanan Tech Lead*

* **Belirli Bir Teknoloji / Dil Ekosisteminde Sıkışmak**

İçerisinde çeşitli modüller barındıran **monolith **yapıda bir uygulama düşünelim. **Monolith **yapıdan dolayı tüm kodumuz tek bir proje yani tek bir **repository **içerisinde.

Talep doğrultusunda projenize yeni eklenecek olan bir özellik için görüntü işleme yapmanız gerekiyor. Kullanıcılarınızın ara yüzden yükleyeceği araba fotoğraflarından plaka tanımlaması yaparak sisteminizde bu plakanın kayıtlı olup olmadığına bakmanız gerekiyor. Bu durumda uygulamayı geliştirdiğiniz dil ve teknolojilerle bu özelliği geliştirmeniz gerekmekte. Eğer diliniz bu iş için pek elverişli bir dil değilse, hem geliştirme eforunuz artacak hem de belki ciddi performans sorunları yaşayacaksınız.

Bu işlemi yapan bağımsız bir servis geliştirelim, ve bu servisi istediğimiz farklı bir dille (**go**,**python **vs..) geliştirerek hem performans kazanımı elde edelim hem de teknoloji **stack**’ imizi genişletelim diye düşünmeye başladıysanız, microservice mimari sizin için uygun **olabilir.**

* **Tightly Coupled (Sıkı sıkıya bağlı) Tasarım Sorunu**

**Monolith** projenizde meydana gelen bir hata, bir veya birden fazla servisinizin devre dışı kalmasına neden oluyorsa mevcut mimarinizi gözden geçirmeniz gerekecektir.

Yine görüntü işleme modülüne sahip uygulamamızdan gidecek olursak; Görüntü işleme modülündeki bir bug’dan kaynaklı olarak **memory-leak** oluştuğu ve buna bağlı olarak uygulama sunucusunun uygulamayı durdurduğu senaryosunu düşünelim. Burada görüntü işleme modülündeki hata tüm uygulamamızı erişilemez hale getirmiş oldu. Microservice mimaride aynı senaryoda, sadece görüntü işleme servisi erişilemez olacağı için uygulamamız diğer fonksiyonlarıyla nefes almaya devam etmiş olacak. (Her servis ayrı bir uygulama sunucusunda koşuyor)

* **Geliştirilmiş Ölçeklenebilirlik**

Yok biz böyle iyiyiz dediniz **monolith **yapıda devam ediyorsunuz ve kullanıcı sayınız da gün geçtikçe artıyor. Derken bizim görüntü işleme modülünün çok geç yanıt döndüğü şikayetleri gelmeye başlıyor. Mimariyi, gelen image’ leri kuyruklayıp tek tek işleme üzerine kurmuşsunuz. Önceleri performans sorununuz yokken sisteme bir** t** anında yüklenen image sayısı arttıkça kuyrukta bekleme süresi uzadı ve neticede kullanıcılarınız mutsuz.

Bu durumda sunucunuzda kaynak arttırımı yapıp dikey ölçekleyebilir (**bir yere kadar**) veya bir kaç sunucu daha devreye alıp yatay ölçeklendirmeye gidebilirsiniz. Peki görüntü işleme modülü dışında kalan modüllerde de bu ölçeklenme ihtiyacı söz konusu mu? Cevabınız hayır ise, tek bir servisi dilediğinizce yatay/dikey ölçekleyebilme imkanını elde edeceğiniz Microservice Mimari’yi düşünebilirsiniz.

* **Kolay ve Hızlı Release Çıkabilme**

“**En ufak bir değişiklikte koca uygulamayı olduğu gibi deploy ediyoruz :-/**” tarzı cümleler kurmaya başladıysanız, sizin gözünüz gibi bakıp besleyip büyüttüğünüz uygulamanızın irili ufaklı microservice’lere bölünme zamanı gelmiş **olabilir**.

## Microservice Mimari’ye Geçiş İçin Hazır mıyız?

Önceki başlıkta bahsettiğimiz konulardan sonra bu mimariye ihtiyacınız olduğuna kanaat getirdiniz. Peki nereden başlamalısınız? Yine madde madde inceleyelim;

* **Güçlü Bir DevOps Ekibi**

Microservice Mimari’yi uygulayabilmek için en önemli şeylerden birisi hatta belki de en önemlisi **DevOps **kültürünün benimsendiği ve hakkıyla uygulandığı bir ekip oluşturabilmektir.

**Martin Fowler **microservice mimarinin getirdiği operasyonel yükü taşıyabilmenin güçlü bir **DevOps **takımına sahip olmaktan geçtiğini söyler. Öncelikle, servislerinizin **building**, **configuration** ve **deploying **süreçleri için bir **CI/CD pipeline’ı **tasarlamanız ve hayata geçirmeniz gerekiyor. Aksi halde çok fazla operasyonel yükle karşı karşıya kalabilir, dolayısıyla çok fazla vakit (**nakit**) kaybına uğrayabilirsiniz.

İlk etapta bazı süreçleri **manuel **yürütebilirsiniz belki. Ancak servis sayınız arttıkça ve **production **ortamınızı hazırlama zamanı geldiğinde artık **full-automated** olmanız gerekecektir.

* **En Az Bir Cloud Provider Üzerinde Uzmanlaşmak**

Servislerinizi **on-premise** olarak kendi organizasyonunuz içerisinde kendi alt yapı ve ekipmanlarınızla sunabilirsiniz elbette. Her ne kadar tavsiye edilen yöntem bu olmasa da en azından ilk etapta bu şekilde ilerlenebilir.

Ancak ilerleyen zamanlarda ürününüz **live **olmadan önce bir **cloud provider **(aws,google cloud, azure) ile, cloud-based mimariye geçerek, hem operasyonel yükünüzü azaltıp hem de çok trafiği olan servisleriniz için **auto-scale** tarifeyi uygulayabilirsiniz. Bizim sunucular ayakta mı, ram/cpu yetersiz mi kaldı gibi soruları hayatınızdan çıkarmak sizin elinizde.

* **Gelişmiş Monitoring ve Notification Araçları**

Birbirinden bağımsız olarak hayatına devam eden ve uygulamanın büyüklüğüne göre onlarca hatta yüzlerce sayıda olabilen microservice’lerin anlık olarak monitor edilebilmesi son derece önemlidir. Bu servislerden herhangi birinde meydana gelecek bir sorunun en hızlı şekilde ilgili yerlere sms, e-mail, slack vb. gibi kanallardan bildirimi, sorunun hızlıca çözümü ve uygulamanın hayatını sürdürebilmesi adına çok kritiktir. Dolayısıyla Microservice Mimari’yi uygulamadan önce bu ihtiyaca cevap verebilecek bir monitoring aracının da devreye alınması veya en azından ön araştırmasının yapılması şarttır.

Geçenlerde Atlassian tarafından satın alınan başarılı yerli girişim [OpsGenie ](https://engineering.opsgenie.com/)ilk aklıma gelen araç. New Relic’i de söylemeden geçmeyelim tabi. Bunların haricinde bir çok ücretsiz açık kaynak araç mevcut elbette.

* **Her Microservice İçin İzole Bir Veritabanı**

Bu konu sıfırdan başlanan uygulamalardan ziyade daha çok mevcut **monolith** bir uygulamanın servislere ayrıştırılması durumunda ortaya çıkıyor.

Microservice mimari, her servisin kendine ait izole bir veri depolama alanı olmasını gerektirir.

**Monolith **bir** **uygulamanın servislere ayrıştırılması konusunda, veritabanı kısmı ilk etapta pek düşünülmeyebiliyor. Servisler şekillenmeye ve monolith yapıdan kopmaya başladıkça veritabanlarının izolasyonu konuşulmaya başlanıyor. Tam bu noktada mevcut veritabanının büyüklüğü ve tasarım kompleksliği aşılması gereken büyük bir sorun olarak karşımıza çıkıyor. Eğer bu sorun bir şekilde aşılamazsa, ya microservice mimariden vazgeçiliyor (servisleri ayrıştırmak için boşa harcanan efor), ya da izole veritabanı konusundan taviz verilerek her servis ortak bir veritabanını kullanacak şekilde ilerleniyor, ki bu microservice mimarinin temel prensiplerine uymayan bir şey. Bu yüzden Microservice Mimari’ye dönüşümde ilk önce veritabanı kısmını tasarlamak ve mevcut veritabanının servislere özel olarak ayrışıp ayrıştırılamayacağı, bu işin eforunun ne olacağı konusunda biraz kafa patlatmak gerekiyor.

## **Microservice Mimari’yi Uygularken Yapılan Hatalar**

Bu mimariyi uygularken zaman içerisinde mimarinin temelini oluşturan prensipler ihlal edilebiliyor. Bunu iki nedene bağlayabiliriz. Birincisi temel prensipleri tam olarak anlayıp benimsemeden işe koyulmak. İkincisi, izole veritabanı başlığı altında bahsettiğimiz istemeden de olsa bilinçli olarak verilen veya verilmek zorunda kalınan tavizler. Şimdi bu mimariyi uygularken aklımızın bir köşesinde sürekli tutmamız gereken önemli noktalardan bahsedelim;

* **Servisler Arası Ortak Kütüphane (library) Kullanmamaya Çalışın**

“E hani **DRY (don’t repeat yourself) **prensibine ne oldu? Aynı kodları her serviste tekrar tekrar yazacak mıyız?” diye düşünebilirsiniz.

Öncelikle tekrarlı kod oranımızı mümkün olduğunca azaltmamız gerekiyor tabi, **DRY **prensibinde bahsedilen konu **bussiness logic**’i içeren kod parçacığının sadece bir kere yazılması. Ancak microservice mimari de ise işler biraz değişiyor. Eğer ortak kullanılan fonksiyonlarınızı bir kütüphane haline getirir ve servislerinizi bu kütüphanelere bağımlı hale getirirseniz mikroservice mimarinin sağladığı en büyük kazanımlardan birinden feragat etmiş olacaksınız: **Bağımsızlık (** **Independence)**

Unutmayın; bu mimaride her servis bağımsız olarak geliştirilip, yine bağımsız olarak **release **edilebilmelidir. Peki bu durumda ne yapmak gerekiyor? Şu 3 seçenekten birisi tercih edilebilir;

1- **Servisler arası** tekrarlı kodu kabullen.(Bir noktaya kadar)
> **Duplication is better than the wrong abstraction.**
[10 Modern Software Over-Engineering Mistakes](https://medium.com/@rdsubhas/10-modern-software-engineering-mistakes-bc67fbef4fc8#577d)

2- Ortak kodlar **business logic **içeriyorsa** **yeni bir **shared service** oluştur.

3- Eğer mümkünse, servisleri yeniden dizayn et ve yeni alt **microservice**/ler oluştur.

* **Bir Servisin Diğer Bir Servisin Verisine Erişim Yöntemi**

İzole veritabanı maddesinde biraz bahsetmiştik, burada biraz detaylandıralım. İzole’den kasıt, bir servisin veritabanına erişimin sadece o servis üzerinden olmasıdır. Yani bir servis başka bir servisin veritabanına doğrudan erişim sağlayamaz, erişmesi gerekiyorsa veritabanının sahibi olan ilgili servis üzerinden erişmelidir. Yani bir client gibi istekte bulunmalıdır. Bu kuralda yine ihlal edilebilen kurallar arasında sayılabilir.

Genelde performans endişesinden dolayı yapılan bu hatada, **A** servisi **B** servisine istekte bulunmak yerine, doğrudan B servisinin veritabanına erişerek, microservice mimarinin **gevşek** **bağlılık (loosely coupled) **prensibinin ihlaline neden olmaktadır. Bu ayrıca **A** servisi daha kompleks ve bakımı zor bir hale getirecektir.

* **Authentication, Throttling gibi işlemlerin merkezileştirilmeMEsi**

Bu maddeyi Throttling (**Rate Limiting**) üzerinden ele alabiliriz. Çok sayıda microservice’e sahip bir sisteminiz var ve her uygulamada olduğu gibi sizin uygulamanız için de client istatistikleri çok önemli. Hangi client veya hangi kanallardan (web, mobil) hangi servisler ne sıklıkla tüketiliyor? Hangi servislere daha çok yük biniyor? gibi soruların cevaplarını merkezi bir yapı üzerinden almak en doğrusu. Bu yapılar **Api Gateway **olarak adlandırılan yazılımlardır. Bu yapılar ile client’lardan gelen tüm istekler bir veya daha fazla olabilen api gateway’ler üzerinden ilgili servislere erişir.

Data önce deneyimle fırsatı da bulduğum **IBM’**in **Datapower **isimli gateway’i ilk aklıma gelen örnek. Bunun yanında open source ve tamamen ücretsiz olan çok başarılı api gateway’ler de mevcut. (**Kong**, **Tyk **vs.)

Api gateway kullanılmadığı takdirde, Rate Limiting ve benzeri işlemler her servis özelinde tek tek implemente edilmek zorunda kalınacak ki bu da bir süre sonra servisleri daha kompleks bir hale getirerek bakım maliyetlerini artıracaktır. Bir diğer sorun tabi gereksiz harcanan efor olacaktır. Api gateway ile merkezileştirilebilecek işlemler, her yenidoğan microservice için yeniden tek tek implemente edilmek zorunda kalınacaktır. Api gateway’ler oldukça yetenekli araçlardır, ve microservice mimari’de kullanımlarının bir çok faydası vardır. Öyleki tek başına ayrı bir yazı konusu olacak genişlikte olduğundan burada kesmekte fayda görüyorum.

![image from infoq.com](https://cdn-images-1.medium.com/max/2000/1*GTdLUIiQ2leZh4R27P4d0w.jpeg)*image from infoq.com*

## **Kişisel Tavsiyelerim**

**Çiçeği burnunda startup’lar veya olgunlaşmış ekiplerdeki yeni başlanacak olan projeler için tavsiyeler;**

Eğer yazılım ekibiniz 2–5 kişilik bir ekipse ve ileride bu ekibi büyütme planlarınız yoksa, monolith monolith devam etmenizde fayda var. Zaten kısıtlı sayıdaki iş gücünüzü microservice mimari’nin getirdiği operasyonel yükün altında ezmek istemezsiniz.

Peki, bu 2–5 kişilik ekibiniz için yakın zamanda büyüme planlarınız var diyelim. Bu durumda da aynı şekilde monolith olarak başlamanızı öneririm. Yani bana göre yeni uygulamalar için başlangıç her zaman monolith olmalı. Ekibiniz ve uygulamanız büyüdükçe DevOps alt yapınızı oluşturup, mimari değişikliğe gidebilirsiniz. Konuyla ilgili [**buradaki](https://martinfowler.com/bliki/MonolithFirst.html)** yazıya da göz atmanızı tavsiye ederim.

**Monolith den microservice mimari’ye dönüşüm için tavsiyeler;**

Bu işi kesinlikle hafife almayın ve sizi neyin beklediğini iyice araştırmadan ilk kazmayı vurmayın. Yukarıda mümkün olduğunca özet haline getirmeye çalıştığım maddelerden her birisi, işe başlamadan önce ve başladıktan sonra bir şekilde karşınıza çıkacaktır emin olun.

Microservice mimari’ye dönüşümde Netflix’in hikayesi oldukça bilinen bir hikayedir. Monolith uygulamayı **cloud-based microservice mimari’**ye dönüştürme işine yanılmıyorsam 2009–2010 gibi başladılar ve bitirdiklerinde 2016 ya gelinmişti. Belki sizin sisteminiz bu denli büyük ve kompleks değildir, ancak işin ciddiyetini ve açılması gereken çok fazla kilit olduğunu Netfilix’in hikayesinden anlamak mümkün.

Toparlarsak; Microservice Mimari’ye girmek aslında DevOps kafasına girmek olmalı. Geliştiricilerin sürecin her aşamasında etkin rol oynaması gerekiyor. Eğer daha önce hiç Microservice Mimari ve DevOps konuları hakkında deneyim edinmemiş bir ekip ile yola çıkıyorsanız, en azından temel konular hakkında bir eğitim ile işe başlamak bana göre elzemdir. Bu size vakit kazandıracağı gibi daha emin adımlarla ilerlemenize de imkan sağlayacaktır.

Microservice Mimari hakkında yazıda katılmadığınız noktalar ve önerileriniz için yorumlarınızı beklerim.
