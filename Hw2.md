大數據行銷 作業二 b05504066
===
## Data input
1. static_reports.csv
2. dynamic_reports.csv
3. real_time_reports.csv


```python
import pandas as pd
import numpy as np
%config Completer.use_jedi = False
%config IPCompleter.greedy=True
pd.set_option('display.max_rows', 200)
##Read three databases##

static_db = pd.read_csv("static_reports.csv")
dynamic_db = pd.read_csv("dynamic_reports.csv")
real_time_db = pd.read_csv("real_time_reports.csv")
# real_time_db
```

## Useful information
get some information that is useful for building RMF table

Change the datetime format in real_time_table

Since last recorded date is 2007-12-31, we assign "*today*" to be 2008-01-01


```python
## take unique categories  and its appearance from one cloumn
appearance = real_time_db['Customer_ID'].value_counts()
Customer_lst = real_time_db['Customer_ID'].value_counts().index

## change time format
real_time_db['Date'] = pd.to_datetime(real_time_db['Date'],format='%Y%m%d')
real_time_db['Date'].sort_values

today = pd.to_datetime("2008-01-01")
real_time_db['Date']
```




    0      2006-01-01
    1      2006-01-02
    2      2006-01-04
    3      2006-01-04
    4      2006-01-04
              ...    
    3072   2007-12-30
    3073   2007-12-30
    3074   2007-12-31
    3075   2007-12-31
    3076   2007-12-31
    Name: Date, Length: 3077, dtype: datetime64[ns]



## Working on monetary
add all amount for every customer_id


```python
Monetary_dict={}
for id in Customer_lst:
    idx = real_time_db.index[real_time_db['Customer_ID']==id]
    sum = real_time_db.iloc[idx]['Amount'].sum()
    Monetary_dict[id]=sum
M_series = pd.Series(Monetary_dict)
# M_series
```

## Working on recency
calculate the recency for every customer_id
Recency_mth for bob stone 


```python

Recency_dict={}
Recency_mth_dict = {}
for id in Customer_lst:
    idx = real_time_db.index[real_time_db['Customer_ID']==id]
    dif = (today-real_time_db.iloc[idx]['Date'].max()).days
    dif_mth = (today-real_time_db.iloc[idx]['Date'].max())/ np.timedelta64(1, "M")
    Recency_dict[id]=dif
    Recency_mth_dict[id] = dif_mth
R_series = pd.Series(Recency_dict)
# R_series.sort_values()

```

## Working on frequency
calculate appeared times for every customer_id 
(Already done in "*appearance*")


```python
F_series = appearance
# F_series
```

## Five equal groups
Each category is divided into 5 segments.
5- highest 
1- lowest


```python
Cus_num = len(R_series)
R_score_dic={}
F_score_dic={}
M_score_dic={}
score= [5,4,3,2,1]
step = int(Cus_num/5)
bound = range(0,Cus_num,step)
count=0
score_idx=0
R_series = R_series.sort_values()
for i in range(Cus_num):
    R_score_dic[R_series.index[i]]=score[score_idx]
    F_score_dic[F_series.index[i]]=score[score_idx]
    M_score_dic[M_series.index[i]]=score[score_idx]
    count += 1
    if (count == step):
        count=0
        score_idx += 1
R_score = pd.Series(R_score_dic)
F_score = pd.Series(F_score_dic)
M_score = pd.Series(M_score_dic)
```

## Bob Stone's scale

Index |  scoring rule | weights
:-----: |:-------------: |:-------:
R     | < 3 months: 24 pts<br>3 ~ 6 months: 12 pts<br> 6 ~ 9 months: 6 pts<br>9 ~ 12 months: 3 pts<br>> 12 months: 0 pt| medium
F     | Frequency *　4 | high
M    |   Monetary * 0.3 % (max = 9pts) | low







```python
R_series_mth = pd.Series(Recency_mth_dict).sort_values()
R_score_bob_dic={}
F_score_bob_dic={}
M_score_bob_dic={}
for idx in Customer_lst:
    # R
    mth = R_series_mth[idx]
    if mth <= 3:
        R_score_bob_dic[idx]=24
    elif mth <= 6:
        R_score_bob_dic[idx]=12
    elif mth <= 9:
        R_score_bob_dic[idx]=6
    elif mth <= 12:
        R_score_bob_dic[idx]=3
    else:
        R_score_bob_dic[idx]=0
    # F
    F_score_bob_dic[idx] = F_series[idx] * 4
    
    # M
    a = M_series[idx] * 0.003
    if a <= 9:
        M_score_bob_dic[idx] = a
    else:
        M_score_bob_dic[idx] = 9

R_score_bob = pd.Series(R_score_bob_dic)
F_score_bob = pd.Series(F_score_bob_dic)
M_score_bob = pd.Series(M_score_bob_dic)

```

## Concatenation of series

concatenate all columns in one dataframe. 

Index | Definition
---------- | --------------
R_raw, F_raw, M_raw | Original value
R, F, M|Ranking based on equally-divided strategy
R_bob, F_bob, M_bob | Score based on Bob Stone strategy
rmf_equal | Total of original RFM ranking
rfm_bs | Total of Bob Stone score
equal_rank, bs_rank | The real ranking of the customer, less means valuable
equal_rank_scaleup | Since the maximmum ranking in original srategy is 15, we multiply it by 6 to compare with BS_rankng.
rank_diff | Difference between scale-up equal_rank and bs_rank
R_score_my, F_score_my, M_score_my | score based on my rules
rfm_my | total score based on my rules
my_rank | ranking based on my rules


```python


result = pd.concat([R_series,F_series,M_series,R_score,F_score,M_score,R_score_bob,F_score_bob,M_score_bob],\
                   axis=1,keys=['R_raw','F_raw','M_raw','R','F','M','R_bob','F_bob','M_bob'],\
                   names='Customer_ID')
result['rfm_equal'] = result['R'] + result['F'] + result['M']
result['rfm_bs'] = result['R_bob'] + result['F_bob'] + result['M_bob']

result.sort_values('rfm_equal',ascending = False)
result['equal_rank']=result['rfm_equal'].rank(method='dense',ascending=False)
result['bs_rank']=result['rfm_bs'].rank(method='dense',ascending=False)
# print(result['bs_rank'].max())
result['equal_rank_scaleup']=result['equal_rank']*6
result['rank_diff'] = abs(result['equal_rank_scaleup'] - result['bs_rank'])
# print(result)


```


```python
result.query('rank_diff > 18')
a =result.query('M_bob == 9 ').index
# import matplotlib.pyplot as plt
print("The minimum monetary value to receive 9 pts:",result.loc[a]['M_raw'].min())

```

    The minimum monetary value to receive 9 pts: 3024
    

## My scale

Index |  scoring rule | weights
:-----: |:-------------: | :-------:
R     | < 1 months: 20 pts<br>1 ~ 3 months: 13 pts<br> 3 ~ 6 months: 2 pts<br>6 ~ months: 0 pts <br>| medium
F     | Frequency *　4 | high
M    |   Monetary * 0.05 % (max = 10pts) | low


## My scoring rules
I doesn't change the frequency's rule of Bob Stone since this index is also important in our supermarket cases.  
We would first discuss Monetary index:
<p style='text-align: justify;padding-left:4em;'> 
First of all, Bob stone scale is based on USD, therefore we change the percentage from 10% to 0.3%. We knew that Bob's scale doesn't care about the monetary value, so it's quite normal that almost every customer has the highest score, 9 in this case. However, although the monetary value is also the least important index in supermarket, about 3/4 is max score.
The lowest value in this segmanet is 3024NTD, which is pretty low for a total spending among two years. <br><br>
Due to this unbalance distribution, the scale is revised down from 0.3% to 0.05%, and the highest point is adjusted to 10 , which in result reduces the count of the max-point segment to 71 and implies that customers who are marked 10 spend about 20000NTD among this two years, a much reasonable number. 
        
</p>

Next, we would discuss on Recency index:
<p style='text-align: justify;padding-left:4em;'> 
    The ordinary cut-off in Bob's scale is three months, which is apparently too long for supermarket's customer. Buying groceries should be weekly or even daily routine. Based on this concept, I revised the breakpoint from 3, 6, 9, 12 months to 1, 3, 6 months. Customers who don't buy things for a while will loss their score from 20 ,13 to 5 and 0.


    
After applying these two changes, if we set the customer whose ranking is below 40 as VIP, we found out that no customer raise their ranking, but about 40 customers of which ranking is below 40 in Bob's scale fall upon 70 in our scale. **This implies that we have delete 61 people out of our VIP list, and remain about 47 people.** These customer should be well treated with customized strategies, and for others, maybe we could just send same coupons to them.


```python
R_score_my_dic={}
F_score_my_dic={}
M_score_my_dic={}
for idx in Customer_lst:
    # R
    mth = R_series_mth[idx]
    if mth <= 1:
        R_score_my_dic[idx]=20
    elif mth <= 3:
        R_score_my_dic[idx]=13
    elif mth <= 6:
        R_score_my_dic[idx]=5
    else:
        R_score_my_dic[idx]=0
    # F
    F_score_my_dic[idx] = F_series[idx] * 4
    
    # M
    a = M_series[idx] * 0.0005
    if a <= 9:
        M_score_my_dic[idx] = a
    else:
        M_score_my_dic[idx] = 10
# print(R_score_my_dic)
result['R_score_my'] = result.index.to_series().map(R_score_my_dic)
result['F_score_my'] = result.index.to_series().map(F_score_my_dic)
result['M_score_my'] = result.index.to_series().map(M_score_my_dic)
result['rfm_my'] = result['R_score_my'] + result['F_score_my'] + result['M_score_my']
result.query('M_score_my ==10').shape[0]
result['my_rank']=result['rfm_my'].rank(method='dense',ascending=False)
# result.loc[:,['bs_rank','equal_rank_scaleup','my_rank']]
result.iloc[:,6:]
print("people whose rank increases:",result.query('bs_rank > my_rank ').shape[0])
print("people whose original rank is below 40 but falls out to 70 in my scale:",result.query('bs_rank < 40 & my_rank > 70').shape[0])
print("people whose original rank is lower than 40:",result.query('bs_rank < 40').shape[0])
print("people whose rank is lower than 40 in our scale:",result.query('my_rank < 40').shape[0])

result.to_csv("rfmtable.csv",index=False)
```

    people whose rank increases: 0
    people whose original rank is below 40 but falls out to 70 in my scale: 16
    people whose original rank is lower than 40: 106
    people whose rank is lower than 40 in our scale: 47
    


```python

```
###### tags: `大數據行銷`
