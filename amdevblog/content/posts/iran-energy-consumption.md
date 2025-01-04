+++
date = '2025-01-04T17:26:31+03:30'
draft = false
title = 'Exploring Energy Consumption'
categories = ['Visualization']
tags = ['Plotly', 'Visualization']
[cover]
image = '/energy-consumption.png'  # Path to your cover image
relative = false  # Set to true if using a page bundle
hidden = true  # Set to true to hide the cover image everywhere
hiddenInList = false  # Set to true to hide the cover image on list pages
hiddenInSingle = true  # Set to true to hide the cover image on single pages
+++

Energy consumption is a frequently discussed topic in Iran, with media often claiming that Iran's per capita energy usage is significantly higher than in other countries. To better understand the reality of this claim, I decided to dive into the data and create a visualization.

Below is the chart comparing per capita energy consumption (measured in kilowatt-hours per person, kWh) for selected countries. The data is taken from [Our World In Data](https://ourworldindata.org/grapher/per-capita-energy-use). The red dots represent the average energy consumption between 2010 and 2021, while the blue dots show the most recent data from 2023. The grey lines between the dots indicate the change in consumption over time.

{{< figure src="/energy-consumption.png" alt="energy-consumption" align=center >}}

While Iran is an energy-rich country with relatively high usage compared to some of its neighbors, its per capita energy consumption is far from excessive when viewed alongside other oil-rich countries like Saudi Arabia or industrialized nations like the US and Canada. Nonetheless, the growth in Iran's energy consumption is significant.

Below is the code for producing this chart:

```python

import plotly.graph_objects as go
import pandas as pd

df = pd.read_csv("per-capita-energy-use.csv")

countries = ["Turkey", "Iraq", "Pakistan","Saudi Arabia","United Arab Emirates" ,"Germany", "France","United Kingdom",
             "Canada", "Iran","United States"]
df = df.rename(columns={"Primary energy consumption per capita (kWh/person)":"kwhp"})

df_avg = (
    df[df['Year'].between(2010, 2021)]
      .groupby('Entity', as_index=False)
      .agg({'kwhp': 'mean'})
      .rename(columns={'kwhp': 'avg_2010_2021'})
)

df_2023 = df[df['Year'] == 2023][['Entity', 'kwhp']]
df_2023 = df_2023.rename(columns={'kwhp': 'use_2023'})

df_plot = pd.merge(df_avg, df_2023, on='Entity', how='inner')

df_plot = df_plot[df_plot['Entity'].isin(countries)].copy()

df_plot = df_plot.sort_values('avg_2010_2021', ascending=True).reset_index(drop=True)


line_x = []
line_y = []

for i, row in df_plot.iterrows():
    line_x.extend([row['avg_2010_2021'], row['use_2023'], None])
    line_y.extend([row['Entity'], row['Entity'], None])


fig = go.Figure()

fig.add_trace(
    go.Scatter(
        x=line_x,
        y=line_y,
        mode='lines',
        line=dict(color='grey'),
        showlegend=False
    )
)

fig.add_trace(
    go.Scatter(
        x=df_plot['avg_2010_2021'],
        y=df_plot['Entity'],
        mode='markers',
        name='<b>2010–2021</b> Average',
        marker=dict(color='red', size=10)
    )
)

fig.add_trace(
    go.Scatter(
        x=df_plot['use_2023'],
        y=df_plot['Entity'],
        mode='markers',
        name='<b>2023</b>',
        marker=dict(color='blue', size=10)
    )
)

fig.update_layout(
     title=dict(
        text="Energy Consumption per Capita (kWh): <b>2010–2021</b> Average vs. <b>2023</b>",
        yref="container",
        y= 0.98,
        yanchor= "bottom"
     ),
    height=1080,
    width=1920,
    plot_bgcolor='white',
    font_family="Roboto",
    title_font_family="Times New Roman",
    legend=dict(
    orientation="h",
    yanchor="top",
    y=1.1,
    xanchor="left",
    x=0.5
)
)
fig.update_xaxes(
    showgrid=True,
    gridcolor='#9AA6B2',
    gridwidth=0.5,
    zerolinewidth=2, zerolinecolor='black',
    tickfont=dict(size=15),
    side='top',
    tickformat=',',
    automargin=True,
    ticklabelstandoff=10,
    rangemode = "tozero"
)

fig.update_yaxes(
    showgrid=True,
    gridcolor='#9AA6B2',
    gridwidth=0.5,
    tickfont=dict(size=15),
    ticklabelstandoff=20,
    range=[-0.2, len(df_plot) - 0.80]

)

fig.show()
```
