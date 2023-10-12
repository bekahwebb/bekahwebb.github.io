---
layout: post
title:  "The Beauty of Beautiful Soup 4: A Powerful Tool for Web Scraping and Data Cleaning in Python."
author: Rebekah Webb
description: This is a Comprehensive Beautiful Soup Tutorial.
image: /assets/images/imagesoup.png
---
Intro
This is a simple tutorial to show how to use beautiful soup4 in python to parse HTML data. From their site {https://www.crummy.com/software/BeautifulSoup/} we learn that Beautiful Soup is a Python library for pulling data out of HTML and XML files. Keep in mind that there are some sites that may not be ethical to scrape. Respect Website Policies: Before scraping any website, make sure to check the website's 'robots.txt' file to see if web scraping is allowed or prohibited. Some websites might have terms of use that you need to adhere to.  For this tutorial, we did check the robots.txt for our scraped data and wikipedia and the bbc were ok with the data we scraped.

Let's start with an example of web scraping from a wikipedia table on production car speed records
Before we begin this tutorial, open up your favorite python notetbook and begin trying out the code by installing these packages.  We import pandas for our code such as pd.read.  Next we import requests so that we may use the code below for for our page = requests.get(url).  From bs4 we import BeautifulSoup for webscraping.  Our last import will be used for our data cleaning code, re which is the Python module for regular expressions. Regular expressions are used for pattern matching and text manipulation.

 # Step 1: Scraping data

 ``` 
import pandas as pd
import requests
from bs4 import BeautifulSoup
import re

# Step 1: Scraping data
url = 'https://en.wikipedia.org/wiki/Production_car_speed_record'
page = requests.get(url)
soup = BeautifulSoup(page.content, 'html.parser')
table = soup.find('table', class_='wikitable')
df = pd.read_html(str(table))[0]
print(df.head())

output
  Year       Make and model                  Top speed  \
0  1894            Benz Velo        20 km/h (12 mph)[3]   
1  1949         Jaguar XK120  200.5 km/h (124.6 mph)[4]   
2  1955  Mercedes-Benz 300SL  242.5 km/h (150.7 mph)[6]   
3  1959  Aston Martin DB4 GT      245 km/h (152 mph)[7]   
4  1963     Iso Grifo GL 365      259 km/h (161 mph)[8]   

                                              Engine Number built  \
0  1,045 cm3 (63.8 cu in) single-cylinder 1.1 kW ...         1200   
1  3,442 cm3 (210.0 cu in) inline-6 119 kW (162 P...        12000   
2  2,996 cm3 (182.8 cu in) inline-6 158 kW (215 P...         1400   
3  3,670 cm3 (224 cu in) inline-6 225 kW (306 PS;...           75   
4  5,354 cm3 (326.7 cu in) V8 268 kW (365 PS; 360...     over 400   

                                             Comment  
0                               First production car  
1  Some publications cite the XK120's timed top s...  
2  Two-way average speed tested by Automobil Revu...  
3              Tested by Autosport in December 1961.  
4  Tested by Autocar in 1966. A total of 412 Iso ...  
```
we have some cleaning to do to organize our table so that our headings are in a row together and we get rid of extra dots etc.

Here's code to clean our table
# Step 2: Cleaning data
```
df = df.rename(columns={'Number\nbuilt[10]': 'Number built'})
df = df[['Year', 'Make and model', 'Top speed', 'Number built']]
df['Top speed'] = df['Top speed'].apply(lambda x: re.findall('\d+\.?\d*', x)[0])
df['Top speed'] = df['Top speed'].astype(float)
print(df.head())

output
    Year       Make and model  Top speed Number built
0  1894            Benz Velo       20.0         1200
1  1949         Jaguar XK120      200.5        12000
2  1955  Mercedes-Benz 300SL      242.5         1400
3  1959  Aston Martin DB4 GT      245.0           75
4  1963     Iso Grifo GL 365      259.0     over 400
```
Better, much better.  We now have a cleaned table that is much easier to follow.

Now let's scrape data from the Provo Wikipedia page and use the same code as above with this Provo wiki url but specify the table we want. I want to look at table 4 to look at the top Employers in Provo.  We will repeat steps 1 and 2 for this tutorial to now handle a new table that we want to scrape and clean.

# Repeat Step 1: Scraping data from another wikipedia page
```
url = 'https://en.wikipedia.org/wiki/Provo,_Utah'
page = requests.get(url)
soup = BeautifulSoup(page.content, 'html.parser')
tables = soup.find_all('table')
table = tables[4]  # Select the second table (0-based index)
df = pd.read_html(str(table))[0]
print(df)

output
    #                             Employer # of Employees
0   1             Brigham Young University    5,000-6,999
1   2  Utah Valley Regional Medical Center    3,000-3,999
2   3                               Vivint    3,000-3,999
3   4                         Arm Security    1,000-1,999
4   5                        Revere Health    1,000-1,999
5   6                       Chrysalis Utah    1,000-1,999
6   7                            Qualtrics    1,000-1,999
7   8                      RBD Acquisition    1,000-1,999
8   9              Frontier Communications        500-999
9  10                Nu Skin International        500-999
```

This table is already pretty readable, it is no surprise that BYU is the #1 Employer with all of the people that they employ.   To make the table a little cleaner, we will just get rid of the numbered column and rename the # of Employees column.

# Repeat Step 2: Cleaning data

I did a little cleaning to get rid of the # column and renamed the Number of employees column

```
# Rename the 'Employee' column 
df = df.rename(columns={'# of Employees': 'Number of Employees'})
# Select specific columns
df = df[['Employer', 'Number of Employees']]
# Print the cleaned DataFrame
print(df)

output
                             Employer Number of Employees
0             Brigham Young University         5,000-6,999
1  Utah Valley Regional Medical Center         3,000-3,999
2                               Vivint         3,000-3,999
3                         Arm Security         1,000-1,999
4                        Revere Health         1,000-1,999
5                       Chrysalis Utah         1,000-1,999
6                            Qualtrics         1,000-1,999
7                      RBD Acquisition         1,000-1,999
8              Frontier Communications             500-999
9                Nu Skin International             500-999
```
I encourage you to use step 1 to find your own pages to scrape, and find the specific table that you want to use, and then do step 2 to clean your data and make the table look how you want.

Now let's try web scraping news headlines from the BBC.

```
#Here's my code:
import requests
from bs4 import BeautifulSoup
# Step 1: Make an HTTP request to the news website
url = 'https://www.bbc.com/news'
response = requests.get(url)
# Check if the request was successful
if response.status_code == 200:
# Step 2: Parse the HTML content with Beautiful Soup
soup = BeautifulSoup(response.content, 'html.parser')
# Step 3: Find and extract the top headlines
headlines = []
headline_elements = soup.find_all('h3', class_='gs-c-promo-heading__title')
for element in headline_elements:
headline_text = element.text.strip()
headline_link = element.find('a')['href']
headlines.append({'text': headline_text, 'link': headline_link})
# Step 4: Display the scraped headlines
    for i, headline in enumerate(headlines, start=1):
        print(f"{i}. {headline['text']}")
        print(f"   URL: {headline['link']}\n")
else:
    print("Failed to retrieve the web page. Status code:", response.status_code)

output
1. Gaza's only power plant runs out of fuel during Israeli siege
   URL: N/A

2. Inside Israeli border village where Hamas killed families in their homes
   URL: N/A

3. Children screamed in street as we fled 2am Gaza air strike
   URL: N/A

4. My daughter’s final moments as Hamas invaded her home
   URL: N/A

5. Hiding at home, blinded and choked by dust - life in Gaza
   URL: N/A

```

Let's clean up this output a little bit.

```
# Convert the list of dictionaries into a DataFrame
df = pd.DataFrame(headlines)
# Print the first few rows of the DataFrame
print(df.head())
output
                                                text link
0  Gaza's only power plant runs out of fuel durin...  N/A
1  Inside Israeli border village where Hamas kill...  N/A
2  Children screamed in street as we fled 2am Gaz...  N/A
3  My daughter’s final moments as Hamas invaded h...  N/A
4  Hiding at home, blinded and choked by dust - 
l...  N/A

```

Webscraping is a great tool to use to find data that you don't already have collected. 

# Lastly, let's save our scraped data to a csv file. 
Data Storage: In a real project, consider storing the scraped data in a structured format like CSV, JSON, or a database for further analysis.  We'll use our code from our webscraping example we did for step 1 repeated and save the Number of Provo Employees table to a csv file.

```
import pandas as pd
# Rename the 'Employee' column to 'Number of Employees'
df = df.rename(columns={'# of Employees': 'Number of Employees'})
# Select specific columns
df = df[['Employer', 'Number of Employees']]
# Print the cleaned DataFrame
print(df)
# Save the modified DataFrame to a CSV file
df.to_csv('employee_data.csv', index=False)

```
Conclusion: Beautiful soup is a great library to use in python to webscrape.  Web scraping can be fun, and the actual scraping does not require too much effort but the cleaning can be trickier and requires more effort.  I have provided a cheat sheet for you to use to try out some more of your own web scraping here. {https://colab.research.google.com/drive/1RkSNKqSQ0secm5wEArBssNVQh0SQ1yLR#scrollTo=e5t-IL_NjXkt}
Have a beautiful time using beautiful soup for your webscraping needs. Enjoy the soup!

