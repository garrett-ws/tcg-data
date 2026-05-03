# TCG Data

## Overview

This project uses the Open TCG API to collect data on Trading Card Games (TCGs), stores the data in a MongoDB database, and allows for data aggregation and analysis on the data.

Data is collected using the Open TCG API, which collects TCG data primarily from [TCGPlayer](https://www.tcgplayer.com/), though some TCG products will have additional information from other TCG sites, such as [ManaPool](https://manapool.com/) and [CardTrader](https://www.cardtrader.com/).

For more detailed information on the Open TCG API reference the [Open TCG API documentation](https://tcgtracking.com/tcgapi/?tab=docs)

## Tools

Open TCG API: https://tcgtracking.com/tcgapi/

MongoDB: https://www.mongodb.com/

## Setup

Create a MongoDB database to store the data by following the instructions in the [Mongo help article]([https://pages.github.com/](https://www.mongodb.com/resources/products/fundamentals/create-database)).

Save a copy of the `.env.example` file as `.env` and update the `MONGO_DB_CONNECTION_STRING` and `MONGO_DATABASE =` to point to your cluster and database. Note that this project was built around a MongoDB Atlas database, and a local MongoDB database has not been tested with it.

>[!WARNING]
>The free tier of MongoDB Atlas only offers 512MB of storage. If using the free tier of MongoDB Atlas you may want to adjust what data is collected or imported to stay under the necessary data size.

For more detailed information on MongoDB reference the [MongoDB documentation](https://www.mongodb.com/docs/)

## Data Collection

Data is collected using two scripts, which each collect a different set of data:

- `tcgapi_mass_download.py`
  - Categories
  - Sets
  - Products
- `pricing_download.py`
  - Prices
  - SKUs

The collected data is saved in a series of JSON files for each data set.

Categories refers to the different TCGs (ex: Magic, Pokemon, Yu-Gi-Oh, etc). Sets are the different sets released for each game, and Products are the individual products released for each set (sealed, singles, misc. items, etc).

Prices are the most recent market (average) and low price for different products, while SKUs are the individual listings for each product. The Open TCG API does not have historical data for either of these, so the script will grab the most recent of each and skip scraping for products where the most recent data is already downloaded locally.

## Data Import

Each data set is imported with its own respective import script. This allows for a more granular import if the full data set is not needed (or if some of it has already been imported). Some of the imports can take a very long time depending on how much data was collected.

The import script will create the target collection in your database if it does not already exist.

## Data Normalization

Once the desired data is imported some normalization needs to be performed on the data. It is necessary for these steps to be run in order. The scripts can be loaded from mongosh or the contents of the scripts copy/pasted directly into mongosh.

Run the following scripts:

- `aggregations_pipelines/field_corrections.js`
- `aggregations_pipelines/prices_add_high_price_field.js`
- `aggregations_pipelines/create_indexes.js`

Both `field_corrections.js` and `prices_add_high_price_field.js` will need to be re-run when new data is added to the database. `create_indexes.js` does not need to be ru-run as MongoDB will automatically update the indexes.

The data normalization is needed to:
- Format field names as all lowercase
- Remove spaces from field names
- Rename fields for clarity (e.g., `id` -> `product_id`)
- Convert fields that imported as the wrong data type (e.g., `String` -> `Int`)
- Add additional fields needed for data aggregation
- Create indexes needed for data aggregation

## Data Aggregation

After the data has been normalized the data aggregation pipeline can be run from mongosh. The aggregation pipeline can be run by calling or copy/pasting the contents of `aggregations_pipelines/prices_most_expensive_agg.js` into mongosh.

The aggregation pipeline:
- Finds the highest priced card listing per set
- Retrieves additional information about each card (set, etc)
- Writes the documents to a new collection

The new collection allows for easier data aggregation on the desired information, as it reduces the load needed when using Atlas Charts on the data.

![TCG Data Atlas Charts Example](https://github.com/garrett-ws/tcg-data/blob/main/assets/tcg_data_atlas_charts_example.png)

## To-do
- [ ] Test setup on local MongoDB database
- [ ] Setup Docker for project
