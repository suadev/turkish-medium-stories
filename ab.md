## Apache Bench (ab) Kullanarak Bir Web Servisin Yük Testi Nasıl Yapılır?

Apache Bench ile ( kısaca ab ) her türlü Http Server için yük testi (Load Test) yapmak mümkün olmakla birlikte, ilerleyen kısımlarda ab kullanımını örnek bir Asp.Net Core Web Api uygulaması üzerinden inceleyeceğiz.

Bir uygulama geliştirilmeye başlandığı andan itibaren aslında test süreci de başlamış oluyor. Burada ilk akla gelen ALM ( Application Lifecycle Management )’ in önemli bir bileşeni olan Test adımında görev alan mühendisler tarafından, belirli prensipler doğrultusunda yapılan testlerdir. Peki ALM’ in Test bileşeni insan eliyle yapılan bu testlerden ibaret midir? Aklınıza hemen Unit Test, Integration Test, Functional Test, Load Test vb. gibi kavramlar gelmeli. Bu kavramlardan her biri birbirinden önemli ve asla göz ardı edilmemesi gereken kavramlardır. Bu yazıda Yük Testi konusuna değineceğiz. Basit bir web api uygulaması üzerinden ab ’in kullanımı ve sonuçların yorumlanmasından bahsedeceğiz.

Eğer uygulamamız risk seviyesi çok yüksek bir uygulama değilse, örneğin; finansal işlemlerin döndüğü ve dakikada binlerce transaction ın execute edildiği ve yine dakikada binlerce transaction log un tutulduğu bir uygulamadan bahsedilmiyorsa, dürüst olalım ki Load Test konusu check list’ imizin sonlarında yer alıyor, hatta bazen hiç gündeme bile gelmiyor. Hatta, “Önce bir canlıya çıkalım da, ihtiyaç olursa ram, cpu takviyesi isteriz. Olmadı bir code-review & refactoring çalışması planlarız.” gibi düşünceler oluşabiliyor. Halbuki code-review ve refactoring gibi süreçler continuous olması gereken süreçlerdir deyip, daha fazla uzatmadan ab konusuna giriş yapalım.

Wikipedia da yer alan tanıma göre;

ApacheBench (ab) is a single-threaded command line computer program for measuring the performance of HTTP web servers.
Ab, Apache Http Server’ ın performans testlerini yapmak için bir tool’a duyulan ihtiyaçtan ortaya çıkmıştır ancak şuan herhangi bir Http Server için aynı amaçla kullanılabilmektedir.

Ab ile yük testlerimizi komut satırından yapıyoruz ve sonuçları da yine komut satırından görebiliyoruz. Öncelikle buradan sistemimize uygun olan apache versiyonunu indiriyoruz, ve zip’i açıyoruz. /bin/ab.exe dizinindeki .exe yi istediğimiz bir dizine kopyalıyoruz ve artık komut satırından ab ‘yi kullanabiliriz. Hemen bir ‘hello world’ yapalım ve sonuçları yorumlamaya çalışalım.

http://www.devnot.com adresine 100 adet isteği, 10 concurrency ( aynı anda 10 kullanıcı istek yapıyor olarak düşünebiliriz ) ile aşağıdaki gibi yapıyoruz.

<img src ="https://github.com/suadev/my-medium-stories/blob/master/ab1.PNG" />




