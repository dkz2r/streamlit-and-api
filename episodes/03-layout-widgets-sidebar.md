---
title: "Layouts, Widgets, and the Sidebar"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

- How can I create ways for the user to interact with my app?
- What are some ways I can customize the layout of my app?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Implement different widgets in a Streamlit app
- Use the Streamlit sidebar to organize an app
- Create a collapsible section with `st.expander`

::::::::::::::::::::::::::::::::::::::::::::::::

## Input Widgets

We can add a variety of interactive widgets to our app. Try adding the following lines to the
bottom of your `streamlit_app.py` file:

```python
my_value = st.slider("Select a value", 0, 100, 50)
st.write(f"You selected: {my_value}")
```

Switching back to the browser window, you should now see a slider widget at the bottom of the page
that looks something like this:

![](fig/03-layout-widgets-sidebar/slider-widget.png){alt="Streamlit Slider Widget"}

Even better, if you move the slider around, you'll see that the value you selected is printed out
below the slider! This is one of the places Streamlit really shines - we don't have to fiddle with
CSS and Javascript to create widgets for our applications.

Let's add a few more widgets to our app to see what they look like. Try adding the following lines
to your `streamlit_app.py` file:

```python
my_checkbox = st.checkbox("Check me!")
st.write(f"Checkbox value: {my_checkbox}")

my_text_input = st.text_input("Enter some text")
st.write(f"You entered: {my_text_input}")

my_number_input = st.number_input("Enter a number", min_value=0, max_value=100, value=50)
st.write(f"You entered: {my_number_input}")

my_radio = st.radio("Choose an option", ["Option 1", "Option 2", "Option 3"])
st.write(f"You selected: {my_radio}")

my_selectbox = st.selectbox("Choose an option", ["Option A", "Option B", "Option C"])
st.write(f"You selected: {my_selectbox}")
```

::: callout

Note that for most of these widgets, the as soon as you interact with them, the app automatically
updates the value in the `st.write` statement, with the exception of the `st.text_input` widget.
This is because of the way that `st.text_input` handles user input - this value is only updated
when the user presses "Enter" or clicks outside of the text input box.

:::

## Action Widgets

We aren't limited to just input widgets - we can also add elements like buttons to our application
that will trigger some kind of action when the user interacts with them. Try adding the following
lines to your `streamlit_app.py` file:

```python
my_button = st.button("Click me!")
st.write(f"Button clicked: {my_button}")
```

Ok, but you might have noticed that this isn't super useful - the value of `my_button` is just
`True` as soon as the button is clicked, but clicking the button again doesn't change the value
back to `False` and nothing else on the screen changes.

There's two ways we can make this more interesting:

1. We can use an `if` statement to trigger some kind of action when the button is clicked.
2. We can use the `on_click` parameter of the `st.button` function to trigger a callback function
when the button is clicked.

### Using an `if` statement

Let's first start by setting up a simple conditional statement based on the button. When we click
the button, we want to check the value if `my_button`. If it is currently `True`, we want to
display "Button clicked: False", and if it is currently `False`, we want to display "Button
clicked: True". Maybe we can do this with a little `if` statement like this:

```python
my_button = st.button("Click me!")
if my_button:
    my_button = False
else:
    my_button = True
st.write(f"Button clicked: {my_button}")
```

Hmm. This doesn't work the way we expect. In fact, there's no difference to the user between this
and the previous version of the code.

### Session State

The reason this doesn't work has to do with the way Streamlit handles the application "state".
Streamlit runs your entire script from top to bottom every time the user interacts with the app.
This means that when we click the button, the entire script is rerun, and the value of `my_button`
is reset to `False` at the beginning of the script, so our `if` statement doesn't do anything. We
can work around this limitation by using a feature of streamlit called "session state". This allows
us to store values across interactions with the app.

Try adding the following lines to your `streamlit_app.py` file:

```python
if "my_counter" not in st.session_state:
    st.session_state.my_counter = 0
st.session_state.my_counter += 1
st.write(f"Counter value: {st.session_state.my_counter}")
```

Now start interacting with the widgets we've already added to the application. You should see that
every time you change a value, the counter value increases by 1. This is because when we first
start the app, `my_counter` is not in `st.session_state`, so we initialize it to 0. Then every time
the app is rerun (which, again, is triggered by any interaction with the app), we add 1 to
`my_counter`, and the updated value is displayed on the screen.


### Our Button and Session State

Ok, so now that we know about session state, let's see about getting our button to work as we
intended. We will need to initialize a value in `st.session_state` to keep track of whether the
button has been clicked or not, and then we can use an `if` statement to toggle this value when
the button is clicked:

```python
st.button("Click me!")
# Initialize session state for the button
if "my_button" not in st.session_state:
    st.session_state.my_button = False

# Toggle the button state based on the stored value
if st.session_state.my_button:
    st.session_state.my_button = False
else:
    st.session_state.my_button = True

# Display the current state of the button
st.write(f"Button clicked: {st.session_state.my_button}")
```

So every time we click the button now, the value of `my_button` in `st.session_state` is toggled!
But wait, do you see any unintended side effects of this code? Try interacting with the other
widgets we added to the app. What happens to the button state?

### Callback Functions

Right! Because the entire script is rerun every time we interact with any widget, the button state
is toggled each and every time we iteract with any widget, which is not what we want. We only want
the button state to toggle when we click the button in particular. To achieve this, we can use
what's called a "callback function".

::: callout

Everything in python is an object, and that includes functions! This means that we can pass
functions around as arguments to other functions, which can then call them when they need.

:::

We'll define a small function using our session state conditional logic to toggle the button state:

```python
def toggle_button():
    if st.session_state.my_button:
        st.session_state.my_button = False
    else:
        st.session_state.my_button = True
```

Then we can pass this function to the `on_click` parameter of the `st.button` function:

```python
if "my_button" not in st.session_state:
    st.session_state.my_button = False
st.button("Click me!", on_click=toggle_button)
st.write(f"Button clicked: {st.session_state.my_button}")
```

::: important

Note that we don't include parentheses after `toggle_button` when we pass it to `on_click`, because
we are passing the function itself, not calling the function!

:::

Try clicking the button, then clicking on the other widgets. You should see that the button state
only toggles when you click the button, and not when you interact with the other widgets!

## Layout Widgets

We can add interactive elements to our app, but we also want to be able to control the layout of
our application. Streamlit provides a few different ways to modify our layout from the standard
top-to-bottom layout.

### Columns

Let's start by creating two columns in our app with the `st.columns` function:

```python
left_column, right_column = st.columns(2)
with left_column:
    st.header("This is the left column")
    st.write("We can put any content we want in this column.")
with right_column:
    st.header("This is the right column")
    st.write("We can put any content we want in this column as well.")
```

You should see something that looks like this in your app:

![](fig/03-layout-widgets-sidebar/streamlit-columns.png){alt="Streamlit Widgets and Layout"}

Note that the two column layout doesn't apply to the entire app - we can stil add widgets and
content above and below the columns, and those will be in the standard single column layout.

### Sidebar

We can also add a sidebar to our app with the `st.sidebar` function. The sidebar is a great place
to put widgets that control the overall behavior of the app, or to organize the app into different
sections. Adding a sidebar is similar to the way we created columns - we use a `with` statement to
specify which content should go into the sidebar:

```python
with st.sidebar:
    st.header("This is the sidebar")
    sidebar_checkbox = st.checkbox("This is a checkbox in the sidebar")
    st.write("The checkbox above is ", sidebar_checkbox)

st.write("The checkbox in the sidebar is ", sidebar_checkbox)
```

We can put the sidebar anywhere in our code we like - often it makes most sense to put it near the
top of the script. Note that the second `st.write` statement that is outside of the
`with st.sidebar` block can still access the value of `sidebar_checkbox` - the sidebar is just a
different section of the app, but it shares the same session state and variables as the rest of the
app.


::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1: Create a collapsible section with `st.expander`

We've seen how to modify the layout of our app with columns and a sidebar and seen the way to create
these sections using `with` statements. Another layout object that we can include in our app is the
`st.expander` object. This creates a section of the app that is collapsed by default, but can be
expanded by the user to reveal more content. Try creating a section in your app with `st.expander`
that looks like the following:

![](fig/03-layout-widgets-sidebar/streamlit-expander.png){alt="Streamlit Expander Widget"}


:::::::::::::::::::::::: solution

```python
with st.expander("This is my Expander Object"):
    st.write("Roses are red,")
    st.write("Violets are blue,")
    st.write("This is an expander,")
    st.write("that can be used to hide content until the user wants to see it.")
```

:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
