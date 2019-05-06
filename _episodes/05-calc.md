---
title: "Calculating New Values"
teaching: 5
exercises: 5
questions:
- "How can I calculate new values on the fly?"
objectives:
- "Write queries that calculate new values for each selected record."
keypoints:
- "Queries can do the usual arithmetic operations on values."
- "Use UNION to combine the results of two or more queries."
---
After carefully re-reading the expedition logs,
we realize that the radiation measurements they report
may need to be corrected upward by 5%.
Rather than modifying the stored data,
we can do this calculation on the fly
as part of our query:

~~~
SELECT 1.05 * reading FROM Survey WHERE quant = 'rad';
~~~
{: .sql}

|1.05 * reading|
|--------------|
|10.311        |
|8.19          |
|8.8305        |
|7.581         |
|4.5675        |
|2.2995        |
|1.533         |
|11.8125       |

When we run the query,
the expression `1.05 * reading` is evaluated for each row.
Expressions can use any of the fields,
all of usual arithmetic operators,
and a variety of common functions, 
depending on which database system is being used.
For example,
we can convert temperature readings from Fahrenheit to Celsius
and round to two decimal places:

~~~
SELECT taken, ROUND(5 * (reading - 32) / 9, 2) FROM Survey WHERE quant = 'temp';
~~~
{: .sql}

|taken|ROUND(5*(reading-32)/9, 2)|
|-----|--------------------------|
|734  |-29.72                    |
|735  |-32.22                    |
|751  |-28.06                    |
|752  |-26.67                    |

As you can see from this example, though, the string describing our
new field (generated from the equation) can become quite unwieldy. SQL
allows us to rename our fields, any field for that matter, whether it
was calculated or one of the existing fields in our database, for
succinctness and clarity. For example, we could write the previous
query as:

~~~
SELECT taken, ROUND(5 * (reading - 32) / 9, 2) AS Celsius FROM Survey WHERE quant = 'temp';
~~~
{: .sql}

|taken|Celsius|
|-----|-------|
|734  |-29.72 |
|735  |-32.22 |
|751  |-28.06 |
|752  |-26.67 |

We can also combine values from different fields,
for example by using the string concatenation operator `||`:

~~~
SELECT personal || ' ' || family FROM Person;
~~~
{: .sql}

|personal || ' ' || family|
|-------------------------|
|William Dyer             |
|Frank Pabodie            |
|Anderson Lake            |
|Valentina Roerich        |
|Frank Danforth           |

> ## Fixing Salinity Readings
>
> After further reading,
> we realize that Valentina Roerich
> was reporting salinity as percentages.
> Write a query that returns all of her salinity measurements
> from the `Survey` table
> with the values divided by 100.
> 
> > ## Solution
> >
> > ~~~
> > SELECT taken, reading / 100 FROM Survey WHERE person = 'roe' AND quant = 'sal';
> > ~~~
> > {: .sql}
> >
> > |taken     |reading / 100|
> > |----------|-------------|
> > |752       |0.416        |
> > |837       |0.225        |
> {: .solution}
{: .challenge}

> ## Unions
>
> The `UNION` operator combines the results of two queries, and does a `SELECT DISTINCT` on the results set:
>
> ~~~
> SELECT * FROM Person WHERE id = 'dyer' UNION SELECT * FROM Person WHERE id = 'roe';
> ~~~
> {: .sql}
>
> |id  |personal |family |
> |----|-------- |-------|
> |dyer|William  |Dyer   |
> |roe |Valentina|Roerich|
>
> The `UNION ALL` command is similiar to `UNION`. 
> The difference is that `UNION ALL` will return all rows from all queries, and will not eliminate duplicate rows.  For example:
>
> ~~~
> SELECT * FROM Person WHERE id IN ('dyer', 'roe') 
> UNION ALL 
> SELECT * FROM Person WHERE id IN ('dyer', 'roe');
> ~~~
> {: .sql}
>
> |id  |personal |family |
> |----|-------- |-------|
> |dyer|William  |Dyer   |
> |roe |Valentina|Roerich|
> |dyer|William  |Dyer   |
> |roe |Valentina|Roerich|
>
> And if you change the `UNION ALL` to `UNION` you will get the following:
>
> |id  |personal |family |
> |----|-------- |-------|
> |dyer|William  |Dyer   |
> |roe |Valentina|Roerich|
>
> If you know that all the records to be returned from your union are unique,
> use `UNION ALL` instead, it gives faster results since it skips the `DISTINCT` step.
> For this section, we shall use UNION.
>
> Use `UNION` to create a consolidated list of salinity measurements
> in which Valentina Roerich's, and only Valentina's,
> have been corrected as described in the previous challenge.
> The output should be something like:
>
> |taken|reading|
> |-----|-------|
> |619  |0.13   |
> |622  |0.09   |
> |734  |0.05   |
> |751  |0.1    |
> |752  |0.09   |
> |752  |0.416  |
> |837  |0.21   |
> |837  |0.225  |
> 
> > ## Solution
> >
> > ~~~
> > SELECT taken, reading 
> >  FROM  Survey 
> >  WHERE person != 'roe' AND quant = 'sal' 
> > UNION 
> > SELECT taken, reading / 100 
> >  FROM  Survey 
> >  WHERE person = 'roe' AND quant = 'sal' ORDER BY taken ASC;
> > ~~~
> > {: .sql}
> {: .solution}
{: .challenge}

> ## Selecting Major Site Identifiers
>
> The site identifiers in the `Visit` table have two parts
> separated by a '-':
>
> ~~~
> SELECT DISTINCT site FROM Visit;
> ~~~
> {: .sql}
>
> |site |
> |-----|
> |DR-1 |
> |DR-3 |
> |MSK-4|
>
> In the Site table, some of the major site identifiers (i.e. the letter codes) are two letters long and some are three.
> If we wanted to identify only the unique sites (ie, the unique letter codes), how could we achieve this?
> 
> The "in string" function `instr(X, Y)`
> returns the position of the first occurrence of string Y in string X, using a 1-based index,
> or 0 if Y does not exist in X.  For example, the following will show the position of the '-' character in site:
> ~~~
> SELECT site, instr(site, '-') FROM Visit;
> ~~~
> {: .sql}
> 
> The substring function `substr(X, I, [L])`
> returns the substring of X starting at index I, with an optional length L.
>
> Use these two functions to produce a list of unique major site identifiers.
> (For this data, the list should contain only "DR" and "MSK").
>
> > ## Solution
> > ```
> > SELECT DISTINCT substr(site, 1, instr(site, '-') - 1) AS MajorSite FROM Visit;
> > ```
> > {: .sql}
> {: .solution}
{: .challenge}
