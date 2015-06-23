Конфигурация
=============

Первое, что нужно знать об Idiorm - *вам не нужно объявлять какие-либо классы модели для её использования*. С почти любой другой ORM, первым шагом является установка моделей и их связка с таблицами из базы данных
(через переменные конфигурации, XML файлы и тому подобное). С Idiorm,
вы можете приступить к использованию ORM сразу же.

Установка
~~~~~

Первым делом, укажите файл исходного кода Idiorm в ``require``:

.. code-block:: php

    <?php
    require_once 'idiorm.php';

Затем передайте *Data Source Name* строку соединения в метод ``configure``
класса ORM. Он используется PDO для соединения с вашей базой данных. Для более детальной информации, смотри `документацию PDO`_.

.. code-block:: php

    <?php
    ORM::configure('sqlite:./example.db');

Вы так же можете передать имя пользователя и пароль в драйвер базы данных, используя параметры конфигурации ``username`` и ``password``.
Например, если вы используете MySQL:

.. code-block:: php

    <?php
    ORM::configure('mysql:host=localhost;dbname=my_database');
    ORM::configure('username', 'database_user');
    ORM::configure('password', 'top_secret');

Смотрите так же секцию “Конфигурация” ниже.

Конфигурация
~~~~~~~~~~~~~

Помимо передачи DSN строки для подключения к базе данных (смотри выше), метод ``configure`` можно использовать и для установки некоторых других простых настроек ORM класса. Изменение настроек подразумевает передачу в метод ``configure`` пар ключ/значение, представляющих название параметра, который вы хотите поменять, и значение, которое хотите ему установить.

.. code-block:: php

    <?php
    ORM::configure('название_параметра', 'значение_для_параметра');

Следующий метод используется для передачи множества пар ключ/значение за раз.

.. code-block:: php

    <?php
    ORM::configure(array(
        'название_параметра_1' => 'значение_для_параметра_1', 
        'название_параметра_2' => 'значение_для_параметра_2', 
        'etc' => 'etc'
    ));

Для чтения текущей конфигурации, используйте метод ``get_config``.

.. code-block:: php

    <?php
    $isLoggingEnabled = ORM::get_config('logging');
    ORM::configure('logging', false);
    // какой-то бешенный цикл, который мы не хотим добавлять в лог
    ORM::configure('logging', $isLoggingEnabled);

Подробности аутентификации в базе данных
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметры: ``username`` и ``password``

Некоторые адаптеры баз данных (такие как MySQL) требуют раздельной передачи имени пользователя (username) и пароля (password) в DSN строке. Эти параметры позволяют вам передать их. Типичная настройка соединения MySQL может выгялдеть так:

.. code-block:: php

    <?php
    ORM::configure('mysql:host=localhost;dbname=my_database');
    ORM::configure('username', 'database_user');
    ORM::configure('password', 'top_secret');

Или вы можете соединить настройку соединения в одну строку, используя массив конфигурации:

.. code-block:: php

    <?php
    ORM::configure(array(
        'connection_string' => 'mysql:host=localhost;dbname=my_database', 
        'username' => 'database_user', 
        'password' => 'top_secret'
    ));

Наборы результатов
^^^^^^^^^^^

Параметр: ``return_result_sets``

Коллекция результатов данных может быть возвращена в качестве массива (по-умолчанию) или в виде результирующего набора.
Смотрите документацию о `find_result_set()` для более подробной информации.

.. code-block:: php

    <?php
    ORM::configure('return_result_sets', true); // возвращает результирующий набор


.. примечание::

   Рекомендуется настроить ваши проекты для работы с результирующими наборами, так как они более гибкие.

PDO Параметры драйвера
^^^^^^^^^^^^^^^^^^

Параметр: ``driver_options``

Некоторые адаптеры базы данных требуют (или позволяют использовать) массив параметров конфигурации для конкретного драйвера. Это позволяет вам передавать эти параметры через конструктор PDO. Для более подробной информации, смотрите `документацию PDO`_. Например, чтобы заставить драйвер MySQL использовать кодировку UTF-8 для соединения:

.. code-block:: php

    <?php
    ORM::configure('driver_options', array(PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8'));

PDO Режим ошибок
^^^^^^^^^^^^^^

Параметр: ``error_mode``

Можно использовать для установки параметра ``PDO::ATTR_ERRMODE`` у класса соединения с базой данных, используемой Idiorm. Должна быть передана одна из объявленных в классе констант PDO. Пример:

.. code-block:: php

    <?php
    ORM::configure('error_mode', PDO::ERRMODE_WARNING);

Параметром по-умолчанию является ``PDO::ERRMODE_EXCEPTION``. Для более подробной информации о доступных режимах ошибок, смотри `документация PDO - присвоение атрибута`_.

PDO доступ к объекту
^^^^^^^^^^^^^^^^^

*Перевод находится в процессе*
Should it ever be necessary, the PDO object used by Idiorm may be
accessed directly through ``ORM::get_db()``, or set directly via
``ORM::set_db()``. This should be an unusual occurance.

After a statement has been executed by any means, such as ``::save()``
or ``::raw_execute()``, the ``PDOStatement`` instance used may be
accessed via ``ORM::get_last_statement()``. This may be useful in order
to access ``PDOStatement::errorCode()``, if PDO exceptions are turned
off, or to access the ``PDOStatement::rowCount()`` method, which returns
differing results based on the underlying database. For more
information, see the `PDOStatement documentation`_.

Идентификатор символа кавычек
^^^^^^^^^^^^^^^^^^^^^^^^^^

Параметр: ``identifier_quote_character``

Set the character used to quote identifiers (eg table name, column
name). If this is not set, it will be autodetected based on the database
driver being used by PDO.

ID Column
^^^^^^^^^

By default, the ORM assumes that all your tables have a primary key
column called ``id``. There are two ways to override this: for all
tables in the database, or on a per-table basis.

Setting: ``id_column``

This setting is used to configure the name of the primary key column for
all tables. If your ID column is called ``primary_key``, use:

.. code-block:: php

    <?php
    ORM::configure('id_column', 'primary_key');

You can specify a compound primary key using an array:

.. code-block:: php

    <?php
    ORM::configure('id_column', array('pk_1', 'pk_2'));

Note: If you use a auto-increment column in the compound primary key then it
should be the first one defined into the array.

Setting: ``id_column_overrides``

This setting is used to specify the primary key column name for each
table separately. It takes an associative array mapping table names to
column names. If, for example, your ID column names include the name of
the table, you can use the following configuration:

.. code-block:: php

    <?php
    ORM::configure('id_column_overrides', array(
        'person' => 'person_id',
        'role' => 'role_id',
    ));

As with ``id_column`` setting, you can specify a compound primary key
using an array.

Limit clause style
^^^^^^^^^^^^^^^^^^

Setting: ``limit_clause_style``

You can specify the limit clause style in the configuration. This is to facilitate
a MS SQL style limit clause that uses the ``TOP`` syntax.

Acceptable values are ``ORM::LIMIT_STYLE_TOP_N`` and ``ORM::LIMIT_STYLE_LIMIT``.

.. note::

    If the PDO driver you are using is one of sqlsrv, dblib or mssql then Idiorm
    will automatically select the ``ORM::LIMIT_STYLE_TOP_N`` for you unless you
    override the setting.

Query logging
^^^^^^^^^^^^^

Setting: ``logging``

Idiorm can log all queries it executes. To enable query logging, set the
``logging`` option to ``true`` (it is ``false`` by default).

When query logging is enabled, you can use two static methods to access
the log. ``ORM::get_last_query()`` returns the most recent query
executed. ``ORM::get_query_log()`` returns an array of all queries
executed.

Query logger
^^^^^^^^^^^^

Setting: ``logger``

.. note::

    You must enable ``logging`` for this setting to have any effect.

It is possible to supply a ``callable`` to this configuration setting, which will
be executed for every query that idiorm executes. In PHP a ``callable`` is anything
that can be executed as if it were a function. Most commonly this will take the
form of a anonymous function.

This setting is useful if you wish to log queries with an external library as it
allows you too whatever you would like from inside the callback function.

.. code-block:: php

    <?php
    ORM::configure('logger', function($log_string, $query_time) {
        echo $log_string . ' in ' . $query_time;
    });

Query caching
^^^^^^^^^^^^^

Setting: ``caching``

Idiorm can cache the queries it executes during a request. To enable
query caching, set the ``caching`` option to ``true`` (it is ``false``
by default).

.. code-block:: php

    <?php
    ORM::configure('caching', true);
    
    
Setting: ``caching_auto_clear``

Idiorm's cache is never cleared by default. If you wish to automatically clear it on save, set ``caching_auto_clear`` to ``true``

.. code-block:: php

    <?php
    ORM::configure('caching_auto_clear', true);

When query caching is enabled, Idiorm will cache the results of every
``SELECT`` query it executes. If Idiorm encounters a query that has
already been run, it will fetch the results directly from its cache and
not perform a database query.

Warnings and gotchas
''''''''''''''''''''

-  Note that this is an in-memory cache that only persists data for the
   duration of a single request. This is *not* a replacement for a
   persistent cache such as `Memcached`_.

-  Idiorm’s cache is very simple, and does not attempt to invalidate
   itself when data changes. This means that if you run a query to
   retrieve some data, modify and save it, and then run the same query
   again, the results will be stale (ie, they will not reflect your
   modifications). This could potentially cause subtle bugs in your
   application. If you have caching enabled and you are experiencing odd
   behaviour, disable it and try again. If you do need to perform such
   operations but still wish to use the cache, you can call the
   ``ORM::clear_cache()`` to clear all existing cached queries.

-  Enabling the cache will increase the memory usage of your
   application, as all database rows that are fetched during each
   request are held in memory. If you are working with large quantities
   of data, you may wish to disable the cache.

Custom caching
''''''''''''''

If you wish to use custom caching functions, you can set them from the configure options. 

.. code-block:: php

    <?php
    $my_cache = array();
    ORM::configure('cache_query_result', function ($cache_key, $value, $table_name, $connection_name) use (&$my_cache) {
        $my_cache[$cache_key] = $value;
    });
    ORM::configure('check_query_cache', function ($cache_key, $table_name, $connection_name) use (&$my_cache) {
        if(isset($my_cache[$cache_key])){
           return $my_cache[$cache_key];
        } else {
        return false;
        }
    });
    ORM::configure('clear_cache', function ($table_name, $connection_name) use (&$my_cache) {
         $my_cache = array();
    });

    ORM::configure('create_cache_key', function ($query, $parameters, $table_name, $connection_name) {
        $parameter_string = join(',', $parameters);
        $key = $query . ':' . $parameter_string;
        $my_key = 'my-prefix'.crc32($key);
        return $my_key;
    });


.. _документацию PDO: http://php.net/manual/ru/pdo.construct.php
.. _the PDO documentation: http://php.net/manual/en/pdo.construct.php
.. _документация PDO - присвоение атрибута: http://php.net/manual/ru/pdo.setattribute.php
.. _PDOStatement documentation: http://php.net/manual/en/class.pdostatement.php
.. _Memcached: http://www.memcached.org/
