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


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
