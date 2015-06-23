Транзакции
============

Idiorm не поставляет каких-либо специальных методов для работы с транзакциями, но можно с легкостью использовать встронные в PDO меетоды:

.. code-block:: php

    <?php
    // Начало транзакции
    ORM::get_db()->beginTransaction();

    // Коммит транзакции
    ORM::get_db()->commit();

    // Откат транзакции
    ORM::get_db()->rollBack();

Для более подробной информации, ознакомьтесь с `документацией PDO по Transactions`_.

.. _документацией PDO по Transactions: http://php.net/manual/ru/pdo.transactions.php
