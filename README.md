# Employment Optimization Problem

> *This is a project for the course of Operations Research (1218183) at Amirkabir University of Technology.*

## Future Work and Improvements
I plan to improve the project in the following ways:
- **Create more modular parts for the data manipulation part:** Use the same code with more diverse datasets.
- **Modularize the optimization model:** Use the model for similar employment problems with a different context.
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

Eventually, The goal is to minimize the personel costs of the company by optimizing the number of employees and their work hours.