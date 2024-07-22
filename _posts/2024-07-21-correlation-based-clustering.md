---
layout: post
title: "Correlation Can Be Used As Distance in Clustering"
author: "Dojun Hwang"
comments: true
mathjax: true
---

Clustering tasks usually considers distances between objects, but mere distances are not effective in capturing the common patterns or moves of objects and clustering by those patterns. 
This article is going to show that **correlation** between objects can be used in clustering to cluster objects by their common patterns.
This article is also going to show that this method has its theorecial background -- correlation between two objects is equal to the Euclidean distance between row-wise standardized data.
By this theoretical background, we can expect that this method will be effective especially in time-series data.

# Illustrative Example

Suppose you have the following time series data.

![Example](/assets/images/correlation/20240702002126.png)
_The left is original scale, and the right is log scaled one._

$$\begin{aligned}x_1 &= [1,  2,  8,  20,  15,  4,  3,  4,  6,  7,  5,  2]^T \\
x_2 &= [10, 18, 75, 330, 210, 60, 40, 45, 50, 70, 15, 18]^T\\
x_3 &= [1, 4, 6, 5, 6, 2, 12, 8, 32, 42, 25, 9]^T \\ 
x_4 &= [10, 36, 50, 52, 30, 15, 77, 70, 280, 385, 245, 60]^T \\ 
x_5 &= [16, 24, 16, 24, 16, 24, 16, 24, 16, 24, 16, 24]^T \end{aligned}$$

This data could be traffic data by time of day at each junction, or power usage data for each region over a single day, or anything else you can imagine. 
**You may want to *cluster* those lines by their pattern.** 
That is, if two lines increase and decrease at the same time, then they must be in the same cluster.

The plot on the left side is the original data. 
Can you capture the common patterns of the lines? 
It may be hard; but looking at the log-scaled data at the right side, you may notice that there are common patterns for these lines. 
The pink and red lines increases drastically for the first half and increases moderately for the second half, whereas the blue and skyblue lines increases drastically for the second half. 
The green line equally increases and decreases periodically. 
So these lines can be clustered in the three different clusters. 

But, **how can we compute these clusters?** *Mere k-means clustering cannot deal with finding patterns.* 
This is because, it computes distances only based on absolute quantities. 
In the plot above, even though pink and red lines have similar patterns, their quantity differs a lot, and the red line is closer to the blue line regarding the quantities. 
So the red line will be clustered with the blue line rather than the pink line.

$$d(x_1, x_2) = d(red, pink) = [(1-10)^2 + (2-18)^2 + \cdots + (2-18)^2]^{1/2} = 389.099$$

$$d(x_1, x_3) = d(red, blue) = [(1-1)^2 + (2-4)^2 + \cdots + (2-9)^2]^{1/2}=52.583$$

There must be many different remedies to compute the clusters properly. 
One can compute rate of change and then cluster data onto the rate of change. 
Or we can use cosine similarity as the metric to cluster data. 
But, here I want to show another way; **using correlation as a distance metric**.

![Scatterplot](/assets/images/correlation/20240702010334.png)

So, the main idea is like this: **if two data have a similar pattern, then they must be positively correlated.** 
It does make sense, as being highly-correlated means that two data are close to the each other, and it is reasonable to group them together into a single cluster.
Look at the scatterplots above. We can notice that $x_1$(red) is strongly positively correlated with $x_2$(pink), whereas not that strongly correlated to the others. In fact, it is that

$$\begin{aligned}Corr(x_1, x_2) &= 0.971 \\ Corr(x_1, x_3) &= -0.028 \\ Corr(x_1, x_4) &= -0.019 \\ Corr(x_1, x_5) &= 0.015. \end{aligned}$$

So this kind of approach must be helpful in capturing patterns and then clustering the data.
To use the correlation as a distance metric in clsutering procedure, **let's use $1 - Corr(x, y)$ as a distance between data $x$ and $y$.** 
This is reasonable, as
- (positivity) $0 \leq 1 - Corr(x, y)$, and $1-Corr(x, y) = 0$ only when $x$ and $y$ are perfectly positively correlated.
- (symmetry) $1 - Corr(x, y) = 1 - Corr(y, x)$.

The plot below is the heatmap of $1-Corr$ for the given data.

![Heatmap](/assets/images/correlation/20240702012043.png)

After computing $1 - Corr(x, y)$, we can deem this as the distance between $x$ and $y$ and and proceed to appropriate clustering methods. 
A typical method is **agglomerative clustering.** The plot at the below is the resulted dendrogram. 

![Dendrogram](/assets/images/correlation/20240702012509.png)

Here, we can conclude that 2 and 3 (blue and skyblue) are in the same cluster, and 0 and 1 (red and pink) are in the same cluster. 
So, in conclusion, **the data are clustered with their patterns!** The absolute quantities did not affect the result of clustering that much.

This kind of technique (using correlations as distances) is shown in some books. 
For example, a textbook by Johnson and Wichern (2002) showed an illustrative example of using the sample correlations are used as similarity measures (pp. 747-748).
However, I couldn't find a theoretical background to justify this technique. Is using correlation as a distance appropriate for clustering task?

# How This Works

From the previous example, we found that clustering using correlations as a distance metric is *somewhat* effective in clustering by patterns. 
But, does it make sense to use correlation as a distance metric? Can we justify our decision?

In fact, the $1-Corr$ itself is not a distance metric. This is because it does not meet the triangle inequality.

### Example: Correlation does not meet the triangle inequality
Suppose that

$$\begin{aligned} x_1 &= [4, 9, 2] \\ x_2 &= [2,4,7] \\ x_3 &= [0,1,4] \end{aligned}$$

Then, the *population* correlations are

$$\rho = \begin{bmatrix}1 &  &  \\ -0.3857 & 1 & \\ -0.5329 & 0.9862 & 1 \end{bmatrix}$$

so that the distance matrix is

$$d = \begin{bmatrix}0 &  &  \\ 1.3857 & 1 & \\ 1.5329 & 0.0138 & 1 \end{bmatrix}$$

Here, $d(x_1, x_3) = 1.5329 > d(x_1, x_2) + d(x_2, x_3) = 1.3995$. Thus the triangle inquality does not hold here.

For this fact, we cannot safely use correlation as a distance metric in every cases, as it may lead to the somewhat strange clustering result. 
However, we can show that correlation can be a distance metric in a different perspective. 
**Actually, $1 - Corr(x, y)$ is a distance between *standardized* $x$ and *standardized* $y$.** 

### (1 - Correlation) is a squared distance between row-wise standardized data

**Theorem**. Suppose we are given sequence of data $x = (x_1, \cdots, x_k)$ and $y = (y_1, \cdots, y_k)$. 
Suppose these data are standardized ones, so that

$$E(x) = \dfrac{1}{k}\sum_i x_i = E(y) = \dfrac{1}{k}\sum_i y_i = 0,$$

and 
 
 $$Var(x) = \dfrac{1}{k}\sum_i [x_i - E(x)]^2 = Var(y) = \dfrac{1}{k}\sum_i [y_i - E(x)]^2 = 1.$$

(Note that the variance is population variance, not sample.) Then, 

$$||x - y||^2_2 \propto 1 - Corr(x, y),$$ 

where $Corr(x, y)$ is the Pearson correlation between $x$ and $y$.

**Proof**. As $E(x) = E(y) = 0$ and $Var(x) = Var(y) = 1$, it is also that $E(x^2) = E(y^2) = 1$. Furthermore,

$$\begin{aligned} Corr(x, y) &= \dfrac{Cov(x, y)}{\sqrt{Var(x) Var(y)}}\\ &= Cov(x, y) \\  &=\dfrac{1}{k} \sum_i (x_i - E(x))(y_i - E(y)) \\ &= \dfrac{1}{k}\sum_i x_i y_i \\ &= E(xy). \end{aligned}$$ 

Then,

$$\begin{aligned} ||x - y||^2_2 &= \sum_i (x_i - y_i)^2 \\ &= \sum_i(x_i^2 + y_i^2 - 2x_iy_i) \\ &= kE(x^2) + kE(y^2) - 2kE(xy) \\ &= 2k[1-Corr(x, y)]. \end{aligned}$$ 

Hence $\|\|x - y\|\|^2_2 \propto 1 - Corr(x, y)$.

This theorem implies that, **using (1 - correlation) between two objects as a distance is equivalent to use the distance between *row-wise standardized* data.** 
Hence clustering data using correlation as similarity measure may make sense.

However, we must focus on that it is equivalent to distance between *row-wise standardized* data; but standardization is usually done on column-wise manner. This is because, generally, there is no point in standardizing observations in different columns. Imagine that, there is data looks like this:

| Age | Income | Number of children |
| --- | ------ | ------------------ |
| 48  | 100000 | 2                  |

Is standardizing this data by row make sense? It does not, as each column delivers very different features of each object. 

However, **time-series data** is special, as each column is entangled with others. We may be able to predict the outcome at $t_5$ using the numbers in column $t_1, t_2, t_3$, and $t_4$. 
Therefore it does make sense to use correlation as a similarity measure in the case of time-series data.

# Case Study: Clustering S&P 500 Companies

Let's apply this method to real data. Stock prices data is typical time-series data.
I am going to cluster S&P 500 Companies by their stock prices in 2023.
The data is from [Kaggle](https://www.kaggle.com/datasets/andrewmvd/sp-500-stocks).

![The data](/assets/images/correlation/stock1.png)
_The raw data_

For stock prices data, we usually consider return rate instead of the price itself.
I calculated rise & fall rate for every day, and then compute correlation between each stock.
Then I calculated the distance matrix, by (1 - correlation).
And then, I conducted agglomerative hierarchical clustering for that distance matrix. I used the complete linkage agglomeration method.
The plot below is the resulting dendrogram.

![Dendrogram](/assets/images/correlation/stock2.png)

How should I decide the number of clusters?
The simple way is, **cutting the dendrogram at the height = 1**.
Height = 1 means correlation = 0, and it implies that two data are not correlated.
Hence if two clusters are fused after the height 1, then they are actually not correlated.

![Dendrogram with line](/assets/images/correlation/stock3.png)
_The dendrogram cut by height 1._

We have got 10 clusters. Let's look at each cluster.

---

### Cluster 1
<details>
  <summary>Symbols</summary>
  
<div markdown="1">
```
    Symbol                                        Longname             Sector
8    BRK-B                         Berkshire Hathaway Inc. Financial Services
12     JPM                            JPMorgan Chase & Co. Financial Services
22      HD                            The Home Depot, Inc.  Consumer Cyclical
23     BAC                     Bank of America Corporation Financial Services
37     TMO                   Thermo Fisher Scientific Inc.         Healthcare
38     WFC                           Wells Fargo & Company Financial Services
42     DHR                             Danaher Corporation         Healthcare
56      MS                                  Morgan Stanley Financial Services
59      GS                   The Goldman Sachs Group, Inc. Financial Services
63     UNP                       Union Pacific Corporation        Industrials
66     HON                    Honeywell International Inc.        Industrials
69     LOW                          Lowe's Companies, Inc.  Consumer Cyclical
78       C                                  Citigroup Inc. Financial Services
79     UPS                     United Parcel Service, Inc.        Industrials
87    SCHW                  The Charles Schwab Corporation Financial Services
90     NKE                                      NIKE, Inc.  Consumer Cyclical
99     ADP                 Automatic Data Processing, Inc.        Industrials
103     FI                                    Fiserv, Inc.         Technology
112    ICE                 Intercontinental Exchange, Inc. Financial Services
115    SHW                    The Sherwin-Williams Company    Basic Materials
120    APH                            Amphenol Corporation         Technology
121   CTAS                              Cintas Corporation        Industrials
124    FDX                               FedEx Corporation        Industrials
130    CMG                    Chipotle Mexican Grill, Inc.  Consumer Cyclical
131    ITW                        Illinois Tool Works Inc.        Industrials
134     PH                     Parker-Hannifin Corporation        Industrials
135    USB                                    U.S. Bancorp Financial Services
136    PNC          The PNC Financial Services Group, Inc. Financial Services
138    ECL                                     Ecolab Inc.    Basic Materials
142    CSX                                 CSX Corporation        Industrials
146    MSI                        Motorola Solutions, Inc.         Technology
153    ROP                        Roper Technologies, Inc.         Technology
161    MMM                                      3M Company        Industrials
162    DHI                               D.R. Horton, Inc.  Consumer Cyclical
163    TFC                    Truist Financial Corporation Financial Services
167      F                              Ford Motor Company  Consumer Cyclical
168     GM                          General Motors Company  Consumer Cyclical
172    MET                                   MetLife, Inc. Financial Services
177    NSC                    Norfolk Southern Corporation        Industrials
182   CPRT                                    Copart, Inc.        Industrials
198    LEN                              Lennar Corporation  Consumer Cyclical
199    GWW                             W.W. Grainger, Inc.        Industrials
202     BK         The Bank of New York Mellon Corporation Financial Services
203    TEL                            TE Connectivity Ltd.         Technology
212    PRU                      Prudential Financial, Inc. Financial Services
214   PAYX                                   Paychex, Inc.        Industrials
218   ODFL                 Old Dominion Freight Line, Inc.        Industrials
220    IQV                             IQVIA Holdings Inc.         Healthcare
221    AMP                      Ameriprise Financial, Inc. Financial Services
224   OTIS                      Otis Worldwide Corporation        Industrials
228   FICO                          Fair Isaac Corporation         Technology
229   MSCI                                       MSCI Inc. Financial Services
230    AME                                    AMETEK, Inc.        Industrials
231     IR                             Ingersoll Rand Inc.        Industrials
233   FAST                                Fastenal Company        Industrials
236      A                      Agilent Technologies, Inc.         Healthcare
238    GLW                            Corning Incorporated         Technology
243   CTSH      Cognizant Technology Solutions Corporation         Technology
244   GEHC                 GE HealthCare Technologies Inc.         Healthcare
247    HPQ                                         HP Inc.         Technology
252   NDAQ                                    Nasdaq, Inc. Financial Services
255     IT                                   Gartner, Inc.         Technology
259   LULU                        Lululemon Athletica Inc.  Consumer Cyclical
261    MLM                 Martin Marietta Materials, Inc.    Basic Materials
262    VMC                        Vulcan Materials Company    Basic Materials
263    XYL                                      Xylem Inc.        Industrials
276    ROK                       Rockwell Automation, Inc.        Industrials
281    PPG                            PPG Industries, Inc.    Basic Materials
282   CBRE                                CBRE Group, Inc.        Real Estate
287    DAL                           Delta Air Lines, Inc.        Industrials
288    WAB Westinghouse Air Brake Technologies Corporation        Industrials
289   TSCO                          Tractor Supply Company  Consumer Cyclical
292    MTD               Mettler-Toledo International Inc.         Healthcare
293   FITB                             Fifth Third Bancorp Financial Services
294    MTB                            M&T Bank Corporation Financial Services
301    FTV                             Fortive Corporation         Technology
304    NVR                                       NVR, Inc.  Consumer Cyclical
307    PHM                                PulteGroup, Inc.  Consumer Cyclical
312    STT                        State Street Corporation Financial Services
313    DOV                               Dover Corporation        Industrials
315    IFF         International Flavors & Fragrances Inc.    Basic Materials
322     BR            Broadridge Financial Solutions, Inc.         Technology
333    RJF                   Raymond James Financial, Inc. Financial Services
337   DECK                     Deckers Outdoor Corporation  Consumer Cyclical
340     WY                            Weyerhaeuser Company        Real Estate
342   HBAN              Huntington Bancshares Incorporated Financial Services
351   CPAY                                    Corpay, Inc.         Technology
352   GDDY                                    GoDaddy Inc.         Technology
354     RF                   Regions Financial Corporation Financial Services
359    PFG                 Principal Financial Group, Inc. Financial Services
365   BLDR                      Builders FirstSource, Inc.        Industrials
366    GPC                           Genuine Parts Company  Consumer Cyclical
367    BBY                              Best Buy Co., Inc.  Consumer Cyclical
368   CINF                Cincinnati Financial Corporation Financial Services
372   APTV                                       Aptiv PLC  Consumer Cyclical
373    CFG                  Citizens Financial Group, Inc. Financial Services
374   ULTA                               Ulta Beauty, Inc.  Consumer Cyclical
383    BAX                       Baxter International Inc.         Healthcare
385    WAT                              Waters Corporation         Healthcare
389   VRSN                                  VeriSign, Inc.         Technology
393   NTRS                      Northern Trust Corporation Financial Services
397   EXPD    Expeditors International of Washington, Inc.        Industrials
401   JBHT              J.B. Hunt Transport Services, Inc.        Industrials
405    LUV                          Southwest Airlines Co.        Industrials
407    FDS                   FactSet Research Systems Inc. Financial Services
409    MAS                               Masco Corporation        Industrials
414    UAL                  United Airlines Holdings, Inc.        Industrials
419    IEX                                IDEX Corporation        Industrials
422    KEY                                         KeyCorp Financial Services
424   AKAM                       Akamai Technologies, Inc.         Technology
427    SNA                            Snap-on Incorporated        Industrials
431   VTRS                                    Viatris Inc.         Healthcare
437   NDSN                             Nordson Corporation        Industrials
438    SWK                    Stanley Black & Decker, Inc.        Industrials
439   RVTY                                   Revvity, Inc.         Healthcare
441    PNR                                     Pentair plc        Industrials
443    AOS                         A. O. Smith Corporation        Industrials
446   POOL                                Pool Corporation        Industrials
449    KMX                                    CarMax, Inc.  Consumer Cyclical
455   JKHY                   Jack Henry & Associates, Inc.         Technology
458    LKQ                                 LKQ Corporation  Consumer Cyclical
460   TECH                          Bio-Techne Corporation         Healthcare
462    BXP                                       BXP, Inc.        Real Estate
465    CRL  Charles River Laboratories International, Inc.         Healthcare
473   CHRW                   C.H. Robinson Worldwide, Inc.        Industrials
475     RL                        Ralph Lauren Corporation  Consumer Cyclical
479   GNRC                           Generac Holdings Inc.        Industrials
480    TPR                                  Tapestry, Inc.  Consumer Cyclical
487   HSIC                              Henry Schein, Inc.         Healthcare
489    BIO                      Bio-Rad Laboratories, Inc.         Healthcare
492   BBWI                         Bath & Body Works, Inc.  Consumer Cyclical
494    MHK                         Mohawk Industries, Inc.  Consumer Cyclical
499    BWA                                 BorgWarner Inc.  Consumer Cyclical
503    AAL                    American Airlines Group Inc.        Industrials
```
</div>
</details>

### Cluster 2
<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                            Longname                 Sector
1     AAPL                          Apple Inc.             Technology
2     MSFT               Microsoft Corporation             Technology
4     GOOG                       Alphabet Inc. Communication Services
5    GOOGL                       Alphabet Inc. Communication Services
6     AMZN                    Amazon.com, Inc.      Consumer Cyclical
7     META                Meta Platforms, Inc. Communication Services
10    TSLA                         Tesla, Inc.      Consumer Cyclical
28    NFLX                       Netflix, Inc. Communication Services
30    ADBE                          Adobe Inc.             Technology
31     CRM                    Salesforce, Inc.             Technology
45     ABT                 Abbott Laboratories             Healthcare
48     AXP            American Express Company     Financial Services
54      BX                     Blackstone Inc.     Financial Services
57    ISRG            Intuitive Surgical, Inc.             Healthcare
60     NOW                    ServiceNow, Inc.             Technology
61    SPGI                     S&P Global Inc.     Financial Services
72     SYK                 Stryker Corporation             Healthcare
80     BLK                     BlackRock, Inc.     Financial Services
86     BSX       Boston Scientific Corporation             Healthcare
93     KKR                      KKR & Co. Inc.     Financial Services
98     MDT                       Medtronic plc             Healthcare
117    MCO                 Moody's Corporation     Financial Services
165    COF   Capital One Financial Corporation     Financial Services
176     EW    Edwards Lifesciences Corporation             Healthcare
213   DXCM                        DexCom, Inc.             Healthcare
216    RCL        Royal Caribbean Cruises Ltd.      Consumer Cyclical
257    DFS         Discover Financial Services     Financial Services
268    EFX                        Equifax Inc.            Industrials
279   CSGP                  CoStar Group, Inc.            Real Estate
296   ANSS                         ANSYS, Inc.             Technology
298   EBAY                           eBay Inc.      Consumer Cyclical
311   TROW           T. Rowe Price Group, Inc.     Financial Services
318    CCL          Carnival Corporation & plc      Consumer Cyclical
332    ZBH        Zimmer Biomet Holdings, Inc.             Healthcare
335    TYL            Tyler Technologies, Inc.             Technology
336    STE                          STERIS plc             Healthcare
338    LYV     Live Nation Entertainment, Inc. Communication Services
363    SYF                 Synchrony Financial     Financial Services
381   HOLX                       Hologic, Inc.             Healthcare
382    COO          The Cooper Companies, Inc.             Healthcare
384   EXPE                 Expedia Group, Inc.      Consumer Cyclical
410    GEN                    Gen Digital Inc.             Technology
436   PODD                 Insulet Corporation             Healthcare
450    BEN            Franklin Resources, Inc.     Financial Services
472    TFX               Teleflex Incorporated             Healthcare
490   NCLH Norwegian Cruise Line Holdings Ltd.      Consumer Cyclical
500   ETSY                          Etsy, Inc.      Consumer Cyclical
501    IVZ                        Invesco Ltd.     Financial Services
```
</div>
</details>

### Cluster 3

<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                         Longname             Sector
13     WMT                     Walmart Inc. Consumer Defensive
21    COST     Costco Wholesale Corporation Consumer Defensive
25    ABBV                      AbbVie Inc.         Healthcare
55      PM Philip Morris International Inc. Consumer Defensive
111     MO               Altria Group, Inc. Consumer Defensive
113    HCA             HCA Healthcare, Inc.         Healthcare
208    STZ       Constellation Brands, Inc. Consumer Defensive
209    KDP            Keurig Dr Pepper Inc. Consumer Defensive
227     KR                   The Kroger Co. Consumer Defensive
297     DG       Dollar General Corporation Consumer Defensive
328   DLTR                Dollar Tree, Inc. Consumer Defensive
350   BF-B         Brown-Forman Corporation Consumer Defensive
386     LH            Labcorp Holdings Inc.         Healthcare
406    DGX   Quest Diagnostics Incorporated         Healthcare
430    DPZ             Domino's Pizza, Inc.  Consumer Cyclical
456    UHS  Universal Health Services, Inc.         Healthcare
457    DVA                      DaVita Inc.         Healthcare
467     LW       Lamb Weston Holdings, Inc. Consumer Defensive
468    TAP    Molson Coors Beverage Company Consumer Defensive
```
</div>
</details>

We can notice that most of the companies in this cluster are in the comsumer defensive sector.

### Cluster 4

<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                                     Longname                 Sector
14       V                                    Visa Inc.     Financial Services
17      MA                      Mastercard Incorporated     Financial Services
47     DIS                      The Walt Disney Company Communication Services
49      GE                                 GE Aerospace            Industrials
58   CMCSA                          Comcast Corporation Communication Services
64    UBER                      Uber Technologies, Inc.             Technology
70    BKNG                        Booking Holdings Inc.      Consumer Cyclical
76     TJX                      The TJX Companies, Inc.      Consumer Cyclical
102   ABNB                                 Airbnb, Inc.      Consumer Cyclical
105   SBUX                        Starbucks Corporation      Consumer Cyclical
116    ZTS                                  Zoetis Inc.             Healthcare
137    MAR                 Marriott International, Inc.      Consumer Cyclical
151   PYPL                        PayPal Holdings, Inc.     Financial Services
169    HLT               Hilton Worldwide Holdings Inc.      Consumer Cyclical
190   ROST                            Ross Stores, Inc.      Consumer Cyclical
204   CHTR                 Charter Communications, Inc. Communication Services
219    FIS Fidelity National Information Services, Inc.             Technology
234   IDXX                     IDEXX Laboratories, Inc.             Healthcare
256     EL              The Estée Lauder Companies Inc.     Consumer Defensive
277    LVS                        Las Vegas Sands Corp.      Consumer Cyclical
310    GPN                         Global Payments Inc.            Industrials
324   AXON                        Axon Enterprise, Inc.            Industrials
325    WST           West Pharmaceutical Services, Inc.             Healthcare
326   FSLR                            First Solar, Inc.             Technology
347    WBD                 Warner Bros. Discovery, Inc. Communication Services
370   ALGN                       Align Technology, Inc.             Healthcare
387    OMC                           Omnicom Group Inc. Communication Services
402   FOXA                              Fox Corporation Communication Services
403    FOX                              Fox Corporation Communication Services
404   ZBRA               Zebra Technologies Corporation             Technology
415   NWSA                             News Corporation Communication Services
416    NWS                             News Corporation Communication Services
425   ENPH                         Enphase Energy, Inc.             Technology
428    MGM                    MGM Resorts International      Consumer Cyclical
434   TRMB                                 Trimble Inc.             Technology
444    HST                  Host Hotels & Resorts, Inc.            Real Estate
448    JBL                                   Jabil Inc.             Technology
466    IPG     The Interpublic Group of Companies, Inc. Communication Services
470    ALB                        Albemarle Corporation        Basic Materials
482   WYNN                        Wynn Resorts, Limited      Consumer Cyclical
484   MTCH                            Match Group, Inc. Communication Services
485   PAYC                        Paycom Software, Inc.             Technology
495    HAS                                 Hasbro, Inc.      Consumer Cyclical
497   PARA                             Paramount Global Communication Services
498    CZR                  Caesars Entertainment, Inc.      Consumer Cyclical
```
</div>
</details>

### Cluster 5

<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                                    Longname             Sector
18      PG                The Procter & Gamble Company Consumer Defensive
27      KO                       The Coca-Cola Company Consumer Defensive
32     PEP                               PepsiCo, Inc. Consumer Defensive
40     MCD                      McDonald's Corporation  Consumer Cyclical
68     RTX                             RTX Corporation        Industrials
73     PGR                 The Progressive Corporation Financial Services
88     LMT                 Lockheed Martin Corporation        Industrials
92     MMC            Marsh & McLennan Companies, Inc. Financial Services
94      CB                               Chubb Limited Financial Services
106     WM                      Waste Management, Inc.        Industrials
107   MDLZ                Mondelez International, Inc. Consumer Defensive
118     CL                   Colgate-Palmolive Company Consumer Defensive
119     GD                General Dynamics Corporation        Industrials
132    CME                              CME Group Inc. Financial Services
147    NOC                Northrop Grumman Corporation        Industrials
148    AON                                     Aon plc Financial Services
150    RSG                     Republic Services, Inc.        Industrials
152   ORLY                   O'Reilly Automotive, Inc.  Consumer Cyclical
156    AJG                   Arthur J. Gallagher & Co. Financial Services
174    AFL                          Aflac Incorporated Financial Services
179    AZO                              AutoZone, Inc.  Consumer Cyclical
180   MNST                Monster Beverage Corporation Consumer Defensive
183    AIG          American International Group, Inc. Financial Services
188    KMB                  Kimberly-Clark Corporation Consumer Defensive
197    TRV               The Travelers Companies, Inc. Financial Services
205    ALL                    The Allstate Corporation Financial Services
211    LHX                 L3Harris Technologies, Inc.        Industrials
222    KHC                     The Kraft Heinz Company Consumer Defensive
225   VRSK                      Verisk Analytics, Inc.        Industrials
235    HSY                         The Hershey Company Consumer Defensive
251   ACGL                     Arch Capital Group Ltd. Financial Services
253    GIS                         General Mills, Inc. Consumer Defensive
254    YUM                           Yum! Brands, Inc.  Consumer Cyclical
284    HIG The Hartford Financial Services Group, Inc. Financial Services
299    WTW Willis Towers Watson Public Limited Company Financial Services
305    BRO                         Brown & Brown, Inc. Financial Services
314    CHD                   Church & Dwight Co., Inc. Consumer Defensive
321    ROL                               Rollins, Inc.  Consumer Cyclical
353   LDOS                       Leidos Holdings, Inc.         Technology
360      K                                   Kellanova Consumer Defensive
361    MKC           McCormick & Company, Incorporated Consumer Defensive
362    WRB                   W. R. Berkley Corporation Financial Services
364   CBOE                   Cboe Global Markets, Inc. Financial Services
390    HRL                    Hormel Foods Corporation Consumer Defensive
391      L                           Loews Corporation Financial Services
399    CLX                          The Clorox Company Consumer Defensive
400     EG                         Everest Group, Ltd. Financial Services
429    CAG                        Conagra Brands, Inc. Consumer Defensive
433    CPB                       Campbell Soup Company Consumer Defensive
447    SJM                   The J. M. Smucker Company Consumer Defensive
474    HII         Huntington Ingalls Industries, Inc.        Industrials
496     GL                             Globe Life Inc. Financial Services
```
</div>
</details>

### Cluster 6

<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                            Longname                 Sector
3     NVDA                  NVIDIA Corporation             Technology
11    AVGO                       Broadcom Inc.             Technology
19    ORCL                  Oracle Corporation             Technology
29     AMD        Advanced Micro Devices, Inc.             Technology
35    QCOM               QUALCOMM Incorporated             Technology
36     ACN                       Accenture plc             Technology
39    CSCO                 Cisco Systems, Inc.             Technology
41     TXN      Texas Instruments Incorporated             Technology
43    INTU                         Intuit Inc.             Technology
50    AMAT             Applied Materials, Inc.             Technology
65    INTC                   Intel Corporation             Technology
75      MU             Micron Technology, Inc.             Technology
81    LRCX            Lam Research Corporation             Technology
84     ADI                Analog Devices, Inc.             Technology
91    PANW            Palo Alto Networks, Inc.             Technology
96    ANET               Arista Networks, Inc.             Technology
97    KLAC                     KLA Corporation             Technology
110   SNPS                      Synopsys, Inc.             Technology
122   CDNS        Cadence Design Systems, Inc.             Technology
129   CRWD          CrowdStrike Holdings, Inc.             Technology
141   NXPI             NXP Semiconductors N.V.             Technology
175   ADSK                      Autodesk, Inc.             Technology
192   MCHP   Microchip Technology Incorporated             Technology
196   SMCI          Super Micro Computer, Inc.             Technology
210   FTNT                      Fortinet, Inc.             Technology
226   MPWR      Monolithic Power Systems, Inc.             Technology
241     EA                Electronic Arts Inc. Communication Services
273     ON        ON Semiconductor Corporation             Technology
275    CDW                     CDW Corporation             Technology
285    RMD                         ResMed Inc.             Healthcare
300    HPE  Hewlett Packard Enterprise Company             Technology
306   TTWO Take-Two Interactive Software, Inc. Communication Services
308   NTAP                        NetApp, Inc.             Technology
329    TER                      Teradyne, Inc.             Technology
330    WDC         Western Digital Corporation             Technology
344    STX     Seagate Technology Holdings plc             Technology
349    PTC                            PTC Inc.             Technology
379   SWKS            Skyworks Solutions, Inc.             Technology
453   JNPR              Juniper Networks, Inc.             Technology
454   EPAM                  EPAM Systems, Inc.             Technology
463   QRVO                         Qorvo, Inc.             Technology
476   FFIV                            F5, Inc.             Technology
491    DAY                        Dayforce Inc             Technology
```
</div>
</details>

Tech companies are clustered in this cluster.

### Cluster 7

<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                                    Longname             Sector
15     XOM                     Exxon Mobil Corporation             Energy
26     CVX                         Chevron Corporation             Energy
34     LIN                                   Linde plc    Basic Materials
51     CAT                            Caterpillar Inc.        Industrials
53     IBM International Business Machines Corporation         Technology
71     COP                              ConocoPhillips             Energy
77     ETN                       Eaton Corporation plc        Industrials
89      BA                          The Boeing Company        Industrials
95      DE                             Deere & Company        Industrials
126     TT                      Trane Technologies plc        Industrials
128    EOG                         EOG Resources, Inc.             Energy
133    SLB                        Schlumberger Limited             Energy
139    TDG                TransDigm Group Incorporated        Industrials
144    EMR                        Emerson Electric Co.        Industrials
145    FCX                       Freeport-McMoRan Inc.    Basic Materials
154   CARR                  Carrier Global Corporation        Industrials
155    CEG            Constellation Energy Corporation          Utilities
157    PSX                                 Phillips 66             Energy
158    APD            Air Products and Chemicals, Inc.    Basic Materials
159    MPC              Marathon Petroleum Corporation             Energy
164   PCAR                                  PACCAR Inc        Industrials
166    OXY            Occidental Petroleum Corporation             Energy
171    WMB                The Williams Companies, Inc.             Energy
186    VLO                   Valero Energy Corporation             Energy
187    OKE                                 ONEOK, Inc.             Energy
189    URI                        United Rentals, Inc.        Industrials
191    KMI                         Kinder Morgan, Inc.             Energy
194    HES                            Hess Corporation             Energy
200    JCI          Johnson Controls International plc        Industrials
223    CMI                                Cummins Inc.        Industrials
232    NUE                           Nucor Corporation    Basic Materials
237   CTVA                               Corteva, Inc.    Basic Materials
239    DOW                                    Dow Inc.    Basic Materials
240    PWR                       Quanta Services, Inc.        Industrials
246   FANG                    Diamondback Energy, Inc.             Energy
250    BKR                        Baker Hughes Company             Energy
264     DD                     DuPont de Nemours, Inc.    Basic Materials
266   GRMN                                 Garmin Ltd.         Technology
269    HWM                       Howmet Aerospace Inc.        Industrials
270    ADM              Archer-Daniels-Midland Company Consumer Defensive
272    LYB              LyondellBasell Industries N.V.    Basic Materials
278    HAL                         Halliburton Company             Energy
280   TRGP                       Targa Resources Corp.             Energy
283    DVN                    Devon Energy Corporation             Energy
303    VST                                Vistra Corp.          Utilities
355   STLD                        Steel Dynamics, Inc.    Basic Materials
357   CTRA                         Coterra Energy Inc.             Energy
358   HUBB                        Hubbell Incorporated        Industrials
375    TDY          Teledyne Technologies Incorporated         Technology
378      J                       Jacobs Solutions Inc.        Industrials
388    AVY                  Avery Dennison Corporation  Consumer Cyclical
394    TXT                                Textron Inc.        Industrials
395    PKG            Packaging Corporation of America  Consumer Cyclical
408    MRO                    Marathon Oil Corporation             Energy
411     IP                 International Paper Company  Consumer Cyclical
412     BG                             Bunge Global SA Consumer Defensive
413    EQT                             EQT Corporation             Energy
418    NRG                            NRG Energy, Inc.          Utilities
420     CE                        Celanese Corporation    Basic Materials
440     CF                CF Industries Holdings, Inc.    Basic Materials
461    APA                             APA Corporation             Energy
464    EMN                    Eastman Chemical Company    Basic Materials
469   ALLE                                Allegion plc        Industrials
477    MOS                          The Mosaic Company    Basic Materials
488    AIZ                              Assurant, Inc. Financial Services
502    FMC                             FMC Corporation    Basic Materials
```
</div>
</details>


### Cluster 8
<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                                     Longname             Sector
62     NEE                         NextEra Energy, Inc.          Utilities
85     PLD                               Prologis, Inc.        Real Estate
100    AMT                   American Tower Corporation        Real Estate
108     SO                         The Southern Company          Utilities
114    DUK                      Duke Energy Corporation          Utilities
127   EQIX                                Equinix, Inc.        Real Estate
149   WELL                               Welltower Inc.        Real Estate
160    SPG                   Simon Property Group, Inc.        Real Estate
170    NEM                          Newmont Corporation    Basic Materials
173    PSA                               Public Storage        Real Estate
178    DLR                   Digital Realty Trust, Inc.        Real Estate
181      O                    Realty Income Corporation        Real Estate
184    AEP        American Electric Power Company, Inc.          Utilities
185    SRE                                       Sempra          Utilities
193    PCG                             PG&E Corporation          Utilities
201   MRNA                                Moderna, Inc.         Healthcare
206    CCI                            Crown Castle Inc.        Real Estate
217      D                        Dominion Energy, Inc.          Utilities
242    PEG Public Service Enterprise Group Incorporated          Utilities
245    SYY                            Sysco Corporation Consumer Defensive
248    EXR                     Extra Space Storage Inc.        Real Estate
249    EXC                           Exelon Corporation          Utilities
267     ED                    Consolidated Edison, Inc.          Utilities
271   VICI                         VICI Properties Inc.        Real Estate
274    XEL                             Xcel Energy Inc.          Utilities
286    AVB                  AvalonBay Communities, Inc.        Real Estate
290    EIX                         Edison International          Utilities
291    IRM                   Iron Mountain Incorporated        Real Estate
295    AWK           American Water Works Company, Inc.          Utilities
302    EQR                           Equity Residential        Real Estate
309    WEC                       WEC Energy Group, Inc.          Utilities
319    DTE                           DTE Energy Company          Utilities
320   KEYS                  Keysight Technologies, Inc.         Technology
323    ETR                          Entergy Corporation          Utilities
331     FE                            FirstEnergy Corp.          Utilities
334   SBAC               SBA Communications Corporation        Real Estate
339   INVH                        Invitation Homes Inc.        Real Estate
341    ARE        Alexandria Real Estate Equities, Inc.        Real Estate
343    VTR                                 Ventas, Inc.        Real Estate
345     ES                            Eversource Energy          Utilities
346    TSN                            Tyson Foods, Inc. Consumer Defensive
348    PPL                              PPL Corporation          Utilities
356    AEE                           Ameren Corporation          Utilities
369   BALL                             Ball Corporation  Consumer Cyclical
371    ESS                   Essex Property Trust, Inc.        Real Estate
376    ATO                     Atmos Energy Corporation          Utilities
377    CNP                     CenterPoint Energy, Inc.          Utilities
380    CMS                       CMS Energy Corporation          Utilities
392    MAA      Mid-America Apartment Communities, Inc.        Real Estate
396    DRI                     Darden Restaurants, Inc.  Consumer Cyclical
417    UDR                                    UDR, Inc.        Real Estate
421    DOC                  Healthpeak Properties, Inc.        Real Estate
423   AMCR                                    Amcor plc  Consumer Cyclical
426    KIM                     Kimco Realty Corporation        Real Estate
432    LNT                   Alliant Energy Corporation          Utilities
435     NI                                NiSource Inc.          Utilities
445   EVRG                                 Evergy, Inc.          Utilities
451    REG                  Regency Centers Corporation        Real Estate
452    AES                          The AES Corporation          Utilities
459    CPT                        Camden Property Trust        Real Estate
471   CTLT                               Catalent, Inc.         Healthcare
483    PNW            Pinnacle West Capital Corporation          Utilities
486    FRT              Federal Realty Investment Trust        Real Estate
493   MKTX                    MarketAxess Holdings Inc. Financial Services
```
</div>
</details>

This cluster is composed mainly of companies in the utilities sector and the real estate sector.

### Cluster 9
<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                        Longname                 Sector
20     JNJ               Johnson & Johnson             Healthcare
33    TMUS               T-Mobile US, Inc. Communication Services
44    AMGN                      Amgen Inc.             Healthcare
46      VZ     Verizon Communications Inc. Communication Services
52     PFE                     Pfizer Inc.             Healthcare
67       T                       AT&T Inc. Communication Services
82    REGN Regeneron Pharmaceuticals, Inc.             Healthcare
104   GILD           Gilead Sciences, Inc.             Healthcare
109    BMY    Bristol-Myers Squibb Company             Healthcare
140    TGT              Target Corporation     Consumer Defensive
143    BDX   Becton, Dickinson and Company             Healthcare
265   BIIB                     Biogen Inc.             Healthcare
442   INCY              Incyte Corporation             Healthcare
478    WBA  Walgreens Boots Alliance, Inc.             Healthcare
```
</div>
</details>

This cluster is mainly composed of healthcare companies.

### Cluster 10
<details>
  <summary>Symbols</summary>
<div markdown="1">
```
    Symbol                            Longname     Sector
9      LLY               Eli Lilly and Company Healthcare
16     UNH     UnitedHealth Group Incorporated Healthcare
24     MRK                   Merck & Co., Inc. Healthcare
74    VRTX Vertex Pharmaceuticals Incorporated Healthcare
83     ELV               Elevance Health, Inc. Healthcare
101     CI                     The Cigna Group Healthcare
123    MCK                McKesson Corporation Healthcare
125    CVS              CVS Health Corporation Healthcare
195    HUM                         Humana Inc. Healthcare
215    COR                       Cencora, Inc. Healthcare
258    CNC                 Centene Corporation Healthcare
327    CAH               Cardinal Health, Inc. Healthcare
398    MOH             Molina Healthcare, Inc. Healthcare
```
</div>
</details>

Only healthcare companies are clustered in this group.

---

Although the resulted clusters are not very accurate, we can notice that these clusters are appealing, and reflects the sectors.
This shows that the proposed method is useful and has its potential.