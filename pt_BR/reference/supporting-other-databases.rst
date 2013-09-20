Suporte a outros Bancos de Dados
================================

Para suportar um banco de dados que não vem junto do Doctrine você tem que implementar 
as seguintes interfaces e classes abstratas:


-  ``\Doctrine\DBAL\Driver\Driver``
-  ``\Doctrine\DBAL\Driver\Statement``
-  ``\Doctrine\DBAL\Platforms\AbstractPlatform``
-  ``\Doctrine\DBAL\Schema\AbstractSchemaManager``

Para uma plataforma já suportada porém com driver não suportado você precisa implementar
somente as primeiras duas interfaces, uma vez que a SQL Generation e o Schema Management 
já são suportados pela respectiva plataforma e instâncias do esquema. Você também pode usar
vários testes unitários no pacote ``\Doctrine\Tests\DBAL`` para verificar se sua plataforma 
se comporta como todos os outros, que é necessário para o suporte do SchemaTool, a saber:


-  ``\Doctrine\Tests\DBAL\Platforms\AbstractPlatformTestCase``
-  ``\Doctrine\Tests\DBAL\Functional\Schema\AbstractSchemaManagerTestCase``

Ficaríamos muito felizes em receber contribuições para suporte de novos bancos de dados para 
tornar o Doctrine um produto ainda melhor.

Etapas de implementação em detalhe
-----------------------------------------------------

1. Adicione o atalho do seu driver inserindo o nome da sua classe em ``Doctrine\DBAL\DriverManager``.
2. Faça uma cópia dos tests/dbproperties.xml.dev e ajuste os valores para o atalho do seu driver e testdatabase.
3. Crie três novas classes implementando ``\Doctrine\DBAL\Driver\Driver``, ``\Doctrine\DBAL\Driver\Statement``
   e ``Doctrine\DBAL\Driver``. Você pode dar uma olhada no driver ``Doctrine\DBAL\Driver\OCI8``.
4. Você pode rodar a suíte de testes do seu novo driver digitando 
"cd tests/ && phpunit -c myconfig.xml Doctrine/Tess/AllTests.php"
5. Comece a implementar as classes AbstractPlatform e AbstractSchemaManager. 
Outras implementações podem servir como bons exemplos.
