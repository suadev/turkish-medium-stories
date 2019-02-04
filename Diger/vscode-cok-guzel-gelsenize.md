
# VS Code Çok Güzel Gelsenize

Bu yazıyı;

— **VS Code (** Visual Studio Code **) **mu o da ne?

— Mis gibi **Visual Studio** var neyine yetmiyor yaa

— Ben .Net yazmıyorum abi ne** VS Code**’u? ( Bana bu yazıyı yazdıran cümle )

— Ben kullanıyorum ama alt yapısı, avantajları vs. hakkında bilgim yok.

diyen yazılım emekçisi kardeşlerim için yazıyorum, toplanın :)

**.Net’ci tayfa için küçük bir not:** Visual Studio ile **VS Code**’u kıyaslamanın doğru olmayacağını düşünüyorum. Zira Visual Studio** full-featured** (tam özellikli ) bir **IDE** iken, **VS Code** eklenti kurulmamış yalın haliyle bir metin editörüdür. Ancak **IDE-type **güçlü eklentilerle editör olmanın ötesine geçebiliyor. Visual studio’nun built-in barındırdığı bir çok özellik ( Debugger, Database Integration vb.. ) VS Code’da eklentiler sayesinde kullanılabiliyor.

Üniversite yıllarımı da hesaba katarsak 10 yılı aşkın süredir , **Visual Studio**, **Sharp Develop**, **Netbeans**, **Borland C++ Builder** gibi IDE’ lerle çok çeşitli uygulamalar geliştirdim. Tabi .Net üzerine yoğunlaştığım için en çok **Visual Studio** ile haşir neşir oldum diyebilirim. Son 1 yılı aktif olmak üzere 2 yıldır da **VS Code** kullanarak, .Net Core Web Api, Type Script, Angular ve React uygulamaları geliştiriyorum. Şuan **.Net Core** ile **yazılmamış **bir kaç .Net projesi dışında** Visual Studio**’ yu hiç açmıyorum diyebilirim.

Şuraya bir tutam nostalji serpiştirip devam edelim;

![](https://cdn-images-1.medium.com/max/2000/1*kO0YADdFkqsnlgnOVf3Rew.jpeg)

**VS Code’u kullanmaya başladığımdan beri Visual Studio’ yu hiç özlemedim diyebilirim. **Nedenlerini iki başlıkta anlatmaya çalışayım;

* **VS Code**’un teknik alt yapısı ve performansı

* **VS Code**’un artıları

### **VS Code Teknik Altyapı**

**VS Code** Microsoft tarafından **Electron** framework’ü üzerinde** typescript **ile geliştirilmeye devam ediyor. Microsoft bildiğiniz üzere son yıllarda açık kaynağa, ve platform bağımsız ürün geliştirmeye hiç olmadığı kadar önem vermekte. Bu arada Microsoft dediysek tabi proje** **tamamen açık kaynak olduğu için aslında** **yüzlerce **contributor **tarafından geliştirilmekte. Öyle ki an itibariyle 40.000 civarı commit sayısı ile Github’ın en çok star alan projeleri arasında yer alıyor.

**Electron’a **bir parantez açmazsak olmaz**. Electron**’un çıkış noktası **Atom **metin editörüdür. Github **Atom **metin** **editörünü **Electron **ile geliştirdi. **Electron**, önceleri **Atom Shell** olarak bilinirken 2015 yılında **Electron **olarak yeniden isimlendirilen ve** Github **tarafından geliştirilen açık kaynak bir uygulama geliştirme framework’ üdür. Peki bu **Electron**’un olayı nedir?

Adamlar diyor ki;
> # “Build cross platform desktop apps with web technologies.”

Yani hem masaüstü uygulaması olacak, hem de web teknolojilerini (js, html, css vb..) kullanacağız. Buna ilaveten uygulamamız platform bağımsız olacak. Daha ne olsun öyle değil mi :)

Microsof’un **VS Code **için** Electron**’u tercih etmesinin en önemli sebebinin bu platform bağımsızlığı olduğunu söylemek yanlış olmaz sanırım. Tabi en az bunun kadar önemli olan diğer bir kriter ise performans.

**Electron**, 2012 yılında bir **Intel **stajyerinin, **Elecktron**’a** **benzer yapıda olan **node-webkit**’ te karşılaştığı bir sorunu çözmeye çalışırken ortaya çıktı. Bu ortaya çıkış hikayesini merak ediyorsanız [**buradan](http://cheng.guru/blog/2016/05/13/from-node-webkit-to-electron-1-0.html) **okuyabilirsiniz.

**Electron**, back-end de nodejs, fron-end de ise Google’ın açık kaynak web browser projesi olan **Chromium’u **kullanmaktadır diyerek, **Electron **bahsini sonlandırıp **VS Code**’un performansı ve sistem kaynaklarını nasıl kullandığına bakalım.

**VS Code, **okuduğum kaynaklara göre hız noktasında rakiplerinden bir adım önde. Ben **Sublime **veya **Atom**’u çok az kullandığım için kıyaslama yapacak konumda değlim. Ancak örneğin, VS Code’un Atom’dan daha hızlı olduğu söyleniyor. Bunun böyle olmasının en önemli sebeplerinden birisi Atom’dan farklı bir text editör kullanmasıymış. Bu editör Microsoft tarafından geliştirilen **browser **tabanlı [**monaco ](https://github.com/Microsoft/monaco-editor)**text editörüdür. (Editör’ün neden **browser **tabanlı olması gerektiğini **Chromium**’dan hatırlamalısınız. )

**VS Code** ‘dan önce **Sublime **veya **Atom **kullanan developer’ lar dan **VS Code**’a geçenlerin sayısı gün geçtikçe artmakta. Bunu destekleyen en önemli anketlerden birisi bana göre stackoverflow’un yıllık düzenli olarak yaptığı developer anketi. 2018 anketinin tamamına [**buradan ](https://insights.stackoverflow.com/survey/2018/)**erişebilirsiniz.

![VS Code Zirvede — 2018](https://cdn-images-1.medium.com/max/2000/1*sH6LJw0LETK2_oJLJDD30g.jpeg)*VS Code Zirvede — 2018*

Peki VS Code’un RAM ve CPU kullanımı nasıl? Bu konuda forumlara ve github da açılan issue’lara bakarsanız eğer ilk versiyondan bugüne dek bu durumdan şikayet edenlerin sayısında bir azalma olduğunu görebilirsiniz. Peki şu an güncel versiyonda aşırı RAM ve CPU kullanımından şikayet eden yok mu? Eminim ki vardır. Ancak RAM ve CPU kullanımını arttıran bazı dış faktörler genelde göz ardı edilebiliyor.

Öncelikle üzerinde çalışılan projenin büyüklüğü, ve aktif olan kod dosya sayısı gibi faktörler gözle görülür bir RAM ve CPU artışına sebep **olmamakta**. Çok fazla sayıda eklenti yükleme bir sebep olabileceği gibi, tek bir eklentide mevcut olan bir **bug**, veya **memory leak**’e sebep olacak bir kod parçacığı bu duruma sebep olabilmekte. Aşırı RAM ve CPU kullanımı sorununda en sık karşılaşılan durum **VS Code**’da yüklü olan eklentilerden bir veya bir kaçından kaynaklı oluyor.

Aşırı RAM ve CPU kullanımı durumunda, özellikle çok fazla indirilme sayısına ulaşmamış, community içerisinde henüz pek kabul görüp yaygınlaşmamış olan eklentilerden şüphelenebilirsiniz. Bu eklentileri pasife çekip veya tamamen kaldırıp kaynak kullanımını gözlemleyebilirsiniz.

Bu arada bu fakirin de hobby amaçlı geliştirip [**burada ](https://marketplace.visualstudio.com/items?itemName=suadev.csharp-region-manager)**yayınladığı gereksiz bir eklentisi var (: Bu basit eklentinin kaynak kodlarına [**buradan](https://github.com/suadev/vscode-region-manager) **ulaşabilirsiniz.

VS Code açık durumdayken komut satırından aşağıdaki komutu çalıştırarak cpu ve ram kullanımı hakkında fikir edinebilirsiniz. Dikkat ederseniz en fazla cpu kullanımı **renderer/window** prosesine ait.

    **code --status**

![](https://cdn-images-1.medium.com/max/2000/1*W6GMTH8OP4HkwoHFPVlhYg.png)

### VS Code’un Artıları

Tamamen açık kaynak ve ücretsiz olması, platform bağımsız (Windows, Linux, Mac) olması ilk akla gelen ve en bilindik artıları.

Bunlara ilave edilebilecek en az bunlar kadar öne çıkan diğer artıları ise hızı ve kolay kullanımı. Evet **VS Code** gerçekten çok hızlı ve stabil çalışıyor. Ben 2 yıllık deneyimimde henüz bir kere bile **crash **olduğunu hatırlamıyorum. Buna ilaveten, büyük boyutlu projelerde dahi çok hızlı bir şekilde açılıp kapanıyor. Yıllarca Visual Studio ile çalıştıysanız, eğer bu açılma kapanma hızının neden önemli olduğunu bilirsiniz :) Bu arada açılma hızı derken tabi, boş bir **instance **açmaktan ziyade**, **bir projeyi** VS Code **ile açmaktan bahsediyorum. Söz buraya gelmişken, komut satırında projenin ana dizinindeyken aşağıdaki basit komut ile projenizi hızlı bir şekilde **VS Code** ile açabilirsiniz. Bunun çok yaygın olarak kullanılmadığını gördüğüm için paylaşmak istedim. Ben şahsem çok sık kullanıyorum.

    code .

**> Güçlü Eklentiler**

**VS Code’u **ilk kurduğunuz haliyle, yani henüz herhangi bir eklenti yüklü değilken, gayet stabil, performanslı ve kullanıcı dostu bir text editör olarak kullanabilirsiniz.** VS Code’**u kurduktan sonra kullandığınız dil ve framework’ler için geliştirilmiş olan eklentileri de kurmalısınız. Hani yazının başında bahsettiğim “**Ben .Net yazmıyorum ki abi ne VS Code’u?**” diyen arkadaş var ya, o arkadaş şu aralar **VS Code** üzerinde cayır cayır python yazıyor :)

Yani burada akıllıca ve modüler bir tasarım söz konusu. Yazdığınız dil veya dillere göre parçaları takıp çıkarıyorsunuz. Bu da **VS Code’**u oldukça **lightweight **bir geliştirme ortamına dönüştürüyor. Örneğin **Python **yazılımcıları** [buradaki](https://marketplace.visualstudio.com/items?itemName=ms-python.python)** eklentiyi kurarken, **C#** dilinde geliştirme yapmak için OmniSharp tarafından geliştirilen [**buradaki ](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp)**eklenti kuruluyor. Burada C# eklentisine bir parantez açayım; Arada bir **eklentinin** **caching tasarımından kaynaklandığını** düşündüğüm can sıkıcı durumlar yaşanabiliyor. ( Örneğin; bir class’ın bir yerden farklı bir yere taşınmasından sonra oluşan, class’ın kullanıldığı yerlerde tanımlanamaması veya intellisense’in çalışmaması durumu) Şimdilik bu gibi durumlarda aç/kapa yapıyoruz. Neyseki **VS Code **çok hızlı açılıp kapandığı için durumu kurtarıyor :)

Bu arada yazıyı yazarken farkettim, Python eklentisi C# eklentisinin 2 katı kadar indirilmiş. Hatta şuan marketin en çok indirilen eklentisi. [**Buradan](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Downloads)** market’in en popüler eklentilerini görebilirsiniz. Anlaşılan python’cılar **VS Code**’u çok sevmiş :)

**> Git Entegrasyonu**

**VS Code**’ un bir diğer çok önemli artısı ve güçlü yönü ise **git **entegrasyonudur. (Built-in olarak TFS’ i desteklemiyor, extension kurmak gerekiyor.)

Git işlemlerini **VS Code**’un dahili terminalinden yapabilceğiniz gibi arayüzden de yapabilmeniz mümkün. Arayüzden, saniyeler içinde yeni bir **branch **oluşturabilir, branchler arası geçiş yapabilir,** unstaged **olarak bekleyen dosyaları görebilir, bu dosyaları tek tıkla **staged/unstaged **yapabilir, yine çok hızlı bir şekilde **commit**’ leyip sunucuya **push **edebilirsiniz. Eğer **merge **esnasında bir **conflict **(çakışma) yaşadıysanız ve **git auto-merge** başarısız olmuşsa, **VS Code** çok başarılı bir arayüz üzerinden size conflict’lerinizi manuel olarak **merge **etme fırsatı sunuyor. Git entegrasyonu hakkında çok daha fazlası yazılabilir ancak kullanıp deneyimlemenizi öneririm.

**> Dahili Terminal**

VS Code dahili terminale sahip olan ilk ve tek editör değil elbette. Ancak en kullanışlısına sahip :) Benim en beğendiğim özelliklerinden birisi birden fazla terminal açabilmek ve bu terminaller arasında kolayca geçiş yapabilmek. Örneğin bir angular projesi geliştiriyorsanız, bir terminal’i webpack, npm için bir diğerini ise git işlemleri için kullanabilirsiniz. Ek olarak bu dahili terminal üzerinde dos komutlarını da çalıştırabilirsiniz. Bir diğer güzel özelliği ise, bu terminali ön tanımlı (**default**) olarak powershell veya bash olarak açabilmeniz.

Şöyle ki; **Windows için** terminal ön tanımlı olarak cmd.exe yi kullanmakta. Siz bu tanımı değiştirebilir, açılan her terminal’in powershell olarak açılmasını sağlayabilirsiniz. Bunun için tek yapmanız gereken **Ctrl+, **(kontrol + virgül)** **ile açtığınız kulanıcı ayarlarının tutulduğu settings.json’a aşağıdaki tanımlamayı yapmak.

    "terminal.integrated.shell.windows": "C:\\WINDOWS\\sysnative\\WindowsPowerShell\\v1.0\\powershell.exe"

Dahili terminal hakkında daha detaylı bilgi için [**buraya](https://code.visualstudio.com/docs/editor/integrated-terminal) **göz atabilirsiniz.

**>** **Kullanıcı Ayarları**

Kullanıcı ayarları konusuna dahili terminalden bahsederken giriş yapmış olduk aslında. **Ctrl+,** kısa yoluyla açılan json formatındaki konfigürasyon dosyamız, bizim **VS Code**’u ve kurmuş olduğumuz eklentileri nasıl kullanmak istediğimize karar verdiğimiz yerdir. VS Code kısmı tamam, peki eklentileri nasıl konfigüre ediyoruz? Bu sorunun cevabı yine VS Code un beğendiğim özelliklerinden birisine yöneltiyor bizi. Bir eklenti geliştirirken, bazı özellikleri opsiyonel yapabilirsiniz. Örneğin C# eklentisinde projenin yüklenme timeout süresi parametrik olarak tasarlanmış. Yani siz bu süreyi settings.json’a ekleyeceğiniz aşağıdaki tanımlama ile kendiniz için özelleştirebilirsiniz.

    "omnisharp.projectLoadTimeout": 120

VS Code’ un kendi özelleştirilebilirliği için de bir örnek vermek gerekirse, üzerinde çalıştığınız kod dosyasının her kayıt işleminden sonra otomatik olarak formatlanmasını (bir nevi **beautify**) isterseniz yine settings.json’a aşağıdaki tanımlamayı yapmanız lazım.

    "editor.formatOnSave": true,

VS Code üzerine daha yazılacak çok şey var. Bu yazıyı, yazımın girişinde 4 madde ile gruplandırdığım meslektaşlarımda VS Code hakkında bir farkındalık oluşturabilmek adına yazmak istedim. Yazı veya VS Code özelinde farklı görüş, eleştiri veya düzeltmeler için şimdiden teşekkür ederim.
