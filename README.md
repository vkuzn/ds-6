# Задание #6 Горизонтальное масштабирование хранилища данных: сегментирование (шардинг)

Задание касается географических аспектов хранения данных: местоположение данных должно определятся кодом региона, полученным от пользователя.
Код региона выбирается через Dropdown поле на форме ввода и отправляется вместе с текстом методом POST.
Коды регионов:
* RUS - Россия
* EU - Европейский Союз
* USA - США

Требуется реализовать сегментирование данных на стратегии поиска. Ключом сегментирования будет идентификатор контекста обработки *contextId*.

*BackendApi* должен по коду региона определить соответствующий номер базы Redis. Все записи касательно текста (непосредственно текст и его оценка) должны находиться в базе с соответсвующим номером.

При обращении к базе *Redis* в консоль каждого компонента должен выводиться лог с указанием *contextId* и номера используемой базы.


## Сегментирование данных

Централизованое хранилище данных в системах, оперирующих большим объёмом данных, сталкивается со следующими ограничениями:

1. Ограниченный объём хранилища
2. Ограниченные вычислительные ресурсы
3. Ограниченная пропускная способность сети
4. Географические аспекты функционирования системы.

Вертикальное масштабирование дискового пространства, процессорных мощностей и пропускной способности сетевых соединений способно отложить на некоторое время проблемы с вышеуказанными ограничениями. Географические аспекты при этом остаются нетронутыми.

Хорошо показавшим себя на практике способом решения указанных проблем с ограничением является горизонтальное масштабирование данных (сегментирование, шардинг, шардирование). В системах, реализующих сегментирование,  данные с одной и той же структурой хранятся и обрабатываются разными узлами (сегментами, шардами).

Для организации сегментирования данных необходимо:

1. Определить в каком сегменте какие данные должны располагаться.
2. Определить атрибуты, по которым будет определяться принадлежность данных к определённому сегменту. Эти атрибуты называются ключом сегментирования. Ключ должен быть постоянным и не зависеть от изменения данных.
3. Определить стратегии распределения данных по сегментам. Стратегия распределения может быть реализована как в приложении на уровне доступа к данным, так и в самом хранилище, если оно поддерживает прозрачность распределения данных.

Виды стратегий сегментирования:

1. Стратегия поиска. Обеспечивает максимальный контроль над настройкой и использованием сегментов. Стратегия основана на карте сопоставления данных определённым сегментам. Из-за необходимости постоянного обращения к данной карте этап поиска для маршрутизации запросов создаёт дополнительные накладные расходы.
2. Стратегия диапазонов. Легко реализуется, не вносит дополнительных существенных накладных расходов. Основной недостаток: стратегия не позволяет обеспечить оптимальную балансировку между сегментами в случае, если данные по диапазону распределены неравномерно, т.е. один диапазон может содержать большое количество записей, другой быть сильно разреженным.
3. Стратегия хэширования. Позволяет распределять данные и нагрузку наиболее равномерно. Маршрутизация выполняется прямо из приложения с использованием хэш-функции. Но к хэш-функции предъявляется ряд требований, такие как скорость вычисления, равномерность распределения. Имеются сложности в случае, если необходимо перераспределить нагрузку между сегментами.

Выбор стратегий и ключа сегментирования зависит от требований к приложению.
