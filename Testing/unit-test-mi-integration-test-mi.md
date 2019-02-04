
# Unit Test mi? Integration Test mi?

Tabi ki ikisi de.

Peki o zaman neden bu başlık?

Her ikisi de farklı amaçlara hizmet ediyorlar. **Integration Test **bize bir şeyin çalışıp çalışmadığını söylerken, **Unit Test** neden çalışmadığını söyler. **Unit Test** yazılımcı perspektifinden bakarken, **Integration Test** kullanıcı perspektifinden yazılır. **Integration Tes**t ile, sistemin kullanıcıların beklediği gibi çalıştığından emin oluruz.

Bu kısa yazıda,** Unit Test** yazılabilecek bir senaryoda **Integration Test** yazmanın sonuçlarına değineceğim. Dolayısıyla**, hali hazırda yazılmış veya bundan sonra yazılacak olan testlerimizin toplam çalıştırılma süresini azaltabilir miyiz? **sorusuna cevap vermeye çalışacağım.

**C**ontinuous **I**ntegration ve **C**ontinuous **D**elivery kavramları iyiden iyiye hayatımıza girmiş durumda. DevOps kültürünün yaygınlaşmaya başlamasıyla birlikte tanıdığım bildiğim hemen her yazılım ekibi mutlaka bir **CI/CD** aracı kullanır duruma geldiler. Bu arada **CI** aracını devreye aldık artık biz de **DevOps** yapıyoruz gibi bir dünya yok tabi :) **DevOps **bundan çok daha fazlası, aslında bir kültür. Bu farklı bir yazının konusu olabilir.

Gelelim konumuzun **CI**/**CD** ile alakasına;

Genel haliyle bir **CI** pipelıne’ı şu 3 adımdan oluşur;

* Kaynak kodun güncel halinin alınması ve derlenmesi - **Build**

* Testlerin çalıştırılması - **Test**

* Build paketinin uygulama sunucusuna deploy edilmesi - **Deploy**

( Tabi projede hiç test yazılmamışsa pipeline’ımız 2 adımlı olacaktır. )

Şimdi, projemizde çok sayıda unit/integration vb. test yazdığımızı ve **CI** pipeline ımızın **Test** adımının bir hayli zaman aldığını düşünelim. Eğer az sayıda proje üzerinde çalışan küçük bir ekipseniz bu durum sizin için önemsiz olabilir. Öyle olsa bile **CI **sunucunuzu yani kaynaklarınızı verimli kullanmak istersiniz. Öte yandan **çok fazla** projenin olduğu, birden fazla alt takımdan oluşan organizasyonlarda ise, uzun süren **CI **pipeline’ları kuyrukta bekleme süresini uzatacak ve eğer tek bir **CI **sunucucu tek hat üzerinden istek kabul ediyorsa,

“Ne oldu bizim kod hala teste gitmedi mi?”

“Acil test edilmesi lazımdı.”

”XXX projesi hala running durumunda, en az 20 dakikası daha var “

gibi homurdanmaların başlaması yakındır.

Yani çok sayıda test senaryonuz( dolayısıyla test metodunuz ) varsa, testlerin çalıştırılma süresini **gereksiz yere** uzatmaktan kaçınmanız gerekiyor. Bu yazıda benim gözlemlediğim en bariz hatalardan birisi olan, “**Gereksiz Integration Test yazılması**” durumunu bir örnek üzerinden ele alacağım.

Mottomuz;
> **Eğer unit test ile halledebiliyorsan, integration test yazma.**

**Integration test, unit test’e göre çok daha maliyetlidir.** Örneğin; içerisinde veri tabanına erişen kodlar barındıran bir metot için **Unit Test** yazarken, gerçekte veri tabanına erişmezsiniz. Veri tabanına erişen servisinizi mock’layarak metodun dış dünya ile olan ilişkisini kesersiniz. Ancak **Integration Test** farklıdır. **Integration Test**, veri tabanı erişimi veya bir web servis çağrısı söz konusu ise bu **IO **işlemini gerçekten icra edecek şekilde yazılır. Buradan hangisinin daha maliyetli olduğu anlaşılabiliyor. Peki neden daha maliyetli olanı seçeyim? Benim gözlemlediğim 2 yaygın neden;

* Yukarıda bahsettiğimiz maliyet konusu hakkında bilinçli olmamak veya önemsememek. (küçük ekipler, az sayıda proje)

* **Integration Test** yazarken birbirine benzer olan senaryolar için copy-paste şeklinde yazılan test metotları.

2. maddeyi açalım;

2 farklı senaryo için 2 **Integration Test** metodumuz olsun. İlk test metodu olması gerektiği gibi **Integration Test** yazılımına uygunken bundan kopyalanarak oluşturulan 2. senaryo esasında Unit Test yazılarak ilerlenebilecek bir senaryo. Örnek üzerinden anlatmaya çalışayım;

Hava durumu bilgisi sunan bir servisiniz var ve bu servisi bireysel kullanıcılar ücretsiz olarak günlük 1000 adet istek limitiyle kullanırken, kurumsal kullanıcılar ise belirli ücret karşılığında limitsiz olarak kullanıyor. Hava durumu bilgisini sağlayan servisin **sözde **kodu şöyle olsun;

![](https://cdn-images-1.medium.com/max/2000/1*d9MLuK8acWHaNPw4t9Zw_A.jpeg)

Sözde kodu metotlar halinde yazdım, yazıda bahsetmeyeceğim metotlara ekran görüntüsünde yer vermedim. Tahmin edeceğiniz üzere tüm metotların içi boş :)

**Not:** **Unit/Integration Tes**t metotlarının içeriklerini ekleme gereği duymadım, konunun özünden sapmak istemedim.

Kısaca açıklamak gerekirse;

**GetUserInfo** servisi ile veri tabanına erişip, bu token bir kullanıcıya atanmışsa bu kullanıcının verisini alıyoruz. Sonrasında **CheckUserCredit** ile kullanıcı bireysel mi kurumsal mı, bireyselse günlük kullanım hakkı var mı kontrolünü yapıyoruz. Kullanıcı kurumsal veya bireysel ve günlük kullanım hakkı varsa hava durumu bilgisini **GetWheatherInfo ile **dönüyoruz. Bireyselse ve günlük kullanım hakkı yoksa **Bad Request **dönerek, günlük kredisinin kalmadığı bilgisini veriyoruz.

Bu servis için şu isimde bir **Integration Test** metodu yazabiliriz;

**Get_WheatherInfo_By_Corporate_User_And_Get_Ok_Result( );**

Veri tabanında mevcut olan kurumsal tipte bir **test kullanıcısı**nın token’ını şehir ve tarih bilgisiyle beraber bu metot içerisinde kullanarak servisimize istekte bulunabiliriz. Bu hava durumu servisimiz için en temel ve akla ilk gelecek olan **Integration Test** metodumuz olur, bunu bir de bireysel müşteri için yazabiliriz.

**Get_WheatherInfo_By_Individual_User_And_Get_Ok_Result( );**

Böylece bu 2 test metoduyla hem bireysel hem kurumsal tipteki kullanıcılarımız için hava durumu servisimizin çalışır vaziyette olduğunu garanti altına aldık.

Peki, günlük limitini aşan bir bireysel kullanıcının **Bad Request** alması senaryosu için de **Ingtegration Test **yazabilir miyiz? Elbette yazabiliriz. Şöyle uzunca bir isim verebiliriz hatta,

**Get_WheatherInfo_By_Individual_User_Has_No_Credit_And_Get_Bad_ Request_Result();**

Peki bu **Bad Request** dönmesi gereken senaryo için neden **Unit Test** yazmayalım? Tek yapmamız gereken, **GetUserInfo **ve **GetWheatherInfo **servislerini mock’lamak ve **GetUserInfo** servisinden, **bireysel tipte ve kalan kullanımı sıfır olan fake bir user objes**i dönmek. Böylece **CheckUserCredit **metodu içerisinde yapılan kontroller sonucu **Bad Request** dönecektir.

3. senaryomuzda **Unit Test’**i tercih ederek **GetUserInfo** servisi içerisinden yapılan veritabanı erişimi maliyetinden kurtulmuş olduk.

Bu yazıda Integration Test yazmanın maliyeti hakkında farkındalık oluşturmayı amaçladım. Farklı görüş, öneri veya düzeltmeler için şimdiden teşekkür ederim.
