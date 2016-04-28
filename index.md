---
layout: default
---

The Drivr API provides the ability to create and retrieve required information in order to book taxis globally.

## Introduction

Every URL is in the format https://api.drivr.com/:resource[/:action[?:params]] such as https://api.drivr.com/places/nearby to get a list of nearby places.

Every request must include a `User-Agent` header with your application name and a URL or email address. We need this for tracking purposes and we might contact you in case this information isn't properly filled out. An example of a good header would be `FooBooking-Version5 (http://foo-bookings.com/)`. Requests without this header might get a `400 Bad Request` in future versions of the API.

The `Accept` header specifies which format to return. Only `application/json` is supported.

Returned format may be overridden by appending a format parameter to a request e.g. https://api.drivr.com/companies?format=json.

Clients must include an `Accept` header with a custom Drivr mime type to specify which version of the API to use e.g.

	application/vnd.drivr[.version]+[type]

If this header isn't specified the latest API version is used but it's highly recommended to specify a version since future revisions might not be fully backward compatible.

Currently supported is `v1`, `v2` and `latest` for `version` and `json` for `type`. 

Responses will be compressed if request includes an `Accept-Encoding` header with a `gzip` value.

<!-- FUTURE: Operations using a `lang` or `language` parameter might use an `Accept-Language` header instead. -->

All timestamps are represented as ISO 8601 format `YYYY-MM-DDTHH:MM:SSZ`.

All IDs are represented as strings and should be stored and treated as strings. The format might change across API versions without notice.

Example requests are constructed using [cURL](http://en.wikipedia.org/wiki/CURL) commands.


## Suggested booking flow

As a minimum requirement to book a taxi, one must first [acquire an access token](#authorizations) with provided credentials to get an access token. This access token must be used for all future requests.

Second step is to [create a client](#clients) and finally [create a booking](#post_clients_bookings).

Usually one would also like to display which companies [operate in a given area](#spot_companies) and check if that company requires a dropoff address and allows automated bookings. To check if a company supports automated bookings, the `automatedBooking` attribute on a company should be checked. If this is `false`, our API cannot automatically book a taxi with this company. Trying to place a booking will fail, but you can use the returned phone number as a fallback in your application flow. Most companies on our platform allow automatic booking, though.

Checking if a company requires a dropoff address is done by looking at `destinationRequired` boolean attribute. When `true`, both a `pickup` and `dropoff` structure must be sent in a booking request.


## Authentication

You authenticate each request by sending a token either as an `Authorization` header:

	curl https://api.drivr.com \
		-H "Authorization: Token token=\"OAUTH-TOKEN\""

Or by appending it as an `access_token` parameter:

	curl https://api.drivr.com?access_token=OAUTH-TOKEN

Tokens can be initially acquired programmatically using basic authentication through [/authorizations](#authorizations).


## JSON-P callbacks

You may send a `?callback` parameter to any GET operations to have results wrapped in a JSON callback function. This is typically used when browsers want to embed content in web pages by getting around cross domain issues. The response includes the same data output as the regular API, plus the relevant HTTP Header information.


## Batching multiple operations into single request

Our API supports batching multiple API operations into a single HTTP request. This might come in handy to simply client flow if multiple sets of data is required on your end. Below is an example of calling two `quotes` operations with different vehicle types.

All operations can be batched by sending a formatted payload to the `batch` endpoint.


#### Example request

This example batches two requests for `quotes` into one HTTP request. The `method` (required) specify which method to perform on our API, the `relativeUrl` (required) is request to perform on API (including query string parameters) and `body` (optional) is any payload required for endpoint.

	curl -H 'Accept: application/vnd.drivr.v2+json' \
	    -H 'Authorization: Token token="<your-token>"' \
	    -H 'Content-type: application/json' \
	    'https://api.drivr.com/batch' \
	    -d '[{"method":"POST","relativeUrl":"quotes","body":"{ \"pickup\": { \"lat\": 55.697047, \"lng\": 12.550911 }, \"dropoff\": { \"lat\": 55.685871, \"lng\": 12.59477 }, \"vehicleType\": \"taxi\" }"},{"method":"POST","relativeUrl":"quotes","body":"{ \"pickup\": { \"lat\": 55.697035, \"lng\": 12.550611 }, \"dropoff\": { \"lat\": 55.685871, \"lng\": 12.59477 }, \"vehicleType\": \"drivr\" }"}]'

#### Example response

	[
	  {
	    "code": 201,
	    "headers": [
	      {
	        "name": "Location",
	        "value": "https://api.drivr.com/quotes/1579823592"
	      },
	      {
	        "name": "Content-Type",
	        "value": "application/json; charset=utf-8"
	      }
	    ],
	    "body": "{\r\n  \"id\": \"1579823592\",\r\n  \"pickup\": {\r\n    \"lat\": 55.697047,\r\n    \"lng\": 12.550911\r\n  },\r\n  \"dropoff\": {\r\n    \"lat\": 55.685871,\r\n    \"lng\": 12.59477\r\n  },\r\n  \"vehicleType\": \"taxi\",\r\n  \"price\": {\r\n    \"value\": 150.3,\r\n    \"currency\": \"DKK\",\r\n    \"currencySymbol\": \"Kr.\",\r\n    \"type\": \"variable\",\r\n    \"distance\": 4171,\r\n    \"duration\": 809,\r\n    \"fees\": []\r\n  },\r\n  \"polyline\": \"}h}rImirkAt@kCUWmDaEmCiDeBmBQU[u@lAkCz@gCdByFfAcEhA}EXc@z@sD`@}Ar@oCRo@hAaCx@yBvBkE|CgG\\\\g@|@iBV}@rDkInDmIlE}JdKkX`CeG|@wBhAcDDWDOMWc@iAIEuAuDgAoCkCwGi@cBu@yDCe@a@{BAoCGk@QcAOSa@cBy@}BuB{DC]i@y@~@}@VKLeAhAwA~DmFfDgExB{ClA{ANUUgE{@iP|HrGrB~A\",\r\n  \"ref\": \"/quotes\"\r\n}"
	  },
	  {
	    "code": 201,
	    "headers": [
	      {
	        "name": "Location",
	        "value": "https://api.drivr.com/quotes/1579823595"
	      },
	      {
	        "name": "Content-Type",
	        "value": "application/json; charset=utf-8"
	      }
	    ],
	    "body": "{\r\n  \"id\": \"1579823595\",\r\n  \"pickup\": {\r\n    \"lat\": 55.697035,\r\n    \"lng\": 12.550611\r\n  },\r\n  \"dropoff\": {\r\n    \"lat\": 55.685871,\r\n    \"lng\": 12.59477\r\n  },\r\n  \"vehicleType\": \"drivr\",\r\n  \"price\": {\r\n    \"value\": 163.63,\r\n    \"currency\": \"DKK\",\r\n    \"currencySymbol\": \"Kr.\",\r\n    \"type\": \"variable\",\r\n    \"distance\": 4186,\r\n    \"duration\": 811,\r\n    \"fees\": []\r\n  },\r\n  \"polyline\": \"ki}rIghrkAbAqDUWmDaEmCiDeBmBQU[u@lAkCz@gCdByFfAcEhA}EXc@z@sD`@}Ar@oCRo@hAaCx@yBvBkE|CgG\\\\g@|@iBV}@rDkInDmIlE}JdKkX`CeG|@wBhAcDDWDOMWc@iAIEuAuDgAoCkCwGi@cBu@yDCe@a@{BAoCGk@QcAOSa@cBy@}BuB{DC]i@y@~@}@VKLeAhAwA~DmFfDgExB{ClA{ANUUgE{@iP|HrGrB~A\",\r\n  \"ref\": \"/quotes\"\r\n}"
	  }
	]


## Operations

### <a id="authorizations" href="#authorizations">POST /authorizations</a>

Requests an authorization token by posting via basic authentication. Upon successful authentication a 256-bit token will be returned which should be used for future requests. These access tokens doesn't by default expire but it might become invalid over time. Using an invalid token in requests will result in `401 Unauthorized` responses and you should in that case re-issue a request for a new token.

#### Example request

	curl https://api.drivr.com/authorizations \
		-u john:doe -d '' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json'

#### Example response

	HTTP/1.0 201 Created
	Content-Type: application/json; charset=utf-8

	{
	    "token": "1111a3bb8d4a40f08100e640621fee10"
	}


### <a id="sessions" href="#sessions">POST /sessions</a>

Authenticates a client or corporation. Using an invalid `email` and `password` combination will result in `401 Unauthorized` response.

#### Example request

	curl https://api.drivr.com/sessions \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "email": "steve@apple.com", "password": "applesnotoranges" }'

#### Example response

	HTTP/1.0 201 Created
	Content-Type: application/json; charset=utf-8
	Location: http://api.drivr.com/sessions/2f43ca299bb6648678b2e5b41aa2aaa2

	{
	  "corporation": {
	    "id": "1580030158",
	    "createdAt": "2076-04-01T10:52:08Z",
	    "name": "Apple",
	    "card": {
	      ...
	    },
	    "homeAddress": {
	      ...
	    },
	    "vatNumber": "..."
	  },
	  "token": "2f43ca299bb6648678b2e5b41aa2aaa2"
	}


### <a id="nearby_places" href="#nearby_places">GET /places/nearby</a>

Returns list of nearby Point Of Interests (POIs). This could include airports, metro- or bus stations, restaurants and others. One POI doesn't necessarily have a category or it might have more than one category.


#### Request parameters

All parameters are optional unless specified otherwise.

`latlng` _required_ Latitude and longitude of users current location separated with comma (example `55.10,12.13`)

`radius` Radius (in meters) for nearby places. Fallbacks to a sensible default (example `25000`)

`language` ISO 639-1 language code for getting localized names for returned places (example `da`)


#### Response attributes

`ref` Unique identifier for this POI and may be used as `placeRef` attribute when [creating a booking](#post_clients_bookings).

`name` A localized name for a POI using requested language. An English name is returned in case none exists in requested language.

`distance` Calculated distance to this POI in meters.

`radius` Radius in meters of this POI.

`categories[type]` A type for this POI

<table>
	<tr>
		<th width="20%">Type</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>airport</td>
		<td>Place is an airport</td>
	</tr>
	<tr>
		<td>restaurant</td>
		<td>Place is a restaurant</td>
	</tr>
	<tr>
		<td>busStation</td>
		<td>Place is a bus station</td>
	</tr>
	<tr>
		<td>trainStation</td>
		<td>Place is a train station</td>
	</tr>
	<tr>
		<td>shopping</td>
		<td>Place is a shop</td>
	</tr>
	<tr>
		<td>other</td>
		<td>Place is uncategories</td>
	</tr>
</table>



#### Example request

	curl https://api.drivr.com/places/nearby?latlng=55.68%2C12.59&radius=1000&language=da \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json'


#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "id": "1",
	        "ref": "COPENHAGEN AIRPORT",
	        "name": "Københavns Lufthavn",
	        "location": {
	            "houseNumber": "6",
	            "streetName": "Lufthavnsboulevarden",
	            "zipCode": "2770",
	            "city": "Copenhagen",
	            "country": "DK",
	            "distance": 3241,
	            "radius": 750,
	            "lng": 12.638780000000001,
	            "lat": 55.629640000000002
	        },
	        "categories": [
	            {
	                "type": "airport",
	                "iconUrl": "http://resource.drivr.com/icons/place_type_airport@2x.png"
	            }
	        ]
	    }
	]


### <a id="get_place_by_reference" href="#get_place_by_reference">GET /places/:placeRef</a>

Returns a place based on a reference.


#### Request parameters

All parameters are optional unless specified otherwise.

`ref` _required_ String with reference of place to fetch

`language` ISO 639-1 language code for getting localized names for returned place (example `da`)

`type` Type of place to retrieve e.g. `google` if place is a Google type. The 


#### Example request

	curl https://api.drivr.com/places/CmRYAAAAciqGsTRX1mXRvuXSH2ErwW-jCINE1aLiwP64MCWDN5vkXvXoQGPKldMfmdGyqWSpm7BEYCgDm-iv7Kc2PF7QA7brMAwBbAcqMr5i1f4PwTpaovIZjysCEZTry8Ez30wpEhCNCXpynextCld2EBsDkRKsGhSLayuRyFsex6JA6NPh9dyupoTH3g&language=en&type=google \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json'


#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "ref": "CmRYAAAAciqGsTRX1mXRvuXSH2ErwW-jCINE1aLiwP64MCWDN5vkXvXoQGPKldMfmdGyqWSpm7BEYCgDm-iv7Kc2PF7QA7brMAwBbAcqMr5i1f4PwTpaovIZjysCEZTry8Ez30wpEhCNCXpynextCld2EBsDkRKsGhSLayuRyFsex6JA6NPh9dyupoTH3g",
	    "location": {
	        "streetName": "Pirrama Road",
	        "houseNumber": "48",
	        "zipCode": "2009",
	        "city": "Pyrmont",
	        "country": "AU",
	        "lat": -33.866975,
	        "lng": 151.195677
	    },
	    "type": "google"
	}


### <a id="coverages" href="#coverages">GET /coverages</a>

Returns a list of coverage for a given area grouped by vehicle type.

#### Request parameters (querystring)

`latlng` _required*_ Latitude and longitude separated with comma where coverages is requested (example `55.10,12.13`)

`language` ISO 639-1 language code for getting localized names for returned description (example `da` for Danish). Defaults to `en` (for English).

### Response attributes

`vehicleType` Vehicle type structure with details about vehicle supported for this coverage area.

`company` Company structure with details about company operating in this coverage area.

`bookingFeatures` List of features supported for this vehicle type in this coverage area.

#### Example request

	curl -X "GET" "https://api.drivr.com/coverages?latlng=55.697035%2C12.550611" \
		-H "Accept: application/vnd.clickataxi.v2+json" \
		-H "Authorization: Token token=\"11113a3bb8d4a40f08065e640621fee63\"" \
		-H "Content-Type: application/json"

#### Example response

	[
	  {
	    "vehicleType": {
	      "id": "taxi",
	      "name": "Taxi",
	      "description": "Licensed taxi",
	      "extendedDescription": "Metered and regulated taxi service. Rates depend on day/time according to the taxi regulation.",
	      "iconUrl": "https://resource-master-drivr-com.s3.amazonaws.com/properties/vehicle-types_taxi_logo_ico_vehicle_taxi_side@2x.png",
	      "seats": 4,
	      "mapIconUrl": "https://resource-master-drivr-com.s3.amazonaws.com/properties/vehicle-types_taxi_logoalternate_ico_vehicle_taxi_top@2x.png",
	      "paymentTypes": [
	        "cash"
	      ],
	      "ref": "/vehicleTypes/taxi"
	    },
	    "company": {
	      "id": "1580029131",
	      "franchiseId": "d1f49ba1827a4a6f8d59fc3cfe7607b2",
	      "name": "DRIVR Copenhagen",
	      "logoUrl": "https://resource-master-clickataxi-com.s3.amazonaws.com/companies/drivr-copenhagen_logo_drivr-logo[1].png",
	      "preBookingDestinationRequired": true,
	      "automatedBooking": true,
	      "driverComment": true,
	      "phone": "+4589885942",
	      "country": {
	        "id": "1580030083",
	        "unitSystem": "metric",
	        "code": "DK",
	        "currency": "DKK",
	        "currencySymbol": "Kr.",
	        "name": "Denmark",
	        "phonePrefix": "+45"
	      },
	      "city": "København",
	      "zipCode": "1050",
	      "utcOffset": 7200,
	      "minimumPreBooking": 3900,
	      "pickupTypes": [
	        "asap",
	        "later"
	      ],
	      "paymentTypes": [
	        "voucher",
	        "card",
	        "corporation",
	        "cash"
	      ],
	      "uid": "806f86181b0f413594c10bf434ebae8b",
	      "ref": "/companies/1580029131"
	    },
	    "bookingFeatures": [
	      {
	        "name": "Eco-friendly",
	        "iconUrl": "https://resource-master-drivr-com.s3.amazonaws.com/properties/bookingFeatureType/logo_eco@2x.png",
	        "type": "ECO",
	        "fee": 0.0,
	        "currency": "DKK",
	        "currencySymbol": "Kr.",
	        "shortName": "ec"
	      },
	      {
	        "name": "Pet",
	        "iconUrl": "https://resource-master-drivr-com.s3.amazonaws.com/properties/bookingFeatureType/logo_animal@2x.png",
	        "type": "DOMESTIC_ANIMAL",
	        "fee": 20.0,
	        "currency": "DKK",
	        "currencySymbol": "Kr.",
	        "shortName": "an"
	      },
	      {
	        "name": "Bike",
	        "iconUrl": "https://resource-master-clickataxi-com.s3.amazonaws.com/properties/services_bike_logo.png",
	        "type": "BIKE",
	        "fee": 0.0,
	        "currency": "DKK",
	        "currencySymbol": "Kr.",
	        "shortName": "bk"
	      }
	    ]
	  }
	]


### <a id="get_company" href="#get_company">GET /companies/:companyId</a>

Returns details about a specific company. Use this operation to a high-level details such as name, phone number and rating for a company. Operation will not return vehicle types, destination required details, etc since these are based on region and could be requested by using [/companies/spot](#spot_companies) instead.

#### Request parameters

`id` _required_ Id of company

#### Response attributes

`name` Name of taxi company

`phone` Number for taxi company using the [E.164 format](http://en.wikipedia.org/wiki/E.164)

`automatedBooking` True if company supports fully automated bookings through API; false otherwise

`destinationRequired` True if company requires a dropoff address for handling orders; false otherwise

`minimumPreBooking` & `maximumPreBooking` Define an interval for pre-booking, value in seconds (both can be empty)

`pickupTypes` List of possible types of pickup being either `asap`, `later` or a combination. When it's only `asap` it is not possible to pre-book on this company (possible associated with a returned vehicle type) and if it's only `later` it is not possible to do an immediate booking.

`paymentTypes` List of possible types of payment. Values are `card` (credit card payment possible), `corporation` (corporation account payment possible) or `cash` (in vehicle cash payment possible).


#### Example request

	curl https://api.drivr.com/companies/1081132172 \
		-H 'Authorization: Token token="11113a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.drivr.v2+json'

#### Example response

	{
	    "id": "1081132172",
	    "name": "Taxi4x35",
	    "logoUrl": "http://resource.drivr.com/logos/taxi4x35.png",
	    "phone": "+4535309184",
	    "rating": 4.0
	}


### <a id="spot_companies" href="#spot_companies">GET /companies/spot (obsolete)</a>

This operation is now obsolete. Use [/companies/lookup](#lookup_companies) instead.

Returns a list of taxi companies operating in specified area. List is ordered by priority and if list is empty, no taxi companies were found to operate in specified area.

#### Request parameters

`latlng` _required*_ Latitude and longitude separated with comma where taxi is requested (example `55.10,12.13`)

`language` ISO 639-1 language code for getting localized names for returned address and description (example `da` for Danish). Defaults to `en` (for English).

`country` _required*_ ISO 3166 country code where taxi is requested (example `DK`)
		
`zipCode` _required*_ Zip code where taxi is requested (example `2200`)

(*) required should be understood as (`latlng` OR (`zipCode` AND `country`))


#### Response attributes

`rating` Average rating of company as a floating point value. The rating is between 1 and 5. If no rating is returned that company doesn't have enough user ratings yet.

`eta` An estimated time of arrival for a taxi within this area at requested time.

`properties.name[vehicleTypes]` A list of available vehicle types offered by company. The `ref` attribute should be used as `vehicleType` attribute when ordering a taxi. List vary from company to company but may be cached by company. Recheck this attribute regularly since taxi companies might start to support more vehicle types. The list below shows some of the available types but this list will vary and thus you should not hardcode or limit bookings to only these types. Display them all and allow your users to select one of them.

<table>
	<tr>
		<th width="20%">Ref</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>taxi</td>
		<td>Standard taxi</td>
	</tr>
	<tr>
		<td>fourSeaterStationCar</td>
		<td>Four seater station car</td>
	</tr>
	<tr>
		<td>fiveSeater</td>
		<td>Five seater car</td>
	</tr>
	<tr>
		<td>sixSeater</td>
		<td>Six seater car</td>
	</tr>
	<tr>
		<td>sevenSeater</td>
		<td>Seven seater car</td>
	</tr>
	<tr>
		<td>executive</td>
		<td>Executive car</td>
	</tr>
	<tr>
		<td>premium</td>
		<td>Premium car</td>
	</tr>
</table>

`properties.name[services]` Additional services offered by company. Examples of services are

<table>
	<tr>
		<th width="20%">Ref</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>childSeat</td>
		<td>Child seats are available upon request</td>
	</tr>
	<tr>
		<td>bike</td>
		<td>Bikes may be brought with you</td>
	</tr>
	<tr>
		<td>animal</td>
		<td>Animals may be brought onboard</td>
	</tr>
	<tr>
		<td>wheelchair</td>
		<td>Wheelchairs may be brought onboard</td>
	</tr>
</table>

These services are dynamic and could change based on taxi company and location. You should not hardcode any of these services in your own integration but query the available list every time you want to use them.


#### Example request

	curl https://api.drivr.com/companies/spot?zipCode=2200&country=dk \
		-H 'Authorization: Token token="11113a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.drivr.v2+json'

#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "id": "42345",
	        "phone": "+4535359001",
	        "name": "Taxi 4x35",
	        "properties": [
	            {
	                "name": "vehicleTypes",
	                "values": [
	                    {
	                        "ref": "fourSeaterAny",
	                        "iconUrl": "http://resource.drivr.com/icons/four-seater-any@2x.png"
	                    },
	                    {
	                        "ref": "sixSeater",
	                        "iconUrl": "http://resource.drivr.com/icons/six-seater@2x.png"
	                    }
	                ],
	                "mutuallyExclusive": true
	            },
	            {
	                "name": "services",
	                "values": [
	                    {
	                        "ref": "childSeat",
	                        "iconUrl": "http://resource.drivr.com/icons/child_seat@2x.png"
	                    },
	                    {
	                        "ref": "animals",
	                        "iconUrl": "http://resource.drivr.com/icons/animals@2x.png"
	                    },
	                    {
	                        "ref": "bike",
	                        "iconUrl": "http://resource.drivr.com/icons/bike@2x.png"
	                    }
	                ],
	                "mutuallyExclusive": false
	            }
	        ],
	        "destinationRequired": false,
	        "automatedBooking": true,
	        "rating": 4.2,
	        "eta": 8
	    }
	]


### <a id="lookup_companies" href="#lookup_companies">GET /companies/lookup</a>

Requests address, company and address formats for a given location.


#### Request parameters

`language` ISO 639-1 language code of client (example `da` for Danish). This value is used to determine in which language addresses should be delivered.

`latlng` _required*_ Latitude and longitude of location for reverse geocoding (example `55.10,12.13`)


#### Response attributes

`place` A Place structure with address details about specified location.

`company` A Company structure with details about company operating in specified location.

`addressFormat` An AddressFormat structure with address formatting rules. This is an optional attribute which might not be available in all locations.


#### Example request

	curl https://api.staging.drivr.com/companies/lookup?language=en-GB&latlng=51%2E509205,-0%2E146690 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' 


#### Example response

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "place": {
	        "location": {
	            "streetName": "Berkeley Square",
	            "houseNumber": "45",
	            "zipCode": "W1J 5AS",
	            "city": "London",
	            "country": "GB",
	            "lat": 51.509208299999997,
	            "lng": -0.14658589999999999,
	            "formattedAddress": "Berkeley Square 45\nW1J 5AS London"
	        }
	    },
	    "company": {
	        "id": "1580030126",
	        "name": "greentomatocars",
	        "destinationRequired": true,
	        "automatedBooking": true,
	        "rating": 4.0,
	        "phone": "+442083808900",
		    "vehicleTypes": [
		      {
		        "id": "taxi",
		        "name": "4 x",
		        "description": "Licensed taxi vehicle",
		        "extendedDescription": "Metered and regulated taxi service. Rates depend on day/time according to the taxi regulation.",
		        "iconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/properties/vehicle-types_four-seater-any_logo.png",
		        "seats": 4,
		        "mapIconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/api/v2/vehicleTypes/ico_vehicle_taxi_top@2x.png"
		      },
		      {
		        "id": "drivrExecutive",
		        "name": "Executive",
		        "description": "Mercedes S-class or similar",
		        "extendedDescription": "High-end car, suited driver. Daytime / nighttime rates apply. Final price, all-inclusive, is reflected in the rates above. All vehicles and drivers are licensed.",
		        "iconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/api/v2/vehicleTypes/ico_vehicle_drivrexecutive_side@2x.png",
		        "seats": 4,
		        "mapIconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/api/v2/vehicleTypes/ico_vehicle_drivrexecutive_top@2x.png"
		      },
		      {
		        "id": "grande",
		        "name": "Executive XL",
		        "description": "6 Seater SUV or VAN",
		        "extendedDescription": "Classy MPV, suited driver. Daytime / nighttime rates apply. Final price, all-inclusive, is reflected in the rates above. All vehicles and drivers are licensed.",
		        "iconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/api/v2/vehicleTypes/ico_vehicle_grande_side@2x.png",
		        "seats": 6,
		        "mapIconUrl": "https://resource-staging-drivr-com.s3.amazonaws.com/api/v2/vehicleTypes/ico_vehicle_grande_top@2x.png"
		      }
		    ],
	        "eta": 14
	    },
	    "addressFormat": {
	    	"titleTemplate": "streetName houseNumber",
	    	"subtitleTemplate": "zipCode city",
	    	"requiredFields": ["streetName", "houseNumber", "zipCode"]
	    }
	}


### <a id="client_bookings" href="#client_bookings">GET /clients/:clientId/bookings</a>

Returns list of all bookings created by client with specified id. Only bookings for clients you have created can be requested. Requesting bookings for a client you haven't created will return a 404 error message.


#### Request parameters

All parameters are optional unless specified otherwise.

`since` Only return bookings after this timestamp (example `2012-03-24T11:00:00Z`). If not specified all bookings are returned.

`completed` A boolean value indicating if only completed bookings should be returned. If not specified all bookings are returned.


#### Response attributes

`company` Full Company object with taxi company which handled booking

`pickup` Full Location object with pickup details

`dropoff` Full Location object with dropoff details (only returned if available).

`vehicleType` Type of car which is picking up client. Valid values are returned in [companies/lookup](#lookup_companies) operations (`vehicleTypes` attribute)

`arrivalAt` Date and time taxi will arrive/arrived.

`statuses` List of statuses associated with booking. Statuses are localized based on clients language. See [clients/:clientId/bookings/:bookingId](#get_client_booking) for details about these statuses.

`state` Current state of booking as `active`, `successful` or `unsuccessful`. 


#### Example request

	curl https://api.drivr.com/clients/61241/bookings?since=2012-01-01 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-Type: application/json' 


#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "id": "1",
	        "company": {
	            "id": "1",
	            "name": "Taxa 4x35",
	            "destinationRequired": false
	        },
	        "pickup": {
	            "streetName": "Amaliegade",
	            "houseNumber": "36",
	            "city": "Copenhagen K",
	            "country": "DK",
	            "distance": 0,
	            "lat": 55.68,
	            "lng": 12.59,
	            "zipCode": "1256"
	        },
	        "dropoff": {
	            "streetName": "Jagtvej",
	            "houseNumber": "111",
	            "city": "Copenhagen N",
	            "country": "DK",
	            "distance": 0,
	            "lat": 54.310000000000002,
	            "lng": 12.199999999999999,
	            "zipCode": "2200"
	        },
	        "vehicleType": "fourSeaterAny",
	        "statuses": [
	            {
	                "createdAt": "2012-08-28T13:43:03.4005202Z",
	                "type": "executed",
	                "header": "Booking is received",
	                "body": "Your taxi will pick you up at the selected time",
	                "cancelable": true,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:42:59.5315289Z",
	                "type": "processing",
	                "header": "Processing",
	                "body": "We are processing thebooking request.",
	                "cancelable": true,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:42:48.9221817Z",
	                "type": "registered",
	                "header": "Booking is received",
	                "body": "Your taxi will pick you up at the selected time",
	                "cancelable": true,
	                "completed": false
	            }
	        ],
	        "createdAt": "2012-03-19T06:30:01.0000000Z",
	        "arrivalAt": "2012-03-20T08:00:00.0000000Z"
	    },
	    {
	        "id": "2",
	        "company": {
	            "id": "1",
	            "name": "Taxa 4x35",
	            "destinationRequired": false
	        },
	        "pickup": {
	            "streetName": "Jagtvej",
	            "houseNumber": "111",
	            "city": "Copenhagen N",
	            "country": "DK",
	            "distance": 0,
	            "lat": 54.310000000000002,
	            "lng": 12.199999999999999,
	            "zipCode": "2200"
	        },
	        "vehicleType": "fourSeaterStationCar",
	        "statuses": [
	            {
	                "createdAt": "2012-08-28T13:24:38.7074821Z",
	                "type": "paid",
	                "header": "Trip finished",
	                "body": "Thank you for your business",
	                "cancelable": true,
	                "completed": true
	            },
	            {
	                "createdAt": "2012-08-28T13:24:30.6422698Z",
	                "type": "ongoing",
	                "header": "Trip in progress",
	                "body": "Enjoy the ride",
	                "cancelable": false,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:24:22.5826582Z",
	                "type": "arrived",
	                "header": "Arrived",
	                "body": "Taxa 4 x 35 has arrived!",
	                "cancelable": false,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:24:14.5090438Z",
	                "type": "arrival_time",
	                "header": "Status update",
	                "body": "Updated arrival time is approx: 5 min.",
	                "cancelable": false,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:24:07.0955614Z",
	                "type": "accepted",
	                "header": "Booking is received",
	                "body": "Your taxi will pick you up at the selected time",
	                "cancelable": false,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:24:00.0468327Z",
	                "type": "executed",
	                "header": "Booking is received",
	                "body": "Your taxi will pick you up at the selected time",
	                "cancelable": true,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:23:56.3048576Z",
	                "type": "processing",
	                "header": "Processing",
	                "body": "We are processing the booking request.",
	                "cancelable": true,
	                "completed": false
	            },
	            {
	                "createdAt": "2012-08-28T13:23:44.7758936Z",
	                "type": "registered",
	                "header": "Booking is received",
	                "body": "Your taxi will pick you up at the selected time",
	                "cancelable": true,
	                "completed": false
	            }
	        ],
	        "createdAt": "2012-03-12T19:05:32.0000000Z",
	        "arrivalAt": "2012-03-12T19:05:32.0000000Z"
	    }
	]


### <a id="clients" href="#clients">POST /clients</a>

Creates a new client which is necessary before a booking can be created. A client must have at least a name and phone number. Both name and phone number might be forwarded to a taxi company in order to book taxis, so make sure your privacy policy tells your users about it. An SMS might be sent to client for verification purposes.

#### Request attributes

`name` Name of client which could be forwarded to taxi company. This is usually a last name but could be a full name too.

`phone` Number of client formatted as a [E.164](http://en.wikipedia.org/wiki/E.164) number.

`language` ISO 639-1 language code of client (example `da` for Danish). This value is used to determine in which language status messages should be delivered. Supported values are ["en", "es", "de", "it", "fr", "pt", "da", "sv", "nb"].

`os` Type of OS used by client, if applicable. Types can be either 'ios', 'android' or 'windowsPhone'.

`guid` Optional. A unique identifier for client which you may use internally since it will never change. If not specified you should read the returned `guid` attribute in response payload and store that when updating clients.

`pushToken` A string with a token used for sending push notifications to clients. This parameter is optional. 


#### Example request

	curl https://api.drivr.com/clients \
		-H 'Authorization: Token token="1993a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "name": "Doe", "phone": "+4561616161", "os": "android" }'

#### Example response

	HTTP/1.1 201 Created
	Content-Type: application/json; charset=utf-8

	{
	    "createdAt": "2012-09-05T12:11:01.9744265Z",
	    "id": "61449",
	    "name": "Doe",
	    "phone": "+4561616161",
	    "language": "en",
	    "os": "android",
	    "guid": "1003a3bb8d4a40f08065e640621fee63"
	}


### <a id="client_by_id" href="#client_by_id">GET /clients/:clientId</a>

Get details about an existing client. Only clients created by you can be fetched. Requesting other clients will return in a 404 Not Found response.

#### Example request

	curl https://api.drivr.com/clients/61449
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.drivr.v2+json'

#### Example response

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "createdAt": "2012-09-05T12:11:01.9744265Z",
	    "id": "61449",
	    "name": "Doe",
	    "phone": "+4561616161",
	    "language": "en",
	    "email": "john@doe.com",
	    "guid": "1003a3bb8d4a40f08065e640621fee63"
	}

### <a id="update_client" href="#update_client">PUT /clients/:clientId</a>

Updates an existing client. Only clients created by you can be updated.

#### Example request

	curl https://api.drivr.com/clients/61449
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.drivr.v2+json'
		-H 'Content-type: application/json'
		-d '{ "firstName": "Steve", "phone": "+4561715099" }'
		-X PUT

#### Example response

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "createdAt": "2012-09-05T12:11:01.9744265Z",
	    "id": "61449",
	    "firstName": "Steve",
	    "lastName": "Jobs",
	    "phone": "+4561715099",
	    "language": "en",
	    "email": "john@doe.com",
	    "guid": "1003a3bb8d4a40f08065e640621fee63",
	    "pushToken": "07a50578c0c94d0da276aab1a9c9166b"
	}


### <a id="post_clients_bookings" href="#post_clients_bookings">POST /clients/:clientId/bookings</a>

Requests a new taxi booking for an existing [client](#clients). An immediate booking is requested by excluding `arrivalAt` attribute from body. A pre-booking should always have an `arrivalAt` in its body and it needs to be in the future. Notice, some taxi companies require a destination address so a call to [/companies/spot](#spot_companies) should be made before booking in a new area. Check `destinationRequired` attribute on response to find out if a destination address is required in that area.

A success response means our system has accepted your request for a taxi but not necessarily that this taxi will be booked. Since we are connecting to real taxi companies around the globe it sometimes will not be possible to fulfill the order. You should check the order status by calling [/clients/:clientId/bookings/:bookingsId](#get_client_booking) regularly e.g. every 15 or 30 second.


#### Request attributes

`pickup` Pickup details

`dropoff` Dropoff details

`vehicleType` Type of car which is requested. Valid values are returned in [companies/spot](#spot_companies) operations (`vehicleTypes` attribute)

`arrivalAt` Optional date and time for when taxi is requested. If not specified a taxi is immediately requested.

`comment` An optional comment for driver to see.

`quoteId` An optional reference to a previously requested [trip quote](#quotes).

`corporationId` An optional reference to a corporation in case this client is associated with an existing corporation.

`payee` An optional string indicating if referenced corporation should pay for this booking of if client should pay. Value must be either `client` or `corporation`.

`voucherId` An optional reference to a voucher.


#### Example request

Creates a pre-booking of a medium sized taxi for Christmas Eve. A `pickup` structure may include a `placeRef` attribute instead of a full address in case you want to book at a [Point Of Interest](#nearby_places). Client is bringing a child so a child seat is requested as a service too.

	curl https://api.drivr.com/clients/9535253/bookings \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "arrivalAt": "2012-12-24T20:00:00.0000000Z", "pickup":{"streetName":"Amaliegade","houseNumber":"36","zipCode":"1256","city":"Copenhagen K","country":"DK","lat":55.68,"lng":12.59}, "vehicleType": "fourSeaterAny", "services": "childSeat" }'

#### Example response

	HTTP/1.0 201 Created
	Location: https://api.drivr.com/clients/9535253/bookings/483233
	Content-Type: application/json; charset=utf-8

	{
	    "id": "483233",
	    "clientId": "9535253",
	    "vehicleType": "fourSeaterAny",
	    "arrivalAt": "2012-12-24T20:00:00.0000000Z",
	    "company": {
	        "id": "1",
	        "name": "Taxa 4x35",
	        "destinationRequired": false,
	        "phone": "+4535309184"
	    },
	    "pickup": {
	        "lng": 12.59,
	        "country": "DK",
	        "zipCode": "1256",
	        "city": "Copenhagen K",
	        "lat": 55.68,
            "streetName": "Amaliegade",
            "houseNumber": "36",
	    },
	    "services": [
	    	"childSeat"
	    ]
	    "createdAt": "2012-04-09T20:17:16.1958490Z"
	}

<!--
#### Request example with place as dropoff

TODO: show example where "pickup" is replaced with "places" reference

#### Response example with place as dropoff

TODO: show example where "pickup" is replaced with "places" reference
-->


### <a id="get_client_booking" href="#get_client_booking">GET /clients/:clientId/bookings/:id</a>

Returns full details about an existing booking created by client.


#### Response attributes

`completed` True if booking may be considered as completed i.e. a client has completed trip or trip has failed.

`state` Current state of booking as `active`, `successful` or `unsuccessful`. 

`statuses` List of statuses associated with booking. Statuses are localized based on client language.

`statuses[id]` Unique id of booking status returned. May be used by you to check if a given status has been handled/displayed for users.

`statuses[createdAt]` Timestamp of when booking status has been created.

`statuses[type]` Type of status returned. Might be one of the below mentioned.

<table>
	<tr>
		<th width="20%">Type</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>registered</td>
		<td>Order has been registered</td>
	</tr>
	<tr>
		<td>dispatched</td>
		<td>Taxi has been dispatched</td>
	</tr>
	<tr>
		<td>arrived</td>
		<td>Taxi has arrived</td>
	</tr>
	<tr>
		<td>passengerOnBoard</td>
		<td>Taxi is driving with passenger</td>
	</tr>
	<tr>
		<td>cancelled</td>
		<td>Order has been cancelled</td>
	</tr>
	<tr>
		<td>noShow</td>
		<td>Taxi driver reported customer did not show up</td>
	</tr>
	<tr>
		<td>completed</td>
		<td>Order has been completed</td>
	</tr>
	<tr>
		<td>delayed</td>
		<td>Taxi is delayed and will arrive later than originally estimated</td>
	</tr>
	<tr>
		<td>failed</td>
		<td>Order failed and <i>no taxi will arrive</i>. This might happen for a number of reasons but primarly if a booking is tried to be placed automatically but fails because of invalid address or external system error.</td>
	</tr>
	<tr>
		<td>processing</td>
		<td>Order is being processed by external systems</td>
	</tr>
	<tr>
		<td>arrivalTime</td>
		<td>Taxi has responded with an updated arrival time</td>
	</tr>
	<tr>
		<td>executed</td>
		<td>Order has been dispatched to external systems</td>
	</tr>
	<tr>
		<td>unfulfilled</td>
		<td>Order was not successfully completed and <i>no taxi will arrive</i>. This might happen when taxi company doesn't pick up their phone, they are too busy to process more orders at the moment or address/area is illegal for some reason. If you are certain on address you could retry request within a couple of minutes.</td>
	</tr>
	<tr>
		<td>assigned</td>
		<td>Order is being manually assigned</td>
	</tr>
	<tr>
		<td>lookOut</td>
		<td>Taxi is about to arrive and customer should get ready for it</td>
	</tr>
	<tr>
		<td>callAssist</td>
		<td>Order requires additional information and customer should call taxi company</td>
	</tr>
</table>

`statuses[header]` Short description of status. Localized in clients language.

`statuses[body]` Longer description including details of status. Localized in clients language.

`statuses[cancelable]` True if a booking is still valid to cancel; false otherwise. Is `true` initially and changes to `false` once a car has been dispatched or if isn't possible to cancel a placed booking.


#### Example request

	curl https://api.drivr.com/clients/61241/bookings/923880 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json'

#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "id": "923880",
	    "clientId": "61241",
	    "company": {
	        "id": "2",
	        "name": "Taxa4 x 35",
	        "properties": [
	            {
	                "name": "vehicleTypes",
	                "values": [
	                    {
	                        "ref": "fourSeaterAny",
	                        "iconUrl": "http://resource.staging.drivr.com/properties/vehicle-types_four-seater-any_logo.png"
	                    },
	                    {
	                        "ref": "fourSeaterStationCar",
	                        "iconUrl": "http://resource.staging.drivr.com/properties/vehicle-types_four-seater-station-car_logo.png"
	                    }
	                ],
	                "mutuallyExclusive": true
	            }
	        ],
	        "rating": 4.0,
	        "phone": "+4535309184"
	    },
	    "pickup": {
            "streetName": "Amaliegade",
            "houseNumber": "36",
	        "city": "Copenhagen K",
	        "country": "DK",
	        "distance": 0,
	        "lat": 55.68,
	        "lng": 12.59,
	        "zipCode": "1256"
	    },
	    "vehicleType": "fourSeaterAny",
        "statuses": [
            {
                "createdAt": "2012-03-19T13:24:00.0468327Z",
                "type": "executed",
                "header": "Booking is received",
                "body": "Your taxi will pick you up at the selected time",
                "cancelable": true
            },
            {
                "createdAt": "2012-03-19T13:23:56.3048576Z",
                "type": "processing",
                "header": "Processing",
                "body": "We are processing the booking request.",
                "cancelable": true
            },
            {
                "createdAt": "2012-03-19T13:23:44.7758936Z",
                "type": "registered",
                "header": "Booking is received",
                "body": "Your taxi will pick you up at the selected time",
                "cancelable": true
            }
        ],
	    "createdAt": "2012-03-19T06:30:01.0000000Z",
	    "arrivalAt": "2012-03-20T08:00:00.0000000Z",
	    "state": "active"
	}


### <a id="cancel_booking" href="#cancel_booking">DELETE /clients/:clientId/bookings/:bookingId</a>

Cancels a booking. The method might fail with a `400 Bad Request` response in case a booking can't be canceled e.g. if taxi is already on its way or at the pickup address. A `422 Unprocessed Entity` response is returned in case specified clientId or bookingId isn't valid.


#### Example request

	curl https://api.drivr.com/clients/61241/bookings/923878 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-X DELETE

#### Example response

When booking has been canceled successfully

	HTTP/1.1 200 OK


### <a id="rating" href="#rating">POST /ratings</a>

Rates a current booking. The `clientId`, `bookingId` and `stars` attributes are required. Attribute `stars` must be a valid integer between 1 and 5 (both inclusive).

#### Example request

	curl https://api.drivr.com/ratings \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "clientId": "1543263", "bookingId": "5436443", "stars": 4, "review": "Funny driver" }'

#### Example response

	HTTP/1.1 201 Created
	Location: https://api.drivr.com/ratings/9dc8bb819c2a4cc4990a9f93be78669e
	Content-Type: application/json; charset=utf-8

	{
		"id": "9dc8bb819c2a4cc4990a9f93be78669e",
		"clientId": "1543263",
		"bookingId": "5436443",
	    "stars": 4,
	    "review": "Funny driver"
	}


### <a id="search_places" href="#search_places">GET /places/search</a>

Searches for addresses or POIs within a given radius. More details can be fetched for a specific result using [/places](#get_place_by_reference).


#### Request parameters

`query` _required*_ String with value to be searched (example `jagtvej`)

`language` ISO 639-1 language code for getting localized names for returned results (example `da`). Defaults to `en`

`latlng` Latitude and longitude of origin (example `55.10,12.13`)

`radius` Radius of results to be returned. Only used if `latlng` is specified. Defaults to `5000` (meters)


#### Response parameters

`ref` Unique reference of Place.

`categories` Array of categories.

`type` Type of place. Currently supports `internal` and `google`.


#### Example request

	curl https://api.drivr.com/places/search?latlng=55.10,12.13&keyword=jagtvej \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' 


#### Example response

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "ref": "ClRKAAAABUa0QH9RPJ4TyqNrPHPPRXcL1Yn9VBGJKFoJzvB0FC2vym_qmQwPQmytzxAXeZtFapkP9VjJ0tK0aRJByGYXCLWLU8mCPxMNF-KdWHDZTK0SEFWe8MTDCECOZD4pvlLoP8UaFAcj8JzkrZUqBXD5tZ6zX9a3pda7",
	        "name": "Jagtvej, Copenhagen, Denmark",
	        "categories": [
	            {
	                "type": "route"
	            },
	            {
	                "type": "geocode"
	            }
	        ],
	        "type": "google"
	    },
	    {
	        "ref": "ClRIAAAAUj6yruZLNXb-3_Qh8RBpnI574KGYfuPCGFFFkBit0Bztz8HcJQziEjSUU6bS4NseS16F2-mzHqkoEM-5Bw7HOUEo3amrS9HIXcK0alu4ooISEMP5GwkWAxDjWM61fxXHIgcaFPIC1p6JqjC46AEujAQjeF5MjopQ",
	        "name": "Jagtvej, Naestved, Denmark",
	        "categories": [
	            {
	                "type": "route"
	            },
	            {
	                "type": "geocode"
	            }
	        ],
	        "type": "google"
	    }
	]


### <a id="quotes" href="#quotes">POST /quotes</a>

Requests a price quote from `pickup` to `dropoff`. 


#### Request parameters

`pickup` _required_* Location structure with pickup details

`dropoff` _required_* Location structure with dropoff details

`vehicleType` _required_* Type of car which is picking up client. Valid values are returned in [companies/spot](#spot_companies) operations (`vehicleTypes` attribute)

`quotingAt` Optional time of price quote. Prices might vary over time e.g. it might be more expensive to order a car friday evening than wednesday at noon


#### Response parameters

`ref` Unique reference of Place.

`categories` Array of categories.

`type` Type of place. Currently supports `internal` and `google`.


#### Example request

Requesting a price quote from Amaliegade to Jagtvej on Christmas Eve.

	curl https://api.drivr.com/quotes \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "quotingAt": "2012-12-24T20:00:00.0000000Z", "pickup":{"streetName":"Amaliegade","houseNumber":"36","zipCode":"1256","city":"Copenhagen K","country":"DK","lat":55.68,"lng":12.59}, "dropoff":{"streetName":"Jagtvej","houseNumber":"111","zipCode":"2200","city":"Copenhagen N","country":"DK","lat":55.696406,"lng":12.550911}, "vehicleType": "taxi" }'


#### Example response

	{
  	  "id": "158afb14172",
	  "pickup": {
	    "streetName": "Amaliegade",
	    "houseNumber": "36",
	    "zipCode": "1256",
	    "city": "Copenhagen K",
	    "country": "DK",
	    "lat": 55.68,
	    "lng": 12.59
	  },
	  "dropoff": {
	    "streetName": "Jagtvej",
	    "houseNumber": "111",
	    "zipCode": "2200",
	    "city": "Copenhagen N",
	    "country": "DK",
	    "lat": 55.696406,
	    "lng": 12.550911
	  },
	  "vehicleType": "taxi",
	  "price": {
	    "value": 179.0,
	    "currency": "Kr."
	  },
	  "quotingAt": "2012-12-24T20:00:00Z",
	  "language": "en"
	}


### <a id="nearby_vehicles" href="#nearby_vehicles">GET /nearbyVehicles</a>

Requests nearby vehicles.


#### Request parameters

`latlng` _required_ Latitude and longitude of users current location separated with comma (example `55.10,12.13`)


#### Response parameters

`vehicleType` Vehicle type identifier

`vehiclePaths` Array of vehicle paths for vehicle type.

`vehiclePaths.id` Unique identifier of a specific vehicle.

`vehiclePaths.positions` Array of latest reported positions for specific vehicle.


#### Example request

Requesting nearby vehicles near Jagtvej 111

	curl https://api.drivr.com/nearbyVehicles?latlng=55.696572,12.550958 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' \
		-H 'Content-type: application/json' \


#### Example response

	[
	  {
	    "vehicleType": "taxi",
	    "vehiclePaths": [
	      {
	        "id": "f1b1bce99e5941f6af3d471f665f6a8f",
	        "positions": [
	          {
	            "id": "1605954987",
	            "createdAt": "2015-01-05T10:13:39Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954901",
	            "createdAt": "2015-01-05T10:13:44Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954887",
	            "createdAt": "2015-01-05T10:13:49Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954929",
	            "createdAt": "2015-01-05T10:13:54Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954915",
	            "createdAt": "2015-01-05T10:13:59Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954829",
	            "createdAt": "2015-01-05T10:14:04Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954879",
	            "createdAt": "2015-01-05T10:14:09Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605954857",
	            "createdAt": "2015-01-05T10:14:14Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605955288",
	            "createdAt": "2015-01-05T10:14:19Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          },
	          {
	            "id": "1605955269",
	            "createdAt": "2015-01-05T10:14:24Z",
	            "lat": 55.6992138736842,
	            "lng": 12.554428378947364,
	            "course": 233.0,
	            "speed": 0.0
	          }
	        ]
	      }
	    ]
	  },
	  {
	    "vehicleType": "drivrExecutive",
	    "vehiclePaths": [
	      {
	        "id": "b09d3df002fd4d79ae49a87eb1b02ec7",
	        "positions": [
	          {
	            "id": "1605954996",
	            "createdAt": "2015-01-05T10:13:35Z",
	            "lat": 55.688852586363637,
	            "lng": 12.576425895454552,
	            "course": 116.0,
	            "speed": 28.93
	          },
	          {
	            "id": "1605954982",
	            "createdAt": "2015-01-05T10:13:40Z",
	            "lat": 55.68863861818182,
	            "lng": 12.57697647272728,
	            "course": 116.0,
	            "speed": 29.48
	          },
	          {
	            "id": "1605954896",
	            "createdAt": "2015-01-05T10:13:45Z",
	            "lat": 55.68842465,
	            "lng": 12.577527050000008,
	            "course": 116.0,
	            "speed": 29.76
	          },
	          {
	            "id": "1605954882",
	            "createdAt": "2015-01-05T10:13:50Z",
	            "lat": 55.688210681818184,
	            "lng": 12.578077627272735,
	            "course": 116.0,
	            "speed": 29.48
	          },
	          {
	            "id": "1605954924",
	            "createdAt": "2015-01-05T10:13:55Z",
	            "lat": 55.687996713636366,
	            "lng": 12.578628204545463,
	            "course": 116.0,
	            "speed": 29.33
	          },
	          {
	            "id": "1605954846",
	            "createdAt": "2015-01-05T10:14:00Z",
	            "lat": 55.687782745454548,
	            "lng": 12.579178781818191,
	            "course": 116.0,
	            "speed": 29.46
	          },
	          {
	            "id": "1605954824",
	            "createdAt": "2015-01-05T10:14:05Z",
	            "lat": 55.68756877727273,
	            "lng": 12.579729359090919,
	            "course": 116.0,
	            "speed": 29.78
	          },
	          {
	            "id": "1605954874",
	            "createdAt": "2015-01-05T10:14:10Z",
	            "lat": 55.687354809090913,
	            "lng": 12.580279936363647,
	            "course": 116.0,
	            "speed": 29.77
	          },
	          {
	            "id": "1605954852",
	            "createdAt": "2015-01-05T10:14:15Z",
	            "lat": 55.687140840909095,
	            "lng": 12.580830513636375,
	            "course": 116.0,
	            "speed": 29.47
	          },
	          {
	            "id": "1605955286",
	            "createdAt": "2015-01-05T10:14:20Z",
	            "lat": 55.686926872727277,
	            "lng": 12.581381090909103,
	            "course": 116.0,
	            "speed": 29.2
	          }
	        ]
	      },
	      {
	        "id": "a5734758d8fd4f49a1107c9f150a61bf",
	        "positions": [
	          {
	            "id": "1605954994",
	            "createdAt": "2015-01-05T10:13:38Z",
	            "lat": 55.693975099999996,
	            "lng": 12.583690399999995,
	            "course": 125.0,
	            "speed": 31.97
	          },
	          {
	            "id": "1605954911",
	            "createdAt": "2015-01-05T10:13:43Z",
	            "lat": 55.6936876,
	            "lng": 12.584183149999994,
	            "course": 125.0,
	            "speed": 31.59
	          },
	          {
	            "id": "1605954889",
	            "createdAt": "2015-01-05T10:13:48Z",
	            "lat": 55.6934001,
	            "lng": 12.5846759,
	            "course": 125.0,
	            "speed": 31.48
	          },
	          {
	            "id": "1605954936",
	            "createdAt": "2015-01-05T10:13:53Z",
	            "lat": 55.693157587499996,
	            "lng": 12.585094400000001,
	            "course": 125.0,
	            "speed": 26.88
	          },
	          {
	            "id": "1605954922",
	            "createdAt": "2015-01-05T10:13:58Z",
	            "lat": 55.692915074999995,
	            "lng": 12.585512900000001,
	            "course": 125.0,
	            "speed": 26.59
	          },
	          {
	            "id": "1605954839",
	            "createdAt": "2015-01-05T10:14:03Z",
	            "lat": 55.692672562499993,
	            "lng": 12.585931400000002,
	            "course": 125.0,
	            "speed": 26.45
	          },
	          {
	            "id": "1605954816",
	            "createdAt": "2015-01-05T10:14:08Z",
	            "lat": 55.692430049999992,
	            "lng": 12.586349900000002,
	            "course": 125.0,
	            "speed": 26.25
	          },
	          {
	            "id": "1605954867",
	            "createdAt": "2015-01-05T10:14:13Z",
	            "lat": 55.69218753749999,
	            "lng": 12.586768400000002,
	            "course": 125.0,
	            "speed": 27.24
	          },
	          {
	            "id": "1605955292",
	            "createdAt": "2015-01-05T10:14:18Z",
	            "lat": 55.691945024999988,
	            "lng": 12.587186900000003,
	            "course": 125.0,
	            "speed": 26.09
	          },
	          {
	            "id": "1605955278",
	            "createdAt": "2015-01-05T10:14:23Z",
	            "lat": 55.691702512499987,
	            "lng": 12.587605400000003,
	            "course": 125.0,
	            "speed": 26.61
	          }
	        ]
	      },
	      {
	        "id": "f5b39d7b1e034a03a0ac00eb170f06fb",
	        "positions": [
	          {
	            "id": "1605954980",
	            "createdAt": "2015-01-05T10:13:39Z",
	            "lat": 55.676689299999992,
	            "lng": 12.578219550000002,
	            "course": 71.0,
	            "speed": 31.25
	          },
	          {
	            "id": "1605954902",
	            "createdAt": "2015-01-05T10:13:44Z",
	            "lat": 55.676928266666657,
	            "lng": 12.578775366666669,
	            "course": 71.0,
	            "speed": 30.89
	          },
	          {
	            "id": "1605954880",
	            "createdAt": "2015-01-05T10:13:49Z",
	            "lat": 55.677167233333321,
	            "lng": 12.579331183333336,
	            "course": 71.0,
	            "speed": 30.77
	          },
	          {
	            "id": "1605954930",
	            "createdAt": "2015-01-05T10:13:54Z",
	            "lat": 55.6774062,
	            "lng": 12.579887,
	            "course": 19.0,
	            "speed": 30.85
	          },
	          {
	            "id": "1605954844",
	            "createdAt": "2015-01-05T10:13:59Z",
	            "lat": 55.677567499999995,
	            "lng": 12.57992755,
	            "course": 19.0,
	            "speed": 13.1
	          },
	          {
	            "id": "1605954830",
	            "createdAt": "2015-01-05T10:14:04Z",
	            "lat": 55.6777288,
	            "lng": 12.5799681,
	            "course": 113.0,
	            "speed": 12.93
	          },
	          {
	            "id": "1605954872",
	            "createdAt": "2015-01-05T10:14:09Z",
	            "lat": 55.6775478,
	            "lng": 12.580517166666667,
	            "course": 113.0,
	            "speed": 28.31
	          },
	          {
	            "id": "1605954858",
	            "createdAt": "2015-01-05T10:14:14Z",
	            "lat": 55.6773668,
	            "lng": 12.581066233333333,
	            "course": 113.0,
	            "speed": 27.89
	          },
	          {
	            "id": "1605955284",
	            "createdAt": "2015-01-05T10:14:19Z",
	            "lat": 55.6771858,
	            "lng": 12.5816153,
	            "course": 72.0,
	            "speed": 28.05
	          },
	          {
	            "id": "1605955270",
	            "createdAt": "2015-01-05T10:14:24Z",
	            "lat": 55.677285166666664,
	            "lng": 12.581857233333333,
	            "course": 72.0,
	            "speed": 13.0
	          }
	        ]
	      },
	      {
	        "id": "8a147d6285084ac2aa04ec725c5d28ae",
	        "positions": [
	          {
	            "id": "1605954993",
	            "createdAt": "2015-01-05T10:13:37Z",
	            "lat": 55.665939800000004,
	            "lng": 12.55937685,
	            "course": 56.0,
	            "speed": 31.75
	          },
	          {
	            "id": "1605954979",
	            "createdAt": "2015-01-05T10:13:42Z",
	            "lat": 55.666273100000005,
	            "lng": 12.559797575,
	            "course": 56.0,
	            "speed": 32.3
	          },
	          {
	            "id": "1605954893",
	            "createdAt": "2015-01-05T10:13:47Z",
	            "lat": 55.666606400000006,
	            "lng": 12.560218299999999,
	            "course": 56.0,
	            "speed": 32.74
	          },
	          {
	            "id": "1605954943",
	            "createdAt": "2015-01-05T10:13:52Z",
	            "lat": 55.666939700000007,
	            "lng": 12.560639024999999,
	            "course": 56.0,
	            "speed": 32.33
	          },
	          {
	            "id": "1605954921",
	            "createdAt": "2015-01-05T10:13:57Z",
	            "lat": 55.667273000000009,
	            "lng": 12.561059749999998,
	            "course": 56.0,
	            "speed": 32.21
	          },
	          {
	            "id": "1605954843",
	            "createdAt": "2015-01-05T10:14:02Z",
	            "lat": 55.66760630000001,
	            "lng": 12.561480474999998,
	            "course": 56.0,
	            "speed": 32.33
	          },
	          {
	            "id": "1605954821",
	            "createdAt": "2015-01-05T10:14:07Z",
	            "lat": 55.667939600000011,
	            "lng": 12.561901199999998,
	            "course": 56.0,
	            "speed": 32.68
	          },
	          {
	            "id": "1605954871",
	            "createdAt": "2015-01-05T10:14:12Z",
	            "lat": 55.668272900000012,
	            "lng": 12.562321924999997,
	            "course": 56.0,
	            "speed": 32.67
	          },
	          {
	            "id": "1605954849",
	            "createdAt": "2015-01-05T10:14:17Z",
	            "lat": 55.668606200000013,
	            "lng": 12.562742649999997,
	            "course": 56.0,
	            "speed": 32.2
	          },
	          {
	            "id": "1605955283",
	            "createdAt": "2015-01-05T10:14:22Z",
	            "lat": 55.668939500000015,
	            "lng": 12.563163374999997,
	            "course": 56.0,
	            "speed": 32.34
	          }
	        ]
	      },
	      {
	        "id": "c3018f4ddffa4a66a0f461a0fe1d3612",
	        "positions": [
	          {
	            "id": "1605954986",
	            "createdAt": "2015-01-05T10:13:39Z",
	            "lat": 55.667002957377179,
	            "lng": 12.568385452458974,
	            "course": 71.0,
	            "speed": 38.1
	          },
	          {
	            "id": "1605954900",
	            "createdAt": "2015-01-05T10:13:44Z",
	            "lat": 55.667298944262427,
	            "lng": 12.569056014754056,
	            "course": 71.0,
	            "speed": 37.38
	          },
	          {
	            "id": "1605954886",
	            "createdAt": "2015-01-05T10:13:49Z",
	            "lat": 55.667594931147676,
	            "lng": 12.569726577049137,
	            "course": 71.0,
	            "speed": 39.05
	          },
	          {
	            "id": "1605954928",
	            "createdAt": "2015-01-05T10:13:54Z",
	            "lat": 55.667890918032924,
	            "lng": 12.570397139344218,
	            "course": 71.0,
	            "speed": 38.02
	          },
	          {
	            "id": "1605954914",
	            "createdAt": "2015-01-05T10:13:59Z",
	            "lat": 55.668186904918173,
	            "lng": 12.571067701639299,
	            "course": 71.0,
	            "speed": 37.21
	          },
	          {
	            "id": "1605954828",
	            "createdAt": "2015-01-05T10:14:04Z",
	            "lat": 55.668482891803421,
	            "lng": 12.57173826393438,
	            "course": 71.0,
	            "speed": 38.36
	          },
	          {
	            "id": "1605954878",
	            "createdAt": "2015-01-05T10:14:09Z",
	            "lat": 55.66877887868867,
	            "lng": 12.572408826229461,
	            "course": 71.0,
	            "speed": 38.48
	          },
	          {
	            "id": "1605954856",
	            "createdAt": "2015-01-05T10:14:14Z",
	            "lat": 55.669074865573918,
	            "lng": 12.573079388524542,
	            "course": 71.0,
	            "speed": 37.94
	          },
	          {
	            "id": "1605955290",
	            "createdAt": "2015-01-05T10:14:19Z",
	            "lat": 55.669370852459167,
	            "lng": 12.573749950819623,
	            "course": 71.0,
	            "speed": 38.05
	          },
	          {
	            "id": "1605955268",
	            "createdAt": "2015-01-05T10:14:24Z",
	            "lat": 55.669666839344416,
	            "lng": 12.574420513114704,
	            "course": 71.0,
	            "speed": 38.49
	          }
	        ]
	      },
	      {
	        "id": "101ea44f4fa24f69a4aeaf9a5f09f769",
	        "positions": [
	          {
	            "id": "1605954998",
	            "createdAt": "2015-01-05T10:13:36Z",
	            "lat": 55.667541366666669,
	            "lng": 12.544077599999993,
	            "course": 206.0,
	            "speed": 32.71
	          },
	          {
	            "id": "1605954976",
	            "createdAt": "2015-01-05T10:13:41Z",
	            "lat": 55.667137493333335,
	            "lng": 12.543922219999992,
	            "course": 206.0,
	            "speed": 32.33
	          },
	          {
	            "id": "1605954898",
	            "createdAt": "2015-01-05T10:13:46Z",
	            "lat": 55.66673362,
	            "lng": 12.543766839999991,
	            "course": 206.0,
	            "speed": 32.68
	          },
	          {
	            "id": "1605954940",
	            "createdAt": "2015-01-05T10:13:51Z",
	            "lat": 55.666329746666669,
	            "lng": 12.54361145999999,
	            "course": 206.0,
	            "speed": 32.19
	          },
	          {
	            "id": "1605954926",
	            "createdAt": "2015-01-05T10:13:56Z",
	            "lat": 55.665925873333336,
	            "lng": 12.54345607999999,
	            "course": 206.0,
	            "speed": 32.36
	          },
	          {
	            "id": "1605954840",
	            "createdAt": "2015-01-05T10:14:01Z",
	            "lat": 55.665522,
	            "lng": 12.543300699999989,
	            "course": 206.0,
	            "speed": 32.67
	          },
	          {
	            "id": "1605954826",
	            "createdAt": "2015-01-05T10:14:06Z",
	            "lat": 55.66511812666667,
	            "lng": 12.543145319999988,
	            "course": 206.0,
	            "speed": 32.3
	          },
	          {
	            "id": "1605954868",
	            "createdAt": "2015-01-05T10:14:11Z",
	            "lat": 55.664714253333337,
	            "lng": 12.542989939999988,
	            "course": 206.0,
	            "speed": 31.8
	          },
	          {
	            "id": "1605954854",
	            "createdAt": "2015-01-05T10:14:16Z",
	            "lat": 55.66431038,
	            "lng": 12.542834559999987,
	            "course": 206.0,
	            "speed": 32.35
	          },
	          {
	            "id": "1605955280",
	            "createdAt": "2015-01-05T10:14:21Z",
	            "lat": 55.66390650666667,
	            "lng": 12.542679179999986,
	            "course": 206.0,
	            "speed": 32.98
	          }
	        ]
	      }
	    ]
	  },
	  {
	    "vehicleType": "drivr",
	    "vehiclePaths": [
	      {
	        "id": "61b1433a24444decb7afa39040558f83",
	        "positions": [
	          {
	            "id": "1605954997",
	            "createdAt": "2015-01-05T10:13:35Z",
	            "lat": 55.675252544444454,
	            "lng": 12.586042944444445,
	            "course": 112.0,
	            "speed": 24.67
	          },
	          {
	            "id": "1605954983",
	            "createdAt": "2015-01-05T10:13:40Z",
	            "lat": 55.6750991,
	            "lng": 12.586526,
	            "course": 125.0,
	            "speed": 24.45
	          },
	          {
	            "id": "1605954897",
	            "createdAt": "2015-01-05T10:13:45Z",
	            "lat": 55.6744768,
	            "lng": 12.5875852,
	            "course": 127.0,
	            "speed": 68.85
	          },
	          {
	            "id": "1605954883",
	            "createdAt": "2015-01-05T10:13:50Z",
	            "lat": 55.67414996875,
	            "lng": 12.588089437499999,
	            "course": 127.0,
	            "speed": 34.39
	          },
	          {
	            "id": "1605954925",
	            "createdAt": "2015-01-05T10:13:55Z",
	            "lat": 55.6738231375,
	            "lng": 12.588593674999998,
	            "course": 127.0,
	            "speed": 34.48
	          },
	          {
	            "id": "1605954847",
	            "createdAt": "2015-01-05T10:14:00Z",
	            "lat": 55.67349630625,
	            "lng": 12.589097912499998,
	            "course": 127.0,
	            "speed": 34.91
	          },
	          {
	            "id": "1605954825",
	            "createdAt": "2015-01-05T10:14:05Z",
	            "lat": 55.673169475,
	            "lng": 12.589602149999997,
	            "course": 127.0,
	            "speed": 34.49
	          },
	          {
	            "id": "1605954875",
	            "createdAt": "2015-01-05T10:14:10Z",
	            "lat": 55.67284264375,
	            "lng": 12.590106387499997,
	            "course": 127.0,
	            "speed": 33.71
	          },
	          {
	            "id": "1605954853",
	            "createdAt": "2015-01-05T10:14:15Z",
	            "lat": 55.6725158125,
	            "lng": 12.590610624999997,
	            "course": 127.0,
	            "speed": 34.66
	          },
	          {
	            "id": "1605955287",
	            "createdAt": "2015-01-05T10:14:20Z",
	            "lat": 55.67218898125,
	            "lng": 12.591114862499996,
	            "course": 127.0,
	            "speed": 35.36
	          }
	        ]
	      }
	    ]
	  },
	  {
	    "vehicleType": "grande",
	    "vehiclePaths": [
	      {
	        "id": "64ebaf4be68f4fe8afea48d85fdd9807",
	        "positions": [
	          {
	            "id": "1605954999",
	            "createdAt": "2015-01-05T10:13:36Z",
	            "lat": 55.672440541176492,
	            "lng": 12.590609658823524,
	            "course": 128.0,
	            "speed": 35.61
	          },
	          {
	            "id": "1605954977",
	            "createdAt": "2015-01-05T10:13:41Z",
	            "lat": 55.672101164705907,
	            "lng": 12.591113735294112,
	            "course": 128.0,
	            "speed": 35.21
	          },
	          {
	            "id": "1605954899",
	            "createdAt": "2015-01-05T10:13:46Z",
	            "lat": 55.671761788235322,
	            "lng": 12.5916178117647,
	            "course": 128.0,
	            "speed": 35.07
	          },
	          {
	            "id": "1605954941",
	            "createdAt": "2015-01-05T10:13:51Z",
	            "lat": 55.671422411764738,
	            "lng": 12.592121888235287,
	            "course": 128.0,
	            "speed": 35.21
	          },
	          {
	            "id": "1605954927",
	            "createdAt": "2015-01-05T10:13:56Z",
	            "lat": 55.671083035294153,
	            "lng": 12.592625964705874,
	            "course": 128.0,
	            "speed": 35.58
	          },
	          {
	            "id": "1605954841",
	            "createdAt": "2015-01-05T10:14:01Z",
	            "lat": 55.670743658823568,
	            "lng": 12.593130041176462,
	            "course": 128.0,
	            "speed": 35.05
	          },
	          {
	            "id": "1605954827",
	            "createdAt": "2015-01-05T10:14:06Z",
	            "lat": 55.670404282352983,
	            "lng": 12.593634117647049,
	            "course": 128.0,
	            "speed": 35.22
	          },
	          {
	            "id": "1605954869",
	            "createdAt": "2015-01-05T10:14:11Z",
	            "lat": 55.6700649058824,
	            "lng": 12.594138194117637,
	            "course": 128.0,
	            "speed": 35.59
	          },
	          {
	            "id": "1605954855",
	            "createdAt": "2015-01-05T10:14:16Z",
	            "lat": 55.669725529411814,
	            "lng": 12.594642270588224,
	            "course": 128.0,
	            "speed": 35.04
	          },
	          {
	            "id": "1605955281",
	            "createdAt": "2015-01-05T10:14:21Z",
	            "lat": 55.669386152941229,
	            "lng": 12.595146347058812,
	            "course": 128.0,
	            "speed": 34.73
	          }
	        ]
	      }
	    ]
	  }
	]


<!--
### GET /geocodings

Performs a geocoding or reverse geocoding based on parameters.

#### Request parameters

`latlng` _required*_ Latitude and longitude of location for reverse geocoding (example `55.10,12.13`)

`address` _required*_ Full address to use for geocoding (example `Jagtvej 111, Denmark`)

(*) Either parameter is required.


#### Example request (address)

	curl https://api.drivr.com/geocodings?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json' 

#### Example response (address)

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "streetName": "Amphitheatre Pkwy",
	        "houseNumber": "1600",
	        "zipCode": "94043",
	        "city": "Mountain View",
	        "country": "US",
	        "lat": 37.423105399999997,
	        "lng": -122.08239880000001
	    }
	]

#### Example request (latlng)

	curl https://api.drivr.com/geocodings?latlng=55.696394,12.55082 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.drivr.v2+json'

#### Example response (latlng)

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "streetName": "Jagtvej",
	        "houseNumber": "111",
	        "zipCode": "2200",
	        "city": "Copenhagen",
	        "country": "DK",
	        "lat": 55.696393999999998,
	        "lng": 12.55082
	    }
	]
-->
