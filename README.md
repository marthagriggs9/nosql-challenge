# nosql-challenge

## Instructions

The UK Food Standards Agency evaluates various establishments across the United Kingdom and gives them a food hygiene rating. You've been contracted by the editors of
a food magazine, *'Eat Safe, Love'* to evaluate some of the ratings data in order to help their journalists and food critics decide where to focus future articles. 

## Part 1: Database and Jupyter Notebook Set Up

[NoSQL_setup_starter.ipynb](https://github.com/marthagriggs9/nosql-challenge/blob/main/NoSQL_setup_starter.ipynb)

1. Import the data provided in the `establishments.json` file from your Terminal. Name the database `uk_food` and the collection `establishments`. 
```ruby
mongoimport --type json -d uk_food -c establishments --drop --jsonArray establishments.json
```

2. Within your notebook, import the libraries you need: PyMongo and Pretty Print
```ruby
# Import dependencies
from pymongo import MongoClient
from pprint import pprint
```
3. Create an instance of the Mongo Client
```ruby
# Create an instance of MongoClient
mongo = MongoClient(port=27017)
```
4. Confirm that you created the database and loaded the data properly
   * List the databases you have in MongoDB. Confirm that `uk_food` is listed.
   ```ruby
   # confirm that our new database was created
   print(mongo.list_database_names())
   ```
   ![Screenshot 2023-03-18 182403](https://user-images.githubusercontent.com/115905663/226145216-46cdf27b-a5d6-49b3-987f-8ed3143022a9.png)
   
   ```ruby
   # assign the uk_food database to a variable name
   uk_food_db = mongo["uk_food"]
   ```

   * List the collection(s) in the database to ensure that `establishments` is there. 
   ```ruby
   # review the collections in our new database
   print(uk_food_db.list_collection_names())
   ```
   ![Screenshot 2023-03-18 182742](https://user-images.githubusercontent.com/115905663/226145251-402e52de-94ff-4cdc-ad3a-b3dae5e56d10.png)

   
   * Find and display one document in the `establishments` collection using `find_one` and display with `pprint`. 
   ```ruby
   # review a document in the establishments collection
   pprint(uk_food_db.establishments.find_one())
   ```
   ![image](https://user-images.githubusercontent.com/115905663/226145280-c2b7dd2e-c5ba-4bad-8049-e8e8346a72bc.png)

5. Assign the `establishments` collection to a variable to prepare the collection for use. 
```ruby
# assign the collection to a variable
establishments = uk_food_db['establishments']
```

## Part 2: Update the Database

The magazine editors have some requested modifications for the database before you can perform any queries or analysis for them. Make the following changes to the `establishments` collection:

1. An exciting new halal restaurant just opened in Greenwich, but hasn't been rated yet. The magazine has asked you to include it in your analysis. Add the following information to the database:
```ruby
halal_restaurant = {
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
}
```
```ruby
# Insert the new restaurant into the collection
establishments.insert_one(halal_restaurant)
```
```ruby
# Check that the new restaurant was inserted
establishments.find_one({'BusinessName': 'Penang Flavours'})
```
![image](https://user-images.githubusercontent.com/115905663/226145361-edbb5ac9-c9a6-4d9d-bf21-7379afef4d50.png)

2. Find the BusinessTypeID for 'Restaurant/Cafe/Canteen' and return only the `BusinessTypeID` and `BusinessType` fields. 
```ruby
# Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the BusinessTypeID and BusinessType fields
query = {'BusinessType': 'Restaurant/Cafe/Canteen'}
fields = {'BusinessType': 1, 'BusinessTypeID': 1}

results = establishments.find(query, fields)

for result in results:
    pprint(result)
```
![image](https://user-images.githubusercontent.com/115905663/226145391-7571354b-9b96-4463-b502-1b67e43cecbf.png)

3. Update the new restaurant with the `BusinessTypeID` you found. 
```ruby
# Update the new restaurant with the correct BusinessTypeID
uk_food_db.establishments.update_one(
    halal_restaurant, 
    {'$set': {'BusinessTypeID': '1'}})
```
```ruby
# Confirm that the new restaurant was updated
establishments.find_one({'BusinessName': 'Penang Flavours'})
```
![image](https://user-images.githubusercontent.com/115905663/226145622-1364239a-68a7-48d6-a972-8e0d5cb62b8b.png)

4. The magazine is not interested in any establishments in Dover, so check how many documents contain the Dover Local Authority. Then, remove any establishments within the Dover Local Authority from the database and check the number of documents to ensure they were deleted. 
```ruby
# Find how many documents have LocalAuthorityName as "Dover"
dover_query = {'LocalAuthorityName': 'Dover'}

print("The number of establishments in Dover:", establishments.count_documents(dover_query))
```
![image](https://user-images.githubusercontent.com/115905663/226145467-3ee11b08-60f7-4e28-a3d4-a333a3a90a69.png)
```ruby
#view results
results = establishments.find(dover_query)
for result in results:
    pprint(result)
```
![image](https://user-images.githubusercontent.com/115905663/226145493-57194ed7-8c3b-4cf3-8209-17497e6d0ebf.png)
```ruby
# Delete all documents where LocalAuthorityName is "Dover"
establishments.delete_many(dover_query)
```
```ruby
# Check if any remaining documents include Dover
print("The number of establishments in Dover:", establishments.count_documents(dover_query))
```
![image](https://user-images.githubusercontent.com/115905663/226145525-614eb7d1-fb3b-4e42-8a01-26b40e435022.png)

5. Some of the number values are stored as strings, when they should be stored as numbers. Use `update_many` to convert `latitude` and `longitude` to decimal numbers. 
```ruby
# Change the data type from String to Decimal for longitude
establishments.update_many({}, [
    {'$set': 
        {'geocode.longitude':
            {'$toDouble': '$geocode.longitude'}}}
])
```
```ruby
# Change the data type from String to Decimal for latitude
establishments.update_many({}, [
    {'$set': 
        {'geocode.latitude':
            {'$toDouble': '$geocode.latitude'}}}
])
```
```ruby
# Check that the coordinates are now numbers
coord_query = {}
fields = {'geocode.longitude': 1, 'geocode.latitude': 1}

results = establishments.find(coord_query, fields)

[result for result in establishments.find({}, ['geocode.longitude', 'geocode.latitude'])]
```
![image](https://user-images.githubusercontent.com/115905663/226145575-931d24f5-7567-4dfc-af51-823a40e35925.png)

## Part 3 Exploratory Analysis

*Eat Safe, Love* has specific questions they want you to answer, which will help them find the locations they wish to visit and avoid. 

[NoSQL_analysis_starter.ipynb](https://github.com/marthagriggs9/nosql-challenge/blob/main/NoSQL_analysis_starter.ipynb)

Some notes to be aware of: 
  * `RatingValue` refers to the overal rating decided by the Food Authority and ranges from 1-5. The higher the value, the better the rating. Note: This field also includes non-numeric valies such as 'Pass', where 'Pass' means that the establishmen passed their inspection but isn't given a number rating. 
  * The score for Hygiene, Structural and ConfidenceManagement work in reverse. This means, the higher the value, the worse the establishment is in these areas. 
  
Use the following questions to explore the database and find the answers, so you can provide them to the magazine editors. 

Unless otherwise stated, for each question:
  * Use `count_documents` to display the number of documents contained in the result. 
  * Display the first document in the results using `pprint`. 
  * Convert the result to a Pandas DataFrame, print the number of rows in the DataFrame and display the first 10 rows. 



1. Which establishments have a hygiene score equal to 20?
```ruby
# Find the establishments with a hygiene score of 20
query = {'scores.Hygiene': 20}

# Use count_documents to display the number of documents in the result
print("Number of restaurants with a Hygiene score of 20:", establishments.count_documents(query))
# Display the first document in the results using pprint
print("First Result:")
results = establishments.find(query)

pprint(results[0])
```
![image](https://user-images.githubusercontent.com/115905663/226145853-3b941fee-92c3-4e26-b5b6-cd142e925993.png)

```ruby
# Convert the result to a Pandas DataFrame
hygiene_df = pd.DataFrame(results)
# Display the number of rows in the DataFrame
print("Rows in DataFrame: ", len(hygiene_df))
# Display the first 10 rows of the DataFrame
hygiene_df.head(10)
```
![image](https://user-images.githubusercontent.com/115905663/226145909-5d0fda07-718e-4e05-8c05-798add5610f0.png)

2. Which establishments in London have a `RatingValue` greater that or equal to 4?
```ruby
# Find the establishments with London as the Local Authority and has a RatingValue greater than or equal to 4.
query = {'LocalAuthorityName': {'$regex': 'London'}, 
         'RatingValue': {'$gte': '4'}}

# Use count_documents to display the number of documents in the result
print("Number of restaurants in London with a Rating Value of 4 or more:", establishments.count_documents(query))
# Display the first document in the results using pprint
print("First Result:")
results = establishments.find(query)
pprint(results[0])
```
![image](https://user-images.githubusercontent.com/115905663/226145926-9a545e71-d6ff-48e2-8df9-6204ca20987e.png)

```ruby
# Convert the result to a Pandas DataFrame
london_rating_df = pd.DataFrame(results)
# Display the number of rows in the DataFrame
print("Rows in DataFrame: ", len(london_rating_df))
# Display the first 10 rows of the DataFrame
london_rating_df.head(10)
```
![image](https://user-images.githubusercontent.com/115905663/226145951-a6a0e37d-53a6-4eeb-9f5d-e6ad9b1313af.png)

3. What are the top 5 establishments with a `RatingValue` of '5', sorted by lowest hygiene score, nearest to the new restaurant added, 'Penang Flavours'?
```ruby
#Find latitude and longitude of 'Penang Flavours'
pprint(establishments.find_one({'BusinessName':'Penang Flavours'}, {'geocode.latitude', 'geocode.longitude'}))
```
![image](https://user-images.githubusercontent.com/115905663/226145965-1dc07343-a22f-4e01-a1c2-1ca26ccc4169.png)

```ruby
# Search within 0.01 degree on either side of the latitude and longitude.
# Rating value must equal 5
# Sort by hygiene score

degree_search = 0.01
latitude = 51.490142
longitude = 0.08384

query = {'RatingValue': '5', 
         'geocode.latitude': {'$lte': (latitude + degree_search), '$gte': (latitude - degree_search)}, 
         'geocode.longitude' : {'$lte': (longitude + degree_search), '$gte': (longitude - degree_search)}}
sort =  [('scores.Hygiene', 1)]

limit = 5
# Print the results
pprint(list(establishments.find(query).sort(sort).limit(limit)))
```
![image](https://user-images.githubusercontent.com/115905663/226145991-7a6d5f0e-9e41-4d22-84c5-42bd471d61ee.png)

```ruby
# Convert result to Pandas DataFrame
rating_value_df = pd.DataFrame(establishments.find(query).sort(sort).limit(limit))
rating_value_df
```
![image](https://user-images.githubusercontent.com/115905663/226146023-c24f7ce1-cbc7-4b1a-8a8e-687c71d446ca.png)

4. How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest and print out the top ten local authority areas. 
```ruby
# Create a pipeline that: 
# 1. Matches establishments with a hygiene score of 0
match_query = {'$match': {'scores.Hygiene': 0}}
# 2. Groups the matches by Local Authority
group_query = {'$group': {'_id': {'LocalAuthorityName': '$LocalAuthorityName'}, 'count': {'$sum': 1}}} 
# 3. Sorts the matches from highest to lowest
sort_values = {'$sort': {'count': -1}}
#put the pipeline together
pipeline = [match_query, group_query, sort_values]
#Run the pipeline through the aggregate method, case the results as a list and save the results to a variable
results = list(establishments.aggregate(pipeline))
# Print the number of documents in the result
print("Number of rows in result: ", len(results))
# Print the first 10 results
pprint(results[0:10])
```
![image](https://user-images.githubusercontent.com/115905663/226146059-f5a91804-94c1-468d-b652-11c96bc4370b.png)

```ruby
#Extract the fields from the _id so they're in separate columns in a Pandas DataFrame
aggregated_df = pd.json_normalize(results)
print(f"There are {len(aggregated_df)} rows in the DataFrame.")
aggregated_df.head(10)
```
![image](https://user-images.githubusercontent.com/115905663/226146076-3767e859-1890-4c5a-b58c-d8f83815f3a1.png)

```ruby
# Convert the result to a Pandas DataFrame
hygiene_score_0_df = pd.DataFrame(aggregated_df)
# Display the number of rows in the DataFrame
print(f"There are {len(hygiene_score_0_df)} rows in the DataFrame.")
# Display the first 10 rows of the DataFrame
hygiene_score_0_df.head(10)
```
![image](https://user-images.githubusercontent.com/115905663/226146090-32fb026b-4433-462d-b689-16551706407a.png)

```ruby
#Rename columns 
hygiene_score_0_df = hygiene_score_0_df.rename(columns= {"count": "Number of Establishments", 
                                               "_id.LocalAuthorityName": "Local Authority Name"})
hygiene_score_0_df.head()
```
![image](https://user-images.githubusercontent.com/115905663/226146115-5fab4510-c97c-4fe4-ae73-399dae66eada.png)

```ruby
#Reorder the columns
hygiene_score_0_df = hygiene_score_0_df[["Local Authority Name", "Number of Establishments"]]
#Print the first 10 rows of the DataFrame
hygiene_score_0_df.head(10)
```
![image](https://user-images.githubusercontent.com/115905663/226146128-5fcbb878-6c70-4267-8670-cbce5912e2c8.png)
