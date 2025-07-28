# Module 21 - Exercice

<img width="597" height="743" alt="image" src="https://github.com/user-attachments/assets/56d72d73-c88f-4803-903f-b0622e66ffd2" />


The questions proposed for this exercise were two.

### 1) Calculate the average by actor's name and surname of the following variables:
- rental_duration
- rental_rate
- length
- replacement_cost


### 2) Calculate the sum of amount (total rental price) by name, surname and email of the customer (customer table) and indicate the:
- 10 customers who spent the most
- 10 customers who spent the least
----

### Importing packs for work in Jupyter Notebook
```python
import psycopg2 as pg2
import pandas as pd
```
----
### Configuring DEFs for:
1) connect to PostgreSQL
2) run the query
3) transform the result into DataFrame

```python
def conectar_postgres(database):
    conn = None
    try:
        print('Connected to PostgreSQL.')
        conn = pg2.connect(host='localhost',
                           port=your_port,
                           dbname=database,
                           user='postgres',
                           password='your_password')
        print('Success in connection.')

    except Exception as e:
        print(e)
        conn = None
    return conn

def resultados_postgres(conn, query, fetch = 'all'):
    colnames = None
    data = None

    try:
        print('Cursor created')
        cur = conn.cursor()

        print('Processing the query')
        cur.execute(query)
        print('Query executed successfully')
        conn.commit()

        if fetch == 'all':
            data = cur.fetchall()
        elif fetch == 'many':
            data = cur.fetchmany()
        elif fetch == 'one':
            data = cur.fetchone()
        else:
            return None

        nome_colunas = [desc.name for desc in cur.description]
        cur.close()
        conn.close()
    except Exception as e:
        print(e)
        cur.close()
        conn.close()
    return data, nome_colunas    

def to_df(data, nome_colunas):
    print('Transforming in DataFrame')
    df = pd.DataFrame(data, columns=nome_colunas)
    print('DataFrame created')
    return df
```
----
### Question 1

I wrote the query in pgAdmin using:
- *subqueries*
- functions *round*, *avg*, *to_char*, *group by* and *order by*
  
Then inserted it into the Python code.

#### SQL
```sql
select 
	first_name as nome,
	last_name as sobrenome,
	round(avg(rental_duration), 2) as duracao_aluguel_dias,
	'US$ ' || to_char(round(avg(rental_rate), 2), 'FM999,990.00') as taxa_aluguel,
	round(avg(length), 2) as duracao_minutos,
	'US$ ' || to_char(round(avg(replacement_cost), 2), 'FM999,990.00') as custo_reposicao
from (
	select *
	from actor as a
		left join film_actor as fa
			on a.actor_id = fa.actor_id
		left join film as f
			on fa.film_id = f.film_id
) as a
group by first_name, last_name
order by first_name;
```
#### Python
```python
database = 'dvdrental'
query = '''select 
        	first_name as nome,
        	last_name as sobrenome,
        	round(avg(rental_duration), 2) as duracao_aluguel_dias,
        	'US$ ' || to_char(round(avg(rental_rate), 2), 'FM999,990.00') as taxa_aluguel,
        	round(avg(length), 2) as duracao_minutos,
        	'US$ ' || to_char(round(avg(replacement_cost), 2), 'FM999,990.00') as custo_reposicao
        from (
        	select *
        	from actor as a
        		left join film_actor as fa
        			on a.actor_id = fa.actor_id
        		left join film as f
        			on fa.film_id = f.film_id
        ) as a
        group by first_name, last_name
        order by first_name;'''
conn = conectar_postgres(database)
data, nome_colunas = resultados_postgres(conn, query, fetch = 'all')
df = to_df(data, nome_colunas)

df
```
----
### Question 2

I wrote the query in pgAdmin using:
- *subqueries*
- functions *to_char*, *group by*, *order by* and *limit*
  
Then inserted it into the Python code.

2.1) 10 customers who spent the most
### SQL
```sql
select
	first_name AS nome,
	last_name AS sobrenome,
	email AS e_mail,
	'US$ ' || to_char(round(SUM(amount),2), 'FM999,990.00') as preco_total
from(
	select *
	from customer as c
		left join payment as pay
			on c.customer_id = pay.customer_id
) as c
group by first_name, last_name, email
order by sum(amount) DESC
limit 10;
```
### Python
```python
database = 'dvdrental'
query = '''select
            	first_name AS nome,
            	last_name AS sobrenome,
            	email AS e_mail,
            	'US$ ' || to_char(round(SUM(amount),2), 'FM999,990.00') as preco_total
            from(
            	select *
            	from customer as c
            		left join payment as pay
            			on c.customer_id = pay.customer_id
            ) as c
            group by first_name, last_name, email
            order by sum(amount) DESC
            limit 10;'''
conn = conectar_postgres(database)
data, nome_colunas = resultados_postgres(conn, query, fetch = 'all')
df_2_1 = to_df(data, nome_colunas)

df_2_1
```

2.2) 10 customers who spent the least
### SQL
```sql
select
	first_name AS nome,
	last_name AS sobrenome,
	email AS e_mail,
	'US$ ' || to_char(round(SUM(amount),2), 'FM999,990.00') as preco_total
from(
	select *
	from customer as c
		left join payment as pay
			on c.customer_id = pay.customer_id
) as c
group by first_name, last_name, email
order by sum(amount) ASC
limit 10;
```
### Python
```python
database = 'dvdrental'
query = '''select
            	first_name AS nome,
            	last_name AS sobrenome,
            	email AS e_mail,
            	'US$ ' || to_char(round(SUM(amount),2), 'FM999,990.00') as preco_total
            from(
            	select *
            	from customer as c
            		left join payment as pay
            			on c.customer_id = pay.customer_id
            ) as c
            group by first_name, last_name, email
            order by sum(amount) ASC
            limit 10;'''
conn = conectar_postgres(database)
data, nome_colunas = resultados_postgres(conn, query, fetch = 'all')
df_2_2 = to_df(data, nome_colunas)

df_2_2
```
