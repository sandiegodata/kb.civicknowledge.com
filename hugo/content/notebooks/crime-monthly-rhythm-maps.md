

{
    "authors": [
        {
            "name": "Eric Busboom"
        }
    ],
    "date": "2017-11-12T21:21:57.611541",
    "description": "Rhythm maps for San Diego Crime incidents, from 2007 to 2014",
    "draft": false,
    "github": "https://github.com/sandiegodata/notebooks/blob/master/crime/Crime%20Monthly%20Rhythm%20Maps.ipynb",
    "section": "notebooks",
    "show_input": "hide",
    "slug": "crime-monthly-rhythm-maps",
    "title": " Crime Monthly Rhythm Maps",
    "toc": false,
    "weight": 3
}


This analysis will examine six years of data that was aggregated by SANDAG and processed by the [San Diego Regional Data Library](http://sandiegodata.org) to visualize crime patterns on a weekly and hourly basis. The analysis uses 797,978 crime incidents. The data is published in CSV and shapefile form at the [Library's data repository](http://data.sandiegodata.org/dataset/clarinova_com-crime-incidents-casnd-7ba4-extract)

These heat maps, also know as rhythm maps,  are square grids where each hour of the week is represented by a cell. The vertical axis is hour of day, and the horizontal axis is day of week, with Sunday being the area between 0 and 1, etc.  The color indicates the number of crimes in that hour of the week, with red being larger than yellow. Note that each grid is scaled independently, so a red square in one heatmap is not comparable to a red square in another heatmap. This visualization is only for looking at the paterns of crimes over time; it doesn't tell you anything about the number of crimes, except the relative relationship between hours in a single grid.

There are three types of incidents:
* Crime Cases
* Arrests
* Citations

However, this version of the dataset doesn't have the column from the original dataset that has these values, so this analysis will include all of them. 

There are 12 crime categories, expressed as the codes established by the FBI. ( UCR codes. )

First we will look at all of the incidents in the city, and then by each type. A grid is blank when there are fewer than 40 incidents of that  type and category. 













<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
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
      <th>id</th>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>3</td>
      <td>4</td>
    </tr>
    <tr>
      <th>date</th>
      <td>2008-01-01 00:00:00</td>
      <td>2008-01-01 00:00:00</td>
      <td>2008-01-01 00:00:00</td>
      <td>2008-01-01 00:00:00</td>
      <td>2008-01-01 00:00:00</td>
    </tr>
    <tr>
      <th>year</th>
      <td>2008</td>
      <td>2008</td>
      <td>2008</td>
      <td>2008</td>
      <td>2008</td>
    </tr>
    <tr>
      <th>month</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>day</th>
      <td>2922</td>
      <td>2922</td>
      <td>2922</td>
      <td>2922</td>
      <td>2922</td>
    </tr>
    <tr>
      <th>week</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>dow</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>2</td>
    </tr>
    <tr>
      <th>time</th>
      <td>2017-11-12 00:00:00</td>
      <td>2017-11-12 00:00:00</td>
      <td>2017-11-12 00:00:00</td>
      <td>2017-11-12 00:00:00</td>
      <td>2017-11-12 00:00:00</td>
    </tr>
    <tr>
      <th>hour</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>is_night</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>type</th>
      <td>THEFT/LARCENY</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
    </tr>
    <tr>
      <th>address</th>
      <td>2500  Block Orion Way</td>
      <td>1200  Block Aguirre Drive</td>
      <td>2400  Block Golfcrest Loop</td>
      <td>2500  Block Catamaran Way</td>
      <td>400  Block J Street</td>
    </tr>
    <tr>
      <th>city</th>
      <td>SndCAR</td>
      <td>SndCHU</td>
      <td>SndOCN</td>
      <td>SndCHU</td>
      <td>SndCHU</td>
    </tr>
    <tr>
      <th>segment_id</th>
      <td>103154</td>
      <td>154361</td>
      <td>71714</td>
      <td>172915</td>
      <td>38410</td>
    </tr>
    <tr>
      <th>nbrhood</th>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
    </tr>
    <tr>
      <th>community</th>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
    </tr>
    <tr>
      <th>comm_pop</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>council</th>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
      <td>NONE</td>
    </tr>
    <tr>
      <th>council_pop</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>asr_zone</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>lampdist</th>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>lat</th>
      <td>33.1378</td>
      <td>32.6303</td>
      <td>33.1934</td>
      <td>32.656</td>
      <td>32.6261</td>
    </tr>
    <tr>
      <th>lon</th>
      <td>-117.266</td>
      <td>-117.004</td>
      <td>-117.316</td>
      <td>-116.958</td>
      <td>-117.079</td>
    </tr>
    <tr>
      <th>desc</th>
      <td>PETTY THEFT</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
      <td>FRAUD</td>
    </tr>
    <tr>
      <th>gctype</th>
      <td>cns/segment</td>
      <td>cns/segment</td>
      <td>cns/segment</td>
      <td>cns/segment</td>
      <td>cns/segment</td>
    </tr>
    <tr>
      <th>gcquality</th>
      <td>65</td>
      <td>65</td>
      <td>22</td>
      <td>65</td>
      <td>65</td>
    </tr>
  </tbody>
</table>
</div>







# Hour vs Week of Year By Major Crime Type

These maps show the hour of the day on the vertical axis and week of year on the horizonal, with all of the years in the dataset combined. The maps clearly show a 5AM daily lull, and a spike in DUIs in the middle of the summer. 





![png](/img/crime-monthly-rhythm-maps/output_8_0.png)


# Week of Year by Data Year

With a vertical axis of the year of the data, these maps plot one box per week for all weeks in the dataset. THe most significant patter is the reduction in DUI and Alcohol crimes in the later years. 





![png](/img/crime-monthly-rhythm-maps/output_10_0.png)


In 'Year vs Week by Category', the years run from 2006 to 2013, where '7' is 2013. 

# Hour of Day Vs Week of Year, for Pacific Beach

These maps show a seasonal rhythm, with the strongest being the cluster of alcohol crimes  in the afternoon and evenings in the summer. 





![png](/img/crime-monthly-rhythm-maps/output_13_0.png)







![png](/img/crime-monthly-rhythm-maps/output_15_0.png)





![png](/img/crime-monthly-rhythm-maps/output_16_0.png)





![png](/img/crime-monthly-rhythm-maps/output_17_0.png)





![png](/img/crime-monthly-rhythm-maps/output_18_0.png)





![png](/img/crime-monthly-rhythm-maps/output_19_0.png)





![png](/img/crime-monthly-rhythm-maps/output_20_0.png)





![png](/img/crime-monthly-rhythm-maps/output_21_0.png)





![png](/img/crime-monthly-rhythm-maps/output_22_0.png)





![png](/img/crime-monthly-rhythm-maps/output_23_0.png)

