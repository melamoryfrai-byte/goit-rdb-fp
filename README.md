
goit-rdb-fp/
# 📊 SQL Homework: Date Functions & JSON (Orders Table)

![MySQL](https://img.shields.io/badge/MySQL-Workbench-blue)
![SQL](https://img.shields.io/badge/SQL-Functions-green)
![Status](https://img.shields.io/badge/status-completed-success)
## 📌 Overview Проєкт охоплює:
---
- створення схеми бази даних;
- імпорт даних;
- нормалізацію таблиці до 3НФ;
- виконання аналітичних SQL-запитів;
- використання вбудованих SQL-функцій;
- створення власної користувацької функції.
---

## 1. Створити схему pandemic
``` sql
CREATE SCHEMA pandemic;
USE pandemic;
```
## 1.2. Перевірити активну базу:

```sql
SELECT DATABASE();
```
## 2. Імпортувати CSV через Table Data Import Wizard
У Workbench перевірка: 

```sql
USE pandemic;

SELECT *
FROM infectious_cases
LIMIT 10;
```
## 2.1 Кількість запитів 

```sql
SELECT COUNT(*) 
FROM infectious_cases;
```

## 3. Нормалізація до 3НФ
Проблема: `Entity` та `Code` повторюються багато разів. Тому виносимо їх в окрему таблицю.
- 3.1 Таблиця країн / територій
```sql
CREATE TABLE entities (
    id INT AUTO_INCREMENT PRIMARY KEY,
    Entity VARCHAR(255) NOT NULL,
    Code VARCHAR(20),
    UNIQUE (Entity, Code)
);
```
- 3.2 Заповнюємо її унікальними значеннями:
```sql
INSERT INTO entities (Entity, Code)
SELECT DISTINCT Entity, Code
FROM infectious_cases;
```
- 3.3 перевірка
```slq
SELECT *
FROM entities
```
## 3.4 Нормалізована таблиця випадків захворювань
```sql
CREATE TABLE infectious_cases_normalized (
    id INT AUTO_INCREMENT PRIMARY KEY,
    entity_id INT NOT NULL,
    Year INT,
    Number_yaws DOUBLE,
    polio_cases DOUBLE,
    cases_guinea_worm DOUBLE,
    Number_rabies DOUBLE,
    Number_malaria DOUBLE,
    Number_hiv DOUBLE,
    Number_tuberculosis DOUBLE,
    Number_smallpox DOUBLE,
    Number_cholera_cases DOUBLE,
    
    FOREIGN KEY (entity_id) REFERENCES entities(id)
);
```

- 3.5 Заповнення нормалізованої таблиці
``` sql
INSERT INTO infectious_cases_normalized (
    entity_id,
    Year,
    Number_yaws,
    polio_cases,
    cases_guinea_worm,
    Number_rabies,
    Number_malaria,
    Number_hiv,
    Number_tuberculosis,
    Number_smallpox,
    Number_cholera_cases
)
SELECT 
    e.id,
    ic.Year,
    NULLIF(ic.Number_yaws, '') AS Number_yaws,
    NULLIF(ic.polio_cases, '') AS polio_cases,
    NULLIF(ic.cases_guinea_worm, '') AS cases_guinea_worm,
    NULLIF(ic.Number_rabies, '') AS Number_rabies,
    NULLIF(ic.Number_malaria, '') AS Number_malaria,
    NULLIF(ic.Number_hiv, '') AS Number_hiv,
    NULLIF(ic.Number_tuberculosis, '') AS Number_tuberculosis,
    NULLIF(ic.Number_smallpox, '') AS Number_smallpox,
    NULLIF(ic.Number_cholera_cases, '') AS Number_cholera_cases
FROM infectious_cases ic
JOIN entities e 
    ON ic.Entity = e.Entity 
   AND ic.Code = e.Code;
LIMIT 10;
```
- 3.6 перевірка
```sql
SELECT *
FROM infectious_cases_normalized
LIMIT 10;
```
## 4. Аналіз Number_rabies
Для кожної унікальної комбінації `Entity` та `Code` порахувати:
- середнє;
- мінімальне;
- максимальне;
- суму.
```sql
SELECT 
    e.Entity,
    e.Code,
    AVG(icn.Number_rabies) AS avg_rabies,
    MIN(icn.Number_rabies) AS min_rabies,
    MAX(icn.Number_rabies) AS max_rabies,
    SUM(icn.Number_rabies) AS sum_rabies
FROM infectious_cases_normalized icn
JOIN entities e 
    ON icn.entity_id = e.id
WHERE icn.Number_rabies IS NOT NULL
GROUP BY e.Entity, e.Code
ORDER BY avg_rabies DESC
LIMIT 10;
```

5. Колонка різниці в роках через SQL-функції
   - Потрібно створити дату 01 січня для кожного року.
``` sql
SELECT 
    id,
    Year,
    STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d') AS year_start_date,
    CURDATE() AS current_date_value,
    TIMESTAMPDIFF(
        YEAR,
        STR_TO_DATE(CONCAT(Year, '-01-01'), '%Y-%m-%d'),
        CURDATE()
    ) AS years_difference
FROM infectious_cases_normalized
LIMIT 10;
```
## 6. Власна функція для різниці в роках
```sql
DROP FUNCTION IF EXISTS get_year_difference;

DELIMITER //

CREATE FUNCTION get_year_difference(input_year INT)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(
        YEAR,
        STR_TO_DATE(CONCAT(input_year, '-01-01'), '%Y-%m-%d'),
        CURDATE()
    );
END //
DELIMITER ;
```

