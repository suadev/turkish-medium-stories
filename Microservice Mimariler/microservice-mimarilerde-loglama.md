
# Microservice Mimari’lerde Loglama

image from memegenerator.net

Önceki yazılarımda da sık sık değindiğim gibi dağıtık sistemlerde bir çok sorun için tek ve kolay bir çözüm yoktur. Loglama konusu da bunlardan bir tanesi.

Bu yazıda loglama için mimarimizi nasıl oluşturmalıyız, **best practice**’ler nelerdir sorularına yanıt arayacağız. Burada bahsedeceğim konuların bir çoğu sadece **Microservices** uygulamalara özel şeyler olmayacak, dolayısıyla **monolith **mimaride uygulama geliştirenler için de ilgi çekici olacağını düşünüyorum. Konuyu iki ana başlıkta ele alacağız;

* Loglama Üzerine Genel Tavsiyeler

* Uçtan Uca Loglama Mimarisi

### Logalama Üzerine Genel Tavsiyeler

Microservice Mimari’leri dağıtık sistemlerin bir türü olarak ele alabiliriz. Dağıtık sistemlerde her bir servisi ve bu servisin üzerinde çalıştığı sunucuyu **trace** etmek **(**izlemek**), **beklenmedik durumlarda hatanın kaynağına hızlı ve etkili şekilde inebilmek için çok önemlidir.

Servislerimizin her birisi farklı bir sunucuda sunulurken, oluşan uygulama loglarımız da haliyle dağıtık yapıda olacaktır. Buna ek olarak bir de **cloud **üzerinde **auto-scaled** bir yapınız varsa, o zaman hangi log hangi sunucuda ve ne zaman oluşmuş gibi soruların yanıtını almak için biraz daha fazla efor harcamanız gerekiyor. Zira bildiğiniz gibi **auto-scale** yapıda yük durumuna göre dinamik olarak yeni sunucular çok kısa sürede devreye alınıp aynı şekilde kapatılabiliyor. Loglama mimarimizi, bu kapatılan sunucularda log kaybı yaşamamak üzere kurgulamamız gerekmekte. Yani kısaca cluster’ınızın auto-scale olup olmaması da mimariyi kurgularken göz önünde bulundurmamız gereken konulardan birisi olmalı.

### Logların Merkezileştirilmesi

Yapımız dağıtık bir yapı olduğundan, ilk düşünmemiz gereken konu tüm servislerimizin ürettiği logların tek yerden erişilebilir olmasıdır. Bir servisin logunu file’a, bir diğerinin DB’ye bir başkasının kuyruğa yazdığı bir ortamda oluşan bir hatanın kaynağına inmek saatler alabilir.

Merkezileştirme için çeşitli yöntemler mevcut. Mimari bölümünde biraz daha detaylı ele alacağımız için burada sadece kaçınmamız gereken bir **bad practice’**den bahsetmek isterim.

Microservice Mimari’yi uygularken, isteriz ki servislerimiz atomik olsun, yani sadece bir işe, bir domain’e odaklansın. Diğer servislere bağımlılığı sıfır noktasında olsun. Hal böyleyken her türlü ihtiyaç için irili ufaklı servisler oluşturmaya başlarız. İş loglamaya geldiğinde de yine bir http log service oluşturup loglama işini de bu servise yükleyerek diğer tüm servislerin bu servis üzerinden log atmasını isteyebiliriz. İlk bakışta her şey normalmiş gibi gözükse de servis sayısı, dolayısıyla üretilen log miktarı arttıkça loglama işlemi için oluşan http trafiği baş ağrısı olabiliyor. Yani bu maddeden çıkaracağımız ders, loglama işini üstlenen bir http service oluşturmaktan kaçınmamız gerektiği.

### Merkezileştirme İçin Aggregation Tool Kullanma

**Log Aggregation **araçları, tüm loglarımızı tek bir ortak lokasyonda ve benzer formatta birleştirme ihtiyacı üzerine ortaya çıkmıştır. Bu birleştirme işi için farklı yöntemler mevcut. Burada seçeceğiniz yöntem, servis loglarınızı nerede ve nasıl tuttuğunuza göre belirlenecektir.

Örneğin, siz her servisin kendi sunucusunda bir dizine .txt olarak log atmasına karar verdiniz. Bu logları birleştirme için bir job yazar, belirli aralıklarla tek ortak bir dizine tüm txt’leri toplayabilirsiniz. Tercih kötü olsa bile sonuçta bir **log aggregation** yapmış olursunuz aslında.

Daha doğru olan ise, bu iş için geliştirilen ücretli veya ücretsiz bir araç kullanmanız yönünde. **ELK (elastic search, logstash,kibana) **stack’inin aggregation aracı olan **logstash**’i önerebilirim mesela. ELK’dan, mimari bölümde bahsedeceğimiz için burada fazla detaya girmiyorum

### Log’daki Tüm Alanların Aranabilir Olması

Loglarınızı sorgularken bazı alanlar üzerinden daha sık sorgulama yapmanız gayet doğaldır. Ancak siz her durumda mevcut alanların tümü üzerinden sorgulama yapabilecek ve hızlıca yanıt alabilecek bir yapı oluşturmaya odaklanırsanız iyi edersiniz.

Hız için veritabanı üzerinde index oluşturma tabi ilk aklımıza gelen çözüm. **CustomerId**, **UserId**, **LogLevel **gibi alanlar üzerinden index’lenme olmazsa olmazdır, ancak kritik bir anda örneğin **InstanceId **alanı üzerinden sorgulama yapmanız gerekirse, dakikalarca beklemek biraz can sıkıcı olabilir.

Eğer çözüm index oluşturmaksa neden tüm alanlar için oluşturmuyoruz diye düşünebilirsiniz. Index’leme işi maliyetlidir. Çok fazla veya çok büyük boyutlu index’ler memory’de büyüklüğüne göre yer kaplayacaktır. Üstelik bununla da kalmaz, her bir insert/delete/update komutundan sonra bu indexlerin güncellenmesi gerekmektedir. Bu da sizin insert/delete/update sürelerinizin uzamasına yol açabilir.

Ancak bu index’leme işini uygulama veritabanında değil de ,bir log aggregation aracı üzerinde yaparsanız performans kaybının önüne geçebilirsiniz.

### Log Seviyesinin (Log Level) Dinamik Ayarlanması

Atılan her bir satır log’un mevcut kaynaklarınızı tükettiğini ve aynı zamanda log sorgulama maliyetinizi (süresini) artırdığını unutmamanız gerekiyor. Bu bağlamda, servislerimiz ayaktayken, **runtime’**da log seviyemizi dinamik olarak,uygulamamızı kesintiye uğratmadan değiştirmek isteyebiliriz. Elbette kullandığımız yardımcı kütüphanenin (varsa) bunu destekliyor olması gerekmekte.

Bu özelliği ağırlıklı olarak **Production** ortamımızda kullanabiliriz. Örneğin geçici bir süreliğine Debug seviyesinde log atılmasını isteyebiliriz. Servisimiz ayağı kalkarken en detaylı log seviyesini pasife çekip, sonrasında aktif hale getirerek aslında bize bir faydası dokunmayan, gereksiz bir sürü logdan kurtulabiliriz.

### Asenkron Loglama

Loglama işlemini hangi yolla yaparsanız yapın, log’u atacak olan birimin asenkron çalışması uygulama performansı açısından önemlidir.

Asenkron loglamada, loglamayı yapacak olan **logger thread**, o an yürütülmekte olan transaction’ın (http isteğinin) thread’ini block’lamayacağı için herhangi bir performans kaybına neden olmayacaktır. Ek olarak uygulamanızın loglama işleminin sonucuyla da ilgilenmemesi gerekiyor. Bir başka deyişle loglar **fire-and-forget** prensibi ile gönderilmelidir.

### CorrelationId Kullanımı

Bir transaction’ın icra edilmesi için arka planda 3–5 farklı web servisin birbirlerini tetiklediği bir senaryoyu ele alalım. Burada yapılan işlem aslında tek bir işlem. Örneğin bir satın alma işlemi. Ancak arka planda 5 farklı servis çalışmakta ve, 5 adet log atıldığını biliyoruz. Bu işlemin loglarına erişmek istediğimizde bu 5 adet log için bir grup numarası olsaydı ve bu no ile sorgulama yaptığımızda sadece bu 5 adet logu görebilseydik güzel olurdu değil mi?

Bu gruplamayı yapmak için **X-Correlation-Id** http header’ını kullanabiliriz. Bu header’a işlem bazında **unique **bir değer set etmemiz gerekmekte. Ek olarak log tablomuza da CorrelationId adında bir alanda eklememiz gerekiyor tabi. Artık geriye, bu header’ı her servis **request** ve **response’**u** **için doğru şekilde set etmek olacaktır. Örnek senaryomuzdaki 5 adet servisten her birisi bir sonraki servise bu CorrelationId değerini geçecektir. Kulaktan kulağa oyunu gibi düşünebilirsiniz.

Asp.Net Core ile uygulama geliştiriyorsanız [**buradaki](https://www.nuget.org/packages/CorrelationId/) **middleware işinizi görecektir. Tabi kendi interceptor’ınızı yazarak ta bu işi kolayca yönetebilirsiniz.

### Detaylı ve Anlaşılabilir Log

Gün içinde bir servisin loglarını inceleme gereği duyuyorsak, bir şeyler ters gidiyor demektir, tabi istisnai durumlar olabilir. Gerek development aşamasında gerekse production ortamımızda loglar tabiri caizse elimiz ayağımız oluyor. Tabi yeteri kadar detaylı bilgi verebildikleri zaman. Bu da yine bizim elimizde olan bir şey. Özellikle **error log**’ları mümkün olduğunca detaylı ve önemli bilgi içermelidir. Detaylı yanında önemli de dedim çünkü gereksiz bir sürü detay içeren ama hatanın kaynağına inmeye yardımcı olacak bilgiyi içermeyen logun kimseye bir faydası olmayacaktır.

Sözün özü, eğer loglarınızı incelerken “Keşke şu bilgi de elimizde olsaydı 😟” diye iç geçiriyorsanız, o bilgi için de ek bir alan açmanın zamanı gelmiş demektir.

### Log Zamanı İçin Utc Time Kullanımı

Özellikle servislerin cloud üzerinde sunulduğu senaryolarda, mevcut sunucuların veya yük oluşması anında devreye alınacak yeni sunucuların hangi time zone’da olacağıyla ilgilenmek istemezsiniz. Sizin için önemli olan sunucunun performansı ve yüksek erişilebilirliği olacaktır.

Ancak iş loglarınızı sorgulamaya gelince, log zamanının hangi time zone’a göre atılacağı konusu önem arz ediyor. Zamana göre order by ettiğiniz logların işlem sırasına göre hatalı olarak gelmesi istemezsiniz. Burada tavsiyem Utc time kullanmanızdır. Böylece çok farklı lokasyonlardaki servislerden atılan loglarda bile zaman karmaşası yaşamamış olursunuz. Bu arada eğer sizin için logu atan sunucunun local zamanı da önemliyse, log veri tabanınıza **TimeZone **adında opsiyonel bir alan ekleyip time zone bilgisini de buraya yazabilirsiniz.

### Instance Id Ekleme

Eğer servislerinizi herhangi bir cloud provider üzerinden sunuyorsanız ve auto-scaling özelliğini kullanıyorsanız, yukarıda da bahsettiğim gibi yük durumuna göre sürekli yeni sunucular (instance) eklenip çıkarılacaktır. Log’un hangi instance’dan atıldığı, logu atan sunucunun hala aktif olup olmadığı gibi konular sizin için önemliyse, logunuza **InstanceId** alanını ekleyerek bu sorunu çözebilirsiniz.

### Log Alert Sistemi

Loglama alt yapınız artık olgunlaştı, loglarınız sorunsuz şekilde akıyor, sorgular gayet hızlı çalışıyor ne mutlu size. Artık işi bir adım öteye taşımak gerekiyor.

Eğer canlı ortamda oluşan hatalarınız için biraz daha proaktif olmak ve daha hızlı aksiyon almak istiyorsanız mutlaka bir alarm sistemini devreye almanız gerekiyor. Bu sistemi sağlayan ücretli ücretsiz birçok **third party** araç mevcut olmakla birlikte kendi implementasyonunuzu da pek ala yapabilirsiniz tabi.

Bu tarz alarm yapılarıyla örneğin, gelen log daki **LogDetail **alanında “**DB Connection Timeout**” hatası geçiyorsa şu kişi veya kişilere slack’ten mesaj at veya sms at gibi aksiyonlar alabildiğimiz için oldukça faydalı olduğunu söyleyebilirim.

Alarm konusu için bir süre deneyimleme fırsatı bullduğum [**Seq](https://getseq.net/)** i incelemenizi öneririm. Tabi [**ELK ](https://www.elastic.co/elk-stack)**ile de veri değişikliklerini izleyerek aynı şekilde alarm kuralları oluşturabilmeniz mümkün.

## Uçtan Uca Loglama Mimarisi

Bu bölümde, yukarıda yaptığımız öneriler ışığında örnek bir mimari tasarlayıp, çizdiğim diagram üzerinden yorumlamaya çalışacağım.

Hatırlarsanız loglamayı bir servis üzerinden http request ile yapmanın kötü bir fikir olduğundan bahsetmiştik. Ancak aynı zamanda logları bir şekilde merkezileştirmemiz gerektiğini de söyledik. Yapmamız gereken şey çokta karmaşık değil aslında.

Her microservice, oluşturduğu logları aracı bir başka http servis olmaksızın asekron olarak bir kuyruğa gönderebilir. Buradaki kuyruk, **Apache** **Kafka **veya** RabbitMQ **gibi bir **message broker **olabileceği gibi, kuyruk kullanmadan doğrudan bir **RDBMS **veya **NoSQL** veritabanı da olabilir. Bu tamamen sizin log yoğunluğuna bağlı olarak yaptığınız tasarıma kalmış. Benim tavsiyem, **Kafka **veya **RabbitMQ **ikilisinden** **birini kullanmanız yönünde olur. Eğer doğrudan bir veritabanına yazacaksanız da bu RDBMS yerine bir NoSQL veritabanı olmalı.

Loglarınızı doğrudan bir kuyruğa göndererek veritabanı sunucunuzu ekstra ve büyük bir yükten kurtarmış olursunuz. Veritabanı sunucuunz sizin en değerli kaynağınızdır. ve unutmayın DB sunucuları ram’i sever yani bol bol kullanır.

Bir diğer aklıma gelen konu da uygulamanız ayağı kalkarken, henüz veritabanı bağlantısı yapılmadan önce loglamaya değer bazı bilgileriniz olabilir ancak veritabanı bağlantısı henüz sağlanmadığı için bu bilgileri loglayamayacaksınız.

Kuyruk yapıları AMQP protokolünü desteklerler. Bu protokol Http’nin aksine asenkron çalışır ve mesajları alıcısına teslim etme garantisi verir. Tabi sizin kuyruğu nasıl konfigüre ettiğiniz de önemlidir. Bir önceki [**yazımda](https://medium.com/devopsturkiye/microservice-mimarilerde-servisler-aras%C4%B1-i%CC%87leti%C5%9Fim-nas%C4%B1l-olmal%C4%B1-3d8db63b4dea)** AMQP’den daha detaylı bahsetmiştim dilerseniz inceleyebilirsiniz.

Tüm bu bilgiler ışığında çizdiğim aşağıdaki basit mimariyi siz de uygulamalarınız için rahatlıkla uygulayabilirsiniz.

![Microservice Uygulamalar İçin Loglama Mimarisi](https://cdn-images-1.medium.com/max/2586/1*egtisUJNs-xDyaN5Lw3TFQ.png)*Microservice Uygulamalar İçin Loglama Mimarisi*

Son olarak mimariyi oluşturan bu 5 parçadan kısaca bahsetmek gerekirse;

### Microservices

Farklı format ve yoğunluklarda loglar üreten ve kuyruğa gönderen mevcut microservislerimiz mimarinin ilk bacağını oluşturmakta. Dikkat ederseniz servisler ve kuyruk arasında herhangib aracı başka bir servis bulunmuyor.

### **Message Queue**

Microservice’lerimizin ürettikleri logları **publish **ettikleri kuyruğu tanımladığımız message broker’ımız. Log yoğunluğuna göre bu broker’ları da yatayda ölçekleyebilirsiniz yalnız, Kafka yatayda ölçeklenebiliyorken yanılmıyorsam RabbitMQ bu imkanı vermiyor. RabbitMQ da, iki farklı sunucu kurup aktif-aktif çalışmasını sağlayabiliyorsunuz ancak yükü 2 suncu arasında dağıtarak yatayda ölçekleneyim diyemiyorsunuz.

### **Aggregation Tool**

Aslında ELK bütün olarak bir log aggregation aracıdır demek de yanlış olmaz. **Logstash**, logları ilk karşılayan, isteğinize göre manipüle edebilen ve sonrasında Elasticsearch’e gönderen taraf olduğu için ben, Logstash ELK stack’inin log aggregation tool’udur demeyi tercih ediyorum.

### **NoSQL Document DB**

Elasticsearch, bir arama motoru gibi çalışmak üzere optimize edilmiş, NoSQL document tipi veritabanıdır. ELK stack’inde logların fiziksel olarak saklandığı, indexlendiği ve sorgulandığı yer burasıdır.

### Visualization Tool

ELK stack’ini tercih ettiğimiz için loglarımızı listelemek, sorgulamak ve çeşitli faydalı grafiklerden faydalanmak için bu stack’in sunmuş olduğu Kibana visualiztion tool’unu kullanıyoruz. Kibana, kullanışlı arayüzü, log sorgularken işimizi kolaylaştıran query syntax’ı, oluşturulan log raporlarını excel export edebilme gibi birçok özelliği nedeniyle Facebook, Microsoft, Linkedin gibi devler tarafından tercih ediliyor.

Geldik bir **Microservices **yazısının daha sonuna. Konuyla alakalı farklı öneri veya eleştirileriniz için yorumlarınızı beklerim.
