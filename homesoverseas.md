# HOMESOVERSEAS.RU XML FEED TECHNICAL REQUIREMENTS V2 #

Data export to be placed in the overseas property database of the website homesoverseas.ru 
is performed in XML format (http://www.w3.org/TR/REC-xml). Here is the description of the 
elements used for Data export, necessary comments and example of the export file.

## 1. Description of the elements used for Data export ##

### &lt;root&gt; ###
The root element of XML-file is &lt;root&gt;.
&lt;root&gt; may contain any number of elements &lt;object&gt;. 
Each &lt;object&gt; describes one object and must contain the following elements necessary for Data export:

### &lt;objectid&gt; ###
&lt;objectid&gt; - the object identifier in the database of the client (non-negative integer). Compulsory element. It is used for the subsequent updating of the object information. When changing to new requirements for xml take note of saving objectid in your objects. 

### &lt;ref&gt; ###
&lt;ref&gt; - number of property for additional identification of listing. (60 symbols max). Not availible for visitors, only for clients
Example: &lt;ref&gt;AB123456-2014&lt;/ref&gt;

### &lt;title&gt; ###
&lt;title&gt; - object name (up to 60 characters). Compulsory element.
Example: &lt;title&gt;House in Elsterwerda&lt;/title&gt;

### &lt;type&gt; ###
&lt; type&gt; - type of advertisement. It can possess the values: 'sale' – advertisement for sale, 'rent' – advertisement for rent. Compulsory element.
Example: &lt;type&gt;sale&lt;/type&gt;

### &lt;market&gt; ###
&lt;market&gt; - primary or secondary market. It can possess the values: 'primary' – primary market, 'secondary' – secondary market, 'mortgage' – mortgage market. Optional element.
Example: &lt;market&gt;primary&lt;/market&gt;

### &lt;annotation&gt; ###
&lt;annotation&gt; - short object description (up to 150 characters). Optional element.
Example: &lt;annotation&gt;Residential and commercial real estate is located in Elsterwerda, in Brandenburg.&lt;/annotation&gt;

### &lt;description&gt; ###
&lt;description&gt; - full object description. The text of this description should not contain text of the short description, as on the page of the object short and full descriptions are placed above each other. Optional element. Possible html-tags in container: &lt;![CDATA[ òåêñò ]]&gt;  Acceptable tags. &lt;br&gt; &lt;strong&gt; &lt;b&gt; &lt;ul&gt; &lt;li&gt; &lt;i&gt; &lt;u&gt; &lt;sup&gt;
 
Example: &lt;description&gt;Built in: 1900
Total area: 261
Condition: partial restoration was done 
Leased: Yes
Number of residential units: 5
Number of commercial units: 1
Current annual income: 15965
Full annual income: 17789
Current rental income: 8.87%
Full rental income: 9.88%&lt;/description&gt;

### &lt;price&gt; ###
&lt;price&gt; - if the advertising type is «sale», the field specifies the price of the object (non-negative integer). If the field value is 0, the web-site will show «Price on request». Compulsory element.

Example: &lt;price&gt;180000&lt;/price&gt;
If the advertising type is «rent», the field specifies the cost for the period (non-negative integer). At least one price must be indicated.
&lt;price_rent_d&gt; - Rent price per day 
&lt;price_rent_w&gt; - Rent price per week 
&lt;price_rent_m&gt; - Rent price per month  
&lt;price_rent_y&gt; - Rent price per year 

Example: 
&lt;price_rent_d&gt;100&lt;/price_rent_d&gt;
&lt;price_rent_w&gt;600&lt;/price_rent_w&gt;
&lt;price_rent_m&gt;2000&lt;/price_rent_m&gt;
&lt;price_rent_y&gt;20000&lt;/price_rent_y&gt;

### &lt;currency&gt; ###
&lt;currency&gt; - currency of price Possible values: eur (Euro), usd (USA dollar), chf (Swiss Franc), gbp (UK pound), rur (Russian ruble). Compulsory element.
Example: &lt;currency&gt;eur&lt;/currency&gt;  

### &lt;region&gt; ###
&lt;region&gt; - id of the region where the object is located (see the guide http://www.homesoverseas.ru/import/countries.php). Compulsory element.
Regions in CSV format
Id;name 
http://homesoverseas.ru/import/countries.csv.php
Id;name_eng;parentid
http://homesoverseas.ru/import/countries.parents.csv.php
Id;name_eng;parentid
http://homesoverseas.ru/import/countries.parents.eng.csv.php

Example: &lt;region&gt;42&lt;/region&gt; 
 
### &lt;realty_type&gt; ###
&lt;realty_type&gt; - type of the real estate. Compulsory element. It can possess the values:
14 - Commercial real estate
15 - Plots 
16 - Flats (apartments)
17 - Houses (villas)
18 - Townhouses
Subtypes of commercial real estate 
20 - Hotel (inn)
21 - Restaurant (cafe)
22 - Store
23 - Office
24 – Warehouse 
25 – Manufacturing 
26 - Other 
Example: &lt;realty_type&gt;16&lt;/realty_type&gt;

### &lt;bedrooms&gt; ###
&lt;bedrooms&gt; - number of bedrooms (non-negative integer). Optional element.
Example: &lt;bedrooms&gt;5&lt;/bedrooms&gt;

### &lt;size_house&gt; ###
&lt;size_house&gt; - living area in square meters. (only for apartments, villas, townhouses) Optional element.
Example: &lt;size_house&gt;5&lt;/size_house&gt;

### &lt;size_land&gt; ###
&lt;size_land&gt; - size of a land plot in square meters. (only for selling townhouses, villas and plots) Optional element.
Example: &lt;size_land&gt;5&lt;/size_land&gt;

### &lt;year&gt; ###
&lt;year&gt; - year of construction (except land plots). Optional element.
Example: &lt;year&gt;1995&lt;/year&gt;

### &lt;not_ready_year&gt; ###
&lt;not_ready_year&gt; - year of completion of construction if year of construction is not indicated. Optional element.
Example: &lt;not_ready_year&gt;5&lt;/not_ready_year&gt;

### &lt;not_ready_quarter&gt; ###
&lt;not_ready_quarter&gt; - quarter of completion of construction if year of construction is not indicated. Optional element. (1,2,3,4)
Example: &lt;not_ready_quarter&gt;4&lt;/not_ready_quarter&gt; 

### &lt;level&gt; ###
&lt;level&gt; - level (only for flats (apartments)). Optional element.
Example: &lt;level&gt;3&lt;/level&gt;

### &lt;levels&gt; ###
&lt;levels&gt; - number of floors in a building (except land plots). Optional element.
Example: &lt;levels&gt;5&lt;/levels&gt;

### &lt;distance_aero&gt; ###
&lt;distance_aero&gt; - Distance to the airport in kilometers. Optional element.
Example: &lt;distance_aero&gt;120&lt;/distance_aero&gt;

### &lt;distance_ sea&gt; ###
&lt;distance_ sea&gt; - Distance to the sea in kilometers. Optional element. Subject to the thousandths decimal values.
Example: &lt;distance_sea&gt;120&lt;/distance_sea&gt;

### &lt;distance_ski&gt; ###
&lt;distance_ski&gt; - Distance to the ski-lift in kilometers. Optional element. Subject to the thousandths decimal values
Example: &lt;distance_ski&gt;10&lt;/distance_ski&gt;

### &lt;distance_rus&gt; ###
&lt;distance_rus&gt; - Distance to the boarder with Russia in kilometers. Only for Finland, Estonia, Latvia, Lithuania. Optional element.
Example: &lt;distance_rus&gt;80&lt;/distance_rus&gt;

### &lt;option&gt; ###
&lt;option&gt; - id of the option (see Part 4 «Options»). Optional element. If there are several options for one object then the element &lt;option&gt; should be repeated.
Example:
&lt;option&gt;9&lt;/option&gt; 
&lt;option&gt;18&lt;/option&gt;
&lt;option&gt;15&lt;/option&gt;


### &lt;lat&gt; &lt;lng&gt; ###
&lt;lat&gt; - latitude
&lt;lng&gt; - longitude 
The coordinates of the object to link to the map are indicated in degrees. Decimal separator is marked by a dot. Optional element.
Example: &lt;lat&gt;56.298457922&lt;/lat&gt;&lt;lng&gt;-23.19283459&lt;/lng&gt;

### &lt;photo&gt; ###
&lt;photo&gt; - url of the file with the object photo. Optional element. For one object there can be up to 15 photos. In this case the element should be repeated. The minimal width of the picture must be 560 pixels.
Example:
&lt;photo&gt;http://www.homesoverseas.ru/pic/objects/7648.jpg&lt;/photo&gt;
&lt;photo&gt;http://www.homesoverseas.ru/pic/objects/7581.jpg&lt;/photo&gt; 

### &lt;ytid&gt; ###
&lt;ytid&gt; - YouTube id video.  Optional element.
Example: &lt;ytid&gt;oHDnTr5O28Q&lt;/ytid&gt;

### &lt;developer&gt; ###
&lt;developer&gt;- property from developer (only for sale).
Optional element. (Y/N)
Example: &lt;developer&gt;Y&lt;/developer&gt;


## 2. Characters and encoding ##

On default (if it isn’t obviously specified in the title) the file is encoded in utf-8. Otherwise it is necessary to set the encoding to xml file. The most commonly used encodings: windows-1251, utf-8, koi8-r
Note: actual encoding, that is given by the web-server, should ALWAYS be the same as the encoding, that is indicated in the title of XML.
The characters in the text &lt; &gt; & ' " should be changed to the appropriate elements:
& to &amp;
&lt; to &lt;
&gt; to &gt;
' to &apos;
" to &quot;
² to &sup2;

(here the semicolon is not the separator of the list but the compulsory part of the element!)

Changes should be made in all the elements &lt;object&gt; - in &lt;title&gt;, &lt;description&gt;,
&lt;annotation&gt;, &lt;photo&gt; etc.
For example, the link "http://some.host.ru/?id=1&page=10" becomes "http://some.host.ru/?id=1&amp;page=10".
If the RSS-file transfers in koi8-r, it is necessary to change in the text the character encodings windows-1251 to the equivalents from koi8-r:

dots character code 133
en-dash, character code 150
em-dash, character code 151
"Russian" number character code 185
Chevrons characters codes 171 and 187
"Smoothed" quotation marks (like this - ") character code 147 and 148
"Smoothed" apostrophes (like this - '): characters codes 145 and 146


## 3. Example of the export file ##

The example is available here: http://www.homesoverseas.ru/import/example_new.xml


## 4. Options ##

```text
Location
     7 first line to the sea/lake 
     8 city center 
     16 second line to the sea/lake 
     21 suburb 
View
     5 mountain view 
     6 sea/ocean/bay view 
     9 panoramic view 
     37 lake view 
     41 city view 
     42 park/garden view 
Plot features 
     43 water supply 
     44 electricity 
     45 gas
     46 road
     47 building permission 
Space planning and additional area
     23 open car park 
     25 balcony
     32 open space 
     33 garage
     34 separate entrance 
     35 terrace
     36 landscape gardening 
Facilities
     10 household appliances and electronics
     11 Jacuzzi
     12 fireplace/stove 
     13 furniture 
     14 under floor heating 
     15 smart home 
     17 heating 
     18 conditioning 
     31 satellite TV
     48 internet
     49 utility space
Leisure and Infrastructure 
     19 pool
     20 golf course 
     22 ski
     24 private beach
     26 marina
     27 fitness center
     28 spa/beauty salon 
     29 park
     30 playground
     50 tennis court
     51 supermarket
     52 school/nursery school
     53 medical centers 
     54 restaurants/cafe
Financing and management 
     55 mortgage 
     56 payment by installments 
     57 price is negotiable
     58 services of managing company 
     59 services for renting out
```
