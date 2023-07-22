# Fitbit Fetch script and Influxdb Grafana integration
A script to fetch data from Fitbit servers using their API and store the data in a local influxdb database. 

## Dashboard Example
![Dashboard](https://github.com/arpanghosh8453/public-fitbit-projects/blob/main/Grafana_Dashboard/Dashboard.png?raw=true)

## How to setup the script

#### Set up influxdb 1.8 ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#influxdb) ). Create a user with a password and an empty database. 

#### Set up grafana recent release ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#grafana) )

#### Use the requirements.txt file to install the required packages. 

#### Follow this [guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/) to create an application. This will give you a client ID, client secret, and a refresh token ( end step )

#### Update the following variables in the script
-  FITBIT_LOG_FILE_PATH = "your/expected/log/file/location/path"
-  TOKEN_FILE_PATH = "your/expected/token/file/location/path"
-  INFLUXDB_USERNAME = 'your_influxdb_username'
-  INFLUXDB_PASSWORD = 'your_influxdb_password'
-  INFLUXDB_DATABASE = 'your_influxdb_database_name'
-  client_id = "your_application_client_ID"
-  client_secret = "your_application_client_secret"
-  DEVICENAME = "Your_Device_Name" # example - "Charge5"
-  AUTO_DATE_RANGE selects current date by default, if you want to load past data into the database, simply make it to False and the script will ask for start and end dates ( in YYYY-MM_DD format ) at runtime. 

#### Run the script; it will request a refresh token as input for the first run to set up the token file. You can check the logs to see the work in progress. The script, by default, keeps running forever, calling different functions at scheduled intervals. 

#### Finally, add the influxdb database as a Data source in Grafana, and use the given Dashboard json file to replicate the dashboard quickly. 

#### You can use the Fitbit_Fetch_Autostart.service template to set up an auto-starting ( and auto-restarting in case of temporary failure ) service in Linux based system ( or WSL )

