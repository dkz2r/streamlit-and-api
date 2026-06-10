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

And lets' add some text above each point, and decrease the size of the markers:

```python
fig = px.line(
    hourly_df,
    x="time",
    y="temperature_2m",
    text="temperature_2m",
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
fig.update_traces(textposition="top center", marker={"size": 5}, textfont={"size": 10})
```

I'm fairly happy with this plot:

![Customized line plot showing temperature over time](./fig/07-making-visuals/updated-line-plot.PNG){alt="Customized line plot showing temperature over time"}

## Adding More Traces and Using Long Format Data

This plot is a little boring though. Our sketch idea way back was to have two lines, one for the
temperature and one for the apparent temperature - let's add the apparent temperature to our plot!
To do this, we will need to request the `apparent_temperature` parameter from the API:

```python
parameters_to_request = ["temperature_2m", "apparent_temperature"]
params = {
    "latitude": latitude,
    "longitude": longitude,
    "hourly": ",".join(parameters_to_request),
    "forecast_days": 1,
}
```

... Huh. So our dataframe updates with the new column, but our plot doesn't update. Why?

We could just add another trace to our plot with another call to `px.line`, but let's be a little
more efficient. Plotly really prefers data that is in the "long" format, where each row is a single
observation and there are columns for the x and y values, and a column that indicates which trace
the observation belongs to. This might be a little frustrating, but fortunately, pandas has a built
in function to convert our data from the "wide" format that we have:

```python
hourly_df = pd.DataFrame(hourly_data_json["hourly"])

# Melt the dataframe to convert it from wide to long format
hourly_df_long = hourly_df.melt(id_vars=["time"], var_name="parameter", value_name="value")

st.dataframe(hourly_df_long)
```

And then we can use the new long data in our plot and see that we get two lines, one for each parameter:

```python
hourly_data_json = response.json()

hourly_df = pd.DataFrame(hourly_data_json["hourly"])

# Melt the dataframe to convert it from wide to long format
hourly_df_long = hourly_df.melt(id_vars=["time"], var_name="parameter", value_name="value")

fig = px.line(
    hourly_df_long, # Use the new long format dataframe
    x="time",
    y="value",  # Change this to "value"
    color="parameter", # Set the color to change by parameter so we get a different line for each
    text="value",
    title="Temperature over Time",
    labels={
        "time": "Time",
        "value": "Temperature (°C)",
        "parameter": "Parameter", # Add a label for the parameter column so we get a nice legend
    },
    hover_data={
        "time": "|%H:%M",  # Format the time as hours and minutes
        "value": ":.1f",  # Format the temperature to one decimal place
    },
)

fig.update_traces(textposition="top center", marker={"size": 5}, textfont={"size": 10})
```

## Color Selection

Think back to the last episode where we talked about color selection for our plots. This plot
contains two traces, one for the temperature and one for the apparent temperature. In my opinion,
it would be nice to have the temperature trace in a bright color, and the apparent temperature in a
more muted color, since the temperature is the main focus of our plot. We can easily change the
colors of our traces by passing in a `color_discrete_map` dictionary to the `px.line` function:

```python
fig = px.line(
    hourly_df_long,
    x="time",
    y="value",
    color="parameter",
    text="value",
    title="Temperature over Time",
    labels={
        "time": "Time",
        "value": "Temperature (°C)",
        "parameter": "Parameter",
    },
    hover_data={
        "time": "|%H:%M",  # Format the time as hours and minutes
        "value": ":.1f",  # Format the temperature to one decimal place
    },
    color_discrete_map={
        "temperature_2m": "blue",
        "apparent_temperature": "lightgrey",
    },
)
fig.update_traces(textposition="top center", marker={"size": 5}, textfont={"size": 10})
```

At this point, the text labels are looking a little messy. Remember to "avoid chartjunk"! I think
the plot is pretty clear without the text labels, so let's remove those:

```python
fig = px.line(
    hourly_df_long,
    x="time",
    y="value",
    color="parameter",
    # text="value",
    title="Temperature over Time",
    labels={
        "time": "Time",
        "value": "Temperature (°C)",
        "parameter": "Parameter",
    },
    hover_data={
        "time": "|%H:%M",  # Format the time as hours and minutes
        "value": ":.1f",  # Format the temperature to one decimal place
    },
    color_discrete_map={
        "temperature_2m": "blue",
        "apparent_temperature": "lightgrey",
    },
)
fig.update_traces(textposition="top center", marker={"size": 5}, textfont={"size": 10})
```

## What is the question we are answering?

We should always keep in mind the question we are trying to answer. In this case, it is something
like "What does the temperature and apparent temperature look like over the course of the day?".
But our data shows the temperatures for the current day, midnight to midnight. There's an additional
aspect of this plot that is useful, which is "what time is it currently?" Let's add a vertical line
to our plot to show the current time:

```python
from datetime import datetime

... # existing code

current_time = datetime.now().strftime("%Y-%m-%dT%H:%M")
fig.add_vline(x=current_time, line_dash="dash", line_color="lightpink")
```

Without context, this is just a red dashed line, so let's add an annotation to explain what it is:

```python
from datetime import datetime

... # existing code

current_time = datetime.now().strftime("%Y-%m-%dT%H:%M")
fig.add_vline(x=current_time, line_dash="dash", line_color="lightpink")
fig.add_annotation(
    x=current_time,
    y=hourly_df_long["value"].max(),  # Position the annotation at the top of the plot
    text="Now",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=-5,  # Shift the text slightly to the left of the line
)

```

Another relevant element - when is sunrise and sunset? We can add shaded areas to the plot to
indicate when it is day and when it is night:

```python
# Add a shaded regions to indicate nighttime hours
sunrise_time = datetime.strptime(daily_sunrise, "%Y-%m-%dT%H:%M")
fig.add_vrect(
    x0=hourly_df["time"].min(),
    x1=sunrise_time,
    fillcolor="lightsteelblue",
    opacity=0.3,
    layer="below",
    line_width=0,
)
fig.add_annotation(
    x=sunrise_time,
    y=hourly_df_long["value"].min(),  # Position the annotation at the bottom of the plot
    text=f"Sunrise ({sunrise_time.strftime('%H:%M')})",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=-5,  # Shift the text slightly to the left of the line
)

# Identical to above, but for sunset time and shifting the annotation to the right of the line instead of the left

sunset_time = datetime.strptime(daily_sunset, "%Y-%m-%dT%H:%M")
fig.add_vrect(
    x0=sunset_time,
    x1=hourly_df["time"].max(),
    fillcolor="lightsteelblue",
    opacity=0.3,
    layer="below",
    line_width=0,
)
fig.add_annotation(
    x=sunset_time,
    y=hourly_df_long["value"].min(),  # Position the annotation at the bottom of the plot
    text=f"Sunset ({sunset_time.strftime('%H:%M')})",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=5,  # Shift the text slightly to the right of the line
)
```

Ok! I think that's looking pretty good so far:

![Line plot with current time and sunrise/sunset annotations](./fig/08-working-with-plotly/line-plot-with-shaded-areas.PNG){alt="Line plot with current time and sunrise/sunset annotations"}

## Mixing plot types

We also had the idea to add bars for precipitation levels in the background of our plot. Let's start
by adding "precipitation" to our list of parameters to request from the API:

```python
parameters_to_request = ["temperature_2m", "apparent_temperature", "precipitation"]
```

Immediately we see that our plot has updated with a new line for precipitation, but that's not what
we really wanted. First, let's exclude the non-temperature parameters from our line plot:

```python
# Limit the temperature data to the two parameters we are interested in
temperature_data = hourly_df_long[
    hourly_df_long["parameter"].isin(["temperature_2m", "apparent_temperature"])
]

fig = px.line(
    temperature_data,
    x="time",
    y="value",
    color="parameter",
    ... # existing code
```

... and then select our precipitation data and create a seperate dataframe for it:

```python
precipitation_data = hourly_df_long[hourly_df_long["parameter"] == "precipitation"]
```

Next we want to create a new trace for our precipitation data, but we want it to be a bar trace
instead of a line trace:

```python
fig.add_trace(
    go.Bar(
        x=precipitation_data["time"],
        y=precipitation_data["value"],
        name="Precipitation",
        yaxis="y2",  # Assign to secondary y-axis
        marker_color="lightblue",
        hovertemplate="Time: %{x}<br>Precipitation: %{y:.1f} mm<extra></extra>",
    )
)
```

And then we need to add a secondary y-axis to our plot for the precipitation data:

```python
fig.update_layout(
    yaxis={"title": "Temperature (°C) / Precipitation (mm)", "side": "left"},
    yaxis2={
        "title": "Precipitation (mm)",
        "overlaying": "y",
        "side": "right",
        "showgrid": False,
        "range": [
            0,
            max(5, precipitation_data["value"].max() * 1.2),
        ],  # Minimum max of 5mm or 20% above actual max
    },
    legend_title_text="Parameter",
)
```

::: callout

plotly.express vs plotly.graph_objects

Plotly has two main interfaces for creating plots: the "express" interface, which is a high level
interface that allows you to create plots with a single function call, and the "graph_objects"
interface, which is a lower level interface that gives you more control over the details of your
plot. In this example, we are using the express interface to create our line plot, and then using
the graph_objects interface to add a bar trace for our precipitation data and to customize the
layout of our plot.

:::

## Deep Breath

Phew! That's a lot of code. Here's the full code for our plot so far:

```python
parameters_to_request = ["temperature_2m", "apparent_temperature", "precipitation"]
params = {
    "latitude": latitude,
    "longitude": longitude,
    "hourly": ",".join(parameters_to_request),
    "forecast_days": 1,
}

response = requests.get(API_URL, params=params)
hourly_data_json = response.json()

hourly_df = pd.DataFrame(hourly_data_json["hourly"])

# Melt the dataframe to convert it from wide to long format
hourly_df_long = hourly_df.melt(id_vars=["time"], var_name="parameter", value_name="value")

# Limit the temperature data to the two parameters we are interested in
temperature_data = hourly_df_long[
    hourly_df_long["parameter"].isin(["temperature_2m", "apparent_temperature"])
]

# Rename the columns for better readability in the plot
temperature_data["parameter"] = temperature_data["parameter"].replace(
    {"temperature_2m": "Temperature (°C)", "apparent_temperature": "Apparent Temperature (°C)"}
)

fig = px.line(
    temperature_data,
    x="time",
    y="value",
    color="parameter",
    # text="value",
    title="Temperature and Precipitation for the next 24 hours",
    labels={
        "time": "Time",
        "value": "Temperature (°C)",
        "parameter": "Parameter",
    },
    hover_data={
        "time": "|%H:%M",  # Format the time as hours and minutes
        "value": ":.1f",  # Format the temperature to one decimal place
    },
    color_discrete_map={
        "Temperature (°C)": "blue",
        "Apparent Temperature (°C)": "lightgrey",
    },
)

# Add precipitation as a bar trace
precipitation_data = hourly_df_long[hourly_df_long["parameter"] == "precipitation"]
fig.add_trace(
    go.Bar(
        x=precipitation_data["time"],
        y=precipitation_data["value"],
        name="Precipitation",
        yaxis="y2",  # Assign to secondary y-axis
        marker_color="lightblue",
        hovertemplate="Time: %{x}<br>Precipitation: %{y:.1f} mm<extra></extra>",
    )
)


current_time = datetime.now().strftime("%Y-%m-%dT%H:%M")
fig.add_vline(x=current_time, line_dash="dash", line_color="lightpink")
fig.add_annotation(
    x=current_time,
    y=temperature_data["value"].max(),
    text="Now",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=-5,  # Shift the text slightly to the left of the line
)

# Add a shaded regions to indicate nighttime hours
sunrise_time = datetime.strptime(daily_sunrise, "%Y-%m-%dT%H:%M")
fig.add_vrect(
    x0=hourly_df["time"].min(),
    x1=sunrise_time,
    fillcolor="lightsteelblue",
    opacity=0.3,
    layer="below",
    line_width=0,
)
fig.add_annotation(
    x=sunrise_time,
    y=hourly_df_long["value"].min(),  # Position the annotation at the bottom of the plot
    text=f"Sunrise ({sunrise_time.strftime('%H:%M')})",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=-5,  # Shift the text slightly to the left of the line
)

sunset_time = datetime.strptime(daily_sunset, "%Y-%m-%dT%H:%M")
fig.add_vrect(
    x0=sunset_time,
    x1=hourly_df["time"].max(),
    fillcolor="lightsteelblue",
    opacity=0.3,
    layer="below",
    line_width=0,
)
fig.add_annotation(
    x=sunset_time,
    y=hourly_df_long["value"].min(),  # Position the annotation at the bottom of the plot
    text=f"Sunset ({sunset_time.strftime('%H:%M')})",
    showarrow=False,  # Don't show an arrow pointing to the line
    textangle=90,  # Rotate the text to be vertical
    xshift=5,  # Shift the text slightly to the left of the line
)

# Update layout to accommodate both y-axes
fig.update_layout(
    yaxis={
        "title": "Temperature (°C)",
        "side": "left",
    },
    yaxis2={
        "title": "Precipitation (mm)",
        "overlaying": "y",
        "side": "right",
        "showgrid": False,
        "range": [
            0,
            max(5, precipitation_data["value"].max() * 1.2),
        ],  # Minimum max of 5mm or 20% above actual max
    },
    legend_title_text="Parameter",
)

st.plotly_chart(fig)
```

Let's take a moment to recap

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Adjust the range of the primary y-axis

In the `fig.update_layout` function, we set the range of the secondary y-axis to be either
5mm or 20% above the actual maximum precipitation value, whichever is greater. This is to ensure
that the precipitation bars are visible even when the maximum precipitation value is very low.

Try to do the same thing for the primary y-axis, which is currently set to adjust automatically
based on the data.


:::::::::::::::::::::::: solution

```python
fig.update_layout(
    yaxis={
        "title": "Temperature (°C)",
        "side": "left",
        "range": [
            min(
                temperature_data["value"].min() * 1.2, -10
            ),  # Minimum of 20% below actual min or -10°C
            max(
                temperature_data["value"].max() * 1.2, 30
            ),  # Maximum of 20% above actual max or 30°C
        ],
    },
    yaxis2={
        "title": "Precipitation (mm)",
        "overlaying": "y",
        "side": "right",
        "showgrid": False,
        "range": [
            0,
            max(5, precipitation_data["value"].max() * 1.2),
        ],  # Minimum max of 5mm or 20% above actual max
    },
    legend_title_text="Parameter",
)
```

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
