
Proposal  

Singapore Visitors and XPATS Venue Recommendation
Introduction

Introduction

Singapore is a small country and is one of the most visited countries in Asia. There are a lot of websites where travelers can check and retrieve recommendations of places to stay or visit.
However, most of this websites creates a recommendation simply based on usual tourist attractions or key residential areas that are mostly expensive or already known for travelers based on
certain keywords like "Hotel", or "Backpackers" etc. The intention on this project is to collect and create a data driven recommendation that can supplement these websites by utilizing data
retrieved from Singapore open data sources and FourSquare API venue recommendations.

To limit the scope of this sample recommender, this notebook will be using a scenario where we compare a cost of 1 month HDB unit rental in a different Singapore towns and then look at the
venue recommendation from FourSquare API.

For this project, this notebook will look at the following data:

    Singapore Median Rental Prices by town.
    Popular Food venues in the vicinity.

With this data, a sample website recommendation is to provide visitors on a limited budget a decision tool to decide on where to stay, or select a location more suitable for his place of interest. Other possible checks that the user can access using this same notebook are categories like Outdoors and Recreation or Nightlife.
Data Acquisition

This part of notebook will be in-charge of retrieving and preparing the data for this project.
This project will be getting needed information from the following sources:

    Singapore Towns and median residential rental prices
        Data will retrieved from Singapore open data-set from "median rent by town and flat-type" from https://data.gov.sg website.
    Singapore Towns Coordinates
        Retrieved using google maps API
    Singapore Top Venue Recommendations
        Retrieved using FourSquare API, we will retrieve a sample user places of interest.
