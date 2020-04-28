# COVID19 Visualization in Every Region in Indonesia

- Farhan Alfariqi    : 1301161770
- Fikhri Masri       : 1301164662
- Taufik Fathurahman : 1301160790

COVID19 merupakan fenomena besar yang sedang melanda dunia secara global. Tak sedikit korban jiwa berjatuhan akibatnya virus tersebut. Tak terkecuali Indonesia, hingga saat ini (28 April 2020 Pukul 14:35) telah terindikasi positif sekitar 9.096 jiwa. Berikut ini merupakan program yang menampilkan persebaran kasus COVID19 di Indonesia dengan menggunakan library Bokeh yang bermanfaat dalam menampilkan visualisasi dengan cara lebih interaktif.


## Getting Started

Berikut ini merupakan instruksi dasar yang diperlukan untuk menjalankan program Visualisasi COVID19 di Tiap Daerah Indonesia dengan menggunakan Bokeh Library

### Prerequisites

Langkah utama yaitu menginstall library yang diperlukan, seperti berikut:

```python
 pip install numpy
 pip install pandas
 pip install geopandas
```

### Installing

Langkah selanjutnya yaitu mengimport library yang telah diinstal kedalam teks editor (Jupyter Notebook) :

```python
 import numpy as np
 import pandas as pd
```

Selain itu juga, tambahkan beberapa extension dari library Bokeh seperti berikut:

```python
from bokeh.plotting import figure, show
from bokeh.io import output_notebook
from bokeh.models.tools import HoverTool
from bokeh.models import ColumnDataSource
from bokeh.transform import dodge
from bokeh.models import Panel, Tabs

output_notebook()
```

Langkah selanjutnya yaitu me-load dataset dengan cara berikut:
```python
df_provinsi   = pd.read_csv('https://raw.githubusercontent.com/farhanalfaa/covid19-visualization/master/dataset/Indo/data%20provinsi.csv')
```

## Running the tests

Berikut ini merupakan potongan source code untuk memudahkan memahami penjelasan mengenai tiap code yang diberikan

### Bokeh All Province Visualization

**Convert into Dictionary**

```python
data = {'provinsi'    : df_provinsi['Provinsi_Asal'].tolist(),
        'Kasus'       : df_provinsi['Kasus'].tolist(),
        'Sembuh'      : df_provinsi['Sembuh'].tolist(),
        'Meninggal'   : df_provinsi['Meninggal'].tolist()}

source = ColumnDataSource(data=data)
```
**Setup vbar Glyphs**

```python
p = figure(x_range=provinsi, y_range=(0, 1000), plot_height=500, plot_width=2000, title="Data Persebaran Virus COVID-19 di Indonesia", toolbar_location="left")

p.vbar(x=dodge('provinsi', -0.25, range=p.x_range), top='Kasus', width=0.2, source=source,
       color="#c9d9d3", legend_label="Kasus")

p.vbar(x=dodge('provinsi',  0.0,  range=p.x_range), top='Sembuh', width=0.2, source=source,
       color="#718dbf", legend_label="Sembuh")

p.vbar(x=dodge('provinsi',  0.25, range=p.x_range), top='Meninggal', width=0.2, source=source,
       color="#e84d60", legend_label="Meninggal")
```

**Setup Style Visualization**

```python
p.x_range.range_padding = 0.01
p.xgrid.grid_line_color = None
p.legend.location = "top_left"
p.legend.orientation = "horizontal"

# Tick labels
p.xaxis.major_label_text_font_size = '6pt'
p.yaxis.major_label_text_font_size = '10pt'
```

**Add Hover Tools**

```python
hover = HoverTool()
hover.tooltips = [
    ("Jumlah Kasus", "@Kasus Pasien"),
    ("Jumlah Sembuh", "@Sembuh Pasien"),
    ("Jumlah Meninggal", "@Meninggal Pasien")]

hover.mode = 'vline'

p.add_tools(hover)
```

**Display Visualization**

```
show(p)
```

![Bokeh Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/bokeh_province.PNG)

### Bokeh Region Visualization using Panel Tabs Widget

**Slicing Dataset into Region Based On Location**
```python
df_sumatera   = df_provinsi.iloc[[10, 11, 14, 16, 20, 22, 25, 31, 32, 33], :]
df_jawa       = df_provinsi.iloc[[0, 1, 2, 3, 5, 17], :]
df_kalnusteng = df_provinsi.iloc[[8, 12, 13, 15, 18, 6, 7, 34], :]
df_sulmalpap  = df_provinsi.iloc[[4, 19, 21, 23, 24, 30, 27, 28, 9, 29], :]
```

**Setup Style Visualization Function**
```python
 def set_style(p):
   # Tick labels
   p.xaxis.major_label_text_font_size = '6pt'
   p.yaxis.major_label_text_font_size = '10pt'
```

**Setup Region Visualization**
```python
 fig1 =figure(title="Data Kasus Positif COVID-19 di Sumatera", x_range=df_sumatera['Provinsi_Asal'].tolist(), plot_width=800, plot_height=300)
 fig1.vbar(x=df_sumatera['Provinsi_Asal'].tolist(), top=df_sumatera['Kasus'].tolist(), width=0.8, color='yellow')
 tab1 = Panel(child=fig1, title="Kasus")

 fig2 =figure(title="Data Pasien Sembuh dari COVID-19 di Sumatera", x_range=df_sumatera['Provinsi_Asal'].tolist(), plot_width=800, plot_height=300)
 fig2.vbar(x=df_sumatera['Provinsi_Asal'].tolist(), top=df_sumatera['Sembuh'].tolist(), width=0.8, color='green')
 tab2 = Panel(child=fig2, title="Sembuh")

 fig3 =figure(title="Data Kematian akibat COVID-19 di Sumatera", x_range=df_sumatera['Provinsi_Asal'].tolist(), plot_width=800, plot_height=300)
 fig3.vbar(x=df_sumatera['Provinsi_Asal'].tolist(), top=df_sumatera['Meninggal'].tolist(), width=0.8, color='red')
 tab3 = Panel(child=fig3, title="Meninggal")
```

**Setup Panel Tabs Widget and Style Function**
```python
 tabs = Tabs(tabs=[ tab1, tab2, tab3])

 set_style(fig1)
 set_style(fig2)
 set_style(fig3)
```

**Display Visualization**
```python
show(tabs)
```

![Bokeh Region Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/bokeh_region.PNG)

## Geopandas Visualization

**Load Indonesia Geospatil Dataset**
```python
import geopandas as gpd

df_geo = gpd.read_file('https://raw.githubusercontent.com/bimaputra1/School_Partitipation_Rates_with_GeoPandas/master/gadm36_IDN_1.json')
```

**Merge Lat and Long Column from Geospatial Dataset into Province Dataset**
```python
df_join = df_geo.merge(df_provinsi, how='inner', left_on='NAME_1', right_on='Provinsi_Asal')
df_join = df_join[['Provinsi_Asal', 'Kasus', 'Sembuh', 'Meninggal', 'Rasio Kematian', 'geometry']]
```

**Convert % Percent into Float Type**
```python
df_join['Rasio'] = df_join['Rasio Kematian'].str.rstrip('%').astype('float')
```

**Setup 'Rasio' as Data Visualization Parameters**
```python
values = 'Rasio'

vmin, vmax = 0, 15
fig, ax = plt.subplots(1, figsize=(30, 10))
ax.axis('off')
```

**Setup Style Visualization**
```python
title = 'Visualisasi {} Kematian (Dalam Persen)'.format(values)
ax.set_title(title, fontdict={'fontsize':'25',
                              'fontweight':'3'})

ax.annotate('Source : kawaldatacovid.org', xy=(0.1,.08),
            xycoords='figure fraction',
            horizontalalignment='left',
            verticalalignment='top',
            fontsize=12,
            color='#555555')

sm = plt.cm.ScalarMappable(cmap='OrRd',
norm=plt.Normalize(vmin=vmin, vmax=vmax))

cbar =  fig.colorbar(sm)
```

**Show Visualization**
```python
df_join.plot(column=values, cmap='OrRd',
             linewidth=0.8,
             ax=ax,
             edgecolor='0.8',
             norm=plt.Normalize(vmin=vmin, vmax=vmax))
```

![Geopandas Region Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/geopandas_province.PNG)

**Show Visualization with Label Names**
```python
df_join['coords'] = df_join['geometry'].apply(lambda x: x.representative_point().coords[:])
df_join['coords'] = [coords[0] for coords in df_join['coords']]
for idx, row in df_join.iterrows():
    plt.annotate(s=row['Provinsi_Asal'],
                 xy=row['coords'],
                 horizontalalignment='center',
                 fontsize=8,
                 color='black')
```

![Geopandas Region Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/geopandas_region_province_with_label.PNG)

## Contributing
 - [**Taufik Fathurahman**](https://github.com/taufikfathurahman)
 - [**Fikhri Masri**](https://github.com/fikhrimasri)

## Authors
 - **Farhan Alfariqi**
