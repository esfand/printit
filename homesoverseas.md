# HOMESOVERSEAS.RU XML FEED TECHNICAL REQUIREMENTS V2 #

Data export to be placed in the overseas property database of the website homesoverseas.ru 
is performed in XML format (http://www.w3.org/TR/REC-xml). Here is the description of the 
elements used for Data export, necessary comments and example of the export file.

## 1. Description of the elements used for Data export ##

### <root> ###
The root element of XML-file is <root>.
<root> may contain any number of elements <object>. 
Each <object> describes one object and must contain the following elements necessary for Data export:

### <objectid> ###
<objectid> - the object identifier in the database of the client (non-negative integer). Compulsory element. It is used for the subsequent updating of the object information. When changing to new requirements for xml take note of saving objectid in your objects. 

###  ###
<ref> - number of property for additional identification of listing. (60 symbols max). Not availible for visitors, only for clients
Example: <ref>AB123456-2014</ref>

###  ###
<title> - object name (up to 60 characters). Compulsory element.
Example: <title>House in Elsterwerda</title>

###  ###
< type> - type of advertisement. It can possess the values: 'sale' – advertisement for sale, 'rent' – advertisement for rent. Compulsory element.
Example: <type>sale</type>

###  ###
<market> - primary or secondary market. It can possess the values: 'primary' – primary market, 'secondary' – secondary market, 'mortgage' – mortgage market. Optional element.
Example: <market>primary</market>

###  ###
<annotation> - short object description (up to 150 characters). Optional element.
Example: <annotation>Residential and commercial real estate is located in Elsterwerda, in Brandenburg.</annotation>

###  ###
<description> - full object description. The text of this description should not contain text of the short description, as on the page of the object short and full descriptions are placed above each other. Optional element. Possible html-tags in container: <![CDATA[ òåêñò ]]>  Acceptable tags. <br> <strong> <b> <ul> <li> <i> <u> <sup>
 
Example: <description>Built in: 1900
Total area: 261
Condition: partial restoration was done 
Leased: Yes
Number of residential units: 5
Number of commercial units: 1
Current annual income: 15965
Full annual income: 17789
Current rental income: 8.87%
Full rental income: 9.88%</description>

###  ###
<price> - if the advertising type is «sale», the field specifies the price of the object (non-negative integer). If the field value is 0, the web-site will show «Price on request». Compulsory element.

Example: <price>180000</price>
If the advertising type is «rent», the field specifies the cost for the period (non-negative integer). At least one price must be indicated.
<price_rent_d> - Rent price per day 
<price_rent_w> - Rent price per week 
<price_rent_m> - Rent price per month  
<price_rent_y> - Rent price per year 

Example: 
<price_rent_d>100</price_rent_d>
<price_rent_w>600</price_rent_w>
<price_rent_m>2000</price_rent_m>
<price_rent_y>20000</price_rent_y>

###  ###
<currency> - currency of price Possible values: eur (Euro), usd (USA dollar), chf (Swiss Franc), gbp (UK pound), rur (Russian ruble). Compulsory element.
Example: <currency>eur</currency>  

###  ###
<region> - id of the region where the object is located (see the guide http://www.homesoverseas.ru/import/countries.php). Compulsory element.
Regions in CSV format
Id;name 
http://homesoverseas.ru/import/countries.csv.php
Id;name_eng;parentid
http://homesoverseas.ru/import/countries.parents.csv.php
Id;name_eng;parentid
http://homesoverseas.ru/import/countries.parents.eng.csv.php

Example: <region>42</region> 
 
###  ###
<realty_type> - type of the real estate. Compulsory element. It can possess the values:
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
Example: <realty_type>16</realty_type>

<bedrooms> - number of bedrooms (non-negative integer). Optional element.
Example: <bedrooms>5</bedrooms>

###  ###
<size_house> - living area in square meters. (only for apartments, villas, townhouses) Optional element.
Example: <size_house>5</size_house>

###  ###
<size_land> - size of a land plot in square meters. (only for selling townhouses, villas and plots) Optional element.
Example: <size_land>5</size_land>

<year> - year of construction (except land plots). Optional element.
Example: <year>1995</year>

###  ###
<not_ready_year> - year of completion of construction if year of construction is not indicated. Optional element.
Example: <not_ready_year>5</not_ready_year>

###  ###
<not_ready_quarter> - quarter of completion of construction if year of construction is not indicated. Optional element. (1,2,3,4)
Example: <not_ready_quarter>4</not_ready_quarter> 

###  ###
<level> - level (only for flats (apartments)). Optional element.
Example: <level>3</level>

###  ###
<levels> - number of floors in a building (except land plots). Optional element.
Example: <levels>5</levels>

###  ###
<distance_aero> - Distance to the airport in kilometers. Optional element.
Example: <distance_aero>120</distance_aero>

###  ###
<distance_ sea> - Distance to the sea in kilometers. Optional element. Subject to the thousandths decimal values.
Example: <distance_sea>120</distance_sea>

###  ###
<distance_ski> - Distance to the ski-lift in kilometers. Optional element. Subject to the thousandths decimal values
Example: <distance_ski>10</distance_ski>

###  ###
<distance_rus> - Distance to the boarder with Russia in kilometers. Only for Finland, Estonia, Latvia, Lithuania. Optional element.
Example: <distance_rus>80</distance_rus>

###  ###
<option> - id of the option (see Part 4 «Options»). Optional element. If there are several options for one object then the element <option> should be repeated.
Example:
<option>9</option> 
<option>18</option>
<option>15</option>


###  ###
<lat> - latitude
<lng> - longitude 
The coordinates of the object to link to the map are indicated in degrees. Decimal separator is marked by a dot. Optional element.
Example: <lat>56.298457922</lat><lng>-23.19283459</lng>

###  ###
<photo> - url of the file with the object photo. Optional element. For one object there can be up to 15 photos. In this case the element should be repeated. The minimal width of the picture must be 560 pixels.
Example:
<photo>http://www.homesoverseas.ru/pic/objects/7648.jpg</photo>
<photo>http://www.homesoverseas.ru/pic/objects/7581.jpg</photo> 

###  ###
<ytid> - YouTube id video.  Optional element.
Example: <ytid>oHDnTr5O28Q</ytid>

###  ###
<developer>- property from developer (only for sale).
Optional element. (Y/N)
Example: <developer>Y</developer>


## 2. Characters and encoding ##

On default (if it isn’t obviously specified in the title) the file is encoded in utf-8. 
Otherwise it is necessary to set the encoding to xml file. 
The most commonly used encodings: windows-1251, utf-8, koi8-r

Note: actual encoding, that is given by the web-server, should ALWAYS be the 
same as the encoding, that is indicated in the title of XML.

The characters in the text < > & ' " should be changed to the appropriate elements:

* & to &amp;
* < to &lt;
* > to &gt;
* ' to &apos;
* " to &quot;
* ² to &sup2;

(here the semicolon is not the separator of the list but the compulsory part of the element!)

Changes should be made in all the elements <object> - in <title>, <description>,
<annotation>, <photo> etc.

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
     
