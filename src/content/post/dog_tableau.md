---
layout: ../../layouts/post.astro
title: Web Scraping for Visualizations 
description: Learning to Web Scrape for a visualization about dog breeds. 
dateFormatted: December 20th, 2024
---

# The Goal 

Capture the data for my [American Kennel Club Visualization.](https://public.tableau.com/app/profile/miles.cumiskey/viz/Dogs_17376027073670/GeneralDashboard)

I also find dog breeds incredibly interesting. Humans have selectively been choosing traits for their pooches for centuries, and now it's codified into a competition. 

My general goal was to explore some of the overall trends for dogs in the AKC, but also provide a dynamic visualization of each breed to showcase them a little bit. 

To do that, I needed data. 

# The Plan

I actually started learning how to web scrape for my [Board Game Recommendation System](https://github.com/mcumiskey/Meeple-Matchmaker), as I needed to get Board Game Geeks IDs in order to use the API, but with a limited timeframe, when my one day expiriment didn't get the results I needed, I used an existing dataset from Kaggle. 

There are already AKC scrapes on Kaggle (and on Github), but with a little more time for this project, I wanted to challenge myself a little. 

I broke the data collection into three steps, in order of perceived difficulty:
- get the rankings for all dog breeds from 2013-2024
- get the breed information (temperament, trainability, grooming, etc)

From there, I could use python to prepare the data and Tableau for visualizations. 

# Starting with Beautiful Soup

To start, I went through [FreeCodeCamp's web-scraping lessons](https://www.freecodecamp.org/news/web-scraping-python-tutorial-how-to-scrape-data-from-a-website/). This fantastic series helped me get a handle on grabbing information from web pages. Their example has a lovely, easy to navigate structure. 


The problem was applying the lessons learned to a real website.

## Set-Up 
```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
```
**Requests** allows HTTPS queries

**BeautifulSoup** takes the resulting soup of html content and provides a framework to search through and retrieve different elements.

**Pandas** is for storing information once it's retrieved. 

To begin, I started in the same way as the FreeCodeCamp example and grabbed the title of the page, just to make sure everything was set up correctly. 

```python
# Make a request to the 2023 top 200 page 
page = requests.get(
    "https://www.akc.org/expert-advice/news/most-popular-dog-breeds-2023/")
#Set up our soup with the HTML content we requested 
soup = BeautifulSoup(page.content, 'html.parser')

# Extract title of page
page_title = soup.title.text

# print the result
print(page_title)
```

Done! 

## Finding the Elements I needed 

The nice part of starting with the top 200 dogs is that all of the information is nicely formatted in a table. 

| Rank | Breed |
---- | ----
| 1 | French Bulldog |
2 | Labrador Retriever
3 | Golden Retriever
4 | German Shepherd Dog
5 | Poodle
6 | Dachshund
7 | Bulldog
8 | Beagle
9 | Rottweiler
10 | German Shorthaired Pointer

With BeautifulSoup, all I needed was a list comprehension to pull the information.

To break it down:
- For each row in the table
- If the row has content and is not a newline character,
- Strip the text of whitespace and add it to the list 


```python
breed_ranks_list = [
    row.text.strip()
    for row in table.findAll("td")
    if not row.text == ""        
    if not "\n" in row.text
]
```
*yes i write list comprehensions on multiple lines, it's easier to read*

The final list comprises of each dog and their rank in an alternating pattern ["Labrador", 1, "Boston Terrier", 2, ...]

## Dealing with Oddball years

I made a list of all of the years to get more rankings. 
```python
year_links = [
    "https://www.akc.org/expert-advice/news/most-popular-dog-breeds-2023/",
    "https://www.akc.org/expert-advice/dog-breeds/most-popular-dog-breeds-2022/",
    "https://www.akc.org/expert-advice/dog-breeds/most-popular-dog-breeds-of-2021/",
    "https://www.akc.org/expert-advice/dog-breeds/the-most-popular-dog-breeds-of-2020/",
    "https://www.akc.org/most-popular-breeds/2019-full-list/",
    "https://www.akc.org/most-popular-breeds/2018-full-list/",
    "http://akc.org/most-popular-breeds/2017-full-list",
    "https://www.akc.org/most-popular-breeds/2016-full-list/",
    "https://www.akc.org/most-popular-breeds/2015-full-list/",
    "https://www.akc.org/most-popular-breeds/2014-full-list/",
    "https://www.akc.org/most-popular-breeds/2013-full-list/"
]
```

Sadly, the world is not perfectly clean and consistient. Some years had the rank in the left column and the breed in the right, others (looking at you 2022) had no headers. 

```python
if breed_ranks_list[0] == "1":
    df["Rank"] = breed_ranks_list[0::2]  # Every other item starting from index 0 
    df["Breed"] = breed_ranks_list[1::2]   # Every other item starting from index 1 
else: 
    #check if the data is a rank or a breed, then build dataframe accordingly. 
    if not breed_ranks_list[3].isdigit():
        df["Rank"] = breed_ranks_list[2::2]  # Every other item starting from index 0 
        df["Breed"] = breed_ranks_list[3::2]   # Every other item starting from index 1 
    else: 
        df["Breed"] = breed_ranks_list[2::2]   # Every other item starting from index 1 
        df["Rank"] = breed_ranks_list[3::2]  # Every other item starting from index 0 

df.set_index("Rank", inplace=True)   
return df
```
Luckily, as the lists always start at the top breed (1) and alternate between rank and dog, a few if statements were all I needed to capture the data and standardize it. 

## Dealing with Oddball names
```python
def standardize_breed_name(breed):
    #check if it is a string just in case
    if isinstance(breed, str):
        # remove plural "s" at the end for breeds like "Pointers", "Retrievers", "Spaniels", etc.
        breed = re.sub(r'\s?(s)$', '', breed)  
        breed = re.sub(r's\s$', '', breed) #really enforce no s
        
        # handle the case where a breed has the format "Dog (Specific Type)"
        match = re.match(r'([a-zA-Z\s]+)\s?\(([^)]+)\)', breed)
        if match:
            # reformat to "Specific Type Dog"
            return f"{match.group(2)} {match.group(1)}"
        
        # replace non-breaking space with regular space
        breed = breed.replace('\xa0', ' ')
        
        #aggressively getting rid of the s does hurt the malligator, fix her
        if breed == 'Belgian Malinoi':
            breed = 'Belgian Malinois'
    
    return breed
```

We absolutely do not live in a perfect world, so I also had to put together a function to standardize the breed names. 

## Step One: Done!
Once I had grabbed the ranking for each year, I moved on to getting the link for each breed. Nothing Particularly interesting, but the code is available [on my Github]() for those who are interested. 

# The Big Challenge: Dog Information 

Woo! I had a link to each of the breeds that the AKC registers. 

Now I had to get all the information about each dog. 

This was... messy. 

The American Kennel Club is a modern website: it does not have a static HTML page unique for each dog breed. It has a template that it populates with JSON data, with a few unique quirks. 

At first, I considered using Selenium, which is more equipted to navigate and interact with  website data. However, it seemed like overkill as I was ABLE to get the JSON with a little Soup Magic. 

## Soup Magic 
Each dog breed has their information stored in a div. However, when I grabbed the  div with BeautifulSoup, a little but of HTML stuck around. 
```python
import html

html_data = soup.find('div', {'data-js-component': 'breedPage'})

#includes '<div data-js-component="breedPage" data-js-props="' at the start and '>  </div>' at the end, which is not json.
raw_div = html.unescape(str(html_data))

# just remove html stuff! 
clean_div = raw_div[50:-9]
```
After testing the 'brute force' clean up with 10 random dogs, I was confident in applying it to the full list. 

It worked! 

## Encoding and Decoding JSON

People who work with JSON every day, avert your eyes. 

(then look again to show me the proper way to do this)

```python
import json

#save to a temporary json file
with open('data.json', 'w', encoding='utf-8') as f:
        f.write(clean_div)
    
# Opening JSON file
f = open('data.json')
    
# # returns JSON object as 
# # a dictionary
data = json.load(f)
# Closing file
f.close()
```

Despite the fact that the data was JSON formatted, I STRUGGLED to read it as JSON. So... we just saved each dog to a temporary data.json file, read it as a dictionary, and paid it hush money to go away. 

```python
if len(data['settings']['breed_data']) > 1:
        current_breed = data['settings']['current_breed']
        row = { 'Breed': data['settings']['breed_data']['basics'][current_breed]['breed_name'], 
                'Group': data['settings']['current_breed_group']['name'], 
                'Origin':  data['settings']['breed_data']['basics'][current_breed]['origin'],
                'Life Expectancy': data['settings']['breed_data']['basics'][current_breed]['life_expectancy'], 
                'Related Breeds': data['settings']['breed_data']['basics'][current_breed]['related_breeds'], 
                'Temperament': data['settings']['breed_data']['traits'][current_breed]['temperament'], 
                'Adaptability Level': data['settings']['breed_data']['traits'][current_breed]['traits']['adaptability_level']['score'], 
                'Affectionate With Family': data['settings']['breed_data']['traits'][current_breed]['traits']['affectionate_with_family']['score'],
                'Barking Level': data['settings']['breed_data']['traits'][current_breed]['traits']['barking_level']['score'],
                'Coat Grooming Frequency': data['settings']['breed_data']['traits'][current_breed]['traits']['coat_grooming_frequency']['score'], 
                'Good with Young Children': data['settings']['breed_data']['traits'][current_breed]['traits']['good_with_young_children']['score'],
                'Coat Length': data['settings']['breed_data']['traits'][current_breed]['traits']['coat_length']['score'],
                'Coat Type': data['settings']['breed_data']['traits'][current_breed]['traits']['coat_type']['selected'],
                'Drooling Level': data['settings']['breed_data']['traits'][current_breed]['traits']['drooling_level']['score'],
                'Energy Level': data['settings']['breed_data']['traits'][current_breed]['traits']['energy_level']['score'],
                'Good With Other Dogs': data['settings']['breed_data']['traits'][current_breed]['traits']['good_with_other_dogs']['score'],
                'Mental Stimulation Needs': data['settings']['breed_data']['traits'][current_breed]['traits']['mental_stimulation_needs']['score'],
                'Openness to Strangers': data['settings']['breed_data']['traits'][current_breed]['traits']['openness_to_strangers']['score'],
                'Playfulness Level': data['settings']['breed_data']['traits'][current_breed]['traits']['playfulness_level']['score'],
                'Shedding Level': data['settings']['breed_data']['traits'][current_breed]['traits']['shedding_level']['score'],
                'Trainability Level': data['settings']['breed_data']['traits'][current_breed]['traits']['trainability_level']['score'],
        }

        if data['settings']['breed_data']['standards'][current_breed]['height_display']: 
                row['Min Height'] = data['settings']['breed_data']['standards'][current_breed]['height_min_f'] or data['settings']['breed_data']['standards'][current_breed]['height_display']
                row['Max Height'] = data['settings']['breed_data']['standards'][current_breed]['height_max_m'] or data['settings']['breed_data']['standards'][current_breed]['height_display']
                row['Weight Min'] = data['settings']['breed_data']['standards'][current_breed]['weight_min_f'] or data['settings']['breed_data']['standards'][current_breed]['weight_display']
                row['Weight Max'] = data['settings']['breed_data']['standards'][current_breed]['weight_max_m'] or data['settings']['breed_data']['standards'][current_breed]['weight_display']
        print(row)
```

From there I read and stored the basic information in a DataFrame and exported it to be on my merry way. 

## Cleaning 

Since the world still isn't perfect, I did a little bit of data cleaning in python before I went to Tableau. 

A few values had been read oddly due to using fractions instead of decimal places, some html was hanging out in the descriptions, and a few values were missing. If you are curious, you can check out the full clean on my [github.](https://github.com/mcumiskey/AKC-data-collection/blob/main/AKC_Data.ipynb)*

* warning, this is a normal working file and not beautiful! 

# The "Too-Much" Impulse

I've made custom icons. For many different shapes of dogs. Even while typing this I feel the urge to make more. 
