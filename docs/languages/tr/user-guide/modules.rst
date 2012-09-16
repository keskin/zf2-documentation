.. EN-Revision: 48c0b96
.. _user-guide.modules:

########
Modüller
########

Zend Framework 2, modül sistemi kullanır ve her modül içinde uygulamaya özgü 
kodlarınızı organize edersiniz. İskelet uygulaması tarafından sağlanan 
Application modülü ise, bütün uygulama için bootstrapping sağlaması ve hata ve 
yönlendirme yapılandırması için kullanılır. Modüller genellikle, örneğin; bir 
uygulamanın giriş sayfası için uygulama seviyesi kontrol sağlamak için kullanılır.
Fakat bu derste İskelet Uygulaması tarafından sağlanan modülü kullanmayacağız. 
Kendi modülümüzde bulunacak albüm listesini ana sayfa olarak kullanmayı düşünüyoruz. 

Bütün kodlarımızı kontroller, modeller, formlar ve görüntü dosyalarını içeren Albüm
modülü içine koyacağız. Ayrıca gerektiğinde Application modülüne de el atacağız.

Gerekli olan dizinler ile başlayalım.

Albüm modülünü yapılandırma
---------------------------

Modül dosyalarını tutmak için aşağıdaki alt dizinleri içeren ``Album`` dizinini
oluşturmakla başlayalım:

.. code-block:: text

    zf2-tutorial/
        /module
            /Album
                /config
                /src
                    /Album
                        /Controller
                        /Form
                        /Model
                /view
                    /album
                        /album

Gördüğünüz gibi ``Album`` modülü ayrı dizinler ve farklı türdeki dosyalardan oluşmaktadır.
``Album`` namespace inde bulunan sınıflarımızı içeren PHP dosyaları ``src/Album``
dizininde bulunur. Böylece gerektiğinde modülümüz için birden fazla namespace e
sahip olabiliriz.

Zend Framework 2 modülleri yüklemek ve yapılandırmak için ``ModuleManager``
kullanır. ``ModuleManager`` modül ana dizininde (``module/Album``), ``Module.php``
ye bakar ve içinde ``Album\Module`` adında sınıf olması gerekir. Modül
içindeki sınıflar modül adı(modül dizininin adı) namespace inde bulunmalıdır.

``Album`` modülünde ``Module.php`` oluştur:

.. code-block:: php

    // module/Album/Module.php
    namespace Album;
    
    class Module
    {
        public function getAutoloaderConfig()
        {
            return array(
                'Zend\Loader\ClassMapAutoloader' => array(
                    __DIR__ . '/autoload_classmap.php',
                ),
                'Zend\Loader\StandardAutoloader' => array(
                    'namespaces' => array(
                        __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                    ),
                ),
            );
        }
    
        public function getConfig()
        {
            return include __DIR__ . '/config/module.config.php';
        }
    }

``ModuleManager``, bizim için ``getAutoloaderConfig()`` ve ``getConfig()`` 
metodlarını otomatik olarak çağırır.

Otomatik dosya yükleme
^^^^^^^^^^^^^^^^^^^^^^

``getAutoloaderConfig()`` metodumuz ZF2'nin ``AutoloaderFactory`` sine uygun
boş bir dizi döndürür. ``ClassmapAutoloader`` a bir sınıf eşleşme dosyası
ekleyerek yapılandırırız ve aynı zamanda modül namespace ini ``StandardAutoloader`` a
ekleriz. Standart autoloader için, namespace ve namespace için gerekli olan dosyaların
bulunduğu dizin bilgisi gerekir. Bu PSR-0 uyumludur ve böylece sınıflar 
`PSR-0 kuralları
<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_ na
göre dosyaları doğrudan eşler.

Geliştirme aşamasında olduğumuz için dosyaları classmap ile yüklemeye ihtiyacımız yok.
Bu nedenle clasmap autoloader a boş bir dizi sağlayalım.

.. code-block:: php

    // module/Album/autoload_classmap.php:
    return array();

Bu boş bir dizi olduğu için, autoloader ``Album`` namespace inde bir sınıf aradığında,
``StandardAutoloader`` a geri düşecektir.

.. note::

    Alternatif olarak Composer kullandığımızda, ``getAutoloaderConfig()`` uyarlaması
    yapamazsınız ve ``composer.json`` a "module/Application/src"`` ``psr-0`` anahtarı
    ekleyemezsiniz.

Yapılandırma
------------

Autoloader kaydını yaptırmışken, ``Album\Module`` içindeki ``getConfig()`` metoduna
bir göz atalım. Bu metod basitçe ``config/module.config.php`` dosyasını yükler.

``Album`` modülü için aşağıdaki yapılandırma dosyasını oluşturalım.

.. code-block:: php

    // module/Album/conﬁg/module.config.php:
    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),
        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

Yapılandırma bilgisi ``ServiceManager`` ile ilgili bileşenlere aktarılır.
Yapılandırma için ``controllers`` ve ``view_manager`` bölümleri gerekir. 
controllers bölümü modül tarafından sağlanan tüm kontrollerin listesini içerir. 
Burada ``AlbumController`` adında bir kontrole ihtiyacımız olacak. kontrol
anahtarı tüm modüller içinde benzersiz olmalı. Bu yüzden kontrol adının başına
modül adını ekledik.

``view_manager`` bölümünde, ``TemplatePathStack`` yapılandırması ile görüntü
dosyaları dizinini ekleriz. Bu, Album modülü için gerekli görüntü dosyalarının 
``views/`` dizininde bulmasını sağlar.

Uygulamaya yeni modülü tanıtma
------------------------------

Şimdi ``ModuleManager`` a yeni bir modülümüzün olduğunu söylemeliyiz. Bu, İskelet 
uygulaması tarafından sağlanan ``config/application.config.php`` dosyası ile yapılır.
``modules`` bölümüne ``Album`` modülünü ekleyin. Böylece dosyanın son hali:

(Değişiklik açıklama bölümü ile belirtilmiştir.)

.. code-block:: php

    // conﬁg/application.conﬁg.php:
    return array(
        'modules' => array(
            'Application',
            'Album',                  // <-- Add this line
        ),
        'module_listener_options' => array( 
            'config_glob_paths'    => array(
                'config/autoload/{,*.}{global,local}.php',
            ),
            'module_paths' => array(
                './module',
                './vendor',
            ),
        ),
    );

Gördüğünüz gibi, ``Album`` modülümüzü modül listesinde ``Application`` modülünün
altına ekledik.

Şimdi, modülümüz uygulamamıza özel kodları yazmak için hazır.
