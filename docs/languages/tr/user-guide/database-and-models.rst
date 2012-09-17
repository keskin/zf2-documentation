.. EN-Revision: none
.. _user-guide.database-and-models:

######################
Veritabanı ve Modeller
######################

Veritabanı
----------

``Album`` modülümüz, denetleyici eylem metodları ve görüntü
dosyaları ile birlikte kuruldu ve şimdi uygulamamızın model bölümüne bakma zamanı 
geldi. Modellerin uygulamanın ana amacı ile ilgili olduğunu unutmayın. Uygulamamız 
için model, veritabanı ile ilgilidir. Zend Framework ``Zend\Db\TableGateway\TableGateway`` 
sınıfını, veritabanı tablosundan arama, ekleme, güncelleme ve silme işlemleri yapmak 
için kullanacağız.

PHP PDO sürücüsü üzerinden MySQL kullanacağız. Bu nedele ``zf2tutorial`` adında
bir veritabanı yaratalım ve album tablosu oluşturup içine birkaç test verisi eklemek
için aşağıdaki SQL’i çalıştıralım.

.. code-block:: sql

    CREATE TABLE album (
      id int(11) NOT NULL auto_increment,
      artist varchar(100) NOT NULL,
      title varchar(100) NOT NULL,
      PRIMARY KEY (id)
    );
    INSERT INTO album (artist, title)
        VALUES  ('The  Military  Wives',  'In  My  Dreams');
    INSERT INTO album (artist, title)
        VALUES  ('Adele',  '21');
    INSERT INTO album (artist, title)
        VALUES  ('Bruce  Springsteen',  'Wrecking Ball (Deluxe)');
    INSERT INTO album (artist, title)
        VALUES  ('Lana  Del  Rey',  'Born  To  Die');
    INSERT INTO album (artist, title)
        VALUES  ('Gotye',  'Making  Mirrors');

(Seçilen test verisi bu dökümanın yazıldığı anda Amazon UK’da en çok satanlardır!)

Veritabanımızda birkaç verimiz var. Şimdi basit bir model yazabiliriz.

Model Dosyaları
---------------

Zend Framework iş mantığınız ile ilgili bir ``Zend\Model`` bileşeni sağlamaz. 
ve iş mantığınızın nasıl çalışacağına karar vermek size kalmıştır. Bunun için 
ihtiyaçlarınıza göre birçok bileşen vardır. Bir yaklaşıma göre; Model sınıflarının 
uygulamanızdaki her varlığı temsil etmesi sonra varlıkları veritabanına yükleyip 
kaydetmesidir. Bir diğeri ise Doctrine veya Propel gibi bir ORM kullanmaktır.

Bu derste, her albümün ``Album`` nesnesi (varlık-*entity*- olarak bilinir) olduğu
``Zend\Db\TableGateway\TableGateway`` sınıfını genişleten(extend) ``AlbumTable`` 
sınıfı yaratarak çok basit bir model oluşturacağız. Bu model, veritabanı 
tablosundaki kayıtlara erişimi sağlayan bir arabirim olan Table Data Gateway
tasarım deseni uyarlamasıdır. Table Data Gateway deseninin büyük sistemlerde
sınırlayıcı olabileceğini unutmayın. ``Zend\Db\TableGateway\AbstractTableGateway``
ile veritabanı erişim kodlarını denetleyici (controller) eylem(action) metodları 
içine koymak gibi bir hata yapmayın!

``Model`` dizinindeki ``Album`` varlık sınıfı ile başlayalım:

.. code-block:: php

    // module/Album/src/Album/Model/Album.php:
    namespace Album\Model;

    class Album
    {
        public $id;
        public $artist;
        public $title;

        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id'])) ? $data['id'] : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title'])) ? $data['title'] : null;
        }
    }

``Album`` varlık nesnemiz basit bir PHP dosyasıdır. ``Zend\Db``’nin 
``AbstractTableGateway`` sınıfı ile çalışmak için ``exchangeArray()`` metodunu
uyarlamamız gerekir. Bu metod basitçe dizi ile gelen veriyi varlığımızın
özelliklerine aktarır. Daha sonra bu özellikleri kullanmak üzere formumuza filtre
kutusu ekleyeceğiz.

Sırada modülümüzün ``Model`` dizininde ``Zend\Db\TableGateway\AbstractTableGateway``’i 
genişleten ``AlbumTable`` sınıfımız var.

.. code-block:: php

    // module/Album/src/Album/Model/AlbumTable.php:
    namespace Album\Model;

    use Zend\Db\Adapter\Adapter;
    use Zend\Db\ResultSet\ResultSet;
    use Zend\Db\TableGateway\AbstractTableGateway;

    class AlbumTable extends AbstractTableGateway
    {
        protected $table ='album';

        public function __construct(Adapter $adapter)
        {
            $this->adapter = $adapter;
            $this->resultSetPrototype = new ResultSet();
            $this->resultSetPrototype->setArrayObjectPrototype(new Album());
            $this->initialize();
        }

        public function fetchAll()
        {
            $resultSet = $this->select();
            return $resultSet;
        }

        public function getAlbum($id)
        {
            $id  = (int) $id;
            $rowset = $this->select(array('id' => $id));
            $row = $rowset->current();
            if (!$row) {
                throw new \Exception("Could not find row $id");
            }
            return $row;
        }

        public function saveAlbum(Album $album)
        {
            $data = array(
                'artist' => $album->artist,
                'title'  => $album->title,
            );
            $id = (int)$album->id;
            if ($id == 0) {
                $this->insert($data);
            } else {
                if ($this->getAlbum($id)) {
                    $this->update($data, array('id' => $id));
                } else {
                    throw new \Exception('Form id does not exist');
                }
            }
        }

        public function deleteAlbum($id)
        {
            $this->delete(array('id' => $id));
        }
    }

Öncelikle; ``$table`` korunan(protected) özelliğine
veritabanı tablo ismi olan ‘album’’ü atadık. Sadece veritabanı bağdaştırıcısı
(Adapter) parametresi alan ve bu parametreyi sınıfın ``$adapter`` bağdaştırıcısına 
atayan bir yapıcı(constructor) yazdık. Sonra table gateway’in sonuç kümesine 
her zaman yeni bir kayıt nesnesi oluşturması ve bu işlem için ``Album`` nesnesini 
kullanması gerektiğini söyledik. ``TableGateway`` sınıfları sonuç kümeleri(result set)
ve varlıkları(entity) oluşturmak için prototype tasarım desenini kullanır. Böylece
sistem daha önce oluşturulmuş neseneyi kullanır, yoksa sadece gerektiğinde nesne 
oluşturulur. Detaylar için: `PHP Constructor Best Practices and the Prototype Pattern 
<http://ralphschindler.com/2012/03/09/php-constructor-best-practices-and-the-prototype-pattern>`_.

Daha sonra uygulamamızın veritabanı tablosu ile arayüzünü kullanacak birkaç yardımcı
metod oluşturduk. ``fetchAll()`` ``ResultSet`` olarak veritabanından bütün
kayıtları getirir. ``getAlbum()`` ``Album`` nesnesi olarak tek bir kayıt getirir. 
``saveAlbum()`` yeni bir kayıt oluşturur veya var olan kaydı günceller. ``deleteAlbum()``
kaydı tamamen siler. Üç metoddaki kodlar oldukça açık zaten.

ServiceManager kullanarak veritabanı erişim bilgilerini yapılandırma ve denetçi’ye aktarma
------------------------------------------------------------------------------------------------------

Daima aynı ``AlbumTable`` örneğimizin kullanılması için, ``ServiceManager``’a örneği
nasıl oluşturacağını tanımlamalıyız. Bu en kolay, ``ModuleManager`` tarafından 
otomatik olarak çağrılan ve ``ServiceManager``’a uygulanan ``getServiceConfig()``
metodu ile yapılır. Böylece örneğimize ihtiyacımız olduğunda denetçimizden kolayca 
erişebiliriz. 

``ServiceManager``’ı yapılandırmak için, temsil edecek sınıfın ismini veya
nesnelerin örneğini oluşturan bir factory(closure veya callback) sağlayabiliriz.
``AlbumTable`` oluşturan bir factory sağlamak için ``getServiceConfig()`` metodunu
uyarlamaya başlayalım. Aşağıdaki metodu ``Module`` sınıfının en altına bu metodu
ekleyin.

.. code-block:: php

    // module/Album/Module.php:
    namespace Album;

    // Add this import statement:
    use Album\Model\AlbumTable;

    class Module
    {
        // getAutoloaderConfig() and getConfig() methods here

        // Add this method:
        public function getServiceConfig()
        {
            return array(
                'factories' => array(
                    'Album\Model\AlbumTable' =>  function($sm) {
                        $dbAdapter = $sm->get('Zend\Db\Adapter\Adapter');
                        $table     = new AlbumTable($dbAdapter);
                        return $table;
                    },
                ),
            );
        }
    }

Bu metod, ``ServiceManager``’a geçmeden önce ``ModuleManager`` tarafından
birleştirilmiş bir ``factories`` dizisi döndürür. Ayrıca ``ServiceManager``’ı
``Zend\Db\Adapter\Adapter`` nesnesini nasıl alacağını bileceği şekilde
yapılandırmalıyız. Bunu; ``Zend\Db\Adapter\AdapterServiceFactory`` nesnesini
çağıran bir factory kullanarak birleştirilmiş yapılandırma sistemi içinde
kolayca yapabiliriz. Zend Framework 2’nin ``ModuleManager``’ı her modülün
``module.config.php`` dosyasını birleştirir ve sonrasında ``config/autoload``’da
(``*.global.php`` ve ``*.local.php`` dosyaları) bulunan dosyalar ile birleştirir.

Şimdi versiyon kontrol sistemine gönderilmiş ``global.php`` dosyasına veritabanı
bağlantısı yapılandırma bilgisini ekleyeceğiz. İsterseniz veritabanı erişim bilgilerini 
``local.php`` dosyasında tutabilirsiniz.

.. note::

    Burada anlatılmak istenen; veritabanı ile ilgili korunması gereken kullanıcı adı
    ve şifre gibi bilgilerin açığa çıkmaması için local.php dosyasının versiyon 
    kontrol sistemlerinde(svn, csv, git vb.) gözükmemesi gerektiğidir. Bu nedenle 
    bağlantı sürücüsü, veri kaynağı ve bağlantı seçenekleri global.php de kullanıcı
    adı ve şifre bilgileri ise local.php de tutuluyor.

.. code-block:: php

    // config/autoload/global.php:
    return array(
        'db' => array(
            'driver'         => 'Pdo',
            'dsn'            => 'mysql:dbname=zf2tutorial;host=localhost',
            'driver_options' => array(
                PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
            ),
        ),
        'service_manager' => array(
            'factories' => array(
                'Zend\Db\Adapter\Adapter' 
                        => 'Zend\Db\Adapter\AdapterServiceFactory',
            ),
        ),
    );

Veritabanı kimlik bilgilerini github depoda bulunmayan ``config/autoload/local.php``
dosyasında tutmalısınız. (github depo da local.php dosyası göz ardı edilir.)

.. code-block:: php

    // config/autoload/local.php:
    return array(
        'db' => array(
            'username' => 'YOUR USERNAME HERE',
            'password' => 'YOUR PASSWORD HERE',
        ),
    );

Şimdi ``ServiceManager`` bir ``AlbumTable`` örneği yaratabilir. Bu örneğe erişmesi
için denetçiye bir metod ekleyebiliriz. ``AlbumController`` sınıfına ``getAlbumTable()``
metodunu ekleyelim.

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:
        public function getAlbumTable()
        {
            if (!$this->albumTable) {
                $sm = $this->getServiceLocator();
                $this->albumTable = $sm->get('Album\Model\AlbumTable');
            }
            return $this->albumTable;
        }

Aynı zamanda sınıfın başına:

.. code-block:: php

    protected $albumTable;

kodunu eklemelisiniz.

Denetçimiz içinden istediğimiz zaman modelimizle etkileşime geçecek 
``getAlbumTable()`` metodunu çağırabiliriz. Şimdi ``index`` eylemi çağrılınca
albümleri listeleyelim.

Albüm Listesi
-------------

Albümleri listelemek için, modelden verileri alıp görüntü dosyasına aktarmalıyız. 
Bunun için ``AlbumController`` içinde ``indexAction()`` eylemini yazmalıyız:

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:
    // ...
        public function indexAction()
        {
            return new ViewModel(array(
                'albums' => $this->getAlbumTable()->fetchAll(),
            ));
        }
    // ...

Zend Framework 2'de, görüntüye değişkenler gönderebilmek için, ilk parametresi, 
ihtiyacımız olan veriyi içeren diziye sahip ``ViewModel`` örneği döndürürüz.
``ViewModel`` nesnesi aynı zamanda hangi görüntü dosyasını kullanılacağını
belirlememize olanak sağlar. Fakat kullanılan, varsayılan dosya
``{controller name}/{action name}`` ’dir. Şimdi ``index.phtml`` görüntü dosyasını
oluşturalım.

.. code-block:: php

    <?php 
    // module/Album/view/album/album/index.phtml:

    $title = 'My albums';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <p><a href="<?php echo $this->url('album', array( 
            'action'=>'add'));?>">Add new album</a></p>

    <table class="table">
    <tr>
        <th>Title</th>
        <th>Artist</th>
        <th>&nbsp;</th>
    </tr>
    <?php foreach($albums as $album) : ?>
    <tr>
        <td><?php echo $this->escapeHtml($album->title);?></td>
        <td><?php echo $this->escapeHtml($album->artist);?></td>    <td>
            <a href="<?php echo $this->url('album',
                array('action'=>'edit', 'id' => $album->id));?>">Edit</a>
            <a href="<?php echo $this->url('album',
                array('action'=>'delete', 'id' => $album->id));?>">Delete</a>
        </td>
    </tr>
    <?php endforeach; ?>
    </table>

Yaptığımız ilk iş sayfa başlığını ayarlamak (layout içinde kullanılan) ve aynı 
zamanda ``headTitle()`` görüntü yardımcısını kullanarak tarayıcının başlık çubuğunda 
görüntülenen ``<head>`` bölümü için başlığı ayarlamak. Sonrasında yeni albüm eklemek
için bir link oluşturduk.

Zend Framework 2 tarafından sağlanan ``url()`` görüntü yardımcısı ihtiyacımız olan linkleri 
oluşturmak için kullanılır. ``url()``’in ilk parametresi URL’i oluşturmak için
kullanmak istediğimiz yoldur. İkinci parametre ise kullanılacak placeholderlar 
içine eşleşen bütün değişkenleri tutan bir dizidir. Örneğimizde yol için ‘album’
placeholder değişkenleri için ``action`` ve ``id``’yi kullandık.

Denetçi eyleminden atadığımız ``$albums`` değişkenini ele alalım. Zend 
Framework 2 görüntü sistemi, değişkenleri otomatik olarak görüntü dosyası kapsamında
parçalar(extract). Böylece Zend Framework 1’deki gibi değişkenlerin başına 
``$this->`` eklemek zorunda kalmıyoruz. Fakat isterseniz ``$this->var``
şeklinde de kullanabilirsiniz.

Daha sonra her bir albümün başlık ve sanatçısını listeleyen ve düzenleme ve silme
işlevi sağlayan bir tablo oluşturduk. Standart bir ``foreach:`` döngüsü ile albüm 
listesini yazdırdık. Ve tekrar düzenleme ve silme linkleri oluşturmak için ``url()`` 
görüntü yardımcısını kullandık.

.. note::

    XSS açıklarından korunmak için her zaman ``escapeHtml()`` görüntü yardımcısını
	kullanırız.

http://zf2-tutorial.localhost/album sayfasını açtığınızdaki şöyle bir ekran görmelisiniz:

.. image:: ../images/user-guide.database-and-models.album-list.png
    :width: 940 px
