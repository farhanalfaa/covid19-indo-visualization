> Online README link : https://github.com/farhanalfaa/covid19-visualization/blob/master/README.md
 
 > Youtube URL : https://youtu.be/Fu8-fDwqXco
 
 > Google Colab URL : https://github.com/farhanalfaa/covid19-visualization/blob/master/program/Tugas%20Besar%20Visualisasi%20Data.ipynb
 
#### Author:
- Taufik Fathurahman (1301160790)
- Farhan Alfariqi (1301161770)
- Fikhri Masri (1301164662)

# COVID19 Visualization using Bokeh Library

COVID19 merupakan fenomena besar yang sedang melanda dunia secara global. Tak sedikit korban jiwa berjatuhan akibatnya virus tersebut. Tak terkecuali Indonesia, hingga saat ini (28 April 2020 Pukul 14:35) telah terindikasi positif sekitar 9.096 jiwa. Informasi mengenai  Persebaran Pasien Penderita COVID19 sangatlah diperlukan pada saat ini. Salah satunya untuk mengantisipasi lonjakan penderita secara masif. Oleh karena itu, Program yang akan dibangun ini diharapkan dapat membantu pengguna untuk memahami persebaran COVID19 baik untuk wilayah ASEAN, maupun Tiap-tiap Provinsi di Indonesia. Dengan bantuan **Bokeh Library**, proses visualisasi data penderita COVID19 dapat diinterpretasikan lebih interaktif dan dapat dipahami oleh pengguna dengan baik.

Program ini akan memanfaatkan fitur-fitur yang tersedia pada **Bokeh Library** untuk memaksimalkan visualisasi data yang didapatkan, serta dengan tambahan fitur dari **Geopandas Library** diharapkan dapat memudahkan pengguna untuk melihat persebaran penderita melalui suatu Data Geospasial

## Built With

* [Bokeh 2.0.2](https://docs.bokeh.org/)
* [Geopandas 0.7.0](https://geopandas.org/)

## Getting Started

Berikut ini merupakan rangkaian program yang menampilkan persebaran kasus COVID19 di Indonesia berdasarkan beragam kategori dengan menggunakan Bokeh Library untuk memudahkan proses visualisasi data. Rangkaian program tersebut akan membahas berikut :

**1. [Part 1] Jumlah Pertumbuhan Harian Penderita COVID19 di Indonesia**

**2. [Part 2] Jumlah Persebaran Penderita COVID19 di Tiap Daerah di Indonesia**

**3. [Part 3] Jumlah Persebaran Penderita COVID19 dalam skala ASEAN**

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
import geopandas as gpd

from bokeh.plotting import figure, show, Figure
from bokeh.io import output_notebook
from bokeh.models import ColumnDataSource, HoverTool, CheckboxGroup, CustomJS, Panel, Tabs, Slider, Row, FactorRange
from bokeh.transform import dodge, factor_cmap
from bokeh.layouts import row, column, widgetbox
from bokeh.palettes import Viridis3, Spectral5
from bokeh.models.widgets import CheckboxGroup, RadioGroup

output_notebook()
```

## [Part 1] The Daily Additional Number in Indonesia

#### Author:
- Taufik Fathurahman (1301160790)

Akan dilakukan visualisasi dari pertambahan kasus COVID-19 perhari di Indonesia selama kurun waktu dua bulan, yaitu Maret sampai bulan April sekarang. Program akan memvisualisasikana kasus yang terkonfirmasi, kasus kematian, dan kasus sembuh. Visualisasi dibuat interaktif dengan menggunakan bantuan Library Bokeh.

## Handle Dataset

Langkah pertama adalah dengan memuat dataset yang akan divisualisasikan.

```python
df_kasusharian = pd.read_csv('https://raw.githubusercontent.com/farhanalfaa/covid19-visualization/master/dataset/Indo/Kasus%20Harian%20(1).csv')
```

Selanjutnya adalah melakukan preprocess data untuk menghilangkan missing value dan value data yang masih belum sesuai seperti format waktu.

```python
def convert_date(x):
    return pd.to_datetime(x.split(' ')[0])

df_kasusharian['DT'] = df_kasusharian['DT'].apply(convert_date)
df_kasusharian = df_kasusharian.fillna(0)
df_kasusharian['Month'] = pd.DatetimeIndex(df_kasusharian['DT']).month
df_kasusharian = df_kasusharian.groupby(['Month', 'DT']).max()
df_kasusharian.rename(columns={"Kasus (Kumulatif)" : "KasusKumulatif", 
                               "Kasus Baru" : "KasusBaru",
                               "Sembuh (baru)" : "Sembuh",
                               "Meninggal (baru)" : "Meninggal"
                               }, inplace = True)
```

Untuk data yang sudah dipreprocess, selanjutnya akan masuk kedalam function yang akan mengubah data tersebut agar dapat dengan mudah divisualisasikan dengan Library Bokeh.

```python
def create_dataset(df): 
    '''
    Membuat dataset agar siap untuk digunakan dalam melakukan visualisasi.
    '''  
    df_maret_kasus = df.loc[3]
    df_maret_kasus = df_maret_kasus.sort_values(by='DT')
    source_maret_kasus = ColumnDataSource(df_maret_kasus)

    df_april_kasus = df.loc[4]
    df_april_kasus = df_april_kasus.sort_values(by='DT')
    source_april_kasus = ColumnDataSource(df_april_kasus)

    df_overall_kasus = df.loc[3]
    df_overall_kasus = df_overall_kasus.append(df.loc[4])
    df_overall_kasus = df_overall_kasus.sort_values(by='DT')
    source_overall = ColumnDataSource(df_overall_kasus)

    return source_maret_kasus, source_april_kasus, source_overall
```

## COVID-19 Trend Visualization

Ketika semua data sudah siap, selanjutnya adalah mekaukan visualisasi dengan lirary bokeh. Akan ada beberapa visualisasi yang dibuat agar dapat mempermudah memahami isi dan mendapatkan informasi dari dataset daily additional number in Indonesia.

### A. Overall Dataset Visualization

Akan dilakukan visualisasi data secara keseluruhan mulai dari data bulan Maret sampai dengan April, dimana didalamnya akan memuat jumlah kasus baru, jumlah kasus sembuh, jumlah kasuk meninggal, dan jumlah secara kumulatif dari kasus yang ada.

```python
def create_plot(source, bulan='Overall'):
    '''
    Membuat plot untuk dataset yang akan divisualisasikan
    '''
    plot = figure(x_axis_type="datetime", title="{} Trend".format(bulan), plot_width=1000, plot_height=500)

    l0 = plot.line(x='DT', y='KasusBaru', line_color='red', source=source, legend_label='Kasus Baru')
    c0 = plot.circle(x='DT', y='KasusBaru', fill_color="white", size=8, source=source)   

    l1 = plot.line(x='DT', y='Sembuh', line_color='green', source=source, legend_label='Sembuh')
    c1 = plot.circle(x='DT', y='Sembuh', fill_color="white", size=8, source=source)      

    l2 = plot.line(x='DT', y='Meninggal', line_color='black', source=source, legend_label='Meninggal')
    c2 = plot.circle('DT', y='Meninggal', fill_color="white", size=8, source=source)      

    l3 = plot.line(x='DT', y='KasusKumulatif', line_color='blue', source=source, legend_label='Kumulatif Kasus')
    c3 = plot.circle(x='DT', y='KasusKumulatif', fill_color="white", size=8, source=source)

    plot.legend.location = "top_left"
    plot.add_tools(HoverTool(tooltips=[("# Kasus Baru", "@KasusBaru"), ("# Sembuh_(baru)", "@Sembuh"), 
                                       ("# Meninggal (baru)", "@Meninggal"), ("# Kasus (Kumulatif)", "@KasusKumulatif")]))
    
    return plot, [l0, l1, l2, l3, c0, c1, c2, c3]
```

Code diatas bertujuan untuk membuat plot dengan menggunakan line dan circle dalam melakukannya. Untuk menjadikan plot lebih interaktif akan ditambahkan widget checkbox dengan code dibawah ini.

```python
def visual_overall_tren(source, bulan='Overall'):
    '''
    Memvisualisasikan tren dari covid-19 secara keseluruhan dengan menambahkan opsi widget checkbbox
    '''
    my_plot, my_lst = create_plot(source, bulan)
    l0, l1, l2, l3, c0, c1, c2, c3 = my_lst

    checkbox = CheckboxGroup(labels=["Kasus Baru", "Sembuh", "Meninggal", "Kumulatif Kasus"], 
                             active=[0, 1, 2, 3], width=200)
    checkbox.callback = CustomJS(args=dict(l0=l0, l1=l1, l2=l2, l3=l3, c0=c0, c1=c1, c2=c2, c3=c3, checkbox=checkbox),
                             code="""                                
                                  l0.visible = 0 in checkbox.active;
                                  l1.visible = 1 in checkbox.active;
                                  l2.visible = 2 in checkbox.active;
                                  l3.visible = 3 in checkbox.active;
                                  c0.visible = 0 in checkbox.active;
                                  c1.visible = 1 in checkbox.active;
                                  c2.visible = 2 in checkbox.active;
                                  c3.visible = 3 in checkbox.active;
                                  """)
    return my_plot, checkbox

```

### B. Each Month Dataset Visualization

Agar lebih mudah dilihat, selanjutnya dataset akan divisualisasikan berdasarkan bulannya dengan bantuan widget Tab.

```python
def visual_eachmonth():
    '''
    Memvisualisasikan trand pertambahan dari COVID-19 berdasarkan bulannya
    '''
    source_maret_kasus, source_april_kasus, source_overall = create_dataset(df_kasusharian)

    my_plot1, _ = create_plot(source_maret_kasus, 'Maret')
    tab1 = Panel(child=my_plot1, title="Maret")

    my_plot2, _ = create_plot(source_april_kasus, 'April')
    tab2 = Panel(child=my_plot2, title="April")

    tabs = Tabs(tabs=[tab1, tab2])
    show(tabs)
```

### C. Add Offset into Dataset

Disini akan dicoba bagaimana bila ada tamahan kedalam dataset, yaitu dengan menambahkan offset dengan bantuan widget slider.

```python
def visual_addoffset():
    '''
    Menambahkan offset pada data dengan widget slider
    '''
    y = list(df_kasusharian['KasusKumulatif'])
    x = list(np.arange(len(df_kasusharian['KasusKumulatif'])))
    source = ColumnDataSource(data=dict(x=x, y=y))
    plot = Figure(plot_width=1000, plot_height=500)
    plot.line('x', 'y', source=source, line_width=3, line_alpha=0.6)
    handler = CustomJS(args=dict(source=source), code="""
    var data = source.data;
    var f = cb_obj.value
    var x = data['x']
    var y = data['y']
    for (var i = 0; i < x.length; i++) {
        y[i] = y[i]+f
    }
    source.change.emit();
    """)
    slider = Slider(start=-2000, end=2000, value=0, step=100, title="Offset")
    slider.js_on_change('value', handler)
    layout = column(slider, plot)
    show(layout)
```

 
## [Part 2] COVID19 Visualization in Every Region in Indonesia

Akan dilakukan visualisasi jumlah persebaran Penderita COVID19 pada tiap Provinsi di Indonesia yang melibatkan setidaknya 34 provinsi. Visualisasi nantinya akan ditampilkan dalam bentuk Data Geospasial yang memperlihatkan Wilayah Indonesia beserta Informasi mengenai rasio kematian yang diakibatkan oleh COVID19 dengan bantuan **Geopandas Library**

#### Author :
- Farhan Alfariqi (1301161770)

### Handle Dataset

Langkah selanjutnya yaitu me-load dataset dengan cara berikut:
```python
df_provinsi   = pd.read_csv('https://raw.githubusercontent.com/farhanalfaa/covid19-visualization/master/dataset/Indo/data%20provinsi.csv')
```

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

### Geopandas Visualization

**Load Indonesia Geospatial Dataset**
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

## [Part 3] COVID19 Visualization in the Association of Southeast Asian Nations (ASEAN) Region

#### Author :
- Fikhri Masri (1301164662)

### Explanation

Data Visualisasi adalah teknik menyajikan data secara visual melalui grafik, chart , peta agar tampilan menarik namun tetap informatif. Alasan ada nya data visualisasi adalah tidak lepas dari "kebosanan" dan "monoton" apabila data disajikan dalam bentuk baris dan kolom seperti dalam tabel tabel. Pada Kasus Covid-19 ASEAN Menampilkan Visualisasi:

**1. Visualisasi Kasus Perwaktu Dari Negara ASEAN**

**2. Visualisasi Jumlah Kasus, Sembuh, dan Meninggal**

**3. Visualisasi Rentang Umur yang Terkena COVID19 di Salah Satu Negara ASEAN (Indonesia)**

### Visualisasi Negara ASEAN yang diambil dari tanggal (24-Jan-2020 - 26-Apr-2020)

**Setup Line Glyphs**
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
```

**Setup CheckBox Configuration**

```python
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
```

**Setup vbar Glyphs**

```python
source = ColumnDataSource(df_kumulatif_asean)
countries = source.data['Country/Region'].tolist()
plot = figure(x_range=countries, plot_height=500, plot_width=1000, title="Jumlah Kasus di ASEAN")
plot.vbar(x='Country/Region', top='Cases', source=source, width=0.9)
plot.xgrid.grid_line_color = None
plot.xaxis.axis_label = 'Country'
plot.yaxis.axis_label = 'Jumlah'
plot.y_range.start = 0
```

**Add Hover Tools**

```python
hover = HoverTool()
hover.tooltips = [
    ("Jumlah Kasus", "@Cases Orang")]

hover.mode = 'vline'

plot.add_tools(hover)
tab2 = Panel(child = plot, title = 'Kumulatif Kasus')

tabs = Tabs(tabs=[ tab1, tab2 ])
```

**Display Visualization**

```python
layout = row(checkbox,tabs)
show(layout)
```

![Bokeh Visualization](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/line%20Asean.PNG)
![Bokeh Visualization1](https://github.com/farhanalfaa/covid19-visualization/blob/master/images/Jumlah%20Cases%20Kumulatif%20ASEAN.png)

#### Visualisasi Jumlah Kasus, Sembuh, dan Meninggal 

```python
source = ColumnDataSource(data=data)
country = source.data['country']
```

**Setup vbar Glyphs**

```python
p = figure(x_range=country, y_range=(0, 15000), plot_height=500, plot_width=2000, title="Data Covid-19 ASEAN", toolbar_location="left")

p.vbar(x=dodge('country', -0.25, range=p.x_range), top='Kasus', width=0.2, source=source,
       color="#c9d9d3", legend_label="Kasus")

p.vbar(x=dodge('country',  0.0,  range=p.x_range), top='Sembuh', width=0.2, source=source,
       color="#718dbf", legend_label="Sembuh")

p.vbar(x=dodge('country',  0.25, range=p.x_range), top='Meninggal', width=0.2, source=source,
       color="#e84d60", legend_label="Meninggal")
```

**Setup Style Visualization**

```python
p.x_range.range_padding = 0.01
p.xgrid.grid_line_color = None
p.legend.location = "top_left"
p.legend.orientation = "horizontal"

# Tick labels
p.xaxis.major_label_text_font_size = '20pt'
p.yaxis.major_label_text_font_size = '20pt'
```

**Add Hover Tools**

```python
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
```

**Setup Style Visualization**

```python
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

#### Visualisasi rentang umur yang terkena Covid-19 di salah satu negara ASEAN (Indonesia)
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
 - [**Taufik Fathurahman**](https://github.com/taufikfathurahman)
 - [**Farhan Alfariqi**](https://github.com/farhanalfaa)
 - [**Fikhri Masri**](https://github.com/fikhrimasri)
  
 ## Authors
 
 - **Taufik Fathurahman**
 - **Farhan Alfariqi**
 - **Fikhri Masri**
