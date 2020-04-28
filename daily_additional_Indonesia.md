> Online README link : https://github.com/farhanalfaa/covid19-visualization/blob/master/daily_additional_Indonesia.md

##### Author:
- Taufik Fathurahman (1301160790)
- Farhan Alfariqi (1301161770)
- Fikhri Masri (1301164662)


# The Daily Additional Number in Indonesia

Akan dilakukan visalisasi dari pertambahan kasus COVID-19 perhari di Indonesia selama kurun waktu dua bulan, yaitu Maret sampai bulan April sekarang. Program akan memvisualisasikana kasus yang terkonfirmasi, kasus kematian, dan kasus sembuh. Visualisasi dibuat interaktif dengan menggunakan bantuan Library Bokeh.

## Getting Started

Berikut ini merupakan instruksi dasar yang diperlukan untuk menjalankan program The Daily Additional Number in Indonesia dengan menggunakan Library Bokeh.

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
from bokeh.plotting import figure, show
from bokeh.io import output_notebook
from bokeh.models import ColumnDataSource, HoverTool, CheckboxGroup, CustomJS, Panel, Tabs, Slider
from bokeh.transform import dodge
from bokeh.layouts import row
from bokeh.plotting import Figure
from bokeh.layouts import column
output_notebook()
```

### Handle Dataset

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

### 3. COVID-19 Trend Visualization

Ketika semua data sudah siap, selanjutnya adalah mekaukan visualisasi dengan lirary bokeh. Akan ada beberapa visualisasi yang dibuat agar dapat mempermudah memahami isi dan mendapatkan informasi dari dataset daily additional number in Indonesia.

#### a. Overall Dataset Visualization

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

#### B. Each Month Dataset Visualization

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

#### C. Add Offset into Dataset

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
