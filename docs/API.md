# API Docs
This page serves as the documentation for the Pirate Weather API call and response format. Since this service is designed to be a drop in replacement for the [Dark Sky API](https://web.archive.org/web/20200723173936/https://darksky.net/dev/docs), the goal is to match that as closely as possible, and any disagreement between their service and Pirate Weather will be treated as a bug. However, as Pirate Weather continues to evolve, I plan on adding small, non-breaking additions where I can, and they will be documented here! Plus, always better to have my own (open source and editable) version of the docs!

## Request
The minimum structure for every request to this service is the same:
```
      https://api.pirateweather.net/forecast/[apikey]/[latitude],[longitude]
``` 
This specifies the service (either `api` or `timemachine`), root url (`pirateweather.net/forecast`), the api key used in the request (`[apikey]`), and the location (`[latitude],[longitude]`). There are many other ways to customize this request, but this is the minimum requirement! Calling the API with this request will return a JSON data structure (described below) with the requested weather information!
All request attributes are contained within the URL. Request headers are not parsed by the API, and returned headers only contain debugging information, with all the data contained in the JSON payload. 

### Request Parameters
The forecast request can be extended in several ways by adding parameters to the URL. The full set of URL options is:
```
      https://api.pirateweather.net/forecast/[apikey]/[latitude],[longitude],[time]?exclude=[excluded]&units=[unit]&extend=[hourly]&tz=[precise]
``` 

#### API Key
The API key needs to be requested from <https://pirateweather.net/>. After signing up for the service, the forecast API needs to be subscribed to, by logging in and clicking subscribe. Once subscribed to the API, it can take up to 20 minutes for the change to propagate to the gateway to allow requests, so go grab a coffee and it should be ready shortly after. 
As a reminder, this key is secret, and unique to each user. Keep it secret, and do not have it hard-coded into an application's source, and definitely don't commit it to a git repo!

#### Location
The location is specified by a latitude (1st) and longitude (2nd) in decimal degrees (ex. `45.42,-75.69`). An unlimited number of decimal places are allowed; however, the API only returns data to the closest 13 km model square, so there's no benefit after 3 digits. While the recommended way to format this field is with positive (North/ West) and negative (South/ East) degrees, results should be valid when submitting longitudes from 0 to 360, instead of -180 to 180. 

#### Time
The time field is optional for the forecast request, but mandatory for a historic request. If present, time can be specified in one of three different ways:

1. UNIX timestamp, or the number of seconds since midnight GMT on 1 Jan 1970 (this is the preferred way).
2. A datestring in the local time zone of the location being requested: `[YYYY]-[MM]-[DD]T[HH]:[MM]:[SS]`.
3. A datestring in UTC time: `[YYYY]-[MM]-[DD]T[HH]:[MM]:[SS]Z`
4. A time delta (in seconds) from the current time (ex. to get results for the previous day): `-86400`.

It's worth noting that Dark Sky also allows strings with a specified time zone (ex. `+[HH][MM]`). Right now this isn't supported, but if it's important for a workflow I can try to get it working.
If the time variable is not included, then the current time is used for the request. If a time variable is included, the request is treated as if it was requested at that time. This means that the API will return the forecast data that would have been returned then- so not quite observations, but the last forecast for that date. Results are always returned in UTC time using UNIX timestamps, and internally UNIX time is used for everything, with the exception of calculating where to begin and end the daily data. Also, for checking time format conversions, I found <https://www.silisoftware.com/tools/date.php> to be an invaluable resource.

#### Units
Specifies the requested unit for the weather conditions. Options are

* `ca`: SI, with Wind Speed and Wind Gust in kilometres per hour.
* `uk`: SI, with Wind Speed and Wind Gust in miles per hour and visibility are in miles.
* `us`: Imperial units
* `si`: SI units

For compatibility with Dark Sky, `us` (Imperial units) are the default if nothing is specified. For reference, the SI units are

* `summary`: Temperatures in degrees Celsius or accumulation in centimetres .
* `precipIntensity`: Millimetres per hour.
* `precipIntensityMax`: Millimetres per hour.
* `precipAccumulation`: Centimetres.
* `temperature`: Degrees Celsius.
* `temperatureMin`: Degrees Celsius.
* `temperatureMax`: Degrees Celsius.
* `apparentTemperature`: Degrees Celsius.
* `dewPoint`: Degrees Celsius.
* `windSpeed`: Meters per second.
* `windGust`: Meters per second.
* `pressure`: Hectopascals.
* `visibility`: Kilometres.

#### Exclude
Newly added as part of the V1.0 release, this parameter removes some of the data blocks from the reply. This can speed up the requests (especially if alerts are not needed!), and reduce the reply size. Exclude parameters can be added as a comma-separated list, with the options being:

* `currently`
* `minutely`
* `hourly`
* `daily`
* `alerts`

#### Extend
If `extend=hourly` is included, hourly data for the next 168 hours will be included, instead of the standard 48! This adds some time (~0.3s) to the response, since additional processing is required.

#### Time Zone
Finally, if `tz=precise` is included, the high precision algorithm of [TimeZoneFinder](https://timezonefinder.readthedocs.io/en/latest/) is used in place of the rapid one. This also adds some time (~0.3s), and in most cases doesn't impact the results (since everything is reported in UTC, the only thing the timezone is used for is to determine the start and end point of the day), but is added as an option if you need an accurate zone.   


### Example
	```
	GET https://api.pirateweather.net/forecast/1234567890abcdefghijklmnopqrstuvwxyz/45.42,-74.30?&units=ca
	{
	   "latitude":45.42,
	   "longitude":-74.30,
	   "timezone":"America/Toronto",
	   "offset":-5.0,
	   "currently":{
	      "time":1641470400,
	      "summary":"Clear",
	      "icon":"clear-night",
	      "nearestStormDistance":0,
	      "nearestStormBearing":0,
	      "precipIntensity":0.0,
	      "precipProbability":0.0,
	      "precipIntensityError":0.0,
	      "precipType":"none",
	      "temperature":-4.48,
	      "apparentTemperature":-12.06,
	      "dewPoint":-6.87,
	      "humidity":0.84,
	      "pressure":992.89,
	      "windSpeed":27.918,
	      "windGust":55.44,
	      "windBearing":242.11,
	      "cloudCover":0,
	      "uvIndex":0.0,
	      "visibility":19.5,
	      "ozone":401.15
	   },
	   "minutely":{
	      "summary":"Clear",
	      "icon":"clear",
	      "data":[
	         {
	            "time":1641470400,
	            "precipIntensity":0.0,
	            "precipProbability":0.07,
	            "precipIntensityError":0.0011,
	            "precipType":"none"
	         },
	   ...
	      ]
	   },
	   "hourly":{
	      "summary":"Cloudy",
	      "icon":"cloudy",
	      "data":[
	         {
	            "time":1641470400,
	            "icon":"clear-night",
	            "summary":"Clear",
	            "precipIntensity":0.365,
	            "precipProbability":0.07,
	            "precipIntensityError":0.4044,
	            "precipAccumulation":0.365,
	            "precipType":"snow",
	            "temperature":-4.48,
	            "apparentTemperature":-12.06,
	            "dewPoint":-6.87,
	            "humidity":0.84,
	            "pressure":992.89,
	            "windSpeed":27.92,
	            "windGust":55.44,
	            "windBearing":242.11,
	            "cloudCover":0,
	            "uvIndex":0.0,
	            "visibility":19.5,
	            "ozone":401.15
	         },
	   ...
	      ]
	   },
	   "daily":{
	      "summary":"Partly Cloudy",
	      "icon":"partly-cloudy-day",
	      "data":[
	         {
	            "time":1641445200,
	            "icon":"cloudy",
	            "summary":"Cloudy",
	            "sunriseTime":1641472616,
	            "sunsetTime":1641504573,
	            "moonPhase":0.14013338096939384,
	            "precipIntensityMax":0.0441,
	            "precipIntensityMaxTime":1641538800,
	            "precipProbability":0.071,
	            "precipAccumulation":0.3479,
	            "precipType":"snow",
	            "temperatureHigh":-3.59,
	            "temperatureHighTime":1641484800,
	            "temperatureLow":-10.27,
	            "temperatureLowTime":1641524400,
	            "apparentTemperatureHigh":-8.73,
	            "apparentTemperatureHighTime":1641538800,
	            "apparentTemperatureLow":-14.54,
	            "apparentTemperatureLowTime":1641524400,
	            "dewPoint":-8.74,
	            "humidity":0.901,
	            "pressure":998.27,
	            "windSpeed":14.74,
	            "windGust":26.68,
	            "windBearing":256.84,
	            "cloudCover":0.943,
	            "uvIndex":1.91,
	            "uvIndexTime":1641492000,
	            "visibility":17.35,
	            "temperatureMin":-10.27,
	            "temperatureMinTime":1641524400,
	            "temperatureMax":-3.59,
	            "temperatureMaxTime":1641484800,
	            "apparentTemperatureMin":-14.54,
	            "apparentTemperatureMinTime":1641524400,
	            "apparentTemperatureMax":-8.73,
	            "apparentTemperatureMaxTime":1641538800
	         },
	  ...
	      ]
	   },
	   "flags":{
	      "sources":[
	         "gfs",
	         "gfes"
	      ],
	      "nearest-station":0,
	      "units":"ca",
	      "verson":"V1.0"
	   }
	}
	```

### Time Machine Request
In progress.
## Response
In progress.
### Data Block
In progress.

### Data Point
In progress.

#### Icon
One of a set of icons to provide a visual display of what's happening. This could be one of: 
`clear-day, clear-night, rain, snow, sleet, wind, fog, cloudy, partly-cloudy-day, partly-cloudy-night`.

The algorithm here is straightforward, coming from this [NOAA resource](https://weather.com/science/weather-explainers/news/common-weather-terms-used-incorrectly):

##### Hourly:

* If precipitation probability is greater than 30% and accumulation is greater than 0.25 mm, then the precipitation type.
* If visibility is less than 1 km, then `fog`.
* If winds are greater than 10 m/s, then `wind`.
* If cloud cover is greater than 75%, then `cloudy`.
* If cloud cover is greater than 37.5% and less than 75%, then `partly-cloudy-day` or `partly-cloudy-night`.
* If cloud cover is less than 37.5%, then `clear`.

##### Daily:
* If max probability is greater than 30% in any hour and total accumulation is greater than 1 mm, then precipitation type.
	* Type is based on the most common (modal) precipitation type.
* If average visibility is less than 1 km, then `fog`.
* If average wind speed is greater than 10 m/s, then `wind`.
* If average cloud cover is greater than 75%, then `cloudy`.
* If average cloud cover is greater than 37.5% and less than 75%, then `partly-cloudy-day`.
* If average cloud cover is less than 37.5%, then `clear`.

For additional details, see [issue #3](https://github.com/alexander0042/pirateweather/issues/3).


### Alerts
In progress.

### Flags
In progress.