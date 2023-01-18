# Employment Optimisation Problem

> *This is a project for the course Linear Optimization (1218183) at Amirkabir University of Technology.*

## Future Work and Improvements
I plan to improve the project in the following ways:
- **Create more modular parts for the data manipulation part:** Use the same code with more diverse datasets.
- **Modularize the optimisation model:** Use the model for similar employment problems with a different context.
- **Fully migrate the problem to English:** Allow the code to be universally used and understood.
- **Add support for more types of food:** EX: Prepared food, bought food, etc.


## The Problem
A catering company has a number of employees with specific job titles *(EX: `Chef_1`, `Chef_2`, `Butcher`, ...)* These employees follow below conditions:

1. Each employee must work 8 hours per weekday. _(Everyday except Thursdays & Fridays)_
1. Each employee can work overtime on any day of the week.
1. The company pays employees a monthly base wage.
1. Overtime wage is 1.4 times the base wage.

Current overtime work hours for each employee is given in a table like below:

|Date\Employee| emp_1 | emp_2 | ... | emp_n |
|:--------|:-------:|:-------:|:-----:|:-------:|
| date_1 |w_1_1|w_1_2|...|w_1_n|
| date_2 |w_2_1|w_2_2|...|w_2_n|
| ...    |...|...|...|...|
| date_m |w_m_1|w_m_2|...|w_m_n|

Where _`w_i_j`_ is the overtime work hours of employee _`j`_ on date _`i`_.

Looking at the table, it seems that the overtime work hours are too high and it leads to extra costs for the company.  
The directors and the HR department have agreed to recruit new employees to reduce the overtime work hours. However, the fillowing conditions must be met:

- No current employee can be fired.
- New employees will be considered to all the conditions for the current employees.
- A new job title cannot be introduced.
- New employees will be payed the average base wage of the current employees with the same job title.

The company has to supply a number of food potions per day which is given in a table like below:

|Date\Portions| food_1 | food_2 |
|:--------|:-------:|:-------:|
| date_1 |p_1_1|p_1_2|
| date_2 |p_2_1|p_2_2|
| ...    |...|...|
| date_m |p_m_1|p_m_2|

Where _`p_i_j`_ is the number of portions of food type _`j`_ on date _`i`_.

The following conditions are given for the demands:
- The supply must exactly meet the demand. since _no food can be stored_.
- Each month has the same menu which is set at the beginnig of the year.
- Both types of foods must be cooked. _(No food can be bought as of now)_

Eventually, The goal is to minimize the personel costs of the company by optimising the number of employees and their work hours.

## Model

### Parameters
The following parameters are used in the model:
- **The time period**: $D = \set{\text{2022-07-23},\dots, \text{2022-08-22} }$
- **Employees**: $Emp = \set{\text{Employee numbers}}$
- **Job Groups**: $JG = \set{\text{All Unique Job Groups}}$

### Variables and Data Structures
To model the problem, the following variable structure is used:
- $x_{i,d}$ is the basic work hour of employee $i$ on day $d$.
- $x \prime_{i,d}$ is the overtime work hour of employee $i$ on day $d$.
- $y_{g,d}$ is the required basic work hour of job group $g$ on day $d$.
- $y \prime_{g,d}$ is the required overtime work hour of job group $g$ on day $d$.
- $w_i$ is the base wage of employee $i$.
- $w \prime_i$ is the overtime wage of employee $i$.
- $w_g$ is the average base wage of job group $g$. (This is the new employee's base wage)
- $w \prime_g$ is the average overtime wage of job group $g$. (This is the new employee's overtime wage)
- $p_{d,j}$ is the number of demanded portions of food type $j$ on day $d$.

**There are 3 job categories:**
- **Food Related**: These job groups have direct relationship with cooked food.
- **Distribution Related**: These job groups have direct relationship with the distribution of food. (Both cooked and bought)
- **HR Related**: These job groups have direct relationship with the HR department. (Hiring, Managment, etc.)

> _**Job groups**_ are employees' _**job titles**_. (Ex: `butcher`, `chef`, `waiter`, etc.) However, employees don't have impact on every part of the supply chain; meaning, their work hours are <ins>_not_</ins> affected by changes in the demand. Therefore, job groups are categorized based on their influence on the supply.  

### Objective Function
The objective function is defined as follows:
```math
\min z = \sum_{d\in D} \sum_{i\in Emp} (x_{i,d} \cdot w_i + x \prime_{i,d} \cdot w\prime_i)+ \sum_{d \in D} \sum_{g \in Job\space Groups} (y_{g,d}\cdot w_g + y\prime_{g,d}\cdot w\prime_g)\\
```

> _The reason to have each individual current employee as a desicion variable is that their base wage is different and their work hours are not optimised. Therefore, initially, the model optimises the current employees' work hours and then uses the remaining demand to hire new employees._

### Constraints
The model has fairly simple constraints as the supply and demand are equal and the demand is fixed. The constraints are defined as follows:

```math
\begin{equation}
x_{i,d}=
    \begin{cases}
        480 & d \in D_{weekdays}\\
        0 & d \in D_{weekends}
    \end{cases}
\end{equation}
,\forall i\in Emp, \forall d\in D
``` 
```math
\begin{equation}
y_{g,d} = 0
\end{equation}
,\forall g\in JG, \forall d\in D_{weekends}
```
```math
\begin{equation}
y_{g,d} \geq 0
\end{equation}
,\forall g\in JG, \forall d\in D_{weekdays}
```
```math
\begin{equation}
\sum_{i \in {Emp_{g}}} (x_{i,d} + x\prime_{i,d}) +
y_{g,d}+y\prime_{g,d}
\geq \delta_{g,d}
,\forall d \in D,\ \forall g \in JG
\end{equation}
```


#### Demands
For each job category, a cost $\theta$ is calculated for each day:  
```math
\begin{equation}
\theta_{g,d} = 
    \begin{cases}
        \sum_{i \in Emp_g}(x_{i,d}+x\prime_{i,d}) \over \sum_{j \in Cooked\ Food}p_{d,j}& g \in JG_{Food\space Related}\\\\

        \sum_{i \in Emp_g}(x_{i,d}+x\prime_{i,d}) \over \sum_{j \in Cooked\ Food}p_{d,j} + \sum_{j \in Distributed\ Food}p_{d,j}& g \in JG_{Distribution\space Related}\\\\

        \sum_{i \in Emp_g}(x_{i,d}+x\prime_{i,d}) \over \sum_{i \in Emp}(x_{i,d}+x\prime_{i,d})& g \in JG_{HR\space Related}
    \end{cases}
\end{equation}
```

The demands are given in the following table:  
|Demand|Description|Formula|
|:-----:|:----------|:------|
|$\delta_{Food,d}$|Food related demand on day $d$|$(\sum_{j \in Cooked\ Food}p_{d,j}) \times \theta_{Food,d}$|
|$\delta_{Distribution,d}$|Distribution related demand on day $d$|$(\sum_{j}p_{d,j}) \times \theta_{Distribution,d}$|
|$\delta_{HR,d}$|HR related demand on day $d$|$(\sum_{i\in Emp} (x_{i,d}+ x \prime_{i,d})+ \sum_{g \in Job\space Groups} (y_{g,d} + y\prime_{g,d}))\times \theta_{HR,d}$ |


> ❗️**Important**: Bear in mind that the $\theta_{g,d}$ values are constants and can not be changed. The summations in $\theta_{g,d}$ shall not be mistaken with the summations in the constraints. It is wrong to cancel them in $\delta_{g,d}$ as the model is not allowed to modify the $\theta_{g,d}$ values.

### Results
Below are the employees' work hours before optimisation:

<div>            <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.5/MathJax.js?config=TeX-AMS-MML_SVG"></script><script type="text/javascript">if (window.MathJax && window.MathJax.Hub && window.MathJax.Hub.Config) {window.MathJax.Hub.Config({SVG: {font: "STIX-Web"}});}</script>                <script type="text/javascript">window.PlotlyConfig = {MathJaxConfig: \'local\'};</script>        <script src="https://cdn.plot.ly/plotly-2.16.1.min.js"></script>                <div id="a874230c-e38d-46fa-a8dd-b752200b2474" class="plotly-graph-div" style="height:100%; width:100%;"></div>            <script type="text/javascript">                                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("a874230c-e38d-46fa-a8dd-b752200b2474")) {                    Plotly.newPlot(                        "a874230c-e38d-46fa-a8dd-b752200b2474",                        [{"name":"\\u0622\\u0634\\u067e\\u0632 1","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[1224,1800,552,1633,567,2237,2345,1778,768,2236,363,932,1325,2621,821,2356,3187,1786,837,744,833,2323,1639,1380,890,823,2245,2263,708,1473,1003],"type":"scatter"},{"name":"\\u0622\\u0634\\u067e\\u0632 2","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[6338,4787,5981,6748,6263,6333,6067,5949,6756,4808,5390,6043,6709,10156,5903,15416,13753,5091,3855,9184,7071,6986,10634,7684,6444,9669,12362,4713,5147,6484,6305],"type":"scatter"},{"name":"\\u0633\\u0631 \\u0622\\u0634\\u067e\\u0632","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[454,480,627,475,375,591,583,539,527,396,527,518,778,786,958,1705,1125,797,642,1576,2241,788,641,1145,1129,787,2040,0,453,768,784],"type":"scatter"},{"name":"\\u0633\\u0631 \\u0634\\u06cc\\u0641\\u062a","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[422,368,807,428,423,166,532,174,293,311,424,282,887,967,860,1214,2901,426,285,522,942,1156,2683,1519,1527,1160,1296,164,429,298,429],"type":"scatter"},{"name":"\\u0633\\u0631 \\u0634\\u06cc\\u0641\\u062a \\u0622\\u0645\\u0627\\u062f\\u0647 \\u0633\\u0627\\u0632\\u06cc","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[119,275,277,304,24,0,788,316,24,386,234,122,787,509,164,109,587,241,281,587,0,394,389,272,128,428,1104,586,139,322,839],"type":"scatter"},{"name":"\\u0633\\u0631 \\u0634\\u06cc\\u0641\\u062a \\u0628\\u0631\\u0646\\u062c","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[122,122,122,268,200,587,0,122,126,177,387,284,1157,51,244,1178,1042,292,238,1214,0,434,1327,792,1179,908,775,631,245,441,394],"type":"scatter"},{"name":"\\u0633\\u0631\\u062f\\u062e\\u0627\\u0646\\u0647 \\u062f\\u0627\\u0631","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[332,240,129,382,137,0,489,131,382,130,0,132,580,753,131,0,778,121,144,777,753,0,372,126,393,128,0,752,243,129,139],"type":"scatter"},{"name":"\\u0633\\u0631\\u067e\\u0631\\u0633\\u062a \\u062a\\u0648\\u0644\\u06cc\\u062f","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],"type":"scatter"},{"name":"\\u0642\\u0635\\u0627\\u0628","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[2480,1351,1256,1965,1426,1238,4040,1329,1071,1036,1244,797,1785,2726,1036,1449,3290,953,1964,1516,5384,736,2685,2029,1754,1524,2259,3536,1177,1584,2199],"type":"scatter"},{"name":"\\u06a9\\u0645\\u06a9 \\u0622\\u0634\\u067e\\u0632","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[4945,4484,4007,5269,4649,6448,8631,4957,5059,3801,4768,3064,9962,5392,4825,10725,10583,3016,6223,5922,9312,3495,6975,6171,5701,6547,14909,3588,3600,5199,7230],"type":"scatter"},{"name":"\\u06a9\\u0645\\u06a9 \\u0627\\u0646\\u0628\\u0627\\u0631\\u062f\\u0627\\u0631","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[479,175,584,627,338,1625,632,321,222,795,442,751,468,1226,940,372,1187,468,792,851,1911,386,648,56,57,230,1611,638,179,489,71],"type":"scatter"},{"name":"\\u0627\\u0646\\u0628\\u0627\\u0631\\u062f\\u0627\\u0631","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[588,462,42,186,590,1191,782,726,253,298,160,43,602,604,163,1036,481,288,179,590,793,590,162,470,164,579,1223,788,171,184,435],"type":"scatter"},{"name":"\\u0627\\u067e\\u0631\\u0627\\u062a\\u0648\\u0631 \\u0627\\u0646\\u0628\\u0627\\u0631","x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[154,305,167,12,160,0,772,316,13,165,162,313,773,0,9,784,583,161,310,0,781,156,0,165,170,165,0,787,155,13,163],"type":"scatter"}],                        {"template":{"data":{"histogram2dcontour":[{"type":"histogram2dcontour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"choropleth":[{"type":"choropleth","colorbar":{"outlinewidth":0,"ticks":""}}],"histogram2d":[{"type":"histogram2d","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmap":[{"type":"heatmap","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"heatmapgl":[{"type":"heatmapgl","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"contourcarpet":[{"type":"contourcarpet","colorbar":{"outlinewidth":0,"ticks":""}}],"contour":[{"type":"contour","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"surface":[{"type":"surface","colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]}],"mesh3d":[{"type":"mesh3d","colorbar":{"outlinewidth":0,"ticks":""}}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"parcoords":[{"type":"parcoords","line":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolargl":[{"type":"scatterpolargl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"scattergeo":[{"type":"scattergeo","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterpolar":[{"type":"scatterpolar","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"scattergl":[{"type":"scattergl","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatter3d":[{"type":"scatter3d","line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattermapbox":[{"type":"scattermapbox","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scatterternary":[{"type":"scatterternary","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"scattercarpet":[{"type":"scattercarpet","marker":{"colorbar":{"outlinewidth":0,"ticks":""}}}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}],"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"pie":[{"automargin":true,"type":"pie"}]},"layout":{"autotypenumbers":"strict","colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"hovermode":"closest","hoverlabel":{"align":"left"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"bgcolor":"#E5ECF6","angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"ternary":{"bgcolor":"#E5ECF6","aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]]},"xaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"yaxis":{"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","automargin":true,"zerolinewidth":2},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white","gridwidth":2}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"geo":{"bgcolor":"white","landcolor":"#E5ECF6","subunitcolor":"white","showland":true,"showlakes":true,"lakecolor":"white"},"title":{"x":0.05},"mapbox":{"style":"light"}}},"title":{"text":"Food Related Employees\' Overtime Work"},"xaxis":{"title":{"text":"Date"}},"yaxis":{"title":{"text":"Minutes"}}},                        {"responsive": true}                    )                };                            </script>        </div>