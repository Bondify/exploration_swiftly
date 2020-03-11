```python
from IPython.display import HTML

HTML('''<script>
code_show=true; 
function code_toggle() {
 if (code_show){
 $('div.input').hide();
 } else {
 $('div.input').show();
 }
 code_show = !code_show
} 
$( document ).ready(code_toggle);
</script>
<form action="javascript:code_toggle()"><input type="submit" value="Click here to toggle on/off the raw code."></form>''')
```




<script>
code_show=true; 
function code_toggle() {
 if (code_show){
 $('div.input').hide();
 } else {
 $('div.input').show();
 }
 code_show = !code_show
} 
$( document ).ready(code_toggle);
</script>
<form action="javascript:code_toggle()"><input type="submit" value="Click here to toggle on/off the raw code."></form>



This are the first impressions of Swiftly data. The sample we have has data for one line of MTA Baltimore for January 2020. 

The data is a collection of events for this line and has information about dwell time in stops and runtimes between stops. It also has the distance covered so it would be possible to calculate the speed for each segment of each trip.

A few lines of the data are shown below.


```python
import pandas as pd
import geopandas as gpd
import keplergl as kp
import partridge as ptg
import plotly.express as px
```


```python
# Raw GTFS from https://www.mta.maryland.gov/developer-resources
# # Partridge to get the lines
service_ids = ptg.read_busiest_date(r"C:\Users\santi\Google Drive\Master\Python\Swiftly\data\google_transit")[1]
view = {'trips.txt': {'service_id': service_ids}}

feed = ptg.load_geo_feed(r"C:\Users\santi\Google Drive\Master\Python\Swiftly\data\google_transit", view)

routes = feed.routes
trips = feed.trips
stops = feed.stops
stop_times = feed.stop_times
shapes = feed.shapes


# Swiftly data
swiftly1 = pd.read_excel(r"C:\Users\santi\Google Drive\Master\Python\Swiftly\data\MTA CityLink Navy Jan 1 - Jan 18.xlsx")
swiftly2 = pd.read_excel(r"C:\Users\santi\Google Drive\Master\Python\Swiftly\data\MTA CityLink Navy Jan 19 - Jan 31 (1).xlsx")
switftly = swiftly1.append(swiftly2)

switftly['stop_id'] = switftly['stop_id'].map(str)
switftly = pd.merge(switftly, stops.loc[:,['stop_id','stop_name', 'geometry']], how='left')
```


```python
# Separete runtimes and dwell_times in two separate df 
day_name= ['Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday', 'Saturday','Sunday']
switftly['hour_of_day'] = switftly.actual_time.apply(lambda x: x.hour)
switftly['day_of_week'] = switftly.actual_date.apply(lambda x: x.weekday())

dwell_times = switftly.loc[switftly.dwell_time_secs.notnull(),:]
runtimes = switftly.loc[switftly.dwell_time_secs.isnull(),:]

dwell_times.loc[:,'stop_id'] = dwell_times.loc[:,'stop_id'].apply(str)
```


```python
switftly.head(4)
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
      <th>block_id</th>
      <th>trip_id</th>
      <th>route_id</th>
      <th>route_short_name</th>
      <th>direction_id</th>
      <th>stop_id</th>
      <th>vehicle_id</th>
      <th>driver_id</th>
      <th>sched_adherence_secs</th>
      <th>scheduled_date</th>
      <th>...</th>
      <th>actual_date</th>
      <th>actual_time</th>
      <th>dwell_time_secs</th>
      <th>travel_time_secs</th>
      <th>is_departure</th>
      <th>stop_path_length_meters</th>
      <th>stop_name</th>
      <th>hour_of_day</th>
      <th>day_of_week</th>
      <th>geometry</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2654</td>
      <td>2408869</td>
      <td>11739</td>
      <td>CityLink NAVY</td>
      <td>0</td>
      <td>27</td>
      <td>4013</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>NaT</td>
      <td>...</td>
      <td>2020-01-18</td>
      <td>13:09:06</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>201.4</td>
      <td>MONDAWMIN STATION</td>
      <td>13</td>
      <td>5</td>
      <td>POINT (-76.65292 39.31797)</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2654</td>
      <td>2408869</td>
      <td>11739</td>
      <td>CityLink NAVY</td>
      <td>0</td>
      <td>27</td>
      <td>4013</td>
      <td>NaN</td>
      <td>225.338</td>
      <td>2020-01-18</td>
      <td>...</td>
      <td>2020-01-18</td>
      <td>13:09:10</td>
      <td>4.28</td>
      <td>NaN</td>
      <td>True</td>
      <td>201.4</td>
      <td>MONDAWMIN STATION</td>
      <td>13</td>
      <td>5</td>
      <td>POINT (-76.65292 39.31797)</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2654</td>
      <td>2408869</td>
      <td>11739</td>
      <td>CityLink NAVY</td>
      <td>0</td>
      <td>13528</td>
      <td>4013</td>
      <td>NaN</td>
      <td>192.999</td>
      <td>2020-01-18</td>
      <td>...</td>
      <td>2020-01-18</td>
      <td>13:09:12</td>
      <td>NaN</td>
      <td>2.66</td>
      <td>False</td>
      <td>157.2</td>
      <td>MONDAWMIN METRO STATION BAY 8</td>
      <td>13</td>
      <td>5</td>
      <td>POINT (-76.65281 39.31784)</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2658</td>
      <td>2408719</td>
      <td>11739</td>
      <td>CityLink NAVY</td>
      <td>0</td>
      <td>1343</td>
      <td>4037</td>
      <td>1351.0</td>
      <td>NaN</td>
      <td>NaT</td>
      <td>...</td>
      <td>2020-01-06</td>
      <td>15:53:31</td>
      <td>NaN</td>
      <td>NaN</td>
      <td>False</td>
      <td>320.0</td>
      <td>LOMBARD ST &amp; EUTAW ST fs wb</td>
      <td>15</td>
      <td>0</td>
      <td>POINT (-76.62116 39.28758)</td>
    </tr>
  </tbody>
</table>
<p>4 rows × 21 columns</p>
</div>



# Visual on a map

## Map # 1: At the stop level

First we wanted to try a similar visual to what Swiftly has. This makes sense for dwell times since it is information at the stop level (not the segment level). Different possible filters would be:
    1. By hour of day
    2. By direction
    3. By day of the week


```python
avg_dwell = dwell_filtered.pivot_table('dwell_time_secs', index = ['route_id', 'direction_id','hour_of_day', 'stop_id','stop_name'], aggfunc='mean').reset_index()
avg_dwell = pd.merge(avg_dwell, stops.loc[:,['stop_id', 'geometry']], how='left')
avg_dwell = gpd.GeoDataFrame(data = avg_dwell.drop('geometry', axis=1), geometry=avg_dwell.geometry)
#avg_dwell.head(2)
```


```python
# Load the config from the  saved file
%run avg_dwell_times_config.py

# Create map
m = kp.KeplerGl(height=600,config=config)
m.add_data(data=avg_dwell.loc[avg_dwell.dwell_time_secs <1000,:], name='Average dwell times')
m
```

    User Guide: https://github.com/keplergl/kepler.gl/blob/master/docs/keplergl-jupyter/user-guide.md
    


    KeplerGl(config={'version': 'v1', 'config': {'visState': {'filters': [{'dataId': 'Average dwell times', 'id': …



```python
# # # Save map_1 config to a file
# with open('avg_dwell_times_config.py', 'w') as f:
#    f.write('config = {}'.format(m.config))

# # load the config
# %run avg_dwell_times_config.py
```

## Map #2: At the segment level

Combining the data from swiftly and the GTFS (yes, it is very easy to combine since the stop_ids are the same!!!) we can also show the dwell time by segment.


```python
switftly = gpd.GeoDataFrame(data = switftly.drop('geometry', axis=1), geometry=switftly.geometry)

# Keep only the one route from the GTFS
route = routes.loc[routes.route_id=='11739']
trips_route = trips.loc[trips.route_id == '11739']
shapes_route = shapes.loc[shapes.shape_id.isin(trips_route.shape_id.unique())]
stop_times_route = stop_times.loc[stop_times.trip_id.isin(trips_route.trip_id.unique())]

trips_per_shape = trips_route.pivot_table('trip_id', index=['direction_id', 'shape_id'], aggfunc='count').reset_index()
trips_per_shape = pd.merge(trips_per_shape, shapes_route, how='left')
trips_per_shape = gpd.GeoDataFrame(data = trips_per_shape.drop('geometry', axis=1), geometry = trips_per_shape.geometry)

# Get routes info in trips
trips_route = pd.merge(trips_route, route, how='left').loc[:, ['trip_id', 'route_id', 
                                                    'service_id', 'direction_id', 'shape_id']]

# Get trips, routes and stops info in stop_times
stop_times_route = pd.merge(stop_times_route, trips_route, how='left') 
stop_times_route = pd.merge(stop_times_route, stops, how='left')

# Data frame with stop sequence for route and direction
sseq = stop_times_route.pivot_table('stop_sequence',
                                     index = ['route_id', 'direction_id', 
                                              'stop_id','stop_name', 'shape_id'],
                                     aggfunc='mean').sort_values(by=[
    'shape_id','route_id', 'direction_id', 'stop_sequence', 'stop_id']).reset_index()

route_shapes = sseq.pivot_table('stop_id',
                               index = ['route_id', 'direction_id', 'shape_id'],
                               aggfunc='count').reset_index()
route_shapes.columns = ['route_id', 'direction_id', 'shape_id', 'stops_count']

from shapely.ops import nearest_points

# Create a DataFrame with the pair (stop, nearest_point) for each shape_id

shape_closest_points = pd.DataFrame()

for index, row in shapes_route.iterrows():
    shape_id = row.shape_id
    route_id = route_shapes.loc[route_shapes.shape_id == shape_id, 'route_id'].values[0]
    direction_id = route_shapes.loc[route_shapes.shape_id == shape_id, 'direction_id'].values[0]
    
    # Look for the shape
    shape = shapes_route.loc[shapes_route.shape_id == shape_id,'geometry'].values[0]

    
    # Look for the stop_ids of this shape
    route_stop_ids = sseq.loc[(sseq['route_id'] == route_id) 
                              & (sseq['direction_id'] == direction_id)
                              &(sseq['shape_id'] == shape_id)]

    # Look for the geometry of these stops
    merged = pd.merge(route_stop_ids, stops, left_on='stop_id', right_on='stop_id', how='left')
    route_stop_geom = merged.geometry
    
    # Look for the nearest points of these stops that are in the shape
    points_in_shape = route_stop_geom.apply(lambda x: nearest_points(x, shape))
   
    # Append to DataFrame
    appendable = pd.DataFrame()
    appendable['points'] = points_in_shape
    appendable['shape_id'] = shape_id
    
    shape_closest_points = shape_closest_points.append(appendable)
    
from shapely.ops import nearest_points
from shapely.geometry import Point, LineString, MultiLineString

shape_trans_lines = pd.DataFrame()
# First we define a function that will help us create the line to intersect the shape
offset = 0.0001

def create_line(row):
    # Formula to make the line longer
    # a = (y1-b)/x1
    # b = (y2-x2/x1*y1)/(1-x2/x1)
    if row[0] == row[1]:
        x1 = row[0].x - offset
        y1 = row[0].y - offset
        
        x2 = row[0].x 
        y2 = row[0].y
        
        x3 = row[0].x + offset
        y3 = row[0].y + offset
        
    else:   
        x1 = row[0].x
        y1 = row[0].y

        x2 = row[1].x
        y2 = row[1].y

        # If x2==x1 it will give the error "ZeroDivisionError"
        if float(x2) != float(x1):
            b = (y2-x2/x1*y1)/(1-x2/x1)
            a = (y1-b)/x1

            if x2 - x1 < 0: # We should create an "if" to check if we need to do -1 or +1 depending on x2-x1
                x3 = x2 - offset
            else:
                x3 = x2 + offset

            y3 = a*x3 + b

        else:
            x3 = x2
            b = 0
            a = 0

            if y2-y1 < 0:
                y3 = y2 - offset/5
            else: 
                y3 = y2 + offset/5

    trans = LineString([Point(x1,y1), Point(x2,y2), Point(x3, y3)])
    return trans

# For each shape we need to create transversal lines and separete the shape in segments
for index, row in shapes_route.iterrows():
    # Choose the shape
    shape_id = row.shape_id
    
    # Choose the pair (stop, nearest point to shape) to create the line
    scp = shape_closest_points.loc[shape_closest_points.shape_id == shape_id, 'points']
    
    lines = scp.apply(create_line)
    lines_gdf = gpd.GeoDataFrame(geometry=lines)
    lines_multi = MultiLineString(list(lines.values)[1:-1])
    
    appendable = pd.DataFrame()
    appendable['trans_lines'] = lines
    appendable['shape_id'] = shape_id
    
    shape_trans_lines = shape_trans_lines.append(appendable)
    
from shapely.ops import split
from shapely import geometry, ops

# Set the tolerance of the cuts
tolerance = 0.0001

segments = pd.DataFrame()

for index, row in shapes_route.iterrows():
    shape_id = row.shape_id
    route_id = route_shapes.loc[route_shapes.shape_id == shape_id, 'route_id'].values[0]
    direction_id = route_shapes.loc[route_shapes.shape_id == shape_id, 'direction_id'].values[0]
    
    df = sseq.loc[(sseq['route_id'] == route_id) 
                  & (sseq['direction_id'] == direction_id)
                  & (sseq['shape_id'] == shape_id)].reset_index()
    
    df['segment'] = ''
    
    # Split the shape in different segments
    line = shapes_route.loc[shapes_route.shape_id == shape_id, 'geometry'].values[0]
    trans_lines = shape_trans_lines.loc[shape_trans_lines.shape_id == shape_id, 'trans_lines']
    
    if len(trans_lines) == 2:
        # In case there is a line with only two stops
        df.loc[0, 'segment']  = line
        segments = segments.append(df)
    else:
        trans_lines_all = MultiLineString(list(trans_lines.values))
        trans_lines_cut = MultiLineString(list(trans_lines.values)[1:-1])

        # Split the shape in different segments, cut by the linestrings created before
        # The result is a geometry collection with the segments of the route
        result = split(line, trans_lines_cut)

        j = 0
        try: 
            if len(result)==len(trans_lines_all)-1:
                for i in range(0, len(result)):
                    df.loc[i, 'segment'] = result[i] 
                segments = segments.append(df)
            else:
                for i in range(0, len(df)-1):
                    #p = result[j].intersects(trans_lines_all[i])*result[j].intersects(trans_lines_all[i+1])     
                    p = result[j].distance(trans_lines_all[i]) + result[j].distance(trans_lines_all[i+1]) 
                    #if p==1:
                    if p < tolerance:
                        df.loc[i, 'segment'] = result[j]
                        j+=1
                    else:

                        #multi_line = result[j]
                        points = []
                        points.extend(result[j].coords)
                        while p > tolerance:
                            # combine them into a multi-linestring
                            j+=1
                            points.extend(result[j].coords)
                            merged_line = geometry.LineString(points)

                            #multi_line = geometry.MultiLineString([multi_line, result[j]])
                            #merged_line = ops.linemerge(multi_line)
                            p = merged_line.distance(trans_lines_all[i]) + merged_line.distance(trans_lines_all[i+1])
                            if p < tolerance:
                                j+=1

                        df.loc[i, 'segment'] = merged_line
                # deberia meter un if que verifique si j == len(result)-1. si el resultado es si, seguimos
                # si el resultado es no, agrego todos los result que queden en el ultimo segmento de df (df[len(df)-1])
                if j < len(result)-1:
                    points = []
                    for r in range(j, len(result)-1):
                        points.extend(result[r].coords)

                    merged_line = geometry.LineString(points)
                    df.loc[len(df)-1, 'segment'] = merged_line

                segments = segments.append(df)

        except IndexError:
            continue

osm_style = gpd.GeoDataFrame()

for index, row in shapes_route.iterrows():
    shape_id = row.shape_id
    df = segments.loc[segments.shape_id==shape_id,:].reset_index()
    s = df['stop_id']
    s_name = df['stop_name']
    df['end_stop_id'] = ''
    df['end_stop_name'] = ''
    
    for i in range(0, len(df)-1):
        df.loc[i, 'end_stop_id'] = s.iloc[i+1]
        df.loc[i, 'end_stop_name'] = s_name.iloc[i+1]
        
    osm_style = osm_style.append(df)
    
osm_style = osm_style.loc[:,['route_id', 'direction_id', 'stop_sequence',
                             'stop_id', 'stop_name', 'end_stop_id',
                             'end_stop_name', 'shape_id', 'segment']]

osm_style.columns = ['route_id', 'direction_id', 'stop_sequence',
                     'start_stop_id', 'start_stop_name', 'end_stop_id',
                     'end_stop_name', 'shape_id', 'segment']

osm_style['segment_id'] = osm_style['start_stop_id'] + '_' + osm_style['end_stop_id']
osm_style['segment_name'] = osm_style['start_stop_name'] + ' - ' + osm_style['end_stop_name']


# Keep in mind that we are filtering out the empty segments
# This means we will have different lengths per shape_id
data = osm_style.loc[osm_style.segment!=''].drop(['segment'], axis=1)
geometry = osm_style.loc[osm_style.segment!='', 'segment']

segments_gdf = gpd.GeoDataFrame(data = data, geometry = geometry)

segments_filtered = segments_gdf.loc[:,['direction_id', 'start_stop_id', 'stop_sequence', 'segment_name','geometry']]

dwell_pivot = dwell_filtered.pivot_table('dwell_time_secs', index = ['stop_id', 'stop_name', 'direction_id', 'hour_of_day'], aggfunc='mean').reset_index()
dwell_gdf = pd.merge(dwell_pivot, segments_filtered, left_on = ['stop_id', 'direction_id'], right_on = ['start_stop_id', 'direction_id'],
                     how='left')
dwell_gdf = gpd.GeoDataFrame(data=dwell_gdf.drop('geometry', axis=1), geometry=dwell_gdf.geometry)
# # Save map_1 config to a file
# with open('segment_dwell_times_config.py', 'w') as f:
#    f.write('config = {}'.format(m1.config))

# load the config
%run segment_dwell_times_config.py

# Create map
m1 = kp.KeplerGl(height=600, config=config)
m1.add_data(data=dwell_gdf, name='Dwell time')
m1
```

    User Guide: https://github.com/keplergl/kepler.gl/blob/master/docs/keplergl-jupyter/user-guide.md
    


    KeplerGl(config={'version': 'v1', 'config': {'visState': {'filters': [{'dataId': 'Dwell time', 'id': '2a27u7uu…


## Impressions

My first impression is that for this data it makes more sense to show the bubbles at the stop level since having a dwell time by segment seems less logical.

Once we have the speeds, maybe it could be good to overlap the two or think of a better way of showing both.

# Dwell time at the stop level by day of the week and direction


```python
by_day_of_week = dwell_filtered.pivot_table('dwell_time_secs', index = ['stop_id','stop_name','day_of_week'], aggfunc = 'mean' ).reset_index().sort_values(by='day_of_week', ascending=True)
avg_by_day = dwell_filtered.pivot_table('dwell_time_secs', index = ['day_of_week'], aggfunc = 'mean' ).reset_index().sort_values(by='day_of_week', ascending=True)
#by_day_of_week.head()

import plotly.graph_objects as go

example = by_day_of_week.loc[by_day_of_week.stop_name=='BOSTON ST & ANGLESEA ST wb',:]

fig = px.bar(example, x = day_name, y = 'dwell_time_secs', template = 'simple_white')
fig.update_layout(title='Stop BOSTON ST & ANGLESEA ST wb', yaxis={'categoryorder':'category ascending'})

fig.add_trace(
    go.Scatter(
        x=day_name,
        y=avg_by_day.dwell_time_secs
    ))

fig.update_yaxes(title_text='Average Dweel time')
fig.update_xaxes(title_text='Day of week')
fig.update_layout(showlegend=False)
```


<div>


            <div id="384006bf-2c25-4f3c-a8ec-091cd676082f" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("384006bf-2c25-4f3c-a8ec-091cd676082f")) {
                    Plotly.newPlot(
                        '384006bf-2c25-4f3c-a8ec-091cd676082f',
                        [{"alignmentgroup": "True", "hoverlabel": {"namelength": 0}, "hovertemplate": "x=%{x}<br>dwell_time_secs=%{y}", "legendgroup": "", "marker": {"color": "#1F77B4"}, "name": "", "offsetgroup": "", "orientation": "v", "showlegend": false, "textposition": "auto", "type": "bar", "x": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"], "xaxis": "x", "y": [18.616266666666657, 17.409683544303782, 17.910318840579702, 17.901761786600492, 18.002846153846146, 18.19492537313433, 24.37373913043477], "yaxis": "y"}, {"type": "scatter", "x": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"], "y": [13.547247511910406, 13.993457305502986, 13.688185778374972, 13.7369883924906, 13.782016362903123, 13.489688642574045, 13.325330358249683]}],
                        {"barmode": "relative", "legend": {"tracegroupgap": 0}, "margin": {"t": 60}, "showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "title": {"text": "Stop BOSTON ST & ANGLESEA ST wb"}, "xaxis": {"anchor": "y", "domain": [0.0, 1.0], "title": {"text": "Day of week"}}, "yaxis": {"anchor": "x", "categoryorder": "category ascending", "domain": [0.0, 1.0], "title": {"text": "Average Dweel time"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('384006bf-2c25-4f3c-a8ec-091cd676082f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


# Dwell time at the stop level by day of the week, direction and time of day


```python
by_day_of_week_hour = dwell_filtered.pivot_table('dwell_time_secs', index = ['day_of_week', 'hour_of_day'], aggfunc = 'mean' ).reset_index()
by_day_of_week_hour.day_of_week = by_day_of_week_hour.day_of_week.apply(lambda x: day_name[x])
#by_day_of_week_hour.head(2)
```

## Heatmap #1: Days in y axis, hours in x axis

This plot makes it very easy to see how the dwell time changes over the different days and times of the day:
    
    - Every weekday has a very similar behavior (at least for this line).
    - Saturday and Sundays also seem very similar.


```python
# Heatmap

fig = go.Figure(data=go.Heatmap(
                   z=by_day_of_week_hour.dwell_time_secs,
                   y=by_day_of_week_hour.day_of_week,
                   x=by_day_of_week_hour.hour_of_day,
                   hoverongaps = False,
                   colorscale=px.colors.colorbrewer.RdYlGn, 
                   reversescale=True
))

fig.update_yaxes(title_text='Day of week', autorange='reversed')
fig.update_xaxes(title_text='Hour of day')
fig.update_layout(showlegend=False)
fig.update_layout(title='Heatmap showing dwell time by hour and day of week')

fig.show()
```


<div>


            <div id="4b8b798a-aa71-43ae-ad2d-d04b9e1d031a" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("4b8b798a-aa71-43ae-ad2d-d04b9e1d031a")) {
                    Plotly.newPlot(
                        '4b8b798a-aa71-43ae-ad2d-d04b9e1d031a',
                        [{"colorscale": [[0.0, "rgb(165,0,38)"], [0.1, "rgb(215,48,39)"], [0.2, "rgb(244,109,67)"], [0.3, "rgb(253,174,97)"], [0.4, "rgb(254,224,139)"], [0.5, "rgb(255,255,191)"], [0.6, "rgb(217,239,139)"], [0.7, "rgb(166,217,106)"], [0.8, "rgb(102,189,99)"], [0.9, "rgb(26,152,80)"], [1.0, "rgb(0,104,55)"]], "hoverongaps": false, "reversescale": true, "type": "heatmap", "x": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "y": ["Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday"], "z": [13.196726457399109, 9.917517084282464, 10.26010799136069, 8.026569343065699, 11.255147247119071, 11.620074224021586, 12.344619454619439, 13.806804043545863, 13.743303509979366, 13.532144420131274, 13.841822011585059, 14.252145636464723, 14.642226720647775, 15.868938257357183, 16.48181918412346, 15.009041723979854, 14.488298710601688, 14.212634637737317, 14.050218196788803, 12.996702412868613, 12.355942492012796, 11.218036175710601, 12.055632478632464, 11.341263440860226, 11.238095238095251, 10.810609137055842, 9.359005235602087, 11.74870801033591, 11.59945175438595, 11.807876106194714, 13.079788101059492, 15.022943738656977, 14.764067853543791, 13.857297510865271, 14.085286525554984, 14.214296130117777, 15.17724873666478, 16.304300268096522, 16.607950775815933, 16.299000846740082, 15.524238947013133, 14.0502895174709, 14.134221052631553, 12.517961445783136, 11.763320941759595, 11.399580200501257, 12.238531353135318, 11.389388489208631, 11.064852734922852, 10.92638603696099, 10.460077669902914, 10.900700218818384, 10.869235209235205, 12.03494611457742, 12.693923725705881, 14.192559166929595, 13.800625748502963, 13.28074311580535, 14.357547357926215, 14.753364681295723, 15.364947683109131, 15.68022854240732, 16.600043562439488, 15.125480112494943, 14.660248528449976, 13.988516552154909, 14.392070151306728, 12.44600084459463, 12.226898734177206, 11.557449417535276, 11.975973487986748, 11.310276243093933, 12.086633802816896, 10.441758893280626, 10.022371541501983, 11.754383259911894, 12.238872549019622, 11.82986773736419, 12.351045495631164, 14.5863472563473, 13.71387389659521, 13.637866121935858, 14.498916666666682, 14.602046147148487, 15.105792185487337, 15.624788494077858, 16.44167805294617, 15.157746666666652, 14.787943205944787, 13.594013952240449, 13.865406381697806, 13.072661960132892, 11.81843047034766, 11.703473022524886, 12.351488178025056, 11.663646694214872, 11.043983628922241, 10.738946322067592, 10.104604166666672, 11.588312236286923, 12.345416078984464, 12.113590637450173, 12.346119309262162, 13.94618954974986, 13.707720880870216, 13.881119298245604, 14.731694307485418, 14.920969669117605, 15.696889908256887, 16.141011235955045, 16.362246575342482, 15.44257534246574, 14.701577099648937, 14.06124353741498, 13.312789443813855, 12.982281441717772, 12.081531393568168, 12.092609649122798, 12.049059771658817, 12.33583333333334, 12.312028301886794, 11.58868894601542, 9.364353312302843, 11.978466257668716, 10.232044943820219, 11.217043740573146, 12.302984615384615, 13.718899676375417, 13.987315508021402, 14.61065934065934, 14.663078313253022, 13.522009237875292, 13.811963574274321, 13.782213162492692, 14.790386167146952, 14.614448818897621, 14.345414398064127, 13.440407701019259, 13.666789473684199, 13.091148482362597, 12.713861171366617, 12.033970588235295, 11.92718104495747, 12.804999999999994, 13.785396419437353, 11.06669491525424, 9.686391304347826, 9.50038596491228, 9.98637883008356, 9.226526806526813, 11.594452214452222, 12.609598145285934, 13.4963994374121, 14.366048064085442, 15.736020864381521, 13.925652694610793, 13.97856092436975, 15.146079027355619, 15.42824175824174, 14.707704026115355, 13.525028121484826, 13.43816326530613, 13.58399572649573, 13.3712925170068, 11.389425403225804, 13.610208333333334, 12.636388308977027, 12.511943319838046]}],
                        {"showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Heatmap showing dwell time by hour and day of week"}, "xaxis": {"title": {"text": "Hour of day"}}, "yaxis": {"autorange": "reversed", "title": {"text": "Day of week"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('4b8b798a-aa71-43ae-ad2d-d04b9e1d031a');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Heatmap #2: Days in x axis, hours in y axis


```python
## Heatmap

fig = go.Figure(data=go.Heatmap(
                   z=by_day_of_week_hour.dwell_time_secs,
                   x=by_day_of_week_hour.day_of_week,
                   y=by_day_of_week_hour.hour_of_day,
                   hoverongaps = False,
                   colorscale=px.colors.colorbrewer.RdYlGn, 
                   reversescale=True
))

fig.update_yaxes(title_text='Hour of day', autorange='reversed')
fig.update_xaxes(title_text='Day of week')
fig.update_layout(showlegend=False)
fig.update_layout(title='Heatmap showing dwell time by hour and day of week')

fig.show()
```


<div>


            <div id="b1b9a9eb-064b-4cc3-9573-e1d1f359544d" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("b1b9a9eb-064b-4cc3-9573-e1d1f359544d")) {
                    Plotly.newPlot(
                        'b1b9a9eb-064b-4cc3-9573-e1d1f359544d',
                        [{"colorscale": [[0.0, "rgb(165,0,38)"], [0.1, "rgb(215,48,39)"], [0.2, "rgb(244,109,67)"], [0.3, "rgb(253,174,97)"], [0.4, "rgb(254,224,139)"], [0.5, "rgb(255,255,191)"], [0.6, "rgb(217,239,139)"], [0.7, "rgb(166,217,106)"], [0.8, "rgb(102,189,99)"], [0.9, "rgb(26,152,80)"], [1.0, "rgb(0,104,55)"]], "hoverongaps": false, "reversescale": true, "type": "heatmap", "x": ["Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday"], "y": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "z": [13.196726457399109, 9.917517084282464, 10.26010799136069, 8.026569343065699, 11.255147247119071, 11.620074224021586, 12.344619454619439, 13.806804043545863, 13.743303509979366, 13.532144420131274, 13.841822011585059, 14.252145636464723, 14.642226720647775, 15.868938257357183, 16.48181918412346, 15.009041723979854, 14.488298710601688, 14.212634637737317, 14.050218196788803, 12.996702412868613, 12.355942492012796, 11.218036175710601, 12.055632478632464, 11.341263440860226, 11.238095238095251, 10.810609137055842, 9.359005235602087, 11.74870801033591, 11.59945175438595, 11.807876106194714, 13.079788101059492, 15.022943738656977, 14.764067853543791, 13.857297510865271, 14.085286525554984, 14.214296130117777, 15.17724873666478, 16.304300268096522, 16.607950775815933, 16.299000846740082, 15.524238947013133, 14.0502895174709, 14.134221052631553, 12.517961445783136, 11.763320941759595, 11.399580200501257, 12.238531353135318, 11.389388489208631, 11.064852734922852, 10.92638603696099, 10.460077669902914, 10.900700218818384, 10.869235209235205, 12.03494611457742, 12.693923725705881, 14.192559166929595, 13.800625748502963, 13.28074311580535, 14.357547357926215, 14.753364681295723, 15.364947683109131, 15.68022854240732, 16.600043562439488, 15.125480112494943, 14.660248528449976, 13.988516552154909, 14.392070151306728, 12.44600084459463, 12.226898734177206, 11.557449417535276, 11.975973487986748, 11.310276243093933, 12.086633802816896, 10.441758893280626, 10.022371541501983, 11.754383259911894, 12.238872549019622, 11.82986773736419, 12.351045495631164, 14.5863472563473, 13.71387389659521, 13.637866121935858, 14.498916666666682, 14.602046147148487, 15.105792185487337, 15.624788494077858, 16.44167805294617, 15.157746666666652, 14.787943205944787, 13.594013952240449, 13.865406381697806, 13.072661960132892, 11.81843047034766, 11.703473022524886, 12.351488178025056, 11.663646694214872, 11.043983628922241, 10.738946322067592, 10.104604166666672, 11.588312236286923, 12.345416078984464, 12.113590637450173, 12.346119309262162, 13.94618954974986, 13.707720880870216, 13.881119298245604, 14.731694307485418, 14.920969669117605, 15.696889908256887, 16.141011235955045, 16.362246575342482, 15.44257534246574, 14.701577099648937, 14.06124353741498, 13.312789443813855, 12.982281441717772, 12.081531393568168, 12.092609649122798, 12.049059771658817, 12.33583333333334, 12.312028301886794, 11.58868894601542, 9.364353312302843, 11.978466257668716, 10.232044943820219, 11.217043740573146, 12.302984615384615, 13.718899676375417, 13.987315508021402, 14.61065934065934, 14.663078313253022, 13.522009237875292, 13.811963574274321, 13.782213162492692, 14.790386167146952, 14.614448818897621, 14.345414398064127, 13.440407701019259, 13.666789473684199, 13.091148482362597, 12.713861171366617, 12.033970588235295, 11.92718104495747, 12.804999999999994, 13.785396419437353, 11.06669491525424, 9.686391304347826, 9.50038596491228, 9.98637883008356, 9.226526806526813, 11.594452214452222, 12.609598145285934, 13.4963994374121, 14.366048064085442, 15.736020864381521, 13.925652694610793, 13.97856092436975, 15.146079027355619, 15.42824175824174, 14.707704026115355, 13.525028121484826, 13.43816326530613, 13.58399572649573, 13.3712925170068, 11.389425403225804, 13.610208333333334, 12.636388308977027, 12.511943319838046]}],
                        {"showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Heatmap showing dwell time by hour and day of week"}, "xaxis": {"title": {"text": "Day of week"}}, "yaxis": {"autorange": "reversed", "title": {"text": "Hour of day"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('b1b9a9eb-064b-4cc3-9573-e1d1f359544d');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Heatmap #3: for one specific stop


```python
example_1 = dwell_filtered.loc[dwell_filtered.stop_name=='BOSTON ST & ANGLESEA ST wb'
                            ,:].pivot_table('dwell_time_secs', index = ['day_of_week', 'hour_of_day'], aggfunc = 'mean' ).reset_index()
example_1.day_of_week = example_1.day_of_week.apply(lambda x: day_name[x])
#example_1.head(2)

# Heatmap

fig = go.Figure(data=go.Heatmap(
                   z=example_1.dwell_time_secs,
                   y=example_1.day_of_week,
                   x=example_1.hour_of_day,
                   hoverongaps = False,
                   colorscale=px.colors.colorbrewer.RdYlGn, 
                   reversescale=True
))

fig.update_yaxes(title_text='Day of week', autorange='reversed')
fig.update_xaxes(title_text='Hour of day')
fig.update_layout(showlegend=False)
fig.update_layout(title='Stop BOSTON ST & ANGLESEA ST wb')

fig.show()
```


<div>


            <div id="6f3832e0-b289-4eb2-aa70-f272c510fd3a" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("6f3832e0-b289-4eb2-aa70-f272c510fd3a")) {
                    Plotly.newPlot(
                        '6f3832e0-b289-4eb2-aa70-f272c510fd3a',
                        [{"colorscale": [[0.0, "rgb(165,0,38)"], [0.1, "rgb(215,48,39)"], [0.2, "rgb(244,109,67)"], [0.3, "rgb(253,174,97)"], [0.4, "rgb(254,224,139)"], [0.5, "rgb(255,255,191)"], [0.6, "rgb(217,239,139)"], [0.7, "rgb(166,217,106)"], [0.8, "rgb(102,189,99)"], [0.9, "rgb(26,152,80)"], [1.0, "rgb(0,104,55)"]], "hoverongaps": false, "reversescale": true, "type": "heatmap", "x": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "y": ["Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Monday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Tuesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Wednesday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Thursday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Friday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Saturday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday", "Sunday"], "z": [24.36, 20.605, 26.8425, 21.12333333333333, 20.928, 20.620625, 18.470869565217388, 24.938823529411767, 14.050526315789472, 18.856470588235293, 19.402499999999996, 15.788461538461538, 20.410666666666664, 26.395000000000003, 19.492857142857144, 16.941176470588236, 19.29409090909091, 14.885, 20.147000000000002, 18.798750000000002, 8.948571428571428, 12.7, 12.065, 16.256666666666668, 10.7275, 14.816666666666668, 16.4725, 24.174285714285713, 20.48923076923077, 18.075517241379313, 24.374999999999996, 19.163, 19.018333333333334, 20.1, 16.840714285714284, 21.17, 21.976875, 21.555000000000003, 16.944499999999998, 16.361363636363638, 11.814, 15.851875, 13.34, 11.305, 3.4012500000000006, 14.57875, 10.684999999999999, 10.866000000000001, 1.39, 15.8925, 0.42, 23.877777777777776, 21.86470588235294, 21.082142857142856, 23.864499999999996, 13.92269230769231, 20.198999999999998, 18.861428571428572, 19.841666666666665, 26.867999999999995, 21.440714285714286, 20.068749999999998, 14.599999999999998, 18.7775, 14.130416666666667, 19.579375000000002, 15.17388888888889, 14.868461538461538, 5.5175, 3.6371428571428575, 11.966, 22.439999999999998, 10.785, 11.532, 6.35, 24.633076923076924, 19.93333333333333, 22.634545454545457, 23.446818181818188, 17.700666666666667, 18.20476190476191, 15.095652173913043, 14.770555555555555, 23.855999999999998, 17.2975, 23.363684210526316, 14.408800000000003, 16.510344827586206, 15.815416666666664, 23.89333333333333, 17.048666666666666, 8.912142857142856, 5.148888888888889, 5.13, 15.681999999999999, 17.866000000000003, 10.469999999999999, 11.568000000000001, 28.982, 18.739090909090912, 19.067272727272726, 19.680000000000003, 13.35148148148148, 20.44105263157895, 19.78095238095238, 22.047647058823532, 27.548235294117642, 22.56, 24.246315789473687, 13.574166666666668, 15.654, 12.991153846153848, 18.805555555555557, 16.774117647058826, 17.22285714285714, 5.261000000000001, 16.247, 7.24, 16.8525, 3.34, 9.06, 2.83, 31.875, 32.824, 26.82166666666667, 37.735, 19.7, 14.786923076923076, 14.36, 17.034615384615385, 16.513333333333335, 19.729285714285712, 23.05461538461538, 17.300666666666665, 23.733846153846155, 11.208, 15.332666666666665, 17.547000000000004, 10.973333333333334, 15.776666666666666, 17.95875, 4.33, 45.22, 31.39, 16.646666666666665, 23.075, 42.82666666666666, 27.6425, 27.27, 21.876666666666665, 17.208, 22.804285714285708, 28.942, 20.256, 26.0475, 34.902, 29.911666666666665, 21.204, 27.90857142857143, 18.028333333333332, 12.931666666666667, 25.87875, 24.67222222222222, 18.046666666666667, 28.67, 15.154000000000002]}],
                        {"showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "#2a3f5f"}, "error_y": {"color": "#2a3f5f"}, "marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "#E5ECF6", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "baxis": {"endlinecolor": "#2a3f5f", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "#2a3f5f"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "heatmapgl"}], "histogram": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 0, "ticks": ""}, "colorscale": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "#EBF0F8"}, "line": {"color": "white"}}, "header": {"fill": {"color": "#C8D4E3"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowcolor": "#2a3f5f", "arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 0, "ticks": ""}}, "colorscale": {"diverging": [[0, "#8e0152"], [0.1, "#c51b7d"], [0.2, "#de77ae"], [0.3, "#f1b6da"], [0.4, "#fde0ef"], [0.5, "#f7f7f7"], [0.6, "#e6f5d0"], [0.7, "#b8e186"], [0.8, "#7fbc41"], [0.9, "#4d9221"], [1, "#276419"]], "sequential": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]], "sequentialminus": [[0.0, "#0d0887"], [0.1111111111111111, "#46039f"], [0.2222222222222222, "#7201a8"], [0.3333333333333333, "#9c179e"], [0.4444444444444444, "#bd3786"], [0.5555555555555556, "#d8576b"], [0.6666666666666666, "#ed7953"], [0.7777777777777778, "#fb9f3a"], [0.8888888888888888, "#fdca26"], [1.0, "#f0f921"]]}, "colorway": ["#636efa", "#EF553B", "#00cc96", "#ab63fa", "#FFA15A", "#19d3f3", "#FF6692", "#B6E880", "#FF97FF", "#FECB52"], "font": {"color": "#2a3f5f"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "#E5ECF6", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "#E5ECF6", "polar": {"angularaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "radialaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "scene": {"xaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "yaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}, "zaxis": {"backgroundcolor": "#E5ECF6", "gridcolor": "white", "gridwidth": 2, "linecolor": "white", "showbackground": true, "ticks": "", "zerolinecolor": "white"}}, "shapedefaults": {"line": {"color": "#2a3f5f"}}, "ternary": {"aaxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "baxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}, "bgcolor": "#E5ECF6", "caxis": {"gridcolor": "white", "linecolor": "white", "ticks": ""}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}, "yaxis": {"automargin": true, "gridcolor": "white", "linecolor": "white", "ticks": "", "title": {"standoff": 15}, "zerolinecolor": "white", "zerolinewidth": 2}}}, "title": {"text": "Stop BOSTON ST & ANGLESEA ST wb"}, "xaxis": {"title": {"text": "Hour of day"}}, "yaxis": {"autorange": "reversed", "title": {"text": "Day of week"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('6f3832e0-b289-4eb2-aa70-f272c510fd3a');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Heatmap with polar coordinates


```python
theta = [0,15,30,45,60,75,90,105,120,135,150,165,180,195,210,225,240,255,
                                                     270,285,300,315,330,345]
theta = pd.DataFrame(theta)
theta.columns = ['degrees']
theta['hour_of_day'] = example_1.hour_of_day.unique()
example_1 = pd.merge(example_1, theta)
#example_1.head()
```


```python
# Bar Polar Plots

fig = px.bar_polar(example_1, r='day_of_week', theta='degrees', color='dwell_time_secs',
            template='simple_white',
            color_continuous_scale= px.colors.colorbrewer.RdYlGn)

# fig = px.line_polar(example_1, r='day_of_week', theta='degrees', color='dwell_time_secs',
#                     line_close=True, #animation_frame = 'day',
#                     hover_name = 'hour_of_day', template = 'plotly_dark',
#                     color_discrete_sequence=px.colors.sequential.Plasma_r)

fig.update_layout(
    polar = dict(
        radialaxis = dict(showticklabels=True),
        angularaxis = dict(tickmode = 'array', 
                           tickvals = example_1.degrees.unique(),
                           ticktext= example_1.hour_of_day.unique()
                           #showticklabels=False, 
                           #ticks=''
                          )
    )
)
```


<div>


            <div id="4b80bdad-076b-4c22-8c22-f7c022dada5f" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("4b80bdad-076b-4c22-8c22-f7c022dada5f")) {
                    Plotly.newPlot(
                        '4b80bdad-076b-4c22-8c22-f7c022dada5f',
                        [{"hoverlabel": {"namelength": 0}, "hovertemplate": "day_of_week=%{r}<br>degrees=%{theta}<br>dwell_time_secs=%{marker.color}", "legendgroup": "", "marker": {"color": [24.36, 10.7275, 10.866000000000001, 22.439999999999998, 17.866000000000003, 16.8525, 45.22, 20.605, 14.816666666666668, 1.39, 10.785, 10.469999999999999, 3.34, 31.39, 26.8425, 16.4725, 15.8925, 11.532, 11.568000000000001, 9.06, 16.646666666666665, 21.12333333333333, 0.42, 6.35, 2.83, 23.075, 20.928, 24.174285714285713, 23.877777777777776, 24.633076923076924, 28.982, 31.875, 42.82666666666666, 20.620625, 20.48923076923077, 21.86470588235294, 19.93333333333333, 18.739090909090912, 32.824, 27.6425, 18.470869565217388, 18.075517241379313, 21.082142857142856, 22.634545454545457, 19.067272727272726, 26.82166666666667, 27.27, 24.938823529411767, 24.374999999999996, 23.864499999999996, 23.446818181818188, 19.680000000000003, 37.735, 21.876666666666665, 14.050526315789472, 19.163, 13.92269230769231, 17.700666666666667, 13.35148148148148, 19.7, 17.208, 18.856470588235293, 19.018333333333334, 20.198999999999998, 18.20476190476191, 20.44105263157895, 14.786923076923076, 22.804285714285708, 19.402499999999996, 20.1, 18.861428571428572, 15.095652173913043, 19.78095238095238, 14.36, 28.942, 15.788461538461538, 16.840714285714284, 19.841666666666665, 14.770555555555555, 22.047647058823532, 17.034615384615385, 20.256, 20.410666666666664, 21.17, 26.867999999999995, 23.855999999999998, 27.548235294117642, 16.513333333333335, 26.0475, 26.395000000000003, 21.976875, 21.440714285714286, 17.2975, 22.56, 19.729285714285712, 34.902, 19.492857142857144, 21.555000000000003, 20.068749999999998, 23.363684210526316, 24.246315789473687, 23.05461538461538, 29.911666666666665, 16.941176470588236, 16.944499999999998, 14.599999999999998, 14.408800000000003, 13.574166666666668, 17.300666666666665, 21.204, 19.29409090909091, 16.361363636363638, 18.7775, 16.510344827586206, 15.654, 23.733846153846155, 27.90857142857143, 14.885, 11.814, 14.130416666666667, 15.815416666666664, 12.991153846153848, 11.208, 18.028333333333332, 20.147000000000002, 15.851875, 19.579375000000002, 23.89333333333333, 18.805555555555557, 15.332666666666665, 12.931666666666667, 18.798750000000002, 13.34, 15.17388888888889, 17.048666666666666, 16.774117647058826, 17.547000000000004, 25.87875, 8.948571428571428, 11.305, 14.868461538461538, 8.912142857142856, 17.22285714285714, 10.973333333333334, 24.67222222222222, 12.7, 3.4012500000000006, 5.5175, 5.148888888888889, 5.261000000000001, 15.776666666666666, 18.046666666666667, 12.065, 14.57875, 3.6371428571428575, 5.13, 16.247, 17.95875, 28.67, 16.256666666666668, 10.684999999999999, 11.966, 15.681999999999999, 7.24, 4.33, 15.154000000000002], "coloraxis": "coloraxis"}, "name": "", "r": ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Wednesday", "Thursday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"], "showlegend": false, "subplot": "polar", "theta": [0, 0, 0, 0, 0, 0, 0, 15, 15, 15, 15, 15, 15, 15, 30, 30, 30, 30, 30, 30, 30, 45, 45, 45, 45, 45, 60, 60, 60, 60, 60, 60, 60, 75, 75, 75, 75, 75, 75, 75, 90, 90, 90, 90, 90, 90, 90, 105, 105, 105, 105, 105, 105, 105, 120, 120, 120, 120, 120, 120, 120, 135, 135, 135, 135, 135, 135, 135, 150, 150, 150, 150, 150, 150, 150, 165, 165, 165, 165, 165, 165, 165, 180, 180, 180, 180, 180, 180, 180, 195, 195, 195, 195, 195, 195, 195, 210, 210, 210, 210, 210, 210, 210, 225, 225, 225, 225, 225, 225, 225, 240, 240, 240, 240, 240, 240, 240, 255, 255, 255, 255, 255, 255, 255, 270, 270, 270, 270, 270, 270, 270, 285, 285, 285, 285, 285, 285, 285, 300, 300, 300, 300, 300, 300, 300, 315, 315, 315, 315, 315, 315, 315, 330, 330, 330, 330, 330, 330, 330, 345, 345, 345, 345, 345, 345, 345], "type": "barpolar"}],
                        {"barmode": "relative", "coloraxis": {"colorbar": {"title": {"text": "dwell_time_secs"}}, "colorscale": [[0.0, "rgb(165,0,38)"], [0.1, "rgb(215,48,39)"], [0.2, "rgb(244,109,67)"], [0.3, "rgb(253,174,97)"], [0.4, "rgb(254,224,139)"], [0.5, "rgb(255,255,191)"], [0.6, "rgb(217,239,139)"], [0.7, "rgb(166,217,106)"], [0.8, "rgb(102,189,99)"], [0.9, "rgb(26,152,80)"], [1.0, "rgb(0,104,55)"]]}, "legend": {"tracegroupgap": 0}, "margin": {"t": 60}, "polar": {"angularaxis": {"direction": "clockwise", "rotation": 90, "tickmode": "array", "ticktext": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "tickvals": [0, 15, 30, 45, 60, 75, 90, 105, 120, 135, 150, 165, 180, 195, 210, 225, 240, 255, 270, 285, 300, 315, 330, 345]}, "domain": {"x": [0.0, 1.0], "y": [0.0, 1.0]}, "radialaxis": {"showticklabels": true}}, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('4b80bdad-076b-4c22-8c22-f7c022dada5f');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>


## Impressions on heatmaps

I really like the first two: it displays a lot of information in a very easy way to understand it. You can have data for the whole day for different days and see how they look alike or where they vary.

Will also try that displaying the data for the whole month.

# Average dwell time by stop


```python
by_stop_dir_and_hour = dwell_filtered.pivot_table('dwell_time_secs', index = ['hour_of_day', 'direction_id', 'stop_id','stop_name'], aggfunc = ['mean', 'std'] ).reset_index()
by_stop_dir_and_hour.columns = ['_'.join(col).strip() for col in by_stop_dir_and_hour.columns.values]
#by_stop_dir_and_hour.head()


#by_dir_and_hour = dwell_filtered.pivot_table('dwell_time_secs', index = ['hour_of_day', 'direction_id'], aggfunc = 'mean' ).reset_index()
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
      <th>hour_of_day_</th>
      <th>direction_id_</th>
      <th>stop_id_</th>
      <th>stop_name_</th>
      <th>mean_dwell_time_secs</th>
      <th>std_dwell_time_secs</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0</td>
      <td>0</td>
      <td>10412</td>
      <td>DUNDALK &amp; WILLIAMS ns sb</td>
      <td>1.931154</td>
      <td>1.160353</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0</td>
      <td>0</td>
      <td>10768</td>
      <td>DUNDALK AVE &amp; HOLABIRD AVE fs nb</td>
      <td>14.386923</td>
      <td>8.993079</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0</td>
      <td>0</td>
      <td>11167</td>
      <td>DUNDALK AVE &amp; BAYSHIP RD nb</td>
      <td>9.134400</td>
      <td>11.536901</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0</td>
      <td>0</td>
      <td>11420</td>
      <td>PRESIDENT ST &amp; FAWN ST nb</td>
      <td>3.871563</td>
      <td>4.097762</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0</td>
      <td>0</td>
      <td>12180</td>
      <td>RIGGS AVE &amp; CAREY ST fs wb</td>
      <td>8.714074</td>
      <td>7.726063</td>
    </tr>
  </tbody>
</table>
</div>



Shows the average dwell time by hour on a specific stop as a line and the shadow shows the standard deviation for each hour of the day.

- This might be the easiest way to read the information.
- The shadow gives an idea of how representative the average is and helps the user to identify if there are hours of the day that vary more than others (for example 3 am has a big lower variation).


```python
fig = go.Figure()

upper_bound = go.Scatter(
    name='Upper Bound',
    x=example2.hour_of_day_,
    y=(example2.mean_dwell_time_secs + example2.std_dwell_time_secs),
    mode='lines',
    marker=dict(color="#444"),
    line=dict(width=0),
    fillcolor='#F0F0F0',
    fill='tonexty',
    opacity = 0.5)

trace = go.Scatter(
    name='Dwell time',
    x=example2.hour_of_day_, 
    y=example2.mean_dwell_time_secs,
    mode='lines',
    line=dict(color='rgb(31, 119, 180)'),
    fillcolor='#F0F0F0',
    fill='tonexty',
    opacity = 0.5)

lower_bound = go.Scatter(
    name='Lower Bound',
    x=example2.hour_of_day_,
    y=(example2.mean_dwell_time_secs - example2.std_dwell_time_secs),
    marker=dict(color="#444"),
    line=dict(width=0),
    mode='lines')

data = [lower_bound, trace, upper_bound]

layout = go.Layout(
    yaxis=dict(title='Average dwell time'),
    title='Average dwell time by hour of day in stop BOSTON ST & ANGLESEA ST wb ',
    showlegend = False, template = 'simple_white')

fig = go.Figure(data=data, layout=layout)
#py.iplot(fig, filename='pandas-continuous-error-bars')

fig.show()
```


<div>


            <div id="0e886878-ede2-4fbd-8aff-d3c1fabe64ae" class="plotly-graph-div" style="height:525px; width:100%;"></div>
            <script type="text/javascript">
                require(["plotly"], function(Plotly) {
                    window.PLOTLYENV=window.PLOTLYENV || {};

                if (document.getElementById("0e886878-ede2-4fbd-8aff-d3c1fabe64ae")) {
                    Plotly.newPlot(
                        '0e886878-ede2-4fbd-8aff-d3c1fabe64ae',
                        [{"line": {"width": 0}, "marker": {"color": "#444"}, "mode": "lines", "name": "Lower Bound", "type": "scatter", "x": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "y": [5.601602426063101, 0.31453467596452533, 3.754211390973637, -3.640694227994498, 16.999174367302643, 10.437914039896022, 10.595630467524146, 13.025214639698213, 5.035127307426226, 8.39259503403252, 8.028879042603906, 6.057303262348464, 11.605931825457443, 10.004997171110826, 12.533766595902707, 5.162224790794399, 5.68224573381613, 2.9320175149639756, 5.888400062646392, 6.074868799642182, 2.2864341070452667, -1.1026393523379294, 0.2666274955030765, 2.41381052871043]}, {"fill": "tonexty", "fillcolor": "#F0F0F0", "line": {"color": "rgb(31, 119, 180)"}, "mode": "lines", "name": "Dwell time", "opacity": 0.5, "type": "scatter", "x": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "y": [18.917857142857144, 12.805555555555555, 15.337500000000004, 14.89, 26.06125, 21.158571428571435, 20.33084415584415, 23.572336448598126, 15.849323308270666, 19.03834782608695, 18.33276785714286, 17.975306122448984, 23.320686274509796, 22.00969696969698, 22.56970297029703, 15.638253968253965, 18.281632653061223, 13.827703703703703, 18.631616161616158, 17.1074, 14.061805555555555, 8.341818181818182, 12.148823529411763, 12.358928571428569]}, {"fill": "tonexty", "fillcolor": "#F0F0F0", "line": {"width": 0}, "marker": {"color": "#444"}, "mode": "lines", "name": "Upper Bound", "opacity": 0.5, "type": "scatter", "x": [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21, 22, 23], "y": [32.234111859651186, 25.296576435146584, 26.92078860902637, 33.4206942279945, 35.12332563269736, 31.879228817246847, 30.066057844164156, 34.11945825749804, 26.663519309115106, 29.684100618141382, 28.636656671681813, 29.893308982549506, 35.03544072356215, 34.014396768283135, 32.605639344691355, 26.11428314571353, 30.881019572306315, 24.72338989244343, 31.374832260585926, 28.139931200357815, 25.837177004065843, 17.78627571597429, 24.03101956332045, 22.304046614146706]}],
                        {"showlegend": false, "template": {"data": {"bar": [{"error_x": {"color": "rgb(36,36,36)"}, "error_y": {"color": "rgb(36,36,36)"}, "marker": {"line": {"color": "white", "width": 0.5}}, "type": "bar"}], "barpolar": [{"marker": {"line": {"color": "white", "width": 0.5}}, "type": "barpolar"}], "carpet": [{"aaxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "baxis": {"endlinecolor": "rgb(36,36,36)", "gridcolor": "white", "linecolor": "white", "minorgridcolor": "white", "startlinecolor": "rgb(36,36,36)"}, "type": "carpet"}], "choropleth": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "choropleth"}], "contour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "contour"}], "contourcarpet": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "contourcarpet"}], "heatmap": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmap"}], "heatmapgl": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "heatmapgl"}], "histogram": [{"marker": {"line": {"color": "white", "width": 0.6}}, "type": "histogram"}], "histogram2d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2d"}], "histogram2dcontour": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "histogram2dcontour"}], "mesh3d": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "type": "mesh3d"}], "parcoords": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "parcoords"}], "pie": [{"automargin": true, "type": "pie"}], "scatter": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter"}], "scatter3d": [{"line": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatter3d"}], "scattercarpet": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattercarpet"}], "scattergeo": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergeo"}], "scattergl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattergl"}], "scattermapbox": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scattermapbox"}], "scatterpolar": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolar"}], "scatterpolargl": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterpolargl"}], "scatterternary": [{"marker": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "type": "scatterternary"}], "surface": [{"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}, "colorscale": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "type": "surface"}], "table": [{"cells": {"fill": {"color": "rgb(237,237,237)"}, "line": {"color": "white"}}, "header": {"fill": {"color": "rgb(217,217,217)"}, "line": {"color": "white"}}, "type": "table"}]}, "layout": {"annotationdefaults": {"arrowhead": 0, "arrowwidth": 1}, "coloraxis": {"colorbar": {"outlinewidth": 1, "tickcolor": "rgb(36,36,36)", "ticks": "outside"}}, "colorscale": {"diverging": [[0.0, "rgb(103,0,31)"], [0.1, "rgb(178,24,43)"], [0.2, "rgb(214,96,77)"], [0.3, "rgb(244,165,130)"], [0.4, "rgb(253,219,199)"], [0.5, "rgb(247,247,247)"], [0.6, "rgb(209,229,240)"], [0.7, "rgb(146,197,222)"], [0.8, "rgb(67,147,195)"], [0.9, "rgb(33,102,172)"], [1.0, "rgb(5,48,97)"]], "sequential": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]], "sequentialminus": [[0.0, "#440154"], [0.1111111111111111, "#482878"], [0.2222222222222222, "#3e4989"], [0.3333333333333333, "#31688e"], [0.4444444444444444, "#26828e"], [0.5555555555555556, "#1f9e89"], [0.6666666666666666, "#35b779"], [0.7777777777777778, "#6ece58"], [0.8888888888888888, "#b5de2b"], [1.0, "#fde725"]]}, "colorway": ["#1F77B4", "#FF7F0E", "#2CA02C", "#D62728", "#9467BD", "#8C564B", "#E377C2", "#7F7F7F", "#BCBD22", "#17BECF"], "font": {"color": "rgb(36,36,36)"}, "geo": {"bgcolor": "white", "lakecolor": "white", "landcolor": "white", "showlakes": true, "showland": true, "subunitcolor": "white"}, "hoverlabel": {"align": "left"}, "hovermode": "closest", "mapbox": {"style": "light"}, "paper_bgcolor": "white", "plot_bgcolor": "white", "polar": {"angularaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "radialaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "scene": {"xaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "zaxis": {"backgroundcolor": "white", "gridcolor": "rgb(232,232,232)", "gridwidth": 2, "linecolor": "rgb(36,36,36)", "showbackground": true, "showgrid": false, "showline": true, "ticks": "outside", "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}, "shapedefaults": {"fillcolor": "black", "line": {"width": 0}, "opacity": 0.3}, "ternary": {"aaxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "baxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}, "bgcolor": "white", "caxis": {"gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside"}}, "title": {"x": 0.05}, "xaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}, "yaxis": {"automargin": true, "gridcolor": "rgb(232,232,232)", "linecolor": "rgb(36,36,36)", "showgrid": false, "showline": true, "ticks": "outside", "title": {"standoff": 15}, "zeroline": false, "zerolinecolor": "rgb(36,36,36)"}}}, "title": {"text": "Average dwell time by hour of day in stop BOSTON ST & ANGLESEA ST wb "}, "yaxis": {"title": {"text": "Average dwell time"}}},
                        {"responsive": true}
                    ).then(function(){

var gd = document.getElementById('0e886878-ede2-4fbd-8aff-d3c1fabe64ae');
var x = new MutationObserver(function (mutations, observer) {{
        var display = window.getComputedStyle(gd).display;
        if (!display || display === 'none') {{
            console.log([gd, 'removed!']);
            Plotly.purge(gd);
            observer.disconnect();
        }}
}});

// Listen for the removal of the full notebook cells
var notebookContainer = gd.closest('#notebook-container');
if (notebookContainer) {{
    x.observe(notebookContainer, {childList: true});
}}

// Listen for the clearing of the current output cell
var outputEl = gd.closest('.output');
if (outputEl) {{
    x.observe(outputEl, {childList: true});
}}

                        })
                };
                });
            </script>
        </div>

