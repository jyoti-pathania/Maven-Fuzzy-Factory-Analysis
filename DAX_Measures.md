# DAX Measures: Maven Fuzzy Factory



## Contents

1. [Calculated tables](#calculated-tables)
2. [Calculated columns](#calculated-columns)
3. [Measures](#measures)
4. [Notes and definitions](#notes-and-definitions)

\---



## Calculated table



### Funnel Stages

Labels and order for the five funnel stages.

```dax
DATATABLE (
    "Stage", STRING,
    "Stage Order", INTEGER,
    {
        { "Website Sessions", 1 },
        { "Product Page", 2 },
        { "Cart", 3 },
        { "Checkout", 4 },
        { "Purchase", 5 }
    }
)
```

### DateTable

A calendar from 2010 to 20.

```dax
ADDCOLUMNS(
    CALENDAR(
        DATE(2010, 1, 1),
        DATE(2020, 12, 31)
    ),
    "Year", YEAR(\[Date]),
    "Month Number", MONTH(\[Date]),
    "Month", FORMAT(\[Date], "MMMM"),
    "Month Short", FORMAT(\[Date], "MMM"),
    "Quarter", "Q" \& FORMAT(\[Date], "Q"),
    "Quarter Number", QUARTER(\[Date]),
    "Day", DAY(\[Date]),
    "Weekday Number", WEEKDAY(\[Date], 2),
    "Weekday", FORMAT(\[Date], "dddd"),
    "Weekday Short", FORMAT(\[Date], "ddd"),
    "Week Number", WEEKNUM(\[Date], 2),
    "Day Type",
        IF(
            WEEKDAY(\[Date], 2) > 5,
            "Weekend",
            "Weekday"
        ),
    "Year-Month", FORMAT(\[Date], "yyyy-MM"),
    "Year-Month Sort",
        YEAR(\[Date]) \* 100 + MONTH(\[Date])
)
```

## Calculated columns

### `website\_sessions\[Session Date]`

Session date without the time of day.

```dax
DATE ( YEAR ( website\_sessions\[created\_at] ),
       MONTH ( website\_sessions\[created\_at] ),
       DAY ( website\_sessions\[created\_at] ) )
```

### `website\_sessions\[Date Only]`

Same as `Session Date`. Date-only version of `created\_at`.

```dax
DATE (
    YEAR ( website\_sessions\[created\_at] ),
    MONTH ( website\_sessions\[created\_at] ),
    DAY ( website\_sessions\[created\_at] )
)
```

### `website\_sessions\[Channel]`

Readable traffic channel. Blank source means `direct / organic`.

```dax
IF (
    ISBLANK ( website\_sessions\[utm\_source] ),
    "direct / organic",
    website\_sessions\[utm\_source] \& " | " \& website\_sessions\[utm\_campaign]
)
```

### `website\_sessions\[Visitor Type]`

Labels each session as `New` or `Repeat`.

```dax
IF ( website\_sessions\[is\_repeat\_session] = "1", "Repeat", "New" )
```

### `website\_sessions\[Landing Page]`

The first page seen in each session (lowest pageview ID). Used in the landing page chart.

```dax
VAR FirstPV =
    CALCULATE ( MIN ( website\_pageviews\[website\_pageview\_id] ) )
RETURN
    LOOKUPVALUE (
        website\_pageviews\[pageview\_url],
        website\_pageviews\[website\_pageview\_id], FirstPV
    )
```

### `website\_pageviews\[Is Exit Page]`

`1` if this pageview is the last one in its session, otherwise `0`.

```dax
VAR CurrentSession =
    website\_pageviews\[website\_session\_id]

VAR CurrentPageviewID =
    website\_pageviews\[website\_pageview\_id]

VAR LastPageviewID =
    MAXX (
        FILTER (
            ALL ( website\_pageviews ),
            website\_pageviews\[website\_session\_id] = CurrentSession
        ),
        website\_pageviews\[website\_pageview\_id]
    )

RETURN
    IF (
        CurrentPageviewID = LastPageviewID,
        1,
        0
    )
```

**Date Only** columns also exist on `orders`, `order\_items`, `order\_item\_refunds`, `website\_pageviews` and `products`. Each one is `DATE ( YEAR ( created\_at ), MONTH ( created\_at ), DAY ( created\_at ) )`, which removes the time of day.

\---

## Measures

### 

### Traffic \& Engagement

#### Sessions

Number of unique website sessions.

```dax
DISTINCTCOUNT ( website\_sessions\[website\_session\_id] )
```

**Used on:** Traffic, Conversion

#### Users

Number of unique users (by `user\_id`) who visited the site.

```dax
DISTINCTCOUNT(website\_sessions\[user\_id])
```

#### Unique Visitors

Same calculation as `Users`. Kept because the report uses this name.

```dax
DISTINCTCOUNT ( website\_sessions\[user\_id] )
```

**Used on:** Traffic, Conversion

#### Pageviews

Number of unique pageviews.

```dax
DISTINCTCOUNT (
    website\_pageviews\[website\_pageview\_id]
)
```

**Used on:** Conversion, Customers \& Products

#### Average Pageviews per Session

Pageviews divided by sessions.

```dax
DIVIDE (
    \[Pageviews],
    \[Sessions],
    0
)
```

**Used on:** Traffic

#### Repeat Session %

Share of sessions that were repeat visits.

```dax
DIVIDE (
    CALCULATE ( \[Sessions], website\_sessions\[is\_repeat\_session] = 1 ),
    \[Sessions]
)
```

#### Mobile Sessions

Sessions from mobile devices.

```dax
CALCULATE ( \[Sessions], website\_sessions\[device\_type] = "mobile" )
```

#### Desktop Sessions

Sessions from desktop devices.

```dax
CALCULATE ( \[Sessions], website\_sessions\[device\_type] = "desktop" )
```

#### Mobile Share %

Mobile sessions as a share of all sessions.

```dax
DIVIDE ( \[Mobile Sessions], \[Sessions] )
```

#### Sessions Share %

Each row's share of sessions within the current selection (`ALLSELECTED`).

```dax
DIVIDE ( \[Sessions], CALCULATE ( \[Sessions], ALLSELECTED () ) )
```

#### Exit Sessions

Sessions whose last pageview is on the page in context. Uses the calculated column `Is Exit Page`.

```dax
CALCULATE (
    DISTINCTCOUNT (
        website\_pageviews\[website\_session\_id]
    ),
    website\_pageviews\[Is Exit Page] = 1
)
```

**Used on:** Traffic, Customers \& Products

#### Page Exit Rate

Exit sessions divided by pageviews.

```dax
DIVIDE (
    \[Exit Sessions],
    \[Pageviews],
    0
)
```

#### Unique Pages Viewed

Number of distinct page URLs viewed.

```dax
DISTINCTCOUNT ( website\_pageviews\[pageview\_url] )
```

**Used on:** Customers \& Products

### Sales \& Profit

#### Orders

Number of unique orders.

```dax
DISTINCTCOUNT(orders\[order\_id])
```

#### Revenue

Order revenue, taken from the `orders` table.

```dax
SUM ( orders\[price\_usd] )
```

#### COGS

Cost of goods sold, from the `orders` table.

```dax
SUM ( orders\[cogs\_usd] )
```

#### Gross Profit

Revenue minus COGS.

```dax
\[Revenue] - \[COGS]
```

#### Gross Margin %

Gross profit as a share of revenue.

```dax
DIVIDE ( \[Gross Profit], \[Revenue] )
```

#### AOV

Average order value: revenue divided by orders.

```dax
DIVIDE ( \[Revenue], \[Orders] )
```

#### Gross Profit per Order

Gross profit divided by orders.

```dax
DIVIDE ( \[Gross Profit], \[Orders] )
```

#### Revenue Share %

Share of revenue within the current selection (`ALLSELECTED`).

```dax
DIVIDE ( \[Revenue], CALCULATE ( \[Revenue], ALLSELECTED () ) )
```

#### Total Sales

Sales taken from `order\_items`. Use this for product-level views, because a product filter reaches `order\_items` but not `orders`.

```dax
SUM ( order\_items\[price\_usd] )
```

**Used on:** Conversion, Customers \& Products

### Refunds \& Net Profit

#### Total\_Refunds

Sum of refunded dollars.

```dax
SUM(order\_item\_refunds\[refund\_amount\_usd])
```

**Used on:** Conversion, Customers \& Products

#### Refund Rate %

Refunds as a share of sales. Divides by `Total Sales` so it stays correct when filtered by product.

```dax
DIVIDE ( \[Total\_Refunds], \[Total Sales] )
```

**Used on:** Customers \& Products

#### Refunded Items

Number of unique refunded items.

```dax
DISTINCTCOUNT ( order\_item\_refunds\[order\_item\_refund\_id] )
```

#### Returns

Number of unique orders that had a refund.

```dax
DISTINCTCOUNT(order\_item\_refunds\[order\_id])
```

#### Net Revenue

Revenue minus refunds.

```dax
\[Revenue] - \[Total\_Refunds]
```

#### Net Profit

Revenue minus COGS minus refunds.

```dax
\[Revenue] - \[COGS] - \[Total\_Refunds]
```

#### Net Margin %

Net profit as a share of revenue.

```dax
DIVIDE ( \[Net Profit], \[Revenue] )
```

### Conversion \& Efficiency

#### Conversion Rate

Orders divided by sessions. This is a session-based conversion rate (about 6.8% overall).

```dax
DIVIDE ( \[Orders], \[Sessions] )
```

**Used on:** Conversion, Customers \& Products

#### Session Conversion Rate

Purchase sessions divided by sessions.

```dax
DIVIDE (
    \[Purchase Sessions],
    \[Sessions],
    0
)
```

**Used on:** Conversion

#### Revenue per Session

Revenue divided by sessions.

```dax
DIVIDE ( \[Revenue], \[Sessions] )
```

#### Net Profit per Session

Net profit divided by sessions.

```dax
DIVIDE ( \[Net Profit], \[Sessions] )
```

#### Mobile Conversion Rate

Conversion rate for mobile sessions only.

```dax
CALCULATE ( \[Conversion Rate], website\_sessions\[device\_type] = "mobile" )
```

#### Desktop Conversion Rate

Conversion rate for desktop sessions only.

```dax
CALCULATE ( \[Conversion Rate], website\_sessions\[device\_type] = "desktop" )
```

**Used on:** Customers \& Products

#### Device Conversion Gap (pp)

Desktop minus mobile conversion, in percentage points.

```dax
( \[Desktop Conversion Rate] - \[Mobile Conversion Rate] ) \* 100
```

#### New Visitor Conversion

Conversion rate for first-time visitors.

```dax
CALCULATE ( \[Conversion Rate], website\_sessions\[is\_repeat\_session] = 0 )
```

#### Repeat Visitor Conversion

Conversion rate for returning visitors.

```dax
CALCULATE ( \[Conversion Rate], website\_sessions\[is\_repeat\_session] = 1 )
```

#### Revenue Lost to Mobile Gap (est.)

Rough estimate of extra revenue if mobile converted like desktop, using the current AOV. A ballpark figure, not a forecast.

```dax
VAR MobileGap = \[Desktop Conversion Rate] - \[Mobile Conversion Rate]
RETURN
    MobileGap \* \[Mobile Sessions] \* \[AOV]
```

#### Customers Who Purchased

Number of unique customers (by `user\_id`) with at least one order.

```dax
DISTINCTCOUNT ( orders\[user\_id] )
```

**Used on:** Conversion

#### Visitor Conversion Rate

Customers who purchased divided by unique visitors. This is user-based, so it differs from the session-based rate.

```dax
DIVIDE (
    \[Customers Who Purchased],
    \[Unique Visitors],
    0
)
```

### Purchase Funnel

#### Product Page Sessions

Sessions that viewed one of the four product pages (URL starts with `/the-`).

```dax
CALCULATE (
    DISTINCTCOUNT (
        website\_pageviews\[website\_session\_id]
    ),
    FILTER (
        website\_pageviews,
        LEFT (
            website\_pageviews\[pageview\_url],
            5
        ) = "/the-"
    )
)
```

#### Cart Sessions

Sessions that reached `/cart`.

```dax
CALCULATE (
    DISTINCTCOUNT (
        website\_pageviews\[website\_session\_id]
    ),
    website\_pageviews\[pageview\_url] = "/cart"
)
```

#### Checkout Sessions

Sessions that reached `/billing-2`. The older `/billing` page (about 3.6K sessions) is not counted.

```dax
CALCULATE (
    DISTINCTCOUNT (
        website\_pageviews\[website\_session\_id]
    ),
    website\_pageviews\[pageview\_url] = "/billing-2"
)
```

#### Purchase Sessions

Sessions that ended in an order.

```dax
CALCULATE (
    DISTINCTCOUNT (
        orders\[website\_session\_id]
    )
)
```

**Used on:** Traffic

#### Funnel Sessions

Feeds the funnel visual. Returns the matching stage count for each row of the `Funnel Stages` table.

```dax
SWITCH (
    SELECTEDVALUE (
        'Funnel Stages'\[Stage]
    ),
    "Website Sessions", \[Sessions],
    "Product Page", \[Product Page Sessions],
    "Cart", \[Cart Sessions],
    "Checkout", \[Checkout Sessions],
    "Purchase", \[Purchase Sessions],
    BLANK ()
)
```

**Used on:** Conversion

#### Product Page Conversion Rate

Cart sessions divided by product page sessions (the product-to-cart rate).

```dax
DIVIDE (
    \[Cart Sessions],
    \[Product Page Sessions],
    0
)
```

#### Cart to Checkout Rate

Checkout sessions divided by cart sessions.

```dax
DIVIDE (
    \[Checkout Sessions],
    \[Cart Sessions],
    0
)
```

#### Checkout to Purchase Rate

Purchase sessions divided by checkout sessions.

```dax
DIVIDE (
    \[Purchase Sessions],
    \[Checkout Sessions],
    0
)
```

### Customers \& Repeat Buying

#### New Customers

Customers with exactly one order.

```dax
VAR CustomerOrders =
    ADDCOLUMNS (
        VALUES ( orders\[user\_id] ),
        "OrderCount",
            CALCULATE ( DISTINCTCOUNT ( orders\[order\_id] ) )
    )
RETURN
    COUNTROWS (
        FILTER ( CustomerOrders, \[OrderCount] = 1 )
    )
```

#### Repeat Customers

Customers with more than one order.

```dax
VAR CustomerOrders =
    ADDCOLUMNS (
        VALUES ( orders\[user\_id] ),
        "OrderCount",
            CALCULATE ( DISTINCTCOUNT ( orders\[order\_id] ) )
    )
RETURN
    COUNTROWS (
        FILTER ( CustomerOrders, \[OrderCount] > 1 )
    )
```

#### Repeat Customer Rate

Repeat customers divided by all customers who purchased.

```dax
DIVIDE (
    \[Repeat Customers],
    \[New Customers] + \[Repeat Customers],
    0
)
```

**Used on:** Conversion

\---

## Notes and definitions

* **Two conversion rates.** `Conversion Rate` and `Session Conversion Rate` are session-based (orders ÷ sessions, about 6.8%). `Visitor Conversion Rate` is user-based (customers ÷ unique visitors, about 8.0%). They answer different questions.
* **Sales vs Revenue.** `Revenue` comes from `orders` and `Total Sales` from `order\_items`. The totals match, but only `Total Sales` responds to a product filter.
* **Funnel stages.** "Product Page" counts sessions that viewed one of the four product pages. "Checkout" counts `/billing-2` only. Purchase counts every session with an order, so the checkout-to-purchase rate is slightly high.
* **Refunds by year.** In the yearly sales vs refunds chart, refunds fall in the year of the original order.
* **Partial 2015.** Data ends on 19 March 2015, so yearly and monthly trends dip at the end because the data stops, not because the business declined.
* **Repeat customers.** Counted by `user\_id` across all orders. About 2% of purchasing customers ordered more than once.

