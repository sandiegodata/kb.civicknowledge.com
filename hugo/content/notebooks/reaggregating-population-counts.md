

{
    "authors": [
        {
            "name": "Eric Busboom"
        }
    ],
    "date": "2018-11-26T17:45:37.811332",
    "description": "How to reaggregate census estimates to other geographies.",
    "draft": false,
    "github": "https://github.com/sandiegodata/notebooks/blob/master/demographics/Population%20Reaggregation.ipynb",
    "identifier": null,
    "section": "notebooks",
    "show_input": "show",
    "slug": "reaggregating-population-counts",
    "title": " Reaggregating Population Counts",
    "toc": false,
    "weight": 3
}



A common problem in demographics involves having census estimates in one greography, but needing them in another. In this example, we convert tract level estimates of races and ethnicities into San Diego City police beats. 

The technique attributes population from tracts to beats by the areas of the overlaps. The basic procedure is to find the overlaps between beats and Census tracts, then addign a portion of the population of the tract to the beat, based on the raio of the size of overlap to the size of the tract. 

```python
import seaborn as sns
import metapack as mp
import pandas as pd
import geopandas as gpd
import numpy as np
import matplotlib.pyplot as plt
from IPython.display import display 

%matplotlib inline
sns.set_context('notebook')

```

We'll start with the Data Library's [San Diego Police Regions](https://data.sandiegodata.org/dataset/sandiego-gov-police_regions) dataset. 

```python
pkg = mp.open_package('http://library.metatab.org/sandiego.gov-police_regions-3.csv')
pkg
```




<h1>San Diego Police Regions</h1>
<p><code>sandiego.gov-police_regions-3</code> Last Update: 2018-11-27T00:25:19</p>
<p><em>Boundary shapes for San Diego neighborhoods, beats and divisions.</em></p>
<h2>Documentation Links</h2>
<ul>
<li><a href="https://data.sandiego.gov/datasets/pd-divisions/">Police Divisions Repository Page</a> Data repository page that links to original files.</li>
</ul>
<h2>Contacts</h2>
<ul>
<li><strong>Wrangler</strong> <a href="mailto:eric@civicknowledge.com">Eric Busboom</a>, <a href="http://civicknowledge.com">Civic Knowledge</a></li>
</ul>
<h2>Resources</h2>
<ul>
<li><strong> <a href="http://library.metatab.org/sandiego.gov-police_regions-3/data/pd_beats.csv">pd_beats</a></strong>. Police beats</li>
<li><strong> <a href="http://library.metatab.org/sandiego.gov-police_regions-3/data/pd_divisions.csv">pd_divisions</a></strong>. Police Divisions</li>
<li><strong> <a href="http://library.metatab.org/sandiego.gov-police_regions-3/data/pd_neighborhoods.csv">pd_neighborhoods</a></strong>. Police Neighborhoods</li>
<li><strong> <a href="http://library.metatab.org/sandiego.gov-police_regions-3/data/beat_demographics.csv">beat_demographics</a></strong>. Counts of people in the beat, by race.</li>
</ul>
<h2>References</h2>
<ul>
<li><strong><a href="metapack+http://library.metatab.org/sandiegodata.org-communities-2018-7.csv#tracts">tracts</a></strong>. Census tracts from 2016 5 year ACS, for San Diego county</li>
<li><strong><a href="census://CA/140/B02001">race</a></strong>. Race, by tract, in San Diego county</li>
</ul>



The beats dataset needs to be modified a bit, to: 

* Remove small outlying beats in disconnected areas
* Convert to a UTM coordinate system, so areas are measured in meters. 
* Calculate areas, in square kilometers. 
* Join disjoin beat polygons into single shapes. 

The disconnected beats are probably for land that the City owns that is not contigous, such as power substations and damns. These beats aren't interesting, and they pull in a lot of tracts, so they are removed. 

```python
beats = pkg.resource('pd_beats').geoframe()

# There are  beats that are way off in east county. Get rid of them.
rightmost_centroid = beats.centroid.x.sort_values(ascending=False).iloc[:6].max()

beats = beats[beats.centroid.x <rightmost_centroid]

# Convert to EPSG:26911, ( A randomly selected UTM Zone 11N CRS) so area calculations 
# will be in square meters, rather than square degrees
beats = beats.to_crs({'init': 'epsg:26911'})

# It looks like the dataset has multiple rows per beat, one feature per row. We need
# it to have one row per beat, with multiple features combined together. 
beats = beats.dissolve(by='beat').reset_index()

#  Add the area
beats['beat_area'] = beats.area / 1_000_000

beats.plot()

```




    <matplotlib.axes._subplots.AxesSubplot at 0x125b2afd0>




![png](/img/reaggregating-population-counts/output_5_1.png)


```python
tracts = pkg.reference('tracts').geoframe()

tracts = tracts.to_crs({'init': 'epsg:26911'})

#  Add the area
tracts['tract_area'] = tracts.area / 1_000_000


tracts.plot()
```




    <matplotlib.axes._subplots.AxesSubplot at 0x10fa43898>




![png](/img/reaggregating-population-counts/output_6_1.png)


Now we can get the census data for [Table B03002: Hispanic or Latino Origin by Race](https://censusreporter.org/tables/B03002). For this, we'll use the census URLs from the [publicdata package](https://github.com/Metatab/publicdata)

```python
from rowgenerators import parse_app_url
t = parse_app_url('census://CA/140/B03002').dataframe()

# White, black, asian, etc are all non hispanic. 
col_map = {
    'B03002_001':'total',
    'B03002_003':'white',
    'B03002_004':'black',
    'B03002_005':'aian',
    'B03002_006':'asian',
    'B03002_007':'nhopi', 
    'B03002_012':'hisp'
    
}

# Optional, add in the margin columns. 
#for k,v in list(col_map.items()):
#    col_map[k+'_m90'] = col_map[k]+'_m90'
    
# Select just the tracts for the county
race_tracts = t[t.COUNTY=='073'].rename(columns=col_map).reset_index().rename(columns={'GEOID':'geoid'})
```

```python
race_tracts = race_tracts[['geoid', 'total', 'white', 'black', 'aian', 'asian', 'nhopi', 'hisp']]
race_tracts.titles.head().T
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
      <th>GEOID</th>
      <td>14000US06073000100</td>
      <td>14000US06073000201</td>
      <td>14000US06073000202</td>
      <td>14000US06073000300</td>
      <td>14000US06073000400</td>
    </tr>
    <tr>
      <th>total</th>
      <td>2773</td>
      <td>2158</td>
      <td>4828</td>
      <td>4946</td>
      <td>3916</td>
    </tr>
    <tr>
      <th>white</th>
      <td>2276</td>
      <td>1628</td>
      <td>3477</td>
      <td>3437</td>
      <td>2655</td>
    </tr>
    <tr>
      <th>black</th>
      <td>0</td>
      <td>0</td>
      <td>37</td>
      <td>177</td>
      <td>52</td>
    </tr>
    <tr>
      <th>aian</th>
      <td>0</td>
      <td>8</td>
      <td>39</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>asian</th>
      <td>84</td>
      <td>84</td>
      <td>368</td>
      <td>196</td>
      <td>475</td>
    </tr>
    <tr>
      <th>nhopi</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>8</td>
      <td>0</td>
    </tr>
    <tr>
      <th>hisp</th>
      <td>290</td>
      <td>389</td>
      <td>767</td>
      <td>886</td>
      <td>649</td>
    </tr>
  </tbody>
</table>
</div>



```python
t = gpd.sjoin(beats, tracts)

ax = t.plot()
beats.centroid.plot(ax=ax, color='red')

t = t[['geoid', 'beat']].drop_duplicates()\
    .merge(tracts[['geoid','geometry', 'tract_area']],on='geoid')\
    .merge(beats[['beat','geometry', 'beat_area']],on='beat')

```


![png](/img/reaggregating-population-counts/output_10_0.png)


Now we can use Geopands to compute the intersections between tracts and beats, and calculate the population in each intersection. 

```python
intr = gpd.overlay(beats, tracts, how='intersection')[['beat','geoid','geometry']]

intr['intr_area'] = (intr.geometry.area/1_000_000.0).astype(float)

# Get rid of really small intersections
intr = intr[intr.intr_area >= .01] 

merged = intr[['beat','geoid', 'intr_area']]\
    .merge(tracts[['geoid', 'tract_area']],on='geoid')\
    .merge(beats[['beat', 'beat_area']],on='beat')\
    .merge(race_tracts, on='geoid')

merged = merged.drop_duplicates(subset=['beat','geoid'])

merged['tract_overlap_proportion'] = merged.intr_area/merged.tract_area
merged['beat_overlap_proportion'] = merged.intr_area/merged.beat_area

# The intersection areas must be smaller than both of the areas being intersected
assert(not any(merged.intr_area > merged.beat_area))
assert(not any(merged.intr_area > merged.tract_area))

# Check that all of the areas of the beats are accounted for
assert(all(merged.groupby('beat').beat_overlap_proportion.sum().round(1) == 1))

merged['total'] = merged.total * merged.tract_overlap_proportion
merged['white'] = merged.white * merged.tract_overlap_proportion
merged['asian'] = merged.asian * merged.tract_overlap_proportion
merged['black'] = merged.black * merged.tract_overlap_proportion
merged['aian']  = merged.aian * merged.tract_overlap_proportion
merged['hisp']  = merged.hisp * merged.tract_overlap_proportion
merged['nhopi']  = merged.nhopi * merged.tract_overlap_proportion

merged.head().T

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
      <th>beat</th>
      <td>0</td>
      <td>721</td>
      <td>0</td>
      <td>0</td>
      <td>511</td>
    </tr>
    <tr>
      <th>geoid</th>
      <td>14000US06073021900</td>
      <td>14000US06073021900</td>
      <td>14000US06073021600</td>
      <td>14000US06073003800</td>
      <td>14000US06073003800</td>
    </tr>
    <tr>
      <th>intr_area</th>
      <td>0.183666</td>
      <td>0.0228637</td>
      <td>0.645752</td>
      <td>0.0366767</td>
      <td>1.77264</td>
    </tr>
    <tr>
      <th>tract_area</th>
      <td>10.6162</td>
      <td>10.6162</td>
      <td>15.2322</td>
      <td>1.82267</td>
      <td>1.82267</td>
    </tr>
    <tr>
      <th>beat_area</th>
      <td>18.2475</td>
      <td>7.63003</td>
      <td>18.2475</td>
      <td>18.2475</td>
      <td>6.80108</td>
    </tr>
    <tr>
      <th>total</th>
      <td>90.741</td>
      <td>11.2959</td>
      <td>155.544</td>
      <td>133.613</td>
      <td>6457.75</td>
    </tr>
    <tr>
      <th>white</th>
      <td>29.4973</td>
      <td>3.67198</td>
      <td>105.137</td>
      <td>68.8994</td>
      <td>3330.02</td>
    </tr>
    <tr>
      <th>black</th>
      <td>14.6016</td>
      <td>1.81769</td>
      <td>8.05486</td>
      <td>26.6422</td>
      <td>1287.66</td>
    </tr>
    <tr>
      <th>aian</th>
      <td>0.899625</td>
      <td>0.11199</td>
      <td>0.127182</td>
      <td>1.04637</td>
      <td>50.5728</td>
    </tr>
    <tr>
      <th>asian</th>
      <td>9.39416</td>
      <td>1.16944</td>
      <td>5.08728</td>
      <td>10.5039</td>
      <td>507.673</td>
    </tr>
    <tr>
      <th>nhopi</th>
      <td>0.70932</td>
      <td>0.0882999</td>
      <td>1.05985</td>
      <td>0.523185</td>
      <td>25.2864</td>
    </tr>
    <tr>
      <th>hisp</th>
      <td>34.255</td>
      <td>4.26424</td>
      <td>31.8803</td>
      <td>24.5696</td>
      <td>1187.49</td>
    </tr>
    <tr>
      <th>tract_overlap_proportion</th>
      <td>0.0173005</td>
      <td>0.00215366</td>
      <td>0.042394</td>
      <td>0.0201225</td>
      <td>0.972553</td>
    </tr>
    <tr>
      <th>beat_overlap_proportion</th>
      <td>0.0100653</td>
      <td>0.00299654</td>
      <td>0.0353886</td>
      <td>0.00200996</td>
      <td>0.260641</td>
    </tr>
  </tbody>
</table>
</div>



From there, we just sum up the populations in each fragment of the beats. 

```python
beat_demographics = merged.groupby('beat')\
    .sum()[['total', 'white', 'black', 'aian', 'asian', 'nhopi', 'hisp']].round()
```

```python
beat_demographics.head()
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
      <th>total</th>
      <th>white</th>
      <th>black</th>
      <th>aian</th>
      <th>asian</th>
      <th>nhopi</th>
      <th>hisp</th>
    </tr>
    <tr>
      <th>beat</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>488.0</td>
      <td>259.0</td>
      <td>55.0</td>
      <td>3.0</td>
      <td>30.0</td>
      <td>2.0</td>
      <td>130.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>241.0</td>
      <td>41.0</td>
      <td>15.0</td>
      <td>0.0</td>
      <td>47.0</td>
      <td>0.0</td>
      <td>119.0</td>
    </tr>
    <tr>
      <th>111</th>
      <td>27810.0</td>
      <td>13741.0</td>
      <td>1245.0</td>
      <td>21.0</td>
      <td>5368.0</td>
      <td>214.0</td>
      <td>6204.0</td>
    </tr>
    <tr>
      <th>112</th>
      <td>11914.0</td>
      <td>8054.0</td>
      <td>154.0</td>
      <td>47.0</td>
      <td>1218.0</td>
      <td>14.0</td>
      <td>2094.0</td>
    </tr>
    <tr>
      <th>113</th>
      <td>12079.0</td>
      <td>8245.0</td>
      <td>61.0</td>
      <td>0.0</td>
      <td>595.0</td>
      <td>0.0</td>
      <td>2742.0</td>
    </tr>
  </tbody>
</table>
</div>


