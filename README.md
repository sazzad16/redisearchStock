# redisearchStock
A simple stock ticker solution based on downloaded stock files
## Initial project setup
Get this github code
```bash 
get clone https://github.com/jphaugla/redisearchStock.git
```
Two options for setting the environment are given:  
  * run with docker-compose using a flask and redis container
  * installing for mac os
  * running on linux (probably in the cloud)

## Code and File discussion


### Download the datafiles to the data subdirectory
* Download stock files here
 [Stooq stock files](https://stooq.com/db/h/)
* Once downloaded, move the file (it should be a directory called *data*) to the main redisearchStock directory

### Set environment

The docker compose file has the environment variables set for the redis connection and the location of the data files.
This code uses redisearch.  The redis database must have both of these modules installed.
As of this writing, this redismod docker image (which includes these modules) does not work on the m1 arm64 based mac.  
Default docker-compose is set to redismod.  Check the environment variables for appropriateness. Especially check the TICKER_DATA_LOCATION because loading all of 
the US tickers with all of the history can be a lot of data on a laptop.  Here is explanation of the non-obvious variables
Modify these values in docker-compose.yml

| variable             | Original Value | Desccription                                                                                                                                                              |
|----------------------|----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| REDIS_HOST           | redis          | The name of the redis docker container                                                                                                                                    |
| REDIS_PORT           | 6379           | redis port                                                                                                                                                                |
| TICKER_FILE_LOCATION | /data          | leave in /data but can use sub-directory to minimize load size                                                                                                            | 
| PROCESSES            | 6              | On larger machines, increases this will increase load speed                                                                                                               |
| WRITE_JSON           | false          | flag to turn on to use JSON instead of Hash structures                                                                                                                    |
| PROCESS_DATES        | true           | have date-based logic instead of just a simple initial load.  Allows for <br/> skipping any records old than a particular date (requires creation of specific redis hash) |   
| PROCESS_RECENTS      | false          | will set most recent flag for specified keys back back to false    (requires creation of specific redis set)                                                              |

The created index is filtered to only the records where MostRecent is set to true

## docker compose startup
```bash
docker-compose up -d 
```


### load Tickers
The redis node and port can be changed. The python code uses 2 environment variable REDIS_SERVER and REDIS_PORT.  The default is REDIS_SERVER=redis and REDIS_PORT=6379
```bash
docker exec -it flask bash -c "python TickerImport.py"
```

Can observe the load progress by watching the load for each file
```bash
docker exec -it redis redis-cli hgetall ticker_load
```
  * THIS IS HOW to start flask app server
  * However, it is already running as part of the flask container
 ```bash
docker exec -it flask bash -c "python appy.py"
 ```
  * run API tests
Easiest is to run the API tests using Postman.  Running Postman, use File->Import to import
the following file for use with Postman-https://github.com/jphaugla/redisJSONProductCatalog/blob/main/scripts/Product-Category%20APIs.postman_collection.json
Once the collection is imported, run each request to test the APIs.
![Postman screen shot](images/postman-collection.png)
Alternatively, use the commands in this file https://github.com/jphaugla/redisJSONProductCatalog/blob/main/scripts/sampleput.sh
Make sure to use *bash* as *zsh* has issues with the curl command 
Note:  there are multiple API tests in the file but only one should be run at a time
So, the tests not to be run should be commented out.  

  * run sample search queries   
run sample redisearch queries as provided.  Run one at a time using

```bash
redic-cli -f scripts/searchQueries.txt
```

##  Notes for running outside of Docker
Follow most of the same steps as above with some changes

### Instead of docker to execute, use python virtualenv
  * create a virtualenv
```bash
cd src
python3 -m venv venv
source venv/bin/activate
```
   * Use an environment file for locations
   * Need to make sure the data location variables are set correctly
   * Can also set the number of concurrent processes for the client using the "PROCESSES" environment variable

```bash
source scripts/app.env
```
  * execute python scripts from the src directory
```bash
cd src
pip install -r requirements.txt
python TickerImport.py
```