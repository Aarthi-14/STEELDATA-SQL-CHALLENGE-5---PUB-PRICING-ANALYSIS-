# PUB PRICING ANALYSIS 

## Objective:
As a Pricing Analyst, working for a pub chain called 'Pubs "R" Us',I have been tasked with analysing the drinks prices and sales to gain a greater insight into how the pubs in your chain are performing.

## Source:
SteelData SQL Challenge-5 Link: https://www.steeldata.org.uk/sql5.html

## Exploratory Data Analysis:
1. How many pubs are located in each country??
```sql
SELECT Country, 
	   COUNT(*) as no_of_pubs
FROM pubs
group by country;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/ce279665-a471-4c9c-85b2-2a9f45aa521c)
Insights/Findings:
* UK, Ireland, US, spain are the countries that all has each pub in their country.
  
2. What is the total sales amount for each pub, including the beverage price and quantity sold?
```sql
with cte as (
			SELECT p.*,
			round((b.price_per_unit * s.quantity),2) as total_sales
			FROM beverages b
			join sales s using (beverage_id)
			join pubs p using (pub_id)
)
select pub_id as Pub_id, 
	   pub_name as Pub_name,
sum(total_sales) as total_sales_amount
from cte
group by pub_id;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/9cfaf297-23b0-4811-a26f-590055662bdc)
Insights/Findings:
* The Red Lion Pub has made the Highest Total sales of $532.66 & the Dubliner has made Lowest Sales of $308.
  
3. Which pub has the highest average rating?
```sql
select pub_id, 
	   pub_name, 
       round(avg(rating),2) as average_rating
from ratings r
join pubs p using (pub_id)
group by pub_id
order by pub_id,avg(rating) desc
limit 1;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/7cbf571d-dbba-4b89-a3e1-207e2f0fa018)
Insights/Findings:
* The Red Lion Pub has made the Highest Average Rating of 4.67

4. What are the top 5 beverages by sales quantity across all pubs?
```sql
select distinct beverage_id,
	   beverage_name,
       sum(quantity) over (partition by beverage_id) as sales_quantity
from sales s
join beverages b using (beverage_id)
order by sales_quantity desc, beverage_id
limit 5;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/b880571b-1794-485d-8e09-6357c77c73fb)
Insights/Findings:
* Guinness, Mojito, Chardonnay, Tequila & IPA are the Top Beverages in all Pubs.
  
5. How many sales transactions occurred on each date?
```sql
select  transaction_date,
		count(*) as sales_transactions
from sales
group by transaction_date
order by transaction_date asc;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/6de676d9-0361-4752-8218-4ec84c3a0f63)
Insights/Findings:
* Maximum Sales Transactions occured is 5 on 09-05-2023.
  
6. Find the name of someone that had cocktails and which pub they had it in.
```sql
WITH CocktailCTE AS (
    SELECT r.customer_name, r.pub_id, p.pub_name
    FROM ratings r
    JOIN sales s USING (pub_id)
    JOIN beverages b USING (beverage_id)
    JOIN pubs p USING (pub_id)
    WHERE b.category = 'Cocktail'
)
SELECT customer_name, pub_id, pub_name
FROM CocktailCTE;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/446fe683-a376-45d6-b733-250f00070740)

7. What is the average price per unit for each category of beverages,
--excluding the category 'Spirit'?
```sql
select category,
       round(avg(price_per_unit),2) as average_price_per_unit
       from beverages
       where category <> "Spirit"
       group by category;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/25646b7c-c78c-4982-8d01-d333f3bcd00a)
Insights/Findings:
* The average price per unit of Beer category is 5.49, cocktail of 8.99, Whiskey of 29.99 & Wine of 12.99 

8. Which pubs have a rating higher than the average rating of all pubs?
```sql
select pub_id,
	   pub_name, 
       round(avg(rating),2) as pub_rating
from ratings r 
join pubs p using (pub_id)
group by pub_id
having pub_rating > (select round(avg(rating),2) as avg_rating from ratings);
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/9ea8b615-44b1-4a62-979b-7d40d601ee23)
Insights/Findings:
* The Red Lion Pub & La Cerveceria has the rating higher than the average rating of all pubs.
  
9. What is the running total of sales amount for each pub, ordered by the transaction date?
```sql
select pub_id,
	   pub_name,
       transaction_date,
       (s.quantity * b.price_per_unit) as sales_amount,
       sum(s.quantity * b.price_per_unit) over(partition by pub_id order by transaction_date) as running_total
from sales s
join beverages b using (beverage_id)
join pubs p using (pub_id)
order by pub_id, transaction_date asc;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/8f6d5c57-0562-4814-b688-fce7b5e0c023)

10. For each country, what is the average price per unit of beverages in each category, and what is the overall average price per unit of beverages 
across all categories?
```sql
with cte1 as (
	select country, category,
		   round(avg(price_per_unit),2) as avg_price_per_unit
	from sales s
	join pubs p using (pub_id)
	join beverages b using (beverage_id)
	group by country,category),
	cte2 as (
	select country,
		   round(avg(price_per_unit),2) as overall_average_of_all_categories
	from sales s
	join pubs p using (pub_id)
	join beverages b using (beverage_id)
	group by country
)select cte2.country,
		cte1.category,
        cte1.avg_price_per_unit,
        cte2.overall_average_of_all_categories
from cte1
join cte2 on cte1.country = cte2.country
order by country,category;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/2b3e56b8-9544-45a4-ab2d-6549bc1f4994)

11. For each pub, what is the percentage contribution of each category of 
beverages to the total sales amount, and what is the pubs overall sales amount?
```sql
with overall_sales as (
	select pub_id, pub_name,
		   sum(b.price_per_unit * s.quantity) as overall_sales_amount
	from sales s 
	join beverages b using (beverage_id)
	join pubs p using (pub_id)
	group by pub_id),
total_sales as (
   select pub_id, category,
		  sum(b.price_per_unit * s.quantity) as total_sales_by_cat
   from sales s 
   join beverages b using (beverage_id)
   join pubs p using (pub_id)
   group by pub_id,category
) select t.pub_id, o.pub_name, t.category, t.total_sales_by_cat, o.overall_sales_amount,
	round((t.total_sales_by_cat/o.overall_sales_amount)*100,2) as percentage_contribution
 from overall_sales o
 join total_sales t using (pub_id)
 order by pub_id;
```
![image](https://github.com/Aarthi-14/STEELDATA-SQL-CHALLENGE-5---PUB-PRICING-ANALYSIS-/assets/147639053/0829264e-06a2-449c-b2fd-245f0d325619)
Insights/Findings:
* Beer category of the Dubliner Pub has contributed the highest Percentage of 53.37% among all other Beverages.

## Key Insights:
1.The Red Lion Pub located in London, has the highest average rating (4.67) of all pubs with highest Total Sales Amount of 532.66$
2.The Dubliner Pub in Ireland, has the second highest average rating (4.6) of all Pubs yet has the Lowest Total sales Amount of 308.62$.
3.Among the Top 5 beverages across all Pubs, Guinness Beverage under Beer category ranked No. 1 and Mojito Beverage under Cocktail category ranked No.2 based on sales quantity.
4.In Dubliner Pub, 53.37 % of total sales was contributed the Beverages under Beer category, which is the highest among % percentage contribution among all Pubs.

