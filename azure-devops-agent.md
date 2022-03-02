Agent'ları oluştururken baz aldığımız doküman:\
[Self-hosted Linux agents](https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops)

Dokümanda yer alan "Linux" alt başlığının altındaki adımları takip edip gerekli düzenlemeleri yapıp agent'ımızı oluşturuyoruz.\
"Dockerfile" ve "start.sh" bizim için önemli olan ve gereken değişiklikleri üzerlerinde yapacağımız dosyalardır.

Önemli not #1:\
Agent'lar varsayılan olarak SSL bağlantı ile çalışmak üzere tasarlanmış. HTTP olan servislere bağlanabilmesi için "start.sh" dosyasındaki "config.sh" dosyasını çalıştırdığımız satıra `--sslskipcertvalidation` parametresini veriyoruz. Bu etkinleştirilmezse ve Azure DevOps'un servis yapıldığı adres HTTPS değilse bağlantı kurulamıyor.

AYNET -> TLS\
Arge2004 -> TLS değil

"config.sh" için çeşitli authentication yöntemleri var, bunlardan ikisi username ve password ile pat. Username ve password ile bağlanma yöntemi daha kolay. parametre olarak `--username "$AZP_USERNAME"` `--password "$AZP_PASSWORD"` verip bilgileri de env. variable olarak container içine yazdığımızda authentication yapılmış oluyor fakat bu yöntem kolay olsa da ileride kullanıcının şifresinin değişmesi durumunda agent sürekli bağlanmayı deneyip kullanıcıları kilitlemekte.

"pat" ile bağlanabilmek için ise "config.sh" komutuna `--auth PAT` `--token $(cat "AZP_TOKEN_FILE")` parametreleri verilerek güvenli bağlantı yine sağlanmakta.

".testagent." olarak tanımlanan agent'lar aslında agent'ın pool'a bağlanıp bağlanamadığını kontrol etmemize, troubleshoot etmemize yardımcı olmak amacıyla oluşturuluyorlar. Eğer bu agent bağlanabiliyorsa bizim asıl agent'a yüklediğimiz custom uygulamalarda, kütüphanelerde problem var anlamı oluşmuş oluyor.

Superagent kullanma nedenimiz: şirkette projelerin kendi agent'larını oluşturup o agent ile o projenin derleme vb. işlerini ilerletiyoruz. Endüstri'deki asıl kullanım ise şirket genelinde ortak pool oluşturup ayrı uygulamalar için ayrı agent'lar tanımlamak. Python için ayrı agent, Java için ayrı, NodeJS için ayrı. Bizde bu yaklaşım olmadığı için superagent'lar ile işleri sürdürmekteyiz.

Bir agent'a farklı versiyonlarda uygulamalar kurmak mümkün. Örneğin Java 1.8, 11, 17 beraber kurulup "JAVA_HOME" variable'ını değiştirerek istediğimiz Java versiyonuna erişiebiliyoruz. Dockerfile'da bunu şu kod satırı ile yapabiliyoruz:\
`ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64`
Container'ı çalıştırırken veya container içinde bir pipeline çalıştırırken bu komut satırını ezip versiyon değişikliği yapabiliyoruz.
Dikkat: Birden fazla JDK kurulumu yapıldıysa PATH'i eklemiyoruz. Eklersek bu sefer Path'i de değiştirmemiz gerekecek. 

Docker in docker kısmı:

Yağtığımız şey host makinedeki docker soketini container içindeki docker socket'ine volume olarak bağlıyoruz:
`"/var/run/docker.sock:/var/run/docker.sock"`\
Bunu container içerisinde kullanabilmek için docker engine'in kurulu olmasına gerek yok, docker cli yeterli. İki soketin haberleşmesi için gereken araç bu.

Buradaki sorunlar ise: Eğer registry SSL'li değilse daemon.json'a eklenmesi gerekiyor, herkes aynı image'ları kullanıyor, bir container'ın docker komutu diğerlerini de etkileyecek durumda olmuş oluyor. 

Buna çözüm oalrak buildah kullanılabilir. Buildah'ın amacı docker kullanmadan image oluşturmak ve bunları Nexus'a push'lamak. Docker'daki komutlar buildah versiyonları ile birebir değiştirilebilmekte (podman'de de geçerli ve buildah da podman'in bir alt aracı aslında). 

CLI'ın neden olabileceği problemlerden başkaları ise kişi tüm docker image'larını silebilir veya cluster seviyesinde crush'lara neden olabilir (privilege'lar host seviyesinde içeri taşınıyor). Ekiplerin buildah'a yönlendirilmeleri bu muhtemel sorunları önlememizi sağlayacaktır.

Buildah'a geçiş için versiyon 2 veya 1.9 gerekmekte. 

Not #2: Buildah'da local'deki image'ların başında "localhost" şeklinde bir tag var. Normalde docker'da olmayan bu tag'leme ile localhost'daki image'ları daha kolay görmüş oluyoruz.

Azure DevOps kendi sürümüne göre agent versiyonu önermekte. "Collection Settings" -> "Agent pools" -> "Pool-ismi" -> "New agent" dediğimiz zaman bize agent versiyonu önermekte.

