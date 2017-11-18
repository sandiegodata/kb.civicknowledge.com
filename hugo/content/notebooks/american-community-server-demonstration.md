

{
    "authors": [
        {
            "email": "eric@civicknowledge.com",
            "name": "Eric Busboom",
            "organization": "Civic Knowledge",
            "type": "wrangler"
        }
    ],
    "date": "2017-11-17T22:11:27.007044",
    "description": "A demonstration of how to use American Community Survey data with Metapack.",
    "github": "https://github.com/sandiegodata/notebooks/blob/master/crime/San%20Diego%20Police%20Calls%20For%20Service.ipynb",
    "identifier": "e14510ec-57b3-4fae-8c4e-c47ca7c6b55b",
    "show_input": "show",
    "slug": "american-community-server-demonstration",
    "title": " American Community Server Demonstration"
}


This tutorial will demonstrate how to use Metapack with data from [Census Reporter](http://censusreporter.org) to analyze US Demographics. You will need to install at least these python packages, using pip:

```bash
pip install metapack
pip install publicdata
pip install seaborn
pip install geopandas
pip install pysal
```

# Finding Data In Census Reporter

The first step is to visit [Census Reporter](http://censusreporter.org) and find the dataset you want to work on. Census Reporter is probably the best search interface for American Community Survey data. You should explore Census Reporter a bit, and when you are ready, enter "Sex by Age" in the Explore search box. To get to the Sex and Age ducumentatino page. 

In the drop down list, select the "Sex By Age" documentation entry, then look for the link to table B01001

![Screen%20Shot%202017-11-16%20at%203.31.34%20PM.png](/img/american-community-server-demonstration/output_0_0.png)

Click on that link to get to the "Table B01001: Sex By Age". In the search box at the top, search for "San Diego County". Then, in the side bar on the left, under "Divide San Diego County, CA into …" click on "Census Tracts"

![Screen%20Shot%202017-11-16%20at%203.34.46%20PM.png](/img/american-community-server-demonstration/output_0_1.png)

The page should shoudl data for Table B01011, and have columns for a few sensus tracts in San Diego County. Now, we need to extract three values: 

* The table name, which we already know is 'B01001'
* The Geoid for the enclosing region, San Diego County, 
* The Summary Level code for the divisions of the county, which is in this case, Tracts. 

Look at the url in the URL bar of your browser, particulary at the end of the URL. 

```
https://..../?table=B01001&geo_ids=05000US06073,140|05000US06073&primary_geo_id=05000US06073
```

The table name is after ``table=``, the value is ``B01001``. The next part of the URL, ``geo_ids=05000US06073,140``. The long string, ``05000US06073`` is the geoid of San Diego County. ( '050' is for counties, '06' is California, and '073' is San Diego County ). After the comma is the summary level of tracts, '140'.  

So, here are the important values we'll need for the next section: 

* Table: **B01001**
* Geoid: **05000US06073**
* Summary Level: **140**

Now we can get started on the rest of the notebook. First, the standard includes. Note the ``%load_ext metapack`` extension loading magic. We'll need this for some of the Metapack features. 


```python

%load_ext metapack
%matplotlib inline

import pandas as pd
import geopandas as gpd
import numpy as np
import metapack as mp
import matplotlib.pyplot as plt
import seaborn as sns
sns.set(color_codes=True)
```

# Metapack

[Metapack](https://github.com/Metatab/metapack) is a data packaging system built on [Metatab](http://metatab.org). Normally, the configuration for Metapack is tored in a .csv file, but it can also be included in a Jupyter notebook. Here is what it looks like: 


```python
%%metatab

Identifier: e14510ec-57b3-4fae-8c4e-c47ca7c6b55b
Name: sandiegodata.org-acs_demo-1
    
Dataset: acs_demo
Origin: sandiegodata.org
Version: 1

Section: Contacts
Wrangler: Eric Busboom
Wrangler.Email: eric@civicknowledge.com
Wrangler.Organization: Civic Knowledge
```

This file just establishes identity and contact information, but it also creates a new variable in the namespace, ``mt_pkg``, which you can display to see what information is being included in the package. 

```python
mt_pkg
```




<h2>sandiegodata.org-acs_demo-1</h2>
<p></p>

<p></p>

<h3>Contacts</h3>
<p><strong>Wrangler:</strong> <a href="mailto:eric@civicknowledge.com">Eric Busboom</a> </p>
</ol>



## Adding Resources

Now we can add resources to the Metatab file, so we can get a handle on the Census Reporter data. Remember the dataq we extracted from the Census Reporter web page: 

Table: B01001
Geoid: 05000US06073
Summary Level: 140

This information can be used to construct a census reporter URL that Metatab can use to get the data via the Census Reporter API. The format of the URLS are ``censusreporter:<table>/<summarylevel>/<geoid>``, so our url will be: 

```
censusreporter:B01001/140/05000US06073
```

Most of the time when you are using a lot of ACS data, all of it will be at the same level in the same geography, so you'll use cut-and-paste these urls, changing only the table name. 

Now, add this URL into the reference section of the Metatab document:


```python
%%metatab
Section: Resources


Reference: censusreporter:B01001/140/05000US06073
Reference.Name: B01001
Reference.Description: Table B01001: Sex by Age

Reference: censusreportergeo://B01003/140/05000US06073
Reference.Name: tracts_cr
Reference.Description: Geo for tracts, from census reporter


```

Note that we also added a second resource, with the same path but a different scheme. This is a URL for the geographic boundaries of all of the tracts in San Diego County. The path also has a table in it, ``B01003``, which is the table for Total Population. That table was chosen because it's the smallest one, and we are not going to use the table information the Census Reporter includes with the geo information, but we could have also used just a geo url for table ``B01001`` and gotten the trace estimate numbers from it. 

Now, if we look at the package again, it will have the new references we added: 

```python
mt_pkg
```




<h2>sandiegodata.org-acs_demo-1</h2>
<p></p>

<p></p>

<h3>Contacts</h3>
<p><strong>Wrangler:</strong> <a href="mailto:eric@civicknowledge.com">Eric Busboom</a> </p>
<h3>Resources</h3>
<p><ol>
<li><p><strong>B01001</strong> - <a target="_blank" href="censusreporter://B01001/140/05000US06073">censusreporter://B01001/140/05000US06073</a> Table B01001: Sex by Age</p></li>
<li><p><strong>tracts_cr</strong> - <a target="_blank" href="censusreportergeo://B01003/140/05000US06073">censusreportergeo://B01003/140/05000US06073</a> Geo for tracts, from census reporter</p></li>
</ol></p>
</ol>



Now we can get the resource from the  data package and inspect it in Jupyter: 

```python
B01001_r = mt_pkg.resource('B01001')
B01001 = B01001_r.dataframe()
B01001.iloc[:6,:6] # Show 6 rows and 6 columns
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
      <th>geoid</th>
      <th>name</th>
      <th>B01001001</th>
      <th>B01001001_m90</th>
      <th>B01001002</th>
      <th>B01001002_m90</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>14000US06073000100</td>
      <td>Census Tract 1, San Diego, CA</td>
      <td>2716.0</td>
      <td>206.0</td>
      <td>1251.0</td>
      <td>162.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>14000US06073000201</td>
      <td>Census Tract 2.01, San Diego, CA</td>
      <td>2223.0</td>
      <td>260.0</td>
      <td>921.0</td>
      <td>121.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>14000US06073000202</td>
      <td>Census Tract 2.02, San Diego, CA</td>
      <td>4683.0</td>
      <td>398.0</td>
      <td>2323.0</td>
      <td>253.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>14000US06073000300</td>
      <td>Census Tract 3, San Diego, CA</td>
      <td>4875.0</td>
      <td>417.0</td>
      <td>2642.0</td>
      <td>320.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>14000US06073000400</td>
      <td>Census Tract 4, San Diego, CA</td>
      <td>3606.0</td>
      <td>272.0</td>
      <td>2290.0</td>
      <td>267.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>14000US06073000500</td>
      <td>Census Tract 5, San Diego, CA</td>
      <td>2873.0</td>
      <td>242.0</td>
      <td>1558.0</td>
      <td>205.0</td>
    </tr>
  </tbody>
</table>
</div>



Note that the table we get back has a geoid for each tract ( with '140' in front to indicate Tracts, and '06073' in the middle, for San Diego County, California ) Also, there are columns with names for variables ( ``B01001001`` ) and also columns for margin-of-errors for those estimates ( ``B01001001_m90``). The Census Repoter provides a lot of features for manipulating these parts of the data frame. 


# Census Data Frames

The CensusReporter package returns, via the ``.dataframe()`` method,  a subclass of dataframes,  ``CensusDataFrame``, that has a few special methods and properties for manipulating census data. 

The core feature of these dataframes is that when you index with an array with column names, you will also get the margin columns.

There are also a few special methods that perform operations on estimates and the margins: 

* ``sum_m``. Sum several columns and their margins. 
* ``add_sum_m``. Like ``sum_m`` but also adds the summed values and margins back into the dataframe
* ``sum_col_range`` Like ``sum_m``, but takes a starting and ending column, and sums the range of columns. 
* ``add_rse`` computes the Relative Standard Error and adds a new column to the dataframe. 
* ``ratio`` Compute a ratio between two column values, also computing the margin. In a ratio, one value is not a subset of the other, such as Males to Females. A ratio can be greater than 1. 
* ``proportion`` Compute a proportion between two column values, also computing the margin. In a proportion, the numerator is a subset of the denominator, such as Males to the whole population. A proportion must be 1 or less. 

One of the first things you'll notice is that from looking at the dataframe, you have no idea what the columns are for. You can use the '.titles' property to get the dataframe with column renamed with more detailed descriptions. ( Note that this view of the table is transposed, so the column names are easier to read ) 

```python
B01001.titles.head(4).T.head(10)
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>geoid</th>
      <td>14000US06073000100</td>
      <td>14000US06073000201</td>
      <td>14000US06073000202</td>
      <td>14000US06073000300</td>
    </tr>
    <tr>
      <th>name</th>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
    </tr>
    <tr>
      <th>B01001001 Total</th>
      <td>2716</td>
      <td>2223</td>
      <td>4683</td>
      <td>4875</td>
    </tr>
    <tr>
      <th>Margins for B01001001 Total</th>
      <td>206</td>
      <td>260</td>
      <td>398</td>
      <td>417</td>
    </tr>
    <tr>
      <th>B01001002 Total Male</th>
      <td>1251</td>
      <td>921</td>
      <td>2323</td>
      <td>2642</td>
    </tr>
    <tr>
      <th>Margins for B01001002 Total Male</th>
      <td>162</td>
      <td>121</td>
      <td>253</td>
      <td>320</td>
    </tr>
    <tr>
      <th>B01001003 Total Male Under 5 years</th>
      <td>53</td>
      <td>54</td>
      <td>115</td>
      <td>51</td>
    </tr>
    <tr>
      <th>Margins for B01001003 Total Male Under 5 years</th>
      <td>39</td>
      <td>53</td>
      <td>84</td>
      <td>59</td>
    </tr>
    <tr>
      <th>B01001004 Total Male 5 to 9 years</th>
      <td>64</td>
      <td>56</td>
      <td>54</td>
      <td>0</td>
    </tr>
    <tr>
      <th>Margins for B01001004 Total Male 5 to 9 years</th>
      <td>43</td>
      <td>31</td>
      <td>42</td>
      <td>12</td>
    </tr>
  </tbody>
</table>
</div>



If there are a lot of columns, it is often convient to search through the column titles. The ``.search_columns()`` method allows you to do that. Each of the arguments to the method can be either a string or a regular expression. 

```python
import re
B01001.search_columns(' 35 ',re.compile(r'2\d years'))
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
      <th>code</th>
      <th>title</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>B01001013</td>
      <td>B01001013 Total Male 35 to 39 years</td>
    </tr>
    <tr>
      <th>1</th>
      <td>B01001013_m90</td>
      <td>Margins for B01001013 Total Male 35 to 39 years</td>
    </tr>
    <tr>
      <th>2</th>
      <td>B01001037</td>
      <td>B01001037 Total Female 35 to 39 years</td>
    </tr>
    <tr>
      <th>3</th>
      <td>B01001037_m90</td>
      <td>Margins for B01001037 Total Female 35 to 39 years</td>
    </tr>
    <tr>
      <th>4</th>
      <td>B01001008</td>
      <td>B01001008 Total Male 20 years</td>
    </tr>
    <tr>
      <th>5</th>
      <td>B01001008_m90</td>
      <td>Margins for B01001008 Total Male 20 years</td>
    </tr>
    <tr>
      <th>6</th>
      <td>B01001009</td>
      <td>B01001009 Total Male 21 years</td>
    </tr>
    <tr>
      <th>7</th>
      <td>B01001009_m90</td>
      <td>Margins for B01001009 Total Male 21 years</td>
    </tr>
    <tr>
      <th>8</th>
      <td>B01001010</td>
      <td>B01001010 Total Male 22 to 24 years</td>
    </tr>
    <tr>
      <th>9</th>
      <td>B01001010_m90</td>
      <td>Margins for B01001010 Total Male 22 to 24 years</td>
    </tr>
    <tr>
      <th>10</th>
      <td>B01001011</td>
      <td>B01001011 Total Male 25 to 29 years</td>
    </tr>
    <tr>
      <th>11</th>
      <td>B01001011_m90</td>
      <td>Margins for B01001011 Total Male 25 to 29 years</td>
    </tr>
    <tr>
      <th>12</th>
      <td>B01001032</td>
      <td>B01001032 Total Female 20 years</td>
    </tr>
    <tr>
      <th>13</th>
      <td>B01001032_m90</td>
      <td>Margins for B01001032 Total Female 20 years</td>
    </tr>
    <tr>
      <th>14</th>
      <td>B01001033</td>
      <td>B01001033 Total Female 21 years</td>
    </tr>
    <tr>
      <th>15</th>
      <td>B01001033_m90</td>
      <td>Margins for B01001033 Total Female 21 years</td>
    </tr>
    <tr>
      <th>16</th>
      <td>B01001034</td>
      <td>B01001034 Total Female 22 to 24 years</td>
    </tr>
    <tr>
      <th>17</th>
      <td>B01001034_m90</td>
      <td>Margins for B01001034 Total Female 22 to 24 years</td>
    </tr>
    <tr>
      <th>18</th>
      <td>B01001035</td>
      <td>B01001035 Total Female 25 to 29 years</td>
    </tr>
    <tr>
      <th>19</th>
      <td>B01001035_m90</td>
      <td>Margins for B01001035 Total Female 25 to 29 years</td>
    </tr>
  </tbody>
</table>
</div>



An important feature of the ``CensusDataFrame`` is that when you index columns with an array, the dataframe will expand the array to also return the margin columns: 

```python
df = B01001[['B01001001','B01001002']].copy()
df.head()
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
      <th>B01001001</th>
      <th>B01001001_m90</th>
      <th>B01001002</th>
      <th>B01001002_m90</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2716.0</td>
      <td>206.0</td>
      <td>1251.0</td>
      <td>162.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2223.0</td>
      <td>260.0</td>
      <td>921.0</td>
      <td>121.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>4683.0</td>
      <td>398.0</td>
      <td>2323.0</td>
      <td>253.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4875.0</td>
      <td>417.0</td>
      <td>2642.0</td>
      <td>320.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>3606.0</td>
      <td>272.0</td>
      <td>2290.0</td>
      <td>267.0</td>
    </tr>
  </tbody>
</table>
</div>



# Creating Summary Variables

One of the most frequent operations on Census data is to sum columns. First, we will select a subset of the columns.


```python
cols = [
    'geoid',
    'B01001001', # Total Population
    'B01001002', # Total Male
    'B01001026' , # Total Female
    'B01001013','B01001014', # Males, 35-39 and 40-44
    'B01001037','B01001038' # Female, 35-39 and 40-44
]


B01001s = B01001[cols].copy()
B01001s.titles.head().T
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>geoid</th>
      <td>14000US06073000100</td>
      <td>14000US06073000201</td>
      <td>14000US06073000202</td>
      <td>14000US06073000300</td>
      <td>14000US06073000400</td>
    </tr>
    <tr>
      <th>B01001001 Total</th>
      <td>2716</td>
      <td>2223</td>
      <td>4683</td>
      <td>4875</td>
      <td>3606</td>
    </tr>
    <tr>
      <th>Margins for B01001001 Total</th>
      <td>206</td>
      <td>260</td>
      <td>398</td>
      <td>417</td>
      <td>272</td>
    </tr>
    <tr>
      <th>B01001002 Total Male</th>
      <td>1251</td>
      <td>921</td>
      <td>2323</td>
      <td>2642</td>
      <td>2290</td>
    </tr>
    <tr>
      <th>Margins for B01001002 Total Male</th>
      <td>162</td>
      <td>121</td>
      <td>253</td>
      <td>320</td>
      <td>267</td>
    </tr>
    <tr>
      <th>B01001026 Total Female</th>
      <td>1465</td>
      <td>1302</td>
      <td>2360</td>
      <td>2233</td>
      <td>1316</td>
    </tr>
    <tr>
      <th>Margins for B01001026 Total Female</th>
      <td>116</td>
      <td>242</td>
      <td>357</td>
      <td>336</td>
      <td>224</td>
    </tr>
    <tr>
      <th>B01001013 Total Male 35 to 39 years</th>
      <td>76</td>
      <td>39</td>
      <td>251</td>
      <td>212</td>
      <td>367</td>
    </tr>
    <tr>
      <th>Margins for B01001013 Total Male 35 to 39 years</th>
      <td>58</td>
      <td>34</td>
      <td>151</td>
      <td>111</td>
      <td>147</td>
    </tr>
    <tr>
      <th>B01001014 Total Male 40 to 44 years</th>
      <td>80</td>
      <td>101</td>
      <td>140</td>
      <td>399</td>
      <td>198</td>
    </tr>
    <tr>
      <th>Margins for B01001014 Total Male 40 to 44 years</th>
      <td>43</td>
      <td>57</td>
      <td>90</td>
      <td>162</td>
      <td>133</td>
    </tr>
    <tr>
      <th>B01001037 Total Female 35 to 39 years</th>
      <td>143</td>
      <td>21</td>
      <td>97</td>
      <td>155</td>
      <td>155</td>
    </tr>
    <tr>
      <th>Margins for B01001037 Total Female 35 to 39 years</th>
      <td>60</td>
      <td>19</td>
      <td>71</td>
      <td>90</td>
      <td>98</td>
    </tr>
    <tr>
      <th>B01001038 Total Female 40 to 44 years</th>
      <td>99</td>
      <td>84</td>
      <td>84</td>
      <td>71</td>
      <td>56</td>
    </tr>
    <tr>
      <th>Margins for B01001038 Total Female 40 to 44 years</th>
      <td>61</td>
      <td>74</td>
      <td>60</td>
      <td>55</td>
      <td>45</td>
    </tr>
  </tbody>
</table>
</div>



Now we can sum the male and female columns and compute some ratios. First, sum the columns. Note that the sum_m() method returns both the summed estimate, and the summed margin, which are re-assigned back in to the data frame. 

```python

B01001s['male_35_44'], B01001s['male_35_44_m90'] = B01001s.sum_m('B01001013', 'B01001014')
B01001s['female_35_44'], B01001s['female_35_44_m90'] = B01001s.sum_m('B01001037', 'B01001038')
B01001s.titles.head().T
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>geoid</th>
      <td>14000US06073000100</td>
      <td>14000US06073000201</td>
      <td>14000US06073000202</td>
      <td>14000US06073000300</td>
      <td>14000US06073000400</td>
    </tr>
    <tr>
      <th>B01001001 Total</th>
      <td>2716</td>
      <td>2223</td>
      <td>4683</td>
      <td>4875</td>
      <td>3606</td>
    </tr>
    <tr>
      <th>Margins for B01001001 Total</th>
      <td>206</td>
      <td>260</td>
      <td>398</td>
      <td>417</td>
      <td>272</td>
    </tr>
    <tr>
      <th>B01001002 Total Male</th>
      <td>1251</td>
      <td>921</td>
      <td>2323</td>
      <td>2642</td>
      <td>2290</td>
    </tr>
    <tr>
      <th>Margins for B01001002 Total Male</th>
      <td>162</td>
      <td>121</td>
      <td>253</td>
      <td>320</td>
      <td>267</td>
    </tr>
    <tr>
      <th>B01001026 Total Female</th>
      <td>1465</td>
      <td>1302</td>
      <td>2360</td>
      <td>2233</td>
      <td>1316</td>
    </tr>
    <tr>
      <th>Margins for B01001026 Total Female</th>
      <td>116</td>
      <td>242</td>
      <td>357</td>
      <td>336</td>
      <td>224</td>
    </tr>
    <tr>
      <th>B01001013 Total Male 35 to 39 years</th>
      <td>76</td>
      <td>39</td>
      <td>251</td>
      <td>212</td>
      <td>367</td>
    </tr>
    <tr>
      <th>Margins for B01001013 Total Male 35 to 39 years</th>
      <td>58</td>
      <td>34</td>
      <td>151</td>
      <td>111</td>
      <td>147</td>
    </tr>
    <tr>
      <th>B01001014 Total Male 40 to 44 years</th>
      <td>80</td>
      <td>101</td>
      <td>140</td>
      <td>399</td>
      <td>198</td>
    </tr>
    <tr>
      <th>Margins for B01001014 Total Male 40 to 44 years</th>
      <td>43</td>
      <td>57</td>
      <td>90</td>
      <td>162</td>
      <td>133</td>
    </tr>
    <tr>
      <th>B01001037 Total Female 35 to 39 years</th>
      <td>143</td>
      <td>21</td>
      <td>97</td>
      <td>155</td>
      <td>155</td>
    </tr>
    <tr>
      <th>Margins for B01001037 Total Female 35 to 39 years</th>
      <td>60</td>
      <td>19</td>
      <td>71</td>
      <td>90</td>
      <td>98</td>
    </tr>
    <tr>
      <th>B01001038 Total Female 40 to 44 years</th>
      <td>99</td>
      <td>84</td>
      <td>84</td>
      <td>71</td>
      <td>56</td>
    </tr>
    <tr>
      <th>Margins for B01001038 Total Female 40 to 44 years</th>
      <td>61</td>
      <td>74</td>
      <td>60</td>
      <td>55</td>
      <td>45</td>
    </tr>
    <tr>
      <th>male_35_44</th>
      <td>156</td>
      <td>140</td>
      <td>391</td>
      <td>611</td>
      <td>565</td>
    </tr>
    <tr>
      <th>male_35_44_m90</th>
      <td>72.2011</td>
      <td>66.3702</td>
      <td>175.787</td>
      <td>196.38</td>
      <td>198.237</td>
    </tr>
    <tr>
      <th>female_35_44</th>
      <td>242</td>
      <td>105</td>
      <td>181</td>
      <td>226</td>
      <td>211</td>
    </tr>
    <tr>
      <th>female_35_44_m90</th>
      <td>85.5628</td>
      <td>76.4003</td>
      <td>92.957</td>
      <td>105.475</td>
      <td>107.838</td>
    </tr>
  </tbody>
</table>
</div>



Not only is the DataFrame class a special subclass, but so is the series. From an estimate columns, the ``.m90`` property returns the corresponding 90% margin column.

```python
B01001s['B01001038'].m90.sum()
```




    48990.0



The Series object also has methods for :

* 95% margin: ``.m95``
* 99% margin: ``.m99``
* Relative standard error: ``.rse``
* Standard error: ``.se``
* Summing the series: ``.sum_m90()``

```python
B01001s['m_ratio'],B01001s['m_ratio_m90'] = B01001s.ratio('male_35_44','B01001002')
B01001s.add_rse('m_ratio')
B01001s['mf_proportion'],B01001s['mf_proportion_m90'] = B01001s.proportion('male_35_44','female_35_44')
B01001s.add_rse('mf_proportion');
```

```python
B01001s.head().T
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
      <th>0</th>
      <th>1</th>
      <th>2</th>
      <th>3</th>
      <th>4</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>geoid</th>
      <td>14000US06073000100</td>
      <td>14000US06073000201</td>
      <td>14000US06073000202</td>
      <td>14000US06073000300</td>
      <td>14000US06073000400</td>
    </tr>
    <tr>
      <th>B01001001</th>
      <td>2716</td>
      <td>2223</td>
      <td>4683</td>
      <td>4875</td>
      <td>3606</td>
    </tr>
    <tr>
      <th>B01001001_m90</th>
      <td>206</td>
      <td>260</td>
      <td>398</td>
      <td>417</td>
      <td>272</td>
    </tr>
    <tr>
      <th>B01001002</th>
      <td>1251</td>
      <td>921</td>
      <td>2323</td>
      <td>2642</td>
      <td>2290</td>
    </tr>
    <tr>
      <th>B01001002_m90</th>
      <td>162</td>
      <td>121</td>
      <td>253</td>
      <td>320</td>
      <td>267</td>
    </tr>
    <tr>
      <th>B01001026</th>
      <td>1465</td>
      <td>1302</td>
      <td>2360</td>
      <td>2233</td>
      <td>1316</td>
    </tr>
    <tr>
      <th>B01001026_m90</th>
      <td>116</td>
      <td>242</td>
      <td>357</td>
      <td>336</td>
      <td>224</td>
    </tr>
    <tr>
      <th>B01001013</th>
      <td>76</td>
      <td>39</td>
      <td>251</td>
      <td>212</td>
      <td>367</td>
    </tr>
    <tr>
      <th>B01001013_m90</th>
      <td>58</td>
      <td>34</td>
      <td>151</td>
      <td>111</td>
      <td>147</td>
    </tr>
    <tr>
      <th>B01001014</th>
      <td>80</td>
      <td>101</td>
      <td>140</td>
      <td>399</td>
      <td>198</td>
    </tr>
    <tr>
      <th>B01001014_m90</th>
      <td>43</td>
      <td>57</td>
      <td>90</td>
      <td>162</td>
      <td>133</td>
    </tr>
    <tr>
      <th>B01001037</th>
      <td>143</td>
      <td>21</td>
      <td>97</td>
      <td>155</td>
      <td>155</td>
    </tr>
    <tr>
      <th>B01001037_m90</th>
      <td>60</td>
      <td>19</td>
      <td>71</td>
      <td>90</td>
      <td>98</td>
    </tr>
    <tr>
      <th>B01001038</th>
      <td>99</td>
      <td>84</td>
      <td>84</td>
      <td>71</td>
      <td>56</td>
    </tr>
    <tr>
      <th>B01001038_m90</th>
      <td>61</td>
      <td>74</td>
      <td>60</td>
      <td>55</td>
      <td>45</td>
    </tr>
    <tr>
      <th>male_35_44</th>
      <td>156</td>
      <td>140</td>
      <td>391</td>
      <td>611</td>
      <td>565</td>
    </tr>
    <tr>
      <th>male_35_44_m90</th>
      <td>72.2011</td>
      <td>66.3702</td>
      <td>175.787</td>
      <td>196.38</td>
      <td>198.237</td>
    </tr>
    <tr>
      <th>female_35_44</th>
      <td>242</td>
      <td>105</td>
      <td>181</td>
      <td>226</td>
      <td>211</td>
    </tr>
    <tr>
      <th>female_35_44_m90</th>
      <td>85.5628</td>
      <td>76.4003</td>
      <td>92.957</td>
      <td>105.475</td>
      <td>107.838</td>
    </tr>
    <tr>
      <th>m_ratio</th>
      <td>0.1247</td>
      <td>0.152009</td>
      <td>0.168317</td>
      <td>0.231264</td>
      <td>0.246725</td>
    </tr>
    <tr>
      <th>m_ratio_m90</th>
      <td>0.0599312</td>
      <td>0.0747792</td>
      <td>0.0778611</td>
      <td>0.0794327</td>
      <td>0.091221</td>
    </tr>
    <tr>
      <th>m_ratio_rse</th>
      <td>29.216</td>
      <td>29.9052</td>
      <td>28.1207</td>
      <td>20.8797</td>
      <td>22.4758</td>
    </tr>
    <tr>
      <th>mf_proportion</th>
      <td>0.644628</td>
      <td>1.33333</td>
      <td>2.16022</td>
      <td>2.70354</td>
      <td>2.67773</td>
    </tr>
    <tr>
      <th>mf_proportion_m90</th>
      <td>0.192528</td>
      <td>1.15791</td>
      <td>1.47447</td>
      <td>1.53202</td>
      <td>1.65999</td>
    </tr>
    <tr>
      <th>mf_proportion_rse</th>
      <td>18.1559</td>
      <td>52.7924</td>
      <td>41.4928</td>
      <td>34.448</td>
      <td>37.6854</td>
    </tr>
  </tbody>
</table>
</div>



# Joining in geographic information

Now that we have the tract estimates that we want, its time to link them into a map. We'll use a similar method to get a dataframe for the geographic resoruce, but then use the ``.geo`` property to convert it to a GeoPandas dataframe, which has special support for geographic manipulations. Also note that we'll set the index to 'geoid', which is important for joining to our tract sex estimates table.  

```python
tracts = mt_pkg.resource('tracts_cr').dataframe().geo.set_index('geoid')
```

The result is a GeoPandas dataframe, which has WKT geography data in the ``geometry`` columns, along with the "total population" estimates that came along with the data request. 

```python
tracts.head()
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
      <th>id</th>
      <th>name</th>
      <th>B01003001</th>
      <th>B01003001e</th>
      <th>geometry</th>
    </tr>
    <tr>
      <th>geoid</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>14000US06073000100</th>
      <td>0</td>
      <td>Census Tract 1, San Diego, CA</td>
      <td>2716.0</td>
      <td>206.0</td>
      <td>POLYGON ((-117.194904 32.75278, -117.19471 32....</td>
    </tr>
    <tr>
      <th>14000US06073000201</th>
      <td>1</td>
      <td>Census Tract 2.01, San Diego, CA</td>
      <td>2223.0</td>
      <td>260.0</td>
      <td>POLYGON ((-117.178867 32.75765, -117.177966 32...</td>
    </tr>
    <tr>
      <th>14000US06073000202</th>
      <td>2</td>
      <td>Census Tract 2.02, San Diego, CA</td>
      <td>4683.0</td>
      <td>398.0</td>
      <td>POLYGON ((-117.184043 32.74571, -117.183827 32...</td>
    </tr>
    <tr>
      <th>14000US06073000300</th>
      <td>3</td>
      <td>Census Tract 3, San Diego, CA</td>
      <td>4875.0</td>
      <td>417.0</td>
      <td>POLYGON ((-117.168645 32.748968, -117.168404 3...</td>
    </tr>
    <tr>
      <th>14000US06073000400</th>
      <td>4</td>
      <td>Census Tract 4, San Diego, CA</td>
      <td>3606.0</td>
      <td>272.0</td>
      <td>POLYGON ((-117.170867 32.75865, -117.170187 32...</td>
    </tr>
  </tbody>
</table>
</div>



The GeoPandas dataframe has a ``.plot`` method that will create a map of the geo data. 

```python
tracts.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x114ee4390>




![png](/img/american-community-server-demonstration/output_32_1.png)


And, it has some features for selecting which colum from the dataframe to use to create a choroplethmap.  

```python
tracts.plot(column='B01003001', cmap='RdYlGn',scheme='fisher_jenks')
```




    <matplotlib.axes._subplots.AxesSubplot at 0x11582e860>




![png](/img/american-community-server-demonstration/output_34_1.png)


Now, we can join the previous ACS dataframe into the tracts, and plot it. 

```python
tracts.join(B01001s.set_index('geoid')).dropna(subset=['m_ratio']).plot(
    column='B01003001', cmap='RdYlGn',scheme='fisher_jenks', legend=True
)
```




    <matplotlib.axes._subplots.AxesSubplot at 0x115886ba8>




![png](/img/american-community-server-demonstration/output_36_1.png)


That plot is a bit small, but we can explicitly define the matplotlib axis and figure, and then set the size. 

```python
fig = plt.figure(figsize = (14,8))
ax = fig.add_subplot(111)

tracts.join(B01001s.set_index('geoid')).dropna(subset=['m_ratio']).plot(
    ax=ax, column='m_ratio', cmap='RdYlGn',scheme='fisher_jenks', legend=True
)

```




    <matplotlib.axes._subplots.AxesSubplot at 0x115942320>




![png](/img/american-community-server-demonstration/output_38_1.png)


And that's how you analyze ACS data with Metapack. 
