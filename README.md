# honeyTEA Projesi
Cloudflare, Centos ve CMS birleşimi ile oluşturduğum bir HoneyPot sistemidir. 

## Sistem Gereksinimleri

### Cloudflare 
Sunucunuz Cloudflare arkasındaysa blokeyi Cloudflare'de yapmanız gerekiyor. Aksi takdirde size gelen ip adresi CloudFlare ip adresi olduğu için, bloke işe yaramayacaktır. Diğer bir yandan Mod_Cloudflare Apache modülünün sunucunuzda kurulu olması gerekiyor. Bu sayede XForward ile isteklerin header bilgisi üzerinden saldırı yapan kişinin gerçek ip adresini alabiliyoruz. 

### CENTOS
Centos işletim sistemine göre hazırlanmıştır. Farklı OS'lerde kodları modifiye etmek gerekebilir.

### CMS
Config.txt içerisindeki Honeypotlar Wordpress için eklenmiştir. Bu dosyaları Joomla veya Drupal'a göre de düzenleyebilirsiniz.

