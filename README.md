<p align="center">
<img src="./extra/Fitbit_dashboard.png" width="500" height="160" align="center">
</p>

# Fitbit Fetch script and Influxdb Grafana integration
A script to fetch data from Fitbit servers using their API and store the data in a local influxdb database. 

## Dashboard Example
![Dashboard](https://github.com/arpanghosh8453/public-fitbit-projects/blob/main/Grafana_Dashboard/Dashboard.png?raw=true)

## Features

- Automatic data collection from Fitbit API
- Support for both InfluxDB 1.x and 2.x
- Collects comprehensive health metrics including:
  - Heart Rate Data (including intraday)
  - Hourly steps Heatmap
  - Daily Step Count
  - Sleep Data and patterns
  - Sleep regularity heatmap
  - SpO2 Data
  - Breathing Rate
  - HRV
  - Activity Minutes
  - Device Battery Level
  - And more...
- Automated token refresh
- Historical data backfilling
- Rate limit aware data collection

## How to setup the script

#### Set up influxdb 1.8 ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#influxdb) ). Create a user with a password and an empty database. 

#### Set up grafana recent release ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#grafana) )

#### Use the requirements.txt file to install the required packages. 

#### Follow this [guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/) to create an application. This will give you a client ID, client secret, and a refresh token ( end step after following OAuth setup )

❗❗ The Fitbit application must be personal type for the access of intraday data series ❗❗

#### Update the following variables in the script ( use the influxdb-v2 specific variables for influxdb-v2 instance )
-  FITBIT_LOG_FILE_PATH = "your/expected/log/file/location/path"
-  TOKEN_FILE_PATH = "your/expected/token/file/location/path"
-  INFLUXDB_USERNAME = 'your_influxdb_username'
-  INFLUXDB_PASSWORD = 'your_influxdb_password'
-  INFLUXDB_DATABASE = 'your_influxdb_database_name'
-  client_id = "your_application_client_ID"
-  client_secret = "your_application_client_secret"
-  DEVICENAME = "Your_Device_Name" # example - "Charge5"
-  LOCAL_TIMEZONE=Automatic # set to "Automatic" for Automatic setup from User profile (if not mentioned here specifically). 

#### Run the script; it will request a refresh token as input for the first run to set up the token file. You can check the logs to see the work in progress. The script, by default, keeps running forever, calling different functions at scheduled intervals. 

#### Finally, add the influxdb database as a Data source in Grafana, and use the given Dashboard json file to replicate the dashboard quickly. 

#### You can use the Fitbit_Fetch_Autostart.service template to set up an auto-starting ( and auto-restarting in case of temporary failure ) service in Linux based system ( or WSL )

## Full Stack install with Docker 

#### Follow this [guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/) to create an application. This will give you a client ID, client secret, and a refresh token ( end step after following OAuth setup )
#### Initial setup : Create a folder named fitbit-fetch-data, cd into the folder, create a docker-compose.yml file with the below compose example ( Change the enviornment variables accordingly )
#### Create folders named logs and tokens inside and make sure to chown them for uid 1000 ( otherwise you may get read/write permission denied errors )
#### Initial set up of Access and Refresh tokens with the command : `docker pull thisisarpanghosh/fitbit-fetch-data:latest && docker compose run --rm fitbit-fetch-data`
#### Enter the refresh token you obtained from your fitbit account and hit enter. ❗❗ The Fitbit application type must be personal for intraday data access ❗❗
#### Then exit out with ctrl + c ( after you see the successful api requests in the stdout log )
#### Finally run : `docker compose up -d` ( to launch the full stack )

```
services:
  fitbit-fetch-data:
    restart: unless-stopped
    image: thisisarpanghosh/fitbit-fetch-data:latest
    container_name: fitbit-fetch-data
    volumes:
      - ./logs:/app/logs # logs folder should exist and owned by user id 1000
      - ./tokens:/app/tokens # tokens folder should exist and owned by user id 1000
      - /etc/timezone:/etc/timezone:ro
    environment:
      - FITBIT_LOG_FILE_PATH=/app/logs/fitbit.log
      - TOKEN_FILE_PATH=/app/tokens/fitbit.token
      - OVERWRITE_LOG_FILE=True
      - INFLUXDB_VERSION=1 # supported values are 1 and 2 
      # Variables for influxdb 2.x ( you need to change the influxdb container config below accordingly )
      - INFLUXDB_BUCKET=your_bucket_name_here # for influxdb 2.x
      - INFLUXDB_ORG=your_org_here # for influxdb 2.x
      - INFLUXDB_TOKEN=your_token_here # for influxdb 2.x
      - INFLUXDB_URL=your_influxdb_server_location_with_port_here # for influxdb 2.x
      # Variables for influxdb 1.x
      - INFLUXDB_HOST=influxdb # for influxdb 1.x
      - INFLUXDB_PORT=8086 # for influxdb 1.x
      - INFLUXDB_USERNAME=fitbit_user # for influxdb 1.x
      - INFLUXDB_PASSWORD=fitbit_password # for influxdb 1.x
      - INFLUXDB_DATABASE=fitbit_database # for influxdb 1.x
      - CLIENT_ID=your_application_client_ID # Change this to your client ID
      - CLIENT_SECRET=your_application_client_secret # Change this to your client Secret
      - DEVICENAME=Your_Device_Name # Change this to your device name - e.g. "Charge5"
      - LOCAL_TIMEZONE=Automatic # set to "Automatic" for Automatic setup from User profile (if not mentioned here specifically). 


  influxdb:
    restart: unless-stopped
    container_name: influxdb
    environment:
      - INFLUXDB_DB=fitbit_database
      - INFLUXDB_USER=fitbit_user
      - INFLUXDB_USER_PASSWORD=fitbit_password
    ports:
        - '8086:8086'
    volumes:
        - './influxdb:/var/lib/influxdb'
    image: 'influxdb:1.8'

  grafana:
    restart: unless-stopped
    volumes:
        - './grafana:/var/lib/grafana'
    ports:
        - '3000:3000'
    container_name: grafana
    image: 'grafana/grafana:latest'
```
## Deploy with Homeassistant integration

User [@Jasonthefirst](https://github.com/Jasonthefirst) has developed a plugin (issue [#24](https://github.com/arpanghosh8453/public-fitbit-projects/issues/24) ) based on the python script which can be used to deploy the setup without docker. Please refer to [fitbit-ha-addon](https://gitlab.fristerspace.de/demian/fitbit-ha-addon) for the setup. 
