
# Yazılım Ekiplerinde Kod Kalitesi ve Kalitede Sürekliliğin Sağlanması

Photo by Adi Goldstein on Unsplash

Bu yazımda;

* Kod standartlarının belirlenmesinin öneminden,

* Ekip içi kod kalitesi ve kalitenin sürdürülebilirliğinden,

* **Continuous Code Quality** kavramından,

* **DevOps **kültürünün sürdürülebilir kod kalitesine katkısından,

bahsetmeye çalışacağım.

![Hassas kod kalite ölçüm yöntemi :)](https://cdn-images-1.medium.com/max/2000/1*LjG2PZmzv_oy4Vajiu9ARg.png)*Hassas kod kalite ölçüm yöntemi :)*

Yazılım ekipleri henüz oluşum aşamasındayken kod standartlarını belirleyerek, hatta bu standartları dokümante ederek yola çıkmalılar. Başlarda sık sık güncellenen bu kural setleri bir süre sonra olgunlaşarak o ekip için bir anayasa haline gelmeli. ( Bu anayasa da tüm maddeler her an değişebilir, değiştirilmesi tartışmaya açılabilir. ) Özellikle büyük ekipler veya büyüme potansiyeli olan ekipler için hayati bir konu bu.

Peki ekip içi yapılan toplantılar ve eldeki mevcut dokümantasyon kaliteyi ve kalitede sürekliliği elde etmede yeterli olabilir mi? Herkesin eldeki dokümana ve konuşulanlara sadık kalacağından nasıl emin olabiliriz?

**Github**’ın hayatımıza soktuğu **Pull Request (PR)( **Gitlab’in **Merge Request**’i ) buna çare olabilir mi? Bilmeyenleriniz için kısaca; **Pull Request** ile belirli bir branch’i **protected **yaparak bu **branch**’e doğrudan **merge **işlemi yapmak yerine **PR **oluşturarak, **reviewer**’lar tarafından onay alınmak suretiyle **merge **işlemi gerçekleştirilir. Bu **reviewer**’lar bir veya daha fazla sayıda olabilmekte.

**Code-review** elbete standartlara bağımlılık ve kod kalitesi açısından faydalı olacaktır, ancak nihayetinde bu işlem insan eliyle yapılan bir işlemdir. Bir sebepten ötürü (iş yoğunluğu, dikkatsizlik vs.. ) standartlara uygun olmayan, kodları devreye almış olabiliriz. Düşünün onayınıza bir **Pull Request **geldi ve incelemek için açtığınızda yüzlerce satır kodla karşılaştınız. Tabi bu senaryonun ideal şartlarda oluşmaması gerekir, yani yapılan **PR**’lar olabildiğince atomik, küçük iş parçalarına bölünmüş halde gönderilmelidir. Ancak pratikte işler her zaman böyle yürümüyor maalesef ve bir süre sonra işler çığırından çıkabiliyor.

### Çare : Continuous Code Quality

**Continuous Integration**, **Continuous Delivery** ve **Continuous Deployment**’ı anladıkta bu **Continuous Code Quality** de ne ola ki?

**Continuous Code Quality **için şöyle bir tanım yapabiliriz**;**

Kod kalitesinin sürekli olarak denetim altında tutulması, projeye eklenen yeni kodlarda mevcut olabilecek **bug**, güvenlik zafiyeti, **anti-pattern**, gereksiz veya tekrarlı kod vb. gibi olumsuzlukların otomatik olarak gözlemlenmesi, ve raporlanabilmesidir.

**Continuous Code Quality’**i** Continuous Integration**’a eklenebilen bir halka olarak düşünebilirsiniz. ( Halkanın nereye ekleneceğine birazdan değineceğiz.)

**DevOps kültürünün benimsendiği ve uygulandığı ekiplerde**, bu kültürün bir sonucu olarak, işleri otomasyona bağlamaya olan eğilim, ekibi bir kod kalite analiz yazılımını **CI pipeline**’ı içerisine dahil etmeye yöneltecektir.

![Image from [https://code-maze.com](https://code-maze.com)](https://cdn-images-1.medium.com/max/2048/1*OClC5mUgrt_6c7AgjIJR7Q.png)*Image from [https://code-maze.com](https://code-maze.com)*

Yukarıdaki görsel 3 kavramın farklarını açıkça göstermekte. Dikkat ederseniz **Continuous Deployment**’ta tüm süreç otomasyona bağlanmış durumdayken **Continuous Delivery **modelinde ise sadece son **Deploy **işlemi manuel yapılmakta.

Peki **Continuous Code Quality** (**Code Analysis** olarakta geçer) adımını nereye konumlandıracağız? Bu kısım aslında ekibinizin önceliklerine göre değişim gösterecektir. Öncelikten kasıt nedir, burayı biraz açalım;

Eğer bir **Continuous Code Quality **aracını devreye aldıysanız (ilerleyen bölümlerde bu araçlara birkaç örnek vereceğim) her **commit **sonrası bu aracınız otomatik olarak tetiklenip, güncel kodlardan bir kod analiz raporu oluşturacaktır. Bu rapor sonucunun sizin belirlediğiniz kalite standartlarında olmaması durumunda **CI** pipeline’ınızı durdurabilirsiniz, veya sonuçlar çok kötü olsa bile bu analiz raporunu daha sonra incelemek üzere **pipeline**’ı durdurmadan yolunuza devam edebilirsiniz. Eğer **pipeline**’ı durdurursanız email, slack, sms vs.. üzerinden ilgili kişilere bildirim göndererek, analiz raporunun başarısız olmasına neden olan eksikliklerin hızlıca giderilmesini sağlayabilirsiniz. Yani buradaki karar sizin kod kalitesi ve standartlarından vereceğiniz tavizin seviyesine göre değişecektir.

Ben olsam nasıl tasarlardım? **Build **ve **Unit Tests** adımları arasına konumlandırırdım. Eğer kod analiz sonuçları başarısız ise, bir sonraki ( **Unit Tests )** adıma geçmeyerek, pipeline’ımı kırıp, **CI **sunucumu azad ederdim. Daha açık ifadesiyle, kod analiz rapor sonucum **başarısız** olarak dönerse, güncel kodu **deploy **dahi etmezdim. Yani benden sıfır taviz :)

Eğer siz kod analiz sonuçlarınızı sadece ekiple paylaşmak üzere bilgi amaçlı tutmak istiyorsanız, **Continuous Code Quality **adımının** CI pipeline’ **ınızdaki** **konumu pek önem arz etmiyor, neticede **pipeline’**ınız** **her durumda yoluna devam edecektir. Raporu sonuçları için kullandığınız kod analiz aracının arayüzünü göz atmanız yetecektir.

### **Kod Analiz Raporu Kime Göre Başarılı / Başarısız?**

Buraya kadar okuduysanız ve henüz bir kod analiz yazılımı kullanmadıysanız bu soru kafanızda belirmiştir diye tahmin ediyorum.

Yukarıda “analiz raporu sonucu başarısızsa” şartını içeren bir kaç cümle kurduk. Peki bu başarı kriterlerini kim belirliyor? **Bu sorunun cevabı kullandığınız kod analiz aracına göre bir nebze değişebilir**, ancak bu eşik değerini büyük ölçüde siz belirlersiniz. Biraz havada kaldı farkındayım, somutlaştırmak adına şöyle bir örnek verebiliriz;

Kod analiz araçlarında arayüzleri kullanarak bazı kural setleri oluşturursunuz. Bu kural setleri ile siz gelen **commit’**in** **içerdiği **yeni kodların **sınavdan geçer not alıp almayacağını belirlersiniz. Bu kuralların her biri **Quality Gate** olarak** **bilinir. Örneğin aşağıdaki gibi bir** başarısız senaryo kuralı** oluşturarak, kurala uymayan durumlarda analiz raporunun başarısız sonuç dönmesini sağlayabilirsiniz. Kuralı **pseudo **olarak şöyle tanımlıyorum;

    **Yeni kodun tekrarlı satır sayısı ( % ) > 3**

Yani son gelen **commit**’teki toplam kod satır sayısının en fazla % 3' lük bir kısmı tekrarlı kod olabilir. Oranın %3' ün üzerinde olması durumunda rapor başarısız sonuç dönecektir. Konuyu somutlaştırmak adın sadece basit bir örnekti bu. Kod analizi tabi ki bundan çok daha fazlası. Kod karmaşıklığı, kodun güvenlik boyutu, okunabilir kod, bakımı kolay kod vb. gibi konularda bir çok kural seti tanımlamak gerekebilir ki, genel de kod analiz araçlarında bir çok kural **ön tanımlı** olarak gelmektedir. Yani tüm kuralları sıfırdan oluşturmanız gerekmez, ön tanımlı kuralları kendi önceliklerinize göre düzenleyebilir, yeni kurallar ekleyebilirsiniz.

### **CI/CD Yapmıyoruz . Kod Analiz Yazılımı Kullanamaz mıyız?**

Tabi ki kullanabilirsiniz. (Versiyon kontrol sistemi (git, tfs vs..)kullandığınızı varsayıyorum :) )

Kullandığınız versiyon kontrol sistemiyle, kod analiz yazılımını entegre ettikten sonra gerçekleşen her **commit** işlemi ile birlikte kod analiz aracı tetiklenerek güncel kodlar üzerinden yeni bir analiz raporu oluşturacaktır.

Tabi **CI** mekanizmanız olmadığı için yukarıda bahsettiğimiz aksiyonlardan, rapor sonuçlarını email, slack vb. bir kanaldan bildirim göndermek dışında başka bir aksiyon alma şansınız ( bildiğim kadarıyla )olmayacaktır.

### Bazı Kod Kalite Analizi Araçları

Deneyimlediğim veya sadece duyduğum bir kaçı;

* Eski ismiyle Sonar yeni ismiyle [**SonarQube](https://www.sonarqube.org/)**

* Veracode

* Blackduck

* Checkmarx

* Squale

Açıkçası bu araçlardan yalnızda S**onarQube**’ü deneyimleme fırsatım oldu. Oldukça başarılı bir araç olduğunu söyleyebilirim. Beğendiğim bazı özellikleri;

* **Neredeyse tüm modern CI araçlarıyla entegre olabiliyor.**

![from [www.sonarqube.org](https://www.sonarqube.org)](https://cdn-images-1.medium.com/max/2000/1*ZUAEJNimoRfp_24iaWuHcw.png)*from [www.sonarqube.org](https://www.sonarqube.org)*

* **20'den fazla dil desteği.**

![from [www.sonarqube.org](https://www.sonarqube.org)](https://cdn-images-1.medium.com/max/2000/1*SA2JwkN9A8DRQ981nUPJ3A.png)*from [www.sonarqube.org](https://www.sonarqube.org)*

* **Projelerde çoklu dili desteklemesi.** Örneğin projenizde back-end ve front-end tümleşik durumda ise ve biri java diğeri javascript ile yazılmış ise, **SonarQube **raporlamayı dilleri gruplandırarak yapabiliyor.

![from [www.sonarqube.org](https://www.sonarqube.org)](https://cdn-images-1.medium.com/max/2000/1*DaMubbTgbLOqiSEMRQtO0A.png)*from [www.sonarqube.org](https://www.sonarqube.org)*

* **Webhook desteği**

![from [www.sonarqube.org](https://www.sonarqube.org)](https://cdn-images-1.medium.com/max/2060/1*KNG6e0Xyr2Me2Raml6nr7g.png)*from [www.sonarqube.org](https://www.sonarqube.org)*

Bir kod kalite analizi aracı olarak **SonarQube**’ü rahatlıkla önerebilirim. İlerleyen günlerde SonarQube kurulumu, konfigrasyonu, kural tanımlamalarını ve kullanım detaylarını içeren bir yazı yazmayı planlıyorum. ( **Güncelleme:** Planladığım yazıyı tamamladım, [**buradan](https://medium.com/@suadev/sonarqube-i%CC%87le-continuous-code-quality-375661d0cf53) **ulaşabilirsiniz. )

Bu yazımda kaliteli kodun ve kalitede sürekliliğin sağlanmasının önemine dikkat çekmek istedim. Konuyla alakalı farklı görüş ve önerileriniz için yorumlarınızı beklerim.
