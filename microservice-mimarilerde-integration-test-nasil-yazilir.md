
# Microservice Mimariler’de Integration Test Nasıl Yazılır ?

image from turnoff.us

**Integration Test **yazmak hatırı sayılır bir efor gerektiriyor. **Unit Test **yazmaya kıyasla daha maliyetli ve açılması gereken kilit sayısı çok daha fazla. Unit Test ile sadece servisimizin fonksiyonelliğini garanti altına alırken, diğer servislerle olan iletişimimizi garantilemek için Integration Test’e ihtiyaç duyarız.( Bkz: [**Integration Test vs Unit Test](https://medium.com/devopsturkiye/unit-test-mi-integration-test-mi-34ddea054696) **)

Konu entegrasyon olunca ve işin içine bir de Microservice Mimari girince zorluk seviyesi daha da artıyor tabi. Eğer daha önce **monolith **bir uygulama için **Integration Test** yazdıysanız aradaki farkı daha net görebilirsiniz. 😏

Bu yazıda **Microservice Mimari** üzerine kurulu sistemlerde** **entegrasyon testlerinin** **öneminden ve konuya farklı bir bakış açısı getirerek bizi Integration Test yazmanın maliyetinden kurtaran **Consumer Driven Contracts Testing **yaklaşımından bahsedeceğim.

### Entegrasyon Testi’ne Neden İhtiyaç Duyuyoruz?

Microservice Mimariler’de servis sayısı arttıkça, servisler arası iletişim ve entegrasyon sayısı da doğru orantılı olarak artacaktır. 10 adet servisten oluşan bir sistem ile 100 adet veya daha fazla servisin birbirleriyle konuştuğu bir sistemin karmaşıklığı ve bakım maliyetleri hiç şüphesiz aynı olmayacaktır.

Şimdi onlarca Microservice’den oluşan büyük ölçekli kurumsal bir uygulamayı ele alalım. **CustomerService **adında bir servisimiz olsun. Bu servis müşterilerimizle alakalı diğer servislerin ihtiyaç duyabileceği tüm servisleri sağlamakta. Dolayısıyla bu serviste yapılacak bir değişikliğin, servisi kullanan diğer servislere bildirilmesi gerekiyor. Aksi halde bu değişiklikten haberdar olmayan ilgili servis veya servisler, eğer şanslı iseniz test ortamında hata verecektir. En kötü senaryoda ise, servis devam eden test süreçlerinin kapsamında değilse ancak canlı ortamda hata vermesiyle haberdar olacaksınız demektir ki bu en son isteyeceğimiz şey olur.

Aynı şekilde, eğer **CustomerService **de** **başka bir servisin verisine ihtiyaç duyuyorsa, o serviste yapılan değişikliğin de **CustomerService **tarafından bilinmesi gerekmekte.

Peki bu değişiklik bildirimlerini nasıl yöneteceğiz? Bunu insan eliyle manuel olarak yönetebilir miyiz? Ayrıca, değiştirdiğimiz servisin hangi servisleri etkileyeceğini biliyor muyuz? Bunun için sürekli güncel tuttuğumuz bir dokümantasyona ihtiyacımız var mı?

Eğer az sayıda Microservice’e sahip bir uygulamamız varsa, söz konusu ekipte küçük bir ekiptir muhtemelen ve bunu belki bir şekilde manuel ilerletebilirsiniz. Burada manuel’den kastımız ekip içerisinde, servislerde yapılan değişikliklerin, değişiklikten etkilenen servislerin sorumlularına doğrudan bildirilmesidir. Ancak onlarca hatta yüzlerce servisten bahsediyorsak bunu manuel olarak yönetmemiz söz konusu değil maalesef.

### Çare: Consumer Driven Contracts Testing (CDC Testing)

Microservislerinizde yaptığınız en ufak bir değişikliğin servisi kullanan **Consumer**’ ları nasıl etkilediğini bilmek mi istiyorsunuz? O zaman doğru yerdesiniz. 😉

**CDC** yaklaşımını en basit haliyle, iki servisin birbirlerine gönderdikleri verinin formatı konusunda anlaşmaya varması olarak tanımlayabiliriz. Bu yaklaşım, **Service Provider **veya **Service Consumer **tarafında yapılan her değişiklik bilgisinin anlık olarak paylaşılması prensibi üzerine kuruludur.

**CDC**’yi uygularken ki en önemli konulardan birisi Consumer ve Provider servislerini yöneten ekipler arasındaki iletişimdir. İletişim ne kadar kopuk olursa CDC’yi uygulamak da o denli zorlaşacaktır. Dolayısıyla en iyi senaryo, tüm servislerin aynı ekibin sorumluluğunda olduğu senaryodur.

Bu kadar konuştuk iyi güzel, peki bu **CDC’** yi nasıl uygulayacağız diye düşünmeye başladığınızı tahmin ederek size [**Pact](https://docs.pact.io/) **framework’den bahsetmek isterim. Pact’in detaylı implementasyonu biraz uzun kaçabileceğinden onu ayrı bir yazıya bırakarak burada sadece hangi yaramıza merhem olduğu, temel yapısı ve kullanımıyla ilgili giriş seviyesinde bilgi vermeyi uygun görüyorum.

### Pact CDC Testing Framework

Pact’in offical tamınına bakarak başlayalım;
> [**Pact](http://pact.io/)** is a consumer-driven contract testing framework. Born out of a microservices boom, Pact was created to solve the problem of integration testing large, distributed systems.

Pact, [**Pact Foundation](https://github.com/pact-foundation)** tarafından Ruby dili kullanılarak geliştirilen açık kaynak bir CDC Testing framework’üdür. Şuan Ruby’nin yanı sıra Php, Go, C# gibi birçok dil için desteği vardır. Pact Fodundation’ın **g[ithub sayfasını](https://github.com/pact-foundation)** incelerseniz eğer her bir dil için ayrı bir repository üzerinden ilerlendiğini görebilirsiniz.

Şimdi çok teknik detayına ve kod kısmına girmeden adım adım implementasyonu nasıl yapacağımıza bakalım. Başka bir yazıda **.Net Core** için [**PactNet](https://github.com/pact-foundation/pact-net)** ile örnek bir proje üzerinden detaylıca anlatmayı planlıyorum.

### Senaryo

**CustomerService **adındaki bir servisimiz ve bu servisimizi kullanan **ProductService **isminde ikinci bir servisimiz var. Burada CustomerService **Provider**, **ProductService **ise **Consumer** rolündeler. Yani **CustomerService**’ de yapılacak bir değişikliğin **ProductService**’i etkileyip etkilemediği konusu önem arz ediyor.

**1- Consumer Servis Tarafında Contract Oluşturma**

**Consumer **rolündeki **ProductService **servisi** **üzerinde eğer mevcut değilse bir test projesi oluşturuyoruz. Test yazmak istediğimiz senaryoları belirleyerek **Unit Test’**lerimizi **PactNet’in **belirlediği formata göre ve provider servisimizi **Mock**’layarak yazıyoruz. Burada **Unit Test **ifadesinin dikkatinizi çekmesi gerekiyordu. 😏 Baştan beri entegrasyon testi deyip duruyorduk nereden çıktı bu diye düşünebilirsiniz Şöyle ki;

**CDC **ile birlikte Integration Test yazmanın maliyetinden kurtulmuş oluyoruz aslında. Bildiğiniz gibi normalde **Integration Test** metodları web service veya veritabanı erişimlerini **Unit Test’**lerden farklı olarak **gerçekten **yapıyorlar. Yani bir **mocking** söz konusu değil. **CDC’**yi **Pact **ile implemente ederken de Unit Test yazar gibi, Provider servisleri mock’luyoruz çünkü bizim için artık önemli olan şey contracts yani Provider ile aramızdaki sözleşmemiz.

**Consumer**, **Provider**’a yapacağı **X, **isteğine karşılık **Y **yanıtını alması gerektiğini Unit Test sınıfları içerisinde **PactNet**’in** **(.Net Pact kütüphanesi) sağladığı fluent metodları kullanarak belirtir.

**2- Pact Contract(Sözleşme) Oluşturulması ve Provider ile Paylaşılması**

**Consumer** tarafında tüm test metotları yazıldıktan sonra testler çalıştırılır. Testler çalıştırıldıktan sonra json formatlı Contract’ımızın oluşması gerekiyor. Bu json dosyası Pact’i konfigure ederken bizim belirlediğimiz bir dizinde oluşacaktır. Test metodlarında yapılan her değişiklikte bu json dosyası otomatik olarak güncellenecek ve belirlenen dizinde her zaman güncel hali yer alacaktır. **Yani sözleşmenin sürekli güncel tutulma işini Pact halletmekte ki bu çok önemli bir konu.**

Oluşan dosyanıın json formatında ve oldukça okunabilir olması çok farklı yapıdaki servisler için bile **CDC **uygulayabilmemize olanak sağlıyor. **Go **dili ile yazılan bir **Provider** ile bir **Php** **Consumer**’ı Pact Contract’ı üzerinden el sıkışabiliyorlar.

Örnek bir Pact Contract’ı için aşağıdaki **gist**’i inceleyebilirsiniz. **Interactions **listesi altında iki adet **interaction **olması, iki adet test metodu yazıldığı anlamına geliyor. **Request **kısmı **Consumer**’ın yapacağı istek, **Response **ise bu isteğe karşı **Consumer**’ın **Provider**’dan beklediği yanıtı ifade eder. Görüldüğü gibi aslında açıklamaya gerek bırakmayacak kadar okunaklı bir format.

<iframe src="https://medium.com/media/79c5cccdee94eafd2af58eba223bad52" frameborder=0></iframe>

**3- Provider’ın Sözleşmeden Haberdar Edilmesi**

Bu son aşamada artık Provider’ımıza kendisiyle anlaşma (contract) imzalamak isteyen **Consumer**’ları bildirip anlaşmaların nerede tutulduğu bilgisini vermemiz gerekiyor ki, **Provider**’ımız kendisinde yaptığı her değişiklikte hangi Consumer’ların etkilenip etkilenmeyeceğini bilsin.

Bu işlemi de yine .Net için yukarıda bahsettiğimiz PactNet’in sağladığı **fluent **metodlar ile kolayca yapabiliyoruz. İstediğimiz kadar Consumer ve Pact Url’ini Provider’ımıza tanımlayabiliyoruz. Kod detayına girmeden konuyu toparlamadan önce son olarak [**Pact Broker](https://github.com/pact-foundation/pact_broker) **kavramından bahsedelim.

**Pact Broker Nedir?**

Oluşan json formatındaki contract’ımızı Provider ile paylaşma bölümünde ‘**Pact’i konfigure ederken bizim belirlediğimiz bir dizinde oluşacaktır.**’ demiştik. **Consumer **ve **Provider**’ın aynı ortamda bulunduğu senaryolarda Contract’ı bir dizinde saklama işimizi görecektir ancak gerçek hayat senaryolarında bu dosyayı internet ortamında, yani bir sunucu üzerinden paylaşmanız gerekecek. **Pact **bize her iki imkanı da sağlamakta. Tek yapmamız gereken json dosyamızın konumu belirtirken dizin yerine **Pact Broker**’ımızın adresini ve varsa **credential** bilgilerini yazmak olacaktır. Aynı işlemi hem **Consumer **hem **Provider **tarafında yaptıktan sonra artık servislerimiz internet üzerinde **public** veya **private **olarak tutulan bir contract üzerinde el sıkışabilecekler.

**Pact Broker** hizmetini **SaaS **olarak almak isterseniz **offical **bir hizmet de mevcut. [**http://pact.dius.com.au](http://pact.dius.com.au)** adresine girerek inceleyebilirsiniz. Eğer Pact’i ciddi olarak kullanmayı düşünürseniz mutlaka bir Pact Broker kullanmanızı öneririm. **pact.dius.com.au **adresindeki **Features **bölümünü inceleyince ne demek istediğimi daha iyi anlayacaksınız😏

Ek olarak Pact Broker’ın arayüzüde Contract’ların son durumunu ve hangi servis hangi servislere bağımlı gibi verileri görebilmekte çok güzel. Örnek olarak Pact Broker’ın github sayfasından aldığım iki ekran görüntüsüyle konuyu sonlandırıyorum.

![image from [https://github.com/pact-foundation/pact_broker](https://github.com/pact-foundation/pact_broker)](https://cdn-images-1.medium.com/max/2000/1*ZkUwwbXuwh5WTvzOf5wQ5Q.png)*image from [https://github.com/pact-foundation/pact_broker](https://github.com/pact-foundation/pact_broker)*

Yukarıdaki görselde tanımlı servislerimiz arasındaki ilişkiyi görebiliyorken, aşağıda ise Contract’ların son durumunu görüyoruz. En sağda yer alan **Last Verified** kolonu, Provider’ın güncel Contract’ı ne zaman **verify** ettiğini göstermekte. Kırmızı renk anlaşmanın bozulduğuna işaret ediyor.

![image from [https://github.com/pact-foundation/pact_broker](https://github.com/pact-foundation/pact_broker)](https://cdn-images-1.medium.com/max/4676/1*NcgahwBMAWIOVKnZzqeITQ.png)*image from [https://github.com/pact-foundation/pact_broker](https://github.com/pact-foundation/pact_broker)*

### **Toparlarsak**

Eğer ürünlerinizi Microservice Mimari ile geliştiriyorsanız, **Pact **veya muadili farklı bir framework’ü yazılım geliştirme yaşam döngünüzün olmazsa olmaz bir parçası haline getirmenizi öneririm. 👌

Bu yazıda Microservice Mimari’lerdeki en sıkıntılı konulardan birisi olan Integration Test konusuna farklı bir bakış getiren **Consumer Driven Contrats **yaklaşımını ve bu yaklaşımı uygulayabilmek için kullanılan **Pact **framework’ünü giriş seviyede tanıtmayı amaçladım. Pact’e alternatif olarak önerebileceğiniz farklı framework’ler veya yaklaşımlar varsa yorumlarınızı beklerim.
