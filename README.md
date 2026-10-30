# Maven Fuzzy Factory: E-commerce Analytics

I wanted to know one thing: **why do so many people visit this website and so few of them buy?**

This project is my attempt to answer that. I took the data (a toy company selling plushies online) from Maven Analytics website and built a Power BI dashboard that follows a visitor from the first click, through the checkout, and all the way to the refund when things go wrong.



## What I was trying to figure out

* Where does the website traffic come from, and what are those people doing?
* Where do visitors drop off before buying?
* Who are the customers, and do they ever come back?
* Which products actually make the money?
* How much of that money goes back out as refunds?
* If I ran this business, what would I fix first?

## 

## The dashboard: 4 pages

Every page has slicers for device, product and month, so you can slice it however you like.

**1. Website Traffic \& Engagement**
This is the "who is showing up, and where do they leave?" page.

**2. Conversion and Purchase Funnel**
I enjoyed building most, because you can actually see where people give up.

**3. Customer \& Purchase Insights**
Which pages get the most views, sales vs refunds year by year, and refund rate by product.

**4. Sales and Refund Analysis**
My key findings, each one with what I would do about it.



## What I found

* **Mobile has a big problem.** Desktop visitors convert at **8.5%** and mobile visitors at only **3.1%**. Mobile is about a third of all sessions (146K of 473K), so that gap is expensive. Very rough napkin math: if mobile converted like desktop, with the same average order value of about $60, that's somewhere around **$470K** of extra sales. It's a ballpark, not a promise.
* **Conversion has been climbing.** The monthly conversion rate started around 3% in 2012 and was over 8% by early 2015. Something the business did was working.
* **Most people never get past the product pages.** Out of 473K sessions, 210K looked at a product, 95K added to cart, 48K reached checkout and 32K bought. Overall that's **6.8%**.
* **One product carries the business.** The Original Mr. Fuzzy made about **$1.211 million**, roughly 62% of all sales. It's also the page where the most people leave without buying (93K exits), which is really interesting.
* **Almost nobody buys twice,** and I think the small product range is a big reason. Only about 2% of customers ever came back for a second order (591 out of 31,696). My theory is simple: the store sells just four plushies, so once you've bought one there isn't much left to tempt you.
I checked the data and it backs me up, at least partly:

  * People do like variety. About 24% of orders have two items, and all 7,712 of those orders had two different plushies.
  * More products meant more repeat buyers. Customers whose first order was in 2012, when Mr. Fuzzy was the only product, came back just 0.9% of the time, even though they had the longest time to do it. For 2014 first-timers, with nearly all four products live, it was 2.2%.
  * It's not the whole story. More than half of repeat orders (344 of 617) were for a plushie the customer had already bought, so some people do rebuy the same one (gifts, maybe?). 
  * Returning visitors aren't the problem. About 17% of sessions are repeat visits, and they convert at 7.8% vs 6.6% for new visitors. Most of those are people buying for the first time on a later visit, not buying a second time.
  * What I'd do: launch more plushies (new characters, seasonal ones, sets and bundles), then track the repeat rate after each launch to see if it actually moves.
* **Sales grew every year, and refunds grew with them.** 2012: $129K sales and $9K refunds. 2013: $393K and $17K. 2014: **$1.076M and $48K**. 2015 (only to mid-March): $340K and $11K. Overall, about 4.4% of sales went back out as refunds.

**Not all products get refunded equally.** The Birthday Sugar Panda has the highest refund rate at about **6.0%**, then Mr. Fuzzy at **5.1%**, the Forever Love Bear at 2.2% and the Hudson River Mini at 1.3%. The best seller is also one of the most refunded.



## Things to know before you read my numbers

* The data runs from **March 2012 to 19 March 2015**. 2015 is only a couple of months, so the yearly sessions line and the last point of the monthly sales trend dip at the end. That's the data stopping, not the business collapsing.
* **Refunds are counted in the year of the original order**, not the day the refund happened.
* **Funnel steps:** "Product Page" means the visitor viewed one of the four product pages, and "Checkout" counts the newer `/billing-2` page. A few thousand sessions used the older `/billing` page, so the checkout step is slightly low.
* Every number follows the definitions in my DAX measures. Define things differently and you'll get different answers.



## The part where I struggled (and what I learned)

My first charts were a mess. Everything went blank, or I got one single bar. It took me a while to work out that my date table wasn't connecting, because the dates in my data had times attached and my calendar didn't. I also had too many relationships pointing at the same tables, so the model didn't know which path to follow.

Fixing the data model taught me more than building any chart did. If your visuals go blank, check your relationships before you touch your DAX.



## Repository structure

```text
Maven-Fuzzy-Factory-Analysis/
│
├── README.md
├── PowerBI/
│   └── Maven\_Fuzzy\_Factory\_Analysis.pbix
├── Screenshots/
│   ├── 01\_Traffic\_Engagement.png
│   ├── 02\_Conversion\_Funnel.png
│   ├── 03\_Customer\_Purchase\_Insights.png
│   └── 04\_Sales\_Refund\_Analysis.png
└── Documentation/
    └── DAX\_Measures.md
```

\---

## Status

The dashboard and my first round of analysis are done. I'm still learning, so if you spot something I could do better, I'd genuinely love to hear it.



