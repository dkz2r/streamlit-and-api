---
title: "Connecting to APIs and Managing Secrets"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

-

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

-

::::::::::::::::::::::::::::::::::::::::::::::::

## What about APIs that require authentication?

So far, we've looked only at APIs that are open and don't require any sort of authentication.
However many APIs require you to have an account or some kind of API key to access the data. We
want to make sure that we don't accidentally share our API keys in our code, so we need a way to
manage our secrets securely.

In this episode, we'll look at how to use the `python-dotenv` library to manage our secrets and
keep them out of our code.

To start with, let's create a new streamlit file called `coscine-app.py` and add the following:

```python
import streamlit as st

st.title("Coscine API Demo")
st.write("This is a demo of how to use the Coscine API in a Streamlit app.")
```


## The `python-dotenv` library and `.env` files

In our setup we included a library called `python-dotenv`, but we haven't used it yet. This library
allows us to store secrets in a special file called `.env`. We can exclude this file from our
version control, so we don't accidentally share it, and we can use this library to put secrets from
this file temporarily into our environment variables when we run our app. This mimics how secrets
are often managed in production environments, so we can be more confident that our app will work
when we deploy it.

### Getting a Secret

For this workshop, we'll be using the Coscine API and python SDK. To begin, we'll get a token from
the Coscine website and store it in our `.env` file. The secrets in the env file are stored as
key-value pairs, so we can add a line like this to our `.env` file:

```
COSCINE_TOKEN=your_token_here
```

### Loading Secrets with `python-dotenv`

Now that we have our token stored in our `.env` file, we can use the `python-dotenv` library to
load the secrets from this file into our environment variables when we run our app.

```python
import os
from dotenv import load_dotenv
import streamlit as st

st.title("Coscine API Demo")
st.write("This is a demo of how to use the Coscine API in a Streamlit app.")

# Load secrets from .env file
load_dotenv()

if os.getenv("COSCINE_TOKEN") is None:
    st.error("No Coscine token found.")
else:
    st.success("Coscine token found!")
```

## Using the Coscine SDK to populate a widget

Now that we can get our token in our app, we can use the Coscine SDK to make API requests and
retrieve data about our projects.

```python
import os
import coscine
from dotenv import load_dotenv
import streamlit as st

st.title("Coscine API Demo")
st.write("This is a demo of how to use the Coscine API in a Streamlit app.")

# Load secrets from .env file
load_dotenv()

if os.getenv("COSCINE_TOKEN") is None:
    st.error("No Coscine token found.")
# I deleted the else statement - we only care to see a message if there is no token

client = coscine.ApiClient(os.getenv("COSCINE_TOKEN"))

project_list = [project.display_name for project in client.projects()]

selected_project = st.selectbox(
    "Project",
    project_list,
    index=None,
    placeholder="Select a project",
)
```

And once we have the project, we can also use the SDK to get a list of resources for that project
and display them in another widget!

```python
import os
import coscine
from dotenv import load_dotenv
import streamlit as st

st.title("Coscine API Demo")
st.write("This is a demo of how to use the Coscine API in a Streamlit app.")

# Load secrets from .env file
load_dotenv()

if os.getenv("COSCINE_TOKEN") is None:
    st.error("No Coscine token found.")
# I deleted the else statement - we only care to see a message if there is no token

client = coscine.ApiClient(os.getenv("COSCINE_TOKEN"))

project_list = [project.display_name for project in client.projects()]

selected_project = st.selectbox(
    "Project",
    project_list,
    index=None,
    placeholder="Select a project",
)

project = client.project(selected_project)
resource_list = [resource.display_name for resource in project.resources()]
selected_resource = st.selectbox(
    "Resource",
    resource_list,
    index=None,
    placeholder="Select a resource",
)
```

Wait, we get an error! Why?

## Controlling Program Flow with Conditionals

Because when the app first runs, there is no project selected yet, so
when we try to get the project with `client.project(selected_project)`, it throws an error because
`selected_project` is `None`. We can fix this by adding a simple check to make sure that a project
is selected before we try to get the project:

```python
selected_project = st.selectbox(
    "Project",
    project_list,
    index=None,
    placeholder="Select a project",
)

if selected_project is not None:
    project = client.project(selected_project)
    resource_list = [resource.display_name for resource in project.resources()]
    selected_resource = st.selectbox(
        "Resource",
        resource_list,
        index=None,
        placeholder="Select a resource",
    )

    if selected_resource:
        resource = project.resource(selected_resource)
        with st.expander("Resource Metadata"):
            st.write(resource)

        all_resource_files = list(resource.files())
        num_files_in_resource = len(all_resource_files)
        st.write(f"Number of files in resource: {num_files_in_resource}")
```

## User Feedback for Long Running Operations

The next thing we'd like to do is gather the metadata for the selected resource and display it in
our application. We can do this by iterating through all of the files in `resource.files()` and
getting the metadata for each file. However, this requires a separate API request for each file,
which can take some time if there are a lot of files. I would be nice to give the user some
feedback that something is happening while we wait for our app to complete this task. We can do this with the
`st.spinner` and `st.progress` widgets:

```python
if selected_project is not None:
    project = client.project(selected_project)
    resource_list = [resource.display_name for resource in project.resources()]
    selected_resource = st.selectbox(
        "Resource",
        resource_list,
        index=None,
        placeholder="Select a resource",
    )

    if selected_resource:
        resource = project.resource(selected_resource)
        with st.expander("Resource Metadata"):
            st.write(resource)

        all_resource_files = list(resource.files())
        num_files_in_resource = len(all_resource_files)
        st.write(f"Number of files in resource: {num_files_in_resource}")

        overall_file_data = []
        with st.spinner("Summarizing metadata across all files in the resource..."):
            bar = st.progress(0)
            for i, file in enumerate(all_resource_files):
                overall_file_data.append(dict(file.metadata_form()))
                bar.progress((i + 1) / num_files_in_resource)

        st.dataframe(overall_file_data)
```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Add a filter widget

Our app currently displays all of the metadata for a given resource. The default dataframe widget
let's us search, but what if we have some numerical data we want to filter on? Add a slider widget
that allows us to filter the files in the resource by these numerical values.

If your resource doesn't have any numerical metadata, you can add some fake data to the metadata
for each file in the resource by adding random numbers to our metadata like this:

```python
import random

...

overall_file_data = []
with st.spinner("Summarizing metadata across all files in the resource..."):
    bar = st.progress(0)
    for i, file in enumerate(all_resource_files):
        file_metadata = dict(file.metadata_form())
        file_metadata["random_number"] = random.randint(0, 100)
        overall_file_data.append(file_metadata)
        bar.progress((i + 1) / num_files_in_resource)
```

::: hint

You can use the `st.slider` widget to create a slider that allows the user to select a range of
values. Then, you can use this range to filter the dataframe before displaying it.

:::


:::::::::::::::::::::::: solution

```python
numerical_filter = st.slider(
    "Filter by 'random number'",
    min_value=0,
    max_value=100,
    value=(0, 100),
    step=1,
)
st.write(f"Filtering to show only files with 'random number' between {numerical_filter[0]} and {numerical_filter[1]}")

filtered_file_data = [
    file_data
    for file_data in overall_file_data
    if numerical_filter[0] <= file_data["random_number"] <= numerical_filter[1]
]

st.dataframe(filtered_file_data)
```

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 2: Caching Data

You might have noticed in the previous challenge, that every time we move the slider, the app
has to re-run the code to load all of the metadata back in. This is of course not idea, since
if there are many files in the resource, this can take a long time. We can implement a caching
solution to this problem using the `st.cache_data` decorator on a function that loads the metadata.

First, we move the code that loads the metadata into a separate function and add the
`@st.cache_data` decorator to it:

```python
@st.cache_data
def load_file_data(project_name, resource_name):
    project = client.project(project_name)
    resource = project.resource(resource_name)
    file_data = []
    for file in resource.files():
        metadata = dict(file.metadata_form())
        metadata["random_number"] = random.randint(0, 100)
        file_data.append(metadata)
    return file_data
```

How would we update the rest of our code to use this new function?

:::::::::::::::::::::::: solution

In our main app code, we call this function to load the metadata:

```python
overall_file_data = load_file_data(selected_project, selected_resource)
```

That's it! When the slider is moved, the app is still technically re-running, but the metadata
is being loaded from the cache instead of making new API requests, so it should be much faster.

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
