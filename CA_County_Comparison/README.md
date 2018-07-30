
# Traffic in California by County - 2016

This was a simple exercise in practicing data cleaning and visualization. Caltrans measures traffic by counting the number of cars whic pass specific mile markers each day and averaging the total over 365 days. Traffic is counted in both directions. 

Orange County had the most daily traffic followed closely by Los Angeles County. As a current resident of one and a former resident of the other, this does not surprise me.

I used this relatively clean data to test out plotly's chorolpleth capabilities. I've discovered that plotly is much more powerful than matplotlib, but less dynamic than dashboard technologies like PowerBI. I will continue to use these choropleths if a project demands a map to be displayed in-line with coding.


```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import plotly
import config
from config import user, apikey
plotly.tools.set_credentials_file(username=user, api_key=apikey)
import plotly.plotly as py
import plotly.figure_factory as ff

traffic_data = pd.read_csv("2016aadt.csv", encoding='ISO-8859-1')
county_abbrev = pd.read_csv("countyabbrev.csv", encoding='ISO-8859-1')
fips_finder = pd.read_csv("FIPS_CA.csv", encoding='ISO-8859-1')
```


```python
#Isolate needed columns, group by county using average
traffic = traffic_data[["County", "Ahead AADT"]].dropna(how='any')
averages = traffic.groupby(["County"]).mean().reset_index()

averages.head()
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
      <th>County</th>
      <th>Ahead AADT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ALA</td>
      <td>117664.383562</td>
    </tr>
    <tr>
      <th>1</th>
      <td>ALP</td>
      <td>1708.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>AMA</td>
      <td>6959.756098</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BUT</td>
      <td>16949.361702</td>
    </tr>
    <tr>
      <th>4</th>
      <td>CAL</td>
      <td>5280.217391</td>
    </tr>
  </tbody>
</table>
</div>




```python
#Rename abbreviation dataframe for merge
abbrev = county_abbrev.rename(columns={
    "COUNTY": "County Name", 
    "ABBREV.":"County"})
abbrev.head()
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
      <th>County Name</th>
      <th>DISTRICT</th>
      <th>County</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alameda</td>
      <td>4</td>
      <td>ALA</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alpine</td>
      <td>10</td>
      <td>ALP</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Amador</td>
      <td>10</td>
      <td>AMA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Butte</td>
      <td>3</td>
      <td>BUT</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Calaveras</td>
      <td>10</td>
      <td>CAL</td>
    </tr>
  </tbody>
</table>
</div>




```python
merge_table = pd.merge(averages, abbrev, on="County")
average_traffic = merge_table[["County Name", "Ahead AADT"]]

average_traffic.loc[:,"County Name"] = average_traffic["County Name"].astype(str) + " County"
average_traffic.head()
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
      <th>County Name</th>
      <th>Ahead AADT</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Alameda County</td>
      <td>117664.383562</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Alpine County</td>
      <td>1708.000000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Amador County</td>
      <td>6959.756098</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Butte County</td>
      <td>16949.361702</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Calaveras County</td>
      <td>5280.217391</td>
    </tr>
  </tbody>
</table>
</div>



## Top 5 Counties with Most Traffic


```python
#Add FIPS
fips_adjust = fips_finder.rename(columns={"countyname": "County Name"})
traffic_fips = pd.merge(average_traffic, fips_adjust, on="County Name")

traffic_sort = traffic_fips.sort_values("Ahead AADT", ascending=False).reset_index(drop=True)
traffic_sort.head()
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
      <th>County Name</th>
      <th>Ahead AADT</th>
      <th>FIPS</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Orange County</td>
      <td>158485.714286</td>
      <td>6059</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Los Angeles County</td>
      <td>157618.237650</td>
      <td>6037</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Contra Costa County</td>
      <td>133358.095238</td>
      <td>6013</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Alameda County</td>
      <td>117664.383562</td>
      <td>6001</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Santa Clara County</td>
      <td>113252.956989</td>
      <td>6085</td>
    </tr>
  </tbody>
</table>
</div>




```python
colorscale = ["00E501","31E000","63DC00","92D800","C0D400","CFB300","CB8100","C75000","C32200","BF0009"]    
    
endpts = list(np.linspace(average_traffic["Ahead AADT"].min(), average_traffic["Ahead AADT"].max(), len(colorscale) - 1))
fips = traffic_fips["FIPS"]
values = traffic_fips["Ahead AADT"]

fig = ff.create_choropleth(
    fips=fips, values=values, scope=['California'], binning_endpoints=endpts, colorscale=colorscale,
    county_outline={'color': 'rgb(255,255,255)', 'width': 0.5}, show_state_data=False, show_hover=True,  
    centroid_marker={'opacity': 0}, asp=2.9, title='Highway Traffic 2016', legend_title='Average Number of Cars')

py.iplot(fig, filename='traffic')
```

![CA Traffic 2016](https://plot.ly/~lauraraynes/6.embed)


<iframe id="igraph" scrolling="no" style="border:none;" seamless="seamless" src="https://plot.ly/~lauraraynes/6.embed" height="450px" width="900px"></iframe>


