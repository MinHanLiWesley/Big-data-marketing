大數據行銷 作業三 b05504066 李旻翰
===
```python

```
## HW3 b05504066 李旻翰

```python
import pandas as pd
import numpy as np
%config Completer.use_jedi = False
%config IPCompleter.greedy=True
pd.set_option('display.max_rows', 200)

df = pd.read_excel("Hw3.xlsx",sheet_name='step1')
# df.dropna(axis='columns')
# np.where(df['Customer_ID'].notnull())[0].shape
df
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
      <th>Customer_ID</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>92</td>
      <td>20060101</td>
    </tr>
    <tr>
      <th>1</th>
      <td>198</td>
      <td>20060102</td>
    </tr>
    <tr>
      <th>2</th>
      <td>338</td>
      <td>20060104</td>
    </tr>
    <tr>
      <th>3</th>
      <td>338</td>
      <td>20060104</td>
    </tr>
    <tr>
      <th>4</th>
      <td>338</td>
      <td>20060104</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3072</th>
      <td>8202</td>
      <td>20071230</td>
    </tr>
    <tr>
      <th>3073</th>
      <td>8202</td>
      <td>20071230</td>
    </tr>
    <tr>
      <th>3074</th>
      <td>6828</td>
      <td>20071231</td>
    </tr>
    <tr>
      <th>3075</th>
      <td>7854</td>
      <td>20071231</td>
    </tr>
    <tr>
      <th>3076</th>
      <td>7854</td>
      <td>20071231</td>
    </tr>
  </tbody>
</table>
<p>3077 rows × 2 columns</p>
</div>



## Step 1
Remove the duplicated records and change the format of date


```python
# remove the duplicates record
df['Customer_ID'] = pd.Series([int(i) for i in df['Customer_ID']])
df['Date'] = pd.to_datetime(df['Date'],format='%Y%m%d').dt.date
df1 = df.drop_duplicates(ignore_index=True)
df1
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
      <th>Customer_ID</th>
      <th>Date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>92</td>
      <td>2006-01-01</td>
    </tr>
    <tr>
      <th>1</th>
      <td>198</td>
      <td>2006-01-02</td>
    </tr>
    <tr>
      <th>2</th>
      <td>338</td>
      <td>2006-01-04</td>
    </tr>
    <tr>
      <th>3</th>
      <td>527</td>
      <td>2006-01-05</td>
    </tr>
    <tr>
      <th>4</th>
      <td>62</td>
      <td>2006-01-05</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1810</th>
      <td>7854</td>
      <td>2007-12-30</td>
    </tr>
    <tr>
      <th>1811</th>
      <td>7923</td>
      <td>2007-12-30</td>
    </tr>
    <tr>
      <th>1812</th>
      <td>8202</td>
      <td>2007-12-30</td>
    </tr>
    <tr>
      <th>1813</th>
      <td>6828</td>
      <td>2007-12-31</td>
    </tr>
    <tr>
      <th>1814</th>
      <td>7854</td>
      <td>2007-12-31</td>
    </tr>
  </tbody>
</table>
<p>1815 rows × 2 columns</p>
</div>



## Step 2
sort, remove the low frequent customer (<3)


```python
# first sort the value by id and date
df2= df1.sort_values(by=['Customer_ID','Date'],ignore_index=True)
# remove the low frequncy id (can't do CRI with appearance < 3 )
valuecount=df2['Customer_ID'].value_counts()
to_remove = valuecount[valuecount<3].index
# drop the index (reset_index to update the index )
df2.drop(df2.query('Customer_ID in @to_remove').index,inplace=True)
df2.reset_index(drop=True,inplace=True)
# calculate the interval between date, if different id, append 9999
a=[]
for i in range(0,len(df2)-1):
    if df2.iloc[i,0] == df2.iloc[i+1,0]:
        a.append((df2.iloc[i+1,1]-df2.iloc[i,1]).days)
    else:
        a.append(9999)
a.append(9999)

#remove 9999
df2['int'] = pd.Series(a)
df2.drop(df2.query('int==9999').index,inplace=True)
df2.reset_index(drop=True,inplace=True)
# df2
df2
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
      <th>Customer_ID</th>
      <th>Date</th>
      <th>int</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>62</td>
      <td>2006-01-05</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>62</td>
      <td>2006-02-24</td>
      <td>40</td>
    </tr>
    <tr>
      <th>2</th>
      <td>62</td>
      <td>2006-04-05</td>
      <td>234</td>
    </tr>
    <tr>
      <th>3</th>
      <td>62</td>
      <td>2006-11-25</td>
      <td>15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>62</td>
      <td>2006-12-10</td>
      <td>27</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1588</th>
      <td>7854</td>
      <td>2007-12-18</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1589</th>
      <td>7854</td>
      <td>2007-12-19</td>
      <td>4</td>
    </tr>
    <tr>
      <th>1590</th>
      <td>7854</td>
      <td>2007-12-23</td>
      <td>5</td>
    </tr>
    <tr>
      <th>1591</th>
      <td>7854</td>
      <td>2007-12-28</td>
      <td>2</td>
    </tr>
    <tr>
      <th>1592</th>
      <td>7854</td>
      <td>2007-12-30</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
<p>1593 rows × 3 columns</p>
</div>



## Step 3
Calculate the weight and prepared to calculate MLE


```python
# need to copy, otherwise it would affect df2
df3 = df2.copy()
# add weight, the closest date has the highest weight
id = df3.iloc[0,0]
count = 0
weight=[]
id = df3.iloc[count,0]
i=1
while(count<len(df3)):
    if df3.iloc[count,0] ==id:
        weight.append(i)
        i+=1
        count+=1
    else:
        id = df3.iloc[count,0]
        i=1
# multiply the date interval and weight to get the weighted data
df3['weight']= pd.Series(weight)
df3['intXweight'] = pd.Series([df3.iloc[i]['int']*df3.iloc[i]['weight']for i in range(len(df3))])
df3       
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
      <th>Customer_ID</th>
      <th>Date</th>
      <th>int</th>
      <th>weight</th>
      <th>intXweight</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>62</td>
      <td>2006-01-05</td>
      <td>50</td>
      <td>1</td>
      <td>50</td>
    </tr>
    <tr>
      <th>1</th>
      <td>62</td>
      <td>2006-02-24</td>
      <td>40</td>
      <td>2</td>
      <td>80</td>
    </tr>
    <tr>
      <th>2</th>
      <td>62</td>
      <td>2006-04-05</td>
      <td>234</td>
      <td>3</td>
      <td>702</td>
    </tr>
    <tr>
      <th>3</th>
      <td>62</td>
      <td>2006-11-25</td>
      <td>15</td>
      <td>4</td>
      <td>60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>62</td>
      <td>2006-12-10</td>
      <td>27</td>
      <td>5</td>
      <td>135</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>1588</th>
      <td>7854</td>
      <td>2007-12-18</td>
      <td>1</td>
      <td>26</td>
      <td>26</td>
    </tr>
    <tr>
      <th>1589</th>
      <td>7854</td>
      <td>2007-12-19</td>
      <td>4</td>
      <td>27</td>
      <td>108</td>
    </tr>
    <tr>
      <th>1590</th>
      <td>7854</td>
      <td>2007-12-23</td>
      <td>5</td>
      <td>28</td>
      <td>140</td>
    </tr>
    <tr>
      <th>1591</th>
      <td>7854</td>
      <td>2007-12-28</td>
      <td>2</td>
      <td>29</td>
      <td>58</td>
    </tr>
    <tr>
      <th>1592</th>
      <td>7854</td>
      <td>2007-12-30</td>
      <td>1</td>
      <td>30</td>
      <td>30</td>
    </tr>
  </tbody>
</table>
<p>1593 rows × 5 columns</p>
</div>



## Step 4
store it to excel and do the PivotTable analysis in excel


```python
with pd.ExcelWriter('Hw3_new.xlsx') as writer:
    df1.to_excel(writer,sheet_name='step1',index=False)
    df2.to_excel(writer,sheet_name='step2',index=False)
    df3.to_excel(writer,sheet_name='step3',index=False)
```

##Step 5
After finish calculating CAI, calculate 漸趨活躍群，穩定消費群，漸趨靜止群，
these three groups'average buying interval and amounts.

Read from HW3_final.xlsx


```python
df5 = pd.read_excel("HW3_final.xlsx",sheet_name='Relative cumulative frequency').iloc[0:-2,0:2]
# 20% = 38 80% =112 from diagram in sheet
# get the id list of each group
lowFre = df5.iloc[:39]['Customer_ID']
midFre = df5.iloc[39:113]['Customer_ID']
highFre = df5.iloc[113:]['Customer_ID']
#calculate the average time interval
lowInterval = df3.iloc[df3.query('Customer_ID in @lowFre').index]['int'].mean()
midInterval = df3.iloc[df3.query('Customer_ID in @midFre').index]['int'].mean()
highInterval = df3.iloc[df3.query('Customer_ID in @highFre').index]['int'].mean()
#calculate the average monetary
df6 = pd.read_csv(r'..\real_time_reports.csv')
lowMonetary = df6.iloc[df6.query('Customer_ID in @lowFre').index]['Amount'].mean()
midMonetary = df6.iloc[df6.query('Customer_ID in @midFre').index]['Amount'].mean()
highMonetary = df6.iloc[df6.query('Customer_ID in @highFre').index]['Amount'].mean()

# make the table

df_cai = pd.DataFrame({'CAI群別':['漸趨活卻群','穩定消費群','漸趨靜止群'],
                     '客戶人數':[38,75,37],
                     '比例':['25%','50%','25%'],
                     '消費日平均消費金額':[highMonetary,midMonetary,lowMonetary],
                     '平均消費間隔天數':[highInterval,midInterval,lowInterval]})
df_cai
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
      <th>CAI群別</th>
      <th>客戶人數</th>
      <th>比例</th>
      <th>消費日平均消費金額</th>
      <th>平均消費間隔天數</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>漸趨活卻群</td>
      <td>38</td>
      <td>25%</td>
      <td>1859.506294</td>
      <td>50.488127</td>
    </tr>
    <tr>
      <th>1</th>
      <td>穩定消費群</td>
      <td>75</td>
      <td>50%</td>
      <td>1460.404591</td>
      <td>49.285714</td>
    </tr>
    <tr>
      <th>2</th>
      <td>漸趨靜止群</td>
      <td>37</td>
      <td>25%</td>
      <td>1673.435443</td>
      <td>41.627002</td>
    </tr>
  </tbody>
</table>
</div>


###### tags: `大數據行銷`