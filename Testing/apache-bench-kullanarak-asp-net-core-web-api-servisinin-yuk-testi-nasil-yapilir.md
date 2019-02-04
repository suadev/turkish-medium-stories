
# Apache Bench (ab) Kullanarak Bir Web Servisin Yük Testi Nasıl Yapılır?

Öncelikle, Apache Bench ile ( kısaca ab ) her türlü Http Server için yük testi (Load Test) yapmak mümkün olmakla birlikte, ilerleyen kısımlarda ab kullanımını örnek bir Asp.Net Core Web Api uygulaması üzerinden inceleyeceğiz.

Bir uygulama geliştirilmeye başlandığı andan itibaren aslında test süreci de başlamış oluyor. Burada ilk akla gelen **ALM (** **Application Lifecycle Management** **)**’ in önemli bir bileşeni olan **Test **adımında görev alan mühendisler tarafından, belirli prensipler doğrultusunda yapılan testlerdir. Peki** ALM**’ in **Test **bileşeni insan eliyle yapılan bu testlerden ibaret midir? Aklınıza hemen **Unit Test**, **Integration Test**, **Functional Test**, **Load Test** vb. gibi kavramlar gelmeli. Bu kavramlardan her biri birbirinden önemli ve asla göz ardı edilmemesi gereken kavramlardır. Zaten herkes çatır çatır **Unit Test** yazdığı ( yazıyorsunuz değil mi :) ) için,** **bu yazıda **Yük Testi** konusuna değineceğiz. Basit bir web api uygulaması üzerinden **ab **’in kullanımı ve sonuçların yorumlanmasından bahsedeceğiz.

Eğer uygulamamız risk seviyesi çok yüksek bir uygulama değilse, örneğin; finansal işlemlerin döndüğü ve dakikada binlerce transaction ın execute edildiği ve yine dakikada binlerce transaction log un tutulduğu bir uygulamadan bahsedilmiyorsa, dürüst olalım ki **Load Test** konusu check list’ imizin sonlarında yer alıyor, hatta bazen hiç gündeme bile gelmiyor. Hatta, “Önce bir canlıya çıkalım da, ihtiyaç olursa ram, cpu takviyesi isteriz. Olmadı bir** code-review & refactoring **çalışması planlarız.” gibi düşünceler oluşabiliyor. Halbuki **code-review** ve **refactoring **gibi** **süreçler **continuous **olması** **gereken süreçlerdir deyip, daha fazla uzatmadan **ab **konusuna giriş yapalım.

Wikipedia da yer alan tanıma göre;
> # ApacheBench (ab) is a single-threaded command line computer program for measuring the performance of HTTP web servers.

Ab, **Apache Http Server**’ ın performans testlerini yapmak için bir tool’a duyulan ihtiyaçtan ortaya çıkmıştır ancak şuan herhangi bir Http Server için aynı amaçla kullanılabilmektedir.

**Ab** ile yük testlerimizi komut satırından yapıyoruz ve sonuçları da yine komut satırından görebiliyoruz. Öncelikle [**buradan](https://www.apachelounge.com/download/)** sistemimize uygun olan apache versiyonunu indiriyoruz, ve zip’i açıyoruz. **/bin/ab.exe **dizinindeki .exe yi istediğimiz bir dizine kopyalıyoruz ve artık komut satırından **ab ‘**yi kullanabiliriz. Hemen bir ‘hello world’ yapalım ve sonuçları yorumlamaya çalışalım.

**http://www.medium.com** adresine 100 adet isteği, 10 concurrency ( aynı anda 10 kullanıcı istek yapıyor olarak düşünebiliriz ) ile aşağıdaki gibi yapıyoruz.

    ab -k -n 100 -c 10 [http://www.medium.com/](http://www.medium.com/)

![](https://cdn-images-1.medium.com/max/2000/1*ZmQbmBR5cPrZUzLX6JfXNQ.jpeg)

Ekran görüntüsünde odaklanılması gereken kısımları belirtmeye çalıştım. Etiketler yeterinde açık olsa da örneğimize geçmeden önce bu sonuç ekranını biraz yorumlamakta fayda var.

Burada kullanılan parametreleri açıklamakla başlayalım. Bu arada tüm parametreler için **ab** ‘nin [**buradaki](https://httpd.apache.org/docs/2.4/tr/programs/ab.html)** dokümantasyonuna göz atabilirsiniz.

**-n: **Belirtilen adrese yapılacak olan toplam istek sayısını belirtir.

**-c: Concurrency **(aynı anda gerçekleşme) ‘yi ifade eder. Burada verdiğimiz 10 değeri herhangi bir t anında medium.com adresine istek yapan kullanıcı sayısını belirtir. Stress testi yapılmak istendiğinde bu değer artırılabilir. Ancak buradaki optimum değerin ne olacağı konusu üzerinden düşünülmesi gereken bir konudur. Zira çok yüksek değerlerde **ab **‘nin kendisi bile **bottleneck **olabilir. Bu gibi durumlarda 2. hatta 3. bir **ab **instance’ı birlikte paralel olarak çalıştırılabilir.

**-k:** KeepAlive Http Header** **‘ının kullanılmasını sağlar. Böylelikle ilk istekten sonra connection sürdürülür. Browser’lar bu işlemi doğal olarak gerçekleştirdiklerinden, testler esnasında bu flag’in set edilmesi tavsiye edilir. Boolean bir parametre olduğu için herhangi bir değer almaz. Ekran görüntüsünde **-k** olmaksızın yapılan testlerde toplam test süresi **5.8 sn.** iken, **-k** parametresiyle yapılan testlerde yaklaşık **2.5 sn.** olduğu görülmüştür.

**-H: **Bu da bonus olsun :) Yapılan isteklere ekstradan http header eklemek için kullanılır. Bunu özellikle **token-based** çalışan api ‘ların testi için kullanıyorum. Örneğin, aşağıdaki şekilde bearer token set ederek api ’ınizin yük testlerini gerçekleştirebilirsiniz.

    ab -k -n 100 -c 10 **-H “Authorization: bearer <…token…>”** [http://api_ur](http://serviceurl/api/)l/

Örneğimizi daha eğlenceli hale getirmek için, **sync **ve **async **çalışan 2 farklı uç noktası oluşturacağız. Aslında bu makalenin konusu olmayan **async /await** in performansa olan olumlu katkısını da görmüş olacağız.

Öncelikle örnek uygulama kodlarına [**buradan** ](https://github.com/suadev/apache-bench-dotnet-core-load-test)ulaşabilirsiniz. Aşağıdaki kod bloğunda aynı işi yapan iki action method görüyorsunuz. Bunlardan birisi **async **diğeri ise **sync **çalışıyor. async/wait in performansa katkısını daha net görebilmek adına aynı isteği döngü içerisinde 5 defa gerçekleştiriyoruz. Burada test için basit ve yoğun matematiksel işlemler de yapabilirdik, ancak bu işlem **cpu-based** olacağı, dolayısıyla **awaitable **olmayacağı için tercih etmedik. Örnek servis parametre olarak aldığı kullanıcı adı için github api’ ınden ilgili kullanıcının profil bilgisini dönmektedir.

Bu arada, Http Client için **Asp.Net Core 2.1** ile gelen **HttpClientFactory **kullanılmıştır.

<iframe src="https://medium.com/media/6fcb4121b534e85c28638e920c6f0466" frameborder=0></iframe>

Sırasıyla önce sync, ardından async servisimize 100 adet istek göndereceğiz. Burada aslında **“10 kullanıcının eş zamanlı olarak toplam 100 adet istek yapması” **senaryosunu test ediyoruz.

İlk olarak sync uç noktamızı aşağıdaki gibi test ediyoruz.

    ab -n 100 -c 10 [http://localhost:5000/api/apachebench/get**sync](http://localhost:5000/api/apachebench/getsync)**?userName=suadev

![](https://cdn-images-1.medium.com/max/2000/1*dkwRiOvwYmtEAZVsxJxBZw.jpeg)

Aynı işlemi async uç noktası için yapıyoruz.

    ab -n 100 -c 10 [http://localhost:5000/api/apachebench/get**async](http://localhost:5000/api/apachebench/getsync)**?userName=suadev

![](https://cdn-images-1.medium.com/max/2000/1*f636MalQJxwH2H0mdJT4zw.jpeg)

**Sync **api ‘ımız için test **16 sn**. civarı iken, **async** tarafta **7.7 sn. **sürdüğünü görmekteyiz. Toplam transfer edilen datanın byte cinsinden aynı olduğu da görülmekte. Diğer değerlerde de yine yaklaşık olarak 1/2 gibi bir oran söz konusu.

Peki nasıl yorumlamalıyız?

Async servisimiz 10 eş zamanlı kullanıcı için saniye de 12.87 adet request işlemiş. Bu performans bizim için yeterli mi? Eğer cevabımız evet ise, -n ve -c parametreleri üzerinde artırıma giderek servisimizi zorlayabiliriz. Hatta stress testine doğru gidebiliriz. Tabi burada uygulamanın büyüklüğü, canlıya alındıktan sonra öngörülen kullanıcı sayıları vb. gibi kriterler **-n** ve **-c** nin değerinin ne olacağı konusunda önemlidir. Bu arada yukarıdaki testler esnasında **-k** parametresini geçmeyi unuttuğumu farkettim. :) **-k** parametresine değinmiştik hatırlarsanız, bir **Http Session** ile birden fazla istek yapabilmemizi sağlıyordu. Siz örnek uygulamayı indirip **-k** parametresiyle test ederseniz saniyede 12.87 adet isteğin üzerinde bir değer görmelisiniz.

Sonuç olarak **Apache Bench** ile aşağıdaki sorulara yanıt aramalıyız;

* Uygulamamın veya servisimin gelen çok sayıda eş zamanlı isteğe ortalama yanıt süresi nedir?

* Uygulamam saniyede maksimum kaç adet isteği işleyebilir?

* Uygulamamın kaldırabileceği yük ne kadardır?

**Not**: Ciddi stress altında çalışacak olan yoğun trafikli uygulamalar için yalnıza **ab **ile yapılan testlere bağımlı kalmamakta fayda var.

Bu yazıda yük testi hakkında bir farkındalık oluşturmayı ve **ab ‘**i giriş seviyesinde tanıtmayı amaçladım. Yük testi için başka alternatif tool’lar veya yaklaşımlar olabilir, paylaşmak isterseniz yorumlarınızı beklerim.

Uygulama kaynak kodlarına [buradan ](https://github.com/suadev/apache-bench-dotnet-core-load-test)erişebilirsiniz. Yazıda veya örnek uygulamada fark ettiğiniz hatalar için [buradan](https://github.com/suadev/apache-bench-dotnet-core-load-test/issues) hata kaydı açabilirsiniz.
