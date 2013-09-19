SQL Query Builder
=================

O Doctrine 2.1 trabalha com um poderoso construtor de query para a linguagem SQL. Esse objeto QueryBuilder 
tem métodos para incluir partes para uma instrução SQL. Se você construiu o estado completo você pode executá-lo
usando a conexão que foi gerada. A API é mais ou menos a mesma que a do Query Builder DQL.

Você poder acessar o objeto QueryBuilder chamando ``Doctrine\DBAL\Connection#createQueryBuilder``:

.. code-block:: php

    <?php

    $conn = DriverManager::getConnection(array(/*..*/));
    $queryBuilder = $conn->createQueryBuilder();

