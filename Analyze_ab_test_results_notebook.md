
## Analyze A/B Test Results

This project will assure you have mastered the subjects covered in the statistics lessons.  The hope is to have this project be as comprehensive of these topics as possible.  Good luck!

## Table of Contents
- [Introduction](#intro)
- [Part I - Probability](#probability)
- [Part II - A/B Test](#ab_test)
- [Part III - Regression](#regression)


<a id='intro'></a>
### Introduction

A/B tests are very commonly performed by data analysts and data scientists.  It is important that you get some practice working with the difficulties of these 

For this project, you will be working to understand the results of an A/B test run by an e-commerce website.  Your goal is to work through this notebook to help the company understand if they should implement the new page, keep the old page, or perhaps run the experiment longer to make their decision.



<a id='probability'></a>
#### Part I - Probability

To get started, let's import our libraries.


```python
import pandas as pd
import numpy as np
import random
import matplotlib.pyplot as plt
%matplotlib inline
#We are setting the seed to assure you get the same answers on quizzes as we set up
random.seed(42)
```

`1.` Now, read in the `ab_data.csv` data. Store it in `df`.  **Use your dataframe to answer the questions in Quiz 1 of the classroom.**

a. Read in the dataset and take a look at the top few rows here:


```python
df = pd.read_csv('ab_data.csv')
df.head(5)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>timestamp</th>
      <th>group</th>
      <th>landing_page</th>
      <th>converted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>851104</td>
      <td>2017-01-21 22:11:48.556739</td>
      <td>control</td>
      <td>old_page</td>
      <td>0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>804228</td>
      <td>2017-01-12 08:01:45.159739</td>
      <td>control</td>
      <td>old_page</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>661590</td>
      <td>2017-01-11 16:55:06.154213</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>853541</td>
      <td>2017-01-08 18:28:03.143765</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>864975</td>
      <td>2017-01-21 01:52:26.210827</td>
      <td>control</td>
      <td>old_page</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



b. Use the below cell to find the number of rows in the dataset.


```python
df.shape
```




    (294478, 5)



c. The number of unique users in the dataset.


```python
len(df['user_id'].unique())
```




    290584



d. The proportion of users converted.


```python
df['converted'].mean()
```




    0.11965919355605512



e. The number of times the `new_page` and `treatment` don't line up.


```python
df.query("group == 'treatment' and landing_page != 'new_page'").count()[0] + \
df.query("group == 'control' and landing_page == 'new_page'").count()[0]
```




    3893



f. Do any of the rows have missing values?


```python
df.info()
```

    <class 'pandas.core.frame.DataFrame'>
    RangeIndex: 294478 entries, 0 to 294477
    Data columns (total 5 columns):
    user_id         294478 non-null int64
    timestamp       294478 non-null object
    group           294478 non-null object
    landing_page    294478 non-null object
    converted       294478 non-null int64
    dtypes: int64(2), object(3)
    memory usage: 11.2+ MB
    

`2.` For the rows where **treatment** is not aligned with **new_page** or **control** is not aligned with **old_page**, we cannot be sure if this row truly received the new or old page.  Use **Quiz 2** in the classroom to provide how we should handle these rows.  

a. Now use the answer to the quiz to create a new dataset that meets the specifications from the quiz.  Store your new dataframe in **df2**.


```python
df2 = df.query("group == 'treatment' and landing_page == 'new_page'").append(
df.query("group == 'control' and landing_page == 'old_page'"))
```


```python
# Double Check all of the correct rows were removed - this should be 0
df2[((df2['group'] == 'treatment') == (df2['landing_page'] == 'new_page')) == False].shape[0]
```




    0



`3.` Use **df2** and the cells below to answer questions for **Quiz3** in the classroom.

a. How many unique **user_id**s are in **df2**?


```python
df2['user_id'].nunique()
```




    290584




```python
df2.shape
```




    (290585, 5)



b. There is one **user_id** repeated in **df2**.  What is it?


```python
a = df2['user_id'].drop_duplicates(keep = 'first')
b = df2['user_id'].drop_duplicates(keep = False)
```


```python
a.append(b).drop_duplicates(keep = False)
```




    1899    773192
    Name: user_id, dtype: int64




```python
df2['user_id'].value_counts().head()
```




    773192    2
    630732    1
    811737    1
    797392    1
    795345    1
    Name: user_id, dtype: int64



c. What is the row information for the repeat **user_id**? 


```python
df2.query("user_id == 773192")
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>timestamp</th>
      <th>group</th>
      <th>landing_page</th>
      <th>converted</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1899</th>
      <td>773192</td>
      <td>2017-01-09 05:37:58.781806</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
    </tr>
    <tr>
      <th>2893</th>
      <td>773192</td>
      <td>2017-01-14 02:55:59.590927</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



d. Remove **one** of the rows with a duplicate **user_id**, but keep your dataframe as **df2**.


```python
df2.drop(1899, axis = 0, inplace = True)
```


```python
df2.shape
```




    (290584, 5)



`4.` Use **df2** in the below cells to answer the quiz questions related to **Quiz 4** in the classroom.

a. What is the probability of an individual converting regardless of the page they receive?


```python
df2['converted'].mean()
```




    0.11959708724499628



b. Given that an individual was in the `control` group, what is the probability they converted?


```python
df2[df2['group'] == 'control']['converted'].mean()
```




    0.1203863045004612



c. Given that an individual was in the `treatment` group, what is the probability they converted?


```python
df2[df2['group'] == 'treatment']['converted'].mean()
```




    0.11880806551510564



d. What is the probability that an individual received the new page?


```python
df2.query("landing_page == 'new_page'").count()[0] / df2.shape[0]
```




    0.50006194422266881



e. Use the results in the previous two portions of this question to suggest if you think there is evidence that one page leads to more conversions?  Write your response below.

**没有明显的证据表明哪一种页面（old_page, new_page）可以明显的提高转化率。但是根据平均转化率相比，old_page的转化率要稍高一些，new_page的转化率反而低于平均值。**

<a id='ab_test'></a>
### Part II - A/B Test

Notice that because of the time stamp associated with each event, you could technically run a hypothesis test continuously as each observation was observed.  

However, then the hard question is do you stop as soon as one page is considered significantly better than another or does it need to happen consistently for a certain amount of time?  How long do you run to render a decision that neither page is better than another?  

These questions are the difficult parts associated with A/B tests in general.  


`1.` For now, consider you need to make the decision just based on all the data provided.  If you want to assume that the old page is better unless the new page proves to be definitely better at a Type I error rate of 5%, what should your null and alternative hypotheses be?  You can state your hypothesis in terms of words or in terms of **$p_{old}$** and **$p_{new}$**, which are the converted rates for the old and new pages.

$$H_0: p_{new} - p_{old} \leq0 $$ 
$$H_1: p_{new} - p_{old} > 0 $$

`2.` Assume under the null hypothesis, $p_{new}$ and $p_{old}$ both have "true" success rates equal to the **converted** success rate regardless of page - that is $p_{new}$ and $p_{old}$ are equal. Furthermore, assume they are equal to the **converted** rate in **ab_data.csv** regardless of the page. <br><br>

Use a sample size for each page equal to the ones in **ab_data.csv**.  <br><br>

Perform the sampling distribution for the difference in **converted** between the two pages over 10,000 iterations of calculating an estimate from the null.  <br><br>

Use the cells below to provide the necessary parts of this simulation.  If this doesn't make complete sense right now, don't worry - you are going to work through the problems below to complete this problem.  You can use **Quiz 5** in the classroom to make sure you are on the right track.<br><br>

a. What is the **convert rate** for $p_{new}$ under the null? 


```python
p_new = df2['converted'].mean()
p_new
```




    0.11959708724499628



b. What is the **convert rate** for $p_{old}$ under the null? <br><br>


```python
p_old = df2['converted'].mean()
p_old
```




    0.11959708724499628



c. What is $n_{new}$?


```python
n_new = df2.query("landing_page == 'new_page'").count()[0]
n_new
```




    145310



d. What is $n_{old}$?


```python
n_old = df2.query("landing_page == 'old_page'").count()[0]
n_old
```




    145274



e. Simulate $n_{new}$ transactions with a convert rate of $p_{new}$ under the null.  Store these $n_{new}$ 1's and 0's in **new_page_converted**.


```python
new_page_converted = np.random.random(n_new) < p_new
new_page_converted
```




    array([False,  True, False, ..., False, False, False], dtype=bool)



f. Simulate $n_{old}$ transactions with a convert rate of $p_{old}$ under the null.  Store these $n_{old}$ 1's and 0's in **old_page_converted**.


```python
old_page_converted = np.random.random(n_old) < p_old
```

g. Find $p_{new}$ - $p_{old}$ for your simulated values from part (e) and (f).


```python
p_diff = new_page_converted.mean() - old_page_converted.mean()
p_diff
```




    -0.00035320857532959715



h. Simulate 10,000 $p_{new}$ - $p_{old}$ values using this same process similarly to the one you calculated in parts **a. through g.** above.  Store all 10,000 values in **p_diffs**.


```python
p_diffs = []
for _ in range(10000):
    new_page_converted = np.random.random(n_new) < p_new
    old_page_converted = np.random.random(n_old) < p_old
    p_diff = new_page_converted.mean() - old_page_converted.mean()
    p_diffs.append(p_diff)
```

i. Plot a histogram of the **p_diffs**.  Does this plot look like what you expected?  Use the matching problem in the classroom to assure you fully understand what was computed here.


```python
plt.hist(p_diffs);
plt.axvline(x = obs_diff, color ='r');
```


![png](output_60_0.png)


j. What proportion of the **p_diffs** are greater than the actual difference observed in **ab_data.csv**?


```python
p_diffs = np.array(p_diffs)
pval = (p_diffs > obs_diff).mean()
pval
```




    0.79779999999999995



k. In words, explain what you just computed in part **j.**.  What is this value called in scientific studies?  What does this value mean in terms of whether or not there is a difference between the new and old pages?

**这个值在科学研究上面被称作p值。在5%的显著性条件下，p值大于0.05则说明我们不能拒绝零假设，即新页面和旧页面的转化率之间没有显著差异。**

l. We could also use a built-in to achieve similar results.  Though using the built-in might be easier to code, the above portions are a walkthrough of the ideas that are critical to correctly thinking about statistical significance. Fill in the below to calculate the number of conversions for each page, as well as the number of individuals who received each page. Let `n_old` and `n_new` refer the the number of rows associated with the old page and new pages, respectively.


```python
import statsmodels.api as sm

convert_old = 0.1203863045004612
convert_new = 0.11880806551510564
n_old = 145274
n_new = 145310
```

    /anaconda3/lib/python3.6/site-packages/statsmodels/compat/pandas.py:56: FutureWarning: The pandas.core.datetools module is deprecated and will be removed in a future version. Please use the pandas.tseries module instead.
      from pandas.core import datetools
    

m. Now use `stats.proportions_ztest` to compute your test statistic and p-value.  [Here](http://knowledgetack.com/python/statsmodels/proportions_ztest/) is a helpful link on using the built in.


```python
z_score, p_value = sm.stats.proportions_ztest([n_new*convert_new, n_old*convert_old], [n_new, n_old], alternative = 'larger')
z_score,p_value
```




    (-1.3109241984234394, 0.90505831275902449)



n. What do the z-score and p-value you computed in the previous question mean for the conversion rates of the old and new pages?  Do they agree with the findings in parts **j.** and **k.**?

**z值的绝对值为1.31，小于95%显著性的临界值1.96，不拒绝零假设，我们认为新旧页面的转化率没有明显差异。p值为0.905>0.05,可以得出前述同样的结论。z检验的结论与之前j和k步得出的结论一致。**

<a id='regression'></a>
### Part III - A regression approach

`1.` In this final part, you will see that the result you acheived in the previous A/B test can also be acheived by performing regression.<br><br>

a. Since each row is either a conversion or no conversion, what type of regression should you be performing in this case?

**逻辑回归。**

b. The goal is to use **statsmodels** to fit the regression model you specified in part **a.** to see if there is a significant difference in conversion based on which page a customer receives.  However, you first need to create a colun for the intercept, and create a dummy variable column for which page each user received.  Add an **intercept** column, as well as an **ab_page** column, which is 1 when an individual receives the **treatment** and 0 if **control**.


```python
df2['intercept'] = 1
df2[['ab_page_a','ab_page_b']] = pd.get_dummies(df2['group'])
df2 = df2.drop('ab_page_a', axis = 1)
df2.rename(columns={'ab_page_b' : 'ab_page'}, inplace = True)
df2.head(10)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>timestamp</th>
      <th>group</th>
      <th>landing_page</th>
      <th>converted</th>
      <th>intercept</th>
      <th>ab_page</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>661590</td>
      <td>2017-01-11 16:55:06.154213</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>853541</td>
      <td>2017-01-08 18:28:03.143765</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>679687</td>
      <td>2017-01-19 03:26:46.940749</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>8</th>
      <td>817355</td>
      <td>2017-01-04 17:58:08.979471</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>9</th>
      <td>839785</td>
      <td>2017-01-15 18:11:06.610965</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>929503</td>
      <td>2017-01-18 05:37:11.527370</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11</th>
      <td>834487</td>
      <td>2017-01-21 22:37:47.774891</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>803683</td>
      <td>2017-01-09 06:05:16.222706</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13</th>
      <td>944475</td>
      <td>2017-01-22 01:31:09.573836</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>718956</td>
      <td>2017-01-22 11:45:11.327945</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>



c. Use **statsmodels** to import your regression model.  Instantiate the model, and fit the model using the two columns you created in part **b.** to predict whether or not an individual converts.


```python
log_mod = sm.Logit(df2['converted'], df2[['intercept', 'ab_page']])
result = log_mod.fit()
```

    Optimization terminated successfully.
             Current function value: 0.366118
             Iterations 6
    

d. Provide the summary of your model below, and use it as necessary to answer the following questions.


```python
result.summary2()
```




<table class="simpletable">
<tr>
        <td>Model:</td>              <td>Logit</td>       <td>No. Iterations:</td>    <td>6.0000</td>   
</tr>
<tr>
  <td>Dependent Variable:</td>     <td>converted</td>    <td>Pseudo R-squared:</td>    <td>0.000</td>   
</tr>
<tr>
         <td>Date:</td>        <td>2018-01-26 11:17</td>       <td>AIC:</td>        <td>212780.3502</td>
</tr>
<tr>
   <td>No. Observations:</td>       <td>290584</td>            <td>BIC:</td>        <td>212801.5095</td>
</tr>
<tr>
       <td>Df Model:</td>              <td>1</td>         <td>Log-Likelihood:</td>  <td>-1.0639e+05</td>
</tr>
<tr>
     <td>Df Residuals:</td>         <td>290582</td>          <td>LL-Null:</td>      <td>-1.0639e+05</td>
</tr>
<tr>
      <td>Converged:</td>           <td>1.0000</td>           <td>Scale:</td>         <td>1.0000</td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>       <th>Coef.</th>  <th>Std.Err.</th>     <th>z</th>      <th>P>|z|</th> <th>[0.025</th>  <th>0.975]</th> 
</tr>
<tr>
  <th>intercept</th> <td>-1.9888</td>  <td>0.0081</td>  <td>-246.6690</td> <td>0.0000</td> <td>-2.0046</td> <td>-1.9730</td>
</tr>
<tr>
  <th>ab_page</th>   <td>-0.0150</td>  <td>0.0114</td>   <td>-1.3109</td>  <td>0.1899</td> <td>-0.0374</td> <td>0.0074</td> 
</tr>
</table>



e. What is the p-value associated with **ab_page**? Why does it differ from the value you found in the **Part II**?<br><br>  **Hint**: What are the null and alternative hypotheses associated with your regression model, and how do they compare to the null and alternative hypotheses in the **Part II**?

**该逻辑回归中和 ab_page 相关的p值为0.1899，Part II中得到的z检验的P值为0.905。为什么它和假设检验中得出的p值不一致？主要原因是z检验方向不同，逻辑回归中的z检验是双向的，而Part II中的z检验是单向的（取大于观测平均值的方向）。**

f. Now, you are considering other things that might influence whether or not an individual converts.  Discuss why it is a good idea to consider other factors to add into your regression model.  Are there any disadvantages to adding additional terms into your regression model?

**为什么考虑给逻辑回归模型增加因素是个好办法？会引入哪些缺陷吗？因为该模型的单自变量（ab_page）与因变量（converted）的相关性非常弱，没有较好的回归结果，如果增加相关性强的因素，则可能得到更好的结果。需要考虑多重共线性对多个自变量间的影响。**

g. Now along with testing if the conversion rate changes for different pages, also add an effect based on which country a user lives. You will need to read in the **countries.csv** dataset and merge together your datasets on the approporiate rows.  [Here](https://pandas.pydata.org/pandas-docs/stable/generated/pandas.DataFrame.join.html) are the docs for joining tables. 

Does it appear that country had an impact on conversion?  Don't forget to create dummy variables for these country columns - **Hint: You will need two columns for the three dummy varaibles.** Provide the statistical output as well as a written response to answer this question.


```python
df_countries  = pd.read_csv('countries.csv')
df_countries.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>834778</td>
      <td>UK</td>
    </tr>
    <tr>
      <th>1</th>
      <td>928468</td>
      <td>US</td>
    </tr>
    <tr>
      <th>2</th>
      <td>822059</td>
      <td>UK</td>
    </tr>
    <tr>
      <th>3</th>
      <td>711597</td>
      <td>UK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>710616</td>
      <td>UK</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_countries.shape
```




    (290584, 2)




```python
df_countries['user_id'].is_unique
```




    True




```python
df_new = pd.merge(df2,df_countries,how = 'left',on='user_id')
df_new.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>user_id</th>
      <th>timestamp</th>
      <th>group</th>
      <th>landing_page</th>
      <th>converted</th>
      <th>intercept</th>
      <th>ab_page</th>
      <th>country</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>661590</td>
      <td>2017-01-11 16:55:06.154213</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>US</td>
    </tr>
    <tr>
      <th>1</th>
      <td>853541</td>
      <td>2017-01-08 18:28:03.143765</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>US</td>
    </tr>
    <tr>
      <th>2</th>
      <td>679687</td>
      <td>2017-01-19 03:26:46.940749</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>CA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>817355</td>
      <td>2017-01-04 17:58:08.979471</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>UK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>839785</td>
      <td>2017-01-15 18:11:06.610965</td>
      <td>treatment</td>
      <td>new_page</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>CA</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_temp = pd.get_dummies(df_new['country'])
df_new = df_new.join(df_temp)
```


```python

log_mod2 = sm.Logit(df_new['converted'], df_new[['intercept', 'UK', 'US']])
result2 = log_mod2.fit()
```

    Optimization terminated successfully.
             Current function value: 0.366116
             Iterations 6
    


```python
result2.summary2()
```




<table class="simpletable">
<tr>
        <td>Model:</td>              <td>Logit</td>       <td>No. Iterations:</td>    <td>6.0000</td>   
</tr>
<tr>
  <td>Dependent Variable:</td>     <td>converted</td>    <td>Pseudo R-squared:</td>    <td>0.000</td>   
</tr>
<tr>
         <td>Date:</td>        <td>2018-01-26 11:17</td>       <td>AIC:</td>        <td>212780.8333</td>
</tr>
<tr>
   <td>No. Observations:</td>       <td>290584</td>            <td>BIC:</td>        <td>212812.5723</td>
</tr>
<tr>
       <td>Df Model:</td>              <td>2</td>         <td>Log-Likelihood:</td>  <td>-1.0639e+05</td>
</tr>
<tr>
     <td>Df Residuals:</td>         <td>290581</td>          <td>LL-Null:</td>      <td>-1.0639e+05</td>
</tr>
<tr>
      <td>Converged:</td>           <td>1.0000</td>           <td>Scale:</td>         <td>1.0000</td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>       <th>Coef.</th>  <th>Std.Err.</th>     <th>z</th>     <th>P>|z|</th> <th>[0.025</th>  <th>0.975]</th> 
</tr>
<tr>
  <th>intercept</th> <td>-2.0375</td>  <td>0.0260</td>  <td>-78.3639</td> <td>0.0000</td> <td>-2.0885</td> <td>-1.9866</td>
</tr>
<tr>
  <th>UK</th>        <td>0.0507</td>   <td>0.0284</td>   <td>1.7863</td>  <td>0.0740</td> <td>-0.0049</td> <td>0.1064</td> 
</tr>
<tr>
  <th>US</th>        <td>0.0408</td>   <td>0.0269</td>   <td>1.5178</td>  <td>0.1291</td> <td>-0.0119</td> <td>0.0935</td> 
</tr>
</table>



h. Though you have now looked at the individual factors of country and page on conversion, we would now like to look at an interaction between page and country to see if there significant effects on conversion.  Create the necessary additional columns, and fit the new model.  

Provide the summary results, and your conclusions based on the results.


```python
log_mod3 = sm.Logit(df_new['converted'], df_new[['intercept', 'ab_page', 'UK', 'US']])
result3 = log_mod3.fit()
result3.summary2()
```

    Optimization terminated successfully.
             Current function value: 0.366113
             Iterations 6
    




<table class="simpletable">
<tr>
        <td>Model:</td>              <td>Logit</td>       <td>No. Iterations:</td>    <td>6.0000</td>   
</tr>
<tr>
  <td>Dependent Variable:</td>     <td>converted</td>    <td>Pseudo R-squared:</td>    <td>0.000</td>   
</tr>
<tr>
         <td>Date:</td>        <td>2018-01-26 11:17</td>       <td>AIC:</td>        <td>212781.1253</td>
</tr>
<tr>
   <td>No. Observations:</td>       <td>290584</td>            <td>BIC:</td>        <td>212823.4439</td>
</tr>
<tr>
       <td>Df Model:</td>              <td>3</td>         <td>Log-Likelihood:</td>  <td>-1.0639e+05</td>
</tr>
<tr>
     <td>Df Residuals:</td>         <td>290580</td>          <td>LL-Null:</td>      <td>-1.0639e+05</td>
</tr>
<tr>
      <td>Converged:</td>           <td>1.0000</td>           <td>Scale:</td>         <td>1.0000</td>   
</tr>
</table>
<table class="simpletable">
<tr>
      <td></td>       <th>Coef.</th>  <th>Std.Err.</th>     <th>z</th>     <th>P>|z|</th> <th>[0.025</th>  <th>0.975]</th> 
</tr>
<tr>
  <th>intercept</th> <td>-2.0300</td>  <td>0.0266</td>  <td>-76.2488</td> <td>0.0000</td> <td>-2.0822</td> <td>-1.9778</td>
</tr>
<tr>
  <th>ab_page</th>   <td>-0.0149</td>  <td>0.0114</td>   <td>-1.3069</td> <td>0.1912</td> <td>-0.0374</td> <td>0.0075</td> 
</tr>
<tr>
  <th>UK</th>        <td>0.0506</td>   <td>0.0284</td>   <td>1.7835</td>  <td>0.0745</td> <td>-0.0050</td> <td>0.1063</td> 
</tr>
<tr>
  <th>US</th>        <td>0.0408</td>   <td>0.0269</td>   <td>1.5161</td>  <td>0.1295</td> <td>-0.0119</td> <td>0.0934</td> 
</tr>
</table>



<a id='conclusions'></a>
## Conclusions

**我们发现，"ab_page"、"UK"、"US"与"converted"的相关系数非常接近0，说明它们与因变量之间的相关性非常弱。而三者的p值均大于0.05，并未体现出明显的显著性。所以我们可以得出结论，即用户所在国家和其使用新旧页面登录，对于转化率并未出现显著性变化。**


