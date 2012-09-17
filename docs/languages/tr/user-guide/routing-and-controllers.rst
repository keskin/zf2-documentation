.. EN-Revision: none
.. _user-guide.routing-and-controllers:

#########################################
Yönlendirme(Route) ve Denetçi(controller)
#########################################

Albüm kolleksiyonumuzu göstermek için basit bir envanter sistemi kuracağız.
Ana sayfa, kolleksiyonumuzu listeleyecek ve ekleme, düzenleme ve silmemizi
sağlayacak. Bu nedenle aşağıdaki sayfalar gerekecektir.

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

Dosyalarımızı oluşturmadan önce, frameworkün sayfaların düzeni ile ilgili ne
beklediğini anlamak çok önemlidir. Bütün sayfalar eylem *action* olarak bilinir ve 
eylemler *modules* içindeki denetçiler *controllers* içinde guruplandırılmıştır.
Genelde eylemler(action), Denetçiler(controllers) içinde gruplandırılır. 
Örneğin; Bir haberler denetçisinin(controller) ``güncel``, ``arşiv``, ``görüntüle`` 
eylemleri(action) olabilir.

Dört sayfamız olduğu için, dört eylemi(action) ``Album`` modülü içinde 
``AlbumController`` adında tek bir denetçide(controller) gruplayacağız. Dört 
eylemimiz(action) aşağıdaki gibi olacak:

+---------------+---------------------+---------------+
| Sayfa         | Denetçi(Controller) | Eylem(Action) |
+===============+=====================+===============+
| Ana Sayfa     | ``AlbumController`` | ``index``     |
+---------------+---------------------+---------------+
| Yeni albüm    | ``AlbumController`` | ``add``       |
+---------------+---------------------+---------------+
| Albüm düzenle | ``AlbumController`` | ``edit``      |
+---------------+---------------------+---------------+
| Albüm sil     | ``AlbumController`` | ``delete``    |
+---------------+---------------------+---------------+

Bir URL ile belirli bir eylemin(action) eşleşmesi, modülün ``module.config.php`` 
dosyasında tanımlanmış yönlendirmeler(routes) ile yapılır. ``album`` eylemimize(action)
bir yönlendirme(route) tanımlayacağız. Yeni kodumuz açıklaması ile birlikte güncellenmiş
yapılandırma dosyamız:

.. code-block:: php

    // module/Album/conﬁg/module.conﬁg.php:
    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),

        // aşağıdaki yeni bölümü dosyanıza eklemelisiniz
        'router' => array(
            'routes' => array(
                'album' => array(
                    'type'    => 'segment',
                    'options' => array(
                        'route'    => '/album[/:action][/:id]',
                        'constraints' => array(
                            'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            'id'     => '[0-9]+',
                        ),
                        'defaults' => array(
                            'controller' => 'Album\Controller\Album',
                            'action'     => 'index',
                        ),
                    ),
                ),
            ),
        ),

        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

Yönlendirme(route) adı ‘album’ ve tipi ‘segment’ tir. Segment yönlendirmeler, URL
kalıbı içinde isim parametrelerinin eşleşmesi için belirli placeholderlar sağlar.
Yapılandırma dosyamızda bulunan **``/album[/:action][/:id]``** segment yönlendirmesi
göre ``/album`` ile başlayan bütün URL ler ile eşleşir. Sonraki segment isteğe bağlı
eylem(action) adı ``[/:action]``, diğer segment ise isteğe bağlı id alanı ``[/:id]``
ile eşleşir. Köşeli parantezler segmentin isteğe bağlı olduğunu belirtir.
``constraints`` bölümü segmentlerin beklenen karakter kurallarını belirlememizi sağlar.
Bu durumda ``action`` harf ile başlayıp sonraki karakterleri harf, rakam, - ve _ olabilecek
şekilde kısıtlanmıştır. ``id`` ise sayısal olarak sınırlandırılmıştır.

Bu yönlendirme bize aşağıdaki URL'leri sağlar:

+---------------------+------------------------------+------------+
| URL                 | Sayfa                        | Eylem      |
+=====================+==============================+============+
| ``/album``          | Ana Sayfa (albüm listesi)    | ``index``  |
+---------------------+------------------------------+------------+
| ``/album/add``      | Yeni Albüm                   | ``add``    |
+---------------------+------------------------------+------------+
| ``/album/edit/2``   | id si 2 olan albümü düzenle  | ``edit``   |
+---------------------+------------------------------+------------+
| ``/album/delete/4`` | id si 4 olan albümü sil      | ``delete`` |
+---------------------+------------------------------+------------+

Denetçiyi(controller) Oluştur
=============================

Şimdi denetçimizi(controller) kurmak için hazırız. Zend Framework 2'de, denetçi 
(controller) ``{Controller name}Controller`` adlandırılan bir sınıftır. 
``{Controller name}`` 'in büyük harf ile başlaması gerektiğini unutmayın. Bu sınıf
``Controller`` dizinindeki ``{Controller name}Controller.php`` dosyası içindedir.
Dersimize göre ``module/Album/src/Album/Controller`` dizinidir. Her eylemin(action)
denetçi(controller) içinde ``{action name}Action`` adında public metodu vardır.
``{action name}`` küçük harf ile başlamalıdır.

.. note::

    Kural Gereği: Zend Framework 2 denetçiler üzerinde ``Zend\Stdlib\Dispatchable``
    arabirimini implemente etmedikleri sürece herhangi bir kısıtlama sağlamaz.
    Framework, bunun için bize 2 soyut sınıf sağlar: 
    ``Zend\Mvc\Controller\AbstractActionController`` ve 
    ``Zend\Mvc\Controller\AbstractRestfulController``. Biz standart
    ``AbstractActionController`` sınıfını kullanacağız. Fakat RESTful web servisi
    yazacaksanız ``AbstractRestfulController`` sizin için kullanışlı olabilir.

Denetçi(controller) sınıfımızı oluşturarak devam edelim:

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:
    namespace Album\Controller;

    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;
    
    class AlbumController extends AbstractActionController
    {
        public function indexAction()
        {
        }
    
        public function addAction()
        {
        }
    
        public function editAction()
        {
        }
    
        public function deleteAction()
        {
        }
    }

.. note::

    Modülümüze, ``config/module.config.php`` dosyasındaki ‘controller’
    bölümümüzde denetçimiz hakkında bilgi vermiştik.

Şimdi kullanmak istediğimiz dört eylemi yazalım. Eylemler, görüntü(view) dosyalarını 
oluşturmadan çalışmazlar. Her eylem için URL'ler aşağıdaki gibidir:

+--------------------------------------------+----------------------------------------------------+
| URL                                        | Çağrılan metod                                     |
+============================================+====================================================+
| http://zf2-tutorial.localhost/album        | ``Album\Controller\AlbumController::indexAction``  |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/add    | ``Album\Controller\AlbumController::addAction``    |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/edit   | ``Album\Controller\AlbumController::editAction``   |
+--------------------------------------------+----------------------------------------------------+
| http://zf2-tutorial.localhost/album/delete | ``Album\Controller\AlbumController::deleteAction`` |
+--------------------------------------------+----------------------------------------------------+

Şu an uygulamamızın çalışan bir yönlendiricisi(router) ve eylemleri hazır.

Görüntü(view) ve modellerimizi oluşturmanın zamanı geldi.

Görüntü dosyalarını hazırlayalım
--------------------------------

Uygulamamıza görüntü entegre etmek için tek yapmamız gereken birkaç görütü(view)
dosyası oluşturmaktır. Görüntü dosyaları ``DefaultViewStrategy`` tarafından çalıştırılacak,
denetçi(controller) eylemine(action) değişken olarak aktarılacak veya görüntü modeli 
olarak dönecektir. Görüntü dosyaları modül görüntü dizini içindeki adı denetçi adı olan
dizinde bulunur. Şimdi aşağıdaki isimlerde boş görüntü dosyaları oluşturalım.

* ``module/Album/view/album/album/index.phtml``
* ``module/Album/view/album/album/add.phtml``
* ``module/Album/view/album/album/edit.phtml``
* ``module/Album/view/album/album/delete.phtml``

Şimdi veritabanı ve modeller ile eksiklerimizi giderebiliriz. 
