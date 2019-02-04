
# Microservice Mimari’lerde Servisler Arası İletişim Nasıl Olmalı?

image from memegenerator.net

Microservice Mimari malum derya deniz bir konu olduğundan, bir süredir konulara ayırarak dilim döndüğünce anlatmaya çalışıyorum. Diğer **Microservices **yazılarım için medium profilime göz atabilirsiniz.

Bu yazıda, “Servisler arasındaki haberleşme için nasıl bir tasarım yapmalıyız?” sorusuna aşağıdaki 3 başlık altında yanıt arayacağız.

* Request-Driven Mimari

* Event-Driven Mimari

* Hybrid Mimari

## Request-Driven Mimari

Servis sayımız arttıkça mimarimizin karmaşıklığı da, http trafiğimiz de doğru orantılı olarak artacaktır. Bir microservice, datasına ihtiyaç duyduğu başka bir servise, bir http client üzerinden istekte bulunur ve bu işlem bir **IO **(Thread Blocking IO)** **işlemi olduğu için aslında maliyetli de bir işlemdir. Servisler (sunucular) client’larıyla olan bu iletişim**i socket**’ler** **üzerinden yaparlar ve bu socket’ler sonsuz sayıda değildir, bu yüzden socket kullanımı iyi yönetilmesi gereken bir konudur.

Dolayısıyla mimarimizi tasarlarken servislerimizin kendi aralarında yaptıkları isteklerin sayısı konusunda biraz cimri davranmamız gerekiyor. Tabi bunu yaparken, aslında ayrıştırılması gereken servisleri sırf http istek sayısını azaltacağız diyerek birleştirmek de yanlış olacaktır. Aşağı tükürsek sakal yukarı tükürsek bıyık durumu yani. Microservice Mimari’nin zorluklarının üstesinden gelmek de bu gibi kararları doğru verebilmekten geçiyor zaten 😏

![Request-Driven Mimari Örnek Çizim](https://cdn-images-1.medium.com/max/2000/1*FIvn6NIKklC0p-607YDRTA.png)*Request-Driven Mimari Örnek Çizim*

Bu mimari de adından da anlaşılacağı üzere bir service, verisine ihtiyaç duyduğu bir diğer servise doğrudan istekte bulunur. Servislerimizin modern **Rest** servisleri olduğunu kabul edersek yapılan istekler, **GET, POST, DELETE ve UPDATE** isteklerinden ibaret olacaktır.

Yukarıdaki çizimde **fire-and-forget** iletişim şeklini de göstermiş olmak adına, sarı renkli olan servisten geri dönüş belirtmedim. Elbette her http request’in bir http response’u olacaktır, burada isteği yapan kırmızı ve yeşil servislerin dönen resonse ile ilgilenmediklerini anlıyoruz. Örneğin bu sarı renkli servisimiz notifikasyon veya log servisimiz olabilir. (Loglama işlemini http servis üzerinden yapmak **genelde **yanlış bir tercih olur)

Request-Driven Mimari’de servislerimiz arasındaki iletişimi iki yolla sağlayabiliriz;

### Senkron (Synchronous) İletişim

Http protokolü senkron çalışan bir protokoldür. Client bir istek yapar ve sunucudan yanıt dönmesini bekler. Client tarafında servis çağrısını yapan kodun senkron (thread’in blocklanması durumu) veya asenkron (thread’in bloklanmaması ve yanıtın **call back** ile gelmesi) yazılması Http’nin senkron bir protokol olduğu gerçeğini değiştirmez.

Burada aslında ilginç bir durum söz konusudur. Client’in servis çağrısını asenkron olarak yapması(**call back** yapısı ile) veya sunucunun yanıtı asenkron olarak dönmesiyle aslında bir anlamda senkron bir protokol olan Http’den asenkron yanıt vermiş/almış oluyor.uz Ancak bu Http protokolünü değiştirdiğimiz onu asenkron bir şekilde çalıştırdığımız anlamına gelmiyor elbette.

Buna örnek olarak .Net Framework 4.5 ile gelen **async/await **yapısını verebiliriz. Bu yapı, asenkron istek yapma veya yanıt dönme işlemlerini çok kolay bir şekilde yapmamıza imkan sağlıyor. Dolayısıyla microservice’lerinizi geliştirdiğiniz framework’ün ve dilin buna benzer bir yapıya sahip olması, yani isteklerinizi ve yanıtlarınızı asenkron olarak yapmanıza imkan sağlaması önem arz ediyor.

DotNet’in yanı sıra diğer popüler framework’lerin (java, php, nodejs, ruby, go vs.) hepsi benzer bir call back yapısına sahip midir, yüksek ihtimalle evet. Ancak dotnet’in **async/await** yapısının kodu sadeleştirdiği kadar sadeleştirebilirler mi açıkçası pek sanmıyorum. Burada biraz dotnet güzellemesi yapmış olduk, anti-dotnet’çiler kızmasın çünkü **async/await** bunu hak ediyor 😏

### Asenkron (Asynchronous) İletişim

Http’nin senkron çalıştığından ve call back mekanizmalarıyla bir şekilde client veya sunucu tarafında bir **asenkronizasyon **elde ettiğimizden bahsettik.

Servisler arası iletişim için Http haricinde kullanabileceğimiz, üstelik asenkron bir protocol olan **AMQP’**den** **(Advanced Message Queuing Protocol) kısaca bahsedelim.

Wikipedia tanımı;
> The **Advanced Message Queuing Protocol** (**AMQP**) is an open standard application layer protocol for message-oriented middleware.

**AMQP**’nin en önemli özelliklerini, mesaj yönlendirme, kuyruklama, routing (p2p ve pub/sub), dayanıklılık (güvenilirlik) ve güvenlik olarak sıralayabiliriz. AMQP, çok farklı yapıda ve birbirinden bağımsız çalışan sistemler arası iletişimi kolaylaştırdığı için, sistemlerin birlikte çalışabilirlik (**interoperability**) yönlerini güçlendirmemize de olanak sağlıyor.

**AMQP**’nin reliability (güvenilirlik) özelliğine ayrı bir parantez açmak gerekiyor. AMQP bu özelliğini, içerdiği 3 farklı teslimat garanti (**delivery guarantees**) modu ile kazanmıştır. Kısaca açıklamak gerekirse;

* **at-most-once : **Publisher mesajı **en fazla** 1 kere gönderir ve bu yöntemde mesajın kaçırılma riski vardır çünkü consumer’dan mesajı aldığına dair bir teyit beklenmez. RabbitMQ ve Kafka destekler.

* **at-least-once : **Publisher mesajı **en az** 1 kere gönderir ve bu yöntemde mesajın tekrarlı (duplicate) gönderilme riski vardır çünkü consumer’dan teyit alınırken bir hata meydana gelebilir. RabbitMQ ve Kafka destekler.

* **exactly-once : **Publisher’ın mesajı bir veya yalnız bir kere göndermiş olmasını garanti eder. Diğer iki yöntem kadar yaygın değildir. Daha spesifik, yani kısıtlı senaryolar için kullanışlı olabilir. Yalnızca Kafka destekler. **exactly-once** için daha fazla bilgiye [**buradan](https://jack-vanlightly.com/blog/2017/12/15/rabbitmq-vs-kafka-part-4-message-delivery-semantics-and-guarantees)** erişebilirsiniz.

Sisteminizin bu 3 yöntemden hangisini tolere edeceğine siz karar vererek uygun konfigrasyonu yapmalısınız. Örneğin, işlemleriniz **idempotent** yapıdaysa ve dolayısıyla bir mesajın kuyruktan 2 kere alınıp işlenmesi durumu sizin için sorun teşkil etmiyorsa **at-least-onc**e modunu seçebilirsiniz.

RabbitMQ, **AMQP**’yi destekleyen modern message broker’lara baktığımızda ilk aklımıza gelenlerden. Öyle ki, AMQP hakkında araştırma yaptıysanız karşınıza sürekli RabbitMQ’nün çıktığını görmüşsünüzdür. RabbitMQ hali hazırda MQTT, STOMP, AMQP gibi bir çok mesajlaşma protocol’ünü desteklemektedir. Sonraki bölümlerde RabbitMQ’den biraz daha bahsedeceğiz.

## Event-Driven Mimari

Microservice Mimari dünyasında en zor konulardan birisi data bütünlüğünün sağlanması ve herhangi bir** t** anında tüm data’mızın anlamlı yani beklenen bir **state’**de olmasının garanti edilmesi konusudur. Buna transaction bütünlüğünün sağlanması da diyebiliriz. Monolith mimariye göre kıyas götürmeyecek kadar zorlu bir süreç olduğu herkesin malumu.

[**Microservice Mimari’lerde Transaction Yönetimi Nasıl Yapılır](https://medium.com/devopsturkiye/microservice-mimarilerde-transaction-y%C3%B6netimi-nas%C4%B1l-yap%C4%B1l%C4%B1r-228317e248ed)** yazımda bu konuyu daha detaylı olarak incelemiştim, burada ise transaction yönetimi konusunun bizi **event-driven **mimariye götüren yanlarına değineceğiz.

### Event Derken?

Bu mimari’de event ile kastettiğimiz şey, bir servisin kendi domain’inde bir kaynağın durumunu değiştirmesiyle birlikte bu değişiklik bilgisini ilgili servis veya servislerle paylaşmasıdır. Bu paylaşımı da genelde bir mesaj kuyruk yapısı üzerinden yapar. Sürekli duyduğumuz **event fırlatma **tabirinin bu mimaride ki karşılığı “ben şu değişikliği yaptım, onu da şu kuyruğa veya kuyruklara yazdım ilgililere duyurulur” demektir.

Event-Driven mimariye hiçte yabancı değiliz aslında, gündelik hayatımız da bile bazı örnek senaryolar mevcut. Örneğin; canınız hamburger çekti ve soluğu bir hamburgercide aldınız ve sipariş için sizi ilk karşılayan görevliyle konuşuyorsunuz, siparişinizin detayını verdiniz ve beklemeye başladınız.

Siparişiniz hazır olana dek mecburen bekleyeceksiniz. Peki siparişinizi alan görevli? Sizinle beraber boş boş bekleyip, siz siparişinizi teslim aldıktan sonra sıradaki müşteriyle ilgilense nasıl olurdu? Elbette saçma olurdu, ancak **senkron **çalışan **request-driven **mimaride olan şey tam da bu 😏 Ama biz asenkron iletişim istiyoruz, kaynaklarımızı daha verimli kullanmak ve daha iyi performans için buna ihtiyacımız var. Devam edelim.

Siparişi sizden alan görevli bunu siparişi hazırlayacak olan kişiye iletir. (**order_created event’ı**) Ardından sıradaki müşteriyle ilgilenirken, siz boş bir masa bulup alarmın sizin için çalmasını beklersiniz. Siparişiniz bitince ilgili çalışan “x nolu sipariş hazır” bilgisini verir, (**order_ready event’ı**) ve sıradaki siparişi hazırlamaya başlar. **Order_ready** event’ının muhatabı olan çalışan size siparişinizin hazır olduğunu bildirir ve siz gider teslim alırsınız. (**order_delivered event’ı**) Çok uzatmamak adına bazı detayları atlamakla birlikte, gündelik hayattan basit bir **event-driven** süreç örneği vermiş olduk aslında.

Aşağıdaki örnek çizimde 4 adet microservice ve bir mesaj kuyruğumuz (event bus) mevcut. Buradaki okların tamamı iki yönlü olmak zorunda değildi. İki yönlü olması, o servisin aslında hem bir **producer **yani event oluşturan, hem de bir **consumer **yani event dinleyen(tüketen) olduğunu gösteriyor bize. Bir servis sadece producer, sadece consumer veya her ikisi de olabilir üstlendiği göreve göre.

Örneğin; örnek senaryomuzda hamburgeri hazırlayıp diğer görevliye teslim eden görevli, **order_created event**’ini dinleyerek, **order_ready event**’ini oluşturduğu için iki yönlü bir servistir diyebiliriz.

![Event-Driven Mimari Örnek Çizim](https://cdn-images-1.medium.com/max/2000/1*6Xcwdt8sB5-tq1GXihwC5A.png)*Event-Driven Mimari Örnek Çizim*

Event-Driven mimarinin avantajları olarak şunları sayabiliriz;

* Gevşek-Bağlı(loosely coupled) bir mimari oluşturmamıza olanak sağlar.

* Gevşek bağlılık servislerin **development **eforunu azaltır.

* İletişimimiz asenkron olacağı için performans kazanımı sağlar.

* Yatayda kolay ölçeklenebilirlik(scalability) sağlar.

* Mesaj kuyruğu sayesinde, consumer servis bir sebepten ötürü erişilemez durumdayken veri kuyrukta kalır ve tekrar erişilebilir olunca data kaybı yaşamadan kuyruğu tüketmeye devam eder. (Sonraki bölümde bahsedeceğimiz notifikasyon servis örneği)
> [**Four years from now, “mere mortals” will begin to adopt an event-driven architecture (EDA) for the sort of complex event processing that has been attempted only by software gurus building operating systems or systems management tools, and sophisticated developers at financial institutions**, predicted Roy Schulte, an analyst at Stamford.](https://www.computerworld.com.au/article/65656/event-driven_architecture_poised_wide_adoption/)

## Hybrid Mimari

Bu mimari adından da anlaşılacağı üzere bahsettiğimiz iki mimarinin birlikte kullanıldığı yapıları ifade eder. Eğer tamamen event-driven mimari ile kurulan bir sisteme sahipseniz veya en azından aşinalığınız varsa, “hybrid yapıya ne zaman ihtiyaç duyulur ki” diye düşünebilirsiniz. İlk bakışta gerçek hayat örneklendirmesi yapamayabilirsiniz doğal olarak. Neticede event-driven yaklaşım, request-driven kadar yaygın değil. En azından benim gördüğüm kadarıyla öyle. Bu yüzden genelde, request-driven ile inşa edilen sistemlere ihtiyaca göre bir **event-bus** sonradan entegre edilebiliyor.

![Hybrid Mimari Örnek Çizim](https://cdn-images-1.medium.com/max/2000/1*M6bABR6mMurV0qk23EfDuQ.png)*Hybrid Mimari Örnek Çizim*

Bir gerçek hayat örneği verecek olursak;

Request-Driven mimariyle kurgulanmış microservice’leriniz var. Uygulamanıza notifikasyon gönderme özelliğini ekleyeceksiniz. Çeşitli servisler kendi domain’lerinin gerektirdiği şekilde kullanıcılara doğrudan bazı notifikasyonlar göndermeliler. Hemen Notification Service adında yeni bir microservice oluşturdunuz ve request-driven yapınıza entegre ettiniz. Bu şekilde devam edebilirsiniz elbette.

Peki burada notification service kendisine gelen notifikasyon gönderim isteklerini bir kuyruktan okuyamaz mı? Diğer servisler RabbitMQ, Kafka gibi bir kuyruk yapısı üzerinden mesajları notification service’in dinlediği bi kuyruğa gönderebilir ve işlerine devam ederler. **Fire-and-Forget** dediğimiz iletişim şekli yani. Notification service ise notifikasyon isteklerini gönderen servislerle doğrudan iletişim içerisinde değildir artık ve isteğin kimden geldiğini de bilmek zorunda değildir. Artık tek yapması gereken bir kuyruğu dinlemek ve gelen datayı alarak notifikasyon gönderme işini icra etmektir.

Bu şekilde tasarlayarak bağımlılıklarımızı azaltmış olduk. Bir diğer avantajı ise, notification service’imiz bir sebepten ötürü erişilemez hale gelirse, iletişimi asenkron hale getirdiğimiz için servis tekrardan erişilebilir olduktan sonra kuyrukta birikmiş olan veriyi alarak notifilasyonları eksiksiz olarak göndermeye devam edecektir. Aynı senaryoda Request-Driven mimaride eğer bir **retry-policy** uygulamadıysak notifikasyonları kaybetmiş olacaktık ki, **retry-policy** uygulamamıza rağmen kaybetme ihtimali yine olabilir.

Son olarak, eğer microservice’lerinizi Openshift veya Kubernetes cluster’ları üzerinde yönetiyorsanız ve Istio kullanıyorsanız, [Emre Özkan](undefined)’ın [**Kiali](https://www.kiali.io/) **açık kaynak projesinden bahsettiği [**buradaki](https://medium.com/devopsturkiye/kiali-ile-microservicelerin-trafi%C4%9Fini-g%C3%B6rselle%C5%9Ftirin-8430e8835fbd)** yazısına da göz atabilirsiniz. Microservisler arasındaki trafiği görsel olarak sunduğu için değinmek istedim.

Bu yazıda Microservice Mimari’nin belki de en önemli konusu olan servisler arası iletişim hakkında bildiklerimi özetlemeye çalıştım. Konu elbetteki burada anlattıklarımdan çok daha fazlası. Bahsettiğim mimariler haricinde bildiğiniz başka alternatifler varsa yorumlarınızı beklerim.
