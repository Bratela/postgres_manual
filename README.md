# Полный мануал по PostgreSQL

## Содержание
1. [Установка и настройка](#установка-и-настройка)
2. [Основные команды psql](#основные-команды-psql)
3. [Работа с базами данных](#работа-с-базами-данных)
4. [Работа с таблицами](#работа-с-таблицами)
5. [Типы данных](#типы-данных)
6. [Индексы и производительность](#индексы-и-производительность)
7. [Связи между таблицами](#связи-между-таблицами)
8. [Транзакции](#транзакции)
9. [Функции и процедуры](#функции-и-процедуры)
10. [Триггеры](#триггеры)
11. [Представления (Views)](#представления-views)
12. [Резервное копирование](#резервное-копирование)
13. [Мониторинг и диагностика](#мониторинг-и-диагностика)
14. [Настройка производительности](#настройка-производительности)
15. [Безопасность](#безопасность)

## Установка и настройка

### Ubuntu/Debian
```bash
# Обновление пакетов
sudo apt update

# Установка PostgreSQL
sudo apt install postgresql postgresql-contrib

# Запуск службы
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### CentOS/RHEL
```bash
# Установка репозитория PostgreSQL
sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

# Установка PostgreSQL
sudo yum install -y postgresql15-server postgresql15-contrib

# Инициализация базы данных
sudo /usr/pgsql-15/bin/postgresql-15-setup initdb

# Запуск службы
sudo systemctl start postgresql-15
sudo systemctl enable postgresql-15
```

### macOS (с Homebrew)
```bash
# Установка
brew install postgresql

# Запуск службы
brew services start postgresql
```

### Docker
```bash
# Запуск контейнера PostgreSQL
docker run --name postgres-db \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -d postgres:15
```

### Первоначальная настройка
```bash
# Подключение к PostgreSQL от имени пользователя postgres
sudo -u postgres psql

# Установка пароля для пользователя postgres
\password postgres

# Выход
\q
```

## Основные команды psql

### Подключение к базе данных
```bash
# Основной синтаксис
psql -h hostname -p port -U username -d database

# Примеры
psql -h localhost -U postgres -d mydb
psql postgresql://username:password@localhost:5432/database
```

### Мета-команды psql
```sql
-- Информация о базах данных
\l                    -- список всех баз данных
\c database_name      -- подключение к базе данных
\conninfo            -- информация о текущем соединении

-- Информация о схемах и таблицах
\dn                  -- список схем
\dt                  -- список таблиц в текущей схеме
\dt schema_name.*    -- список таблиц в определенной схеме
\d table_name        -- описание структуры таблицы
\d+ table_name       -- подробное описание таблицы

-- Информация о пользователях и правах
\du                  -- список пользователей
\dp                  -- права доступа к таблицам
\z                   -- альтернативный способ просмотра прав

-- Информация о функциях и процедурах
\df                  -- список функций
\df+ function_name   -- подробная информация о функции

-- Системная информация
\timing              -- включить/выключить отображение времени выполнения
\x                   -- расширенный вывод (вертикальный формат)
\q                   -- выход из psql
```

### Настройка среды psql
```sql
-- Включение расширенного вывода
\x auto

-- Настройка формата вывода
\pset format wrapped
\pset columns 120

-- Сохранение истории команд
\set HISTFILE ~/.psql_history
\set HISTCONTROL ignoredups
```

## Работа с базами данных

### Создание и управление базами данных
```sql
-- Создание базы данных
CREATE DATABASE mydb
    WITH 
    OWNER = postgres
    ENCODING = 'UTF8'
    LC_COLLATE = 'en_US.utf8'
    LC_CTYPE = 'en_US.utf8'
    TEMPLATE = template0;

-- Удаление базы данных
DROP DATABASE mydb;

-- Переименование базы данных
ALTER DATABASE oldname RENAME TO newname;

-- Изменение владельца базы данных
ALTER DATABASE mydb OWNER TO newowner;
```

### Создание и управление пользователями
```sql
-- Создание пользователя
CREATE USER myuser WITH PASSWORD 'securepassword';

-- Создание пользователя с дополнительными привилегиями
CREATE USER admin_user WITH 
    PASSWORD 'adminpass'
    CREATEDB
    CREATEROLE;

-- Изменение пароля пользователя
ALTER USER myuser WITH PASSWORD 'newpassword';

-- Предоставление привилегий
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;
GRANT SELECT, INSERT, UPDATE ON TABLE mytable TO myuser;
GRANT USAGE ON SCHEMA public TO myuser;

-- Отзыв привилегий
REVOKE ALL PRIVILEGES ON DATABASE mydb FROM myuser;

-- Удаление пользователя
DROP USER myuser;
```

### Схемы (Schemas)
```sql
-- Создание схемы
CREATE SCHEMA myschema;

-- Создание схемы с владельцем
CREATE SCHEMA myschema AUTHORIZATION myuser;

-- Установка пути поиска схем
SET search_path TO myschema, public;

-- Просмотр текущего пути поиска
SHOW search_path;

-- Удаление схемы
DROP SCHEMA myschema CASCADE;
```

## Работа с таблицами

### Создание таблиц
```sql
-- Базовый пример создания таблицы
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    date_of_birth DATE,
    is_active BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Создание таблицы с внешним ключом
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    content TEXT,
    author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    published_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT CURRENT_TIMESTAMP
);

-- Создание таблицы с ограничениями
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2) CHECK (price > 0),
    category_id INTEGER NOT NULL,
    stock_quantity INTEGER DEFAULT 0 CHECK (stock_quantity >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Изменение структуры таблиц
```sql
-- Добавление столбца
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- Удаление столбца
ALTER TABLE users DROP COLUMN phone;

-- Изменение типа столбца
ALTER TABLE users ALTER COLUMN username TYPE VARCHAR(100);

-- Добавление ограничения
ALTER TABLE users ADD CONSTRAINT check_email 
    CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$');

-- Удаление ограничения
ALTER TABLE users DROP CONSTRAINT check_email;

-- Переименование столбца
ALTER TABLE users RENAME COLUMN username TO login;

-- Переименование таблицы
ALTER TABLE users RENAME TO app_users;
```

### Операции с данными
```sql
-- Вставка данных
INSERT INTO users (username, email, first_name, last_name) 
VALUES ('john_doe', 'john@example.com', 'John', 'Doe');

-- Множественная вставка
INSERT INTO users (username, email, first_name, last_name) VALUES
    ('jane_smith', 'jane@example.com', 'Jane', 'Smith'),
    ('bob_wilson', 'bob@example.com', 'Bob', 'Wilson'),
    ('alice_brown', 'alice@example.com', 'Alice', 'Brown');

-- Вставка с возвратом данных
INSERT INTO users (username, email, first_name, last_name)
VALUES ('new_user', 'new@example.com', 'New', 'User')
RETURNING id, created_at;

-- Обновление данных
UPDATE users 
SET 
    first_name = 'Johnny',
    updated_at = CURRENT_TIMESTAMP
WHERE username = 'john_doe';

-- Обновление с подзапросом
UPDATE posts 
SET published_at = CURRENT_TIMESTAMP 
WHERE author_id IN (
    SELECT id FROM users WHERE is_active = TRUE
);

-- Удаление данных
DELETE FROM users WHERE is_active = FALSE;

-- Удаление с возвратом данных
DELETE FROM users 
WHERE created_at < CURRENT_DATE - INTERVAL '1 year'
RETURNING id, username;
```

### Выборка данных
```sql
-- Базовая выборка
SELECT id, username, email FROM users;

-- Выборка с условием
SELECT * FROM users 
WHERE is_active = TRUE 
AND created_at >= CURRENT_DATE - INTERVAL '30 days';

-- Выборка с сортировкой
SELECT username, email, created_at 
FROM users 
ORDER BY created_at DESC, username ASC;

-- Выборка с ограничением
SELECT * FROM users 
ORDER BY created_at DESC 
LIMIT 10 OFFSET 20;

-- Группировка и агрегация
SELECT 
    DATE_TRUNC('month', created_at) as month,
    COUNT(*) as user_count,
    COUNT(CASE WHEN is_active THEN 1 END) as active_users
FROM users 
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month DESC;

-- Соединения таблиц
SELECT 
    u.username,
    u.email,
    p.title,
    p.published_at
FROM users u
INNER JOIN posts p ON u.id = p.author_id
WHERE p.published_at IS NOT NULL
ORDER BY p.published_at DESC;

-- Левое соединение
SELECT 
    u.username,
    COUNT(p.id) as post_count
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id, u.username
ORDER BY post_count DESC;
```

## Типы данных

### Числовые типы
```sql
-- Целые числа
SMALLINT        -- -32,768 to 32,767
INTEGER (INT)   -- -2,147,483,648 to 2,147,483,647
BIGINT          -- -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
SERIAL          -- автоинкремент для INTEGER
BIGSERIAL       -- автоинкремент для BIGINT

-- Числа с плавающей точкой
REAL            -- 6 десятичных знаков точности
DOUBLE PRECISION -- 15 десятичных знаков точности
NUMERIC(p,s)    -- точное число с p цифрами, s после запятой
DECIMAL(p,s)    -- синоним для NUMERIC
```

### Строковые типы
```sql
-- Строки
CHAR(n)         -- строка фиксированной длины
VARCHAR(n)      -- строка переменной длины до n символов
TEXT            -- строка неограниченной длины

-- Примеры использования
CREATE TABLE example_strings (
    fixed_char CHAR(10),
    variable_char VARCHAR(100),
    unlimited_text TEXT
);
```

### Типы даты и времени
```sql
-- Типы даты и времени
DATE                        -- только дата
TIME                        -- только время
TIMESTAMP                   -- дата и время
TIMESTAMP WITH TIME ZONE    -- дата и время с часовым поясом
INTERVAL                    -- интервал времени

-- Примеры
CREATE TABLE time_examples (
    event_date DATE,
    event_time TIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    scheduled_at TIMESTAMP WITH TIME ZONE,
    duration INTERVAL
);

-- Работа с датами
SELECT 
    CURRENT_DATE,
    CURRENT_TIME,
    CURRENT_TIMESTAMP,
    NOW(),
    DATE_TRUNC('month', NOW()),
    EXTRACT(YEAR FROM NOW()),
    AGE(CURRENT_DATE, '1990-01-01'::DATE);
```

### Логические и специальные типы
```sql
-- Логический тип
BOOLEAN (BOOL)  -- TRUE, FALSE, NULL

-- JSON типы
JSON            -- текстовое представление JSON
JSONB           -- бинарное представление JSON (рекомендуется)

-- Массивы
INTEGER[]       -- массив целых чисел
TEXT[]          -- массив строк

-- UUID
UUID            -- универсальный уникальный идентификатор

-- Примеры использования
CREATE TABLE advanced_types (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    is_active BOOLEAN DEFAULT TRUE,
    metadata JSONB,
    tags TEXT[],
    coordinates POINT
);
```

## Индексы и производительность

### Типы индексов
```sql
-- B-tree индекс (по умолчанию)
CREATE INDEX idx_users_email ON users(email);

-- Составной индекс
CREATE INDEX idx_users_name_email ON users(last_name, first_name);

-- Частичный индекс
CREATE INDEX idx_active_users ON users(username) WHERE is_active = TRUE;

-- Уникальный индекс
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- GIN индекс для JSONB
CREATE INDEX idx_users_metadata ON users USING GIN(metadata);

-- GiST индекс для полнотекстового поиска
CREATE INDEX idx_posts_search ON posts USING GIN(to_tsvector('english', title || ' ' || content));

-- Hash индекс (только для равенства)
CREATE INDEX idx_users_id_hash ON users USING HASH(id);
```

### Анализ производительности
```sql
-- Анализ плана выполнения
EXPLAIN SELECT * FROM users WHERE email = 'john@example.com';

-- Анализ с реальным выполнением
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';

-- Подробный анализ
EXPLAIN (ANALYZE, BUFFERS, VERBOSE) 
SELECT u.username, COUNT(p.id) 
FROM users u 
LEFT JOIN posts p ON u.id = p.author_id 
GROUP BY u.username;

-- Обновление статистики
ANALYZE users;
ANALYZE; -- для всех таблиц
```

### Оптимизация запросов
```sql
-- Использование индекса для сортировки
CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- Оптимизация WHERE условий
-- Плохо: функция в WHERE
SELECT * FROM users WHERE UPPER(username) = 'JOHN';

-- Хорошо: функциональный индекс
CREATE INDEX idx_users_username_upper ON users(UPPER(username));
SELECT * FROM users WHERE UPPER(username) = 'JOHN';

-- Оптимизация LIKE запросов
CREATE INDEX idx_users_email_pattern ON users(email varchar_pattern_ops);
SELECT * FROM users WHERE email LIKE 'john%';
```

## Связи между таблицами

### Внешние ключи
```sql
-- Создание таблицы с внешним ключом
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id) ON DELETE RESTRICT,
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    order_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Добавление внешнего ключа к существующей таблице
ALTER TABLE posts 
ADD CONSTRAINT fk_posts_author 
FOREIGN KEY (author_id) REFERENCES users(id) ON DELETE CASCADE;

-- Опции для внешних ключей
-- ON DELETE CASCADE    - удалить связанные записи
-- ON DELETE RESTRICT   - запретить удаление если есть связанные записи
-- ON DELETE SET NULL   - установить NULL в связанных записях
-- ON DELETE SET DEFAULT - установить значение по умолчанию
```

### Типы связей
```sql
-- Один к одному (1:1)
CREATE TABLE user_profiles (
    user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    bio TEXT,
    avatar_url VARCHAR(255),
    website VARCHAR(255)
);

-- Один ко многим (1:N) - уже показано выше с users и posts

-- Многие ко многим (M:N)
CREATE TABLE user_roles (
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    role_id INTEGER REFERENCES roles(id) ON DELETE CASCADE,
    assigned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id, role_id)
);

-- Самосвязь
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    parent_id INTEGER REFERENCES categories(id) ON DELETE SET NULL
);
```

## Транзакции

### Основы транзакций
```sql
-- Простая транзакция
BEGIN;
    INSERT INTO users (username, email) VALUES ('test_user', 'test@example.com');
    UPDATE users SET is_active = FALSE WHERE id = 1;
COMMIT;

-- Транзакция с откатом
BEGIN;
    DELETE FROM posts WHERE author_id = 1;
    -- Если что-то пошло не так
    ROLLBACK;
    
-- Транзакция с точкой сохранения
BEGIN;
    INSERT INTO users (username, email) VALUES ('user1', 'user1@example.com');
    SAVEPOINT sp1;
    
    INSERT INTO users (username, email) VALUES ('user2', 'user2@example.com');
    SAVEPOINT sp2;
    
    -- Откат к точке сохранения
    ROLLBACK TO sp1;
    
    INSERT INTO users (username, email) VALUES ('user3', 'user3@example.com');
COMMIT;
```

### Уровни изоляции
```sql
-- Установка уровня изоляции для транзакции
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
-- или
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
-- или  
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;

-- Установка уровня изоляции для сессии
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Блокировки
```sql
-- Явная блокировка строк
SELECT * FROM users WHERE id = 1 FOR UPDATE;

-- Блокировка для чтения
SELECT * FROM users WHERE id = 1 FOR SHARE;

-- Блокировка таблицы
LOCK TABLE users IN ACCESS EXCLUSIVE MODE;
```

## Функции и процедуры

### Встроенные функции
```sql
-- Строковые функции
SELECT 
    LENGTH('Hello World'),
    UPPER('hello'),
    LOWER('WORLD'),
    SUBSTRING('PostgreSQL' FROM 1 FOR 8),
    REPLACE('Hello World', 'World', 'PostgreSQL'),
    CONCAT('Hello', ' ', 'World'),
    LEFT('PostgreSQL', 8),
    RIGHT('PostgreSQL', 3);

-- Математические функции
SELECT 
    ABS(-15),
    CEIL(4.3),
    FLOOR(4.7),
    ROUND(4.567, 2),
    POWER(2, 3),
    SQRT(16),
    RANDOM();

-- Функции даты и времени
SELECT 
    NOW(),
    CURRENT_DATE,
    CURRENT_TIME,
    DATE_PART('year', NOW()),
    DATE_TRUNC('month', NOW()),
    AGE('2023-01-01', '1990-01-01'),
    NOW() + INTERVAL '1 month';
```

### Пользовательские функции
```sql
-- Простая функция
CREATE OR REPLACE FUNCTION get_user_full_name(user_id INTEGER)
RETURNS TEXT AS $$
BEGIN
    RETURN (
        SELECT CONCAT(first_name, ' ', last_name) 
        FROM users 
        WHERE id = user_id
    );
END;
$$ LANGUAGE plpgsql;

-- Использование функции
SELECT get_user_full_name(1);

-- Функция с несколькими параметрами
CREATE OR REPLACE FUNCTION calculate_age(birth_date DATE)
RETURNS INTEGER AS $$
BEGIN
    RETURN DATE_PART('year', AGE(birth_date));
END;
$$ LANGUAGE plpgsql;

-- Функция возвращающая таблицу
CREATE OR REPLACE FUNCTION get_user_posts(user_id INTEGER)
RETURNS TABLE(
    post_id INTEGER,
    title VARCHAR(200),
    created_at TIMESTAMP
) AS $$
BEGIN
    RETURN QUERY
    SELECT p.id, p.title, p.created_at
    FROM posts p
    WHERE p.author_id = user_id
    ORDER BY p.created_at DESC;
END;
$$ LANGUAGE plpgsql;

-- Использование табличной функции
SELECT * FROM get_user_posts(1);
```

### Процедуры (PostgreSQL 11+)
```sql
-- Создание процедуры
CREATE OR REPLACE PROCEDURE update_user_activity()
LANGUAGE plpgsql AS $$
BEGIN
    UPDATE users 
    SET is_active = FALSE 
    WHERE last_login < CURRENT_DATE - INTERVAL '90 days';
    
    RAISE NOTICE 'Updated % inactive users', ROW_COUNT;
END;
$$;

-- Вызов процедуры
CALL update_user_activity();
```

## Триггеры

### Создание триггеров
```sql
-- Функция для триггера обновления времени
CREATE OR REPLACE FUNCTION update_modified_time()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updated_at = CURRENT_TIMESTAMP;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Создание триггера
CREATE TRIGGER trigger_users_updated_at
    BEFORE UPDATE ON users
    FOR EACH ROW
    EXECUTE FUNCTION update_modified_time();

-- Триггер для логирования изменений
CREATE TABLE user_audit (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    action VARCHAR(10),
    old_data JSONB,
    new_data JSONB,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE OR REPLACE FUNCTION log_user_changes()
RETURNS TRIGGER AS $$
BEGIN
    IF TG_OP = 'INSERT' THEN
        INSERT INTO user_audit (user_id, action, new_data)
        VALUES (NEW.id, 'INSERT', row_to_json(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'UPDATE' THEN
        INSERT INTO user_audit (user_id, action, old_data, new_data)
        VALUES (NEW.id, 'UPDATE', row_to_json(OLD), row_to_json(NEW));
        RETURN NEW;
    ELSIF TG_OP = 'DELETE' THEN
        INSERT INTO user_audit (user_id, action, old_data)
        VALUES (OLD.id, 'DELETE', row_to_json(OLD));
        RETURN OLD;
    END IF;
END;
$$ LANGUAGE plpgsql;

-- Создание триггеров для всех операций
CREATE TRIGGER trigger_user_audit
    AFTER INSERT OR UPDATE OR DELETE ON users
    FOR EACH ROW
    EXECUTE FUNCTION log_user_changes();
```

## Представления (Views)

### Создание представлений
```sql
-- Простое представление
CREATE VIEW active_users AS
SELECT id, username, email, first_name, last_name, created_at
FROM users
WHERE is_active = TRUE;

-- Использование представления
SELECT * FROM active_users ORDER BY created_at DESC;

-- Сложное представление с соединениями
CREATE VIEW user_post_stats AS
SELECT 
    u.id,
    u.username,
    u.email,
    COUNT(p.id) as total_posts,
    COUNT(CASE WHEN p.published_at IS NOT NULL THEN 1 END) as published_posts,
    MAX(p.created_at) as last_post_date
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id, u.username, u.email;

-- Материализованное представление (для производительности)
CREATE MATERIALIZED VIEW monthly_user_stats AS
SELECT 
    DATE_TRUNC('month', created_at) as month,
    COUNT(*) as new_users,
    COUNT(CASE WHEN is_active THEN 1 END) as active_users
FROM users
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY month;

-- Обновление материализованного представления
REFRESH MATERIALIZED VIEW monthly_user_stats;

-- Обновление без блокировки (если есть уникальный индекс)
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_user_stats;
```

## Резервное копирование

### Создание резервных копий
```bash
# Полное резервное копирование базы данных
pg_dump -U username -h localhost -d database_name > backup.sql

# Резервное копирование с сжатием
pg_dump -U username -h localhost -d database_name | gzip > backup.sql.gz

# Резервное копирование в формате directory
pg_dump -U username -h localhost -d database_name -F d -f backup_dir/

# Резервное копирование только схемы
pg_dump -U username -h localhost -d database_name --schema-only > schema.sql

# Резервное копирование только данных
pg_dump -U username -h localhost -d database_name --data-only > data.sql

# Резервное копирование конкретных таблиц
pg_dump -U username -h localhost -d database_name -t users -t posts > tables.sql

# Резервное копирование всех баз данных
pg_dumpall -U postgres > all_databases.sql
```

### Восстановление из резервных копий
```bash
# Восстановление из SQL файла
psql -U username -h localhost -d database_name < backup.sql

# Восстановление из сжатого файла
gunzip -c backup.sql.gz | psql -U username -h localhost -d database_name

# Восстановление из directory формата
pg_restore -U username -h localhost -d database_name backup_dir/

# Восстановление с созданием базы данных
pg_restore -U username -h localhost -C -d postgres backup.dump

# Восстановление только схемы
pg_restore -U username -h localhost -d database_name --schema-only backup.dump

# Восстановление конкретных таблиц
pg_restore -U username -h localhost -d database_name -t users backup.dump
```

### Автоматизация резервного копирования
```bash
#!/bin/bash
# Скрипт автоматического резервного копирования

DB_NAME="mydb"
DB_USER="postgres"
BACKUP_DIR="/backup/postgresql"
DATE=$(date +"%Y%m%d_%H%M%S")

# Создание директории если не существует
mkdir -p $BACKUP_DIR

# Создание резервной копии
pg_dump -U $DB_USER -d $DB_NAME | gzip > $BACKUP_DIR/backup_$DATE.sql.gz

# Удаление старых резервных копий (старше 7 дней)
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +7 -delete

echo "Backup completed: backup_$DATE.sql.gz"
```

## Мониторинг и диагностика

### Системные таблицы и представления
```sql
-- Активные соединения
SELECT 
    pid,
    usename,
    application_name,
    client_addr,
    state,
    query_start,
    query
FROM pg_stat_activity
WHERE state = 'active';

-- Статистика по таблицам
SELECT 
    schemaname,
    tablename,
    n_tup_ins as inserts,
    n_tup_upd as updates,
    n_tup_del as deletes,
    n_live_tup as live_rows,
    n_dead_tup as dead_rows
FROM pg_stat_user_tables
ORDER BY n_tup_ins + n_tup_upd + n_tup_del DESC;

-- Статистика по индексам
SELECT 
    schemaname,
    tablename,
    indexname,
    idx_tup_read,
    idx_tup_fetch,
    idx_scan
FROM pg_stat_user_indexes
ORDER BY idx_scan DESC;

-- Размеры таблиц
SELECT 
    tablename,
    pg_size_pretty(pg_total_relation_size(tablename::regclass)) as size,
    pg_size_pretty(pg_relation_size(tablename::regclass)) as table_size,
    pg_size_pretty(pg_total_relation_size(tablename::regclass) - pg_relation_size(tablename::regclass)) as index_size
FROM pg_tables 
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(tablename::regclass) DESC;

-- Медленные запросы (требует настройки log_min_duration_statement)
SELECT 
    query,
    calls,
    total_time,
