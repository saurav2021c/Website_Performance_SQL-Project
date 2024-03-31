# Advanced SQL Project: `Traffic Sources` & `Website Performance`
This is a SQL Project which uses Advanced SQL concepts for Analysis. This Project focuses on gathering insights from a database and storytelling is done by me in this Project.
## Topic Covered in this Project:
| Sr no.       |  Used                              | Sr no.       |  Used                                      |
| ------------- |:-----------------------------------:| ------------- |:-----------------------------------:|
| `1`           |  *Finding Top Traffic Sources*      | `5`           |  *Traffic Source Segment Trending*      |
| `2`           |  *Traffic Conversion Rates*         | `6`           |  *Identifying Top Website Pages*        |
| `3`           |  *Traffic Source Trending*          | `7`           |  *Building Conversion Funnels*          |
| `4`           |  *Traffic Source Bid Optimization*  | `8`           |  *Analyzing Conversion Funnel Tests*    |




## Let's understand the database first:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/cafbcbee-532d-4bbf-b454-0b7f375f6e76)
- we will be focusing on these tables only in the database.
- In this, Analysis There will 8 Question to be Answered by me as a **Business Analyst** to the **Website Manager** (Lavkush Yadav).
- This Project will be like a conversation in which I am solving issues and finding insights from the Tables of Database.

## `Question 1:`
### Gsearch seems to be the biggest driver of our business. Could you pull monthly trends for gsearch sessions and orders so that we can showcase the growth there? 
### Analysis:
```
SELECT
    YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions, 
    COUNT(DISTINCT orders.order_id) AS orders, 
    COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conv_rate
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/83a5922e-1aec-43c3-b8cb-1ec962680e88)

### Conclusion:
- Here we got year, month and sessions volumes are growing by month and orders with conversion rates is growing as well substantially. This is nice improvement for a business. 

## `Question 2:`
### Next, it would be great to see a similar monthly trend for Gsearch, but this time splitting out nonbrand and brand campaigns separately. I am wondering if brand is picking up at all. If so, this is a good story to tell. 
### Analysis:
```
SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN website_sessions.website_session_id ELSE NULL END) AS nonbrand_sessions, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'nonbrand' THEN orders.order_id ELSE NULL END) AS nonbrand_orders,
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN website_sessions.website_session_id ELSE NULL END) AS brand_sessions, 
    COUNT(DISTINCT CASE WHEN utm_campaign = 'brand' THEN orders.order_id ELSE NULL END) AS brand_orders
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
GROUP BY 1,2;
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/808d72b5-61a0-40df-90d1-e4311b4e17b7)

### Conclusion:
- Brand campaign is something like someone going to search engine and searching for our business.
- But in this analysis it can be seen that most sessions are from nonbrand campaign. 

## `Question 3:`
### While we’re on Gsearch, could you dive into nonbrand, and pull monthly sessions and orders split by device type? I want to flex our analytical muscles a little and show the board we really know our traffic sources.  
### Analysis:
```
SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN website_sessions.website_session_id ELSE NULL END) AS desktop_sessions, 
    COUNT(DISTINCT CASE WHEN device_type = 'desktop' THEN orders.order_id ELSE NULL END) AS desktop_orders,
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN website_sessions.website_session_id ELSE NULL END) AS mobile_sessions, 
    COUNT(DISTINCT CASE WHEN device_type = 'mobile' THEN orders.order_id ELSE NULL END) AS mobile_orders
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
	AND website_sessions.utm_source = 'gsearch'
    AND website_sessions.utm_campaign = 'nonbrand'
GROUP BY 1,2;
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/a37472f1-6ed2-4109-9560-4714039681d3)

### Conclusion:
- Here we can see a lot more desktop sessions from beginning as compared to mobile sessions.
- Initially we can see desktop sessions  and mobile sessions were in 2:1 but after a couple months it increased to more than 3:1 ratios.
- Similarly , in orders desktop to mobile orders in beginning was 5:1 ratios and it increased to 10:1 ratios.

## `Question 4:` 
### I’m worried that one of our more pessimistic board members may be concerned about the large % of traffic from Gsearch. Can you pull monthly trends for Gsearch, alongside monthly trends for each of our other channels?
 
### Analysis: 
### `Part 1:` *finding the various utm sources and referers to see the traffic we're getting*
```
SELECT DISTINCT 
	  utm_source,
    utm_campaign, 
    http_referer
FROM website_sessions
WHERE website_sessions.created_at < '2012-11-27';
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/076a14af-d53c-4677-8471-7efbe35e01e4)

### Conclusion:
- When utm_source, utm_campaign and http_referer is null  then there is direct type in traffic. When only utm_source , utm_campaign are null then it shows it is organic search traffic.

### `Part 2:` *Sessions comparison*
```
SELECT
	YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT CASE WHEN utm_source = 'gsearch' THEN website_sessions.website_session_id ELSE NULL END) AS gsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source = 'bsearch' THEN website_sessions.website_session_id ELSE NULL END) AS bsearch_paid_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NOT NULL THEN website_sessions.website_session_id ELSE NULL END) AS organic_search_sessions,
    COUNT(DISTINCT CASE WHEN utm_source IS NULL AND http_referer IS NULL THEN website_sessions.website_session_id ELSE NULL END) AS direct_type_in_sessions
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/5d33fb89-0752-4ec3-bcfc-675a078262b0)

### Conclusion:
- Here we can see gsearch_paid_sessions are building over time, bsearch_paid_sessions is much more increased then it was before.
- Orgainic search and direct type in session are growing more and it show engagement of customers to our website without any paid marketing.

## `Question 5:`
### I’d like to tell the story of our website performance improvements over the course of the first 8 months. Could you pull session to order conversion rates, by month? 
### Analysis:

```
SELECT
	  YEAR(website_sessions.created_at) AS yr, 
    MONTH(website_sessions.created_at) AS mo, 
    COUNT(DISTINCT website_sessions.website_session_id) AS sessions, 
    COUNT(DISTINCT orders.order_id) AS orders, 
    COUNT(DISTINCT orders.order_id)/COUNT(DISTINCT website_sessions.website_session_id) AS conversion_rate    
FROM website_sessions
	LEFT JOIN orders 
		ON orders.website_session_id = website_sessions.website_session_id
WHERE website_sessions.created_at < '2012-11-27'
GROUP BY 1,2;
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/7e5629c7-b7c7-4d31-8428-c9f3ffe3554b)

### Conclusion:
- Here it can be seen that sessions and orders are increased over a month and conversion rate increased from 3% to 4% over a month.

## `Question 6:`
### For the gsearch lander test, please estimate the revenue that test earned us 
- (Hint: Look at the increase in CVR from the test (Jun 19 – Jul 28), and use nonbrand sessions and revenue since then to calculate incremental value)
- This Question is breakdown in 7 parts for better understanding.
 
### Analysis:
### `Part 1:`
```
SELECT
	MIN(website_pageview_id) AS first_test_pv
FROM website_pageviews
WHERE pageview_url = '/lander-1';
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/57ec7d69-22ba-4cd5-a493-bf37c3ef6524)

### Conclusion:
- we'll get the first pageview id 

### `Part 2:`
```
CREATE TEMPORARY TABLE first_test_pageviews
SELECT
	  website_pageviews.website_session_id, 
    MIN(website_pageviews.website_pageview_id) AS min_pageview_id
FROM website_pageviews 
	INNER JOIN website_sessions 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
		AND website_sessions.created_at < '2012-07-28' -- prescribed by the assignment
		AND website_pageviews.website_pageview_id >= 23504 -- first page_view
        AND utm_source = 'gsearch'
        AND utm_campaign = 'nonbrand'
GROUP BY 
	website_pageviews.website_session_id;

select * from first_test_pageviews;
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/ced10011-5d8c-44cc-a8ef-4bd6915d814f)

### Conclusion:
- Here we will create temporary table as first_test_pageviews.
- It have website  session id and min pageview id.

### `Part 3:`
```
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_landing_pages
SELECT 
	  first_test_pageviews.website_session_id, 
    website_pageviews.pageview_url AS landing_page
FROM first_test_pageviews
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_pageview_id = first_test_pageviews.min_pageview_id
WHERE website_pageviews.pageview_url IN ('/home','/lander-1'); 

SELECT * FROM nonbrand_test_sessions_w_landing_pages;
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/c9b31a36-6668-4c5d-8f95-c3f6918c4f20)

### Conclusion:
- we'll bring in the landing page to each session, but restricting to home or lander-1 this time. 
- This will be used to figure out which pages someone saw and it will help to understand which page perform better.


### `Part 4:`
```
CREATE TEMPORARY TABLE nonbrand_test_sessions_w_orders
SELECT
	  nonbrand_test_sessions_w_landing_pages.website_session_id, 
    nonbrand_test_sessions_w_landing_pages.landing_page, 
    orders.order_id AS order_id

FROM nonbrand_test_sessions_w_landing_pages
LEFT JOIN orders 
	ON orders.website_session_id = nonbrand_test_sessions_w_landing_pages.website_session_id
;

SELECT * FROM nonbrand_test_sessions_w_orders;
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/514da8d9-6a26-4f07-b969-f5ad95c094fe)

### Conclusion:
- we made a temporary table to bring in orders. This will be used to find final performance counts.

### `Part 5:`
```
SELECT
	landing_page, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    COUNT(DISTINCT order_id) AS orders,
    COUNT(DISTINCT order_id)/COUNT(DISTINCT website_session_id) AS conv_rate
FROM nonbrand_test_sessions_w_orders
GROUP BY 1; 
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/c71ba30d-c69e-4caa-a9b6-15bf36658a7e)

### Conclusion:
- Here, we find the difference between conversion rates .
- 0.0319 for /home, vs .0406 for /lander-1 
- 0.0087 additional orders per session
- It shows that we can more focus on lander-1 page

### `Part 6:`
```
SELECT 
	MAX(website_sessions.website_session_id) AS most_recent_gsearch_nonbrand_home_pageview 
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_pageviews.website_session_id = website_sessions.website_session_id
WHERE utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand'
    AND pageview_url = '/home'
    AND website_sessions.created_at < '2012-11-27';	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/1ca2cef3-6161-41a8-8b25-5e4624a1c116)

### Conclusion:
- Here, we got the most recent pageview for gsearch nonbrand where the traffic was sent to /home
- max website_session_id = 17145
- That is the maximum session id of gsearch  nonbrand campaign going to /home page.
- Since then all the traffic is rerouted elsewhere.

### `Part 7:`
```
SELECT 
	COUNT(website_session_id) AS sessions_since_test
FROM website_sessions
WHERE created_at < '2012-11-27'
	AND website_session_id > 17145 -- last /home session
	AND utm_source = 'gsearch'
	AND utm_campaign = 'nonbrand';
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/57fa1e0e-6871-44ce-a591-c48cba9da19a)

### Conclusion:
- 22,972 website sessions since the test
- X times 0.0087 incremental conversion = 202 incremental orders since July 29
- roughly 4 months, so roughly 50 extra orders per month. Not bad!
- Hopefully this analysis make sense, we have improved the performance of website by changing over to the new page.
- We have quantified that improvements by 0.0087 amount of lift of orders generated per session.
- Found total number of session 22,972 * 0.0087 we get 202 incremental orders.
- If we talk about orders per month , it generates 50 extra orders per month.

## `Question 7:`
### For the landing page test you analyzed previously, it would be great to show a full conversion funnel from each of the two pages to orders. You can use the same time period you analyzed last time (Jun 19 – Jul 28).
- This Question is breakdown in 3 parts for better understanding.
 
### Analysis:
### `Part 1:`
```
CREATE TEMPORARY TABLE session_level_made_it_flagged
SELECT
	  website_session_id, 
    MAX(homepage) AS saw_homepage, 
    MAX(custom_lander) AS saw_custom_lander,
    MAX(products_page) AS product_made_it, 
    MAX(mrfuzzy_page) AS mrfuzzy_made_it, 
    MAX(cart_page) AS cart_made_it,
    MAX(shipping_page) AS shipping_made_it,
    MAX(billing_page) AS billing_made_it,
    MAX(thankyou_page) AS thankyou_made_it
FROM(
SELECT
	website_sessions.website_session_id, 
    website_pageviews.pageview_url, 
    -- website_pageviews.created_at AS pageview_created_at, 
    CASE WHEN pageview_url = '/home' THEN 1 ELSE 0 END AS homepage,
    CASE WHEN pageview_url = '/lander-1' THEN 1 ELSE 0 END AS custom_lander,
    CASE WHEN pageview_url = '/products' THEN 1 ELSE 0 END AS products_page,
    CASE WHEN pageview_url = '/the-original-mr-fuzzy' THEN 1 ELSE 0 END AS mrfuzzy_page, 
    CASE WHEN pageview_url = '/cart' THEN 1 ELSE 0 END AS cart_page,
    CASE WHEN pageview_url = '/shipping' THEN 1 ELSE 0 END AS shipping_page,
    CASE WHEN pageview_url = '/billing' THEN 1 ELSE 0 END AS billing_page,
    CASE WHEN pageview_url = '/thank-you-for-your-order' THEN 1 ELSE 0 END AS thankyou_page
FROM website_sessions 
	LEFT JOIN website_pageviews 
		ON website_sessions.website_session_id = website_pageviews.website_session_id
WHERE website_sessions.utm_source = 'gsearch' 
	AND website_sessions.utm_campaign = 'nonbrand' 
    AND website_sessions.created_at < '2012-07-28'
		AND website_sessions.created_at > '2012-06-19'
ORDER BY 
	website_sessions.website_session_id,
    website_pageviews.created_at
) AS pageview_level

GROUP BY 
	website_session_id
;

select * from session_level_made_it_flagged; 	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/3d5579f0-5415-4bab-a12a-32d8a7aea0cb)

### Conclusion:
- Here, we created temporary table to find which session completed upto which page.
- In this we added flags as 0 and 1 to find which page it is.

### `Part 2:`
```
SELECT
	CASE 
    WHEN saw_homepage = 1 THEN 'saw_homepage'
    WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
    ELSE 'uh oh... check logic' 
	END AS segment, 
    COUNT(DISTINCT website_session_id) AS sessions,
    COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS to_products,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS to_mrfuzzy,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS to_cart,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS to_shipping,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS to_billing,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END) AS to_thankyou
FROM session_level_made_it_flagged 
GROUP BY 1
;
	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/910b7654-cf4a-4428-a650-d9255cb93a74)

### Conclusion:
- Here we can see that more orders were placed through lander page as compared to homepage.

### `Part 3:`
```
SELECT
	CASE 
		WHEN saw_homepage = 1 THEN 'saw_homepage'
        WHEN saw_custom_lander = 1 THEN 'saw_custom_lander'
        ELSE 'uh oh... check logic' 
	END AS segment, 
	COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT website_session_id) AS lander_click_rt,
    COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN product_made_it = 1 THEN website_session_id ELSE NULL END) AS products_click_rt,
    COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN mrfuzzy_made_it = 1 THEN website_session_id ELSE NULL END) AS mrfuzzy_click_rt,
    COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN cart_made_it = 1 THEN website_session_id ELSE NULL END) AS cart_click_rt,
    COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN shipping_made_it = 1 THEN website_session_id ELSE NULL END) AS shipping_click_rt,
    COUNT(DISTINCT CASE WHEN thankyou_made_it = 1 THEN website_session_id ELSE NULL END)/COUNT(DISTINCT CASE WHEN billing_made_it = 1 THEN website_session_id ELSE NULL END) AS billing_click_rt
FROM session_level_made_it_flagged
GROUP BY 1;	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/4288006d-87b3-4006-875d-35bad1e34ffd)

### Conclusion:
- Here it shows clickthrough rates of one page to another.
- It is a conversion funnel of moving from initial page to final billing page.

## `Question 8:`
### I’d love for you to quantify the impact of our billing test, as well. Please analyze the lift generated  from the test (Sep 10 – Nov 10), in terms of revenue per billing page session, and then pull the number of billing page sessions for the past month to understand monthly impact.

### Analysis:
### `Part 1:`
```
SELECT 
	website_pageviews.website_session_id, 
    website_pageviews.pageview_url AS billing_version_seen, 
    orders.order_id, 
    orders.price_usd
FROM website_pageviews 
	LEFT JOIN orders
		ON orders.website_session_id = website_pageviews.website_session_id
WHERE website_pageviews.created_at > '2012-09-10' -- prescribed in assignment
	AND website_pageviews.created_at < '2012-11-10' -- prescribed in assignment
    AND website_pageviews.pageview_url IN ('/billing','/billing-2');
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/52eaa6ea-f69a-4330-bb5c-5797aac4836a)

### Conclusion:
- Here, we can see billing versions with order id and price.
- Have you noticed that some billing versions have null order id and price?
- Reason behind it is the purchase of product haven’t proceeded.
- We can connect our customer supports to that user.

### `Part 2:`
```
SELECT
	billing_version_seen, 
    COUNT(DISTINCT website_session_id) AS sessions, 
    SUM(price_usd)/COUNT(DISTINCT website_session_id) AS revenue_per_billing_page_seen
 FROM( 
SELECT 
	website_pageviews.website_session_id, 
    website_pageviews.pageview_url AS billing_version_seen, 
    orders.order_id, 
    orders.price_usd
FROM website_pageviews 
	LEFT JOIN orders
		ON orders.website_session_id = website_pageviews.website_session_id
WHERE website_pageviews.created_at > '2012-09-10' -- prescribed in assignment
	AND website_pageviews.created_at < '2012-11-10' -- prescribed in assignment
    AND website_pageviews.pageview_url IN ('/billing','/billing-2')
) AS billing_pageviews_and_order_data
GROUP BY 1;	
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/3659fc5b-3d5a-4cda-895c-26bc3be3bac8)

### Conclusion:
- We have Analyzed that
- $22.83 revenue per billing page seen for the old version
- $31.34 for the new version
- LIFT: $8.51 per billing page view
- We can now move our overall website traffic towards billing-2 page as it is has more user friendly payment system which increases revenue per billing page

### `Part 3:`
```
SELECT 
	COUNT(website_session_id) AS billing_sessions_past_month
FROM website_pageviews 
WHERE website_pageviews.pageview_url IN ('/billing','/billing-2') 
	AND created_at BETWEEN '2012-10-27' AND '2012-11-27' -- past month;
```
### Output:
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/0700ad23-c08b-490e-8c74-aa85884455c2)

### Conclusion:
- Here we can see  1,193 billing sessions past month
- In previous analysis we seen a LIFT: $8.51 per billing session
- Total billing over past month = billing session * LIFT = (1,193 * $8.51)= $10,152.43
- VALUE OF BILLING TEST: $10,152.43 over the past month
- Now, we can conclude that modification of billing page helped alot to generate more revenue than previous billing page.

# THE END
![image](https://github.com/saurav2021c/Website_Performance_SQL-Project/assets/97289683/d28fb664-104e-4373-90ed-74d165371a93)
<p>
 <a href="https://www.linkedin.com/in/sauravsinghhhh/" target="_blank" rel="noreferrer"><img src="https://github.com/saurav2021c/Portfolio-project/blob/master/src/images/LinkedIN_black.png" width="35" /></a>
</p>

