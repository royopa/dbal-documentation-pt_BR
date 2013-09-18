Caching
=======

Um ``Doctrine\DBAL\Statement`` pode fazer o cache dos result sets automaticamente.

Para que isso funcione uma instância do ``Doctrine\Common\Cache\Cache`` é necessária.
Isso pode ser setado no objeto de configuração (pode ser passado também na execução da query 
opcionalmente):

::

    <?php
    $cache = new \Doctrine\Common\Cache\ArrayCache();
    $config = $conn->getConfiguration();
    $config->setResultCacheImpl($cache);

Para pegar o result set de uma query que está em cache é necessário passar uma instância 
``Doctrine\DBAL\Cache\QueryCacheProfile`` para a instância ``executeQuery`` ou ``executeCacheQuery``.
A diferença entre esses dos métodos é que o primeiro não precisa da instância enquanto o último
precisa da instância como um parâmetro obrigatório:

::

    <?php
    $stmt = $conn->executeQuery($query, $params, $types, new QueryCacheProfile(0, "some key"));
    $stmt = $conn->executeCacheQuery($query, $params, $types, new QueryCacheProfile(0, "some key"));

Também é possível passar em uma instância ``Doctrine\Common\Cache\Cache`` no construtor da 
``Doctrine\DBAL\Cache\QueryCacheProfile``. Nesse caso ele substitui a instância padrão de cache:

::

    <?php
    $cache = new \Doctrine\Common\Cache\FilesystemCache(__DIR__);
    new QueryCacheProfile(0, "some key", $cache);

Para que todos os dados sejam realmente armazenados em cache é necessário garantir que o result set
seja lido por inteiro (uma forma fácil de garantir isso é usar ``fetchAll``) e o objeto statement
seja fechado:

::

    <?php
    $stmt = $conn->executeCacheQuery($query, $params, $types, new QueryCacheProfile(0, "some key"));
    $data = $stmt->fetchAll();
    $stmt->closeCursor(); // at this point the result is cached


.. aviso::

    Ao usar uma camada de cache nem todos os modos de fetch são suportados. Veja o código do 
    `ResultCacheStatement <https://github.com/doctrine/dbal/blob/master/lib/Doctrine/DBAL/Cache/ResultCacheStatement.php#L156>`_ para detalhes.
