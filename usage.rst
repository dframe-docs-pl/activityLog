.. title:: ActivityLog - Prosty system do obsługi logów z systemem templatek

.. meta::
    :description: Najprostszy log jest akcji, nie przyjmuje on żadnych parametrów, jedynie informacje w poniższym przykładnie ze użytkownik odświeżył stronę. 
    :keywords: dframe, log, psr3, logger, log system

Proste Logi
====
Prosty system do obsługi logów z systemem templatek. Najprostszy log jest akcji, nie przyjmuje on żadnych parametrów, jedynie informacje w poniższym przykładnie ze użytkownik odświeżył stronę. 


Przykład użycia
====
.. code-block:: php

 use Dframe\ActivityLog\Activity;
 use Dframe\ActivityLog\Demo\Drivers\FileLog;
 
 require_once __DIR__ . '/../../vendor/autoload.php';

 $log = (new Activity(new FileLog()));
 $log->log('Hello Word!')->entity(\Dframe\ActivityLog\Demo\Entity\Action::class)->push();


PSR-3 Adapter
====

.. code-block:: php

 use Dframe\ActivityLog\Activity;
 use Dframe\ActivityLog\Demo\Drivers\PSR3FileLog;
 use Dframe\ActivityLog\Helper\Psr3Adapter;
 use Psr\Log\LogLevel;

 require_once __DIR__ . '/../../vendor/autoload.php'; 

 $log = new Activity(new PSR3FileLog());

 $logger = new Psr3Adapter($log, 'System', \Dframe\ActivityLog\Entity\PSR3::class);
 $logger->log(LogLevel::ERROR, 'This is {error}', ['error' => 'error #500']);


Najpierw potrzebujemy interpreter który umieszczamy w pliku **app/Libs/Extensions/ActivityLog/Action.php**

.. code-block:: php

 namespace Libs\Extensions\ActivityLog;

 class Action
 {

     /**
      * Action constructor.
      */
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
     /**
      * @var array
      */
     public $interpreter;
 
     /**
      * @var array
      */
     public $changes;
 
     /**
      * @param $key
      *
      * @return mixed
      */
     public function interpreter($key)
     {
         $this->interpreter = [
             'users' => ['id', 'first_name', 'last_name']
         ];
 
         return $this->interpreter[$key];
     }
 
     /**
      * @param $before
      * @param $after
      *
      * @return $this
      * @throws \Exception
      */
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
 
         $this->changes = ['before' => $before, 'after' => $after];
         return $this;
     }
 
 }


Powyższy interpterer pozwala nam na logowanie 3 parametrów id, firstname oraz lastname. Jest to ważne głównie dla odczytu jeśli chcemy logować więcej informacji poprostu dopisujemy kolejne parametry.

.. code-block:: php
 
 $before = [
     'first_name' => 'Before Change'
 ];
 
 $after = [
     'first_name' => 'After Change'
 ];
 
 $dataId = '1';
 $this->activity->log('Update Data')->entity('\Libs\Extensions\ActivityLog\Change', [$before, $after])->on('data.id',
    $dataId)->push();


