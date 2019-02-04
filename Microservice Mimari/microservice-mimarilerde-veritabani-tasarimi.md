
# Microservice Mimari’lerde Veritabanı Tasarımı

image from https://www.slideshare.net/MarineKaram/les-microservices-cest-pas-automatique-linkvalue-tech

Diğer **Microservices **yazılarım da olduğu gibi, bu yazının da monolith mimari ile uygulama geliştirenler için de faydalı olacak bilgiler içerdiğini belirterek başlamak isterim.

Gerek Monolith’den Microservice’e geçiş sürecinde, gerekse sıfırdan Microservice Mimari ile geliştirmeye başlayacağınız projelerinize başlamadan önce üzerinde uzun uzun düşünüp, tartışıp, araştırmalar yapmanız ve cevap aramanız gereken en önemli 3 soru şunlar olacaktır;

* Kaç adet servisimiz olacak?

* ‘**Şu**’ modülü/özelliği ‘**bu’ **servisten ayırıp, yeni bir servis mi oluşturmalı yoksa burada mı kalmalı?

* Hangi servisiçin hangi tip (RDBMS, NoSQL) veritabanı seçmeli?

Dikkat ederseniz, yazının konusu veritabanı seçimi olmasına rağmen servisleri nasıl ayrıştırmamız gerektiğinden bahsettik. Çünkü servisleri nasıl ayrıştırdığınız, bir servisin tek bir iş mi, yoksa birden fazla iş mi yapması gibi konular, doğrudan seçeceğiniz veritabanının tipine etki edecektir. İkinci bölümde örnek bir senaryo üzerinden incelediğimizde nasıl etki ettiğini daha net görmüş olacağız.

Konuyu üç ana bölümde ele alacağız;

* Veritabanı Tipleri ve Karşılaştırması

* Microservice Mimari’de Doğru Veri Tabanını Seçme

* Yanlış Seçim Yaptığımı Nasıl Anlarım?

## **Veritabanı Tipleri**

Konu bir seçim yapmaksa eğer, öncelikle alternatiflerimizin ne olduğunu bilmemiz gerekir. Bu alternatifler hakkında ne kadar derinlemesine bilgi sahibi olursak alacağımız kararlarda o kadar doğru olacaktır.

Bu bölümde ilişkisel veri tabanları ve yönetim sistemleri(RDBMS) ile ilişkisel olmayan (NoSQL-Not Only SQL) veri tabanlarından bahsedeceğiz. Her iki tip veri tabanından bahsettikten sonra kısa bir karşılaştırma yapacağız.

### İlişkisel Veri Tabanları ve Yönetim Sistemleri (RDBMS)

İlişkisel Veri Tabanları; veriyi diğer verilerle bir ilişki içerisinde tanımlayabilmemize ve erişebilmemize imkan sağlayan veri tabanlarıdır. Bir ilişkisel veri tabanında veri genellikle tablolar halinde tutulur. Tablolar satır ve sütunlardan oluşurlar. Bu tablo/satır/sütun yapısı, ilişkilerin kolay bir şekilde tanımlanabilmesine imkan sağlar.

İlişkisel veri tabanı yönetim sistemleri (**RDBMS**) ise ilişkisel bir veritabanı oluşturma, güncelleme ve yönetme işlerini yapan yazılımlardır. Çoğu RDBMS veri tabanına erişim ve etkileşim için **SQL **(Structured Query Language) dilini kullanmaktadır.

İlişkisel veri tabanlarında bir tablodaki kayıtlar birbirleriyle ilişkili olduğu gibi tablolar arasında da ilişki olabilir. Bu ilişkiler RDBMS’lerin** veri tutarlılığını **(**data consistency**) sağlamasındaki en önemli etkendir. Tabi burada ilişkilerin sizin tarafınızdan doğru bir şekilde tanımlanmış olması önemlidir.

### **Veri Tutarlılığına Basit Bir Örnek**

**Musteri **ve **MusteriDetay **isimli iki tablomuz olsun. Bu tablolar arasındaki ilişkiyi doğru olarak kurarsanız, **Musteri **tablosundan bir kayıt silerken size bu kaydın detay tablosundaki bir kayıtla bağlantılı olduğunu ve o kayıt silinmeden silinemeyeceğini söyler. Eğer bu ilişkiyi kurmamış olsaydınız, doğrudan **Musteri **tablosundaki bir kaydı silinebilir ve detay tablosundaki karşılığı silinmeden kalabilirdi. Bu kötü senaryonun gerçekleştiğini, yani **MusteriDetay **tablosunda var olan, ancak **Musteri **tablosunda hiçbir karşılığı olmayan detay satırlarının olduğunu düşünelim. İşte bu veri tutarlılığının bozulduğu duruma bir örnektir. Veri tutarlılığı, bu gibi durumların önüne geçilebilmesi için ortaya atılan bir kavramdır ve RDBMS’lerin vaad ettiği en önemli özelliklerindendir.

Bugün piyasaya baktığımızda en popüler ilişkisel veri tabanları olarak, MySQL (ücretsiz), PostgreSQL (ücretsiz), MSSQL, Oracle, IBM DB2 gibi veri tabanlarını sıralayabiliriz. Bu veri tabanlarından hepsi ilişkisel olmasına rağmen, bunlar arasında da yine seçim yaparken bilinçli bir seçim yapmak bizim faydamıza olacaktır.

Örneğin coğrafi bilgi sistemleri (GIS) ile alakalı bir proje geliştiriyorsanız, hiç düşünmeden PostgreSQL kullanmanızı öneririm. PostgreSQL içerdiği GIS spesifik veri tipleri ve eklentileri ( [https://postgis.net](https://postgis.net/) ) ile bu alanda öne çıkmaktadır.

GIS örneğini verme sebebim; Yapacağımız seçimin bir RDBMS / NoSQL seçimden ibaret olmadığını, bunlardan birisinde karar kıldıktan sonra kendi içlerindeki alternatifler arasından da yine doğru tercihi yapmamız gerektiğini vurgulamaktı.

### İlişkisel Olmayan Veri Tabanları (NoSQL)

İlişkisel veri tabanlarının geçmişi yaklaşık 40 yıl öncesine dayanır. Şüphesiz o zamandan bu zamana ilişkisel veri tabanları gelişim göstermektedir. Ancak yapı itibariyle RDBMS’ler büyük boyutta verilerle başa çıkmak üzere ve bir **cluster **üzerinde dağıtık yapıda çalışmaya pek uygun değillerdir. Büyük miktarda veri ile başa çıkma konusunda cluster üzerinde daha efektif çalışan bir veri depolama teknolojisine olan ihtiyaç günden güne artarken, NoSQL kavramı bu ihtiyaca bir çözüm olarak 1998 yılında ortaya atıldı.

NoSQL camiası, ağırlıklı olarak, yüksek miktarda veri ve yüksek trafiğe odaklandı. **NoSQL, RDBMS’lerin sahip olduğu güçlü ve anlık veri tutarlılığından vereceği taviz ile daha yüksek performans ve erişilebilirlik elde etmeye yöneldi.**

Peki neydi bu** anlık/hızlı veri tutarlılığı**? Önceki bölümde verdiğimiz veri tutarlılığı örneği anlık veri tutarlılığına da bir örnek aynı zamanda. İlişkisel veri tabanlarında anlık veri tutarlılığı (**immediate consistency**) vardır. NoSQL de ise bu anlık, yerini nihai veri tutarlılığına (**eventual consistency**) bırakır. Bu konuyu son bölümdeki **Yüksek Seviye Veri Tutarlılığı **alt başlığında bir gerçek hayat örneği ile ele alacağımız için burada kesiyoruz.

### **CAP Teoremi**

NoSQL’in performans ve erişilebilirlik için veri tutarlılığından taviz verdiğinden bahsettik. Taviz vermeden bahsetmişken, CAP teoremine de değinmek gerekiyor. Muhtemelen bir çoğunuzun daha önce bir şekilde denk geldiği düşündüğüm aşağıda görsel, teoremi tek başına açıklıyor aslında. Dikkat ederseniz ortada kırmızı bir çarpı işareti var, yani bu üç özelliğin tamamının bir NoSQL mimari de olamayacağını anlamamız gerekiyor. CAP’ın C,A,P sine birer cümle ile bakalım;

**Consistency**: Dağıtık sistemdeki tüm node’ların aynı veriye sahip olması durumu.

**Availability**: Sisteme yapılan her isteğin başarılı olsun başarısız olsun, bir yanıt alabilmesi durumu. (En güncel veriye sahip olmasa bile)

**Partition Tolerance**: Mevcut node’lardan bir kısmında network veya başka bir sebepten ötürü bir sorun meydana gelerek erişilmez hale gelse bile, sistemin çalışmasına devam edebilmesidir.

![image from [https://www.researchgate.net/figure/Visualization-of-CAP-theorem_fig2_282679529](https://www.researchgate.net/figure/Visualization-of-CAP-theorem_fig2_282679529)](https://cdn-images-1.medium.com/max/2000/1*Br1FrvKnK3hU6Xl_LbDkwg.png)*image from [https://www.researchgate.net/figure/Visualization-of-CAP-theorem_fig2_282679529](https://www.researchgate.net/figure/Visualization-of-CAP-theorem_fig2_282679529)*

Son olarak NoSQL veri tabanı çeşitlerine ve tercih edildikleri uygulama tiplerine bakacak olursak;

**Document Based** ( MongoDB, CouchDB, etc.) E-ticaret siteleri, İçerik yönetim sistemleri vb.

**Key/Value** ( Redis etc.) Kullanıcı oturum bilgisi saklama, Alış veriş sepeti verisi saklama vb.

**Graph Based **( Neo4J etc.) Sosyal medya uygulamaları, Graph tabanlı arama uygulamaları vb.

**Column Based** ( Cassandra, HBase etc.) Transaction loglama, IoT uygulamaları vb.

Tam listeye [**buradan](http://nosql-database.org/) **göz atabilirsiniz.

### RDBMS vs NoSQL

İki tür arasındaki farklılıklar çok ta karmaşık ve anlaşılması güç farklılıklar değil aslında. Bütün olay verinin nasıl saklandığı ve nasıl sorgulandığı meselesinden ibaret diyebiliriz. Şimdi alt başlıklar halinde karşılaştırmasını yapalım.

**Ölçeklenebilirlik**

RDBMS’lerinin yatay da ölçeklenebilmesi zor olduğundan, güçlü ve pahalı sunucularla dikeyde ölçeklendirme yoluna gidilir. NoSQL kolayca yatayda ölçeklenebileceğinden sunucu maliyetleri noktasında avantajlı **olabilir**.

Bu arada ölçeklenmenin zorluğundan bahsetmişken, geçenlerde Microsoft’un satın aldığı başarılı yerli girişim **Citus Data’**dan bahsetmek isterim. Citus Data aslında ilişkisel bir veri tabanı olan PostgreSQL’i dağıtık bir yapıda çalıştırmanıza imkan sağlıyor. Yani siz bir sorgu gönderiyorsunuz ve bu sorgu arka planda birden fazla node üzerinde paralel çalıştırılarak sonuç üretiliyor. Bu da yüksek performans demek tabi.

Açıkçası satın alma haberine kadar Citus Data’yı hiç duymamıştım. Bu teknoloji RDBMS’ler için devrim niteliğinde bir gelişme bana göre. Şöyle de bir blog post’a denk geldim, göz atmakta fayda var.

[https://www.citusdata.com/blog/2018/11/30/why-rdbms-is-the-future-of-distributed-databases/](https://www.citusdata.com/blog/2018/11/30/why-rdbms-is-the-future-of-distributed-databases/)

**ACID Prensipler**

NoSQL veri tabanlarının ACID olmadıkları yönünde aslında tam da doğru olmayan görüşler var. Önceki bölümde NoSQL veri tabanı tiplerinden bahsettik. Örneğin birçok Graph veri tabanı yapıları gereği ACID’dir. (Örneğin Neo4J) Peki ya diğerleri? CAP teoreminden bahsetmiştik. NoSQL veri tabanları çoğu zaman AP yi seçerek Strong Consistency’den taviz verebiliyorlar. Dikkat ederseniz strong ifadesini kullandım, yani eventual da olsa neticede bir Consistency sağlamış oluyorlar AP’yi seçtikleri zaman.

Dolayısıyla NoSQL veri tabanları ACID uyumlu değildir demek yanlış olacaktır. Kaldı ki ACID, Consistency’den ibaret değildir. ACID prensiplerden daha detaylı bahsettiğim [**buradaki](https://medium.com/devopsturkiye/microservice-mimarilerde-transaction-y%C3%B6netimi-nas%C4%B1l-yap%C4%B1l%C4%B1r-228317e248ed) **yazıma göz atabilirsiniz.

**Bakım Maliyetleri**

RDBMS’lerin bakım maliyetleri yüksektir ve özellikle büyük ölçekli sistemlerde eğitilmiş insan gücüne olan ihtiyaç NoSQL’e göre daha fazladır. Bu da eğer danışmanlık alınıyorsa daha fazla danışmanlık maliyeti anlamına gelmektedir. NoSQL veri tabanları daha az yönetim ve onarım maliyeti getirir. Maliyet konusunda NoSQL veri tabanlarının bir çoğunun open source oluşu da önemli bir etkendir. Lisans ücretleri noktasında RDBMS ile ciddi fark vardır.

**Olgunluk**

RDBMS’ler çok daha eskiye dayandıkları için daha geniş bir topluluğa ve yetişmiş insan gücüne sahip olmakla birlikte oldukça **stabil **çalıştıklarını söylemek yanlış olmaz. Ek olarak hemen hemen tüm RDBMS’ler ortak bir dil olan **SQL **ile veri tanımlaması ve manipülasyonu yaptığından, bu dilde uzmanlaşan birisi için veri tabanları arasında geçiş yapmak kolaydır.

**Big Data Uygulamalarında Kullanım**

Direkt olarak şu tip veri tabanı büyük veri uygulamaları için daha uygundur demek yanlış olmakla birlikte, bazı projelerde her iki tür birlikte kullanılabilmektedir. Veri, öncelikle performans ve ölçeklenebilirlik düşünülerek **unstructered** bir yapıda NoSQL veri tabanına kaydedilir ve asenkron olarak işlenerek **structered **bir yapı halinde ilişkisel veri tabanına yazılır. Ham verinin NoSQL de, işlenmiş olanın ise SQL de tutulması diyebiliriz yani. Böylelikle her iki veri tabanının da en önemli avantajlarından faydalanmış olunur.

**Veri Tutarlılığı (Data Consistency)**

RDBMS yüksek seviye veri tutarlılığı vaat ederken, NoSQL de durum böyle değildir. Önceki bölümde bir örnek vermiştik bu konuya ama bununla yetinmeyeceğiz ve bir sonraki bölümde vereceğimiz sosyal medya uygulaması örneği ile daha net anlaşılacağından eminim.

**Şema Bağımlılığı**

NoSQL şema bağımsız olduğundan veri formatı uygulamaya çok fazla dokunmadan da değiştirilebilir. RDBMS’ler de **change management** büyük bir sorun haline gelebiliyor, özellikle kötü tasarlanmış ve ilişkileri doğru olarak kurulmamış veri tabanları için.

Sonuç olarak; RDBMS’ler NoSQL veri tabanlarından iyidir veya tam tersi doğrudur gibi söylemler kesinlikle yanlıştır. Her ikisinin de diğerine göre daha avantajlı olduğu senaryolar mevcuttur. Burada bize düşen hangi durumda hangisinin daha avantajlı olduğunu bilmek ve doğru seçimi yapabilmektir.

## Microservice Mimari’de Doğru Veri Tabanını Seçme

Bildiğiniz üzere Microservice Mimari’yi uygulamak sizi bir programlama diline, framework’e, veri tabanına vs. bağlı kalmaktan kurtarıyor. Her bir servisinizi farklı farklı diller ve platformlarda bağımsız olarak geliştirerek yine bağımsız olarak canlıya alabiliyorsunuz. Tabi mimarinin temel prensiplerine sadık kalırsanız. 😏

Bu dil, framework ve veri tabanı bağımsızlığı konusunda en çok göz ardı edilen konu veri tabanı konusu olabilir. Özellikle monolith’den microservice’e geçişlerde, mevcut veri tabanı servisler özelinde parçalanırken, her microservice için mevcut monolith veri tabanı ile aynı veri tabanını kullanmak ilk seçenek oluyor ve genelde de böyle ilerleniyor. En azından gözlemlerim ve okuduklarım kadarıyla durum böyle. Konfor alanımızdan çıkmak istemeyişimiz aklıma gelen ilk sebeplerden. Bunun bir neticesi olarak ilk hatayı yapmak kaçınılmaz oluyor. Şöyle ki;
> **Servislerimizi seçtiğimiz veri tabanının türüne göre ayrıştırmaya başlıyoruz. Aslında yapmamız gereken bunun tam tersi olmalı. Servislerimizi veri tabanından bağımsız düşünerek, atomik ve en doğru şekilde tasarlayıp, veri tabanını ilgili servisin yapısına göre seçmeliyiz.**

İyi güzel de hangi kriterleri baz alarak seçim yapmamız gerekiyor diye düşünebilirsiniz. Bizim için en önemli iki parametre, yüksek veri tutarlılığı ve yatayda ölçeklenebilme konuşarıdır.

### **Yüksek Seviye Veri Tutarlılığı ve Ölçeklenebilme Gereksinimi**

Servisiniz için yüksek seviye **Consistency **önemliyse RDBMS kaçınılmazken, aksi durumda daha iyi performans için NoSQL tercih edebilirsiniz. NoSQL de **immediate consistency **yerine **eventual consistency** olduğunu söylemiştik.

Biraz açmak gerekirse; Örneğin bir sosyal medya uygulamanız var ve veri tabanı olarak NoSQL’i tercih ettiniz. (doğru tercih 👌)

Trafiğiniz çok olduğundan dolayı servislerinizi bir çok node(sunucu) üzerinde sunuyor, yatayda ölçekleniyorsunuz. Sosyal medya uygulamalarında paylaşılan içeriklere yapılan yorumlar veya beğeni sayılarının, tüm **node**’lar da herhangi bir **t** anında aynı değere sahip **olmaması **büyük bir sorun teşkil etmez. Şöyle ki;

Türkiye’de ki bir kullanıcı, bir paylaşımdaki beğeni sayısını 500 olarak görürken aynı anda aynı paylaşıma bakan Rusya’da ki kullanıcı bunu 495 olarak görebilir. Burada **Eventual Consistency** bir süre sonra sağlanacak ve tüm node’lar da aynı beğeni sayısı tutuluyor olacaktır. Biz bu gibi durumların oluşma ihtimalini bilerek, performans ve yüksek ölçeklenebilirlik için NoSQL tercihinde bulunduk ve doğru olanı yaptık.

**Immediate Consistecy** için ise finansal uygulamaları örnek gösterebiliriz. Parasal işlemler söz konusu olduğu için ilişkisel veri tabanlarının daha kararlı **ACID **özelliği önem kazandığından, genelde RDBMS’ler tercih edilmekte.

### Örnek Senaryo

Hatırlarsanız, servislerinizin yüklendikleri sorumluluğun boyutlarının doğrudan veri tabanı seçiminiz üzerinde etkisi olduğundan bahsetmiştik. Örnek bir senaryo üzerinden ne demek istediğimize geçmeden önce bir kaç kelam etmekte fayda var.

İdeal Dünya’da bir microservice’in yalnızca tek bir işi yapması ve o işi en iyi şekilde yapması istenir. **SOLID **prensiplerin ilki olan **Single Responsibility** prensibini bilirsiniz, sınıfları ve metotları mümkün olduğunda atomik tutarak onlara tek bir sorumluluk yüklemeye yöneltir bizi. Microservice Mimari’de servisleri tasarlarken bu prensibe bağlı kalmalıyız.

Servisimiz ne kadar büyür ve yaptığı iş ne kadar karmaşıklaşırsa, RDBMS kullanımı mecburiyet haline gelebilir. O zaman bizde ilişkisel veri tabanı kullanırız diye düşünebilirsiniz. Bu durumda RDBMS ile gelen **JOIN **kavramı sizi bekliyor demektir.JOIN’leme işlemi oldukça maliyetli bir işlemdir esasında. JOIN’lenen tabloların içerdiği veri miktarı arttıkça performansınız yerlerde sürünecektir, öyle ki index’ler bile sizi kurtaramayabilir.

Şimdi, servisleri **mümkün olduğunca **atomik tasarlayarak, NoSQL kullanmak mı? Yoksa ayırabileceğiniz servisleri ayırmayarak RDBMS kullanmak mı? Karar sizin. Bu arada konuyla ilgili Martin Fowler reisin [**buradaki](https://martinfowler.com/articles/microservices.html#DecentralizedDataManagement) **yazısına da göz atmanızı tavsiye ederim**.**

Gelelim örnek senaryomuza. Öncelikle hatalı bir tasarım ile veri tabanı seçimlerini yapacağız. Ardından daha doğrusu nasıl olur sorusunu sorarak, problemli noktaları saptayıp daha doğru bir tasarıma geçeceğiz.
> **Ön Not: **NoSQL / RDBMS seçimini yapmadan önce her servisin yalnızca kendisinin doğrudan erişebildiği izole bir veri tabanı olması gerektiğini bilmelisiniz. Bir servis başka bir servisin verisine ihtiyaç duyuyorsa bu ihtiyacını o servisin veri tabanına doğrudan erişerek **gideremez.** En doğrusu bir **event-bus** üzerinden **full asenkron **bir haberleşme yapılmasıdır. Eğer bu yapılamıyorsa, servisler bir birlerine anlık olarak **http call **yapabilir. Bu anlık http çağrıları, servisleri birbirine bağımlı kıldığı için çok doğru bir yöntem değildir ama başka bir servisin veri tabanına direkt erişmek kadar da hayati bir hata değildir en azından.

### **Uygulama**

Bir çok alt firması olan bir holding için ilişkisel veri tabanı kullanan monolith yapıda bir insan kaynakları uygulamamız olsun. Bu uygulamanın bir kullanıcı girişi bir de admin panel tarafı var.

Kullanıcılar sisteme giriş yaparak kişisel bilgilerini girme, izin talep etme, masraf fişi gönderme gibi işlemleri yapabildiği gibi, kalan izin gibi bilgilerini de **read-only** olarak görüntüleyebiliyorlar. Sistem adminleri ise, yeni kullanıcı, yeni departman ve yeni firma ekleme gibi işlemleri yönetim panelinden yapabiliyorlar.

Microservice mimariye geçiş için yazının girişinde sorduğumuz 3 soruyu sorarak başlıyoruz ve neticede aşağıdaki yanlış tasarımı yaptığımızı düşünelim. (Doğru tasarıma bakmadan yanlışlığın nerede olabileceğini tahminleyebilirsiniz.)
> **Not:** Bu örnek uygulamanın, işlevi ve kullanıcı sayısı göz önüne alındığında, büyük miktarda veri ve ölçeklenme gibi problemleri olmayacağı aşikardır. Dolayısıyla hatalı tasarım diye belirttiğimiz yapıda kalmasında bir mahsur olmayabilir. Siz bu hatalı ve doğru tasarımların , büyük ölçekli ve yüksek trafikli bir uygulamada yapıldığını hayal edebilirsiniz.

### **Hatalı Mimari**

Monolith uygulamamız MySQL veri tabanına sahip olduğu için microservice mimariye dönüşüm sonrasında tüm servislerimiz için yine MySQL ile devam etmek istedik. Sebebi neydi ki? Şimdi kim NoSQL’i araştırıp öğrenecek değil mi? 😏

Toplamda 6 adet servisimiz olsun dedik. Bunlardan 5 tanesinin kendisine ait izole bir veri tabanı var. Aşağıdaki gibi çizmeye çalıştım, idare edin artık.

![Hatalı Mimari](https://cdn-images-1.medium.com/max/2454/1*S4rDaKr_XNQsxXYs12KYMw.png)*Hatalı Mimari*

### **İyileştirilmiş Mimari**

Geliştirmeler ve testler esnasında fark ettik ki, User, Company ve Department servisleri bir birlerine çok fazla http isteği yapıyorlar, yani bağımlılık oranları bir hayli yüksek. Bu durum kullanıcı sayısı arttıkça daha büyük bir sorun haline gelecek gibi duruyor.

Dolayısıyla bu 3 servisi birleştiriyoruz ve yine ilişkisel bir veri tabanı olan MySQL ile devam ediyoruz. Expense (masraf) ve Annual Leave (izin) servisleri ise bağımsız kalmaya devam ediyorlar. Ancak ilişkisel bir veri tabanı kullanmamızı gerektiren bir durum olmadığını düşünerek, MongoDB ile yola devam diyoruz. Mimarinin son hali kabaca şöyle şekilleniyor;

![Doğru Mimari](https://cdn-images-1.medium.com/max/2470/1*e7eP7BrUot_Xqd00t2Bwtw.png)*Doğru Mimari*

**Ne Yaptık?**

Servis sayımızı azaltarak, 2 servislik bir DevOps yükünden kurtulmuş olduk. Bununla kalmadı tabi, bir birleri arasında çok yoğun bir http trafiği oluşturan 3 servisin meydana getirdiği bu trafiği de bitirmiş olduk. Eğer MySQL yerine ücretli olan MsSQL gibi ücretli bir veri tabanı kullanmış olsaydık ki özellikle ülkemizde çok yaygın bir kullanıma sahip, bu lisans ücretinden de tasarrufa gitmiş olacaktık.

**Daha İyisi Olamaz mı?**

Her zaman olur 😏Burada dikkat ettiyseniz servisler bir birlerine doğrudan http call ile erişmekte. Full asenkron ve daha gevşek bağlı bir tasarım için RabbitMQ gibi bir message broker kullanmamız gerekir. Servisler arası iletişim konusuyla alakalı daha detaylı bilgi için [**buradaki](https://medium.com/devopsturkiye/microservice-mimarilerde-servisler-aras%C4%B1-i%CC%87leti%C5%9Fim-nas%C4%B1l-olmal%C4%B1-3d8db63b4dea) **yazıma göz atabilirsiniz.

## Yanlış Bir Seçim Yaptığımı Nasıl Anlarım?

Her iki tip veri tabanı içinde kısaca ip ucu vermek gerekirse;

Örneğin, Doküman tabanlı NoSQL veri tabanı kullandığınız bir servisiniz de, transaction kullanımına ihtiyaç duyuyorsanız, aynı anda birden fazla dokümanı güncelleme gibi bir ihtiyacınız oluyorsa, NoSQL’in kapsama alanından çıkmış, ilişkisel veri tabanı Dünyasına girmişsiniz demektir.

Benzer şekilde, ilişkisel veri tabanı kullandığınız servisiniz de, tabloları JOIN leme, transaction gibi özellikleri hiç kullanmıyorsanız ve büyük miktarda veri söz konusuysa ki bu durumda yatayda ölçeklenme kaçınılmazdır, o halde NoSQL kullanmak daha doğru bir tercih olacaktır.

## Kapanış

NoSQL veri tabanlarını RDBMS’lere ucuz bir alternatif olarak düşünerek kullanmak hata olur. Ne olduklarını ve hangi yaraya merhem olduklarını bilerek, bilinçli bir seçim yapmak zorundayız.

Eğer farklı görüş veya önerileriniz varsa lütfen yorum olarak belirtin. Hatta veri tabanı ve microservices konusunda bilgisine güvendiğiniz kişilere mention atabilirseniz, doğru bildiğimi zannettiğim ama aslında yanlış olan noktalar varsa düzeltme ve öğrenme fırsatım olabilir. ✋
