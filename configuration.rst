Zapisywanie Logów
^^^^^^^^^^

Logi aktywności w systemie mogą być dowolnie użyte. Dane jakie mogą zapisywać ogranicza się tylko do wyobraźni. Dzięki możliwości typowania logów można do w łatwy sposób napisać interpreter który będzie je odczytywał w określony przez nas sposób. 
Poniżej znajduje się przykłąd użycia.

W miejscu którym mają być używane należy zdefiniować. W naszym przypadku zdefinujemy w głownym abstrakcyjnym kontrolerze.

.. code-block:: php

 namespace Controller;

 use Dframe\ActivityLog\Activity;

 abstract class Controller extends \Dframe\Controller
 {
     public function start()
     {   

         /** 
          * @param Object $driver
          * @param Int $loggedId
          */
         $this->activity = new Activity($this->loadModel('ActivityLog/Drivers/Log'), $this->baseClass->session->get('id', 0));
     }
 
Pierwszy parametr odpowiada za połączenie w przykładzie z Mysql który powinnyśmy umieścić w folderze naszego projektu. Kod znajdziecie tutaj https://github.com/dframe/activityLog/tree/master/demo/app/Drivers
Jako $loggedId użyjemy np id użytkownika 
