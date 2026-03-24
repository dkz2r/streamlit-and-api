---
title: "Using Markdown"
teaching: 10 # teaching time in minutes
exercises: 2 # exercise time in minutes
---

:::::::::::::::::::::::::::::::::::::: questions

-

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

-

::::::::::::::::::::::::::::::::::::::::::::::::

## Building Additional Apps

We've made a demo app, a simple weather app, and a Coscine app. There's lots of possibilities for
other apps we can build! This section contains some ideas and starting code for some other apps
for you to play around with.

### Open APIs

There's plenty of open APIs out there you can request data from on a variety of topics. Here's a
list to get you started:

- https://mixedanalytics.com/blog/list-actually-free-open-no-auth-needed-apis/

## App Ideas

### Use the Deck Of Cards API to simulate a simple card game.

The url https://deckofcardsapi.com/ has endpoints to let you shuffle a deck of cards, draw cards,
and even create a "pile" of cards that you can draw from later. It also returns information about
the cards in the deck, such as their suit and value and an image you can display. Some code to get
started:

```python
DECK_OF_CARDS_API_URL = "https://deckofcardsapi.com/api/"

if st.button("Shuffle a new deck of cards"):
    response = requests.get(f"{DECK_OF_CARDS_API_URL}deck/new/shuffle/?deck_count=1")
    data = response.json()
    st.session_state.deck_id = data["deck_id"]

if "deck_id" in st.session_state:
    if st.button("Draw a card"):
        response = requests.get(f"{DECK_OF_CARDS_API_URL}deck/{st.session_state.deck_id}/draw/?count=1")
        data = response.json()
        card = data["cards"][0]
        # Display the card image and information about the card
```

::: callout

We haven't used `st.session_state` yet, but it's a way to store information across interactions in
a Streamlit app. In this case, we can use it to store the `deck_id` that we get back from the API
when we shuffle a new deck of cards, so that we can use that same `deck_id` when we draw cards from
the deck.

:::

### Use the Dictionary API to create a simple dictionary app.

```python

DICTIONARY_API_URL = "https://api.dictionaryapi.dev/api/v2/entries/en/"

word = st.text_input("Enter a word to look up in the dictionary", "hello")

response = requests.get(f"{DICTIONARY_API_URL}{word}")
data = response.json()

st.title(f"Results for '{word}'")

phonetics = data[0].get("phonetics", [])

if phonetics:
    # Display the phoenetic transcriptions, and use `st.audio` to play the audio if available

meanings = data[0].get("meanings", [])

if meanings:
    # Display the definitions for the word, grouped by part of speech

```

### Use the Met Museum API to create an art search app.

The Metropolitan Museum of Art has an open API that allows you to search their collection and get
information about the objects in their collection. Here's some starter code to create a simple
form for searching the collection. See if you can take the output of this form and use it to set
the paramters for an API request to the Met Museum API to get search results based on the user's
input.

```python
API_URL = "https://collectionapi.metmuseum.org/public/collection/v1/search"

with st.form("search_form"):
    search_query = st.text_input(
        "Search Query", placeholder="e.g. sunflowers, portrait, Greek vase"
    )

    with st.expander("Advanced Search", expanded=False):
        col1, col2 = st.columns(2)

        with col1:
            filter_title = st.checkbox("Search in Title")
            filter_tags = st.checkbox("Search in Tags")
            filter_artist = st.checkbox("Search by Artist / Culture")

        with col2:
            medium = st.text_input(
                "Medium", placeholder="e.g. Paintings, Sculpture (comma-separated)"
            )
            geo_location = st.text_input(
                "Geographic Location", placeholder="e.g. France, Europe (comma-separated)"
            )

        date_col1, date_col2 = st.columns(2)
        with date_col1:
            date_begin = st.number_input(
                "Start Year", min_value=-10000, max_value=2100, value=None, placeholder="e.g. 1500"
            )
        with date_col2:
            date_end = st.number_input(
                "End Year", min_value=-10000, max_value=2100, value=None, placeholder="e.g. 1800"
            )

    submitted = st.form_submit_button("Search", use_container_width=True)

st.write(f"Search Query: {search_query}")
st.write(f"Filter by Title: {filter_title}")
st.write(f"Filter by Tags: {filter_tags}")
st.write(f"Filter by Artist/Culture: {filter_artist}")
st.write(f"Medium: {medium}")
st.write(f"Geographic Location: {geo_location}")
st.write(f"Date Range: {date_begin} - {date_end}")

```

::::::::::::::::::::::::::::::::::::: challenge

## Challenge 1:


:::::::::::::::::::::::: solution


:::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::


::::::::::::::::::::::::::::::::::::: keypoints

-

::::::::::::::::::::::::::::::::::::::::::::::::
