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

<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
<div>                            <div id="dfa693da-b300-4bbf-b94c-ab2f33f426d1" class="plotly-graph-div" style="height:100%; width:100%;"></div>            <script type="text/javascript">                                    window.PLOTLYENV=window.PLOTLYENV || {};                                    if (document.getElementById("dfa693da-b300-4bbf-b94c-ab2f33f426d1")) {                    Plotly.newPlot(                        "dfa693da-b300-4bbf-b94c-ab2f33f426d1",                        [{"name":"Optimized Total","opacity":1,"x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[94560.0,94560.0,94560.0,94560.0,94560.0,0.0,0.0,94560.0,94560.0,94560.0,94560.0,94560.0,0.0,0.0,94560.0,94560.0,94560.0,94560.0,94560.0,32927.0,36374.0,94560.0,94560.0,94560.0,94560.0,94560.0,55198.0,24846.0,94560.0,94560.0,94560.0],"type":"scatter"},{"name":"Old total","opacity":0.4,"x":["2022-07-23T00:00:00","2022-07-24T00:00:00","2022-07-25T00:00:00","2022-07-26T00:00:00","2022-07-27T00:00:00","2022-07-28T00:00:00","2022-07-29T00:00:00","2022-07-30T00:00:00","2022-07-31T00:00:00","2022-08-01T00:00:00","2022-08-02T00:00:00","2022-08-03T00:00:00","2022-08-04T00:00:00","2022-08-05T00:00:00","2022-08-06T00:00:00","2022-08-07T00:00:00","2022-08-08T00:00:00","2022-08-09T00:00:00","2022-08-10T00:00:00","2022-08-11T00:00:00","2022-08-12T00:00:00","2022-08-13T00:00:00","2022-08-14T00:00:00","2022-08-15T00:00:00","2022-08-16T00:00:00","2022-08-17T00:00:00","2022-08-18T00:00:00","2022-08-19T00:00:00","2022-08-20T00:00:00","2022-08-21T00:00:00","2022-08-22T00:00:00"],"y":[88217,85409,85111,88857,85712,20416,25661,87218,86054,85099,84661,83841,25813,25791,86614,106904,110057,84200,86310,23483,30021,88004,98715,92369,90096,93508,39824,18446,83206,87944,90551],"type":"scatter"}],                        {"template":{"data":{"barpolar":[{"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"barpolar"}],"bar":[{"error_x":{"color":"#2a3f5f"},"error_y":{"color":"#2a3f5f"},"marker":{"line":{"color":"#E5ECF6","width":0.5},"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"bar"}],"carpet":[{"aaxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"baxis":{"endlinecolor":"#2a3f5f","gridcolor":"white","linecolor":"white","minorgridcolor":"white","startlinecolor":"#2a3f5f"},"type":"carpet"}],"choropleth":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"choropleth"}],"contourcarpet":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"contourcarpet"}],"contour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"contour"}],"heatmapgl":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmapgl"}],"heatmap":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"heatmap"}],"histogram2dcontour":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2dcontour"}],"histogram2d":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"histogram2d"}],"histogram":[{"marker":{"pattern":{"fillmode":"overlay","size":10,"solidity":0.2}},"type":"histogram"}],"mesh3d":[{"colorbar":{"outlinewidth":0,"ticks":""},"type":"mesh3d"}],"parcoords":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"parcoords"}],"pie":[{"automargin":true,"type":"pie"}],"scatter3d":[{"line":{"colorbar":{"outlinewidth":0,"ticks":""}},"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatter3d"}],"scattercarpet":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattercarpet"}],"scattergeo":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergeo"}],"scattergl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattergl"}],"scattermapbox":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scattermapbox"}],"scatterpolargl":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolargl"}],"scatterpolar":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterpolar"}],"scatter":[{"fillpattern":{"fillmode":"overlay","size":10,"solidity":0.2},"type":"scatter"}],"scatterternary":[{"marker":{"colorbar":{"outlinewidth":0,"ticks":""}},"type":"scatterternary"}],"surface":[{"colorbar":{"outlinewidth":0,"ticks":""},"colorscale":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"type":"surface"}],"table":[{"cells":{"fill":{"color":"#EBF0F8"},"line":{"color":"white"}},"header":{"fill":{"color":"#C8D4E3"},"line":{"color":"white"}},"type":"table"}]},"layout":{"annotationdefaults":{"arrowcolor":"#2a3f5f","arrowhead":0,"arrowwidth":1},"autotypenumbers":"strict","coloraxis":{"colorbar":{"outlinewidth":0,"ticks":""}},"colorscale":{"diverging":[[0,"#8e0152"],[0.1,"#c51b7d"],[0.2,"#de77ae"],[0.3,"#f1b6da"],[0.4,"#fde0ef"],[0.5,"#f7f7f7"],[0.6,"#e6f5d0"],[0.7,"#b8e186"],[0.8,"#7fbc41"],[0.9,"#4d9221"],[1,"#276419"]],"sequential":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]],"sequentialminus":[[0.0,"#0d0887"],[0.1111111111111111,"#46039f"],[0.2222222222222222,"#7201a8"],[0.3333333333333333,"#9c179e"],[0.4444444444444444,"#bd3786"],[0.5555555555555556,"#d8576b"],[0.6666666666666666,"#ed7953"],[0.7777777777777778,"#fb9f3a"],[0.8888888888888888,"#fdca26"],[1.0,"#f0f921"]]},"colorway":["#636efa","#EF553B","#00cc96","#ab63fa","#FFA15A","#19d3f3","#FF6692","#B6E880","#FF97FF","#FECB52"],"font":{"color":"#2a3f5f"},"geo":{"bgcolor":"white","lakecolor":"white","landcolor":"#E5ECF6","showlakes":true,"showland":true,"subunitcolor":"white"},"hoverlabel":{"align":"left"},"hovermode":"closest","mapbox":{"style":"light"},"paper_bgcolor":"white","plot_bgcolor":"#E5ECF6","polar":{"angularaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","radialaxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"scene":{"xaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"yaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"},"zaxis":{"backgroundcolor":"#E5ECF6","gridcolor":"white","gridwidth":2,"linecolor":"white","showbackground":true,"ticks":"","zerolinecolor":"white"}},"shapedefaults":{"line":{"color":"#2a3f5f"}},"ternary":{"aaxis":{"gridcolor":"white","linecolor":"white","ticks":""},"baxis":{"gridcolor":"white","linecolor":"white","ticks":""},"bgcolor":"#E5ECF6","caxis":{"gridcolor":"white","linecolor":"white","ticks":""}},"title":{"x":0.05},"xaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2},"yaxis":{"automargin":true,"gridcolor":"white","linecolor":"white","ticks":"","title":{"standoff":15},"zerolinecolor":"white","zerolinewidth":2}}},"title":{"text":"FOOD"},"xaxis":{"title":{"text":"Date"}},"yaxis":{"title":{"text":"Minutes"}}},                        {"responsive": true}                    )                };                            </script>        </div>
