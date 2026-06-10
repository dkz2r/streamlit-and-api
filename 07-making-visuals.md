---
title: "Making Visuals"
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
with the precipitation levels. Let's


::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1:


:::::::::::::::::::::::: solution


:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
