Configuração
============

Fazendo uma conexão
-------------------

Você pode fazer uma conexão DBAL através da classe ``Doctrine\DBAL\DriverManager``.

.. code-block:: php

    <?php
    $config = new \Doctrine\DBAL\Configuration();
    //..
    $connectionParams = array(
        'dbname' => 'mydb',
        'user' => 'user',
        'password' => 'secret',
        'host' => 'localhost',
        'driver' => 'pdo_mysql',
    );
    $conn = \Doctrine\DBAL\DriverManager::getConnection($connectionParams, $config);

O ``DriverManager`` retorna uma instância da ``Doctrine\DBAL\Connection`` 
que empacota a conexão do driver (que muitas vezes é uma instância PSO).

As seções seguintes descrevem os parâmetros de conexão disponíveis detalhadamente.

Driver
~~~~~~

O driver especifica a real implementação das interfaces DBAL para 
utilização. Ele pode ser configurado de uma das três formas:

-  ``driver``: A implementação interna do driver que será utilizado. Os seguintes
driver estão disponíveis atualmente:

   -  ``pdo_mysql``: Um driver MySQL que usa a PDO pdo\_mysql
      extension.
   -  ``pdo_sqlite``: Um driver SQLite que usa a PDO pdo\_sqlite
      extension.
   -  ``pdo_pgsql``: Um driver PostgreSQL que usa a PDO pdo\_pgsql
      extension.
   -  ``pdo_oci``: Um driver Oracle que usa a PDO pdo\_oci
      extension.
      **Fique atento que esse driver causou problemas em nossos teste. Prefira usar
      o driver oci8 se possível.**
   -  ``pdo_sqlsrv``: Um driver Microsoft SQL Server que usa a PDO pdo\_sqlsrv
   -  ``oci8``: Um driver Oracle que usa a extensão PHP oci8.

-  ``driverClass``: Especifica uma implementação de driver personalizada se não 
tiver nenhum "driver" especificado. Isso permite o uso de drivers personalizados 
que não fazem parte da própria DBAL.
-  ``pdo``: Especifica uma instância PDO existente para utilização.

Classe Wrapper
~~~~~~~~~~~~~~

A classe ``Doctrine\DBAL\Connection`` empacota por padrão um driver
``Connection``. A opção ``wrapperClass`` permite especificar uma 
implementação de wrapper personalizada, no entanto, uma wrapper
personalizada deve ser uma subclasse da ``Doctrine\DBAL\Connection``.

Detalhes de Conexão
~~~~~~~~~~~~~~~~~~~

Os detalhes da conexão identificam o banco de dados e as credenciais 
usadas na conexão. Os detalhes da conexão pode ser diferentes
dependendo do driver utilizado. As seções seguintes descrevem as
opções reconhecidas por cada driver.

.. nota::

    Não é necessário especificar detalhes de conexão ao usar uma instância 
    PDO através da opção ``pdo``.

pdo\_sqlite
^^^^^^^^^^^


-  ``user`` (string): Usuário usado para conectar no banco de dados.
-  ``password`` (string): Senha usada para conectar no banco de dados.
-  ``path`` (string): O caminho do arquivo da base de dados.
   Mutualmente exclusiva com ``memory``. O ``path`` tem precedência.
-  ``memory`` (boolean): True se a base de dados SQLite deve ser in-memory (não-persistente). 
Mutualmente exclusiva com o ``path``.
   ``path`` tem precedência.

pdo\_mysql
^^^^^^^^^^


-  ``user`` (string): Usuário usado para conectar no banco de dados.
-  ``password`` (string): Senha usada para conectar no banco de dados.
-  ``host`` (string): O hostname do servidor de banco de dados.
-  ``port`` (integer): A porta usada para conectar no banco de dados.
-  ``dbname`` (string): O nome da base de dados/schema para conectar no banco de dados.
-  ``unix_socket`` (string): O nome do socket usado para conectar no banco de dados.
-  ``charset`` (string): O charset usado para conectar na base de dados.

pdo\_pgsql
^^^^^^^^^^


-  ``user`` (string): Usuário usado para conectar no banco de dados.
-  ``password`` (string): Senha usada para conectar no banco de dados.
-  ``host`` (string): O hostname do servidor de banco de dados.
-  ``port`` (integer): A porta usada para conectar no banco de dados.
-  ``dbname`` (string): O nome da base de dados/schema para conectar no banco de dados.

O PostgreSQL se comporta diferente com valores booleanos quando você usa ou não 
``PDO::ATTR_EMULATE_PREPARES``. Para alterar o uso de ``'true'`` e ``'false'`` 
como strings você pode mudar para inteiros usando: 
``$conn->getDatabasePlatform()->setUseBooleanTrueFalseStrings($flag)``.

pdo\_oci / oci8
^^^^^^^^^^^^^^^


-  ``user`` (string): Usuário usado para conectar no banco de dados.
-  ``password`` (string): Senha usada para conectar no banco de dados.
-  ``host`` (string): O hostname do servidor de banco de dados.
-  ``port`` (integer): A porta usada para conectar no banco de dados.
-  ``dbname`` (string): O nome da base de dados/schema para conectar no banco de dados.
-  ``charset`` (string): O charset usado para conectar na base de dados.

pdo\_sqlsrv
^^^^^^^^^^


-  ``user`` (string): Usuário usado para conectar no banco de dados.
-  ``password`` (string): Senha usada para conectar no banco de dados.
-  ``host`` (string): Hostname of the database to connect to.
-  ``port`` (integer): A porta usada para conectar no banco de dados.
-  ``dbname`` (string): O nome da base de dados/schema para conectar no banco de dados.

Plataforma Personalizada
~~~~~~~~~~~~~~~~~~~~~~~~

Cada driver usa uma implementação padrão do ``Doctrine\DBAL\Platforms\AbstractPlatform``. 
Se quiser usar uma implementação customizada, você pode passar uma instância pré-criada
na opção ``platform``.

Opções de Driver Personalizadas
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A opção ``driverOptions`` permite passar opções arbitrárias para o driver. 
Isso é equivalente ao quarto argumento do `construtor PDO <http://php.net/manual/en/pdo.construct.php>`_.
