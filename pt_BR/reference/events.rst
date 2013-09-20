Eventos
=======

O ``Doctrine\DBAL\DriverManager`` e ``Doctrine\DBAL\Connection`` aceitam uma instância do
``Doctrine\Common\EventManager``. O EventManager tem um par de eventos dentro da 
camada DBAL que são acionados para o usuário os ouça.

Evento PostConnect
------------------

O ``Doctrine\DBAL\Events::postConnect`` é acionado logo que a conexão ao banco de dados
for estabelecida. Ele permite especificar qualquer opção de conexão relevante e dá acesso à
instância ``Doctrine\DBAL\Connection`` que é responsável pelo gerenciamento da conexão através
de uma instância de argumentos de evento ``Doctrine\DBAL\Event\ConnectionEventArgs``.


O Doctrine inclui uma implementação do evento "PostConnect":

-  ``Doctrine\DBAL\Event\Listeners\OracleSessionInit`` permite especificar qualquer quantidade 
   de variáveis de ambiente de sessões Oracle logo após a conexão ser estabelecida.
   

Você pode registrar eventos assinando-os à instância ``EventManager`` passada na factory Connection:

   
.. code-block:: php

    <?php
    $evm = new EventManager();
    $evm->addEventSubscriber(new OracleSessionInit(array(
        'NLS_TIME_FORMAT' => 'HH24:MI:SS',
    )));
    
    $conn = DriverManager::getConnection($connectionParams, null, $evm);


