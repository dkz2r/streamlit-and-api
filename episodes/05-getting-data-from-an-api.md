---
title: "Getting Data From an API"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

- What is an API?
- How can we retrieve data from an API?
- How do we responsibly use APIs in our applications?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Retrieve data from a public API using the `requests` library.
- Understand the structure of API responses and how to parse them.
- Connect the widgets in our app with the data retrieved from the API to create dynamic content.

::::::::::::::::::::::::::::::::::::::::::::::::

## Getting our Data

We've talked in a very abstract way about "getting data from the API", but what does this actually
mean in practice? In this section we'll talk about what APIs are, how to make requests to an API,
and how to parse the responses we get back from the API.

## What is an API?

"API" stands for "Application Programming Interface", and is a set of rules and protocols that
are designed to allow software applications to communicate with each other. APIs represent the
internet-facing side of a service, allowing developers to access data and functionality from a
service in a standardized way.

You use APIs every day without even realizing it. When you check the status of the bus service on
your phone, the actual application on your phone really only consists of all the images and code to
organize the data on your screen. When you ask for the times for a particular stop, the app sends
a request to an API, which then retrieves the data from a database and returns it to your phone,
where the app fills in the blanks and shows you the times.

### GET and POST requests

Generally, there are two main types of requests that we can make to an API: GET and POST. A GET
request is requesting data from the API, while a POST request is sending something to the API. For
the purposes of our app, we will only be making GET requests.

### Parameters

When we make a request to an API, we can often include additional parameters in our request, which
can help specify exactly what data we want to retrieve. You can see these often in a url at the
end, after a question mark. We can also have multiple paramaters, which are separated by an
ampersand (&).

For our make believe bus API, we might have a url like this for the main website:
`https://my-bus-info-example.com/`. If this were a real site, we might get a webpage with a search
box where we can enter in our bus stop and the time we are interested in.

But we might also be aware of an API, which we can access at `https://my-bus-info-example.com/api`.
This API might allow us to make GET requests with parameters for the bus stop and time, so we could
make a request like `https://my-bus-info-example.com/api?stop="My Street 123, My City"&time=10:00`.

## The Open Meteo API

For our application, we're going to be using the Open Meteo API, which provides weather data. You
can find the documentation for the API here: https://open-meteo.com/en/docs#api_documentation.

We can see that there is an endpoint for the API called "/v1/forecast/" that has a number of
parameters that we can use to specify the exact data we want to retrieve. There are two required
parameters: `latitude` and `longitude`, which specify the location for which we want to retrieve
the weather data. There are also a number of optional parameters we can use to specify what data
we receive. Let's craft a request by hand:

- Latitude: 50.77664
- Longitude: 6.08342
- Forcast Days: 1

Our crafted URL looks like this:
`https://api.open-meteo.com/v1/forecast?latitude=50.77664&longitude=6.08342`

If we put this in a browser and hit enter, we will see a JSON response similar to this:

```
{
  "latitude": 50.782,
  "longitude": 6.0899997,
  "generationtime_ms": 0.00154972076416016,
  "utc_offset_seconds": 0,
  "timezone": "GMT",
  "timezone_abbreviation": "GMT",
  "elevation": 178
}
```

But where is our weather data? Refering back to the documentation we can see that there is a
parameter called `daily` that accepts a comma seperated list of variables that we can use to
retrieve specific weather data. Let's add `daily=temperature_2m_max` to our URL, so it looks like
this:

`https://api.open-meteo.com/v1/forecast?latitude=50.77664&longitude=6.08342&daily=temperature_2m_max`

Your result should look something like this:

```
{
  "latitude": 50.782,
  "longitude": 6.0899997,
  "generationtime_ms": 0.0591278076171875,
  "utc_offset_seconds": 0,
  "timezone": "GMT",
  "timezone_abbreviation": "GMT",
  "elevation": 178,
  "daily_units": {
    "time": "iso8601",
    "temperature_2m_max": "°C"
  },
  "daily": {
    "time": [
      "2026-05-29",
      "2026-05-30",
      "2026-05-31",
      "2026-06-01",
      "2026-06-02",
      "2026-06-03",
      "2026-06-04"
    ],
    "temperature_2m_max": [
        31.9,
        26.5,
        24.5,
        20.6,
        18.5,
        16.5,
        15.2
    ]
  }
}
```

Of course, the exact values will be different depending on the date, but that's why APIs are so
neat! We can make this exact request every day and get an up to date 7 day forcast for the location
we specified.

## Making API requests in Python

To make API requests in Python, we can use the `requests` library, which provides a simple and
intuitive way to make and sent HTTP requests.

::: prereq

As requsts is not part of the python standard library, you will need to have it installed in your
environment. This should have happened back in episode 1, but if you missed that step, you can add
it here with the command `uv add requests`.

:::

Let's step outside of Streamlit for a minute and go back to a basic python script. Create a new
file called `api_test.py` and add the following code:

```python
import requests

response = requests.get(
    "https://api.open-meteo.com/v1/forecast?latitude=50.77664&longitude=6.08342&daily=temperature_2m_max"
)
print("status:", response.status_code)
print("response:", response.json())
```

Run this script with `uv run api_test.py` and you should see something like this:

```
$ uv run api_test.py
200
{'latitude': 50.782, 'longitude': 6.0899997, 'generationtime_ms': 0.06186962127685547, 'utc_offset_seconds': 0, 'timezone': 'GMT', 'timezone_abbreviation': 'GMT', 'elevation': 178.0, 'daily_units': {'time': 'iso8601', 'temperature_2m_max': '▒C'}, 'daily': {'time': ['2026-05-29', '2026-05-30', '2026-05-31', '2026-06-01', '2026-06-02', '2026-06-03', '2026-06-04'], 'temperature_2m_max': [31.9, 26.5, 24.5, 20.6, 18.5, 16.5, 15.2]}}
```

Neat! Ok, next, let's look at crafting our request. We just pasted a big string in here, but that's
not very flexible. What if we want to change the location or the parameters? We could use string
concatenation or f-strings to build our url, but this can get messy and error prone. Better to let
someone else worry about the details of how to format the url correctly, and luckily for us, the
`requests` library can do just that:

```python
import requests

params = {
    "latitude": 50.77664,
    "longitude": 6.08342,
    "daily": "temperature_2m_max",
}
response = requests.get("https://api.open-meteo.com/v1/forecast", params=params)
print("status:", response.status_code)
print("response:", response.json())
```

You should see the same result as before, but now we have a nice dictionary of values that will
allow us to easily change, add, or remove parameters without having to worry about the exact URL.

## Parsing API responses

The response from the API is in JSON format, which is a common format for API responses. The JSON
format is one of the native formats for python, and we can generally interpret this as a dictionary
object. In our case, the response is a dictionary with a number of key-value pairs. The weather
data that we are interested in is nested inside the `daily` key, which we can extract like this:

```python
import requests

params = {
    "latitude": 50.77664,
    "longitude": 6.08342,
    "daily": "temperature_2m_max",
}
response = requests.get("https://api.open-meteo.com/v1/forecast", params=params)
print("status:", response.status_code)
print("response:", response.json())

data = response.json()
daily_data = data["daily"]
print(daily_data)
```

::: callout

### Responsible API Use

When using APIs, it's important to be mindful of how we are using them. Many APIs have rate limits,
which means that there is a limit to how many requests we can make in a certain time period. If we
exceed these limits we might be temporarily or permanently blocked from using the API. Always check
the documentation to see if there are restrictions on how you can use the API, and try to be
respectful of these limits when building your applications.

:::

::: callout

### Caching API responses

A simple way to be more respectful of API rate limits is to cache the responses we get back from
the API. This means that, before we submit a request, we first check our local machine to see if
we have made this exact same request recently, and if so, we can just use the stored data instead.

This can also speed up the application, since reading data from the local machine is usually much
faster (computationally speaking) than waiting for a round trip data request to the API.

The `requests` library has a built in way to do this with the `requests_cache` library, which you
can install with `uv add requests_cache`. You can find the documentation for this library here:
https://requests-cache.readthedocs.io/en/stable/.

:::


::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Use the Open Meteo API to Geocode a City Name

The Open Meteo API has an endpoint for geocoding, which allows us to retrieve the latitude and
longitude for a given city name. This would make it much easier for users to get the weather data
for a location, since they wouldn't have to look up the latitude and longitude themselves.

You can find the documentation for the geocoding endpoint here: https://open-meteo.com/en/docs/geocoding-api.

1. Start by adding a text input to your app where users can enter a city name.
2. When the user hits "return" on the text input, make a request to the geocoding endpoint.
3. Parse the response to extract the latitude and longitude for the city that the user entered.
4. Populate the latitude and longitude number inputs in your app with the values you extracted
   from the geocoding API response.

::: hint

Add the base URL to the top of your file as a constant to make it easier to work with:

```python
GEOCODING_API_URL = "https://geocoding-api.open-meteo.com/v1/search"
```

:::


::: hint

The geocoding endpoint returns a list of results, since there can be multiple cities with the same
name. For simplicity, you can use the "count" parameter to limit the results to 1, so you only get
one result back.

:::



:::::::::::::::::::::::: solution

```python
GEOCODING_API_URL = "https://geocoding-api.open-meteo.com/v1/search"

... # existing code

city_name = st.text_input(
    "Enter a location (e.g., city name or coordinates)", value="Aachen, Germany"
)

params = {
    "name": city_name,
    "count": 1,  # Limit to 1 result for simplicity
}
response = requests.get(GEOCODING_API_URL, params=params)
geocoding_data = response.json()

latitude = geocoding_data["results"][0]["latitude"]
longitude = geocoding_data["results"][0]["longitude"]

lat_lon_columns = st.columns(2)
with lat_lon_columns[0]:
    latitude = st.number_input("Latitude", min_value=-90.0, max_value=90.0, value=latitude)
with lat_lon_columns[1]:
    longitude = st.number_input("Longitude", min_value=-180.0, max_value=180.0, value=longitude)
```

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Error Handling

At the moment, if we make a request to the goecoding API and it doesn't return any results, we just
get an ugly red error message in our app. See if you can add some error handling to the app to
catch this error and prevent the later code from trying to run against missing or incorrect
latitude and longitude values.

Check the streamlit documentation to see if there are are any nice ways to display an error, and
how to stop the execution of the app if we encounter an error.

::: hint

We still get a response back from the API even if there are no results, so we can just check the
length of the "results" list in the response, and if it's empty with
`len(geocoding_data["results"]) == 0`.

:::

::: hint

Look for documentation on `st.error` to display an error message, and `st.stop` to stop the
execution of the app.

:::

:::::::::::::::::::::::: solution

```python
... # existing code

response = requests.get(GEOCODING_API_URL, params=params)
geocoding_data = response.json()

# Check that we got a valid response with at least one result
if len(geocoding_data["results"]) == 0:
    st.error("Location not found. Please enter a valid location.")
    st.stop()

```

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
