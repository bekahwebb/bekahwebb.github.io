---
layout: post
title:  "A look at the Cost of Camping at all 63 National Parks."
author: Rebekah Webb
description: This is an Exploratory Data Analysis.
image: /assets/images/camping.png
---
# Introduction:

My family and I like to go camping typically in Utah at the National Parks down south.  We have camped in various parts of the country and found the camping fees to be pretty reasonable.  I thought it might be interesting to look at the data of the National Park service, NPS and find the most expensive campgrounds of the different National Parks around the country.  I want to explore the data of the NPS and see if there are any correlations between cost of campsite and first come first serve campsites vs. reserved campsites.  I also want to explore the location of the campsite and see if there is correlation between cost and latitude and longitude and zip code of the campsite to see if the location affects the higher campsite costs.

# Scraping the NPS Data

I started with registering with the NPS for an apikey which they emailed me.  I then loaded in the libraries including a pd.set_option to display all the columns in case there were any embedded columns that I couldn't examine.  Then I put a url request in to the NPS API for the camping data.

``` 
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

My request was successful and my output looked like this:
output
 
id	url	name	parkCode	description	latitude	longitude	latLong	audioDescription	isPassportStampLocation	passportStampLocationDescription	passportStampImages	geometryPoiId	reservationInfo	reservationUrl	regulationsurl	regulationsOverview	fees	directionsOverview	directionsUrl	operatingHours	addresses	images	weatherOverview	numberOfSitesReservable	numberOfSitesFirstComeFirstServe	multimedia	relevanceScore	lastIndexedDate	amenities.trashRecyclingCollection	amenities.toilets	amenities.internetConnectivity	amenities.showers	amenities.cellPhoneReception	amenities.laundry	amenities.amphitheater	amenities.dumpStation	amenities.campStore	amenities.staffOrVolunteerHostOnsite	amenities.potableWater	amenities.iceAvailableForSale	amenities.firewoodForSale	amenities.foodStorageLockers	contacts.phoneNumbers	contacts.emailAddresses	campsites.totalSites	campsites.group	campsites.horse	campsites.tentOnly	campsites.electricalHookups	campsites.rvOnly	campsites.walkBoatTo	campsites.other	accessibility.wheelchairAccess	accessibility.internetInfo	accessibility.cellPhoneInfo	accessibility.fireStovePolicy	accessibility.rvAllowed	accessibility.rvInfo	accessibility.rvMaxLength	accessibility.additionalInfo	accessibility.trailerMaxLength	accessibility.adaInfo	accessibility.trailerAllowed	accessibility.accessRoads	accessibility.classifications
0	EA81BC45-C361-437F-89B8-5C89FB0D0F86	https://www.nps.gov/amis/planyourvisit/277-nor...	277 North Campground	amis	277 North Campground is generally open year-ro...	29.512373695509215	-100.90816633365614	{lat:29.512373695509215, lng:-100.90816633365614}	277 North Campground is generally open year-ro...	0		[]	582AB459-28A8-453A-B22E-BAD481E92098	Standard overnight camping is first-come, firs...	https://www.recreation.gov/camping/campgrounds...	https://cms.nps.gov/amis/learn/management/comp...	Site capacity is not to exceed eight persons a...	[{'cost': '6.00', 'description': 'Sites are av...	Directions from Amistad National Recreation Ar...		[{'exceptions': [], 'description': '277 North ...	[]	[{'credit': 'NPS Photo', 'crops': [], 'title':...	The climate at Amistad is semi-arid in moistur...	1	17	[]	1.0		Yes - year round	[Vault Toilets - year round]	No	[None]	Yes - year round	No	No	No	No	No	[No water]	No	No	No	[]	[{'description': '', 'emailAddress': 'AMIS_Int...	18	1	0	0	0	0	0	0	Limited. However, some sites do have a cement ...			Ground fires are not permitted. Each campsite ...	1	RV and Trailers are permitted	0		0	The main road leading to the campground is pav...	1	[Paved Roads - All vehicles OK]	[Limited Development Campground] 
```
This is just the first row of code so there are definitly a lot of columns to go through to get to the heart of what my data analysis is.  There is a lot of text and url websites etc. that I just don't need so I organized my columns by printing the columns of my data and commenting out the ones that I don't want and making a copy for python to read the new copy of my data.


# Organizing data columns

```
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
Next I cleaned the data by pulling two variables out of two of the columns: fees and addresses and made individual columns of cost, description, zip, and city.  I wanted to examine the description column closer to see if the output would be useful.  It was really messy in putting random text in different rows and it didn't give me to much information about why the camp charged the cost that it did so I dropped description from the dataset.  I renamed the columns by making a dictionary list, and my output showed the data output I wanted.

# Cleaning the data and renaming columns
```
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
# Saving data to a CSV file to work with cleaned data.
```
data.reset_index(drop=True, inplace=True)
data.to_csv('nps_camp.csv') 
Next I examined the different statistics of my data by performing a describe function.
```
# Examining data
We can look at the statistics of our variable in our data set by using a describe function.

```
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
I was blown away by the campground max cost of $1000.  I was not expecting a campground to be that expensive.  We can further investigate the stats on the campground with camp stats and location and see if we can find some more answers for the most expensive campground.

I am going to plot the data on a histogram and see the most expensive campsites.  This will also show the very expensive outliers.
# Visualizations

```
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

Output
```
![cost_hist.png](/assets/images/cost_hist.png)


There are two campsites with the max cost.  I wanted to examine the details of the row with Camp Round Meadow.

```
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

# Conclusion:

 In this blog post I wanted to scrape data from the National Park Service using an API.  I cleaned and organized the scraped data and saved it to a csv file. I wanted to look at the most expensive campsite.  I plotted a histogram and examined details further for each row.

# Looking forward:
 In my next blog post I will go through more visualizations as I analyze potential correlation between cost of campsite, and reserved vs. first come first serve campsites, and location of campsite with latitude, longitude and zip code.

![camp2.png](/assets/images/camp2.png)

