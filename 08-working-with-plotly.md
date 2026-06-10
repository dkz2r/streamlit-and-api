---
title: "Working with Plotly"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

-

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

-

::::::::::::::::::::::::::::::::::::::::::::::::

## Making Plots with Plotly

Streamlit has built in support for a variety of plotting libraries, including Plotly, Matplotlib,
and Seaborn. For our application, we will be using Plotly to create interactive plots of our
data. Plotly is based off of D3.js, and allows us to create interactive plots that can be easily
embedded in our Streamlit app.

## First Plot: Temperature over Time

So far we have our metrics, but we don't have any plots yet. And this is supposed to be a workshop
about making visuals after all! Let's start by gathering our data. In the last couple of episodes
we have retrieved the current weather data and the daily weather data, but we can also retrieve
the hourly weather data. On our sketch we had a kind of plot that we imagined, where there were
two lines showing the temperature and apparent temperature over time, and a bar in the background
with the precipitation levels. Let's start with just one line, or "trace" in Plotly terms, showing
the temperature over time.

To begin, we will need to retrieve the hourly weather data from the API. We can do this similarly
to how we retrieved the current weather data and the daily weather data:

```python
parameters_to_request = [
    "temperature_2m",
]
params = {
    "latitude": latitude,
    "longitude": longitude,
    "hourly": ",".join(parameters_to_request),
    "forecast_days": 1,
}

response = requests.get(API_URL, params=params)
hourly_data_json = response.json()

st.write(hourly_data_json)
```

### JSON to DataFrame

A few episodes ago we saw that we could print very nice dataframes to our application. However, the
data that we get back from the API is in JSON format, which python has interpreted as a dictionary.
It looks like we get a key in our response called `hourly`, which contains an entry for each of the
parameters we requested, where each value is a list of values for each hour. Lucky us! This is one
of the formats we can use to easily create a dataframe! Let's replace our `st.write(hourly_data_json)`
with the following code to create a dataframe from our JSON response:

```python
import pandas as pd

... # existing code

hourly_df = pd.DataFrame(hourly_data_json["hourly"])
st.dataframe(hourly_df)
```

### Creating a Plotly Line Chart

Now that we have our data in a dataframe, we can create a line chart using Plotly. We can do this
using the `px.line` function from the Plotly Express module. This function takes in a dataframe and
the names of the columns to use for the x and y axes, and returns a Plotly figure object that we
can display in our Streamlit app:

```python
import plotly.express as px

... # existing code

fig = px.line(hourly_df, x="time", y="temperature_2m", title="Temperature over Time")
st.plotly_chart(fig)
```

![Temperature over Time](./fig/07-making-visuals/simple-line-plot.PNG){alt="Basic line plot showing temperature over time"}

Eh, it works to show the data, but we want to tidy things up a bit. A couple things we notice right
away:

- The x and y axis labels seem to come right from the dataframe column names
- Hovering over the plot gives us a tooltip, but it's not very readable
- A legend might be nice to have for when we add more traces to the plot later

### Customizing the Plot

We won't go too deep into the Plotly API in this workshop, but let's see how we can quickly
make some changes to our plot. First, we can change the axis labels by passing in a `labels`
dictionary to the `px.line` function:

```python
fig = px.line(
    hourly_df,
    x="time",
    y="temperature_2m",
    title="Temperature over Time",
    labels={
        "time": "Time",
        "temperature_2m": "Temperature (°C)",
    },
)
st.plotly_chart(fig)
```

Next, we can customize the tooltip by passing in a `hover_data` dictionary to the `px.line` function:

```python
fig = px.line(
    hourly_df,
    x="time",
    y="temperature_2m",
    title="Temperature over Time",
    labels={
        "time": "Time",
        "temperature_2m": "Temperature (°C)",
    },
    hover_data={
        "time": "|%H:%M",  # Format the time as hours and minutes
        "temperature_2m": ":.1f",  # Format the temperature to one decimal place
    },
)
st.plotly_chart(fig)
```



::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1:


:::::::::::::::::::::::: solution


:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
