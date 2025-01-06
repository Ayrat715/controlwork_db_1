# controlwork_db_1
Написание запросов, генерирующих псевдослучайные данные для всех таблиц из домашней работы (см. https://github.com/Ayrat715/homework_db_1). Общее количество строк для всех таблиц не менее 500. В результате выполнения запроса все таблицы содержат данные, не нарушая целостности базы данных.

За основу я взял 6 сущностей из домашней работы №1 (ссылка на репозиторий Github выше), и написал запросы, согласно требованиям выше. И еще добавил свои комментарии с пояснениями своих действий.

```
-- 1. Генерируем записи в "Department" (нет внешних ключей, поэтому можно вставлять сразу)
INSERT INTO Department (DepartmentName, PhoneNumber, BuildingName)
SELECT
    'Department ' || i,
    '+7987' || LPAD((FLOOR(RANDOM() * 10000))::text, 7, '0'),
    'Building ' || ((RANDOM() * 5)::int + 1)
FROM generate_series(1, 20) AS i;


-- 2. Генерируем записи в "Student" (нет внешних ключей в Student, поэтому вставляем сразу)
-- Для поля Gender используем случайный выбор из массива. Для DateOfBirth используем случайную дату в диапазоне +- 30 лет.
INSERT INTO Student (FirstName, LastName, DateOfBirth, Gender, Email)
SELECT
    'FirstName_' || i,
    'LastName_' || i,
    DATE '1990-01-01' + ((RANDOM() * 365 * 30)::int) -- рандомная дата рождения
        ,
    (ARRAY['Male','Female'])[ (1 + floor(random()*3))::int ],
    'user' || i || '@gmail.com'
FROM generate_series(1, 200) AS i;

-- 3. Генерируем записи в "Course" (нет внешних ключей в Course, поэтому вставляем сразу)
INSERT INTO Course (CourseName, Description)
SELECT
    'Course ' || i,
    'Description of course ' || i
FROM generate_series(1, 50) AS i;


-- 4. Генерируем записи в "Classroom" (нет внешних ключей в Classroom, поэтому вставляем сразу)
INSERT INTO Classroom (BuildingName, RoomNumber, Capacity)
SELECT
    'Building ' || ((RANDOM()*5)::int + 1),
    'Room ' || i,
    (RANDOM()*50)::int + 20  -- вместимость от 20 до 70
FROM generate_series(1, 30) AS i;

-- 5. Генерируем записи в "Professor" (FK -> Department). Для ссылки подбираем случайный DepartmentID из уже вставленных Department.
INSERT INTO Professor (FirstName, LastName, DepartmentID)
SELECT
    'ProfessorFirstName_' || i,
    'ProfessorLastName_'  || i,
    (SELECT DepartmentID
       FROM Department
       ORDER BY RANDOM()
       LIMIT 1)  -- любая случайная кафедра
FROM generate_series(1, 70) AS i;


-- 6. Генерируем записи в "Task" (FK -> Course). Для CourseID берём случайную запись из Course.
INSERT INTO Task (CourseID, Title, Description, DueDate)
SELECT
    (SELECT CourseID
       FROM Course
       ORDER BY RANDOM()
       LIMIT 1),
    'Task ' || i,
    'Some task description ' || i,
    CURRENT_DATE + ((RANDOM() * 30)::int)  -- дедлайн в течение 30 дней
FROM generate_series(1, 150) AS i;
```
В результате применения insert-ов получили 6 таблиц сущностей суммарно с 520 строками в них всех. В процессе написания учтено наличие внешних ключей для таблиц-сущностей "Professor" и "Task" для избежания нарушения целостности базы данных.
