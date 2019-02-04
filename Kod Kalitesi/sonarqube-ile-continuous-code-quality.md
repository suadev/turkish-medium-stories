
# SonarQube İle Continuous Code Quality

Bir önceki yazımda ( Yazılım Ekiplerinde Kod Kalitesi ve Kalitede Sürekliliğin Sağlanması ) Continuous Code Quality kavramından, DevOps kültüründeki, ve bir CI pipleline’ındaki yerinden bahsetmiştim. Özellikle büyük veya büyüme kapasitesi olan ekiplerde her commit’in ana branch’e merge edilmeden önce mutlaka code-review işlemine tabi tutulması gerektiğini ve bu işlemden önce bir statik kod analiz aracı kullanımının, kod kalitesi, hatasızlık ve zaman tasarrufu açısından faydalı olacağından bahsetmiştim. Bu yazımda SonarQube statik kod analiz aracından bahsedeceğim.

![](https://cdn-images-1.medium.com/max/2000/1*yzNDZgOLC9TeszIfInKlBw.png)

**SonarQube **(eski ismiyle Sonar), [**Sonarsource ](https://www.sonarsource.com/)**firmasının iki ana ürününden birisidir. Diğer ürün **SonarLint’**de yine bir **code quality too**l’udur, ancak **SonarQube**’den farklı olarak yazılımın geliştirme aşamasında, yani henüz kodumuzu commit’lemeden önce bizi uyarır. Bir çok popüler **IDE **için desteği mevcuttur.** “Fix issues before they exist”** diye de bir sloganları var. Bu yazıda **SonarLint**’den bahsetmeyeceğim. Merak edenleriniz [**buradan](https://www.sonarlint.org/) **inceleyebilirler.

**SonarQube**’ü açık kaynak projeleriniz için **cloud **üzerinden ücretsiz olarak kullanabileceğiniz gibi, **private **projeler için **on-premise** kurulum yaparak da kullanabilirsiniz. **Cloud **üzerinden **private **projeleriniz için ücret ödemeniz gerekiyor.

Amacım **SonarQube **kullanımı hakkında bilgi vermek olduğu için, kurulum kısmını atlayıp, konuyu **cloud** üzerinden yani [**https://sonarcloud.io](https://sonarcloud.io) **uygulaması üzerinden** **anlatacağım. **SonarQube**’ü aşağıda listelediğim 6 ana başlıkta inceleyeceğiz;

* [**https://sonarcloud.io](https://sonarcloud.io) **uygulamasına giriş ve boş bir proje oluşturma

* Kişisel bilgisayarımıza **SonarScanner **kurulumu

* **SonarScanner **ile örnek bir .Net projesinin analiz edilmesi ve raporun sonarcloud.io ya gönderilmesi

* sonarcloud.io ya gönderilen raporun yorumlanması

* **Quality Gates**, **Quality Profiles **ve **Rules **kavramları hakkında

* **SonarQube**’ün bir CI pipeline’ına dahil edilmesi

### **sonarcloud.io SaaS Uygulamasına Giriş**

SonarQube’ü açık kaynak projeler için cloud üzerinden ücretsiz olarak kullanabiliyoruz. Giriş ekranında sizi 3 OAuth seçeneği karşılıyor. Ben github hesabımla giriş yaptım.

![**Ekran 1: OAuth login seçenekleri**](https://cdn-images-1.medium.com/max/2000/1*__hEK5CKPS0Brey_x8efVw.png)***Ekran 1: OAuth login seçenekleri***

Sağ üst köşedeki menüden “**Create new organization**” diyerek bir organizasyon oluşturarak başlıyoruz. SonarQube de her proje bir organizasyona bağlıdır. Daha sonra yine sağ üst köşeden “**Analyze new project**” diyoruz ve aşağıdaki ekranla karşılaşıyoruz.

![**Ekran 2: Mevcut organizasyona proje ekleme**](https://cdn-images-1.medium.com/max/2590/1*wImA594gk3FSCwhpmIpkYQ.png)***Ekran 2: Mevcut organizasyona proje ekleme***

Önceki adımda oluşturduğumuz organizasyonumuz seçili geliyor. Devam diyoruz ve sonraki adımda bir token oluşturuyoruz. Oluşan token değeri lazım olacak, hemen bir yere kopyalayıp devam ediyoruz ve aşağıdaki ekrandan projemizin ana programlama dilini seçiyoruz ve projemiz için bir anahtar kelime belirliyoruz.( Projede birden fazla dil olabileceğinden ötürü (c#, html, js) ana dil ifadesi kullanılmış )

![**Ekran 3: Projenin ana programlama dili seçimi**](https://cdn-images-1.medium.com/max/2000/1*pgXP6ZK0LAsPKHm5FrWfSA.png)***Ekran 3: Projenin ana programlama dili seçimi***

**Done **butonuna tıkladıktan sonra aşağıdaki ekranla karşılaşacaksınız. Bu ekranda gördüğünüz üzere Scanner’ı çalıştırmak ve projenizi analiz edebilmek için komut satırından çalıştırmanız gereken 3 adet komut yer almakta. Bu 3 komutu bir yere kopyalayıp ekranı kapatabilirsiniz.

![**Ekran 4: Proje ekleme son adım**](https://cdn-images-1.medium.com/max/2000/1*qlqtTZiekbLBfnAjk-QH_w.png)***Ekran 4: Proje ekleme son adım***

### SonarScanner Kurulumu

Yukarıdaki ekran görüntüsünde dikkat ederseniz ”**Download and unzip the Scanner for MSBuild**” ifadesi yer alıyor. Peki neden MSBuild Scanner? Dil olarak **C# **seçtiğimiz için **SonarQube **ilgili Scanner hangisiyse onu indirmemiz gerektiğini söylüyor. **Download **butonu ile yönlendirileceğiniz ekrandaki talimatları yerine getirerek MSBuild Scanner’ını bilgisayarınıza kurmuş olacaksınız. (**%PATH% güncellemesini unutmayınız**) Bu arada hangi tür Scanner kurarsanız kurun sisteminizde mutlaka **java**’nın kurulu olması gerektiğini belirteyim. **JRE **hakkındaki detayları da yine download butonuyla yönlendirileceğiniz sayfadan görebilirsiniz.

### MSBuild SonarScanner İle Projenin Kod Analizi

Önceki adımda kurduğumuz Scanner’ı kullanarak C# dili ile yazılmış olan projemiz için static kod analizini yapabiliriz artık. Bunun için projenin ana dizinindeyken sırasıyla aşağıdaki 3 komutu çalıştırmamız yeterli. Son komuttan sonra 5 nolu ekran görüntüsüyle karşılaşmanız raporun oluştuğu ve sonarcloud’a gönderildiği anlamına gelir. (**rabbitmq** sonarcloud da projemi oluştururken verdiğim key’in ismi)

![**Ekran 5: Kod analizinin başarıyla yapılması**](https://cdn-images-1.medium.com/max/2000/1*ve5b5QNLtjy2PNSz88k_Lg.png)***Ekran 5: Kod analizinin başarıyla yapılması***

    SonarScanner.MSBuild.exe begin /k:"**<proje unique key buraya>**" /d:sonar.organization="**<organizasyon_adi_buraya>**" /d:sonar.host.url="[https://sonarcloud.io](https://sonarcloud.io)" /d:sonar.login="<**olusturulan_token_adi_buraya>**"

    MsBuild.exe /t:Rebuild

    SonarScanner.MSBuild.exe end /d:sonar.login="**<olusturulan_token_degeri_buraya>**"

Bu 3 komutta koyulaştırdığım kısımlar sizin proje ve organizasyon isminize göre değişecektir. 4 nolu ekran görüntüsünde yer alan bu 3 komutu bir yere kopyaladıysanız, hızlıca komut satırından çalıştırabilirsiniz. Komut satırında “**Post-processing succeded**” ifadesini gördükten sonra artık sonarcloud’a geçiş yaparak projenizin analiz edildiğini görebilmelisiniz.

![**Ekran 6: “Passed” :)**](https://cdn-images-1.medium.com/max/2598/1*Zmu1Moj1Z3SBgJGMwO2wqg.png)***Ekran 6: “Passed” :)***

106 sayısı kod satır sayısını ifade etmekte. Hemen altında da C# ibaresi, programlama dilini işaret ediyor. Evet, çok küçük ve **XS** (X Small) kategorisinde bir proje olmasına rağmen yeşil renkli “**Passed**” ifadesini görmek güzel hissettiriyor :) [**Buradan](https://sonarcloud.io/organizations/suadev-github/projects)** inceleyebilirsiniz.

### Oluşan Raporun Yorumlanması

Raporu yorumlamadan önce hemen belirteyim, örnek proje zamanında RabbitMQ yü kurcalamak için oluşturduğum bir Asp.Net MVC [**uygulaması](https://github.com/suadev/Asp.NetMvc.RabbitMQ)**.

Proje public bir proje olduğu için sonarcloud hesabınız olmasa bile, yani sonarcloud a giriş yapmadan da oluşan son raporu [**buradan ](https://sonarcloud.io/dashboard?id=rabbitmq)**görebilirsiniz.

Linke tıkladıysanız aşağıdaki ekranla karşılaşmış olmanız gerekiyor.

![**Ekran 7: rabbitmq key’i ile oluşturduğum projemin sonarcloud dashboard’u**](https://cdn-images-1.medium.com/max/2598/1*M-oBxMGj7pjqwR9mu-AlQg.png)***Ekran 7: rabbitmq key’i ile oluşturduğum projemin sonarcloud dashboard’u***

Dikkat ederseniz rapor 5 ana başlıktan oluşuyor. Bunlar;

**Bugs**

Projenizde bu kategoriye alınan kodlarınız varsa bir şeylerin yanlış veya eksik olduğundan emin olabilirsiniz. Eğer düzeltilmezse ileride başınız ağrıyabilir, dolayısıyla bu kategoriyi önemseyin. Bizim 109 satırlık küçük uygulamamızda **Bug** olarak nitelendirilecek bir şey bulamamış sevgili SonarQube. **A **alarak geçmişiz bu dersten :) Eğer ne tür durumların bug olarak ele alındığını merak ediyorsanız [**buraya](https://sonarcloud.io/explore/projects?sort=-analysis_date)** tıklayarak public projeleri keşfedebilir, raporları dilediğinizce inceleyerek bir çok şey öğrenebilirsiniz.

**Vulnerabilities**

Güvenlik zafiyetine sebebiyet verecek olan kod parçacıkları bu kategoride raporlanmaktadır. Yani bu kategoride en az **Bug** kategorisi kadar önemli. Göz ardı etmemekte fayda var, bir örnek olması açısından rastgele **public** bir projeden aldığım aşağıda ekran görüntüsünü paylaşmak istedim.

![**Ekran 8: SonarQube’ün zafiyet olarak gördüğü alert kullanımı**](https://cdn-images-1.medium.com/max/2600/1*Y6bLUSpekApijPvwQrkURw.png)***Ekran 8: SonarQube’ün zafiyet olarak gördüğü alert kullanımı***

Burada bir js dosyası içerisinde kullanılan alert metodu zafiyet olarak görülmüş. Üç nokta (…) icon’una tıklayarak açılan aşağıdaki bilgilendirme ekranında **SonarQube’**ün gerekçesini de görebilmeniz mümkün. Özetle, debug modda alert’e tamam ama production da hassas veri gösterme riski taşıdığından burada bir zafiyet var arkadaş, diyor **SonarQube**.

![**Ekran 9: SonarQube: Production ortamında unutulan alert can yakabilir**](https://cdn-images-1.medium.com/max/2700/1*D5SWFo-Bm5qiMyJX6vR-qA.png)***Ekran 9: SonarQube: Production ortamında unutulan alert can yakabilir***

**Code Smells**

İlk 2 maddemiz kadar risk teşkil etmeyen ancak, kod okunabilirliği ve bakım maliyetleri açısından negatif etki yapabilecek kod parçacıkları bu kategoride değerlendirilmekte. Raporumuzda 3 adet **code smells **olduğu görülüyor. Hemen yanında yazan **14 min** ifadesi ise, bu 3 adet sorunun tahmini çözüm süresi :) Evet **SonarQube **sağ olsun onu da hesaplıyor ama tabi bu değerde bir hata payı olabileceğini söylememe gerek yok sanırım. Şimdi bu 3 rakamına tıklayarak aşağıdaki detay ekranına erişiyoruz.

![**Ekran 10: Code Smell liste ekranı**](https://cdn-images-1.medium.com/max/2582/1*EfjJq8poNymFwJIhAkmQFA.png)***Ekran 10: Code Smell liste ekranı***

Listede koyu renkle yazılı olan özet bilgisi aslında gerekli detayı veriyor. Örneğin ilk maddede sadece constructor içerisinden değer atanan **_hostName **adındaki **field’**ın readonly olarak işaretlenmesi gerektiğinden bahsediyor. Eğer bu maddenin üzerine tıklarsanız aşağıdaki detay ekranında ilgili kod dosyasının ilgili satırını görebiliyorsunuz.

![**Ekran 11: Code Smell detayı**](https://cdn-images-1.medium.com/max/2570/1*L8exXKELQIYt2gGEujenzQ.png)***Ekran 11: Code Smell detayı***

Burada yine **Make _hostName readonly **ifadesinin yanındaki üç nokta (…) ya tıklarsanız ekranın en altında o maddeyle alakalı bilgilendirici açıklamayı görebilirsiniz. **SonarQube **sadece açıklarınız bulmuyor aynı zamanda eğitiyor da :)** **Gördüğünüz gibi içerikte örnek kod bile var.

![**Ekran 12: İlk Code Smell için SonarQube’ün bilgilendirici içeriği**](https://cdn-images-1.medium.com/max/2298/1*6cG0gaP9H66ossozgxxfJQ.png)***Ekran 12: İlk Code Smell için SonarQube’ün bilgilendirici içeriği***

**Coverage**

Buradan yazılan testlerin projenizin ne kadarını cover ettiğiniz görebilirsiniz. Bu küçük projede test yazılmadığı için değerimiz %0 ve kırmızı renk bir uyarı manasında, yani unit test yazılmadığını ifade ediyor.

**Duplications**

Proje genelinde tekrarlı kod satırlarının toplam kod satırına oranını ifade eder. Örneğin 1000 satır kod içerisinde toplamda 100 satır eğer tekrarlı ise bu değer %10 olacaktır. Örnek projemiz az sayıda kod içerdiğinden ötürü hiç tekrarlı kod yoktur. Ben orta ve büyük ölçekli projelerde bu değerin %40- 50 'lerde olabildiğini görmüştüm ki çok ciddi bir oran. **SonarQube **detaylı olarak aşağıdaki ekran görüntüsünde gördüğünüz gibi dosya bazında tekrarlı kod oranlarını başarılı şekilde raporlayabiliyor.

![**Ekran 13: Kod dosyası bazında rapor detayları**](https://cdn-images-1.medium.com/max/2594/1*ceBBt_qOL2x_fOzmdX8zHw.png)***Ekran 13: Kod dosyası bazında rapor detayları***

### **Quality Gates, Quality Profiles ve Rules Kavramları Hakkında**

Quality Gate’ler projemizin **production **ortamı için hazır olup olmadığına karar veren kural dizileridir. Bir başka değişle, kod analiz raporunun başarılı mı (**Passed**), başarısız mı (**Failed**) olacağının belirlenmesi için kullanılırlar. 6 nolu ekran görüntüsünde yer alan yeşil renkli “**Passed**” ifadesi, kod analiz işlemi esnasında mevcut Quality Gate’lerin hepsinden geçildiği ve rapor sonucunun başarılı olduğu anlamına geliyor.

SonarQube desteklediği her programlama dili için **default (ön tanımlı) **olarak, **Quality Profile**, **Rules **ve **Quality Gate** tanımlamalarına sahip. Örneğin C# dili için [**buraya ](https://sonarcloud.io/organizations/suadev-github/quality_gates/show/9)**tıklayarak ulaşabileceğiniz, benim oluşturduğum public organizasyonun aşağıdaki ekranında ön tanımlı olarak gelen **Quality Gate**’leri görebilirsiniz.

Örneğin; en altta belirtilen kuraldan, Security notunun **A**’dan daha kötü olması raporun başarısız sonuçlanacağını anlamalıyız. Ve en önemlisi, sol üst köşedeki Create butonuyla, kendi **custom Quality Gate**’inizi oluşturabilirsiniz. Örneğin siz projenizde Security notunun **B** den daha kötü olması durumunda raporun başarısız olmasını sağlayabilirsiniz.

![**Ekran 14: Ön tanımlı Quality Gate’ler**](https://cdn-images-1.medium.com/max/2610/1*f5Z7-FilbQHxPbjHxFognA.png)***Ekran 14: Ön tanımlı Quality Gate’ler***

Peki Security notu nasıl belirleniyor? Burada **Rules** kavramına değinmemiz gerekiyor. Yukarıdaki ekran görüntüsünde **Rules **sekmesine tıkladığınızda aşağıdaki ekrana erişeceksiniz. Sol taraftaki filtreleme özelliğini kullanarak yalnızca **C#** için tanımlı olan kuralları listeledim. Kurallara tıklayarak örnek kod parçacıklarının da yer verildiği bilgilendirici detay sayfalarına göz atabilirsiniz. Dikkat ederseniz C# dili için kural havuzunda toplam 358 adet (**21.08.2018 tarihli kural sayısı** ) kural tanımlı. Bu kurallar C# ve diğer diller için sürekli olarak artmakta, eklenen yeni kurallarla **SonarQube** geliştirilmeye devam etmektedir.

![**Ekran 15: C# dili için tanılı kurallar listesi**](https://cdn-images-1.medium.com/max/2634/1*TRjQmHbo7EP689BPImTNQg.png)***Ekran 15: C# dili için tanılı kurallar listesi***

**Quality Profile **içinse en basit haliyle, çeşitli kuralları içeren gruplardır diyebiliriz. Yani C# dili için kendinize özel oluşturacağınız bir Quality Profile, en az 1 en fazla 358 adet kuraldan oluşabilir.

Özetle SonarQube’ün ön tanımlı **Quality Gate** ve **Quality Profile** ını kullanmayıp, kendi tanımlamalarınızı yapabilirsiniz. Üstelik bunları proje bazlı da yapabilirsiniz. Aşağıdaki ekrandan rabbitmq projemiz için Administrator sekmesinde tıkladığımızda açılan menüden bu değişikliği yapabiliriz.

![**Ekran 16: Proje bazlı admin ekranları için menü**](https://cdn-images-1.medium.com/max/2000/1*D3lxb3Jd6UU91_loBOQSvg.png)***Ekran 16: Proje bazlı admin ekranları için menü***

Bu menüden **Quality Gate** sekmesinde tıkladığımızda gelen ekrandan, projemiz için istediğimiz Quality Gate’i seçebiliriz. Buradaki **sample gate **isimli **Quality Gate** benim örnek olması için yaptığım tanımlama. Dikkat ederseniz **Sonar way** için, **default** ibaresi var yani tüm projeler ilk oluşturulduklarında ön tanımlı olarak **Sonar way Quality Gate**’ine tabi tutulmaktalar.

![**Ekran 17: Projenin Quality Gate’inin değiştirilmesi**](https://cdn-images-1.medium.com/max/2000/1*s65lqh1pplrxaHl18eQLLQ.png)***Ekran 17: Projenin Quality Gate’inin değiştirilmesi***

Aynı şekilde **Quality Profiles** linkine tıklayarak SonarQube’ün projenize atadığı **ön tanımlı **profili kullanmayıp kendi tanımladığınız profilin kullanılmasını sağlayabilirsiniz. Ön tanımlı gelen **Sonar way** isimli **Quality Profile** tüm kuralları içerirken siz kendi profiliniz için kapsamı daraltmak isteyebilirsiniz mesela.

Örnek vermek gerekirse; kendi özel tanımlayacağınız Quality Profile’ınızdan projemizde 2 adet Code Smell olarak değerlendirilen, **readonly field **kuralını çıkarırsanız, Code Smell sayımız bire inecektir. Bu yolla siz, “**readonly field kuralı benim için code smell değildir”**, demiş olursunuz aslında.

### SonarQube’ü CI Pipeline’ınıza Dahil Edin

Eğer hali hazırda bir CI yazılımı kullanmaktaysanız, **SonarQube **veya benzeri bir statik kod analiz aracını CI pipeline’ınıza dahil edebilirsiniz. Bu noktada kullandığınız CI yazılımına göre eforunuz değişecektir. Jenkins, TeamCtiy, Travis CI vs. gibi bazı CI araçları SonarQube için sahip oldukları plugin’ler sayesinde bu eforu minimuma indirmekteler. Diğer bir ifadeyle **SonarQube **için doğal desteğe sahipler.

Doğal entegrasyona sahip CI araçlarında SonarQube rapor sonucuna göre pipeline’ınızı kırıp, ilgili yerlere bildirim yapabilirsiniz. Örneğin build adımından sonra SonarQube’ü tetikler, raporu sonucunu bekletir ve rapor başarısız olursa bir sonraki adıma geçmeyebilirsiniz.

Plugin ile doğal entegrasyon imkanı olmayan CI araçlarında SonarQube’ü tetiklemek için ise, yml scriptinize **MSBuild SonarScanner İle Projenin Kod Analizi **bölümünde belirttiğimiz komutları ekleyerek ilerleyebilirsiniz. Bu noktada rapor sonucunu bekleyerek, gelen sonucuna göre bir aksiyon almak mümkün müdür açıkçası bilmiyorum. Yorumlarınızı alabilirim.

![**Ekran 18: SonarQube ve CI yazılımları**](https://cdn-images-1.medium.com/max/2000/1*CV1-CyBj_0mWsb_FIV1kJA.png)***Ekran 18: SonarQube ve CI yazılımları***

Bu yazımda statik kod analiz yazılımları arasında oldukça popüler olan **SonarQube**’ü tanıtmak istedim. Eksik veya hatalı kısımlar ve** SonarQube**’ e alternatif farklı özelliklerde araçlar için yorumlarınızı beklerim.
