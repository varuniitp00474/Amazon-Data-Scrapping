# Amazon-Data-Scrapping
This project involves web scraping product information from Amazon using Python libraries. The script extracts product titles, prices, ratings, review counts, and availability status, and saves the data to a CSV file. The main tools used are BeautifulSoup for parsing HTML, requests for making HTTP requests, and pandas for data manipulation.

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Installation](#installation)
3. [Code Explanation](#code-explanation)
   - [Imports](#imports)
   - [Helper Functions](#helper-functions)
   - [Main Execution](#main-execution)
4. [Running the Script](#running-the-script)
5. [Output](#output)
6. [Alternatives and Improvements](#alternatives-and-improvements)

## Prerequisites

Before running the script, ensure you have Python installed on your machine. Additionally, you'll need to install the required libraries.

## Installation

Install the necessary Python libraries using pip:
```bash
pip install beautifulsoup4 requests pandas numpy
```

## Code Explanation

### Imports

```python
from bs4 import BeautifulSoup
import requests
import pandas as pd
import numpy as np
```

- **BeautifulSoup**: Used for parsing HTML content.
- **requests**: Used for making HTTP requests to fetch web pages.
- **pandas**: Used for data manipulation and creating dataframes.
- **numpy**: Used for handling missing values in the data.

### Helper Functions

#### `get_title(soup)`

Extracts the product title from the HTML content.

```python
def get_title(soup):
    try:
        title = soup.find("span", attrs={"id":'productTitle'})
        title_value = title.text
        title_string = title_value.strip()
    except AttributeError:
        title_string = ""
    return title_string
```

#### `get_price(soup)`

Extracts the product price. It handles both standard prices and deal prices.

```python
def get_price(soup):
    try:
        price = soup.find("span", attrs={'id':'priceblock_ourprice'}).string.strip()
    except AttributeError:
        try:
            price = soup.find("span", attrs={'id':'priceblock_dealprice'}).string.strip()
        except:
            price = ""
    return price
```

#### `get_rating(soup)`

Extracts the product rating.

```python
def get_rating(soup):
    try:
        rating = soup.find("i", attrs={'class':'a-icon a-icon-star a-star-4-5'}).string.strip()
    except AttributeError:
        try:
            rating = soup.find("span", attrs={'class':'a-icon-alt'}).string.strip()
        except:
            rating = ""    
    return rating
```

#### `get_review_count(soup)`

Extracts the number of user reviews.

```python
def get_review_count(soup):
    try:
        review_count = soup.find("span", attrs={'id':'acrCustomerReviewText'}).string.strip()
    except AttributeError:
        review_count = ""    
    return review_count
```

#### `get_availability(soup)`

Extracts the availability status of the product.

```python
def get_availability(soup):
    try:
        available = soup.find("div", attrs={'id':'availability'})
        available = available.find("span").string.strip()
    except AttributeError:
        available = "Not Available"    
    return available
```

### Main Execution

#### Setting Headers and URL

```python
if __name__ == '__main__':
    HEADERS = ({'User-Agent':'YOUR_USER_AGENT', 'Accept-Language': 'en-US, en;q=0.5'})
    URL = "https://www.amazon.com/s?k=playstation+4&ref=nb_sb_noss_2"
```

- **HEADERS**: Includes the User-Agent and Accept-Language to mimic a browser request.
- **URL**: The Amazon search URL for the product.

#### Making the Request and Parsing the HTML

```python
    webpage = requests.get(URL, headers=HEADERS)
    soup = BeautifulSoup(webpage.content, "html.parser")
```

- **requests.get**: Fetches the webpage content.
- **BeautifulSoup**: Parses the HTML content of the webpage.

#### Extracting Links

```python
    links = soup.find_all("a", attrs={'class':'a-link-normal s-no-outline'})
    links_list = [link.get('href') for link in links]
```

- **find_all**: Finds all anchor tags with the specified class.
- **links_list**: Stores the extracted links.

#### Extracting Product Details

```python
    d = {"title":[], "price":[], "rating":[], "reviews":[],"availability":[]}
    for link in links_list:
        new_webpage = requests.get("https://www.amazon.com" + link, headers=HEADERS)
        new_soup = BeautifulSoup(new_webpage.content, "html.parser")
        d['title'].append(get_title(new_soup))
        d['price'].append(get_price(new_soup))
        d['rating'].append(get_rating(new_soup))
        d['reviews'].append(get_review_count(new_soup))
        d['availability'].append(get_availability(new_soup))
```

- **new_webpage**: Fetches the product detail page.
- **new_soup**: Parses the product detail page.
- **d**: Dictionary to store the extracted product details.

#### Saving Data to CSV

```python
    amazon_df = pd.DataFrame.from_dict(d)
    amazon_df['title'].replace('', np.nan, inplace=True)
    amazon_df = amazon_df.dropna(subset=['title'])
    amazon_df.to_csv("amazon_data.csv", header=True, index=False)
```

- **DataFrame**: Converts the dictionary to a pandas DataFrame.
- **replace**: Replaces empty titles with NaN.
- **dropna**: Drops rows where the title is NaN.
- **to_csv**: Saves the DataFrame to a CSV file.

## Running the Script

1. Ensure you have the required libraries installed.
2. Replace `'YOUR_USER_AGENT'` with your actual user agent string.
3. Run the script:

```bash
python amazon_scraper.py
```

## Output

The output is a CSV file named `amazon_data.csv` containing the extracted product information.

![image](https://github.com/varuniitp00474/Amazon-Data-Scrapping/assets/143409781/57c426bb-ca54-4c7f-9971-ecfa79381c7a)

## Alternatives and Improvements

- **Libraries**: Use `scrapy` for more robust scraping capabilities.
- **Error Handling**: Improve error handling for network issues and parsing errors.
- **Dynamic Content**: Handle JavaScript-loaded content using `Selenium`.
- **Proxies**: Use proxy servers to avoid IP blocking.
- **Data Enrichment**: Extract additional product information like images, descriptions, and categories.
