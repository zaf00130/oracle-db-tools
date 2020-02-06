```
WITH unique_person_id AS ( /* the combination of person_id with max effective_start_date gives the current assignment, I suppose */
SELECT b.person_id, MAX(b.effective_start_date) effective_start_date
FROM apps.per_all_assignments_f b
GROUP BY b.person_id),
unique_people_person AS (
SELECT person_id, MAX(effective_start_date) effective_start_date
FROM apps.per_all_people_f i
GROUP BY person_id),
current_person AS (
SELECT k.*
FROM unique_people_person j
LEFT JOIN apps.per_all_people_f k ON (j.person_id = k.person_id AND j.effective_start_date = k.effective_start_date)),
current_assignment AS (
SELECT d.*
FROM unique_person_id c
LEFT JOIN apps.per_all_assignments_f d ON (c.person_id = d.person_id AND c.effective_start_date = d.effective_start_date)
WHERE d.effective_end_date > sysdate)
SELECT
level, lpad(' ',level*6) || l.full_name, substr(sys_connect_by_path(l.full_name,' -> '),5)
FROM current_assignment e
LEFT JOIN current_person l ON (e.person_id = l.person_id)
START WITH e.person_id = 2134 /* enter the person_id of the starting supervisor (such as Captain/President/CEO) but could be an intermediate position */
CONNECT BY e.supervisor_id = PRIOR e.person_id
ORDER SIBLINGS BY l.full_name
;
```
Note: below query results may not be historically accurate!

| LEVEL | LPAD('',LEVEL*6)&#124;&#124;L.FULL_NAME | SUBSTR(SYS_CONNECT_BY_PATH(L.FULL_NAME,'->'),5)           |
|-------|-------------------------------|-----------------------------------------------------------|
| 1     | Picard, Jean-Luc              | Picard, Jean-Luc                                          |
| 2     | Crusher, Beverly              | Picard, Jean-Luc -> Crusher, Beverly                      |
| 2     | Data                          | Picard, Jean-Luc -> Data                                  |
| 2     | La Forge, Geordi              | Picard, Jean-Luc -> La Forge, Geordi                      |
| 3     | Barclay, Reginald             | Picard, Jean-Luc -> La Forge, Geordi -> Barclay, Reginald |
| 3     | Crusher, Wesley               | Picard, Jean-Luc -> La Forge, Geordi -> Crusher, Wesley   |




The above query will reproduce the employee hierarchy within EBS (as organized in the HR tables).  
I wrote this SQL query mostly for entertainment value, although it demonstrates use of the  
`WITH` clause (underused; in my humble opinion it increases readability vs. using complex sub-queries)  
as well as the `CONNECT BY` statement.  

In the above example query results, Jean-Luc Picard is at the top level (LEVEL=1),  
there are three people (LEVEL=2) reporting directly to Picard, namely Beverly Crusher, Data, and Geordi La Forge,  
and there are two people reporting to La Forge (LEVEL=3), Reginald Barclay and Wesley Crusher. 

As I recall, the tricky part about writing this query was to realize that in  
`PER_ALL_ASSIGNMENTS_F` an employee could have multiple assignments throughout their  
career, but we are generally interested in the current assignment `(max(effective_start_date))`.
