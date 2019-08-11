# honeyTEA Projesi V2 Yayında!

Cloudflare, Centos ve CMS birleşimi ile oluşturduğum bir HoneyPot sistemidir. 

## Sistem Gereksinimleri

### Cloudflare 
Sunucunuz Cloudflare arkasındaysa blokeyi Cloudflare'de yapmanız gerekiyor. Aksi takdirde size gelen ip adresi CloudFlare ip adresi olduğu için, bloke işe yaramayacaktır. Diğer bir yandan Mod_Cloudflare Apache modülünün sunucunuzda kurulu olması gerekiyor. Bu sayede XForward ile isteklerin header bilgisi üzerinden saldırı yapan kişinin gerçek ip adresini alabiliyoruz. 

### CENTOS
Centos işletim sistemine göre hazırlanmıştır. Farklı OS'lerde kodları modifiye etmek gerekebilir.

### CMS
Config.txt içerisindeki Honeypotlar Wordpress için eklenmiştir. Bu dosyaları Joomla veya Drupal'a göre de düzenleyebilirsiniz.

### Geniş anlatım için...

Geniş Anlatım: http://www.teakolik.com/honeytea-ile-sazan-avlayalim-mi/

Twitter: https://twitter.com/TEAkolik

# HoneyTEA Nasıl Çalışıyor?

Sunucu loglarınızı okuyarak web sayfamıza gelen istekleri inceliyor. Sitenize kötü niyetli bir istek gelirse (teakolik.com/wp-config.php.bak gibi) bu isteği tespit ediyor.

honeyTEA log kayıtlarını okurken öncelikle bu istekler arasından kötü niyetli olanları tespit etmesi lazım. Bunun için config.txt dosyasını kullandım. Script çalışırken

**config.txt**

dosyasının içeriğini okuyor. Config.txt içerisine daha önce…

`/wp-config.php`
`/download.php`

…şeklinde tanımlamalar yaptığım için, log kayıtlarında bu dosyalara erişim sağlamak isteyen kişiyi kötü niyetli olarak tespit edebiliyor. Bu isteği gönderen kişinin ip adresini alıp…

**blacklist.dat**

…dosyası içerisine yazıyorum. Ip adresini aldığı gibi o isteğin geldiği saat ve tarihi de kayıt ediyorum. Sonrasında honeyTEA, blacklist.dat içerisindeki ip adreslerini inceliyor. Bu ip adreslerini Cloudflare’ye gidip ne zaman bloke ettiyse, (zamanı bu sebeple alıyorum) kontrol edip, üzerinden 30 dakika geçmişse blokesini kaldırıyor. Yeni gelen kötü istekleri ise yine saat ve tarihinden bakarak, Cloudflare Apisi üzerinden bloke ediyor.

### Soru: Neden Cloudflare üzerinden bloke ediyoruz?

Cevap: Sunucunuz üzerinde iptables veya firewalld kullanarak da bloke edebilirsiniz. Ancak Cloudflare’nin çalışma mantığını unutmamak gerekiyor. Eğer iptables veya firewalld üzerinden bloke edersek hiçbir işe yaramayacaktır. Çünkü Cloudflare hiçbir zaman ziyaretçiyi sitenize göndermez. Sunucunuza gelen Cloudflare’dir. Ziyaretçi Cloudflare’den alır isteği, Cloudflare’ye sorar, aracılığı yapan Cloudflare olduğu için kötü niyetli kişinin ipsini bloke etmeniz bir işe yaramayacaktır.

### Soru: Ziyaretçinin gerçek ip adresini o zaman nasıl alıyoruz?

Cevap: Cloudflare’den ziyaretçinin gerçek ip adresini Apache’nin mod_cloudflare modülünü kullanarak header bilgilerinden alıyoruz. Kısacası Cloudflare sizin sunucunuza geliyor ve bu modülü kurarsanız, gelirken isteğin header bilgisinde ziyaretçi ip adresini de size verebiliyor.

Sonrasında honeyTEA işlerini bitiriyor. Tabi bu sırada…

**Line.dat**

..dosyası içerisine bir satır numarası ekliyor. Bir sonraki çalışmasında okuduğu access.log içerisinde kaçıncı satırada kaldığını bu dosya içine yazdırıyorum. Mesela önceki çalıştığında log dosyamız 3100 satırdı. Sonra yeni istekler geldi. Log dosyası 3.500 satır oldu. honeyTEA Line.dat doyasına bakıyor ve 3.100’üncü satırdan itibaren log dosyasını okuyor. Bu sayede her seferinde log dosyasını baştan aşağıya okumasına gerek kalmıyor ve işler yolunda gidiyor.

Aksi takdirde her çalıştığı zaman baştan aşağı okuyacak ve her seferinde daha büyük bir yükle çalışacak. Doğal olarak da işler sarpa sarabilir. Sunucu da log dosyalarını belirli zaman aralıklarında yeniler. Mesela 100kb olduğu zaman access.log.tar yapar. Sıfırdan yeni bir access.log dosyası açar. Böyle bir durumda line.dat dosyasında 3.100 yazarken, access.log dosyamızda bir satır log kaydı olur ve patlardı… Bunun için de birkaç kod yazıp, Line.dat dosyasındaki satır sayısı ile access.log içerisindeki satır sayısını karşılaştırıyorum. Line.dat daha büyükse yani 3.100’de kalmışken, access.log üzerinde 100 satır log varsa en başa dönüyor ve birinci satırdan itibaren okumaya başlıyor.

Kısacası aklıma gelen her türlü senaryoyu düşünerek bu projeyi hazırladım.

## Kurulum!

Bu dosyaları sunucunuzda bir klasöre kopyalayın. Ben /etc/local/TEAkolik/ klasörü içerisinde kullanıyorum.
honeyTEA.sh dosyasına Chmod +x komutu ile çalıştırma izni veriyoruz.
nano veya benzeri bir editörle honeyTEA.sh dosyamızı açıp, ayarlar bölümüne gerekli ayarları yazıyoruz.
Crontab üzerinde bu scripti 5 dakika, 10 dakika gibi bir aralıkla çalıştırıyoruz.

## Ayarlar!

`[root@TEAkolik] nano honeyTEA.sh` komutu ile scripti düzenlediğiniz zaman aşağıdaki gibi ayarlar bölümünü göreceksiniz.

BANNED_TIME değeri dakika cinsinden bloke süresini belirtiyor. Ben 30 dakika yaptım ve yeterli olacağını düşünüyorum. Bu süreden sonra script otomatik olarak bloke ettiği ip adresinin blokesini Cloudflare’ye gidip açıyor.

USER değerine Cloudflare üyelik mail adresini yazmanız gerekiyor. Cloudflare Apisi bu şekilde çalışıyor. Üyelik mailinizi yazıyorsunuz.

TOKEN değerinde ise Cloudflare Apisinin size özel verdiği Keyi yazıyoruz. Cloudflare hesap ayarlarına (Cloudflare.com/a/account/my-account) girdiğiniz zaman hemen aşağıda GLOBAL API KEY satırının karşısında VIEW API KEY butonuna tıkladığınız zaman görebilirsiniz.

LINE_FILE değeri satır numaralarını tuttuğumuz dosyadır ve hangi klasöre kopyaladıysanız dosyaları bu klasörün yolunu yazıyorsunuz.

BLACKLIST_FILE değerinde ise bloke ettiğimiz ip adreslerini tuttuğumuz blacklist.dat dosyasının yolunu yazıyoruz. Aynı şekilde nereye kopyaladıysanız dosya yolunu yazıyoruz.

LOG_FILE değerinde ise sunucunuzun access.log dosyasını belirtiyorsunuz. Centos için /var/log/httpd/access.log şeklindedir. Ancak Nginx kullanıyorsanız bu dosya yolu farklı olacaktır. Ya da Centos yerine Ubuntu kullanıyorsanız yine bu yol farklı olacaktır. Ne kullanıyorsanız ona göre doğru klasörü göstererek yazın.

Önceki yazımda bahsettiğim gibi hırsıza yaptığımız muamelenin aynısını lamerlere, hackerlara, spammerlara ve benzeri bilimum mahlukata bu proje ile yapıyoruz… Hırsıza karşı en büyük silahınız caydırıcılıktır. Kötü niyetli saldırganlara karşı da bu silahı kullanabilirsiniz. Etkilidir ve güvende hissetmenizi sağlar. Ancak unutmayalım, %100 güvenlik diye bir şey yoktur!

## Config.txt Dosyasını Unutmuyoruz!

Config.txt dosyasını açarsanız WordPress için bazı kötü niyetli istekleri yazdığımı göreceksiniz. Bu dosya içeriğini kullandığınız CMS sistemine göre değiştirebilirsiniz. Joomla, Drupal gibi sistemler için de rahatlıkla çalışabilecek bir projedir.

WordPress için eklediğim satırları silip, Joomla veya Drupal ya da özel bir Web sitesi için kendinize göre bu istekleri yazabilirsiniz. En kötü sunucunuzun Access.Log dosyasını açıp, gelen istekleri bir süre izleyerek, kendinize göre geliştirmenizi sağlamanız mümkün! Kısacası WordPress dışındaki ve Centos dışındaki sistemleri de unutmadım…

## Nasıl çalışıyor?

Çok basit bir şekilde çalışıyor. Çalıştırırken Config.txt dosyasını unutmayın. Aksi takdirde hata verecektir. Ayarlarınızı yaptıktan sonra Crontab’a 5 dakikalık aralarla tekrar çalışmasını sağlamayı da unutmayın. Çalıştırmak için aşağıdaki komutu kullanabilirsiniz.

`[root@TEAkolik] ./honeyTEA.sh config.txt `

Crontab üzerine aşağıdaki değeri girerek 5 dakikada bir otomatik olarak çalışmasını sağlayabilirsiniz.

`5 * * * * /etc/local/teakolik/honeyTEA.sh  /etc/local/teakolik/config.txt`
