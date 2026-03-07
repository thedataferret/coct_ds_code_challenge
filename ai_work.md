## AI Work Summary

This document summarises how I used AI in my assignment. 

I started by opening all the data files and just visually inspecting them. I took notes on the side about what the structure seemed to be, and what methods I might use to test those assumptions, and test for common data quality issues that I know from long, hard-earned, often stressful experience.

I used Cursor as my IDE, with Opus 4.5 as my model. I have been using 4.5 for more than a month at work, and I somewhat trust the outputs. I haven't had a chance to play with 4.6 yet, and I'm not feeling curious this weekend.

I spent almost an hour crafting my prompt, I find that front-loading that effort - doing a LOT of thinking and crafting a really detailed and structured prompt - is quite economical when it comes to token usage. 
My dashboard says that one mega prompt cost about 2.9m tokens, or $2.98. Well worth it IMO.
To save tokens I'll probably clear my context and start a new chat. Questions from here are likely to be much smaller.

### Initial prompt

Full text of initial prompt

I’m going to create a model to classify areas of the City of Cape Town as sparsely or densely populated. 
I want to output a hex map of the city, with areas colour-coded on a 5-point scale of sparsity / density.
The assumption is that high volumes of service requests will originate from densely-populated areas.

All work should be contained in the classification.ipynb notebook, using Python and following PEP8 guidelines

PART 1: Input data
Data source 1: the city’s service request data, by level 8 H3 Geo hex - see raw_data/sr_hex.csv

Data source 2: The polygons and index values for the H3 Geo hexes - see raw_data/hex_polygons.geojson

- ‘h3_level8_index’ in sr_hex.csv links to ‘index’ in hex_polygons.geojson
- example index value: ‘88ad361801fffff’
- if the service request does not have valid geolocation data, the h3_level8_index will be 0

Expected data schema of sr_hex.csv
notification_number - string, 12 numbers, often with leading zeroes, not nullable.
reference_number - 10 numbers
creation_timestamp - timestamp in the format ‘2020-06-24 13:33:44+02:00’
completion_timestamp - timestamp in the format ‘2020-06-24 13:33:44+02:00’
directorate,department - string, ALL CAPS
branch - string
section - string
code_group - string
code - string, not nullable
cause_code_group - string
cause_code - string (may contain commas. String is escaped with “quotation marks” when necessary)
official_suburb - string
latitude - float (double precision, 14 decimal places) values between 17 and 20
longitude - float (double precision, 14 decimal places) values between -35 and -32
h3_level8_index - string, 15 alphanumeric OR 0, not nullable.

Suggested data cleaning checks for the sr_hex.csv data:

- data type discrepancies in columns
- parsing issues and malformed lines (eg; unescaped commas in string fields)
- invalid coordinates (out of bounds)
- timestamps older than 2020, or in the future.
- timestamps not showing timezone as ‘+02:00’

Show a report of 

- the null rates per field
- min and max values
- malformed lines

Print a sample of the malformed lines for inspection

PART 2: Exploration of data

Output 1: 
Create and print a table of statistics where each row represents one service request code, that shows the suburb that logged the most SRs with that code, the total and what the median is across suburbs. 
code the code of the service request
highest_suburb: the suburb that had the highest count of service requests for that particular code
highest_requests: the number of service requests with that specific code logged by that suburb
median_requests: the median number of service requests with that specific code per suburb
total_requests: the count of all requests in the data that were logged with this code.

Order the table by total_requests descending

Output 2:
Create and print a table with one row per official_suburb and columns as follows:
official_suburb - suburb name
total_requests - sum of all service requests logged by this suburb
decile - which decile by total_sr does this suburb fall into?

Order the table by total_requests descending

Output 3:
Print a bar chart showing the number of service requests (y-axis) by code (x-axis)
Order from left (highest) to right (lowest)

Output 4: 
Print a bar chart showing the number of service requests (y-axis) by suburb (x-axis)
Order from left (highest) to right (lowest)

Output 5: 
Print a 3d bar chart showing the number of service requests (y-axis) by suburb (x-axis) by code (z-axis)
Colourise with a colourblind-friendly palette with high values in hot colours, low values in cool colors

PART 3. Classification and presentation

Output 1: 
Create an interactive map that loads in browser window showing the hex shapes colorized by the related service request data
The user should be able to hover over a hex to see a tooltip with the official_suburb name, and the number of service requests
The user should be able to include or exclude specific service request codes with checkboxes 

---

## Review of the Opus 4.5 output

---

#### Unhelpful grouping of data validation tests

The initial build produced this data test that I found a bit useless - if you group the out of bounds coordinates with the null coordinates that gives you nothing actionable: 

```
    invalid_lat = df[
        (df['latitude'] < -35) | (df['latitude'] > -32) |
        df['latitude'].isnull()
    ]
    issues['invalid_latitude'] = len(invalid_lat)
```

So I've split that data check into null_lat and oob_lat

---

#### Timezone bad assumption

Another data test issue - the 44 out of bounds timestamps are an artifact of timezone nonsense

```
    cutoff_date = pd.Timestamp('2020-01-01', tz='UTC')
    future_date = pd.Timestamp.now(tz='UTC')
```

All fixed now that tz='Africa/Johannesburg'

---

#### Malformed lines check was nonsense

I queried how the malformed lines checker works, and it gave me a very pretty answer.  
Then I tested it by manually typing `FROGGY!,FROGGY!,FROGGY!` to the first line of data in sr_hex.csv and ... no malformed lines were found. My froggies are definitely there, because it threw all my other data tests out of whack.

Oh and then it lied to me about the problem. Twice. But we got there in the end.

It happened because Pandas will silently expand the dataframe on encountering this kind of failure. The initial code written by Opus4.5 did not account for this.

We've handled it by adding explicit column expectations. It's rustic but readable and unambiguous.
:frog: :frog: :frog:

---

#### Sample records of validation issues

I was supposed to get some sample lines of any problematic data, so I can see it for myself. 
Unfortunately Opus4.5 wrote something that only looks at out of bounds coordinates.
We've updated it to look at ALL data validation checks, print a short table of records that fail validation per type of failure (and grouped lat/long tests together because they generally fail together).