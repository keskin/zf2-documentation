.. EN-Revision: none
.. _user-guide-forms-and-actions:

###################
Formlar ve Eylemler
###################

Yeni albüm ekleme
-----------------

Şimdi yeni albüm ekleme işlevi için kod yazalım. Bu işlevde iki nokta var:

* Kullanıcıya detayları girmesi için bir form görüntüleme
* Formu işleme ve veritabanına kaydetme
  
Bu işlemler için ``Zend\Form``’u kullanacağız. ``Zend\Form`` bileşeni form ve 
doğrulamayı yöneterek ``Album`` varlığımız için ``Zend\InputFilter`` ekler.
Formu tanımlamak için ``module/Album/src/Album/Form`` dizininde ``AlbumForm.php`` 
dosyasında bulunan ``Zend\Form\Form`` sınıfını genişleten(extend) 
``Album\Form\AlbumForm`` sınıfını oluşturmakla başlayalım:

.. code-block:: php

    // module/Album/src/Album/Form/AlbumForm.php:
    namespace Album\Form;

    use Zend\Form\Form;

    class AlbumForm extends Form
    {
        public function __construct($name = null)
        {
            // we want to ignore the name passed
            parent::__construct('album');
            $this->setAttribute('method', 'post');
            $this->add(array(
                'name' => 'id',
                'attributes' => array(
                    'type'  => 'hidden',
                ),
            ));
            $this->add(array(
                'name' => 'artist',
                'attributes' => array(
                    'type'  => 'text',
                ),
                'options' => array(
                    'label' => 'Artist',
                ),
            ));
            $this->add(array(
                'name' => 'title',
                'attributes' => array(
                    'type'  => 'text',
                ),
                'options' => array(
                    'label' => 'Title',
                ),
            ));
            $this->add(array(
                'name' => 'submit',
                'attributes' => array(
                    'type'  => 'submit',
                    'value' => 'Go',
                    'id' => 'submitbutton',
                ),        
            ));
        }
    }

``AlbumForm`` sınıfının yapıcısında *constructor*, ebeveyn yapıcısını çağırarak
form ismini atadık. Daha sonra, id, artist, title form elemanlarını ve submit 
butonunu oluşturduk. Her form elemanı için çeşitli nitelik ve seçenekler atadık.
Görüntülenecek etiket vb. gibi.

Aynı zamanda bu form için bir doğrulama kurmamız gerekir. Zend Framework 2’de
doğrulama, input filter kullanarak, bağımsız yada model varlığı gibi
``InputFilterAwareInterface`` arayüzünü uyarlayan bir sınıf içinde yapılabilir.
Bu örnekte ``Album`` varlığımıza input filter ekleyeceğiz.

.. code-block:: php

    // module/Album/src/Album/Model/Album.php:
    namespace Album\Model;

    use Zend\InputFilter\Factory as InputFactory;
    use Zend\InputFilter\InputFilter;
    use Zend\InputFilter\InputFilterAwareInterface;
    use Zend\InputFilter\InputFilterInterface;

    class Album implements InputFilterAwareInterface
    {
        public $id;
        public $artist;
        public $title;
        protected $inputFilter;

        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        public function setInputFilter(InputFilterInterface $inputFilter)
        {
            throw new \Exception("Not used");
        }

        public function getInputFilter()
        {
            if (!$this->inputFilter) {
                $inputFilter = new InputFilter();
                $factory     = new InputFactory();

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'id',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'Int'),
                    ),            
                )));

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'artist',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                )));

                $inputFilter->add($factory->createInput(array(
                    'name'     => 'title',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                )));

                $this->inputFilter = $inputFilter;        
            }

            return $this->inputFilter;
        }
    }

``InputFilterAwareInterface`` arayüzünde iki metod vardır. ``setInputFilter()``
ve ``getInputFilter()``. Sadece ``getInputFilter()`` metodunu uyarlama ihtiyacımız
var, ``setInputFilter()`` metodunda sadece bir hata oluşturacağız.

``getInputFilter()`` metodunda, bir ``InputFilter`` örneği oluşturarak gerekli
girişleri ekleyeceğiz. Filtre ve doğrulama istediğimiz her özellik için bir 
giriş ekleyeceğiz. ``id`` alanına sadece integer olması için bir ``Int`` filtresi
ekledik. Metin elemanlar için istenmeyen HTML ve gereksiz boş karakterleri 
kaldıran ``StripTags`` ve ``StringTrim`` filtrelerini ekledik.  Aynı zamanda
*required* (zorunlu) olarak ayarladık ve kullanıcının veritabanında saklayabileceğimiz
karakter sayısından daha fazla giriş yapamayacağından emin olmak için ``StringLength``
doğrulaması ekledik.

Şimdi formu görüntüleme ve gönderildikten sonraki işlemlere bakalım. Bu işlemler
``AlbumController`` denetçisinin ``addAction()`` eyleminde yapılır:

.. code-block:: php

    // module/Album/src/Album/Controller/AlbumController.php:

    //...
    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;
    use Album\Model\Album;          // <-- Add this import
    use Album\Form\AlbumForm;       // <-- Add this import
    //...

        // Add content to this method:
        public function addAction()
        {
            $form = new AlbumForm();
            $form->get('submit')->setValue('Add');

            $request = $this->getRequest();
            if ($request->isPost()) {
                $album = new Album();
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $album->exchangeArray($form->getData());
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }
            return array('form' => $form);
        }
    //...

``AlbumForm``’u namespace use listesine ekledikten sonra ``addAction()`` metodunu
uyarladık. Şimdi ``addAction()`` metodu kodlarına detaylı olarak bakalım.

.. code-block:: php

    $form = new AlbumForm();
    $form->submit->setValue('Add');

`AlbumForm` örneğini oluşturduk ve submit butonu etiketini “Add” olarak ayarladık.
Bunu, formu albüm düzenlemede farklı bir etiket ile tekrar kullanacağımız 
için yaptık.

.. code-block:: php

    $request = $this->getRequest();
    if ($request->isPost()) {
        $album = new Album();
        $form->setInputFilter($album->getInputFilter());
        $form->setData($request->getPost());
        if ($form->isValid()) {

Eğer ``Request`` nesnesinin ``isPost()`` metodu true döndürürse form gönderilmiş
demektir. *form*’un input filtresine *album* örneğinden aldığımız filtreyi gönderdik.
Daha sonra post edilmiş veriyi *form*’a göderdik ve formun ``isValid()`` metodu ile
geçerliliğini kontrol ettik.

.. code-block:: php

    $album->exchangeArray($form->getData());
    $this->getAlbumTable()->saveAlbum($album);

*form* geçerli ise, veriyi *form* dan alıp ``saveAlbum()`` metodu ile modele aktardık.

.. code-block:: php

    // Redirect to list of albums
    return $this->redirect()->toRoute('album');

Yeni albüm kaydını ekledikten sonra ``Redirect`` denetçi eklentisi ile albüm
listesine dönüş için yönlendirdik.

.. code-block:: php

    return array('form' => $form);

Son olarak görüntüye aktaracağımız değişkenleri(bu durumda sadece formu) atadık.
Zend Framework 2, aynı zamanda görüntüye aktarılacak gerekli değişkenleri içeren bir diziyi
basitçe geri döndürmenize izin verir. Aslında perde arkasında bir ``ViewModel`` yaratılır.

Şimdi formu görüntülemek için add.phtml görüntü dosyasına ihtiyacımız var:

.. code-block:: php

    <?php
    // module/Album/view/album/album/add.phtml:

    $title = 'Add new album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>
    <?php
    $form = $this->form;
    $form->setAttribute('action', $this->url('album', array('action' => 'add')));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

Daha önce yaptığımız gibi sayfa başlığını ayarladık sonra formu işledik. Zend 
Framework, form işlemeyi kolaylaştıran bazı görüntü yardımcıları sunar. ``form()``
görüntü yardımcılarında bulunan ``openTag()`` ve ``closeTag()`` metodları ile formu 
açıp kapattık (HTML de <form ...> ... </form>). Sonrasında, etiketi olan form 
elemanları için ``formRow()`` diğer iki eleman için ise ``formHidden()`` ve
``formSubmit()`` metodlarını kullandık.

.. image:: ../images/user-guide.forms-and-actions.add-album-form.png
    :width: 940 px

Şimdi yeni albüm kaydı eklemek için uygulama ana sayfasında bulunan 
“Add new album” linkini kullanabiliyor olmalısınız.

Albüm düzenleme
---------------

Albüm düzenleme, ekleme ile neredeyse aynıdır ve kodlar birbirine çok benzer.
Bu sefer ``AlbumController`` içinde ``editAction()`` eylemini kullanacağız.

.. code-block:: php

    // module/Album/src/Album/AlbumController.php:
    //...

        // Add content to this method:
        public function editAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album', array(
                    'action' => 'add'
                ));
            }
            $album = $this->getAlbumTable()->getAlbum($id);

            $form  = new AlbumForm();
            $form->bind($album);
            $form->get('submit')->setAttribute('value', 'Edit');
            
            $request = $this->getRequest();
            if ($request->isPost()) {
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }

            return array(
                'id' => $id,
                'form' => $form,
            );
        }
    //...

Bu kod oldukça tanıdık geliyor olmalı. Albüm ekleme kodundan farklılıklarına 
bakalım. İlk önce, eşleşen *route* ’da bulunan ve düzenlenecek albümü yüklemek
için kullanacağımız ``id``’ ye bakalım:

.. code-block:: php

    $id = (int) $this->params()->fromRoute('id', 0);
    if (!$id) {
        return $this->redirect()->toRoute('album', array(
            'action' => 'add'
        ));
    }
    $album = $this->getAlbumTable()->getAlbum($id);

``params`` eşleşen *route* ’dan uygun parametreleri getirmeye yarayan bir
denetçi eklentisidir. ``params`` ile modülün ``module.config.php`` dosyasında 
oluşturduğumuz *route* ’dan ``id``’yi almak için kullandık. Eğer ``id`` sıfır(0) 
ise ekle eylemine yönlendirdik, değilse veritabananından albüm kaydını getirdik.

.. code-block:: php

    $form = new AlbumForm();
    $form->bind($album);
    $form->get('submit')->setAttribute('value', 'Edit');

*form* ’un ``bind()`` metodu, modeli forma iliştirir. Bu iki şekilde kullanılmıştır.

# Form görüntülenirken, her form elemenanın başlangıç değerleri 
  modelden aktarılır.
# isValid() ile başarılı doğrulama yapıldıktan sonra, formdaki veri 
  modele geri konur.

Bu operasyonlar hydrator nesnesi kullanılarak yapılır. Birçok hydrator
vardır. Bunlardan bir tanesi ise, modelde  ``getArrayCopy()`` ve
``exchangeArray()`` metodları bekleyen ``Zend\Stdlib\Hydrator\ArraySerializable``
hydratorüdür. ``Album`` varlığında ``exchangeArray()`` metodunu zaten
yazmıştık. Şimdi sadece ``getArrayCopy()`` metodunu yazmamız gerekiyor.

.. code-block:: php

    // module/Album/src/Album/Model/Album.php:
    // ...
        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        // Add the following method:
        public function getArrayCopy()
        {
            return get_object_vars($this);
        }
    // ...

``$album``’e form verisini tekrar doldurmaya gerek yok, çünkü hydratorü ile 
``bind()`` kullanmanın bir sonucu olarak bu zaten yapılmıştır. Böylece sadece
``saveAlbum()`` metodunu kullanarak değişiklikleri veritabanına kaydederiz.

``edit.phtml`` görüntü şablonu albüm eklemede kullanılan şablona oldukça benzerdir:

.. code-block:: php

    <?php
    // module/Album/view/album/album/edit.phtml:

    $title = 'Edit album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <?php
    $form = $this->form;
    $form->setAttribute('action', $this->url(
        'album', 
        array(
            'action' => 'edit',
            'id'     => $this->id,
        )
    ));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

Değişiklikler sadece, başlık olarak ‘Edit Album’ ve form eylemi olarak ‘edit’ eylemi 
kullanmaktır.

Şimdi albümleri düzenleyebiliyor olmalısınız.

Albüm Silme
-----------

Uygulamamızı tamamlamak için silme işlemine ihtiyacımız var. Liste sayfamızda
her albüm kaydının yanında bir silme linki var. Link tıklandığında kaydı hemen
silme gibi safça bir yaklaşım yanlış olur. HTTP tanımlamalarını göz önüne alarak, 
GET kullanarak geri dönüşü olmayan bir eylem **yapmamanız** ve POST kullanmanız
gerektiğini hatırlatırız.

Kullanıcı delete’e tıkladığında bir onay formu göstermek zorundayız. “yes”’e
tıklayınca silme işlemini yapacağız. Burdaki form çok önemli olmadığı için
bunu doğrudan view da yapacağız. 

``AlbumController::deleteAction()`` eylem koduna bakalım:

.. code-block:: php

    // module/Album/src/Album/AlbumController.php:
    //...
        // Add content to the following method:
        public function deleteAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album');
            }

            $request = $this->getRequest();
            if ($request->isPost()) {
                $del = $request->getPost('del', 'No');

                if ($del == 'Yes') {
                    $id = (int) $request->getPost('id');
                    $this->getAlbumTable()->deleteAlbum($id);
                }

                // Redirect to list of albums
                return $this->redirect()->toRoute('album');
            }

            return array(
                'id'    => $id,
                'album' => $this->getAlbumTable()->getAlbum($id)
            );
        }
    //...

Daha önce yaptığımız gibi, eşleşen route’dan ``id``’yi alıyoruz ve onay 
sayfasını göstermek ya da silme işlemini belirlemek için istek nesnesinin 
``isPost()`` metodunu kontrol ediyoruz. ``deleteAlbum()`` ile kaydı silmek için 
album table nesnesini kullanıyoruz ve albüm listesine yönlendirme yapıyoruz.
İstek POST değil ise ``id`` ile ilgili veritabanı kaydını alarak görüntüye
aktarıyoruz.

Görüntü dosyamız basit bir formdan ibaret:

.. code-block:: php

    <?php
    // module/Album/view/album/album/delete.phtml:

    $title = 'Delete album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <p>Are you sure that you want to delete 
        '<?php echo $this->escapeHtml($album->title); ?>' by 
        '<?php echo $this->escapeHtml($album->artist); ?>'?
    </p>
    <?php 
    $url = $this->url('album', array(
        'action' => 'delete', 
        'id'     => $this->id,
    )); 
    ?>
    <form action="<?php echo $url; ?>" method="post">
    <div>
        <input type="hidden" name="id" value="<?php echo (int) $album->id; ?>" />
        <input type="submit" name="del" value="Yes" />
        <input type="submit" name="del" value="No" />
    </div>
    </form>

Bu dosyada, kullanıcıya bir onay mesajı ve "Yes" ve "No" butonlarından oluşan bir
form gösteriyoruz. Eylemde silme işlemini yapmak için özellikle "Yes" değerini
kontrol ediyoruz.

Ana sayfanın albüm listesini görüntülemesi
------------------------------------------

Son olarak; şu an ana sayfamız http://zf2-tutorial.localhost/ albüm listesini
görüntülemiyor.

Nedeni ``Application`` modülünün ``module.config.php`` dosyasındaki *route* 
yapılandırmasıdır. Değiştirmek için, ``module/Application/config/module.config.php``
dosyasını açın ve *route*’u bulun:

.. code-block:: php

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Application\Controller\Index',
                'action'     => 'index',
            ),
        ),
    ),

``controller`` dan ``Application\Controller\Index``’i 
``Album\Controller\Album`` olarak değiştir:

.. code-block:: php

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Album\Controller\Album', // <-- burayı değiştir
                'action'     => 'index',
            ),
        ),
    ),

İşte bu kadar. Şimdi tamammen çalışan bir uygulamamız var!
