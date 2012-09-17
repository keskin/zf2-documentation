.. EN-Revision: none
.. _user-guide.overview:
##########################
Zend Framework 2 Başlarken
##########################

Bu ders giriş niteliğinde, Zend Framework 2 ile Model-View-Controller paradigması 
kullanarak, veritabanı desteği olan basit bir uygulamanın nasıl 
oluşturulacağına yönelik hazırlanmıştır. Dersin sonunda çalışan bir ZF2 
uygulamanız olacak. Sonrasında nasıl çalıştığı hakkında daha fazla bilgi 
edinmek için kodu kurcalayabilirsiniz.

.. note::

    **Çeviri notu:** Yazılım dünyasında genel kabül görmüş ve belirli bir durum veya 
    özelliği tanımlayan/anlatan bazı terimlerin Türkçe’ye çevrimi çok zordur. Örneğin:
    clousure, placeholder, hydrator vb. Bu terimlerin doğrudan kelime anlamını vermek
    bazı anlam kaymalarına neden olacak ve dökümanının anlaşılmasını zorlaştıracaktır.
    
    Bu nedenle bu terimleri türkçeye çevirmeden olduğu gibi bıraktım. Aşağıdaki listede
    bulunan terimleri ise en yakın türkçe kelimeye çevrimini yaptım. Dökümanı okumadan önce
    bu listeye bir göz atmanızda fayda var.
    
    * controller:	denetçi
    * action:		eylem
    * view:			görüntü
    * routing:		yönlendirme
    * router:		yönlendirici
    * route:		yön
    * constructor:	yapıcı
    * helper:		yardımcı
    * plugin:		eklenti
    * extend:		genişletmek
    * implement:	uyarlamak
    * interface:	arayüz

.. _user-guide.overview.assumptions:

Bazı Varsayımlar
----------------

Bu ders, PDO uzantısına erişilebilen PHP 5.3.10, Apache web sunucusu ve MySQL 
çalıştırdığınızı varsayar. Apache, mod_rewrite uzantısı ile kurulmuş
ve yapılandırılmış olmalıdır.

Ayrıca Apache'nin ``.htaccess`` dosya desteği ile yapılandırılmış olduğuna emin olunuz.
Genelde, ``httpd.conf`` dosyasında;

.. code-block:: apache

    AllowOverride None

ayarı

.. code-block:: apache

    AllowOverride  All

şeklinde değiştirilerek yapılır.

Tam detaylar için dağıtımınızın dökümanlarına bakınız. mod_rewrite ve .htaccess 
yapılandırmanız doğru olmadığı müddetçe bu dersteki ana sayfa dışında diğer 
sayfaları gezinmek mümkün olmayacaktır.

Ders Uygulaması
---------------

Yapacağımız uygulama, sahip olduğumuz albümleri listeleyen basit bir envanter 
sistemi olacak. Ana sayfa albüm kolleksiyonumuzu listeleyecek ve CD’leri ekleme, 
düzenleme ve silmemize olanak sağlayacak. Web sitemiz için dört sayfaya
ihtiyacımız olacak:

+----------------+------------------------------------------------------------+
| Sayfa          | Açıklama                                                   |
+================+============================================================+
| Albüm Listesi  | Bu sayfa albüm listesini görüntüleyecek ve düzenleme ve    |
|                | silme için link sağlayacak. Aynı zamanda yeni albüm        |
|                | eklemek için bir bağlantı sağlayacak.                      |
+----------------+------------------------------------------------------------+
| Yeni albüm     | Bu sayfa yeni bir albüm eklemek için form sağlayacak       |
+----------------+------------------------------------------------------------+
| Albüm düzenle  | Bu sayfa bir albümü düzenlemek için form sağlayacak        |
+----------------+------------------------------------------------------------+
| Albüm Sil      | Bu sayfa istediğimiz bir albümün silinmesi için bir onay   |
|                | sağlayacak.                                                |
+----------------+------------------------------------------------------------+

Aynı zamanda verileri tutmak için bir veritabanına ihtiyacımız olacak. Sadece
içinde aşağıdaki alanlar olan bir tabloya ihtiyacımız var.

+------------+--------------+-------+-----------------------------+
| Alan Adı   | Tip          | Boş?  | Notlar                      |
+============+==============+=======+=============================+
| id         | integer      | No    | Primary key, auto-increment |
+------------+--------------+-------+-----------------------------+
| artist     | varchar(100) | No    |                             |
+------------+--------------+-------+-----------------------------+
| title      | varchar(100) | No    |                             |
+------------+--------------+-------+-----------------------------+
