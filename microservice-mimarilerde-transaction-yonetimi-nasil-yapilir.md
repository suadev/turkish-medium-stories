
# Microservice Mimari’lerde Transaction Yönetimi Nasıl Yapılır?

image from makeameme.org

[Bir önceki yazım](https://medium.com/devopsturkiye/microservice-mimarilerde-integration-test-nas%C4%B1l-yaz%C4%B1l%C4%B1r-e6b45daa7914)’ı yayınladıktan sonra [Burak Selim Şenyurt](undefined) hocamın da önerisi üzerine transaction bütünlüğü üzerine bir şeyler yazmaya karar verdim. Yazarken hem bilgilerimi pekiştirdim hem de yeni şeyler öğrendim, umarım sizlere de faydası dokunur.

Bu arada yazının başlığı için 3–5 farklı seçenek arasından seçim yapmak durumunda kaldım. **Transaction Yönetimi**, **Transaction Bütünlüğü**, **Veri Tutarlılığı **(Data Consistency) vb. gibi kavramların aslında aynı kapıya çıktığını söyleyebiliriz.

Konuyu aşağıdaki başlıklara bölerek anlatmayı uygun buldum;

* Transaction ve Transaction bütünlüğü nedir?

* ACID prensipler hakkında

* Monolith uygulamalarda transaction yönetimi

* Microservice Mimari’lerde transaction yönetimi

* Microservice Mimari’lerde transaction yönetimi için **Two-Phase Commit** ve **Saga **tasarım kalıpları

* **Two-Phase Commit** vs. **Saga**

### Transaction Kavramı

Transaction kelime anlamı olarak iş/işlem anlamına gelmekle birlikte kullanıldığı alana göre farklı anlamlar kazanabilmekte. Bankacılık sektöründe, yapılan bir EFT için kullanılırken, muhasebe dünyasında deftere yapılan her bir yazma işlemi için kullanılabilir. Veritabanı üzerinde yapılan işlemlerin her birisi bizim için bir **transaction**’dır.

Bazı business transaction’lar, birden fazla transactionın çalışmasını gerektirebilir. Eğer microservice mimari söz konusuysa, bu aslında birden fazla servisin ard arda çalışması anlamına gelir. Bu arda arda çalışan transaction’lar dizisinin yönetilmeye ihtiyacı vardır. Yönetiminden kastımızın ne olduğuna bir örnek senaryo üzerinden bakalım.

Bir e-ticaret sitesinde bir ürünün siparişinden müşteriye teslim edilmesine kadar geçen sürede bir çok sürecin dolayısıyla transaction’ın işletildiğini tahmin etmek zor değil.

Örneğin ödeme işlemi ve sonrasında ürünün stoktan düşülmesi süreçlerini ele alalım. Ödeme işlemi başarılı olmadan, stoktan düşme süreci ve sonraki süreçler işletilemez. Peki ödeme işlemi başarılı olduktan sonraki süreçlerin birisinde bir hata meydana gelirse ne yapmalıyız? Yazılım tarafında bu durumu nasıl yöneteceğiz? Bu hata oluştuktan sonra o ana kadar veritabanı üzerinde yapılmış olan işlemlerin tümünü geri almak gibi bir sorunumuz var. İşte bu sorun ve çözümü **transaction bütünlüğü/tutarlılığı/yönetimi **konusunun temelini oluşturmakta.

### ACID Prensipler

ACID, değişikliklerin bir veritabanına nasıl uygulanacağını yöneten 4 adet prensip sunar. Bunlar, **Atomicity**, **Consistency**, **Isolation **ve **Durability **prensipleridir**. **Bir kaç cümle ile açıklamak gerekirse;

**Atomicity**: En ksıa ifadesiyle ya hep, ya hiç. Arda arda çalışan transaction’lar için iki olası senaryo vardır. Ya tüm transaction’lar başarılı olmalı ya da bir tanesi bile başarısız olursa tümünün iptal edilmesi durumudur.

**Consistency**: Veritabanındaki datalarımızın tutarlı olması gerekir. Eğer bir transaction geçersiz bir veri üreterek sonuçlanmışsa, veritabanı veriyi en son güncel olan haline geri alır. Yani bir transaction, veritabanını ancak bir geçerli durumdan bir diğer geçerli duruma güncelleyebilir.

**Isolation**: Transaction’ların güvenli ve bağımsız bir şekilde işletilmesi prensibidir. Bu prensip sıralamayla ilgilenmez.Bir transaction, henüz tamamlanmamış bir başka transaction’ın verisini okuyamaz.

**Durability**: Commit edilerek tamamlanmış trasnaction’ların verisinin kararlı, dayanıklı ve sürekliliği garanti edilmiş bir ortamda (sabit disk gibi) saklanmasıdır. Donanım arızası gibi beklenmedik durumlarda transaction log ve alınan backup’lar da prensibe bağlılık adına önem arz etmektedir.

### Monolith Uygulamalarda Transaction Yönetimi

Monolith mimaride transaction yönetimi Microservice Mimariye kıyasla oldukça kolaydır. Bir çok framework veya dil transaction yönetimi için kendi içlerinde bazı api’lar içerirler. (dotnet için **TransactionScope** class’ı gibi) Bu api’lar tüm uygulamanın tek bir veritabanına sahip olduğu, dolayısıyla tüm transaction’ların tek bir context üzerinde çalıştığı senaryolar için geliştirilmişlerdir. Yani monolith mimarilerde bu api’lar ile basitçe **commit **ve **rollback **işlemlerini yapabiliyoruz.

**Commit **işlemi **scope**’a dahil edilen tüm transaction’lar başarıyla çalıştığında en son yapacağımız işlem iken, **rollback **ise scope’dak herhangi bir transaction’da bir hata oluşması durumunda tüm işlemi iptal etmek için kullanılır.

Transaction scope içerisinde işletilen transactionlar **commit **edilene kadar diske yazılmadan memory’de tutulurlar ve eğer herhangi bir **t** anında **Rollback **yapılırsa, scope içerisinde o ana kadar işletilmiş tüm transactionlar memory’den silinerek işlem iptal edilmiş olur. **Rollback **yapmadan, **Commit **edildiğinde ise diske (veritabanına) yazılarak transaction başarıyla tamamlanmış olur.

Aşağıdaki ekran alıntısında asp.net core da **TransactionScope **kullanım örneğini görebilirsiniz. Burada söz konusu transaction 2 step’li bir transaction’dır. Örneğin ikinci transaction’da meydana gelecek bir hata **Complete** metodunun çalışmaması, dolayısıyla birinci işlemin de iptaliyle sonuçlanacaktır.

![Asp.Net Core transaction yönetimi](https://cdn-images-1.medium.com/max/2000/1*CUkCwUjQp5nG5PNP4k1VaQ.png)*Asp.Net Core transaction yönetimi*

### Microservice Mimari’de Transaction Yönetimi

Microservice Mimaril’er dağıtık mimarilerdir ve dağıtık mimaride bir çok konu için tek ve kolay bir çözüm genelde yoktur. Kimlik doğrulamadan, loglamaya, caching’den integration test yazmaya kadar bir çok konu uzmanlık gerektiren zor konular olarak karşımıza çıkmakta. Transaction yönetimi konusunu da bu konulara dahil ettiğimizi söylememe gerek yok zira yazının konusu bu 😏

Microservice Mimari’lerde yukarıda bahsettiğim ACID prensiplerini korumak kolay bir iş değildir ve birden fazla yol mevcuttur diyebiliriz. Burada yalnızca **2PC **ve **Saga **konularına değineceğiz.

### Two-Phase Commit (2PC)

2PC, dağıtık mimarilerde yukarıda bahsettiğimiz ACID prensipleri korumaya imkan sağlayan bir protokoldür. İsminden de anlaşılacağı üzere 2 fazdan oluşmaktadır ve bu 2 fazı yöneten bir **Kordinatör**’ ümüz mevcuttur. İlk faz **prepare **(hazırlık veya oylama (voting)olarak da geçer), ikinci faz ise **commit **fazı olarak adlandırılır.

Yine e-ticaret örneği üzerinden açıklayalım. Ödeme ve stoktan düşme transaction’larını ele almıştık. Akış başladıktan ve transaction’lar tamamlandıktan sonra **Kordinatör, **hazırlık fazında bu 2 transaction’ın başarılı olup olmadığını yani **commit **işlemi için hazır olup olmadıklarını sorar. Eğer her iki transaction’dan commit için hazırız yanıtını alırsa 2. yani **commit** fazını icra ederek, işlemlerin kalıcı olarak diske yazılmasını sağlar.

Hata senaryosuna gelecek olursak; **Kordinatör**, birinci faz sonunda transaction’lardan **birisinden bile** commit edilemez yani hata oluştu bilgisini alırsa mevcut tüm transaction’ları iptal eder. Böylece işlem bir bütün olarak iptal edilmiş olur.

### Saga Pattern

Dağıtık mimarilerde transaction yönetimi için en bilindik yöntemlerden birisi olan **Saga Pattern, **ilk olark 1987 yılında** [akademik bir makalede](https://www.cs.cornell.edu/andru/cs711/2002fa/reading/sagas.pdf) **ortaya atıldı.

**Saga**, her transaction’ın farklı ve bağımsız bir servis üzerinde lokal olarak çalıştığı ve yine o servis içerisinde verisini güncellediği transaction’lar dizisidir. Bu tasarım kalıbına göre, ilk transaction, dış bir etki ile (kullanıcının kaydet butonuna tıklaması gibi ) tetiklenir ve artık sonraki tüm transaction’lar bir önceki transaction’ın başarılı olması durumunda tetiklenecektir. Transaction’lardan herhangi birisinde meydana gelecek bir hata durumunda ise tüm süreç iptal edilerek **Atomicty **presibine bağlılık sağlanmış olur. Biraz havada kalmış olabilir ancak aşağıdaki örnek senaryo çizimlerimle biraz daha net anlaşılacağını düşünüyorum.

Saga’yı uygulamak için bir kaç farklı yöntem mevcuttur. Ben **Events/Choreography **metodu ile Saga’yı servislerimiz arasında nasıl implemente edeceğimizden bahsedeceğim. Dilerseniz diğer bilindik yöntem olan **Command/Orchestration **metodunu da araştırabilirsiniz. Eğer vakit ayırabilirsem belki bir yazı da orkestrasyon yöntemi için yazabilirim. Bu arada bu iki yöntem haricinde deneyimleyebildiğiniz bir yöntem biliyorsanız yorum bölümünden paylaşabilirsiniz.

### **Events/Choreography Yöntemiyle Saga Uygulaması**

Bu yöntemde ilk servis işini icra ettikten sonra bir **event** fırlatır ve bu event’ı dinleyen servis veya servisler tetiklenerek kendi local transaction’larını çalıştırır. Yani her servis aslında bir önceki servisin “ben işimi hallettim sıra sende” demesini bekler. Son servis çalıştıktan sonra artık bir event fırlatmaz ve süreç sonlanır.

**Events/Choreography **yöntemi**, **Saga’nın prensiplerine en uygun olan yöntemdir. Uygulaması ve yönetmesi çok kolaydır. Transaction’lar birbirlerinden tamamen izoledir ve birbirleri hakkında bilgi sahibi olmak zorunda değildirler. Ancak bu yöntemde servislerimizin ve dolayısıyla event’larımızın sayısı arttıkça sistemin karmaşıklığının da artacağını ve yönetilmesi zor bir hal alabileceğini unutmamak gerekiyor**.**

Yine sipariş verme örneğimiz üzerinden başarılı ve başarısız sipariş işlemlerini, iki farklı akış diagramı üzerinden inceleyelim.

**Örnek Başarılı İşlem Senaryosu (Commit)**

Senaryomuz gayet basit. Hatırlarsanız ilk transaction’ımız harici bir müdahale ile tetikleniyordu. Kullanıcının ekrandan **Satın Al** butonuna tıklamasıyla;

* Saga’mızın ilk transaction’ı yani **Sipariş Servisi** tetiklenir.

* Sipariş servisi **Sipariş Oluşturuldu** Event’ını fırlatır.

* Bu event’ı dinleyen **Ödeme Servisi** tetiklenir.

* Ödeme servisi **Ödeme Alındı** Event’ı fırlatılır.

* Bu event’ı dinleyen **Stok Servisi** tetiklenir.

* Stok servisi **Stoktan Düşüldü** Event’ını fırlatır.

* Bu event’ı dinleyen **Bildirim** **ve Sipariş Servisleri** tetiklenir.

* Bildirim servisi kullanıcıya e-mail/sms gönderir.

* Sipariş Servisi siparişin durumunu** Başarıyla Tamamlandı **durumuna günceller.

![Başarılı sipariş Saga implementasyonu](https://cdn-images-1.medium.com/max/2000/1*TQ_dRqOK0ATd0r4SSdHM0Q.png)*Başarılı sipariş Saga implementasyonu*

**Örnek Başarısız İşlem Senaryosu (Rollback)**

Kullanıcının ekrandan **Satın Al** butonuna tıklamasıyla;

* Saga’mızın ilk transaction’ı yani **Sipariş Servisi** tetiklenir.

* Sipariş servisi **Sipariş Oluşturuldu** Event’ını fırlatır.

* Bu event’ı dinleyen **Ödeme Servisi** tetiklenir.

* Ödeme servisi **Ödeme Alındı** Event’ı fırlatılır.

* Bu event’ı dinleyen **Stok Servisi** tetiklenir.

* Stok servisi **Ürün Stokta Yok Hata **Event’ını fırlatır.

* Bu event’ı dinleyen **Ödeme ve Sipariş Servisleri** tetiklenir.

* Ödeme servisi kullanıcıya para iadesi yapar.

* Sipariş Servisi siparişin durumunu** Başarısız **durumuna günceller.

![Başarısız sipariş Saga implementasyonu](https://cdn-images-1.medium.com/max/2000/1*jYXcPwGbIyL428GCxxgERw.png)*Başarısız sipariş Saga implementasyonu*

**Diagramları Yorumlayalım**

Öncelikle **Event Fırlatma **ifadesine aşina olmayanlarınız için biraz hava da kalmış olabileceğinden buna değinelim. Diagramlarda berlittiğim Event’ların her birisi aslında veritabanına veya bir message queue’ya atılan bir kayıttan ibaret. Burada veritabanı mı yoksa mesaj kuyruğu mu olacağı sizin tasarımınıza kalmış. Eğer ciddi yük altında çalışan bir uygulama söz konusuyla RabbitMQ, Msmq veya Kafka gibi mesaj kuyruk yapıları kullanmanızı öneririm.

Gelelim **Event Dinleme **olayına**. **RabbitMQ Message Queue** **kullanılan bir mimaride** **açıklayacak olursak. Oluşan her event bir kuyruğa yazılır. Ödeme servisi **Sipariş Oluştu **event’ını dinliyor demek, bu event’ın yazıldığı kuyruğa atılan her yeni mesajın Ödeme Servisi tarafından **Subscribe **edilmesi yani tüketilmesi demektir. Her servis için ayrı bir kuyruk olduğunu hayal edin. Hangi event’ın hangi kuyruk veya kuyruklara yazılacağını RabbitMq’nun gelişmiş routing yapısıyla kolayca yönetebiliyorsunuz. Başarılı ve başarısız senaryo diagramlarını açıklayalım biraz daha net anlaşılacaktır.

Başarılı senaryo için çok fazla söylenecek bir şey yok aslında. Yalnızca **Stoktan Düşüldü** event’ına dikkatinizi çekmek isterim. Dikkat ederseniz bu event’ı dinleyen iki servis var. Sipariş servisi ve bildirim servisimiz. Birkaç cümle önce** **‘**kuyruk veya kuyruklara’ **ifadesini kullanmıştık yani biz RabbitMQ’ya bir mesaj gönderirken bu mesajı birden fazla kuyruğa yaz diyebiliyoruz.

Gelelim başarısız senaryoya;

Stok servisine kadar her şey yolundaydı, ancak kullanıcının ekranında **Stokta Var** olarak gördüğü ürünün aslında stokta mevcut olmadığı ortaya çıktı ve neticede stok servisi **Stoktan Düşme Hata** event’ını fırlattı. Dikkat ederseniz yine iki servis tarafından dinleniyor bu event. Birisi başlangıç yani sipariş servisimiz, diğeri ise bir önce çalışan ödeme servisimiz. Yani aslında kural olarak, “**Hata durumlarında ilk servis ve bir önceki servis için event fırlatılır**.” diyebiliriz.

Tabi olay bu kadar da basit değil. Örneğin, ödeme servisinin hata alması durumunda sadece ilk servis için event oluşturulması yeterli olacakken, bildirim servisi için farklı bir tasarım yapmak gerekebilir. Neticede bildirim servisi **Saga **akışını bozan, işlemi sekteye uğratan bir servis değil. Yani oluşacak bir hatanın başka bir servis tarafından dinlenmesi gerekmeyebilir. Bildirim servisinde oluşacak hatalar için **retry policy’**ler tanımlayabiliriz.

### Saga mı? 2PC mi?

Yoruma açık bir konu olmakla birlikte, ben her durumda Saga’yı uygulamayı tercih ederdim. Çünkü 2PC ile gerçekleştireceğiniz her senaryoyu Saga ile de yapabilirken, tersi her zaman mümkün olmayabilir.

Saga’yı **Long Running Transaction** dediğimiz, transaction’ı birbirinden bağımsız ve asenkron çalışabilen step’lere bölerek yönetmenin doğru olduğu senaryolarda kullanırken, **2PC** nisbeten daha hızlı sonlanan ani transaction’lar için tercih edilebilir.

Saga’da bir sipariş işlemini step’lere (**sipariş-ödeme-stok-email vs..**) bölerek yönetmek kolaylık sağlıyor. Bunun yanında Saga’da her step’ten sonra commit işlemi yapıldığından, yani sonuç dış dünyaya gerçek olarak yansıdığından dolayı (**ödeme adımında müşteriden ödeme alınması** **gibi**) rollback işlemi daha kritik bir hal alıyor. Ödeme işleminin rollback yapılması yani müşteriye para iadesi yapılması esnasında oluşabilecek bir hata durumunda nasıl aksiyon almamız gerekir? Bu konu belki de Saga’nın en kritik konusu olabilir.

Geldik bir **Microservices **yazısının daha sonuna. Sizler Microservice Mimari’de veri bütünlüğünü nasıl sağlıyorsunuz? Deneyimlerinizi veya önerilerinizi yorum olarak iletirseniz çok memnun olurum.
