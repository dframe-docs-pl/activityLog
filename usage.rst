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


Logi z parametrami
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

