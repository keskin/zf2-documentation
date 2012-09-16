.. EN-Revision: 4e4055e
.. _user-guide.overview:
##########################
Zend Framework 2 Başlarken
##########################

Bu ders, Zend Framework 2 ile Model-View-Controller paradigması kullanarak
giriş niteliğinde veritabanı desteği olan basit bir uygulamanın nasıl 
oluşturulacağına yönelik hazırlanmıştır. Dersin sonunda çalışan bir ZF2 
uygulamanız olacak. Sonrasında nasıl çalıştığı hakkında daha fazla bilgi 
edinmek için kodu kurcalayabilirsiniz.

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

Tam detaylar için dağıtımınızın dökümanlarına bakınız.
mod_rewrite ve .htaccess yapılandırmanız doğru olmadığı müddetçe bu dersteki
ana sayfa dışında diğer sayfaları gezinmek mümkün olmayacaktır.

Ders Uygulaması
---------------

Yapacağımız uygulama, sahip olduğumuz albümleri listeleyen basit bir envanter 
sistemi olacak. Ana sayfa albüm kolleksiyonumuzu listeleyecek ve CD leri ekleme, 
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
