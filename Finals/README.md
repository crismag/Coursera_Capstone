
# CAPSTONE PROJECT: BATTLE OF THE NEIGHBORHOODS
## Singapore Visitors and Expatriates Venue Recommendation

________
## I. PURPOSE
This document provides the details of my final peer reviewed assignment for the IBM Data Science Professional Certificate program  – Coursera Capstone.

________
## II. INTRODUCTION

Singapore is a small country and one of the most visited countries in Asia. There are a lot of websites where travelers can check and retrieve recommendations of places to stay or visit. However, most of these websites provides recommendation simply based on usual tourist attractions or key residential areas that are mostly expensive or already known for travelers based on certain keywords like "Hotel", or "Backpackers" etc. The intention on this project is to collect and provide a data driven recommendation that can supplement the recommendation with statistical data. This will also be utilizing data retrieved from Singapore open data sources and FourSquare API venue recommendations.

The sample recommender in this notebook will provide the following use case scenario:
* A person planning to visit Singapore as a Tourist or an Expat and looking for a reasonable accommodation.
* The user wants to receive venue recommendation where he can stay or rent an HDB apartment with close proximity to places of interest or search category option.
* The recommendation should not only present the most viable option, but also present a comparison table of all possible town venues.

For this demonstration, this notebook will make use of the following data:
* Singapore Median Rental Prices by town.
* Popular Food venues in the vicinity. (Sample category selection)

Note: While this demo makes use of Food Venue Category, Other possible categories can also be used for the same implementation such as checking categories like:
* Outdoors and Recreation
* Nightlife
* Nearby Schools, etc.

I will limit the scope of this search as FourSquare API only allows 50 free venue query limit per day when using a free user access.

________

## III. DATA ACQUISITION
This demonstration will make use of the following data sources:

#### Singapore Towns and median residential rental prices.
Data will retrieved from Singapore open dataset from <a href='https://data.gov.sg/dataset/b35046dc-7428-4cff-968d-ef4c3e9e6c99'>median rent by town and flattype</a> from https://data.gov.sg website. 

The original data source contains median rental prices of Singapore HDB units from 2005 up to 2nd quarter of 2018. I will retrieve rental the most recent recorded rental prices from this data source (Q2 2018) being the most relevant price available at this time. For this demonstration, I will simplify the analysis by using the average rental prices of all available flat type.

#### Singapore Towns location data retrieved using Google maps API.
Data coordinates of Town Venues will be retrieved using google API. I also make use of MRT stations coordinate as a more important center of for all towns included in venue recommendations.

#### Singapore Top Venue Recommendations from FourSquare API
(FourSquare website: www.foursquare.com)

I will be using the FourSquare API to explore neighborhoods in selected towns in Singapore. The Foursquare explore function will be used to get the most common venue categories in each neighborhood, and then use this feature to group the neighborhoods into clusters.  The following information are retrieved on the first query:
* Venue ID
* Venue Name
* Coordinates : Latitude and Longitude
* Category Name

Another venue query will be performed to retrieve venue ratings for each location. Note that rating information is a paid service from FourSquare and we are limited to only 50 queries per day. With this constraint, we limit the category analysis with only one type for this demo. I will try to retrieve as many ratings as possible for each retrieved venue ID.


________

## IV. METHODOLOGY

#### Singapore Towns List with median residential rental prices.
The source data contains median rental prices of Singapore HDB units from 2005 up to 2nd quarter of 2018. I will retrive the most recent recorded rental prices from this data source (Q2 2018) being the most relevant price available at this time. For this demonstration, I will simplify the analysis by using the average rental prices of all available flat type.

**Data Cleanup and re-grouping.** The retrieved table contains some un-wanted entries and needs some cleanup.

The following tasks will be performed:
* Drop/ignore cells with missing data.
* Use most current data record.
* Fix data types.

#### Post Processed Singapore towns list with and median residential rental prices

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town</th>
      <th>median_rent</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANG MO KIO</td>
      <td>2033.333333</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BEDOK</td>
      <td>2087.500000</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BISHAN</td>
      <td>2233.333333</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BUKIT BATOK</td>
      <td>1962.500000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BUKIT MERAH</td>
      <td>2162.500000</td>
    </tr>
    <tr>
      <th>5</th>
      <td>BUKIT PANJANG</td>
      <td>1737.500000</td>
    </tr>
    <tr>
      <th>6</th>
      <td>CENTRAL</td>
      <td>2450.000000</td>
    </tr>
    <tr>
      <th>7</th>
      <td>CHOA CHU KANG</td>
      <td>1933.333333</td>
    </tr>
    <tr>
      <th>8</th>
      <td>CLEMENTI</td>
      <td>2263.333333</td>
    </tr>
    <tr>
      <th>9</th>
      <td>GEYLANG</td>
      <td>2166.666667</td>
    </tr>
    <tr>
      <th>10</th>
      <td>HOUGANG</td>
      <td>1962.500000</td>
    </tr>
    <tr>
      <th>11</th>
      <td>JURONG EAST</td>
      <td>2150.000000</td>
    </tr>
    <tr>
      <th>12</th>
      <td>JURONG WEST</td>
      <td>1975.000000</td>
    </tr>
    <tr>
      <th>13</th>
      <td>KALLANG/WHAMPOA</td>
      <td>2300.000000</td>
    </tr>
    <tr>
      <th>14</th>
      <td>MARINE PARADE</td>
      <td>1950.000000</td>
    </tr>
    <tr>
      <th>15</th>
      <td>PASIR RIS</td>
      <td>2066.666667</td>
    </tr>
    <tr>
      <th>16</th>
      <td>PUNGGOL</td>
      <td>1825.000000</td>
    </tr>
    <tr>
      <th>17</th>
      <td>QUEENSTOWN</td>
      <td>2162.500000</td>
    </tr>
    <tr>
      <th>18</th>
      <td>SEMBAWANG</td>
      <td>1883.333333</td>
    </tr>
    <tr>
      <th>19</th>
      <td>SENGKANG</td>
      <td>1900.000000</td>
    </tr>
    <tr>
      <th>20</th>
      <td>SERANGOON</td>
      <td>2187.500000</td>
    </tr>
    <tr>
      <th>21</th>
      <td>TAMPINES</td>
      <td>2075.000000</td>
    </tr>
    <tr>
      <th>22</th>
      <td>TOA PAYOH</td>
      <td>2210.000000</td>
    </tr>
    <tr>
      <th>23</th>
      <td>WOODLANDS</td>
      <td>1762.500000</td>
    </tr>
    <tr>
      <th>24</th>
      <td>YISHUN</td>
      <td>1900.000000</td>
    </tr>
  </tbody>
</table>
</div>


* Adding geographical coordinates of each town location.


#### 2. Retrieve town coordinates.
Google apiwas be used to retrive the coordinates (latitude and longitude of each town centers. 
For this exercise, I just used the MRT stations as the center points of each evaluated towns.
The town coordinates will be used in retrieval of Foursquare API location data. 


```python
singapore_average_rental_prices_by_town['Latitude'] = 0.0
singapore_average_rental_prices_by_town['Longitude'] = 0.0

for idx,town in singapore_average_rental_prices_by_town['Town'].iteritems():
    address = town + " MRT station, Singapore" ; # I prefer to use MRT stations as more important central location of each town
    url = 'https://maps.googleapis.com/maps/api/geocode/json?address={}&key={}'.format(address,google_key)
    lat = requests.get(url).json()["results"][0]["geometry"]["location"]['lat']
    lng = requests.get(url).json()["results"][0]["geometry"]["location"]['lng']
    singapore_average_rental_prices_by_town.loc[idx,'Latitude'] = lat
    singapore_average_rental_prices_by_town.loc[idx,'Longitude'] = lng
```

#### Singapore Median Rental Prices per Town merged with Coordinate Data
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>median_rent</th>
      <th>Latitude</th>
      <th>Longitude</th>
    </tr>
    <tr>
      <th>Town</th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ANG MO KIO</th>
      <td>2033.333333</td>
      <td>1.369972</td>
      <td>103.849588</td>
    </tr>
    <tr>
      <th>BEDOK</th>
      <td>2087.500000</td>
      <td>1.324011</td>
      <td>103.930172</td>
    </tr>
    <tr>
      <th>BISHAN</th>
      <td>2233.333333</td>
      <td>1.351042</td>
      <td>103.849930</td>
    </tr>
    <tr>
      <th>BUKIT BATOK</th>
      <td>1962.500000</td>
      <td>1.348506</td>
      <td>103.749222</td>
    </tr>
    <tr>
      <th>BUKIT MERAH</th>
      <td>2162.500000</td>
      <td>1.289642</td>
      <td>103.816798</td>
    </tr>
    <tr>
      <th>BUKIT PANJANG</th>
      <td>1737.500000</td>
      <td>1.276068</td>
      <td>103.791904</td>
    </tr>
    <tr>
      <th>CENTRAL</th>
      <td>2450.000000</td>
      <td>1.288155</td>
      <td>103.846718</td>
    </tr>
    <tr>
      <th>CHOA CHU KANG</th>
      <td>1933.333333</td>
      <td>1.385385</td>
      <td>103.744337</td>
    </tr>
    <tr>
      <th>CLEMENTI</th>
      <td>2263.333333</td>
      <td>1.315070</td>
      <td>103.765246</td>
    </tr>
    <tr>
      <th>GEYLANG</th>
      <td>2166.666667</td>
      <td>1.316367</td>
      <td>103.882772</td>
    </tr>
    <tr>
      <th>HOUGANG</th>
      <td>1962.500000</td>
      <td>1.371331</td>
      <td>103.892544</td>
    </tr>
    <tr>
      <th>JURONG EAST</th>
      <td>2150.000000</td>
      <td>1.333143</td>
      <td>103.742329</td>
    </tr>
    <tr>
      <th>JURONG WEST</th>
      <td>1975.000000</td>
      <td>1.338556</td>
      <td>103.705828</td>
    </tr>
    <tr>
      <th>KALLANG/WHAMPOA</th>
      <td>2300.000000</td>
      <td>1.311478</td>
      <td>103.871351</td>
    </tr>
    <tr>
      <th>MARINE PARADE</th>
      <td>1950.000000</td>
      <td>1.308410</td>
      <td>103.888814</td>
    </tr>
    <tr>
      <th>PASIR RIS</th>
      <td>2066.666667</td>
      <td>1.373191</td>
      <td>103.949353</td>
    </tr>
    <tr>
      <th>PUNGGOL</th>
      <td>1825.000000</td>
      <td>1.405170</td>
      <td>103.902356</td>
    </tr>
    <tr>
      <th>QUEENSTOWN</th>
      <td>2162.500000</td>
      <td>1.294835</td>
      <td>103.805902</td>
    </tr>
    <tr>
      <th>SEMBAWANG</th>
      <td>1883.333333</td>
      <td>1.449080</td>
      <td>103.820058</td>
    </tr>
    <tr>
      <th>SENGKANG</th>
      <td>1900.000000</td>
      <td>1.391661</td>
      <td>103.895453</td>
    </tr>
    <tr>
      <th>SERANGOON</th>
      <td>2187.500000</td>
      <td>1.349787</td>
      <td>103.873635</td>
    </tr>
    <tr>
      <th>TAMPINES</th>
      <td>2075.000000</td>
      <td>1.354430</td>
      <td>103.942760</td>
    </tr>
    <tr>
      <th>TOA PAYOH</th>
      <td>2210.000000</td>
      <td>1.332330</td>
      <td>103.847425</td>
    </tr>
    <tr>
      <th>WOODLANDS</th>
      <td>1762.500000</td>
      <td>1.436945</td>
      <td>103.786516</td>
    </tr>
    <tr>
      <th>YISHUN</th>
      <td>1900.000000</td>
      <td>1.429548</td>
      <td>103.835033</td>
    </tr>
  </tbody>
</table>
</div>


________

## V. Segmenting and Clustering Towns in Singapore
### Retrieving FourSquare Places of interest.
Using the Foursquare API, the **explore** API function was be used to get the most common venue categories in each neighborhood, and then used this feature to group the neighborhoods into clusters. The *k*-means clustering algorithm was used for the analysis.
Fnally, the Folium library is used to visualize the recommended neighborhoods and their emerging clusters.

In the ipynb notebook, the function **getNearbyVenues** extracts the following information for the dataframe it generates:
* Venue ID
* Venue Name
* Coordinates : Latitude and Longitude
* Category Name

The function **getVenuesByCategory** performs the following:
  1. **category** based venue search to simulate user venue searches based on certain places of interest. This search extracts the following information:
   * Venue ID
   * Venue Name
   * Coordinates : Latitude and Longitude
   * Category Name
  2. For each retrieved **venueID**, retrive the venues category rating.

The generated data frame in the second function contains the following column:
<TABLE align='left'>
    <tr>
        <th>Column Name</th><th>Description</th>
    </tr>
<tr><td>Town</td><td>Town Name</td></tr>
<tr><td>Town Latitude</td><td>Towns MRT station Latitude</td></tr>
<tr><td>Town Longitude</td><td>Town MRT station Latitude</td></tr>
<tr><td>VenueID</td><td>FourSquare Venue ID</td></tr>
<tr><td>VenueName</td><td>Venue Name</td></tr>
<tr><td>score</td><td>FourSquare Venue user rating</td></tr>
<tr><td>category</td><td>Category group name</td></tr>
<tr><td>catID</td><td>Category ID</td></tr>
<tr><td>latitude</td><td>Venue Location - latitude</td></tr>
<tr><td>longitude</td><td>Venue Location - longitude</td></tr>


#### Search Venues with recommendations on  : Food Venues (Restaurants,Fastfoods, etc.)

To demonstrate user selection of places of interest, We will use this Food Venues category in our further analysis.
* This Foursquare search is expected to collect venues in the following category:
 * category
 * Food Courts
 * Coffee Shops
 * Restaurants
 * Cafés
 * Other food venues

List of my FourSquare Data collection saved in Github can be found in the following location:
 * https://github.com/crismag/Coursera_Capstone/tree/master/saved_data

I used the FourSquare API to retrieve venue scores of locations. Note that there is max query limit of 50 in FourSquare API for free subscription. So use or query carefully.

#### Data cleanup uneeded entries
* Eliminate possible venue duplicates.
* Improve the quality of our venue selection by removing venues with no ratings or 0.0


<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town</th>
      <th>Town Latitude</th>
      <th>Town Longitude</th>
      <th>VenueID</th>
      <th>VenueName</th>
      <th>score</th>
      <th>category</th>
      <th>catID</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>ANG MO KIO</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>50138eaee4b05d9dc80ae5b0</td>
      <td>Hong Kong Sheng Kee Dessert ??????</td>
      <td>5.8</td>
      <td>Dessert Shops</td>
      <td>4bf58dd8d48988d1d0941735</td>
      <td>1.369473</td>
      <td>103.849241</td>
    </tr>
    <tr>
      <th>4</th>
      <td>ANG MO KIO</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>5be2d3831af8520039a38da2</td>
      <td>Malaysia Boleh!</td>
      <td>7.5</td>
      <td>Food Courts</td>
      <td>4bf58dd8d48988d120951735</td>
      <td>1.369669</td>
      <td>103.848900</td>
    </tr>
    <tr>
      <th>5</th>
      <td>ANG MO KIO</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>4b7cdf36f964a520fda72fe3</td>
      <td>BreadTalk / Toast Box</td>
      <td>5.5</td>
      <td>Breakfast Spots</td>
      <td>4bf58dd8d48988d143941735</td>
      <td>1.369177</td>
      <td>103.848874</td>
    </tr>
    <tr>
      <th>7</th>
      <td>ANG MO KIO</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>4b1e9fc6f964a520d21c24e3</td>
      <td>Ichiban Sushi</td>
      <td>5.6</td>
      <td>Sushi Restaurants</td>
      <td>4bf58dd8d48988d1d2941735</td>
      <td>1.369156</td>
      <td>103.849109</td>
    </tr>
    <tr>
      <th>10</th>
      <td>ANG MO KIO</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>5340222411d247b11bb27bb8</td>
      <td>Eighteen Chefs</td>
      <td>5.5</td>
      <td>Diners</td>
      <td>4bf58dd8d48988d147941735</td>
      <td>1.369265</td>
      <td>103.848706</td>
    </tr>
  </tbody>
</table>
</div>



#### Results: Venue Count Per Town

<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town Latitude</th>
      <th>Town Longitude</th>
      <th>VenueID</th>
      <th>VenueName</th>
      <th>score</th>
      <th>category</th>
      <th>catID</th>
      <th>latitude</th>
      <th>longitude</th>
    </tr>
    <tr>
      <th>Town</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ANG MO KIO</th>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
    </tr>
    <tr>
      <th>BEDOK</th>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
      <td>29</td>
    </tr>
    <tr>
      <th>BISHAN</th>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
      <td>36</td>
    </tr>
    <tr>
      <th>BUKIT BATOK</th>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
      <td>22</td>
    </tr>
    <tr>
      <th>BUKIT MERAH</th>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
      <td>9</td>
    </tr>
    <tr>
      <th>BUKIT PANJANG</th>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
    </tr>
    <tr>
      <th>CENTRAL</th>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
      <td>46</td>
    </tr>
    <tr>
      <th>CHOA CHU KANG</th>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
      <td>27</td>
    </tr>
    <tr>
      <th>CLEMENTI</th>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
    </tr>
    <tr>
      <th>GEYLANG</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
    </tr>
    <tr>
      <th>HOUGANG</th>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
      <td>26</td>
    </tr>
    <tr>
      <th>JURONG EAST</th>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
      <td>39</td>
    </tr>
    <tr>
      <th>JURONG WEST</th>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
    </tr>
    <tr>
      <th>KALLANG/WHAMPOA</th>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
      <td>15</td>
    </tr>
    <tr>
      <th>MARINE PARADE</th>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
      <td>21</td>
    </tr>
    <tr>
      <th>PASIR RIS</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
    </tr>
    <tr>
      <th>PUNGGOL</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
    </tr>
    <tr>
      <th>QUEENSTOWN</th>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
      <td>8</td>
    </tr>
    <tr>
      <th>SEMBAWANG</th>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
    </tr>
    <tr>
      <th>SENGKANG</th>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
      <td>17</td>
    </tr>
    <tr>
      <th>SERANGOON</th>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
      <td>42</td>
    </tr>
    <tr>
      <th>TAMPINES</th>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
      <td>25</td>
    </tr>
    <tr>
      <th>TOA PAYOH</th>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
      <td>34</td>
    </tr>
    <tr>
      <th>WOODLANDS</th>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
      <td>31</td>
    </tr>
    <tr>
      <th>YISHUN</th>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
      <td>18</td>
    </tr>
  </tbody>
</table>
</div>

#### RESULTS: How many unique categories can be curated from all the returned venues?
    * There are 67 uniques categories.


#### What are the top 20 most common venue types?

<img src = "https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/top10_reco_venue_food_category.png" width = 800, align = "center">
<pre>
    category
    Food Courts              92
    Coffee Shops             62
    Fast Food Restaurants    61
    Chinese Restaurants      58
    Cafés                    35
    Japanese Restaurants     34
    Asian Restaurants        22
    Noodle Houses            20
    Thai Restaurants         19
    Sushi Restaurants        16
    Seafood Restaurants      15
    Italian Restaurants      13
    Sandwich Places          13
    Indian Restaurants       11
    Bubble Tea Shops         11
    American Restaurants      8
    Burger Joints             8
    Dim Sum Restaurants       8
    Dessert Shops             8
    Fried Chicken Joints      7
</pre>
#### What are the top 20 venues given with highest score rating?

<img src = "https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/top_town_venue_reccount.png" width = 800, align = "center">

<pre>
    Town           category            
    MARINE PARADE  Food Courts             8.70
    CENTRAL        Asian Restaurants       8.50
                   Seafood Restaurants     8.35
    YISHUN         Noodle Houses           8.30
    CENTRAL        Udon Restaurants        8.30
    GEYLANG        BBQ Joints              8.30
    CENTRAL        Breweries               8.20
                   Dumpling Restaurants    8.10
    YISHUN         Bubble Tea Shops        8.10
                   Chinese Restaurants     8.00
    BEDOK          Chinese Restaurants     8.00
    JURONG WEST    Wings Joints            8.00
    CENTRAL        Bubble Tea Shops        8.00
    HOUGANG        Thai Restaurants        7.90
    JURONG EAST    Hotpot Restaurants      7.90
    BUKIT PANJANG  Cafés                   7.90
    CENTRAL        Bistros                 7.90
    BUKIT PANJANG  Thai Restaurants        7.85
    GEYLANG        Cafés                   7.80
    TAMPINES       Fried Chicken Joints    7.80
</pre>



## Analyze Each Singapore Town nearby recommended venues
* Technique : **One Hot Encoding**

```python
# one hot encoding
sg_onehot = pd.get_dummies(singapore_town_venues[['category']], prefix="", prefix_sep="")

# add Town column back to dataframe
sg_onehot['Town'] = singapore_town_venues['Town'] 

# move neighborhood column to the first column
fixed_columns = [sg_onehot.columns[-1]] + list(sg_onehot.columns[:-1])
sg_onehot = sg_onehot[fixed_columns]

# Check returned one hot encoding data:
print('One hot encoding returned "{}" rows.'.format(sg_onehot.shape[0]))

# Regroup rows by town and mean of frequency occurrence per category.
sg_grouped = sg_onehot.groupby('Town').mean().reset_index()

print('One hot encoding re-group returned "{}" rows.'.format(sg_grouped.shape[0]))
sg_grouped.head()
```

    One hot encoding returned "644" rows.
    One hot encoding re-group returned "25" rows.





<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town</th>
      <th>American Restaurants</th>
      <th>Asian Restaurants</th>
      <th>BBQ Joints</th>
      <th>Bakeries</th>
      <th>Bars</th>
      <th>Bistros</th>
      <th>Breakfast Spots</th>
      <th>Breweries</th>
      <th>Bubble Tea Shops</th>
      <th>Buffets</th>
      <th>Burger Joints</th>
      <th>Cafeterias</th>
      <th>Cafés</th>
      <th>Cantonese Restaurants</th>
      <th>Cha Chaan Tengs</th>
      <th>Chinese Breakfast Places</th>
      <th>Chinese Restaurants</th>
      <th>Coffee Shops</th>
      <th>Comfort Food Restaurants</th>
      <th>Dessert Shops</th>
      <th>Dim Sum Restaurants</th>
      <th>Diners</th>
      <th>Dongbei Restaurants</th>
      <th>Dumpling Restaurants</th>
      <th>English Restaurants</th>
      <th>Fast Food Restaurants</th>
      <th>Fish &amp; Chips Shops</th>
      <th>Food Courts</th>
      <th>French Restaurants</th>
      <th>Fried Chicken Joints</th>
      <th>Frozen Yogurt Shops</th>
      <th>Gastropubs</th>
      <th>Grocery Stores</th>
      <th>Hainan Restaurants</th>
      <th>Halal Restaurants</th>
      <th>Hong Kong Restaurants</th>
      <th>Hotpot Restaurants</th>
      <th>Ice Cream Shops</th>
      <th>Indian Restaurants</th>
      <th>Indonesian Restaurants</th>
      <th>Italian Restaurants</th>
      <th>Japanese Curry Restaurants</th>
      <th>Japanese Restaurants</th>
      <th>Korean Restaurants</th>
      <th>Macanese Restaurants</th>
      <th>Malay Restaurants</th>
      <th>Mexican Restaurants</th>
      <th>Miscellaneous Shops</th>
      <th>Modern European Restaurants</th>
      <th>Noodle Houses</th>
      <th>Pizza Places</th>
      <th>Portuguese Restaurants</th>
      <th>Ramen Restaurants</th>
      <th>Restaurants</th>
      <th>Sandwich Places</th>
      <th>Seafood Restaurants</th>
      <th>Shopping Malls</th>
      <th>Snack Places</th>
      <th>Soup Places</th>
      <th>Steakhouses</th>
      <th>Sushi Restaurants</th>
      <th>Taiwanese Restaurants</th>
      <th>Thai Restaurants</th>
      <th>Udon Restaurants</th>
      <th>Vegetarian / Vegan Restaurants</th>
      <th>Vietnamese Restaurants</th>
      <th>Wings Joints</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANG MO KIO</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.058824</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.058824</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.117647</td>
      <td>0.0</td>
      <td>0.176471</td>
      <td>0.000000</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.029412</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.058824</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.029412</td>
      <td>0.029412</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.029412</td>
      <td>0.029412</td>
      <td>0.029412</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.029412</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.058824</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BEDOK</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.034483</td>
      <td>0.0</td>
      <td>0.034483</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.034483</td>
      <td>0.206897</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.034483</td>
      <td>0.0</td>
      <td>0.068966</td>
      <td>0.0</td>
      <td>0.103448</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.068966</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.034483</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.068966</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.034483</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BISHAN</td>
      <td>0.027778</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.055556</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.083333</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.111111</td>
      <td>0.138889</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.027778</td>
      <td>0.0</td>
      <td>0.083333</td>
      <td>0.0</td>
      <td>0.083333</td>
      <td>0.027778</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.027778</td>
      <td>0.027778</td>
      <td>0.000000</td>
      <td>0.027778</td>
      <td>0.0</td>
      <td>0.111111</td>
      <td>0.027778</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.027778</td>
      <td>0.027778</td>
      <td>0.000000</td>
      <td>0.027778</td>
      <td>0.027778</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.027778</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BUKIT BATOK</td>
      <td>0.000000</td>
      <td>0.045455</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.136364</td>
      <td>0.136364</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.136364</td>
      <td>0.0</td>
      <td>0.272727</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.045455</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.045455</td>
      <td>0.0</td>
      <td>0.045455</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.045455</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.045455</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.045455</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BUKIT MERAH</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.111111</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.111111</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.444444</td>
      <td>0.222222</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.111111</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.000000</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>0.000000</td>
    </tr>
  </tbody>
</table>
</div>



#### Analysis of Singapore Town most visited venues
________
    # Town=< ANG MO KIO >
                       venue  freq
    0            Food Courts  0.18
    1  Fast Food Restaurants  0.12
    2   Japanese Restaurants  0.06
    3          Dessert Shops  0.06
    4      Sushi Restaurants  0.06
    5                  Cafés  0.06
    6           Snack Places  0.03
    7          Noodle Houses  0.03
    8      Ramen Restaurants  0.03
    9            Restaurants  0.03
    
________    
    # Town=< BEDOK >
                        venue  freq
    0            Coffee Shops  0.21
    1             Food Courts  0.10
    2       Sushi Restaurants  0.07
    3    Japanese Restaurants  0.07
    4   Fast Food Restaurants  0.07
    5    American Restaurants  0.03
    6     Chinese Restaurants  0.03
    7         Sandwich Places  0.03
    8  Indonesian Restaurants  0.03
    9      Indian Restaurants  0.03
    
________
    # Town=< BISHAN >
                       venue  freq
    0           Coffee Shops  0.14
    1   Japanese Restaurants  0.11
    2    Chinese Restaurants  0.11
    3            Food Courts  0.08
    4  Fast Food Restaurants  0.08
    5                  Cafés  0.08
    6       Bubble Tea Shops  0.06
    7   American Restaurants  0.03
    8        Sandwich Places  0.03
    9    Italian Restaurants  0.03
    
________    
    # Town=< BUKIT BATOK >
                       venue  freq
    0            Food Courts  0.27
    1           Coffee Shops  0.14
    2    Chinese Restaurants  0.14
    3  Fast Food Restaurants  0.14
    4   Japanese Restaurants  0.05
    5        Sandwich Places  0.05
    6        Ice Cream Shops  0.05
    7    Italian Restaurants  0.05
    8       Thai Restaurants  0.05
    9      Asian Restaurants  0.05
    
________    
    # Town=< BUKIT MERAH >
                             venue  freq
    0          Chinese Restaurants  0.44
    1                 Coffee Shops  0.22
    2                  Food Courts  0.11
    3                      Bistros  0.11
    4                        Cafés  0.11
    5            Malay Restaurants  0.00
    6                Noodle Houses  0.00
    7  Modern European Restaurants  0.00
    8          Miscellaneous Shops  0.00
    9          Mexican Restaurants  0.00
    
  ________  
    # Town=< BUKIT PANJANG >
                        venue  freq
    0     Chinese Restaurants  0.20
    1      Indian Restaurants  0.13
    2        Thai Restaurants  0.13
    3             Food Courts  0.13
    4           Burger Joints  0.07
    5     Seafood Restaurants  0.07
    6  Vietnamese Restaurants  0.07
    7       Asian Restaurants  0.07
    8           Noodle Houses  0.07
    9                   Cafés  0.07
    
________
    # Town=< CENTRAL >
                       venue  freq
    0                  Cafés  0.09
    1    Chinese Restaurants  0.09
    2           Coffee Shops  0.07
    3      Ramen Restaurants  0.07
    4            Food Courts  0.07
    5                 Diners  0.04
    6   Japanese Restaurants  0.04
    7     Hotpot Restaurants  0.04
    8          Noodle Houses  0.04
    9  Fast Food Restaurants  0.04
    
________
    # Town=< CHOA CHU KANG >
                        venue  freq
    0   Fast Food Restaurants  0.15
    1             Food Courts  0.11
    2            Coffee Shops  0.11
    3           Noodle Houses  0.11
    4       Asian Restaurants  0.07
    5           Dessert Shops  0.04
    6           Burger Joints  0.04
    7  Portuguese Restaurants  0.04
    8         Sandwich Places  0.04
    9                   Cafés  0.04
    
________
    # Town=< CLEMENTI >
                       venue  freq
    0            Food Courts  0.24
    1  Fast Food Restaurants  0.09
    2           Coffee Shops  0.09
    3    Dim Sum Restaurants  0.06
    4   Japanese Restaurants  0.06
    5       Thai Restaurants  0.06
    6      Asian Restaurants  0.06
    7    Chinese Restaurants  0.06
    8   Fried Chicken Joints  0.06
    9                  Cafés  0.03
    
________
    # Town=< GEYLANG >
                                venue  freq
    0             Chinese Restaurants  0.24
    1             Dim Sum Restaurants  0.12
    2                     Food Courts  0.08
    3  Vegetarian / Vegan Restaurants  0.08
    4                    Coffee Shops  0.08
    5                   Noodle Houses  0.08
    6             Seafood Restaurants  0.08
    7                      BBQ Joints  0.04
    8           Fast Food Restaurants  0.04
    9               Asian Restaurants  0.04
    
________
    # Town=< HOUGANG >
                       venue  freq
    0            Food Courts  0.19
    1  Fast Food Restaurants  0.15
    2           Coffee Shops  0.12
    3      Asian Restaurants  0.08
    4    Chinese Restaurants  0.08
    5          Dessert Shops  0.04
    6                  Cafés  0.04
    7     Fish & Chips Shops  0.04
    8        Sandwich Places  0.04
    9         Shopping Malls  0.04
    
________
    # Town=< JURONG EAST >
                          venue  freq
    0               Food Courts  0.13
    1      Japanese Restaurants  0.13
    2       Chinese Restaurants  0.10
    3     Fast Food Restaurants  0.08
    4                     Cafés  0.08
    5              Coffee Shops  0.05
    6                    Diners  0.03
    7  Chinese Breakfast Places  0.03
    8         Ramen Restaurants  0.03
    9             Noodle Houses  0.03
    
________
    # Town=< JURONG WEST >
                        venue  freq
    0   Fast Food Restaurants  0.16
    1     Chinese Restaurants  0.13
    2       Asian Restaurants  0.10
    3    Japanese Restaurants  0.10
    4             Food Courts  0.10
    5                   Cafés  0.06
    6            Wings Joints  0.03
    7         Sandwich Places  0.03
    8   Hong Kong Restaurants  0.03
    9  Indonesian Restaurants  0.03
    
________
    # Town=< KALLANG/WHAMPOA >
                     venue  freq
    0          Food Courts  0.20
    1        Noodle Houses  0.13
    2         Snack Places  0.13
    3           BBQ Joints  0.13
    4  Chinese Restaurants  0.13
    5  Seafood Restaurants  0.07
    6          Soup Places  0.07
    7     Thai Restaurants  0.07
    8   Indian Restaurants  0.07
    9  Italian Restaurants  0.00
    
________
    # Town=< MARINE PARADE >
                     venue  freq
    0        Noodle Houses  0.29
    1  Chinese Restaurants  0.14
    2  Seafood Restaurants  0.10
    3    Asian Restaurants  0.10
    4           BBQ Joints  0.05
    5                 Bars  0.05
    6  Dim Sum Restaurants  0.05
    7     Thai Restaurants  0.05
    8        Dessert Shops  0.05
    9         Coffee Shops  0.05
    
________
    # Town=< PASIR RIS >
                       venue  freq
    0  Fast Food Restaurants  0.18
    1            Food Courts  0.18
    2           Coffee Shops  0.12
    3        Sandwich Places  0.06
    4  Hong Kong Restaurants  0.06
    5               Bakeries  0.06
    6      Asian Restaurants  0.06
    7    Italian Restaurants  0.06
    8      Sushi Restaurants  0.06
    9            Restaurants  0.06
    
________
    # Town=< PUNGGOL >
                          venue  freq
    0               Food Courts  0.16
    1       Seafood Restaurants  0.08
    2     Fast Food Restaurants  0.08
    3                     Cafés  0.08
    4               Soup Places  0.04
    5         Ramen Restaurants  0.04
    6           Sandwich Places  0.04
    7  Comfort Food Restaurants  0.04
    8              Coffee Shops  0.04
    9       Chinese Restaurants  0.04
    
________
    # Town=< QUEENSTOWN >
                             venue  freq
    0          Chinese Restaurants  0.38
    1                  Food Courts  0.25
    2             Thai Restaurants  0.12
    3          Italian Restaurants  0.12
    4            Malay Restaurants  0.12
    5         American Restaurants  0.00
    6         Macanese Restaurants  0.00
    7  Modern European Restaurants  0.00
    8          Miscellaneous Shops  0.00
    9          Mexican Restaurants  0.00
    
________
    # Town=< SEMBAWANG >
                       venue  freq
    0            Food Courts  0.28
    1           Coffee Shops  0.17
    2      Asian Restaurants  0.11
    3  Fast Food Restaurants  0.11
    4    Italian Restaurants  0.06
    5                Bistros  0.06
    6   Japanese Restaurants  0.06
    7      Sushi Restaurants  0.06
    8    Chinese Restaurants  0.06
    9                  Cafés  0.06
    
________
    # Town=< SENGKANG >
                       venue  freq
    0            Food Courts  0.18
    1           Coffee Shops  0.18
    2  Fast Food Restaurants  0.18
    3                  Cafés  0.18
    4    English Restaurants  0.06
    5    Chinese Restaurants  0.06
    6            Restaurants  0.06
    7      Sushi Restaurants  0.06
    8        Sandwich Places  0.06
    9    Mexican Restaurants  0.00
    
________
    # Town=< SERANGOON >
                       venue  freq
    0  Fast Food Restaurants  0.14
    1           Coffee Shops  0.10
    2   Japanese Restaurants  0.10
    3     Indian Restaurants  0.05
    4                  Cafés  0.05
    5            Steakhouses  0.05
    6      Sushi Restaurants  0.05
    7    Chinese Restaurants  0.05
    8            Food Courts  0.05
    9       Thai Restaurants  0.05
    
________
    # Town=< TAMPINES >
                       venue  freq
    0            Food Courts  0.20
    1           Coffee Shops  0.16
    2  Fast Food Restaurants  0.12
    3    Italian Restaurants  0.08
    4   Japanese Restaurants  0.08
    5   American Restaurants  0.04
    6    Seafood Restaurants  0.04
    7   Fried Chicken Joints  0.04
    8   Dumpling Restaurants  0.04
    9           Pizza Places  0.04
    
________
    # Town=< TOA PAYOH >
                       venue  freq
    0           Coffee Shops  0.18
    1  Fast Food Restaurants  0.12
    2            Food Courts  0.12
    3    Chinese Restaurants  0.12
    4           Snack Places  0.06
    5                  Cafés  0.06
    6          Burger Joints  0.03
    7        Sandwich Places  0.03
    8    Italian Restaurants  0.03
    9      Asian Restaurants  0.03
    
________
    # Town=< WOODLANDS >
                       venue  freq
    0            Food Courts  0.16
    1   Japanese Restaurants  0.13
    2                  Cafés  0.10
    3  Fast Food Restaurants  0.10
    4           Coffee Shops  0.10
    5    Chinese Restaurants  0.10
    6   American Restaurants  0.06
    7    Italian Restaurants  0.03
    8   Fried Chicken Joints  0.03
    9           Pizza Places  0.03
    
________
    # Town=< YISHUN >
                       venue  freq
    0            Food Courts  0.28
    1     Hainan Restaurants  0.06
    2  Hong Kong Restaurants  0.06
    3      Halal Restaurants  0.06
    4       Thai Restaurants  0.06
    5  Fast Food Restaurants  0.06
    6      Sushi Restaurants  0.06
    7       Bubble Tea Shops  0.06
    8     Fish & Chips Shops  0.06
    9          Burger Joints  0.06
________

#### RESULTS: Categorized Result
<div>
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANG MO KIO</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sushi Restaurants</td>
      <td>Cafés</td>
      <td>Ramen Restaurants</td>
      <td>Hong Kong Restaurants</td>
      <td>Halal Restaurants</td>
      <td>Miscellaneous Shops</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BEDOK</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Sushi Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Wings Joints</td>
      <td>Fried Chicken Joints</td>
      <td>Indian Restaurants</td>
      <td>Ice Cream Shops</td>
      <td>Hotpot Restaurants</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BISHAN</td>
      <td>Coffee Shops</td>
      <td>Japanese Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Food Courts</td>
      <td>Cafés</td>
      <td>Bubble Tea Shops</td>
      <td>American Restaurants</td>
      <td>Portuguese Restaurants</td>
      <td>Dumpling Restaurants</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BUKIT BATOK</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Pizza Places</td>
      <td>Ice Cream Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sandwich Places</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BUKIT MERAH</td>
      <td>Chinese Restaurants</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Bistros</td>
      <td>Cafés</td>
      <td>Dongbei Restaurants</td>
      <td>Comfort Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Dim Sum Restaurants</td>
      <td>Diners</td>
    </tr>
  </tbody>
</table>
</div>


## RESULTS : *k*-means Cluster Results 
#### Clustered results for *k*-means to cluster with 5 clusters.

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Town</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>ANG MO KIO</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sushi Restaurants</td>
      <td>Cafés</td>
      <td>Ramen Restaurants</td>
      <td>Hong Kong Restaurants</td>
      <td>Halal Restaurants</td>
      <td>Miscellaneous Shops</td>
    </tr>
    <tr>
      <th>1</th>
      <td>BEDOK</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Sushi Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Wings Joints</td>
      <td>Fried Chicken Joints</td>
      <td>Indian Restaurants</td>
      <td>Ice Cream Shops</td>
      <td>Hotpot Restaurants</td>
    </tr>
    <tr>
      <th>2</th>
      <td>BISHAN</td>
      <td>Coffee Shops</td>
      <td>Japanese Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Food Courts</td>
      <td>Cafés</td>
      <td>Bubble Tea Shops</td>
      <td>American Restaurants</td>
      <td>Portuguese Restaurants</td>
      <td>Dumpling Restaurants</td>
    </tr>
    <tr>
      <th>3</th>
      <td>BUKIT BATOK</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Pizza Places</td>
      <td>Ice Cream Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sandwich Places</td>
    </tr>
    <tr>
      <th>4</th>
      <td>BUKIT MERAH</td>
      <td>Chinese Restaurants</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Bistros</td>
      <td>Cafés</td>
      <td>Dongbei Restaurants</td>
      <td>Comfort Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Dim Sum Restaurants</td>
      <td>Diners</td>
    </tr>
  </tbody>
</table>
</div>

#### RESULTS: Merged Cluster Table with rental prices.
<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>median_rent</th>
      <th>Latitude</th>
      <th>Longitude</th>
      <th>Cluster Labels</th>
      <th>1st Most Common Venue</th>
      <th>2nd Most Common Venue</th>
      <th>3rd Most Common Venue</th>
      <th>4th Most Common Venue</th>
      <th>5th Most Common Venue</th>
      <th>6th Most Common Venue</th>
      <th>7th Most Common Venue</th>
      <th>8th Most Common Venue</th>
      <th>9th Most Common Venue</th>
      <th>10th Most Common Venue</th>
    </tr>
    <tr>
      <th>Town</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>ANG MO KIO</th>
      <td>2033.333333</td>
      <td>1.369972</td>
      <td>103.849588</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sushi Restaurants</td>
      <td>Cafés</td>
      <td>Ramen Restaurants</td>
      <td>Hong Kong Restaurants</td>
      <td>Halal Restaurants</td>
      <td>Miscellaneous Shops</td>
    </tr>
    <tr>
      <th>BEDOK</th>
      <td>2087.500000</td>
      <td>1.324011</td>
      <td>103.930172</td>
      <td>3</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Sushi Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Wings Joints</td>
      <td>Fried Chicken Joints</td>
      <td>Indian Restaurants</td>
      <td>Ice Cream Shops</td>
      <td>Hotpot Restaurants</td>
    </tr>
    <tr>
      <th>BISHAN</th>
      <td>2233.333333</td>
      <td>1.351042</td>
      <td>103.849930</td>
      <td>3</td>
      <td>Coffee Shops</td>
      <td>Japanese Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Food Courts</td>
      <td>Cafés</td>
      <td>Bubble Tea Shops</td>
      <td>American Restaurants</td>
      <td>Portuguese Restaurants</td>
      <td>Dumpling Restaurants</td>
    </tr>
    <tr>
      <th>BUKIT BATOK</th>
      <td>1962.500000</td>
      <td>1.348506</td>
      <td>103.749222</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Pizza Places</td>
      <td>Ice Cream Shops</td>
      <td>Japanese Restaurants</td>
      <td>Sandwich Places</td>
    </tr>
    <tr>
      <th>BUKIT MERAH</th>
      <td>2162.500000</td>
      <td>1.289642</td>
      <td>103.816798</td>
      <td>2</td>
      <td>Chinese Restaurants</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Bistros</td>
      <td>Cafés</td>
      <td>Dongbei Restaurants</td>
      <td>Comfort Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Dim Sum Restaurants</td>
      <td>Diners</td>
    </tr>
    <tr>
      <th>BUKIT PANJANG</th>
      <td>1737.500000</td>
      <td>1.276068</td>
      <td>103.791904</td>
      <td>4</td>
      <td>Chinese Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Food Courts</td>
      <td>Indian Restaurants</td>
      <td>Cafés</td>
      <td>Asian Restaurants</td>
      <td>Noodle Houses</td>
      <td>Seafood Restaurants</td>
      <td>Vietnamese Restaurants</td>
      <td>Burger Joints</td>
    </tr>
    <tr>
      <th>CENTRAL</th>
      <td>2450.000000</td>
      <td>1.288155</td>
      <td>103.846718</td>
      <td>3</td>
      <td>Cafés</td>
      <td>Chinese Restaurants</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Ramen Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Noodle Houses</td>
      <td>Diners</td>
      <td>Hotpot Restaurants</td>
    </tr>
    <tr>
      <th>CHOA CHU KANG</th>
      <td>1933.333333</td>
      <td>1.385385</td>
      <td>103.744337</td>
      <td>1</td>
      <td>Fast Food Restaurants</td>
      <td>Food Courts</td>
      <td>Noodle Houses</td>
      <td>Coffee Shops</td>
      <td>Asian Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Bakeries</td>
      <td>Italian Restaurants</td>
      <td>Dessert Shops</td>
      <td>Cafés</td>
    </tr>
    <tr>
      <th>CLEMENTI</th>
      <td>2263.333333</td>
      <td>1.315070</td>
      <td>103.765246</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Coffee Shops</td>
      <td>Fried Chicken Joints</td>
      <td>Asian Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Dim Sum Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Cafés</td>
    </tr>
    <tr>
      <th>GEYLANG</th>
      <td>2166.666667</td>
      <td>1.316367</td>
      <td>103.882772</td>
      <td>4</td>
      <td>Chinese Restaurants</td>
      <td>Dim Sum Restaurants</td>
      <td>Food Courts</td>
      <td>Noodle Houses</td>
      <td>Coffee Shops</td>
      <td>Seafood Restaurants</td>
      <td>Vegetarian / Vegan Restaurants</td>
      <td>Steakhouses</td>
      <td>Fast Food Restaurants</td>
      <td>BBQ Joints</td>
    </tr>
    <tr>
      <th>HOUGANG</th>
      <td>1962.500000</td>
      <td>1.371331</td>
      <td>103.892544</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Coffee Shops</td>
      <td>Chinese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Dessert Shops</td>
      <td>Frozen Yogurt Shops</td>
      <td>Pizza Places</td>
      <td>Bubble Tea Shops</td>
      <td>Sandwich Places</td>
    </tr>
    <tr>
      <th>JURONG EAST</th>
      <td>2150.000000</td>
      <td>1.333143</td>
      <td>103.742329</td>
      <td>3</td>
      <td>Food Courts</td>
      <td>Japanese Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Cafés</td>
      <td>Coffee Shops</td>
      <td>Ramen Restaurants</td>
      <td>Dim Sum Restaurants</td>
      <td>Chinese Breakfast Places</td>
      <td>Diners</td>
    </tr>
    <tr>
      <th>JURONG WEST</th>
      <td>1975.000000</td>
      <td>1.338556</td>
      <td>103.705828</td>
      <td>3</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Food Courts</td>
      <td>Cafés</td>
      <td>Wings Joints</td>
      <td>Japanese Curry Restaurants</td>
      <td>Indonesian Restaurants</td>
      <td>Pizza Places</td>
    </tr>
    <tr>
      <th>KALLANG/WHAMPOA</th>
      <td>2300.000000</td>
      <td>1.311478</td>
      <td>103.871351</td>
      <td>0</td>
      <td>Food Courts</td>
      <td>Chinese Restaurants</td>
      <td>BBQ Joints</td>
      <td>Noodle Houses</td>
      <td>Snack Places</td>
      <td>Thai Restaurants</td>
      <td>Soup Places</td>
      <td>Seafood Restaurants</td>
      <td>Indian Restaurants</td>
      <td>Diners</td>
    </tr>
    <tr>
      <th>MARINE PARADE</th>
      <td>1950.000000</td>
      <td>1.308410</td>
      <td>103.888814</td>
      <td>0</td>
      <td>Noodle Houses</td>
      <td>Chinese Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Seafood Restaurants</td>
      <td>Snack Places</td>
      <td>Dessert Shops</td>
      <td>Dim Sum Restaurants</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Bars</td>
    </tr>
    <tr>
      <th>PASIR RIS</th>
      <td>2066.666667</td>
      <td>1.373191</td>
      <td>103.949353</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Coffee Shops</td>
      <td>Italian Restaurants</td>
      <td>Asian Restaurants</td>
      <td>Bakeries</td>
      <td>Sushi Restaurants</td>
      <td>Hong Kong Restaurants</td>
      <td>Seafood Restaurants</td>
      <td>Sandwich Places</td>
    </tr>
    <tr>
      <th>PUNGGOL</th>
      <td>1825.000000</td>
      <td>1.405170</td>
      <td>103.902356</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Seafood Restaurants</td>
      <td>Cafés</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Cantonese Restaurants</td>
      <td>Ice Cream Shops</td>
      <td>Burger Joints</td>
      <td>Ramen Restaurants</td>
      <td>Comfort Food Restaurants</td>
    </tr>
    <tr>
      <th>QUEENSTOWN</th>
      <td>2162.500000</td>
      <td>1.294835</td>
      <td>103.805902</td>
      <td>4</td>
      <td>Chinese Restaurants</td>
      <td>Food Courts</td>
      <td>Malay Restaurants</td>
      <td>Thai Restaurants</td>
      <td>Italian Restaurants</td>
      <td>Wings Joints</td>
      <td>Dongbei Restaurants</td>
      <td>Comfort Food Restaurants</td>
      <td>Dessert Shops</td>
      <td>Dim Sum Restaurants</td>
    </tr>
    <tr>
      <th>SEMBAWANG</th>
      <td>1883.333333</td>
      <td>1.449080</td>
      <td>103.820058</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Asian Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Sushi Restaurants</td>
      <td>Bistros</td>
      <td>Japanese Restaurants</td>
      <td>Cafés</td>
      <td>Italian Restaurants</td>
    </tr>
    <tr>
      <th>SENGKANG</th>
      <td>1900.000000</td>
      <td>1.391661</td>
      <td>103.895453</td>
      <td>3</td>
      <td>Coffee Shops</td>
      <td>Cafés</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Sandwich Places</td>
      <td>Sushi Restaurants</td>
      <td>English Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Restaurants</td>
      <td>Comfort Food Restaurants</td>
    </tr>
    <tr>
      <th>SERANGOON</th>
      <td>2187.500000</td>
      <td>1.349787</td>
      <td>103.873635</td>
      <td>3</td>
      <td>Fast Food Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Coffee Shops</td>
      <td>Food Courts</td>
      <td>Steakhouses</td>
      <td>Indian Restaurants</td>
      <td>Chinese Restaurants</td>
      <td>Cafés</td>
      <td>Sushi Restaurants</td>
      <td>Thai Restaurants</td>
    </tr>
    <tr>
      <th>TAMPINES</th>
      <td>2075.000000</td>
      <td>1.354430</td>
      <td>103.942760</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Coffee Shops</td>
      <td>Fast Food Restaurants</td>
      <td>Japanese Restaurants</td>
      <td>Italian Restaurants</td>
      <td>American Restaurants</td>
      <td>Fried Chicken Joints</td>
      <td>Pizza Places</td>
      <td>Portuguese Restaurants</td>
      <td>Seafood Restaurants</td>
    </tr>
    <tr>
      <th>TOA PAYOH</th>
      <td>2210.000000</td>
      <td>1.332330</td>
      <td>103.847425</td>
      <td>3</td>
      <td>Coffee Shops</td>
      <td>Chinese Restaurants</td>
      <td>Food Courts</td>
      <td>Fast Food Restaurants</td>
      <td>Cafés</td>
      <td>Snack Places</td>
      <td>Sandwich Places</td>
      <td>Dessert Shops</td>
      <td>Italian Restaurants</td>
      <td>Burger Joints</td>
    </tr>
    <tr>
      <th>WOODLANDS</th>
      <td>1762.500000</td>
      <td>1.436945</td>
      <td>103.786516</td>
      <td>3</td>
      <td>Food Courts</td>
      <td>Japanese Restaurants</td>
      <td>Coffee Shops</td>
      <td>Cafés</td>
      <td>Chinese Restaurants</td>
      <td>Fast Food Restaurants</td>
      <td>American Restaurants</td>
      <td>Indian Restaurants</td>
      <td>Sandwich Places</td>
      <td>Burger Joints</td>
    </tr>
    <tr>
      <th>YISHUN</th>
      <td>1900.000000</td>
      <td>1.429548</td>
      <td>103.835033</td>
      <td>1</td>
      <td>Food Courts</td>
      <td>Hainan Restaurants</td>
      <td>Bubble Tea Shops</td>
      <td>Halal Restaurants</td>
      <td>Hong Kong Restaurants</td>
      <td>Fish &amp; Chips Shops</td>
      <td>Fast Food Restaurants</td>
      <td>Coffee Shops</td>
      <td>Chinese Restaurants</td>
      <td>Noodle Houses</td>
    </tr>
  </tbody>
</table>
</div>

<img src = "https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/clustered_vicimap.png" width = 800, align = "center">
<a href="https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/singapore_food_venues.600Scores.Category.csv" target="_blank">Download Github:singapore_food_venues</a>
<a href="https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/singapore_outdoorAndRecration.Category.noscore.csv" target="_blank">Download Github:singapore_outdoorAndRecreation</a>
<a href="https://raw.githubusercontent.com/crismag/Coursera_Capstone/master/saved_data/singapore_Nightlife_by_town.Category.csv" target="_blank">Download Github:singapore_Nightlife</a>

________

## Discussion and Conclusion

On this notebook, Analysis of best town venue recommendations based on Food venue category has been presented. Recommendations based on other user searches like available outdoor and recreation areas are also available. As singapore is a small country with a whole host of interesting venues scattered around the town, the information extracted in this notebook present on the town areas, will be a good supplement to web based recommendations for visitors to find out nearby venues of interest and be a useful aid in deciding a place to stay or where to go during their visits.

Using Foursquare API, we have collected a good amount of venue recommnedations in Singapore Towns. Sourcing from the venue recommendations from FourSquare has its limitation, The list of venues is not exhaustive list of all the available venues is the area. Furthermore, not all the venues found in the the area has a stored ratings. For this reason, the number of analyzed venues are only about 50% of all the available venues initially collected. The results therefore may significantly change, when more information are collected on those with missing data. 

The generated clusters from our results shows that there are very good and interesting places located in areas where the median rents are cheaper. This kind of results may be very interesting for travelers who are also on budget constraints. Our results also yielded some interesting findings. For instance, The initial assumption among websites providing recommendations is that the Central Area that have the highest median rent also have better food venues. The results however shows that while Marine Parade, a cheaper location has better rated food courts. Result shows that most popular food venue among Singaporeans, residents and visitors are **Food Courts, Coffee Shops and Fast Food Restaurants**. The highest rated Food Courts are located in __Marine Parade__, and in __Central Area__.

I will be providing a other supplementary Inferential Statics in the future about on these data collected and also update in a new notebook using other categories. For now, this completes the requirements for this task.

Thank you.

Cris Magalang  
email: crism.dev@gmail.com  
linkedin: www.linkedin.com/in/crismagalang  


Created For: COURSERA __**IBM Applied Data Science Capstone** Project__
