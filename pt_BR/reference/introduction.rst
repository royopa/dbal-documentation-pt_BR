Introdução
==========

A Camada de Abstração de Banco de Dados Doctrine (DBAL) oferece uma
camada leve através de uma API parecida com a PDO e muitos outros 
recursos adicionais, como a introspecção do esquema do banco de dados
e a manipulação através de uma API orientada a objetos.

A DBAL abstrai a PDO através do uso de interfaces que se 
parecem com a PDO existente, tornando possível implementar drivers 
personalizados que podem utilizar APIs nativas ou não. Por exemplo, 
a DBAL trabalha com um driver para banco de dados Oracle que usa a
extensão oci8 internamente.

A camada de banco de dados do Doctrine 2 pode ser usada independente
do mapper objeto-relacional. Para utilizar a DBAL tudo que você precisa
são os namespaces ``Doctrine\Common`` and ``Doctrine\DBAL``. Após ter 
os namespaces Common e DBAL você deve configurar um class loader para
fazer o autoload das classes:

.. code-block:: php

    <?php
    use Doctrine\Common\ClassLoader;
    
    require '/path/to/doctrine/lib/Doctrine/Common/ClassLoader.php';
    
    $classLoader = new ClassLoader('Doctrine', '/path/to/doctrine');
    $classLoader->register();

Agora você já consegue carregar as classes que estão na pasta 
``/path/to/doctrine`` como ``/path/to/doctrine/Doctrine/DBAL/DriverManager.php``
que usaremos mais tarde para configurar nossa primeira conexão com a camada DBAL.


