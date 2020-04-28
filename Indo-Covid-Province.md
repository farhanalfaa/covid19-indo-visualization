# Visualisasi Data COVID19 pada Tiap Provinsi di Indonesia

COVID19 merupakan fenomena besar yang sedang melanda dunia secara global. Tak sedikit korban jiwa berjatuhan akibatnya virus tersebut. Tak terkecuali Indonesia, hingga saat ini (28 April 2020 Pukul 14:35) telah terindikasi positif sekitar 9.096 jiwa. Berikut ini merupakan program yang menampilkan persebaran kasus COVID19 di Indonesia dengan menggunakan library Bokeh yang bermanfaat dalam menampilkan visualisasi dengan cara lebih interaktif.


## Getting Started

Berikut ini merupakan instruksi dasar yang diperlukan untuk menjalankan program Visualisasi COVID19 di Tiap Daerah Indonesia dengan menggunakan Bokeh Library

### Prerequisites

Langkah utama yaitu menginstall library yang diperlukan, seperti berikut:

```python
 !pip install numpy
 !pip install pandas
 !pip install geopandas
```

### Installing

Langkah selanjutnya yaitu mengimport library yang telah diinstal kedalam teks editor (Jupyter Notebook) :

```python
 import numpy as np
 import pandas as pd
 import geopandas as gpd
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

### Coding Style Tests



## Geopandas Visualization



## Contributing

## Authors
 - **Farhan Alfariqi**
