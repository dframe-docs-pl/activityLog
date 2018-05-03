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
 
Pierwszy parametr odpowiada za połączenie w przykładzie z Mysql który powinnyśmy umieścić w folderze Modelu kod znajdziecie tutaj https://github.com/dframe/activityLog/blob/master/example/app/Model/ActivityLog/Drivers/Log.php
Jako $loggedId użyjemy id użytkownika 


Proste Logi
====

Najprostszy log jest akcji, nie przyjmuje on żadnych parametrów, jedynie informacje w poniższym przykładnie ze użytkownik odświeżył stronę. 

Najpierw potrzebujemy interpreter który umieszczamy w pliku **app/Libs/Extensions/ActivityLog/Action.php**

.. code-block:: php

 namespace Libs\Extensions\ActivityLog;

 class Action
 {

     public function __construct()
     {
         return $this;
     }

 }


Następnie już możemy zapisać pierwszy log.
 
.. code-block:: php

 $this->activity->log('Refresh Page')->entity('\Libs\Extensions\ActivityLog\Action'))->push();

Dodatkowe parametry
====

Teraz spróbujmy dodać log z jakimiś parametrami. Jednak by rozpocząc musimy mieć odpowiedzi Interpterer.

.. code-block:: php

 namespace Libs\Extensions\ActivityLog;

 class Change
 {

     public function interpreter($key)
     {
         $this->interpreter = array(
             'users' => array('id', 'firstname', 'lastname')
         );
 
         return $this->interpreter[$key];
     }

     public function build($before, $after)
     {

         if (!empty(array_diff_key($before, $after))) {
             throw new \Exception("Keys in array MUST be same", 1);
         }
 
         foreach ($after as $key => $value) {
             if ($before[$key] == $value) {
                 unset($before[$key]);
                 unset($after[$key]);
             }
         }
         
         $this->changes = array('before' => $before, 'after' => $after);
         return $this;
     }

 }

Powyższy interpterer pozwala nam na logowanie 3 parametrów id, firstname oraz lastname. Jest to ważne głównie dla odczytu jeśli chcemy logować więcej informacji poprostu dopisujemy kolejne parametry.
 
.. code-block:: php
 
 $before = array(
     'firstname' => 'Before Change'
 );
  
 $after = array(
     'firstname' => 'After Change'
 );
  
 $dataId = '1';
 $this->activity->log('Update Data')->entity('\Libs\Extensions\ActivityLog\Change', array($before, $after))->on('data.id', $dataId)->push();

