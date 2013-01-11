---
layout: default
---

The Click A Taxi API provides the ability to create and retrieve required information in order to book taxis globally.

## Introduction

Every URL is in the format https://api.clickataxi.com/:resource[/:action[?:params]] such as https://api.clickataxi.com/places/nearby to get a list of nearby places.

Every request must include a `User-Agent` header with your application name and an url or email address. We need this for tracking purposes and we might contact you in case this information isn't probably filled out. An example of a good header would be `FooBooking-Version5 (http://foo-bookings.com/)`. Requests without this header might get a `400 Bad Request` in future versions of the API.

The `Accept` header specifies which format to return. Only `application/json` is supported.

Returned format may be overridden by appending a format parameter to a request e.g. https://api.clickataxi.com/companies?format=json.

Clients must include an `Accept` header with a custom Click A Taxi mime type to specify which version of the API to use e.g.

	application/vnd.clickataxi[.version]+[type]

If this header isn't specified the latest API version is used but it's highly recommended to specific a specific version since future revisions might not be fully backward compatible.

Currently `v1`, `v2` and `latest` are supported as `version` and `json` as `type`. 

Responses will be compressed if request includes an `Accept-Encoding` header with a `gzip` value.

<!-- FUTURE: Operations using a `lang` or `language` parameter might use an `Accept-Language` header instead. -->

All timestamps are represented as ISO 8601 format `YYYY-MM-DDTHH:MM:SSZ`.

IDs are represented as strings and should be stored and treated as strings. Format of IDs might change across API versions.

Example requests are constructed using [cURL](http://en.wikipedia.org/wiki/CURL) commands.


## Suggested booking flow

As as minimum requirement to book a taxi, one must first [authenticate](#authorizations) then create a [create a client](#clients) and finally [create a booking](#post_clients_bookings). Usually one would also like to display which taxi companies [operate in a given area](#spot_companies) and check if that company allows automated bookings and/or a special type of vehicle. To check if company supports automated bookings the "automatedBooking" attribute on a company should be checked. If this is `false` our API can't book on this company i.e. trying to place a booking through the API will fail but you may use returned phone number for company to display this as a fallback in your application flow. Most companies on our platform allows automatic booking, though.


## Authentication

You authenticate by sending token either as an `Authorization` header:

	curl https://api.clickataxi.com \
		-H "Authorization: Token token=\"OAUTH-TOKEN\""

Or by appending it as an `access_token` parameter:

	curl https://api.clickataxi.com?access_token=OAUTH-TOKEN

Tokens can be initially acquired programmatically using basic authentication through [/authorizations](#authorizations).


## JSON-P callbacks

You may send a `?callback` parameter to any GET operations to have results wrapped in a JSON callback function. This is typically used when browsers want to embed content in web pages by getting around cross domain issues. The response includes the same data output as the regular API, plus the relevant HTTP Header information.


## Operations

### <a id="authorizations"></a> POST /authorizations

Requests an authorization token by posting via basic authentication. Upon successful authentication a 256-bit token will be returned which should be used for future requests. These access tokens doesn't by default expire but it might become invalid over time. Using an invalid token in requests will result in `401 Unauthorized` responses and you should in that case re-issue a request for a new token.

#### Example request

	curl https://api.clickataxi.com/authorizations \
		-u john:doe -d '' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
		-H 'Content-type: application/json'

#### Example response

	HTTP/1.0 201 Created
	Content-Type: application/json; charset=utf-8

	{
	    "token": "1111a3bb8d4a40f08100e640621fee10"
	}


### <a id="nearby_places"></a> GET /places/nearby

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

	curl https://api.clickataxi.com/places/nearby?latlng=55.68%2C12.59&radius=1000&language=da \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json'


#### Example response

	HTTP/1.0 200 OK
	Content-Type: application/json; charset=utf-8

	[
	    {
	        "id": "1",
	        "ref": "COPENHAGEN AIRPORT",
	        "name": "Københavns Lufthavn",
	        "location": {
	            "streetName": "Lufthavnsboulevarden",
	            "houseNumber": "6",
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
	                "iconUrl": "http://resource.clickataxi.com/icons/place_type_airport@2x.png"
	            }
	        ]
	    }
	]


### <a id="get_company"></a> GET /companies/:companyId

Returns details about a specific company. Use this operation to a high-level details such as name, phone number and rating for a company. Operation will not return vehicle types, destination required details, etc since these are based on region and could be requested by using [/companies/spot](#spot_companies) instead.

#### Request parameters

`id` _required_ Id of company

#### Response attributes

`name` Name of taxi company

`phone` Number for taxi company using the [E.164 format](http://en.wikipedia.org/wiki/E.164)

`automatedBooking` True if company supports fully automated bookings through API; false otherwise

`destinationRequired` True if company requires a dropoff address for handling orders; false otherwise


#### Example request

	curl https://api.clickataxi.com/companies/1081132172 \
		-H 'Authorization: Token token="11113a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.clickataxi.v2+json'

#### Example response

	{
	    "id": "1081132172",
	    "name": "Taxi4x35",
	    "logoUrl": "http://resource.clickataxi.com/logos/taxi4x35.png",
	    "phone": "+4535309184",
	    "rating": 4.0
	}


### <a id="spot_companies"></a> GET /companies/spot

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

`properties.name[vehicleTypes]` A list of available vehicle types offered by company. The `ref` attribute should be used as `vehicleType` attribute when ordering a taxi. List vary from company to company but may be cached by company. Consider rechecking this attribute over time, though since taxi companies might start to support more vehicle types.

<table>
	<tr>
		<th width="20%">Ref</th>
		<th>Description</th>
	</tr>
	<tr>
		<td>fourSeaterAny</td>
		<td>Standard four seater car</td>
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
		<td>eightSeater</td>
		<td>Eight seater car</td>
	</tr>
	<tr>
		<td>premium</td>
		<td>Premium car</td>
	</tr>
</table>

`properties.name[services]` Additional services offered by company.

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
		<td>animals</td>
		<td>Animals may be brought onboard</td>
	</tr>
	<tr>
		<td>wheelchair</td>
		<td>Wheelchairs may be brought onboard</td>
	</tr>
</table>


#### Example request

	curl https://api.clickataxi.com/companies/spot?zipCode=2200&country=dk \
		-H 'Authorization: Token token="11113a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.clickataxi.v2+json'

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
	                        "iconUrl": "http://resource.clickataxi.com/icons/four-seater-any@2x.png"
	                    },
	                    {
	                        "ref": "sixSeater",
	                        "iconUrl": "http://resource.clickataxi.com/icons/six-seater@2x.png"
	                    }
	                ],
	                "mutuallyExclusive": true
	            },
	            {
	                "name": "services",
	                "values": [
	                    {
	                        "ref": "childSeat",
	                        "iconUrl": "http://resource.clickataxi.com/icons/child_seat@2x.png"
	                    },
	                    {
	                        "ref": "animals",
	                        "iconUrl": "http://resource.clickataxi.com/icons/animals@2x.png"
	                    },
	                    {
	                        "ref": "bike",
	                        "iconUrl": "http://resource.clickataxi.com/icons/bike@2x.png"
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


### <a id="lookup_companies"></a> GET /companies/lookup

Requests address, taxi company and address formats for a given location.


#### Request parameters

`language` ISO 639-1 language code of client (example `da` for Danish). This value is used to determine in which language addresses should be delivered.

`latlng` _required*_ Latitude and longitude of location for reverse geocoding (example `55.10,12.13`)


#### Response attributes

`place` A Place structure with address details about specified location.

`company` A Company structure with details about company operating in specified location.

`addressFormat` An AddressFormat structure with address formatting rules. This is an optional attribute which might not be available in all locations.


#### Example request

	curl https://api.master.clickataxi.com/companies/lookup?language=en-GB&latlng=51%2E509205,-0%2E146690 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' 


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
	        "properties": [
	            {
	                "name": "vehicleTypes",
	                "values": [
	                    {
	                        "ref": "fourSeaterAny",
	                        "iconUrl": "http://resource.master.clickataxi.com/properties/vehicle-types_four-seater-any_logo.png"
	                    },
	                    {
	                        "ref": "fiveSeater",
	                        "iconUrl": "http://resource.master.clickataxi.com/properties/vehicle-types_five-seater_logo.png"
	                    },
	                    {
	                        "ref": "fiveSeaterBlackCab",
	                        "iconUrl": "http://resource.master.clickataxi.com/properties/vehicle-types_five-seater-black-cab_logo.png"
	                    }
	                ],
	                "mutuallyExclusive": true
	            }
	        ],
	        "rating": 4.0,
	        "phone": "+442083808900",
	        "eta": 14
	    },
	    "addressFormat": {
	    	"titleTemplate": "streetName houseNumber",
	    	"subtitleTemplate": "zipCode city",
	    	"requiredFields": ["streetName", "houseNumber", "zipCode"]
	    }
	}


### GET /clients/:clientId/bookings

Returns list of all bookings created by client with specified id. Only bookings for clients you have created can be requested. Requesting bookings for a client you haven't created will return a 404 error message.


#### Request parameters

All parameters are optional unless specified otherwise.

`since` Only return bookings after this timestamp (example `2012-03-24T11:00:00Z`). If not specified all bookings are returned.

`completed` A boolean value indicating if only completed bookings should be returned. If not specified all bookings are returned.


#### Response attributes

`company` Full Company object with taxi company which handled booking

`pickup` Full Location object with pickup details

`dropoff` Full Location object with dropoff details (only returned if available).

`vehicleType` Type of car which is picking up client. Valid values are returned in [companies/spot](#spot_companies) operations (`vehicleTypes` attribute)

`arrivalAt` Date and time taxi will arrive/arrived.

`statuses` List of statuses associated with booking. Statuses are localized based on clients language. See [clients/:clientId/bookings/:bookingId](#get_client_booking) for details about these statuses.


#### Example request

	curl https://api.clickataxi.com/clients/61241/bookings?since=2012-01-01 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
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
	                "createdAt": "2012-08-28T13:24:06.9895402Z",
	                "type": "cancel_disabled",
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


### <a id="clients"></a> POST /clients

Creates a new client which is necessary before a booking can be created. A client must have at least a name and phone number. Both name and phone number might be send to a taxi company in order to book the taxi, so make sure your privacy policy tells your users about it. An SMS might be sent to client for verification purposes.

#### Request attributes

`name` Name of client which could be formatted to taxi company. This is usually a last name but could be a full name too.

`phone` Number of client formatted as a [E.164](http://en.wikipedia.org/wiki/E.164) number.

`language` ISO 639-1 language code of client (example `da` for Danish). This value is used to determine in which language status messages should be delivered.

`os` Type of OS used by client, if applicable. Types can be either 'ios', 'android' or 'windowsPhone'.

#### Example request

	curl https://api.clickataxi.com/clients \
		-H 'Authorization: Token token="1993a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
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


### GET /clients/:clientId

Get details about an existing client. Only clients created by you can be fetched. Requesting other clients will return in 404.

#### Example request

	curl https://api.clickataxi.com/clients/61449
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.clickataxi.v2+json'

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

### PUT /clients/:clientId

Updates an existing client. Only clients created by you can be updated.

#### Example request

	curl https://api.clickataxi.com/clients/61449
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"'
		-H 'Accept: application/vnd.clickataxi.v2+json'
		-H 'Content-type: application/json'
		-d '{ "name": "Jobs", "phone": "+4561715099" }'
		-X PUT

#### Example response

	HTTP/1.1 200 OK
	Content-Type: application/json; charset=utf-8

	{
	    "createdAt": "2012-09-05T12:11:01.9744265Z",
	    "id": "61449",
	    "name": "Jobs",
	    "phone": "+4561715099",
	    "language": "en",
	    "email": "john@doe.com",
	    "guid": "1003a3bb8d4a40f08065e640621fee63"
	}


### <a id="post_clients_bookings"></a> POST /clients/:clientId/bookings

Requests a new taxi booking for an existing [client](#clients). An immediate booking is requested by excluding `arrivalAt` attribute from body. A pre-booking should always have an `arrivalAt` in its body. Notice, some taxi companies require a destination address so a call to [/companies/spot](#spot_companies) should be made before booking in a new area. Check `destinationRequired` attribute on response to find out if a destination address is required in that area.

A success response means our system has accepted your request for a taxi but not necessarily that this taxi will be booked. Since we are connecting to real taxi companies around the globe it sometimes will not be possible to fulfill the order. You should check the order status by calling [/clients/:clientId/bookings/:bookingsId](#get_client_booking) regularly.


#### Request attributes

`pickup` Pickup details

`dropoff` Dropoff details

`vehicleType` Type of car which is requested. Valid values are returned in [companies/spot](#spot_companies) operations (`vehicleTypes` attribute)

`arrivalAt` Optional date and time for when taxi is requested. If not specified a taxi is immediately requested.

`comment` An optional comment for driver to see.



#### Example request

Creates a pre-booking of a medium sized taxi for Christmas Eve. A `pickup` structure may include a `placeRef` attribute instead of a full address in case you want to book at a [Point Of Interest](#nearby_places). Client is bringing a child so a child seat is requested as a service too.

	curl https://api.clickataxi.com/clients/9535253/bookings \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "arrivalAt": "2012-12-24T20:00:00.0000000Z", "pickup":{"streetName":"Amaliegade","houseNumber":"36","zipCode":"1256","city":"Copenhagen K","country":"DK","lat":55.68,"lng":12.59}, "vehicleType": "fourSeaterAny", "services": "childSeat" }'

#### Example response

	HTTP/1.0 201 Created
	Location: https://api.clickataxi.com/clients/9535253/bookings/483233
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


### <a id="get_client_booking"></a> GET /clients/:clientId/bookings/:id

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

	curl https://api.clickataxi.com/clients/61241/bookings/923880 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
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
	                        "iconUrl": "http://resource.master.clickataxi.com/properties/vehicle-types_four-seater-any_logo.png"
	                    },
	                    {
	                        "ref": "fourSeaterStationCar",
	                        "iconUrl": "http://resource.master.clickataxi.com/properties/vehicle-types_four-seater-station-car_logo.png"
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


### DELETE /clients/:clientId/bookings/:bookingId

Cancels a booking. The method might fail with a `400 Bad Request` response in case a booking can't be canceled e.g. if taxi is already on its way or at the pickup address. A `422 Unprocessed Entity` response is returned in case specified clientId or bookingId isn't valid.


#### Example request

	curl https://api.clickataxi.com/clients/61241/bookings/923878 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
		-X DELETE

#### Example response

When booking has been canceled successfully

	HTTP/1.1 200 OK


### POST /ratings

Rates a current booking. The `clientId`, `bookingId` and `stars` attributes are required. Attribute `stars` must be a valid integer between 1 and 5 (both inclusive).

#### Example request

	curl https://api.clickataxi.com/ratings \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' \
		-H 'Content-type: application/json' \
		-d '{ "clientId": "1543263", "bookingId": "5436443", "stars": 4, "review": "Funny driver" }'

#### Example response

	HTTP/1.1 201 Created
	Location: https://api.clickataxi.com/ratings/9dc8bb819c2a4cc4990a9f93be78669e
	Content-Type: application/json; charset=utf-8

	{
		"id": "9dc8bb819c2a4cc4990a9f93be78669e",
		"clientId": "1543263",
		"bookingId": "5436443",
	    "stars": 4,
	    "review": "Funny driver"
	}

<!--
### GET /geocodings

Performs a geocoding or reverse geocoding based on parameters.

#### Request parameters

`latlng` _required*_ Latitude and longitude of location for reverse geocoding (example `55.10,12.13`)

`address` _required*_ Full address to use for geocoding (example `Jagtvej 111, Denmark`)

(*) Either parameter is required.


#### Example request (address)

	curl https://api.clickataxi.com/geocodings?address=1600+Amphitheatre+Parkway,+Mountain+View,+CA \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json' 

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

	curl https://api.clickataxi.com/geocodings?latlng=55.696394,12.55082 \
		-H 'Authorization: Token token="1111a3bb8d4a40f08065e640621fee63"' \
		-H 'Accept: application/vnd.clickataxi.v2+json'

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