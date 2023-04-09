# A case of utilizing GPT-4 to query a database.
## Motivation
One of the main advantages of using GPT-4 for querying a database is that it allows us to ask direct questions about the data in natural language, just as we would to a personal assistant. This means that we don't need to have extensive knowledge of the database schema or complex SQL queries to extract the information we need.

Another advantage of using GPT-4 for querying a database is that it can help us identify patterns and trends in the data that may not be immediately apparent when using extensive reports or dashboards.

Finally, using GPT-4 for querying a database can help bridge the gap between technical and non-technical persons. By allowing anyone to ask questions about the data in natural language, we can make the information more accessible and easier to understand for non-technical stakeholders, such as executives or marketing teams.

## Description of the application

The app has been developed with the aim of simplifying the querying of purchase data. The database stores data by day, providing a clear view of transactions, their amounts, and final prices. To make this information more accessible, the data is grouped by day and converted into a JSON format that allows for easy parsing and querying.

When users make requests for information, the system uses a GPT-4 query with a specially designed prompt to identify the relevant dates then, the data for the corresponding dates are collected and appended to the original query. This helps ensure that GPT-4 has access to all the relevant data required to provide a comprehensive and accurate response.

To further streamline the process, the app restricts queries only to those that are relevant to the app. Another special prompt is prepared for GPT-4 to decide whether the query is allowed or not. This approach helps ensure that only valid and appropriate queries are processed by the app.

By using this app, users can easily access and analyze the purchase data without needing extensive technical knowledge or complex querying skills. Overall, the app provides a user-friendly and efficient way to manage and process purchase data.


## Data Preparation

The code contains a notebook called ‘PrepareData’; here we start reading the 'salesdata.csv' containing the purchase data for one customer by day with attributes: purchase date, product name, purchase amount and total price.

This data is grouped by day and then each day is converted to JSON format, at the end, a new csv file will be exported (sales_data_emb.csv) that will constitute the central Database for the project.


## Generating Embeddings

In the notebook called 'CalcEmbeddings' the embeding vectors for each 'products' row of the sales_data_emb.csv database is calculated. These embeddings will be stored in the column 'embeddings' of the sales_data_emb.csv file.

This is not actually necessary for the project but will be used in future versions to test how well the Database search using the embeddings works compared with the method used here.

To calculate the embeddings the openai 'text-embedding-ada-002' model is used.


## Quering the Database

The actual application is developed in the notebook called ‘QueringData’. Here the following functions are defined:

get_responde_from_gpt4 – This function will execute the actual request from the GPT-4 model using its API.

is_a_valid_qry – This function will query GPT-4 to define if the user query is a valid one, basically one that is in the scope of the app. You can see in the code the text that were used to obtain that result. This is only an example of how GPT-4 could be used to sensor itself from unwanted queries.

dates_to_be_included_to_answer_the_query – Here GPT-4 is queried again for the dates that the user query refers to. For example, queries containing terms like ‘last quarter’ will return the begin and end date that last quarter refers to. The current date is assumed to be the last date stored in the database. If no dates can be inferred from the user query then GPT-4 shuld return the dates corresponding to last semester. In the code you can see the prompt used to obtain these results.

get_answer_from_query – Tis last function is basically the one that execute the final GPT-4 request to obtain the response for the user query. It basically ad the data obtained from the Database (corresponding to the selected dates) to a special prompt to let GPT-4 do its work.

The rest of the program is a loop executing the following tasks:
a-	Ask for a user query.
b-	Call is_a_valid_qry to see if the query can be processed.
c-	Call dates_to_be_included_to_answer_the_query to determine which data should be extracted from the Database according to those dates.
d-	Search the Database and concatenate all the data into a new JSON string.
e-	Call get_answer_from_query passing the JSON data and the user query.
f-	Print the result obtained from GPT-4


## Example

### Compare last purchase with the last trimester ones.

To compare the last purchase with the last trimester ones, let's first identify the purchases within the last trimester and the last purchase date.

Last purchase date: 5/27/2016
Last trimester purchases: 2/5/2016, 2/6/2016, 4/26/2016, 4/28/2016, 5/27/2016

Now let's compare the purchases:

Last Purchase (5/27/2016):
1. 10 mm Double sided bubble wrap 20m - 20 units - $600
2. Tape dispenser (Black) - 50 units - $1600
Total amount spent: $2200

Last Trimester Purchases:
1. 2/5/2016 - Total amount spent: $562
2. 2/6/2016 - Total amount spent: $2760
3. 4/26/2016 - Total amount spent: $830.4
4. 4/28/2016 - Total amount spent: $615
5. 5/27/2016 - Total amount spent: $2200

Comparing the last purchase with the last trimester ones, it can be observed that the last purchase has a higher total amount spent than the purchases made on 2/5/2016, 4/26/2016, and 4/28/2016. However, the total amount spent on the last purchase is lower than the purchase made on 2/6/2016.

Adjusting the temperature of the query we can obtain differnt analysis levels from the most consice to the more verbose ones.

If I want to buy Tape dispenser (Black) what other products can you suggest?
Based on your previous purchases, I would suggest the following products:

1. Black and orange handle with care despatch tape 48mmx75m
2. Permanent marker red 5mm nib (Red) 5mm
3. Packing knife with metal insert blade (Yellow) 9mm
4. Shipping carton (Brown) 500x310x310mm
5. 10 mm Anti static bubble wrap (Blue) 20m
6. 10 mm Double sided bubble wrap 20m
7. 20 mm Double sided bubble wrap 20m

### If I want to buy Tape dispenser (Black) what other products can you suggest?

These products are related to packaging and shipping, which might be useful along with the Tape dispenser (Black).

Based on your previous purchases, I would suggest the following products:

1. Black and orange handle with care despatch tape 48mmx75m
2. Permanent marker red 5mm nib (Red) 5mm
3. Packing knife with metal insert blade (Yellow) 9mm
4. Shipping carton (Brown) 500x310x310mm
5. 10 mm Anti static bubble wrap (Blue) 20m
6. 10 mm Double sided bubble wrap 20m
7. 20 mm Double sided bubble wrap 20m

These products are related to packaging and shipping, which might be useful along with the Tape dispenser (Black).

In this second query GPT-4 cannot determine a date so it will respond with the default  'last semester'.

The most interesting thing here is how GPT-4 finds potentially related items based in its own knowledge because there is no indication in the Database of wich products might be similar to others.

## Conclusions

GPT-4 opens a new paradigm for application development in which you combine natural language descriptions with actions that will be driven by the results obtained from the language model. In this case we query the model first to cancel inappropriate questions and then to find the potential purchase dates involved in the user query. As the result of both request certain action are then executed, in the first case if the query is not an appropriate one then that query is not executed and in the second the dates range obtained will be used to search the database.

This opens a wide range of new possibilities of chaining natural language requests with program executions until a final result is obtained.

One thing that is needed is to incorporate a caching of the last ‘n’ queries and answer to maintain a context of the conversation.











