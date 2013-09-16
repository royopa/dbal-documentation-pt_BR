Arquitetura
===========

Como já foi dito, a DBAL é uma fina camada no topo da PDO. A PDO é 
definida principalmente em duas classes ``PDO`` e ``PDOStatement``.
As classes equivalentes na DBAL são ``Doctrine\DBAL\Connection`` e
``Doctrine\DBAL\Statement``. Uma ``Doctrine\DBAL\Connection`` empacota 
uma ``Doctrine\DBAL\Driver\Connection`` e uma classe ``Doctrine\DBAL\Statement`` 
epacota uma ``Doctrine\DBAL\Driver\Statement``.

``Doctrine\DBAL\Driver\Connection`` e
``Doctrine\DBAL\Driver\Statement`` são apenas interfaces. Essas interfaces
são implementadas por drivers concretos. Para todo driver baseado em PDO,
``PDO`` e ``PDOStatement`` são implementações dessas interfaces. Assim, 
para os drivers baseados em PDO, uma ``Doctrine\DBAL\Connection`` 
empacota uma instância ``PDO`` e uma ``Doctrine\DBAL\Statement`` empacota
uma instância ``PDOStatement``. E mais, uma ``Doctrine\DBAL\Connection`` 
*é uma* ``Doctrine\DBAL\Driver\Connection`` e um ``Doctrine\DBAL\Statement`` 
*é um* ``Doctrine\DBAL\Driver\Statement``.

Quais implementações adicionais o ``Doctrine\DBAL\Connection`` ou um 
``Doctrine\DBAL\Statement`` implementam ao driver? As melhorias
incluem o log do SQL, eventos e controle sobre o nível de isolamento
da transação de forma portátil, entre outros.

Um driver DBAL é definido para o exterior em 3 interfaces:
``Doctrine\DBAL\Driver``, ``Doctrine\DBAL\Driver\Connection`` e 
``Doctrine\DBAL\Driver\Statement``. O dois últimos se assemelham aos
seus correspondentes na PDO.

Uma implementação concreta do driver deve fornecer a implementação 
de classes para essas 3 interfaces.

A DBAL é separada em vários pacotes que tem responsabilidades bem separadas
das diferentes camadas do RDBMS.

Drivers
-------

Os drivers abstraem a API especifíca do banco de dados através de 
duas interfaces:

-  ``\Doctrine\DBAL\Driver\Driver``
-  ``\Doctrine\DBAL\Driver\Statement``

As duas interfaces acima precisam exatamente dos mesmos metódos da PDO.

Plataformas
-----------

As plataformas abstraem a geração de queries e as características que
cada banco de dados suporta. A ``\Doctrine\DBAL\Platforms\AbstractPlatform`` 
define um denominador comum de qual plataforma de banco de dados tem de 
mostrada no nível de usuário para ser suportada pelo Doctrine. Isso
inclui a SchemaTool, isolamento de transação e muitos outros. A
plataforma para o MySQL por exemplo pode ser usada por todas as 3 extensões
MySQL, PDO, Mysqli e ext/mysql.

Logging
-------

O logging contém a interface e algumas implementações para debug da execução
da query SQL Doctrine durante uma requisição.

Esquema
-------

O esquema oferece uma API para cada banco de dados para executar instruções 
DDL na sua plataforma ou recuperar metadados sobre isso. Ele também mantém 
a camada de abstração do schema que é usado pelo gerenciador de esquema 
do Doctrine DBAL e ORM


Types
-----

Os types oferecem uma camada de abstração para a conversão e geração de tipos
entre as base de dados e o PHP. O Doctrine vem com alguns tipos comuns, mas 
oferece a possibilidade de definir tipos personalizados ou ampliar os já 
existentes de forma fácil.
