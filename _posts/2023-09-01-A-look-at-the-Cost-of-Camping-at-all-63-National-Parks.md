---
layout: post
title:  "A look at the Cost of Camping at all 63 National Parks."
author: Rebekah Webb
description: This is an Exploratory Data Analysis.
image: /assets/images/Camping1.png
---
# Introduction:

My family and I like to go camping, typically in Utah, at the National Parks down south.  We have camped in various parts of the country and found the camping fees to be pretty reasonable.  I thought it might be interesting to look at the data of the National Park service, NPS and find the most expensive campgrounds of the different National Parks around the country.  I want to explore the data of the NPS and see if there is any correlation between cost of campsite with first come first serve campsites vs. reserved campsites.  I also want to explore the location of the campsite and see if there is correlation between cost and latitude and longitude with zip code of the campsite to see if the location affects the higher campsite costs.

Before I scraped the data, I needed to check that it's ethical to do so.  The National Park Service uses an API to interface with their data.  API stand for Application Programming Interface.  The word application in this context is referring to any software with a distinct function.  Interface can be thought of as a two way contract between you and the Organization that you want to collect data from. You send out requests using their specified documentation code and they will respond with a success or denial for your request.  The ethics in this case of whethor you can gather this data is determined by a yes response.  I also should be mindful to not send too many requests to the NPS and use the data responsibly. I determined I am using this data in an educational, ethical way. I then proceeded to register with the NPS for an apikey which they emailed me to start a sort of contract with the NPS.


# Scraping the NPS Data
  I then loaded in the libraries including a pd.set_option to display all the columns in case there were any embedded columns that I couldn't examine.  Then I put a url request to NPS for the camping data.

```python 
file = open('nps_apikey.txt', 'r') 
api_key = file.read().strip()
import pandas as pd
pd.set_option('display.max_columns', None)
import requests

# Step 1: Scraping data
# Read your API key from a text file
with open('nps_apikey.txt', 'r') as file:
    api_key = file.read().strip()

# Construct the API URL
base_url = 'https://developer.nps.gov/api/v1/campgrounds'
# Define the state codes as a comma-delimited string
#state_codes = 'chis, seki, yose' 'parkCode': state_codes
params = {'api_key': api_key, 'limit': 638 }#state_codes = 'CA,OR,WA,UT' 'stateCode': state_codes

# Make the API request
response = requests.get(base_url, params=params)

# Check the response status code
if response.status_code == 200:
    # Request was successful
    data = response.json()
    data = pd.json_normalize(data,'data')
else:
    # Request failed
    print(f"Failed to retrieve data. Status code: {response.status_code}")
```
My request was successful and my output looked like this:
```python
output
 
id	url	name	parkCode	description	latitude	longitude	latLong	audioDescription	isPassportStampLocation	passportStampLocationDescription	passportStampImages	geometryPoiId	reservationInfo	reservationUrl	regulationsurl	regulationsOverview	fees	directionsOverview	directionsUrl	operatingHours	addresses	images	weatherOverview	numberOfSitesReservable	numberOfSitesFirstComeFirstServe	multimedia	relevanceScore	lastIndexedDate	amenities.trashRecyclingCollection	amenities.toilets	amenities.internetConnectivity	amenities.showers	amenities.cellPhoneReception	amenities.laundry	amenities.amphitheater	amenities.dumpStation	amenities.campStore	amenities.staffOrVolunteerHostOnsite	amenities.potableWater	amenities.iceAvailableForSale	amenities.firewoodForSale	amenities.foodStorageLockers	contacts.phoneNumbers	contacts.emailAddresses	campsites.totalSites	campsites.group	campsites.horse	campsites.tentOnly	campsites.electricalHookups	campsites.rvOnly	campsites.walkBoatTo	campsites.other	accessibility.wheelchairAccess	accessibility.internetInfo	accessibility.cellPhoneInfo	accessibility.fireStovePolicy	accessibility.rvAllowed	accessibility.rvInfo	accessibility.rvMaxLength	accessibility.additionalInfo	accessibility.trailerMaxLength	accessibility.adaInfo	accessibility.trailerAllowed	accessibility.accessRoads	accessibility.classifications
0	EA81BC45-C361-437F-89B8-5C89FB0D0F86	https://www.nps.gov/amis/planyourvisit/277-nor...	277 North Campground	amis	277 North Campground is generally open year-ro...	29.512373695509215	-100.90816633365614	{lat:29.512373695509215, lng:-100.90816633365614}	277 North Campground is generally open year-ro...	0		[]	582AB459-28A8-453A-B22E-BAD481E92098	Standard overnight camping is first-come, firs...	https://www.recreation.gov/camping/campgrounds...	https://cms.nps.gov/amis/learn/management/comp...	Site capacity is not to exceed eight persons a...	[{'cost': '6.00', 'description': 'Sites are av...	Directions from Amistad National Recreation Ar...		[{'exceptions': [], 'description': '277 North ...	[]	[{'credit': 'NPS Photo', 'crops': [], 'title':...	The climate at Amistad is semi-arid in moistur...	1	17	[]	1.0		Yes - year round	[Vault Toilets - year round]	No	[None]	Yes - year round	No	No	No	No	No	[No water]	No	No	No	[]	[{'description': '', 'emailAddress': 'AMIS_Int...	18	1	0	0	0	0	0	0	Limited. However, some sites do have a cement ...			Ground fires are not permitted. Each campsite ...	1	RV and Trailers are permitted	0		0	The main road leading to the campground is pav...	1	[Paved Roads - All vehicles OK]	[Limited Development Campground] 
``````
# Organizing data columns

When examining the first row of code output, it is apparent, there are definitly a lot of columns to go through to get to the heart of what my data analysis will examine.  There is a lot of text and url websites that I don't need so I organized my columns by printing the columns of my data and commenting out the ones that I don't want and making a copy of of my new data columns.


```python
data.columns

output
   data = data[['id', #'url', 
      'name', 'parkCode', 
      #'description', 
      'latitude', 'longitude',
       #'latLong', #'audioDescription', 'isPassportStampLocation',
       #'passportStampLocationDescription', 'passportStampImages',
       #'geometryPoiId', 'reservationInfo', 'reservationUrl', 'regulationsurl',
       #'regulationsOverview', 
       'fees', #'directionsOverview', 'directionsUrl',
       #'operatingHours', 
       'addresses', #'images', 
       #'weatherOverview',
       'numberOfSitesReservable', 'numberOfSitesFirstComeFirstServe',
       #'multimedia', 'relevanceScore', 'lastIndexedDate',
       #'amenities.trashRecyclingCollection', 
       #'amenities.toilets',
       #'amenities.internetConnectivity', 'amenities.showers',
       #'amenities.cellPhoneReception', 'amenities.laundry',
       #'amenities.amphitheater', 
       #'amenities.dumpStation',
       #'amenities.campStore', 
       #'amenities.staffOrVolunteerHostOnsite',
       #'amenities.potableWater', 'amenities.iceAvailableForSale',
       #'amenities.firewoodForSale', 'amenities.foodStorageLockers',
       #'contacts.phoneNumbers', #'contacts.emailAddresses',
       #'campsites.totalSites', 'campsites.group', 'campsites.horse',
       #'campsites.tentOnly', 'campsites.electricalHookups', 'campsites.rvOnly',
       #'campsites.walkBoatTo', 'campsites.other',
       #'accessibility.wheelchairAccess', 'accessibility.internetInfo',
       #'accessibility.cellPhoneInfo', 
       #'accessibility.fireStovePolicy',
       #'accessibility.rvAllowed', 'accessibility.rvInfo',
       #'accessibility.rvMaxLength', 'accessibility.additionalInfo',
       #'accessibility.trailerMaxLength', 'accessibility.adaInfo',
       #'accessibility.trailerAllowed', 'accessibility.accessRoads',
       'accessibility.classifications']].copy()
```

# Cleaning the data and renaming columns
Next I cleaned the data by pulling two variables out of two of the columns: fees and addresses and made individual columns of cost, description, zip, and city.  I wanted to examine the description column closer to see if the output would be useful.  It was really messy in putting random text in different rows and it didn't give me to much information about why the camp charged the cost that it did so I dropped description from the dataset.  I renamed the columns by making a dictionary list, and my output showed the data output I wanted.
```python
# Split 'fees' column into 'fees' and 'description'
data[['cost', 'description']] = pd.DataFrame(data['fees'].apply(lambda x: [x[0]['cost'], x[0]['description']] if x else [None, None]).tolist(), index=data.index)
# Split 'addresses' column into 'zip' and 'city'
data[['zip', 'city']] = pd.DataFrame(data['addresses'].apply(lambda x: [x[0]['postalCode'], x[0]['city']] if x else [None, None]).tolist(), index=data.index)
# Drop the original 'fees' and 'addresses' columns
data = data.drop(['fees', 'addresses'], axis=1)
# Print the updated DataFrame
print(data)

# Drop the 'Description' column
data = data.drop('Description', axis=1)

#rename columns
data = data.rename(columns={'id': 'ID', 'name':'Name', 'parkCode': 'Park Code','latitude': 'Latitude', 'longitude':'Longitude','numberOfSitesReservable' :'Number Of Sites Reservable','numberOfSitesFirstComeFirstServe' : 'Number Of Sites First Come First Serve', 'accessibility.classifications' : 'Accessibility Classifications', 'cost' : 'Cost', 'zip' : 'Zip', 'city' : 'City'})

output
	Unnamed: 0	ID	Name	Park Code	Latitude	Longitude	Number Of Sites Reservable	Number Of Sites First Come First Serve	Accessibility Classifications	Cost	Zip	City
0	0	EA81BC45-C361-437F-89B8-5C89FB0D0F86	277 North Campground	amis	29.512374	-100.908166	1	17	['Limited Development Campground']	6.0	NaN	NaN
```
Saving data to a CSV file to work with cleaned data.
```python
data.reset_index(drop=True, inplace=True)
data.to_csv('nps_camp.csv') 
Next I examined the different statistics of my data by performing a describe function.
```

Examining the Statistics in my Camp Data:
We can look at the averages, the count, quantile percentages, min and max of our variables in our data set by using a describe function.

```python
data.describe()

output
 	Unnamed: 0	Latitude	Longitude	Number Of Sites Reservable	Number Of Sites First Come First Serve	Cost
count	597.000000	596.000000	596.000000	597.000000	597.000000	597.000000
mean	316.631491	40.256554	-100.666231	33.046901	28.103853	27.773702
std	183.788437	6.880251	19.059403	61.664614	409.626339	82.856382
min	0.000000	18.352310	-156.237592	0.000000	0.000000	0.000000
25%	156.000000	36.190970	-116.507948	0.000000	0.000000	5.000000
50%	317.000000	38.988440	-103.410696	4.000000	0.000000	20.000000
75%	476.000000	45.932857	-83.621052	37.000000	10.000000	28.000000
max	637.000000	63.733360	-64.754081	432.000000	10000.000000	1000.000000
```
I was blown away by the campground max cost of $1000.  I was not expecting a campground to be that expensive. 

I am going to plot the data on a histogram and see the most expensive campsites.  This will also show the very expensive outliers.
# Visualizations

```python
import matplotlib.pyplot as plt
# Convert 'Cost' column to numeric (remove dollar signs, convert to float)
#data['Cost'] = pd.to_numeric(data['Cost'].replace('[\$,]', '', regex=True), errors='coerce')
# Filter out rows where 'Cost' is NaN
data = data.dropna(subset=['Cost'])
# Sort the DataFrame by 'Cost' in descending order and get the top 10
top_costs = data.sort_values(by='Cost', ascending=False).head(10)
# Plot the histogram
plt.figure(figsize=(10, 6))
plt.bar(top_costs['Name'], top_costs['Cost'], color='green')
plt.xlabel('Campground Name')
plt.ylabel('Cost')
plt.title('Top 10 Campground Costs')
plt.xticks(rotation=45, ha='right')
plt.tight_layout()
plt.show()
```
![cost_hist.png](/assets/images/cost_hist.png)


There are two campsites with the max cost.  I wanted to examine the details of the row with Camp Greentop Park.

```python
# Filter out rows where 'Cost' is NaN
data = data.dropna(subset=['Cost'])
# Find the index of the maximum value in the 'Cost' column
max_cost_index = data['Cost'].idxmax()
# Access the row corresponding to the maximum cost using loc
row_with_max_cost = data.loc[max_cost_index]
print("Row with the highest cost:")
print(row_with_max_cost)
#Thurmont MD

Output
Row with the highest cost:
Unnamed: 0                                                                              103
ID                                                     D378656F-72C7-4F18-A3CC-741CBEE9B394
Name                                                                          Camp Greentop
Park Code                                                                              cato
Latitude                                                                          39.644541
Longitude                                                                        -77.476565
Number Of Sites Reservable                                                                1
Number Of Sites First Come First Serve                                                    1
Accessibility Classifications                                      ['Developed Campground']
Cost                                                                                 1000.0
Description                               Entire facility sleeps 140 people in cabins. \...
Zip                                                                                   21788
City                                                                               Thurmont
Name: 103, dtype: object

```


# Further Investigation 

I then wanted to look at the top 10 most expensive campsites.
```python
# Find the top 10 most expensive campgrounds
top_10_expensive = camp_data.nlargest(10, 'Cost')
print("Top 10 most expensive campgrounds:")
print(top_10_expensive)

 Truncated Output
Top 10 most expensive campgrounds:
     Unnamed: 0                                    ID  \
103         103  D378656F-72C7-4F18-A3CC-741CBEE9B394   
105         105  905221FE-094D-4E73-90EB-205CBCAD69AA   
95           95  2D5D7B26-19D2-4583-A370-86575EA22C2F   
91           91  9CAD260F-57A8-4814-87FA-E61D14108C81   
94           94  67817EE3-C197-45DD-AEF0-E5E86061691B   
92           92  CBA8D848-FB74-4492-9E5D-E1E4979A976B   
93           93  B26D8D06-514E-4230-A314-CF0ED2023B98   
254         254  6A989D7D-8AA1-432E-A915-337BB2B45E4C   
141         141  2E829940-0ECC-49A7-85FB-F0AE8D45EBA0   
178         178  665A07F7-CD99-401A-8674-4C65AC41954C   

                                   Name Park Code   Latitude   Longitude  \
103                       Camp Greentop      cato  39.644541  -77.476565   
105                   Camp Round Meadow      cato  39.644917  -77.487628   
95   Cabin Camp 5 (By Reservation Only)      prwi  38.575779  -77.412568   
91   Cabin Camp 1 (By Reservation Only)      prwi  38.597018  -77.356053   
94   Cabin Camp 4 (By Reservation Only)      prwi  38.591509  -77.353861   
92   Cabin Camp 2 (By Reservation Only)      prwi  38.581375  -77.415737   
93   Cabin Camp 3 (By Reservation Only)      prwi  38.563672  -77.364930   
254                      Group Campsite      care  38.278900 -111.251400   
141                  Colter Bay RV Park      grte  43.905642 -110.641324   
178                   Dunbar Group Site      indu  41.682660  -87.001714   
93   This is the price to rent out the group part o...  55347       Triangle  
254   Nightly cost to stay at the group site in Fruita  84775         Torrey  
141  Fee per night for campers with vehicle. All in...  83013          Moran  
178  The group site rate is valid for up to 30 peop...  46304         Porter 

```

# Conclusion:

 In this blog post I wanted to scrape data from the National Park Service using an API.  I cleaned and organized the scraped data and saved it to a csv file that I can use to write further code for my project, and my streamlit app. I wanted to look at the most expensive campsites.  I plotted a histogram and examined details further for each row to try to understand more details about the top camping costs from my dataset.

# Looking Forward
 In my next blog post I want to answer these questions with my camp data:
- Is there potential correlation between cost of campsite with reserved vs. first come first serve campsites, and location of campsite with latitude, longitude and zip code?
- What are the cost outliers in my data?
- Does my data follow a Normal or a Power Law Distribution? 
- What the are the Costs of campgrounds by Zip code regions?
- How does Utah's camping costs compare to surrounding states?
- What the frequency of Park Codes, ie total campsites per Park Code in my data?

Here is a link to my [github repository](https://github.com/bekahwebb/Camp_Blog_Code/). It contains the code I used for both this blog post and the next post and data.


![Nature_camp.png](/assets/images/Nature_camp.png)

