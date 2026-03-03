1. Отобразите все записи из таблицы `company` по компаниям, которые закрылись.

```sql
SELECT *
FROM company
WHERE status = 'closed';
```

2. Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы `company`. Отсортируйте таблицу по убыванию значений в поле `funding_total`.

```sql
SELECT funding_total
FROM company
WHERE category_code = 'news'
AND country_code = 'USA'
ORDER BY funding_total DESC;
```

3. Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.

```sql
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
AND EXTRACT(YEAR FROM acquired_at) IN (2011,2012,2013);
```

4. Отобразите имя, фамилию и названия аккаунтов людей в поле `network_username`, у которых названия аккаунтов начинаются на `'Silver'`.

```sql
SELECT first_name,
       last_name,
       network_username
FROM people
WHERE network_username LIKE 'Silver%';
```

5. Выведите на экран всю информацию о людях, у которых названия аккаунтов в поле `network_username` содержат подстроку `'money'`, а фамилия начинается на `'K'`.

```sql
SELECT *
FROM people
WHERE network_username LIKE '%money%'
AND last_name LIKE 'K%';
```

6. Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.

```sql
SELECT country_code,
       SUM(funding_total)
FROM company
GROUP BY country_code
ORDER BY sum DESC;
```

7. Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату. Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.

```sql
SELECT funded_at,
       MIN(raised_amount),
       MAX(raised_amount)
FROM funding_round
GROUP BY funded_at
HAVING MIN(raised_amount) != 0 AND MIN(raised_amount) != MAX(raised_amount);
```

8. Создайте поле с категориями:
- Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию `high_activity`.
- Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию `middle_activity`.
- Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию `low_activity`.
Отобразите все поля таблицы `fund` и новое поле с категориями.

```sql
SELECT *,
       CASE
        WHEN invested_companies >= 100 THEN 'high_activity'
        WHEN invested_companies >= 20 THEN 'middle_activity'
        ELSE 'low_activity'
       END
FROM fund;
```

9. Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонды принимали участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.

```sql
SELECT CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity,
       ROUND(AVG(investment_rounds))
FROM fund
GROUP BY activity
ORDER BY round;
```

10. Проанализируйте, в каких странах находятся фонды, которые чаще всего инвестируют в стартапы. Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды этой страны, основанные с 2010 по 2012 год включительно. Исключите страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Выгрузите десять самых активных стран-инвесторов: отсортируйте таблицу по среднему количеству компаний от большего к меньшему. Затем добавьте сортировку по коду страны в лексикографическом порядке.

```sql
SELECT country_code,
       MIN(invested_companies),
       MAX(invested_companies),
       AVG(invested_companies)
FROM fund
WHERE EXTRACT(YEAR FROM founded_at) IN (2010,2011,2012)
GROUP BY country_code
HAVING MIN(invested_companies) != 0
ORDER BY AVG(invested_companies) DESC, country_code
LIMIT 10;
```

11. Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.

```sql
SELECT p.first_name,
       p.last_name,
       ed.instituition
FROM people AS p
LEFT JOIN education AS ed ON p.id = ed.person_id;
```

12. Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.

```sql
SELECT c.name,
       COUNT(DISTINCT instituition)
FROM company AS c
JOIN people AS p ON c.id = p.company_id
JOIN education AS ed ON p.id = ed.person_id
GROUP BY c.name
ORDER BY COUNT(DISTINCT instituition) DESC
LIMIT 5;
```

13. Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.

```sql
SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.status = 'closed' AND is_first_round = 1 AND is_last_round = 1;
```

14. Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.

```sql
WITH company_name AS (SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.status = 'closed' AND is_first_round = 1 AND is_last_round = 1)

SELECT DISTINCT p.id
FROM company_name AS t
JOIN company AS c ON t.name = c.name
JOIN people AS p ON c.id = p.company_id;
```

15. Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.

```sql
WITH company_name AS (SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.status = 'closed' AND is_first_round = 1 AND is_last_round = 1)

SELECT DISTINCT p.id,
       ed.instituition
FROM company_name AS t
JOIN company AS c ON t.name = c.name
JOIN people AS p ON c.id = p.company_id
JOIN education AS ed ON p.id = ed.person_id;
```

16. Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания. При подсчёте учитывайте, что некоторые сотрудники могли окончить одно и то же заведение дважды.

```sql
WITH company_name AS (SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.status = 'closed' AND is_first_round = 1 AND is_last_round = 1)

SELECT p.id,
       COUNT(ed.instituition)
FROM company_name AS t
JOIN company AS c ON t.name = c.name
JOIN people AS p ON c.id = p.company_id
JOIN education AS ed ON p.id = ed.person_id
GROUP BY p.id;
```

17. Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.

```sql
WITH company_name AS (SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.status = 'closed' AND is_first_round = 1 AND is_last_round = 1)

SELECT AVG(count)
FROM (SELECT p.id,
       COUNT(ed.instituition)
FROM company_name AS t
JOIN company AS c ON t.name = c.name
JOIN people AS p ON c.id = p.company_id
JOIN education AS ed ON p.id = ed.person_id
GROUP BY p.id) AS inst_number;
```

18. Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Socialnet.

```sql
WITH company_name AS (SELECT DISTINCT c.name
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE c.name = 'Socialnet')

SELECT AVG(count)
FROM (SELECT p.id,
       COUNT(ed.instituition)
FROM company_name AS t
JOIN company AS c ON t.name = c.name
JOIN people AS p ON c.id = p.company_id
JOIN education AS ed ON p.id = ed.person_id
GROUP BY p.id) AS inst_number;
```

19. Составьте таблицу из полей:
- `name_of_fund` — название фонда;
- `name_of_company` — название компании;
- `amount` — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.

```sql
SELECT f.name AS name_of_fund,
       c.name AS name_of_company,
       fr.raised_amount AS amount
FROM funding_round AS fr
JOIN investment AS i ON fr.id = i.funding_round_id
JOIN company AS c ON fr.company_id = c.id
JOIN fund AS f ON i.fund_id = f.id
WHERE EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2012 AND 2013
AND c.milestones > 6;
```

20. Выгрузите таблицу, в которой будут такие поля:
- название компании-покупателя;
- сумма сделки;
- название компании, которую купили;
- сумма инвестиций, вложенных в купленную компанию;
- доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в лексикографическом порядке. Ограничьте таблицу первыми десятью записями.

```sql
WITH

acquiring_company_tab AS (SELECT acq.id,
       acq.acquiring_company_id,
       c.name AS acquiring_company_name,
       acq.price_amount
FROM acquisition AS acq
LEFT JOIN company AS c ON acq.acquiring_company_id = c.id
WHERE acq.price_amount != 0),

acquired_company_tab AS (SELECT acq.id,
       acq.acquired_company_id,
       c.name AS acquired_company_name,
       c.funding_total
FROM acquisition AS acq
LEFT JOIN company AS c ON acq.acquired_company_id = c.id
WHERE c.funding_total != 0)

SELECT acquiring_company_name,
       price_amount,
       acquired_company_name,
       funding_total,
       ROUND(price_amount/funding_total)
FROM acquiring_company_tab AS a
JOIN acquired_company_tab AS b ON a.id=b.id
ORDER BY price_amount DESC, acquired_company_name
LIMIT 10;
```

21. Выгрузите таблицу, в которую войдут названия компаний из категории `social`, получившие финансирование с 2010 по 2013 год включительно. Проверьте, что сумма инвестиций не равна нулю. Выведите также номер месяца, в котором проходил раунд финансирования.

```sql
SELECT name,
       EXTRACT(MONTH FROM funded_at)
FROM company AS c
JOIN funding_round AS fr ON c.id = fr.company_id
WHERE raised_amount != 0
AND EXTRACT(YEAR FROM funded_at) BETWEEN 2010 AND 2013
AND category_code = 'social';
```

22. Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
- номер месяца, в котором проходили раунды;
- количество уникальных названий фондов из США, которые инвестировали в этом месяце;
- количество компаний, купленных за этот месяц;
- общая сумма сделок по покупкам в этом месяце.

```sql
WITH

usa_funds_info AS (
    SELECT
        EXTRACT(MONTH FROM fr.funded_at) AS month,
        COUNT(DISTINCT f.id) AS usa_companies_num
    FROM
        funding_round AS fr
    LEFT OUTER JOIN
        investment AS i ON fr.id = i.funding_round_id
    LEFT OUTER JOIN
        fund AS f ON i.fund_id = f.id
    WHERE
        EXTRACT(YEAR FROM fr.funded_at) BETWEEN 2010 AND 2013
        AND f.country_code = 'USA'
    GROUP BY
        EXTRACT(MONTH FROM fr.funded_at)
),

acquired_companies_info AS (
    SELECT
        EXTRACT(MONTH FROM a.acquired_at) AS month,
        COUNT(a.acquired_company_id) AS companies_num,
        SUM(a.price_amount) AS total_sum_per_month
    FROM
        acquisition AS a
    WHERE
        EXTRACT(YEAR FROM a.acquired_at) BETWEEN 2010 AND 2013
    GROUP BY
        EXTRACT(MONTH FROM a.acquired_at)
)

SELECT
    ufi.month,
    ufi.usa_companies_num,
    aci.companies_num,
    aci.total_sum_per_month
FROM
    usa_funds_info AS ufi
LEFT OUTER JOIN
    acquired_companies_info AS aci
    ON ufi.month = aci.month;
```

23. Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.

```sql
WITH

     inv_2011 AS (
        SELECT country_code,
               AVG(funding_total)
        FROM company
        WHERE EXTRACT(YEAR FROM founded_at) = 2011
        GROUP BY country_code),

     inv_2012 AS (
        SELECT country_code,
               AVG(funding_total)
        FROM company
        WHERE EXTRACT(YEAR FROM founded_at) = 2012
        GROUP BY country_code),

     inv_2013 AS (
        SELECT country_code,
               AVG(funding_total)
        FROM company
        WHERE EXTRACT(YEAR FROM founded_at) = 2013
        GROUP BY country_code)

SELECT inv_2011.country_code,
       inv_2011.avg AS avg_2011,
       inv_2012.avg AS avg_2012,
       inv_2013.avg AS avg_2013
FROM inv_2011
INNER JOIN inv_2012 ON inv_2011.country_code = inv_2012.country_code
INNER JOIN inv_2013 ON inv_2012.country_code = inv_2013.country_code
ORDER BY inv_2011.avg DESC;
```