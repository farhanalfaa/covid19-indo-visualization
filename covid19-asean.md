# Visualisasi Covid-19 Pada Wilayah Association of Southeast Asian Nations (ASEAN)
COVID19 merupakan fenomena besar yang sedang melanda dunia secara global. Tak sedikit korban jiwa berjatuhan akibatnya virus tersebut. Tak terkecuali Indonesia, hingga saat ini (26 April 2020) telah terindikasi positif sekitar 9.096 jiwa. Berikut ini merupakan program yang menampilkan persebaran kasus COVID19 di Indonesia dengan menggunakan library Bokeh yang bermanfaat dalam menampilkan visualisasi dengan cara lebih interaktif.

## Getting Started

### Visualisasi
Data Visualisasi adalah teknik menyajikan data secara visual melalui grafik, chart , peta agar tampilan menarik namun tetap informatif. Alasan ada nya data visualisasi adalah tidak lepas dari "kebosanan" dan "monoton" apabila data disajikan dalam bentuk baris dan kolom seperti dalam tabel tabel. Pada Kasus Covid-19 ASEAN Menampilkan Visualisasi:
1. Visualisasi kasus perwaktu dari Negara AsEAN
2. Visualisasi jumlah Kasus, Sembuh, dan Meninggal
3. Visualisasi rentang umur yang terkena Covid-19 di salah satu negara ASEAN (Indonesia)

### Built With

* [Bokeh 2.0.2](https://docs.bokeh.org/)

### Prerequisites

Langkah utama yaitu menginstall library yang diperlukan, seperti berikut:
```python
 pip install numpy
 pip install pandas
 pip install bokeh
```
### Installing

Langkah selanjutnya yaitu mengimport library yang telah diinstal kedalam teks editor (Jupyter Notebook) :
```
from bokeh.plotting import figure, show
from bokeh.io import output_file, output_notebook
from bokeh.models.tools import HoverTool
from bokeh.models import ColumnDataSource
from bokeh.layouts import column
import numpy as np
from bokeh.layouts import row
from bokeh.palettes import Viridis3
from bokeh.plotting import figure
from bokeh.models import FactorRange
from bokeh.palettes import Spectral5
from bokeh.models import CheckboxGroup, CustomJS, Panel, Tabs, Row
from bokeh.transform import dodge
from bokeh.models.widgets import CheckboxGroup, RadioGroup
from bokeh.transform import factor_cmap
from bokeh.layouts import widgetbox
%matplotlib inline
import pandas as pd
import numpy as np
```

### Visualisasi Negara ASEAN yang diambil dari tanggal (24-Jan-2020 - 26-Apr-2020)

```python
p = figure(x_axis_type="datetime", plot_width=1000, plot_height=450, title = 'Kasus ASEAN Time Series')
props = dict(line_width=4, line_alpha=0.7)
indo = p.line(df_indo['Time'], df_indo['Cases'],color=Viridis3[0], legend_label="Indonesia", **props)
singa = p.line(df_singa['Time'], df_singa['Cases'],color=Viridis3[1], legend_label="Singapore", **props)
thai = p.line(df_thai['Time'], df_thai['Cases'],color=Viridis3[2], legend_label="Thailand", **props)
camb = p.line(df_camb['Time'], df_camb['Cases'],color='black', legend_label="Cambodia", **props)
viet = p.line(df_viet['Time'], df_viet['Cases'],color='brown', legend_label="Vietnam", **props)
phil = p.line(df_phil['Time'], df_phil['Cases'],color='darkred', legend_label="Phillipines", **props)
laos = p.line(df_laos['Time'], df_laos['Cases'],color='blue', legend_label="Laos", **props)
malay = p.line(df_malay['Time'], df_malay['Cases'],color='gold', legend_label="Malaysia", **props)
p.legend.location = "top_left"
checkbox = CheckboxGroup(labels=["Indonesia", "Singapore", "Thailand", "Cambodia", "Vietnam", "Phillipines", "Laos", "Malaysia"],
                         active=[0, 1, 2, 3, 4, 5, 6, 7], width=100)
checkbox.callback = CustomJS(args=dict(indo=indo, singa=singa, thai=thai, camb=camb, viet=viet, phil=phil, laos= laos, malay=malay,checkbox=checkbox),
                             code="""
                                  indo.visible = 0 in checkbox.active;
                                  singa.visible = 1 in checkbox.active;
                                  thai.visible = 2 in checkbox.active;
                                  camb.visible = 3 in checkbox.active;
                                  viet.visible = 4 in checkbox.active;
                                  phil.visible = 5 in checkbox.active;
                                  laos.visible = 6 in checkbox.active;
                                  malay.visible = 7 in checkbox.active;
                                  """)
tab1 = Panel(child = p, title = 'Kasus Perhari')

source = ColumnDataSource(df_kumulatif_asean)
countries = source.data['Country/Region'].tolist()
plot = figure(x_range=countries, plot_height=500, plot_width=1000, title="Jumlah Kasus di ASEAN")
plot.vbar(x='Country/Region', top='Cases', source=source, width=0.9)
plot.xgrid.grid_line_color = None
plot.xaxis.axis_label = 'Country'
plot.yaxis.axis_label = 'Jumlah'
plot.y_range.start = 0
hover = HoverTool()
hover.tooltips = [
    ("Jumlah Kasus", "@Cases Orang")]

hover.mode = 'vline'

plot.add_tools(hover)
tab2 = Panel(child = plot, title = 'Kumulatif Kasus')

tabs = Tabs(tabs=[ tab1, tab2 ])


layout = row(checkbox,tabs)
show(layout)
```
**Display Visualization**
![Bokeh Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/line%20Asean.PNG)
![Bokeh Visualization1](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/Jumlah%20Cases%20Kumulatif%20ASEAN.png)

### Visualisasi Juumlah Kasus, Sembuh, dan Meninggal 
```source = ColumnDataSource(data=data)
country = source.data['country']
p = figure(x_range=country, y_range=(0, 15000), plot_height=500, plot_width=2000, title="Data Covid-19 ASEAN", toolbar_location="left")

p.vbar(x=dodge('country', -0.25, range=p.x_range), top='Kasus', width=0.2, source=source,
       color="#c9d9d3", legend_label="Kasus")

p.vbar(x=dodge('country',  0.0,  range=p.x_range), top='Sembuh', width=0.2, source=source,
       color="#718dbf", legend_label="Sembuh")

p.vbar(x=dodge('country',  0.25, range=p.x_range), top='Meninggal', width=0.2, source=source,
       color="#e84d60", legend_label="Meninggal")

p.x_range.range_padding = 0.01
p.xgrid.grid_line_color = None
p.legend.location = "top_left"
p.legend.orientation = "horizontal"

# Tick labels
p.xaxis.major_label_text_font_size = '20pt'
p.yaxis.major_label_text_font_size = '20pt'
hover = HoverTool()
hover.tooltips = [
    ("Jumlah Kasus", "@Kasus Pasien"),
    ("Jumlah Sembuh", "@Sembuh Pasien"),
    ("Jumlah Meninggal", "@Meninggal Pasien")]

hover.mode = 'vline'
tab3 = Panel(child = p, title = 'Jumlah Kasus Covid-19 ASEAN')

country_1 = df_kum_death['Country/Region'].tolist()
kasuss = ['Kasus', 'Sembuh', 'Meninggal']

plot = figure(x_range=country_1, y_range=(0, 15000), plot_height=700, plot_width=1000, title="Data Covid-19 ASEAN", toolbar_location="left", tooltips="$name: @$name")
plot.vbar_stack(kasuss, x='country', width=0.9, color=colors, source=data,
             legend_label=kasuss)

plot.y_range.start = 0
plot.x_range.range_padding = 0.1
plot.xgrid.grid_line_color = None
plot.axis.minor_tick_line_color = None
plot.outline_line_color = None
plot.legend.location = "top_left"
plot.legend.orientation = "horizontal"

tab4 = Panel(child = plot, title = 'Jumlah Kasus Covid-19 ASEAN')
tabs = Tabs(tabs=[ tab3, tab4])

output_file("1.html")
p.add_tools(hover)
layout = row(tabs)
show(layout)
```
**Display Visualization**
![Bokeh Visualization2](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/StackBarplot.png)
![Bokeh Visualization3](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/barplot%20ASEAN.png)

### Visualisasi rentang umur yang terkena Covid-19 di salah satu negara ASEAN (Indonesia)
```
df_kasus_usia.rename(columns={"Jumlah Kasus": "JumlahKasus"}, inplace = True)
output_notebook()
source = ColumnDataSource(df_kasus_usia)
usia = source.data['Usia Group'].tolist()
p = figure(x_range=usia, plot_height=500, plot_width=1000, title="Jumlah Kasus Berdasarkan Usia di Indonesia")
p.vbar(x='Usia Group', top='JumlahKasus', source=source, width=0.9)
p.xgrid.grid_line_color = None
p.xaxis.axis_label = 'Usia'
p.yaxis.axis_label = 'Jumlah'
p.y_range.start = 0
hover = HoverTool()
hover.tooltips = [
    ("Jumlah Kasus", "@JumlahKasus Pasien")]

hover.mode = 'vline'
output_file("4.html")
p.add_tools(hover)

show(p)
```
**Display Visualization**
![Bokeh Visualization4](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/Usiaindo.png)

## Contributing
 - [**Taufik Fathurrahman**](https://github.com/taufikfathurahman)
 - [**Farhan Alfariqi**](https://github.com/farhanalfaa)
 ## Authors
 - **Fikhri Masri**
