.. EN-Revision: d4e4bc6
.. _user-guide.styling-and-translations:

###################
Biçimleme ve Çeviri
###################

SkeletonApplication'ın biçimlemesini olduğu gibi almıştık, fakat sayfa başlığını
ve telif hakkı mesajını değiştirmemiz gerekiyor.

ZendSkeletonApplication tüm metinler için ``Zend\I18n``'in çeviri işlevini kullanmak
için yapılandırılmıştır. Bileşen ``Application/language`` dizinindeki ``.po`` dosyalarını
kullanır. Metinleri değiştirmek için `poedit <http://www.poedit.net/download.php/>`_
'e ihtiyacınız olacak. ``Original`` yazı listesindeki “Skeleton Application” 'a 
tıklayıp çevirisine “Tutorial” yazın.

.. image:: ../images/user-guide.styling-and-translations.poedit.png

Araç çubuğundaki Save butonuna tıklayınca poedit bizim için ``en_US.mo`` dosyası 
oluşturacak. Eğer ``.mo`` dosyasını oluşturulmamışsa, ``Preferences -> Editor -> Behavior``
daki ``Automatically compile .mo file on save`` kutusunun işaretleyin.

Telif hakkı mesajını değiştirmek için, ``Application`` modülünün ``layout.phtml``
view dosyasını değiştirmeliyiz.

.. code-block:: php

    // module/Application/view/layout/layout.phtml:
    // Remove this line:
    <p>&copy; 2005 - 2012 by Zend Technologies Ltd. <?php echo $this->translate('All 
    rights reserved.') ?></p>

Sayfa şimdi her zamankinden daha iyi görünüyor.

.. image:: ../images/user-guide.styling-and-translations.translated-image.png
    :width: 940 px
