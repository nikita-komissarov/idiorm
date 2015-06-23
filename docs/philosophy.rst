Философия
==========

`Закон Парето`_ гласит, что *20% усилий дают 80% результата.* В терминах разработки ПО, это можно перевести приблизительно так: *20% сложностей дают 80% результата*. Другими словами, вы можете многого добиться, будучи не особо умным.

**Idiorm умышленно создан простым**. В то время, как другие ORM состоят из множества классов с сложной иерархией наследования, у Idiorm только один класс,
``ORM``, который функционирует и как беглый API для создания запросов ``SELECT`` и как простой класс модели CRUD. Если моя догадка верна, этого должно быть достаточно для многих современных приложений. Посмотрим правде в глаза: большинство из нас не создает свой Facebook. Мы работает над проектами от маленьких до больших размеров, где акцент делается на простоту и быстрое развитие, а не на бесконечную гибкость и возможности.

Можно рассматривать **Idiorm** как *микро-ORM*. It could, perhaps, be
“the tie to go along with `Slim`_\ ’s tux” (to borrow a turn of phrase
from `DocumentCloud`_). Or it could be an effective bit of spring
cleaning for one of those horrendous SQL-littered legacy PHP apps you
have to support.

**Idiorm** might also provide a good base upon which to build
higher-level, more complex database abstractions. Например, `Paris`_
это реализация паттерна `Active Record pattern`_ построенная на базе
Idiorm.

.. _Закон Парето: https://ru.wikipedia.org/wiki/%D0%97%D0%B0%D0%BA%D0%BE%D0%BD_%D0%9F%D0%B0%D1%80%D0%B5%D1%82%D0%BE
.. _Slim: http://github.com/codeguy/slim/
.. _DocumentCloud: http://github.com/documentcloud/underscore
.. _Paris: http://github.com/j4mie/paris
.. _Active Record pattern: http://martinfowler.com/eaaCatalog/activeRecord.html
