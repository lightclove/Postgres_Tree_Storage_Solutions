# Postgres_Tree_Storage_Solutions
Tree storage of data structures solutions
# Реализовать в Postgresql хранение древовидной структуры данных


Реализовать в Postgresql хранение древовидной структуры данных в одной или нескольких таблицах.

## Традиционное решение с одной таблицей

В качестве древовидной структуры данных может выступать обычная ячейка общества  или семья, где проще всего на жизненном примере проявляется логическое отношение "родитель-потомок" с использованием традиционных реляционных отношений БД
Создадим базу данных для хранения информации о семье: папе, маме и их трех детях. Допустим  назовем её`family_members`.

 **Шаг 1: Создаем Таблицу**
Мы создаем таблицу для хранения информации о каждом члене семьи. У каждого члена семьи есть уникальный номер (`id`), его имя (`name`), возраст (`age`), и мы добавим столбец `parent_id`, чтобы указать, кто является родителем этого члена семьи.

    CREATE TABLE family_members (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    age INT,
    parent_id INT REFERENCES family_members(id)
);

**Шаг 2: Заполняем Таблицу Данными**

Добавляем информацию о членах семьи. Если член семьи - это ребенок, то в столбце `parent_id` указываем id своего родителя.

    INSERT INTO family_members (name, age, parent_id) VALUES
    ('Папа', 40, NULL),
    ('Мама', 38, NULL),
    ('Сын', 15, 1), -- Сын Папы
    ('Дочь', 10, 1), -- Дочь Папы
    ('Другая Дочь', 8, 2); -- Дочь Мамы
Cоздаем родителей "Папа" и "Мама", а затем добавляем их детей ("Сын", "Дочь", "Другая Дочь") и указываем `parent_id`, чтобы связать их с соответствующими родителями.
Здесь мы добавляем следующих членов семьи:

-   "Папа" (id=1, parent_id=NULL, так как он корневой элемент).
-   "Мама" (id=2, parent_id=NULL, также корневой элемент).
-   "Сын" (id=3, parent_id=1, потому что он сын "Папы").
-   "Дочь" (id=4, parent_id=1, также дочь "Папы").
-   "Другая Дочь" (id=5, parent_id=2, так как она дочь "Мамы").

**Шаг 3: Выполняем Запросы**

Используем SQL-запросы для получения информации о семье. 
Например, чтобы узнать, кто является родителем "Дочь", мы можем использовать следующий запрос:

    SELECT name FROM family_members WHERE id = 5;
  
  Результат будет "Мама", так как "Дочь" имеет `parent_id` равный 2, что соответствует `id` "Мамы".
  
  **Узнаем имена детей "Папы":**
  
    SELECT name FROM family_members WHERE parent_id = 1;

Результат: "Сын" и "Дочь".

  **Узнаем имена родителей "Дочи":**
  

    SELECT name FROM family_members WHERE id = 4;

Результат: "Папа".

**Узнаем имена всех членов семьи:**

    SELECT * FROM family_members;

| id |      name       | age | parent_id |
|----|-----------------|-----|-----------|
|  1 | Папа            |  40 |           |
|  2 | Мама            |  38 |           |
|  3 | Сын             |  15 |         1 |
|  4 | Дочь            |  10 |         1 |
|  5 | Другая Дочь     |   8 |         2 |

  
Можно хранить информацию о семье в базе данных с использованием традиционных реляционных отношений. Каждый член семьи представлен записью в таблице, и отношения между ними отражаются через `parent_id`.

В PostgreSQL нет встроенной готовой структуры данных, специально предназначенной для хранения и работы с вложенными элементами, но существует тип данных `ltree`, который может быть использован для работы с иерархическими структурами данных. Тип `ltree` предназначен для хранения "лексикографических деревьев" и может быть использован для представления иерархических путей.

Пример использования типа данных `ltree`:

    CREATE TABLE nested_tree (
    node_path ltree PRIMARY KEY,
    node_name VARCHAR(255) NOT NULL
);

Пример данных:

    INSERT INTO nested_tree (node_path, node_name) 
    VALUES('root', 'Root'),
    ('root.folderA', 'FolderA'),
    ('root.folderA.subfolderA1', 'SubfolderA1'),
    ('root.folderA.subfolderA2', 'SubfolderA2'),
    ('root.folderB', 'FolderB'),
    ('root.folderB.subfolderB1', 'SubfolderB1');
Запросы для работы с `ltree` могут включать в себя операторы, такие как `@>` (содержит), `<@` (содержится в), `~` (поддерево), что обеспечивает гибкость при работе с иерархическими данными.

Пример запроса для выбора всех узлов, содержащихся внутри определенного узла:

    SELECT * FROM nested_tree WHERE node_path <@ 'root.folderA';
Этот запрос выберет все узлы, которые являются подчиненными узлу root.folderA. Использование `ltree` может иметь свои ограничения и может быть не столь эффективным, как более сложные модели, особенно для больших объемов данных или для выполнения сложных запросов.

Пример заполнения данных с использованием типа данных `ltree`: 
В этом примере мы создадим таблицу для хранения файловой системы с использованием иерархии папок и файлов. Каждая запись будет содержать путь (представленный типом `ltree`) и имя файла или папки.

      CREATE TABLE file_system (
		    path ltree PRIMARY KEY,
		    name VARCHAR(255) NOT NULL,
		    is_folder BOOLEAN NOT NULL
	    );

Вставляем данные:

    INSERT INTO file_system (path, name, is_folder) 
    VALUES('root', 'Root', true),
    ('root.folderA', 'FolderA', true),
    ('root.folderA.subfolderA1', 'SubfolderA1', true),
    ('root.folderA.subfolderA1.fileA1', 'FileA1', false),
    ('root.folderA.subfolderA1.fileA2', 'FileA2', false),
    ('root.folderA.subfolderA2', 'SubfolderA2', true),
    ('root.folderB', 'FolderB', true),
    ('root.folderB.subfolderB1', 'SubfolderB1', true),
    ('root.folderB.subfolderB1.fileB1', 'FileB1', false);

В этом примере:

-   Каждая запись представляет собой элемент файловой системы.
-   `path` представляет собой иерархический путь, представленный типом `ltree`.
-   `name` содержит имя файла или папки.
-   `is_folder` указывает, является ли элемент папкой (true) или файлом (false).

Помимо вышеупомянутых подходов, существуют некоторые дополнительные методы для хранения деревьев в PostgreSQL, хотя они могут требовать дополнительной работы и поддержки. Вот еще несколько подходов:

-   Materialized Path with Arrays (Материализованный путь с использованием массивов)
-   Nested Interval Model (Модель вложенных интервалов)
-   Graph-Based Models (Модели на основе графов) как расширение PostGIS
