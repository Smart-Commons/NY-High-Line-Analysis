# Methodology

### Dataset choice:

We analysed property prices in Manhattan, between 2007 and 2018 using  publicly available data from [nyc.gov](https://www1.nyc.gov/).

[“Property Assessment”](https://www1.nyc.gov/site/finance/taxes/property-assessments.page) is the primary dataset for our analysis. It is collected for property tax calculations and has consistent data for 2007 to 2018, and it correlates well with the [“Rolling Sales Data”](https://www1.nyc.gov/site/finance/taxes/property-rolling-sales-data.page) dataset.

<p align="center"><img width="30%" src="https://lh6.googleusercontent.com/lffy1I3o60c8IG5M-dCfNagxn6E4qrhOI4mzmf-xRq1_dkctT5ZR1SBS_eCanMGyU_1KUXsDuFAlSqaA9Vbi4u3a4jv3NzvpxIz9SoALoSl8uVDJ09ZLqhbhNgyTWFpQs9rOYJRS" /></p>

*We filtered properties that were sold over the period of time from 2007 to 2018 and compared their assessed value to their actual sales price, to make sure the datasets were well correlated.*

### Dataset preparation:

For the purposes of this analysis we had to slightly modify the existing dataset.

The way the dataset locates each property is through assigning them a block number and a tax lot. While blocks always remain unchanged, sometimes lots will be divided or united (or split between land value and apartment value as in the case of condos), which makes it impossible to make an “apples to apples” comparison of property values over time. Different property types are recorded in different ways and many lots change land use and [tax class](https://www1.nyc.gov/site/finance/taxes/property-tax-rates.page). In order to overcome this issue we have substituted each property’s lot, assigned in the dataset to a synthetic parent lot. This is done through uniting lots that changed shape over the period of time from 2007 to 2018 and adding all the values within one synthetic parent lot. In this way, we will be able to assess how a sets of properties changed value over time.  

```sql
SELECT
    CASE
        WHEN
            ap_lot = 0 THEN
            lot ELSE ap_lot
        END AS aplot,
    CASE
        WHEN
            ap_block = 0 THEN
            block ELSE ap_block
        END AS apblock,
    cur_fv_t,
    year4
FROM
    `property assessments`
GROUP BY
    apblock,
    aplot,
    year4
```

### Property value change:

Next, we calculate the coefficients of value change for each year. This is done by calculating the cumulative value of properties, within one synthetic parent lot, in a particular year and dividing it by the same value from the previous year.

<!---
```cs
code placeholder
```
--->

*We did not filter out new construction in this, but capped max value growth at 100000%*

<p align="center"><img width="100%" src="img/value_uplift.gif" /></p>

### High Line effect and distance bands:

We’ve used data from [NYC Street Centerline](https://data.cityofnewyork.us/Business/Zip-Code-Boundaries/i8iw-xf4u) to calculate distances from each of our newly generated lots to the High Line. Using the [Closeness Centrality](https://en.wikipedia.org/wiki/Closeness_centrality) method we can calculate how many kilometres you would have to travel to reach High Line via sidewalks. We then divided the lots into four distance ranges to High Line:

<p align="center"><img width="100%" src="img/distance.gif" /></p>

1. 0-1 km
2. 1-2 km
3. 2-3 km
4. the rest of New York City

### Two methods for calculating value uplift:

We have used two distinct methodologies for calculating property value uplifts across these bands, each appropriate for different situtaions.

### Methodology 1 - Total market value change per band:

This method looks at the change in total market value of all properties within each band, between 2007 and 2018. Summing the values of synthetic plots within each band and tracking that value change over time we can see how total market value of the band grew over the period. We have used this methodology to calculate additional property tax revenue received by the government as a result of High Line. To do this we calculated the difference between the total market value of nearby properties if they had grown at the Manhattan mean rate compared to their actual growth, and then calculated the additional property tax revenue resulting from this uplift.

<!---
```cs
code placeholder
```
--->

<p align="center"><img width="100%" src="img/total_uplift.jpg" /></p>

### Methodology 2 - Mean individual property value change by band:

This method takes each individual property and calculates the value uplift it experienced over the period, then takes a mean of these values within each band. This gives a different measure of average uplift since low value properties which experience significant uplift will impact this measure more significantly than the total market value measure. We have therefore used this methodology when focussing on the average uplift experienced by individual landowners.

<!---
```cs
code placeholder
```
--->

<p align="center"><img width="100%" src="img/average_uplift.jpg" /></p>

### Datasets used in this study

* [PLUTO](https://www1.nyc.gov/site/planning/data-maps/open-data/dwn-pluto-mappluto.page)
* [Property Assessment](https://www1.nyc.gov/site/finance/taxes/property-assessments.page)
* [Sales Data]()
* [NYC Street Centerline](https://data.cityofnewyork.us/Business/Zip-Code-Boundaries/i8iw-xf4u)
* [Building Footprints](https://data.cityofnewyork.us/Housing-Development/Building-Footprints/nqwf-w8eh)
