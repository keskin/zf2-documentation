.. EN-Revision: none
.. _user-guide.skeleton-application:

###########################
Başlarken: İskelet Uygulama
###########################

Uygulamamızı oluştururken `github <https://github.com/>`_ 
ta bulunan `ZendSkeletonApplication <https://github.com/zendframework/ZendSkeletonApplication>`_ 
ile başlayacağız. https://github.com/zendframework/ZendSkeletonApplication adresine
gidip “Zip” butonuna tıklayarak ``zendframework-ZendSkeletonApplication-zfrelease-2.0.0beta5-2-gc2c7315.zip``
benzer bir isimdeki dosyayı indiriniz.

Bu dosyayı sanal hostlarınızı tuttuğunuz dizine açıp ismini ``zf2-tutorial`` 
olarak değiştiriniz.

ZendSkeletonApplication bağımlılıkları çözümlemek için Composer (http://getcomposer.org)
kullanmak üzere ayarlanmıştır. Dersimizde sözkonusu bağımlılık Zend Framework 2’dir.

Zend Framework 2’yi kurmak için ``zf2-tutorial`` dizininden basitçe şu komutları yazıyoruz:

.. code-block:: bash

    php composer.phar self-update
    php composer.phar install

Bu biraz zaman alabilir. Şöyle bir çıktı görmelisiniz:

.. code-block:: bash

    Installing dependencies from lock file
    - Installing zendframework/zendframework (dev-master)
      Cloning 18c8e223f070deb07c17543ed938b54542aa0ed8

    Generating autoload files

.. note::

    Eğer şu mesajı görürseniz: 
    
    .. code-block:: bash

        [RuntimeException]      
          The process timed out. 

    bağlantınız tüm paketi belli bir zamanda indirmek için yavaş ve Composer zaman
    aşımına uğramıştır. Bunu önlemek için:

    .. code-block:: bash

        php composer.phar install

    yerine:

    .. code-block:: bash

        COMPOSER_PROCESS_TIMEOUT=5000 php composer.phar install

    çalıştırıyoruz.

Şimdi sanal host yapılandırmasına geçebiliriz.

Sanal host
----------

Şimdi, uygulama için Apache sanal host oluşturmanız gerekiyor. hosts dosyanızı 
http://zf2-tutorial.localhost adresinin ``zf2-tutorial/public`` dizinindeki
``index.php`` üzerinden hizmet verecek şekilde düzenleyiniz.

Sanal host yapılandırması genelde ``httpd.conf`` veya ``extra/httpd-vhosts.conf``
içinden yapılır. (Eğer ``httpd-vhosts.conf`` dosyasını kullanıyorsanız, bu dosyanın
``httpd.conf`` dosyasından çağrıldığından emin olunuz.)

``NameVirtualHost`` direktifinin “\*:80” veya benzeri şekilde tanımlandığına emin olunuz
ve aşağıdaki gibi bir sanal host tanımlayınız:

.. code-block:: apache

    <VirtualHost *:80>
        ServerName zf2-tutorial.localhost
        DocumentRoot /path/to/zf2-tutorial/public
        SetEnv APPLICATION_ENV "development"
        <Directory /path/to/zf2-tutorial/public>
            DirectoryIndex index.php
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>
    </VirtualHost>

``zf2-tutorial.localhost`` adresinin ``127.0.0.1`` i gösterecek şekilde ``/etc/hosts`` 
veya ``c:\windows\system32\drivers\etc\hosts`` dosyanızı güncellediğinizden emin olunuz.
Bu durumda web sitesi http://zf2-tutorial.localhost adresinden erişilebilir.

.. code-block:: txt

    127.0.0.1               zf2-tutorial.localhost localhost

Eğer yapılandırmayı doğru şekilde yaparsanız şöyle birşey görmelisiniz:

.. image:: ../images/user-guide.skeleton-application.hello-world.png
    :width: 940 px

``.htaccess`` dosyasının çalıştığını görmek için http://zf2-tutorial.localhost/1234 
sayfasına gidiniz. Şöyle bir sayfa görmelisiniz:

.. image:: ../images/user-guide.skeleton-application.404.png
    :width: 940 px

Eğer standart Apache 404 hatası görürseniz, devam etmeden önce ``.htaccess``
kullanım hatasını gidermelisiniz.

Şimdi çalışan bir iskelet uygulamamız var ve uygulamamıza özellikler eklemeye başlayabiliriz.
