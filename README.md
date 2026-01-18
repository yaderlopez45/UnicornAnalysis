
Did you know that the average return from investing in stocks is [10% per year](https://www.nerdwallet.com/article/investing/average-stock-market-return) (not accounting for inflation)? But who wants to be average?! 

You have been asked to support an investment firm by analyzing trends in high-growth companies. They are interested in understanding which industries are producing the highest valuations and the rate at which new high-value companies are emerging. Providing them with this information gives them a competitive insight as to industry trends and how they should structure their portfolio looking forward.

You have been given access to their `unicorns` database, which contains the following tables:

## dates
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `date_joined` | The date that the company became a unicorn.  |
| `year_founded` | The year that the company was founded.       |

## funding
| Column           | Description                                  |
|----------------- |--------------------------------------------- |
| `company_id`       | A unique ID for the company.                 |
| `valuation`        | Company value in US dollars.                 |
| `funding`          | The amount of funding raised in US dollars.  |
| `select_investors` | A list of key investors in the company.      |

## industries
| Column       | Description                                  |
|------------- |--------------------------------------------- |
| `company_id`   | A unique ID for the company.                 |
| `industry`     | The industry that the company operates in.   |

## companies
| Column       | Description                                       |
|------------- |-------------------------------------------------- |
| `company_id`   | A unique ID for the company.                      |
| `company`      | The name of the company.                          |
| `city`         | The city where the company is headquartered.      |
| `country`      | The country where the company is headquartered.   |
| `continent`    | The continent where the company is headquartered. |


# The output

Your query should return a table in the following format:
| industry  | year | num\_unicorns       | average\_valuation\_billions |
| --------- | ---- | ------------------- | ---------------------------- |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |
| industry1 | 2021 |        ---          |             ---              |
| industry2 | 2020 |        ---          |             ---              |
| industry3 | 2019 |        ---          |             ---              |

Where `industry1`, `industry2`, and `industry3` are the three top-performing industries.

# **Identifying the top unicorn-producing industries**

## **Goal**

Determine which industries created the most unicorns between 2019 and 2021. This establishes where the highest concentration of growth is happening.

## **Method**

I count how many companies reached unicorn status in each industry by extracting the year they became unicorns from the date_joined field. I group the results by industry and year to calculate a total number of new unicorns per year. After ranking industries by the size of their counts, I isolate the three industries with the most unicorns.

## **Result**

The three top-performing industries in 2021 are:
- Fintech with 138 new unicorns
- Internet software and services with 119
- E-commerce and direct-to-consumer with 47

Fintech leads by a wide margin, showing how quickly capital is flowing into digital finance. Internet software and online commerce follow, driven by rapid adoption of digital services and business model innovation during the 2020–2021 period.
These industries represent the strongest growth clusters and will be the focus for deeper valuation analysis in the next steps.


```python
WITH top_industries AS (
	Select 
		i.industry as industry, 
		EXTRACT(YEAR FROM d.date_joined) as year, 
		COUNT(i.company_id) as num_unicorns
	From public.industries as i
		Join public.dates as d
		 	on i.company_id = d.company_id
	Where EXTRACT(YEAR FROM d.date_joined) IN (2019,2020,2021)
	GROUP BY industry, year
	ORDER BY num_unicorns DESC
)

Select *
From top_industries
	LIMIT 3;
```




<div>


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>industry</th>
      <th>year</th>
      <th>num_unicorns</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Fintech</td>
      <td>2021</td>
      <td>138</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Internet software &amp; services</td>
      <td>2021</td>
      <td>119</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2021</td>
      <td>47</td>
    </tr>
  </tbody>
</table>
</div>



# **Ranking unicorn creation and valuation by industry**

## **Goal**
Measure yearly performance of each industry by counting how many new unicorns were created and calculating the average valuation of those companies. This provides a combined view of both volume and financial strength.

## **Method**
The year each company became a unicorn is extracted from the date_joined field. Industry-level results are grouped by year to calculate two metrics: the number of new unicorns and the average valuation of those companies. The results are ordered by the count of new unicorns, and valuation is used as a secondary ranking factor when counts are similar.

## **Result**
The highest-performing industry in 2021 is:
- Fintech with 138 new unicorns and an average valuation of about 2.75 billion dollars

The next strongest industries in 2021 are:
- Internet software and services with 119 new unicorns and an average valuation of about 2.15 billion dollars
- E-commerce and direct-to-consumer with 47 new unicorns and an average valuation of about 2.47 billion dollars
- Health with 40 new unicorns and an average valuation of about 1.95 billion dollars
- Artificial intelligence with 36 new unicorns and an average valuation of about 1.42 billion dollars

Fintech dominates in both volume and valuation, indicating a major concentration of investment in digital financial services. Internet software remains the second strongest cluster, supported by large-scale platform businesses. E-commerce and health show fewer total unicorns but higher valuations, signaling deeper capital intensity. These patterns guide the choice of the top three industries that will be used for trend analysis in later steps.



```python
WITH yearly_ranking AS (
	Select 
		i.industry as industry, 
		EXTRACT(YEAR FROM d.date_joined) as year, 
		COUNT(i.company_id) as num_unicorns,
		AVG(f.valuation) as average_valuation
	From public.industries as i
		Join public.dates as d
		 	on i.company_id = d.company_id
		Join public.funding as f
			on f.company_id = i.company_id
	Group by industry, year
)

Select *
From yearly_ranking
WHERE Year IN (2019, 2020, 2021)
Order by num_unicorns DESC, average_valuation DESC
```




<div>



<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>industry</th>
      <th>year</th>
      <th>num_unicorns</th>
      <th>average_valuation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Fintech</td>
      <td>2021</td>
      <td>138</td>
      <td>2.753623e+09</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Internet software &amp; services</td>
      <td>2021</td>
      <td>119</td>
      <td>2.151261e+09</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2021</td>
      <td>47</td>
      <td>2.468085e+09</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Health</td>
      <td>2021</td>
      <td>40</td>
      <td>1.950000e+09</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Artificial intelligence</td>
      <td>2021</td>
      <td>36</td>
      <td>1.416667e+09</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Cybersecurity</td>
      <td>2021</td>
      <td>27</td>
      <td>2.518519e+09</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Supply chain, logistics, &amp; delivery</td>
      <td>2021</td>
      <td>25</td>
      <td>2.200000e+09</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Data management &amp; analytics</td>
      <td>2021</td>
      <td>21</td>
      <td>2.142857e+09</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Other</td>
      <td>2021</td>
      <td>21</td>
      <td>1.714286e+09</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Fintech</td>
      <td>2019</td>
      <td>20</td>
      <td>6.800000e+09</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Internet software &amp; services</td>
      <td>2020</td>
      <td>20</td>
      <td>4.350000e+09</td>
    </tr>
    <tr>
      <th>11</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2020</td>
      <td>16</td>
      <td>4.000000e+09</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Fintech</td>
      <td>2020</td>
      <td>15</td>
      <td>4.333333e+09</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Artificial intelligence</td>
      <td>2019</td>
      <td>14</td>
      <td>4.500000e+09</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Hardware</td>
      <td>2021</td>
      <td>14</td>
      <td>2.000000e+09</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Internet software &amp; services</td>
      <td>2019</td>
      <td>13</td>
      <td>4.230769e+09</td>
    </tr>
    <tr>
      <th>16</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2019</td>
      <td>12</td>
      <td>2.583333e+09</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Edtech</td>
      <td>2021</td>
      <td>12</td>
      <td>2.000000e+09</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Other</td>
      <td>2020</td>
      <td>11</td>
      <td>3.000000e+09</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Health</td>
      <td>2020</td>
      <td>9</td>
      <td>3.111111e+09</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Other</td>
      <td>2019</td>
      <td>9</td>
      <td>2.888889e+09</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Mobile &amp; telecommunications</td>
      <td>2020</td>
      <td>8</td>
      <td>3.125000e+09</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Supply chain, logistics, &amp; delivery</td>
      <td>2019</td>
      <td>8</td>
      <td>3.000000e+09</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Cybersecurity</td>
      <td>2020</td>
      <td>7</td>
      <td>3.285714e+09</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Consumer &amp; retail</td>
      <td>2021</td>
      <td>7</td>
      <td>2.571429e+09</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Auto &amp; transportation</td>
      <td>2019</td>
      <td>6</td>
      <td>4.166667e+09</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Data management &amp; analytics</td>
      <td>2020</td>
      <td>6</td>
      <td>3.000000e+09</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Mobile &amp; telecommunications</td>
      <td>2021</td>
      <td>6</td>
      <td>1.833333e+09</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Auto &amp; transportation</td>
      <td>2020</td>
      <td>5</td>
      <td>3.000000e+09</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Data management &amp; analytics</td>
      <td>2019</td>
      <td>4</td>
      <td>1.150000e+10</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Auto &amp; transportation</td>
      <td>2021</td>
      <td>4</td>
      <td>3.750000e+09</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Edtech</td>
      <td>2020</td>
      <td>4</td>
      <td>2.750000e+09</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Cybersecurity</td>
      <td>2019</td>
      <td>4</td>
      <td>2.250000e+09</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Mobile &amp; telecommunications</td>
      <td>2019</td>
      <td>4</td>
      <td>2.000000e+09</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Artificial intelligence</td>
      <td>2020</td>
      <td>3</td>
      <td>4.000000e+09</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Travel</td>
      <td>2019</td>
      <td>3</td>
      <td>4.000000e+09</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Consumer &amp; retail</td>
      <td>2019</td>
      <td>3</td>
      <td>3.666667e+09</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Health</td>
      <td>2019</td>
      <td>3</td>
      <td>3.333333e+09</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Travel</td>
      <td>2021</td>
      <td>3</td>
      <td>2.666667e+09</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Supply chain, logistics, &amp; delivery</td>
      <td>2020</td>
      <td>2</td>
      <td>2.000000e+09</td>
    </tr>
    <tr>
      <th>40</th>
      <td>Consumer &amp; retail</td>
      <td>2020</td>
      <td>1</td>
      <td>1.500000e+10</td>
    </tr>
    <tr>
      <th>41</th>
      <td>Hardware</td>
      <td>2020</td>
      <td>1</td>
      <td>2.000000e+09</td>
    </tr>
    <tr>
      <th>42</th>
      <td>Edtech</td>
      <td>2019</td>
      <td>1</td>
      <td>1.000000e+09</td>
    </tr>
  </tbody>
</table>
</div>



# **Comparing yearly trends in unicorn creation and valuation**

## **Goal**
Analyze how the top three industries performed over time by showing how many unicorns each industry produced in 2019, 2020, and 2021, and comparing their average valuations in billions of dollars.

## **Method**
The top three industries are selected based on total unicorn creation across the full period. The yearly results are filtered to include only these industries and only the years 2019 to 2021. The number of new unicorns is counted for each industry and year, and the average valuation of those companies is converted into billions for easier comparison. The results are ordered by year so that changes can be observed over time.

## **Result**
Across the three-year period, the strongest industry by volume is:
- Fintech with a peak of 138 new unicorns in 2021 and an average valuation of about 2.75 billion dollars

Internet software and services shows consistent high growth with:
- 119 new unicorns in 2021 and valuations around 2.15 billion dollars

E-commerce and direct-to-consumer has fewer total unicorns but higher valuations, including:
- 47 new unicorns in 2021 and an average valuation of about 2.47 billion dollars

The results show that 2021 was the breakout year for all three industries. Fintech led in both scale and investment intensity, supported by rapid digital adoption. Internet software followed a stable upward path through all three years. E-commerce grew sharply in 2021 but showed the highest valuations in earlier years, indicating deeper capital deployment during the earlier pandemic cycle.



```python
WITH top_industries AS (
	Select 
		i.industry as industry, 
		EXTRACT(YEAR FROM d.date_joined) as year, 
		COUNT(i.company_id) as num_unicorns
	From public.industries as i
		Join public.dates as d
		 	on i.company_id = d.company_id
	GROUP BY industry, year
	ORDER BY num_unicorns DESC
),

yearly_ranking AS (
	Select 
		i.industry as industry, 
		EXTRACT(YEAR FROM d.date_joined) as year, 
		COUNT(i.company_id) as num_unicorns,
		AVG(f.valuation) as average_valuation
	From public.industries as i
		Join public.dates as d
		 	on i.company_id = d.company_id
		Join public.funding as f
			on f.company_id = i.company_id
	Group by industry, year
)

Select industry, 
	year, 
	num_unicorns, 
	ROUND(average_valuation/1000000000,2) as average_valuation_billions
From yearly_ranking
Where year IN (2019,2020,2021)
AND industry IN (
	Select industry
From top_industries
	LIMIT 3
)
Order by year DESC, num_unicorns DESC;

```




<div>


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>industry</th>
      <th>year</th>
      <th>num_unicorns</th>
      <th>average_valuation_billions</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Fintech</td>
      <td>2021</td>
      <td>138</td>
      <td>2.75</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Internet software &amp; services</td>
      <td>2021</td>
      <td>119</td>
      <td>2.15</td>
    </tr>
    <tr>
      <th>2</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2021</td>
      <td>47</td>
      <td>2.47</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Internet software &amp; services</td>
      <td>2020</td>
      <td>20</td>
      <td>4.35</td>
    </tr>
    <tr>
      <th>4</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2020</td>
      <td>16</td>
      <td>4.00</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Fintech</td>
      <td>2020</td>
      <td>15</td>
      <td>4.33</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Fintech</td>
      <td>2019</td>
      <td>20</td>
      <td>6.80</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Internet software &amp; services</td>
      <td>2019</td>
      <td>13</td>
      <td>4.23</td>
    </tr>
    <tr>
      <th>8</th>
      <td>E-commerce &amp; direct-to-consumer</td>
      <td>2019</td>
      <td>12</td>
      <td>2.58</td>
    </tr>
  </tbody>
</table>
</div>


