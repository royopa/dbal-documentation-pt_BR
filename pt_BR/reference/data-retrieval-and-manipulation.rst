Recuperação e Manipulação de Dados
==================================

A Doctrine DBAL segue bastante a API PDO. Se você trabalhou com PDO antes você vai 
conhecer a DBAL muito rapidamente. NO topo da API fornecida pela PDO hpa toneladas
de funções convenientes na DBAL.

Recuperação de Dados
--------------------

Usar um banco de dados implica em recuperar dados. É o caso de uso principal de um banco de dados.
Para esse propósito, cada vendor expõe uma API cliente que pode ser integrada dentro das linguagens
de programação. O PHP tem uma camada de abstração genérica para esse tipo de API chamada PDO (PHP Data Objects).
No entanto, por causa de divergências entre a comunidade PHP existem extensões nativas para 
cada banco de dados e tem muito mais manutenção (OCI8 por exemplo).

A API Doctrine DBAL é construída no topo da PDO e integra extensões nativas por envolvê-los na também na PDO. Se 
você já tiver uma conexão aberta através do método ``Doctrine\DBAL\DriverManager::getConnection()`` você pode 
começar usando essa API para recuperar dados facilmente.

Começe escrevendo uma consulta SQL e passe para o método ``query()`` da sua conexão:

.. code-block:: php

    <?php
    use Doctrine\DBAL\DriverManager;

    $conn = DriverManager::getConnection($params, $config);

    $sql = "SELECT * FROM articles";
    $stmt = $conn->query($sql); // Simples, mas com vários inconvenientes

O método query executa o SQL e retorna um objeto database statement. Um database statement pode ser iterado
para recuperar todas as linhas retornadas na consulta até que não haja mais linhas:

.. code-block:: php

    <?php

    while ($row = $stmt->fetch()) {
        echo $row['headline'];
    }

O método query é o mais simples para recuperação de dados, mas ele também tem vários inconvenientes:

-   Não há nenhuma maneira de adicionar parâmetros dinâmicos na consulta SQL sem alterar o ``$sql`` em si. 
    Isso pode permitir ataques de **SQL injection**, onde alguém pode modificar o SQL executado e até executar suas 
    próprias consultas explorando essa falha de segurança.
-   Fazer o **Quoting** de parâmetros dinâmicos para uma consulta SQL é uma tarefa tediosa e exige muito uso do 
    método ``Doctrine\DBAL\Connection#quote()``, o que torna a consulta SQL difícil de ler e entender.
-   Os bancos de dados otimizam as consultas SQL antes de elas serem executadas. Usando o método query você irá 
    acionar o processo de otimização toda vez, embora ele pudesse reutilizar essa informação facilmente usando uma
    técnica chamada **prepared statements**.

Esperamos que estes três argumentos e alguns detalhes mais técnicos tenham convencido você a investigar o uso de 
prepared statements para acessar sua base de dados.

Parâmetros Dinâmicos e Prepared Statements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Considere a consulta anterior, agora parametrizada para buscar apenas um único artigo pelo id.
Usando **ext/mysql** (ainda a primeira opção de acesso ao MySQL para muitos desenvolvedores) você teria que 
escapar cada valor passado na query usando ``mysql_real_escape_string()`` para evitar a SQL injection:

.. code-block:: php

    <?php
    $sql = "SELECT * FROM articles WHERE id = '" . mysql_real_escape_string($id, $link) . "'";
    $rs = mysql_query($sql);

Se você adicionar diversos parâmetros para uma query (por exemplo num UPDATE ou INSERT) essa abordagem pode 
tornar difícil a manutenção das consultas SQL. A razão é simples, a consulta SQL real não é claramente separada
dos parâmetros de entrada. Os prepared statements separam esses dois conceitos exigindo que o desenvolvedor 
adicione **placeholders** para a query SQL (prepare) que em seguida são substituídos por seus valores reais num
segundo passo (execute).

.. code-block:: php

    <?php
    // $conn instanceof Doctrine\DBAL\Connection

    $sql = "SELECT * FROM articles WHERE id = ?";
    $stmt = $conn->prepare($sql);
    $stmt->bindValue(1, $id);
    $stmt->execute();

Os placeholders nos prepared statements são simples pontos de interrogação (?) ou labels nomeados com dois pontos 
(:nome1). Você não pode misturar o posicionamento e a abordagem de labels. A abordagem usando pontos de interrogação
é chamada de posicional, pois os valores estão ligados em ordem da esquerda para a direito para qualquer ponto de 
interrogação encontrado na consulta SQL do prepared statement. É por isso que você deve especificar a posição da 
variável para ligar no método ``bindValue()``:

.. code-block:: php

    <?php
    // $conn instanceof Doctrine\DBAL\Connection

    $sql = "SELECT * FROM articles WHERE id = ? AND status = ?";
    $stmt = $conn->prepare($sql);
    $stmt->bindValue(1, $id);
    $stmt->bindValue(2, $status);
    $stmt->execute();

Os parâmetros nomeados tem a vantagem de que seus labels podem ser reutilizados e só precisam são necessários
uma vez:

.. code-block:: php

    <?php
    // $conn instanceof Doctrine\DBAL\Connection

    $sql = "SELECT * FROM users WHERE name = :name OR username = :name";
    $stmt = $conn->prepare($sql);
    $stmt->bindValue("name", $name);
    $stmt->execute();

A seção seguinte descreve a API da Doctrine DBAL no que diz respeito às prepared statements.

.. nota::

    O suporte para prepared statements posicionais e nomeados variam entre as diferentes extensões 
    de banco de dados. A PDO implementa seu próprio parser do lado cliente para que ambas as abordagens
    sejam viáveis para todos os drivers PDO. O OCI8/Oracle suporta somente parâmetros nomeados, mas o 
    Doctrine implementa um parser do lado cliente para também permitir parâmetros posicionais.

Usando Prepared Statements
~~~~~~~~~~~~~~~~~~~~~~~~~~

Existem três métodos na ``Doctrine\DBAL\Connection`` que permite que você utilize os prepared 
statements:

-   ``prepare($sql)`` - Cria um prepared statement do tipo ``Doctrine\DBAL\Statement``.
    É preferível usar esse método se você quiser reutilizar o statement para executar várias queries 
    com o mesmo SQL somente com parâmetros diferentes.
-   ``executeQuery($sql, $params, $types)`` - Cria um prepared statement para a query passada, vincula
    os parâmetros com seus tipos e executa a query.
    Esse método retorna o prepared statement executado para iteração e é útil para stements de SELECT.
-   ``executeUpdate($sql, $params, $types)`` - Cria um prepared statement para a query passada, vincula
    os parâmetros com seus tipos e executa a query.
    Esse método retorna a quantidade de linhas afetadas pela query executada e é útil para statements de 
    UPDATE, DELETE e INSERT.

Uma utilização simples do prepare foi mostrada na seção anterior, no entando é útil detalhar as
características do ``Doctrine\DBAL\Statement`` um pouco mais. Existem essencialmente dois tipos
diferentes de métodos disponíveis em um statement. São métodos para a ligação de parâmetros e tipos
e métodos para recuperar dados a partir de um statement.

-   ``bindValue($pos, $value, $type)`` - Faz a ligação de um determinado valor para o parâmetro posicional ou 
    nomeado no prepared statement.
-   ``bindParam($pos, &$param, $type)`` - Faz a ligação de uma determinada referência para o parâmetro posicional
    ou nomeado no prepared statement.

Se você terminou de fazer a ligação dos parâmetros você tem que chamar ``execute()`` no statement, que irá 
desencadear uma consulta no banco de dados. Após a conclusão da consulta você pode acessar os 
resultados da query usando a API fetch de um statement:

-   ``fetch($fetchStyle)`` - Recupera a próxima linha do statement ou falso se não existir pŕoxima linha.
    Move o cursor para frente de uma linha, de modo que chamadas consecutivas sempre retornará a próxima 
    linha.
-   ``fetchColumn($column)`` - Recupera apenas uma coluna da próxima linha especificada pelo índice da
    coluna.
    Move o cursor para frente de uma linha, de modo que chamadas consecutivas sempre retornará a próxima 
    linha.
-   ``fetchAll($fetchStyle)`` - Recupera todas as linhas de um statement.

É óbvio que a API fetch de um prepared statement funciona somente para ``SELECT``.

Se você achar tedioso ter escrever todo o código do prepared statement você pode alternativamente
usar os métodos ``Doctrine\DBAL\Connection#executeQuery()`` e ``Doctrine\DBAL\Connection#executeUpdate()``.
Veja a seção API seguinte para detalhes de como usá-los.

Adicionalmente existem vários métodos de conveniência para recuperação e manipulação em uma Connection, 
que serão descritas na seção API seguinte.

Fazendo a Ligação de Tipos
--------------------------

A Doctrine DBAL extende a manipulação de vinculação de tipos da PDO de forma considerável.
Além da conhecida constante ``\PDO::PARAM_*`` você pode fazer o uso de duas características 
adicionais muito poderosas.

A Conversão de tipos Doctrine\DBAL\Types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Se você não especificar um número inteiro (através de uma constante ``PDO::PARAM*``) para 
qualquer um dos métodos de ligação de parâmetros mas especificar uma string, a Doctrine DBAL
irá solicitar o tipo a camada de abstração para converter o valor passado de seu código PHP 
para uma representação de dados. Desta forma, você pode passar instâncias ``\DateTime`` para
um prepared statements e o Doctrine tem que convertê-los para o formato de base de dados 
adequado:

.. code-block:: php

    <?php
    $date = new \DateTime("2011-03-05 14:00:21");
    $stmt = $conn->prepare("SELECT * FROM articles WHERE publish_date > ?");
    $stmt->bindValue(1, $date, "datetime");
    $stmt->execute();

Se você der uma olhada em ``Doctrine\DBAL\Types\DateTimeType`` você verá que parte da 
conversão é delegada para um método na plataforma de banco de dados que está sendo 
usada, o que significa que esse código funciona independente do banco de dados que você está 
usando.

.. nota::

    Esteja ciente que esse tipo de conversão só funciona com ``Statement#bindValue()``,
    ``Connection#executeQuery()`` e ``Connection#executeUpdate()``. Ele não é suportado
    para passar um nome de tipo do doctrine para o ``Statement#bindParam()``, porque 
    isso não iria funcionar com a ligação por referência.

Lista de Parâmetros de Conversão
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. nota::

    Isso é uma característica do Doctrine 2.1.

Uma funcionalidade que falta no SQL é o suporte para lista de parâmetros. Você não
pode vincular um array de valores para um único parâmetro do prepared statement. Considere
o statement SQL muito comum:

.. code-block:: sql

    SELECT * FROM articles WHERE id IN (?)

Como você está usando uma expressão ``IN`` você com certeza gostaria de usá-la da seguinte maneira 
(e acho que todo mundo já tentou fazer isso uma vez na vida, antes de perceber que não funciona):

.. code-block:: php

    <?php
    $stmt = $conn->prepare('SELECT * FROM articles WHERE id IN (?)');
    // THIS WILL NOT WORK:
    $stmt->bindValue(1, array(1, 2, 3, 4, 5, 6));
    $stmt->execute();

Implementar uma forma genérica para lidar com esse tipo de query é uma tarefa chata.
É por isso que muitos desenvolvedores inserem os parâmetros diretamente na query, o que
pode abrir possibildiades de SQL injection se não for tratada com cuidado.

A Doctrine DBAL implementa um processo de análise muito poderoso que fará que tornará esse tipo de prepared statement possível nativamente no sistema de tipos 
de ligação.
O parsing necessariamente vem com uma sobrecarga de desempenho, mas somente se você realmente usar uma lista de parâmetros.
Existem dois tipos de tipos de ligação especiais que desvrevem uma lista de inteiros ou strings:

-   ``\Doctrine\DBAL\Connection::PARAM_INT_ARRAY``
-   ``\Doctrine\DBAL\Connection::PARAM_STR_ARRAY``

Usando uma destas constantes como um tipo você pode ativar o SQLParser dentro do Doctrine que reescreve o SQL e achata os valores especificados para o conjunto de parâmetros. Considere o exemplo anterior:

.. code-block:: php

    <?php
    $stmt = $conn->executeQuery('SELECT * FROM articles WHERE id IN (?)',
        array(array(1, 2, 3, 4, 5, 6)),
        array(\Doctrine\DBAL\Connection::PARAM_INT_ARRAY)
    );

O statement SQL passado para ``Connection#executeQuery`` não é o único realmente passado para o banco de dados. Ele é internamente reescrito para se parecer com o seguinte código explícito que poderia ser especificado, bem como:

.. code-block:: php

    <?php
    // Same SQL WITHOUT usage of Doctrine\DBAL\Connection::PARAM_INT_ARRAY
    $stmt = $conn->executeQuery('SELECT * FROM articles WHERE id IN (?, ?, ?, ?, ?, ?)',
        array(1, 2, 3, 4, 5, 6),
        array(\PDO::PARAM_INT, \PDO::PARAM_INT, \PDO::PARAM_INT, \PDO::PARAM_INT, \PDO::PARAM_INT, \PDO::PARAM_INT)
    );

Isso é muito mais complicado e é feio escrever genericamente.

.. nota::

    O parâmetro list support somente trabalhad com ``Doctrine\DBAL\Connection::executeQuery()`` e ``Doctrine\DBAL\Connection::executeUpdate()``, NÃO com os métodos de binding de um prepared statement.

API
---

O DBAL contém vários métodos para executar queries contra o banco de dados configurado para recuperação e manipulação de dados. Abaixo vamos apresentar os métodos e fornecer alguns exemplos para cada um deles.

prepare()
~~~~~~~~~

Prepara o dado statement SQL e retorna a instância ``\Doctrine\DBAL\Driver\Statement``:

.. code-block:: php

    <?php
    $statement = $conn->prepare('SELECT * FROM user');
    $statement->execute();
    $users = $statement->fetchAll();
    
    /*
    array(
      0 => array(
        'username' => 'jwage',
        'password' => 'changeme
      )
    )
    */

executeUpdate()
~~~~~~~~~~~~~~~

Executa o prepared statement com o dado SQL e os parâmetros e retorna a quantidade de linhas afetadas:

.. code-block:: php

    <?php
    $count = $conn->executeUpdate('UPDATE user SET username = ? WHERE id = ?', array('jwage', 1));
    echo $count; // 1

A variável ``$types`` contém a constante PDO ou Doctrine Type para realizar a conversão de tipos necessária entre a entrada de valores real e os valores esperados do banco de dados. Veja a seção `Types <./types#type-conversion>`_ para mais informações.

executeQuery()
~~~~~~~~~~~~~~

Cria um prepared statement para a dada SQL e passa os parâmetros para o método execute, retornando em seguida o statement:

.. code-block:: php

    <?php
    $statement = $conn->executeQuery('SELECT * FROM user WHERE username = ?', array('jwage'));
    $user = $statement->fetch();
    
    /*
    array(
      0 => 'jwage',
      1 => 'changeme
    )
    */

A variável ``$types`` contém a constante PDO ou Doctrine Type para realizar a conversão de tipos necessária entre a entrada de valores real e os valores esperados do banco de dados. Veja a seção `Types <./types#type-conversion>`_ para mais informações.

fetchAll()
~~~~~~~~~~

Executa a query e armazena todos os resultados em um array:

.. code-block:: php

    <?php
    $users = $conn->fetchAll('SELECT * FROM user');
    
    /*
    array(
      0 => array(
        'username' => 'jwage',
        'password' => 'changeme
      )
    )
    */

fetchArray()
~~~~~~~~~~~~

Recuperação númerica dos índices da primeira linha do resultado da query dada:

.. code-block:: php

    <?php
    $user = $conn->fetchArray('SELECT * FROM user WHERE username = ?', array('jwage'));
    
    /*
    array(
      0 => 'jwage',
      1 => 'changeme
    )
    */

fetchColumn()
~~~~~~~~~~~~~

Recupera apenas a coluna dada da primeira linha do resultado.

.. code-block:: php

    <?php
    $username = $conn->fetchColumn('SELECT username FROM user WHERE id = ?', array(1), 0);
    echo $username; // jwage

fetchAssoc()
~~~~~~~~~~~~

Recupera um array da primeira linha do resultado.

.. code-block:: php

    <?php
    $user = $conn->fetchAssoc('SELECT * FROM user WHERE username = ?', array('jwage'));
    /*
    array(
      'username' => 'jwage',
      'password' => 'changeme
    )
    */

Existem também métodos de conveniência para queries de manipulação de dados:

delete()
~~~~~~~~~

Exclui todas as linhas da tabela que corresponderem com o dado identificador, onde as chaves são os nomes da coluna.

.. code-block:: php

    <?php
    $conn->delete('user', array('id' => 1));
    // DELETE FROM user WHERE id = ? (1)

insert()
~~~~~~~~~

Insere uma linha na tabela dada usando os pares de dados chave-valor.

.. code-block:: php

    <?php
    $conn->insert('user', array('username' => 'jwage'));
    // INSERT INTO user (username) VALUES (?) (jwage)

update()
~~~~~~~~~

Atualiza todas as linhas da tabela que corresponderem com o identificador com os dados passados.

.. code-block:: php

    <?php
    $conn->update('user', array('username' => 'jwage'), array('id' => 1));
    // UPDATE user (username) VALUES (?) WHERE id = ? (jwage, 1)

Por padrão, o Doctrine DBAL não faz escaping. Escaping é um négócio muito complicado para fazer automaticamente, portanto, não existe nenhum por padrão. A ORM internamente faz escape de todos os seus valores, porque ele tem lotes de metadados disponíveis sobre o contexto atual. Quando você usa a Doctrine DBAL sozinha, você tem que cuidar disso você mesmo. Os seguintes métodos podem te ajudar com isso:


quote()
~~~~~~~~~

Faz quote de um valor:

.. code-block:: php

    <?php
    $quoted = $conn->quote('value');
    $quoted = $conn->quote('1234', \PDO::PARAM_INT);

quoteIdentifier()
~~~~~~~~~~~~~~~~~

Faz quote de um identificador de acordo com os detalhes da plataforma.

.. code-block:: php

    <?php
    $quoted = $conn->quoteIdentifier('id');

