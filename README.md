# Traffic_data

import numpy as np
import pandas as pd
import seaborn as sb
import matplotlib as plt
import os
import csv
import datetime

data=pd.read_csv('Traffic_Violations.csv')

data

data.info()

data["Color"].value_counts()

data["Arrest Type"].value_counts()

def parse_float(x):
    try:
        x=float(x)
    except Exception:
        x=0
    return x
data["Longitude"]=data["Longitude"].apply(parse_float)
data["Latitude"]=data["Latitude"].apply(parse_float)

def parse_full_date(row):
    date = datetime.datetime.strptime(row["Date Of Stop"],"%m-%d-%YT%H:%M:%S")
    time = row["Time Of Stop"].split(":")
    date=date.replace(hour = int(time[0]), minute = int(time[1]), second = int(time[2]))
    return date
data["date"]=data.apply(parse_full_date, axis=1)

%matplotlib inline

plt.hist(data["date"].dt.weekday,bins=6)

plt.hist(data["date"].dt.hour, bins=24)

last_year = data[data["date"] > datetime.datetime(year=2015, month=2, day=18)]

morning_rush = last_year[(last_year["date"].dt.weekday < 5) & (last_year["date"].dt.hour > 5) & (last_year["date"].dt.hour < 10)]
print(morning_rush.shape)
last_year.shape

import folium
from folium import plugins

data_map = folium.Map(location=[39.0836, -77.1483], zoom_start=11)
marker_cluster = folium.MarkerCluster().add_to(data_map)
for name, row in morning_rush.iloc[:1000].iterrows():
    folium.Marker([row["longitude"], row["latitude"]], popup=row["description"]).add_to(marker_cluster)
data_map.create_map('stops.html')
data_map

data_heatmap = folium.Map(location=[39.0836, -77.1483], zoom_start=11)
data_heatmap.add_children(plugins.HeatMap([[row["longitude"], row["latitude"]] for name, row in morning_rush.iloc[:1000].iterrows()]))
data_heatmap.save("heatmap.html")
data_heatmap
