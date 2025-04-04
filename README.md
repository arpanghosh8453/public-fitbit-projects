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

Set up influxdb 1.8 ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#influxdb) ). Create a user with a password and an empty database.

Set up grafana recent release ( direct install or via [docker](https://github.com/arpanghosh8453/public-docker-config#grafana) )

Use the requirements.txt file to install the required packages.

Follow this [guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/) to create an application. This will give you a client ID, client secret, and a refresh token ( end step after following OAuth setup )

❗ **The Fitbit application must be personal type for the access of intraday data series** ❗ - Otherwise you might encounter `KeyError: 'activities-heart-intraday'` Error.

Update the following variables in the script ( use the influxdb-v2 specific variables for influxdb-v2 instance )

- FITBIT_LOG_FILE_PATH = "your/expected/log/file/location/path"
- TOKEN_FILE_PATH = "your/expected/token/file/location/path"
- INFLUXDB_USERNAME = 'your_influxdb_username'
- INFLUXDB_PASSWORD = 'your_influxdb_password'
- INFLUXDB_DATABASE = 'your_influxdb_database_name'
- client_id = "your_application_client_ID"
- client_secret = "your_application_client_secret"
- DEVICENAME = "Your_Device_Name" # example - "Charge5"
- LOCAL_TIMEZONE=Automatic # set to "Automatic" for Automatic setup from User profile (if not mentioned here specifically).

Run the script; it will request a refresh token as input for the first run to set up the token file. You can check the logs to see the work in progress. The script, by default, keeps running forever, calling different functions at scheduled intervals.

Finally, add the influxdb database as a Data source in Grafana, and use the given Dashboard json file to replicate the dashboard quickly.

You can use the Fitbit_Fetch_Autostart.service template to set up an auto-starting ( and auto-restarting in case of temporary failure ) service in Linux based system ( or WSL )

## Full Stack install with Docker

Follow this [guide](https://dev.fitbit.com/build/reference/web-api/developer-guide/getting-started/) to create an application. This will give you a client ID, client secret, and a refresh token ( end step after following OAuth setup )

Initial setup : Create a folder named fitbit-fetch-data, cd into the folder, create a compose.yml file with the below compose example ( Change the enviornment variables accordingly )

Create folders named logs and tokens inside and make sure to chown them for uid 1000 ( otherwise you may get read/write permission denied errors )

Initial set up of Access and Refresh tokens with the command : `docker pull thisisarpanghosh/fitbit-fetch-data:latest && docker compose run --rm fitbit-fetch-data`

Enter the refresh token you obtained from your fitbit account and hit enter. ❗ **The Fitbit application type must be personal for intraday data access** ❗- Otherwise you might encounter `KeyError: 'activities-heart-intraday'` Error.

Then exit out with ctrl + c ( after you see the successful api requests in the stdout log )

Finally run : `docker compose up -d` ( to launch the full stack )

In some cases, for the Grafana container, you may need to chown the corresponding mounted folders as *472*:*472* if you are having read/write errors inside the grafana container.

To use the Grafana dashboard, please use the [JSON files](https://github.com/arpanghosh8453/public-fitbit-projects/tree/main/Grafana_Dashboard) downloaded directly from the Grafana_Dashboard of the project (there are separate versions of the dashboard for influxdb v1 and v2) or use the import code **23088** (for influxdb-v1) or **23090** (for influxdb-v2) to pull them directly from the Grafana dashboard cloud.

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

## Historical Data Update

#### Background

The primary purpose of this script is to visualize long term data and if you have just discovered it, you may need to wait a long time to acheive this by automatic daily data fetch process. But fear not! this script was written with that fact in mind. As you may know, **fitbit rate limits the API calls to their server from their users, so only 150 API calls are allowed per hour** and it resets every hour. Some API endpoints allows fetching long term data for months and years while most **intraday data is limited to 24 hours per API call**. So this means if you need to fetch HR and steps data for 5 days, there is no other way but making 5x2=10 API calls to their servers. Now imagine this at scale, multiple measurements over the years of data. I was faced with this exact problem and it really took me a long time to figure out the most optimal way to fetch bulk historic data is to group them into categories based on their period limits and implement robust handing of `429 Error` ('too many requests within an hour' error). 

This script has a feature that in the bulk update mode it will fill up the less limited data first and finally fill up the intraday data so you can see the data filling up in grafana real time as the script progresses. After it exausts it's available 150 calls for the hour, it will go to dormant mode for the remaining duration for that hour, and resume fetching the data as soon as the wait time is up automatically (so you can just leave it and let it work). To give you a timeline, **it took a little more than 24 hours to fetch all the historic data for my 2 years of historic data from their servers**. 

#### Procedure

The process is quite simple. you need to add an ENV variable and rerun the container in interactive mode. here is a step-by-step guide

- Stop the running container and remove it with `docker compose down` if running already

- In the docker compose file, add a new ENV variable `AUTO_DATE_RANGE=False` under the `environment` section along with other variables. This variable switches the mode to bulk update instead of regular daily update

- Assuming you are already in the directory where the `compose.yml` file is, run `docker compose run --rm fitbit-fetch-data` - this will run this container in _"remove container automatically after finish"_ mode which is useful for one time running like this. This will also attach the container to the shell as interactive mode, so don't close the shell until the bulk update is complete. 

- After initialization, you will be requested to input the start and end dates in YYYY-MM-DD format. the format is very important so please enter the dates like this `2024-03-13`. Start date must be earlier than end date. The script should work for any given range, but if you encounter an error during the bulk update with large date range, please break the date range into one year chunks (maybe a few days less than one year just to be safe), and run it for each one year chunk one after another. I personally did not encounter any issue with longer date ranges, but this is just a heads up. 

- You will see the update logs in the attached shell. Please wait until it shpws `Bulk Update Complete` and exits. It might take a long time depending on the given duration and 150 API call limit per hour. 

- You are done with the bulk update at this point. Remove the ENV variable from the compose or change it to `AUTO_DATE_RANGE=True`, save the compose file and run `docker compose up` to resume daily update. 

## Troubleshooting

- If you are getting `KeyError: 'activities-heart-intraday'` please double check if your Fitbit Oauth application is set as `personal` type before you open an issue

- If you are missing GPS data, but you know you have some within the selected time range in grafana, check if the variable GPS Activity variable is properly set or not. You should have a dropdown there. If you do not see any values, please go to the dashboard settings and check if the GPS variable datasource is properly set or not. 

## Deploy with Homeassistant integration

User [@Jasonthefirst](https://github.com/Jasonthefirst) has developed a plugin (issue [#24](https://github.com/arpanghosh8453/public-fitbit-projects/issues/24) ) based on the python script which can be used to deploy the setup without docker. Please refer to [fitbit-ha-addon](https://gitlab.fristerspace.de/demian/fitbit-ha-addon) for the setup.

## Support me

If you enjoy the script and love how it works with simple setup, please consider supporting me with a coffee ❤. You can view more detailed health statistics with this setup than paying a subscription fee to Fitbit, thanks to their free REST API services.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/A0A84F3DP)
