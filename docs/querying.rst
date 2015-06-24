Запросы
========

Idiorm предоставляет *`fluent
interface <https://ru.wikipedia.org/wiki/Fluent_interface>`_* позволяющий построение простых запросов без единого использования SQL. Если вы использовали `jQuery <http://jquery.com>`_ вообще, то будете уже знакомы с концепцией fluent interface. Просто напросто это значит что вы можете
*сцеплять в цепочку* вызов методов вместе, один после другого. Это может сделать ваш код более читабельным, нанизывая методы друг на друга, как-будто вы составляете предложение.

Все запросы в Idiorm начинаются с вызова статического метода ``for_table`` класса ORM. Это сообщает ORM какую таблицу использовать при построении запроса.

*Обратите внимание, что этот метод **не** экранирует свои параметры запроса, так что имя таблицы **не** должно быть передано напрямую от от вводимых пользователем данных.*

Вызовы метода добавляют фильтры и ограничения в ваш запрос, нанизываясь друг на друга. Наконец, цепочка заканчивается вызовом метода
``find_one()`` или ``find_many()``\, которые выполняют запрос и возращают результат.

Начнем с простых примеров. Скажим у нас есть таблица ``person`` содержащая столбцы ``id`` (первичный ключ записи -
Idiorm предполагает что столбец первчиных ключей называется ``id`` но это можно настроить, смотри ниже), ``name``\, ``age`` и ``gender``\.

Примечание по PSR-1 и стилю camelCase
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Все методы, описанные в документации могут так же быть вызваны, используя стандарт PSR-1:
подчеркивания (_) заменяются на стиль написания camelCase. Далее приведен пример одной цепочки запроса, конвертированной в совместимый с PSR-1 стиль.

.. code-block:: php

    <?php
    // документация и стиль по-умолчанию
    $person = ORM::for_table('person')->where('name', 'Fred Bloggs')->find_one();

    // PSR-1 совместимый стиль
    $person = ORM::forTable('person')->where('name', 'Fred Bloggs')->findOne();

Как вы можете заметить, любой метод может быть изменен из формата с подчеркиваниями (_) как в документации в стиль camelCase.

.. примечание::

    В фоне, PSR-1 совместимый стиль использует магические методы `__call()` и 
    `__callStatic()` для описания имен методов в стилистике camelCase у переданных методов с использованием подчеркивания. Затем используется `call_user_func_array()` для применения аргументов к методу. Если такие минимальные расходы ресурсов для вас большие, то вы можете просто вернуться к методам с подчеркиваниями, для избежания всего этого. В общем, это не будет узким местом в каком-либо приложении, однако и должно рассматриваться как микро-оптимизация.

    Так как `__callStatic()` был добавлен в PHP 5.3.0 то вам нужно использовать как минимум эту версию PHP для использования этой возможности, так что подход к этому методу должен быть осмысленным.

Одиночные записи
^^^^^^^^^^^^^^

Любая цепочка методов, заканчивающаяся на ``find_one()`` вернет либо *единичный* экземпляр класса ORM представляющий ряд из базы данных по указанному запросу, или ``false`` если не было найдено удовлетворяющих запросу записей.

Чтобы найти одиночную запись, где столбец ``name`` имеет значение "Fred
Bloggs":

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->where('name', 'Fred Bloggs')->find_one();

Грубо переводя на язык SQL, это:
``SELECT * FROM person WHERE name = "Fred Bloggs"``

Чтобы найти единичную запись по ID, вы можете передать ID прямо в метод ``find_one``:

.. code-block:: php

    <?php
    $person = ORM::for_table('person')->find_one(5);

Если вы используете составной первичный ключ, то можете найти записи используя массив в качестве параметра:

.. code-block:: php

    <?php
    $person = ORM::for_table('user_role')->find_one(array(
        'user_id' => 34,
        'role_id' => 10
    ));


Множество записей
^^^^^^^^^^^^^^^^

.. примечание::

   Рекомендуется использовать результирующие наборы над массивами - смотрите `Как результирующий набор`
   ниже.

Любая цепочка методов, заканчивающаяся на ``find_many()`` вернет *массив* экземпляров ORM-класса, по одному для каждой удовлетворяющей запросу строки. Если не было найдено ни одной строки, то будет возвращен пустой массив.

Чтобы найти все записи в таблице:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->find_many();

Чтобы найти все записи, где ``gender`` равен ``female``:

.. code-block:: php

    <?php
    $females = ORM::for_table('person')->where('gender', 'female')->find_many();

Как результирующий набор
'''''''''''''''

.. примечание::

   Существует параметр конфигурации ``return_result_sets`` который заставляет метод
   ``find_many()`` по-умолчанию возвращать данные в видео результирующего набора. Рекомендуется включить этот параметр:

   ::

       ORM::configure('return_result_sets', true);

Вы так же можете найти множество записей в качестве результирующих наборов вместо массива экземплятров Idiorm. Это дает преимущество в том, что вы можете запустить пакетные операции на наборе результатов.

Итак, для примера, вместо этого:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->find_many();
    foreach ($people as $person) {
        $person->age = 50;
        $person->save();
    }

Вы можете использовать это:

.. code-block:: php

    <?php
    ORM::for_table('person')->find_result_set()
    ->set('age', 50)
    ->save();

Чтобы это сделать, замените любой вызов метода ``find_many()`` методом ``find_result_set()``.

Результирующий набор ведет себя так же, как и массив, так что вы можете использовать на нем `count()` и `foreach`
как и с массивом.

.. code-block:: php

    <?php
    foreach(ORM::for_table('person')->find_result_set() as $record) {
        echo $record->name;
    }

.. code-block:: php

    <?php
    echo count(ORM::for_table('person')->find_result_set());

.. примечание::
   
   Для удаления множества записей рекомендуется использовать `delete_many()`\, так как этот метод более эффективен, нежели вызов `delete()` на результирующем наборе.

Как ассоциативный массив
'''''''''''''''''''''''

Так же вы можете найти множество записей в виде ассоциативного массива, вместо экземпляров Idiorm. Для этого замените любой вызов метода ``find_many()`` на метод
``find_array()``.

.. code-block:: php

    <?php
    $females = ORM::for_table('person')->where('gender', 'female')->find_array();

Это полезно, если вам нужно преобразовать результат запроса в последовательную форму записи(сериализация массива) для JSON, и вам не нужно дополнительной возможности обновлять возвращаемые данные.

Подсчет результатов
^^^^^^^^^^^^^^^^

Для подсчета числа строк, возвращаемых запросом, вызовите метод ``count()``.

.. code-block:: php

    <?php
    $number_of_people = ORM::for_table('person')->count();

Фильтрация результатов
^^^^^^^^^^^^^^^^^

Idiorm предоставляет семейство методов, позволяющих извлечь только те записи, которые удовлетворяют определенному условию(ям). Эти методы можно вызывать множество раз для построения запроса, а fluent interface у Idiorm позволяет строить *цепочку* из таких методов, для построения читабельных и простых к пониманию запросов.

*Предостережения*
'''''''''

Только подмножество доступных условий, поддерживаемых SQL доступны
при использовании Idiorm. Кроме того, все пункты ``WHERE`` будут соединены с использованием
``AND`` при выполнении запроса. Поддержка ``OR`` в пунктах
``WHERE`` в настоящее время отстутствует.

Данные ограничения являются преднамеренными: ведь это наиболее используемые критерии, и избегая поддержки очень сложных запросов, код Idiorm может оставаться маленьким и простым.

Некоторая поддержка более сложных условий и запросов реализована в методах ``where_raw`` и ``raw_query`` (смотрите ниже). Если вы поймете, что чаще нуждаетесь в в большем функционале, нежели содержит Idiorm,
то возможно пришло время рассмотреть более полнофункциональный ORM.

Равенство: ``where``, ``where_equal``, ``where_not_equal``
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''

По-умолчанию, вызывая ``where`` с двумя параметрами (название столбца и значение), они будут соединены, используя оператор равенства (``=``). Например, вызов ``where('name', 'Fred')`` вернет следующее: ``WHERE name = "Fred"``.

Если ваш стиль написания кода направлен на ясность написанного, а не на краткость, то можно использовать метод ``where_equal`` идентичный методу ``where``.

Метод ``where_not_equal`` добавляет пункт ``WHERE column != "value"`` к вашему запросу.

Можно указать множество столбцов и их значений в пределах одного вызова. В этом случае, вам нужно передать ассоциативный массив в качестве первого параметра. В нотации массива, ключи используются как названия стобцов.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->where(array(
                    'name' => 'Fred',
                    'age' => 20
                ))
                ->find_many();

    // Создаст следующий запрос SQL:
    SELECT * FROM `person` WHERE `name` = "Fred" AND `age` = "20";

Короткая запись: ``where_id_is``
'''''''''''''''''''''''''

Это простой вспомогательный метод, для составления запроса по первичному ключу таблицы.
Смотрит относительно ID столбца, указанного в конфигурации. Если вы используете составной первичный ключ, то нужно передать массив, где ключом является название столбца. Столбцы, не принадлежащие к этому ключу будут игнорироваться.

Короткая запись: ``where_id_in``
'''''''''''''''''''''''''

Этот вспомагательный метод аналогичен ``where_id_is``\, но он ожидает массив первичных ключей для выборки. Так же понимает и составной первичный ключ.

Меньше чем / больше чем: ``where_lt``, ``where_gt``, ``where_lte``, ``where_gte``
''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Есть четыре метода, доступные для неравенств:

-  Меньше чем (less than):
   ``$people = ORM::for_table('person')->where_lt('age', 10)->find_many();``
-  Больше чем (greater than):
   ``$people = ORM::for_table('person')->where_gt('age', 5)->find_many();``
-  Меньше или равен (less than or equal):
   ``$people = ORM::for_table('person')->where_lte('age', 10)->find_many();``
-  Больше или равен (greater than or equal_:
   ``$people = ORM::for_table('person')->where_gte('age', 5)->find_many();``

Сравнение строк: ``where_like`` и ``where_not_like``
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Для добавления пункта ``WHERE ... LIKE``\, используйте:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where_like('name', '%fred%')->find_many();

Аналогично и для ``WHERE ... NOT LIKE``\, используйте:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where_not_like('name', '%bob%')->find_many();

Множественные условия OR
'''''''''''''''''''''''''

Можно добавить простое условие OR в тот же пункт WHERE используя ``where_any_is``. Если вам нужно указать множество условий, используйте массив элементов. Каждый элемент будет ассоциативным массивом, содержащим множество условий.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->where_any_is(array(
                    array('name' => 'Joe', 'age' => 10),
                    array('name' => 'Fred', 'age' => 20)))
                ->find_many();

    // Создаст SQL запрос:
    SELECT * FROM `widget` WHERE (( `name` = 'Joe' AND `age` = '10' ) OR ( `name` = 'Fred' AND `age` = '20' ));

По-умолчанию, оператор равенства используется для каждого столбца, но его можно переопределить для любого столбца, используя второй параметр:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->where_any_is(array(
                    array('name' => 'Joe', 'age' => 10),
                    array('name' => 'Fred', 'age' => 20)), array('age' => '>'))
                ->find_many();

    // Создаст SQL запрос:
    SELECT * FROM `widget` WHERE (( `name` = 'Joe' AND `age` > '10' ) OR ( `name` = 'Fred' AND `age` > '20' ));

Если вы хотите задать свой оператор по-умолчанию для всех столбцов, то нужно передать его как второй параметр:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->where_any_is(array(
                    array('score' => '5', 'age' => 10),
                    array('score' => '15', 'age' => 20)), '>')
                ->find_many();

    // Создаст SQL запрос:
    SELECT * FROM `widget` WHERE (( `score` > '5' AND `age` > '10' ) OR ( `score` > '15' AND `age` > '20' ));

Определение принадлежности: ``where_in`` и ``where_not_in``
'''''''''''''''''''''''''''''''''''''''''''''''''

Для добавления пунктов ``WHERE ... IN ()`` или ``WHERE ... NOT IN ()``\, используйте методы
``where_in`` и ``where_not_in`` соответственно.

Оба метода принимают два аргумента. Первый - название столбца, с которым сравнивать. Второй - *массив* возможных значений. Как и во всех методах ``where_``\, вы можете указать множество столбцов, используя ассоциативный *массив* в качестве параметра.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where_in('name', array('Fred', 'Joe', 'John'))->find_many();

Работа с ``NULL`` значениями: ``where_null`` и ``where_not_null``
'''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''''

Для добавления пункта ``WHERE column IS NULL`` или ``WHERE column IS NOT NULL``\, используйте методы ``where_null`` и ``where_not_null`` соответственно. Оба метода принимают один параметр: название столбца для сравнения.

Необработанный WHERE
'''''''''''''''''

Если вам необходимо создать более сложный запрос, то можно использовать метод ``where_raw`` для указания нужного SQL-фрагмента для пункта WHERE. Данный метод принимает два аргумента: строку, добавляемую к запросу, и
(опционально) массив параметров, который будет связан со строкой. Если параметры были переданы, строка должна содержать знаки вопроса (``?``) как плейсхолдеры, для подстановки вместо них значений из массива, а массив должен содержать значения, которые будут подставлены в строку в соответствующем порядке.

Данный метод можно использовать в цепочке методов вместе с другими методами ``where_*`` а так же с методами вроде ``offset``, ``limit`` и ``order_by_*``. Содержимое переданной строки будет соединено с предыдущими и последующими пунктами WHERE с AND в качестве соединения.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->where('name', 'Fred')
                ->where_raw('(`age` = ? OR `age` = ?)', array(20, 25))
                ->order_by_asc('name')
                ->find_many();

    // Создаст SQL запрос:
    SELECT * FROM `person` WHERE `name` = "Fred" AND (`age` = 20 OR `age` = 25) ORDER BY `name` ASC;

.. примечание::

    Необходимо оборачивать выражение в скобки при использовании ``ALL``,
    ``ANY``, ``BETWEEN``, ``IN``, ``LIKE``, ``OR`` и ``SOME``. В противном случае, приоритет ``AND`` станет сильнее и в примере выше мы получим уже следующее ``WHERE (`name` = "Fred" AND `age` = 20) OR `age` = 25``

Обратите внимание, что этот метод поддерживает только синтакс "плейсхолдера в виде вопроса",
а НЕ синтакс "именной плейсхолдер". Все потому, что PDO не позволяет создавать запросы, содержащие смешанные типы плейсхолдеров. Так же, необходимо убедиться в том, что число вопросов-плейсхолдеров в строке соответствует числу элементов в массиве.

Если вам нужно ещё больше гибкости, вы можете вручную указать всю строку запроса. Смотрите *Необработанные запросы* ниже.

Limit и offset
''''''''''''''''''

*Обратите внимание, что эти методы **не** экранируют свои паораметры в запросе, поэтому они **не** должны передаваться напрямую от пользователя.*

Методы ``limit`` и ``offset`` очень похожи на эквивалентные им в SQL.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where('gender', 'female')->limit(5)->offset(10)->find_many();

Порядок
''''''''

*Обратите внимание, что эти методы **не** экранируют свои паораметры в запросе, поэтому они **не** должны передаваться напрямую от пользователя.*

Доступны два метода для добавления к запросу пункта ``ORDER BY``\.
Это ``order_by_desc`` и ``order_by_asc``, каждый из которых принимает название столбца для сортировки. Имена столбцов будут писаться в кавычках.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->order_by_asc('gender')->order_by_desc('name')->find_many();

Если вы хотите упорядочить по какому-то другому признаку, отличному от названия столбца, то используйте метод ``order_by_expr`` для добавления SQL выражения без кавычек, как в пункте ``ORDER BY``\.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->order_by_expr('SOUNDEX(`name`)')->find_many();

Группировка
^^^^^^^^

*Обратите внимание, что эти методы **не** экранируют свои паораметры в запросе, поэтому они **не** должны передаваться напрямую от пользователя.*

Для добавления пункта ``GROUP BY`` в строку запроса, вызовите метод ``group_by`` передав название столца в качестве аргумента. Можно вызывать этот метод множество раз для добавления большего числа колонок.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where('gender', 'female')->group_by('name')->find_many();

Так же возможно использование ``GROUP BY`` с выражениями из базы данных:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->where('gender', 'female')->group_by_expr("FROM_UNIXTIME(`time`, '%Y-%m')")->find_many();

Having
^^^^^^

При использовании агрегирующих функций в комбинации с ``GROUP BY`` вы можете использовать
``HAVING`` для фильтрации, относительно этих значений.

``HAVING`` работает точно таким же способом, что и все методы ``where*`` в Idiorm.
Замените ``where_`` на ``having_`` для использования этих функций.

Например:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->group_by('name')->having_not_like('name', '%bob%')->find_many();

Result columns
^^^^^^^^^^^^^^

По-умолчанию, все столбцы в выражении ``SELECT`` будут возвращены после запроса. То есть, вызывая:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->find_many();

В результате сформирует запрос:

.. code-block:: php

    <?php
    SELECT * FROM `person`;

Метод ``select`` дает контроль над тем, какие столбцы будут возвращены.
Вызовите ``select`` несколько раз для указания нужных столбцов или используйте ```select_many <#shortcuts-for-specifying-many-columns>```_ для указания нескольких столбцов за раз.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select('name')->select('age')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `name`, `age` FROM `person`;

Optionally, you may also supply a second argument to ``select`` to
specify an alias for the column:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select('name', 'person_name')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `name` AS `person_name` FROM `person`;

Column names passed to ``select`` are quoted automatically, even if they
contain ``table.column``-style identifiers:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select('person.name', 'person_name')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `person`.`name` AS `person_name` FROM `person`;

If you wish to override this behaviour (for example, to supply a
database expression) you should instead use the ``select_expr`` method.
Again, this takes the alias as an optional second argument. You can
specify multiple expressions by calling ``select_expr`` multiple times
or use ```select_many_expr`` <#shortcuts-for-specifying-many-columns>`_ to specify many expressions at once.

.. code-block:: php

    <?php
    // NOTE: For illustrative purposes only. To perform a count query, use the count() method.
    $people_count = ORM::for_table('person')->select_expr('COUNT(*)', 'count')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT COUNT(*) AS `count` FROM `person`;

Shortcuts for specifying many columns
'''''''''''''''''''''''''''''''''''''

``select_many`` and ``select_many_expr`` are very similar, but they
allow you to specify more than one column at once. For example:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select_many('name', 'age')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `name`, `age` FROM `person`;

To specify aliases you need to pass in an array (aliases are set as the
key in an associative array):

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select_many(array('first_name' => 'name'), 'age', 'height')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `name` AS `first_name`, `age`, `height` FROM `person`;

You can pass the the following styles into ``select_many`` and
``select_many_expr`` by mixing and matching arrays and parameters:

.. code-block:: php

    <?php
    select_many(array('alias' => 'column', 'column2', 'alias2' => 'column3'), 'column4', 'column5')
    select_many('column', 'column2', 'column3')
    select_many(array('column', 'column2', 'column3'), 'column4', 'column5')

All the select methods can also be chained with each other so you could
do the following to get a neat select query including an expression:

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->select_many('name', 'age', 'height')->select_expr('NOW()', 'timestamp')->find_many();

Will result in the query:

.. code-block:: php

    <?php
    SELECT `name`, `age`, `height`, NOW() AS `timestamp` FROM `person`;

DISTINCT
^^^^^^^^

To add a ``DISTINCT`` keyword before the list of result columns in your
query, add a call to ``distinct()`` to your query chain.

.. code-block:: php

    <?php
    $distinct_names = ORM::for_table('person')->distinct()->select('name')->find_many();

This will result in the query:

.. code-block:: php

    <?php
    SELECT DISTINCT `name` FROM `person`;

Соединения Join
^^^^^

Idiorm has a family of methods for adding different types of ``JOIN``\ s
to the queries it constructs:

Methods: ``join``, ``inner_join``, ``left_outer_join``,
``right_outer_join``, ``full_outer_join``.

Each of these methods takes the same set of arguments. The following
description will use the basic ``join`` method as an example, but the
same applies to each method.

The first two arguments are mandatory. The first is the name of the
table to join, and the second supplies the conditions for the join. The
recommended way to specify the conditions is as an *array* containing
three components: the first column, the operator, and the second column.
The table and column names will be automatically quoted. For example:

.. code-block:: php

    <?php
    $results = ORM::for_table('person')->join('person_profile', array('person.id', '=', 'person_profile.person_id'))->find_many();

It is also possible to specify the condition as a string, which will be
inserted as-is into the query. However, in this case the column names
will **not** be escaped, and so this method should be used with caution.

.. code-block:: php

    <?php
    // Not recommended because the join condition will not be escaped.
    $results = ORM::for_table('person')->join('person_profile', 'person.id = person_profile.person_id')->find_many();

The ``join`` methods also take an optional third parameter, which is an
``alias`` for the table in the query. This is useful if you wish to join
the table to *itself* to create a hierarchical structure. In this case,
it is best combined with the ``table_alias`` method, which will add an
alias to the *main* table associated with the ORM, and the ``select``
method to control which columns get returned.

.. code-block:: php

    <?php
    $results = ORM::for_table('person')
        ->table_alias('p1')
        ->select('p1.*')
        ->select('p2.name', 'parent_name')
        ->join('person', array('p1.parent', '=', 'p2.id'), 'p2')
        ->find_many();

Необработанные соединения JOIN
'''''''''''''''''

If you need to construct a more complex query, you can use the ``raw_join``
method to specify the SQL fragment for the JOIN clause exactly. This
method takes four required arguments: the string to add to the query,
the conditions is as an *array* containing three components: 
the first column, the operator, and the second column, the table alias and
(optional) the parameters array. If parameters are supplied, 
the string should contain question mark characters (``?``) to represent 
the values to be bound, and the parameter array should contain the values 
to be substituted into the string in the correct order.

This method may be used in a method chain alongside other ``*_join``
methods as well as methods such as ``offset``, ``limit`` and
``order_by_*``. The contents of the string you supply will be connected
with preceding and following JOIN clauses.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')
                ->raw_join(
                    'JOIN (SELECT * FROM role WHERE role.name = ?)', 
                    array('person.role_id', '=', 'role.id'), 
                    'role', 
                    array('role' => 'janitor'))    
                ->order_by_asc('person.name')
                ->find_many();

    // Creates SQL:
    SELECT * FROM `person` JOIN (SELECT * FROM role WHERE role.name = 'janitor') `role` ON `person`.`role_id` = `role`.`id` ORDER BY `person`.`name` ASC

Note that this method only supports "question mark placeholder" syntax,
and NOT "named placeholder" syntax. This is because PDO does not allow
queries that contain a mixture of placeholder types. Also, you should
ensure that the number of question mark placeholders in the string
exactly matches the number of elements in the array.

If you require yet more flexibility, you can manually specify the entire
query. See *Raw queries* below.


Aggregate functions
^^^^^^^^^^^^^^^^^^^

There is support for ``MIN``, ``AVG``, ``MAX`` and ``SUM`` in addition
to ``COUNT`` (documented earlier).

To return a minimum value of column, call the ``min()`` method.

.. code-block:: php

    <?php
    $min = ORM::for_table('person')->min('height');

The other functions (``AVG``, ``MAX`` and ``SUM``) work in exactly the
same manner. Supply a column name to perform the aggregate function on
and it will return an integer.

Необработанные запросы
^^^^^^^^^^^

If you need to perform more complex queries, you can completely specify
the query to execute by using the ``raw_query`` method. This method
takes a string and optionally an array of parameters. The string can
contain placeholders, either in question mark or named placeholder
syntax, which will be used to bind the parameters to the query.

.. code-block:: php

    <?php
    $people = ORM::for_table('person')->raw_query('SELECT p.* FROM person p JOIN role r ON p.role_id = r.id WHERE r.name = :role', array('role' => 'janitor'))->find_many();

The ORM class instance(s) returned will contain data for all the columns
returned by the query. Note that you still must call ``for_table`` to
bind the instances to a particular table, even though there is nothing
to stop you from specifying a completely different table in the query.
This is because if you wish to later called ``save``, the ORM will need
to know which table to update.

Note that using ``raw_query`` is advanced and possibly dangerous, and
Idiorm does not make any attempt to protect you from making errors when
using this method. If you find yourself calling ``raw_query`` often, you
may have misunderstood the purpose of using an ORM, or your application
may be too complex for Idiorm. Consider using a more full-featured
database abstraction system.
