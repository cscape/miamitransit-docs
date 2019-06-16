---
title: Miami-Dade County Public Transportation Documentation 1.0 (June 2019)
author: 'Alejandra Agredo (CC0: Public Domain)'
---

Introduction
============

There are various public transportation systems in Miami-Dade County, each
belonging to a separate technical cluster of APIs. Many of these systems are
also run by different operators, such as the Coral Gables Trolley being run by
the City of Coral Gables as opposed to the Miami Trolley which is contracted to
a company, and the Metrobus which is run by Miami-Dade Transit. In the following
documentation, these systems will be grouped as follows (not a complete list of
systems, please see the reference):

-   **TSO Mobile**: Miami-Dade Transit (Contracted buses to LSF Shuttle),
    municipal trolleys (Miami, Miami Beach, Doral), other public transit
    (Aventura, North Miami Beach, South Miami)

-   **ETA Transit**: SFRTA/Tri-Rail, Coral Gables Trolley

-   **Transitime**: Miami-Dade Transit (Metrobus, Metrorail)

-   **Miami-Dade Transit XML Feed**: Miami-Dade Transit (Metromover)

-   **Brightline**: Brightline Trains

Focus on Tri-Rail, Metrorail, and the Metrobus first. Following these
high-ridership systems should be contracted buses/municipal trolleys, the Coral
Gables Trolley, the Metromover, and Brightline.

In cases where real-time data may overlap (such as Miami-Dade Transit buses via
TSO Mobile and Transitime), locate vehicle information and merge it together.
Most recently updated information takes priority over the older data, regardless
of which source it came from. Vehicles should be properly filtered by name to
prevent duplicates.

In other cases where data overlaps, such as the Miami Beach Trolley routes, data
from TSO Mobile (the vehicle tracking vendor) takes priority over the route
shape and stops from Miami-Dade Transit GTFS. Route data from the GTFS should be
*entirely discarded* for all trolley routes whenever data from TSO Mobile is
available.

Discrepancies
-------------

There’s inconsistency between the Miami-Dade Transit GTFS and what routes are
actually being serviced. As a result, there are at least 40 routes that do not
appear on any well-known app like Google Maps or MDT Tracker. Coordination
between municipalities and the county is poor, and many municipal trolleys have
routes changed or added without the change being reflected in GTFS. In this
case, real-time data from TSO Mobile and ETA Transit must be used and factored
in for trip planning.

For example, the Miami Trolley’s Coconut Grove route is not available on the
GTFS, but is available from TSO Mobile with real-time information. The route
data and real-time arrivals should be displayed to the user like any other
route. It should also be factored in for trip planning if possible.

Reference
=========

Real-Time Data Feeds
--------------------

| **Provider**           | **Feed Label**       | **Base URL**                                                                   |
|------------------------|----------------------|--------------------------------------------------------------------------------|
| Miami-Dade Transit XML | Basic Data           | <https://www.miamidade.gov/transit/WebServices>                                |
| Miami-Dade Transit XML | Verbose Data         | <https://www.miamidade.gov/transit/mobile/xml>                                 |
| TSO Mobile             | Public Transit       | <https://rest.tsoapi.com/PubTrans/GetModuleInfoPublic>                         |
| ETA Transit            | Coral Gables Trolley | <http://cgpublic.etaspot.net/service.php>                                      |
| ETA Transit            | Tri-Rail/SFRTA       | <http://trirailpublic.etaspot.net/service.php>                                 |
| Brightline             | Train Status         | <https://luxapi.verbinteractive.com/api/TrainStatus>                           |
| Transitime             | Tri-Rail             | <https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/trirail/command> |
| Transitime             | Miami-Dade Transit   | <https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/miami/command>   |

Static Transit
--------------

The three large transit systems can all be fetched live from a server.

| **Transportation Agency** | **Static GTFS file URL**                                                 |
|---------------------------|--------------------------------------------------------------------------|
| Miami-Dade Transit        | <https://miamidade.gov/transit/googletransit/current/google_transit.zip> |
| Tri-Rail/SFRTA            | <https://www.tri-rail.com/GTDF/Current/google_transit.zip>               |
| Brightline Trains         | <https://gobrightline.com/gtfs/gtfs.zip>                                 |

TSO Mobile Agencies
-------------------

| **Service Name**                      | **Vehicle Type** | **Company ID** |
|---------------------------------------|------------------|----------------|
| Miami-Dade Transit (Private Operator) | Jitney           | `30109`        |
| Miami Trolley                         | Trolley          | `21241`        |
| Miami Beach Trolley                   | Trolley          | `22844`        |
| Doral Trolley                         | Trolley          | `2500`         |
| NOMI Express                          | Jitney           | `24600`        |
| Aventura Express                      | Jitney           | `26082`        |
| SIB*shuttle*                          | Jitney           | `31344`        |
| NMB Line                              | Trolley          | `31934`        |
| Pinecrest People Mover                | Jitney           | `24560`        |
| Bal Harbour Express                   | Jitney           | `34323`        |
| I-Bus                                 | Jitney           | `49454`        |
| SoMi Shuttle                          | Jitney           | `34423`        |
| Miami Gardens Express                 | Trolley          | `21699`        |

Miami-Dade Transit GTFS/TSO Mobile Route Mappings
-------------------------------------------------

This is an extensive list best served by a configuration file. See
<https://github.com/cscape/wayline-config/blob/master/config.toml> for a TOML
representation of route mappings to and from Miami-Dade Transit GTFS internal
route IDs and TSO Mobile internal route IDs. An exportable JavaScript module is
provided at <https://github.com/cscape/wayline-config>

Real-Time Transit (Miami-Dade Transit)
======================================

Metrorail and Metrobus (Transitime)
-----------------------------------

Since Metrorail and Metrobus AVL data is sent directly to the Transitime server,
it can be fetched as GTFS Realtime by calling the Transitime Miami-Dade Transit
feed and using the `/gtfs-rt` endpoint. **This data is very reliable** and
recommended for the Metrorail and Metrobus.

### Vehicle Positions

Fetch the vehicle positions as a GTFS Realtime Vehicle Positions feed by calling
`/vehiclePositions` as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/miami/command/gtfs-rt/vehiclePositions
GET https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/miami/command/gtfs-rt/vehiclePositions?format=json
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the second GET request, the `format` parameter can be added to make the
response return in `json` or `human` (a human-readable version of the default
protocol buffer response). The response in JSON would look like the following:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ json
{
  "header": {
    "gtfsRealtimeVersion": "2.0",
    "incrementality": "FULL_DATASET",
    "timestamp": "1560460087"
  },
  "entity": [{
    "id": "AT5778",
    "vehicle": {
      "trip": {
        "tripId": "4291475",
        "startDate": "20190613",
        "routeId": "20910"
      },
      "position": {
        "latitude": 25.15502,
        "longitude": -80.38789,
        "bearing": 217.32881
      },
      "currentStopSequence": 13,
      "currentStatus": "IN_TRANSIT_TO",
      "timestamp": "1560460031",
      "stopId": "9840",
      "vehicle": {
        "id": "AT5778",
        "label": "AT5778"
      }
    }
  }]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Note:** In the response above, the vehicle ID is also the entity ID. **Any
vehicle ID that begins with “AT” or “LSF” should be treated specially**, as
these vehicles are contracted to a private transportation company and are
tracked with their own equipment and (sometimes) in addition to Miami-Dade
Transit AVL equipment. **Not accounting for this will produce false bunching and
wrong bus counts.** See the TSO Mobile feed on how to properly merge and handle
duplicate data.

### Trip Updates

Similar to the Vehicle Positions feed, this is a GTFS Realtime Trip Updates
feed. It can be fetched by calling `/tripUpdates` as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/miami/command/gtfs-rt/tripUpdates
https://transitime-api.goswift.ly/api/v1/key/81YENWXv/agency/miami/command/gtfs-rt/tripUpdates?format=json
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The JSON response will be similar to this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ json
{
  "header": {
    "gtfsRealtimeVersion": "2.0",
    "incrementality": "FULL_DATASET",
    "timestamp": "1560460683"
  },
  "entity": [{
    "id": "4282805",
    "tripUpdate": {
      "trip": {
        "tripId": "4282805",
        "startDate": "20190613",
        "scheduleRelationship": "SCHEDULED",
        "routeId": "20835",
        "directionId": 1
      },
      "stopTimeUpdate": [{
        "stopSequence": 1,
        "departure": {
          "time": "1560461880"
        },
        "stopId": "421",
        "scheduleRelationship": "SCHEDULED"
      }, {
        "stopSequence": 2,
        "arrival": {
          "time": "1560461733"
        },
        "stopId": "2293",
        "scheduleRelationship": "SCHEDULED"
      }],
      "vehicle": {
        "id": "LSF412"
      },
      "timestamp": "1560460645"
    }
  }]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Notice the “LSF” in the vehicle ID. The data from the GTFS Realtime Trip Updates
may be used for contracted vehicles and routes, but is **not recommended**. Do
not include it unless data from TSO Mobile is unavailable.

Metrobus\* and Trolleys
-----------------------

This section applies for the transit systems which use TSO Mobile for fleet
tracking. Namely, this includes certain Metrobus routes (including 35/35A) and
many municipal transit systems (see TSO Mobile Agencies in the reference).
**It’s recommended to ignore GTFS route and stop data when data from TSO Mobile
is available**, except for Metrobus routes. GTFS files may be slow to update to
reflect actual route shapes and stops for municipal transit systems. It may be
used as a fallback in the case that the user is offline.

### Using This Data

For all systems **except** Miami-Dade Transit, use the listed routes and data
from TSO Mobile as-is. This means that routes such as *Coral Way* with Miami
Trolley should entirely replace the data available on the GTFS. Additionally,
routes that do not appear in the GTFS such as the *Coconut Grove* route must
still be available for the end-user to see, regardless of its presence in the
GTFS feed or not. Other transit systems not present in the GTFS such as
SIB*shuttle* must also be available, using the accurate data directly from TSO
Mobile’s API.

#### Miami-Dade Transit Special Exception

There are nearly 30 routes operated by a private contractor for Miami-Dade
Transit (see<https://publictransportation.tsomobile.com/mdt.htm>, and the
GTFS/TSO Route Mappings in the reference). In the Transitime feeds for
Miami-Dade Transit, vehicle IDs will sometimes begin prefixed with “LSF”
(Limousines of South Florida) or “AT” (America’s Transportation). In the TSO
Mobile API, for the “Miami-Dade Transit” service (see TSO Mobile Agencies in the
reference), the `ShortName` property for a vehicle will appear as “Bus 964” or
“6221”.

In the case that the TSO vehicle `ShortName` begins with “Bus”, the Transitime
feed’s vehicle ID will begin with “LSF”. If the vehicle `ShortName` is just a
number, the Transitime feed’s vehicle ID will begin with “AT”. However, not all
vehicles tracked on Transitime (with Miami-Dade County AVL equipment) are also
tracked with TSO Mobile equipment, and vice-versa. This means that there are
some vehicles exclusive to the Transitime feed, and some exclusive to the TSO
Mobile API, as well as significant overlap.

#### Example

Let’s assume the Transitime feed has Trip Updates on a route like 35A.
Additionally, the TSO Mobile API shows where all the vehicles are (in terms of
between which stops and lat/long). How would you know which vehicles are the
same between feeds? If the Transitime feed has a vehicle ID that says “LSF294”,
you match it with any of the vehicles in the TSO Mobile API by checking for “Bus
294”. If it’s similar to “AT9327”, then you match it by looking for “9327” in
the TSO vehicles.

For trip updates, the stop IDs between the Transitime feed (using GTFS stop IDs)
are different than that from TSO Mobile’s Route Stops data. How do you account
for this difference? One way is to merge the data together by mapping identical
stop names to one another. Since the stop names in TSO Mobile’s data are nearly
identical to those in the GTFS, mapping between them should be rather simple. To
account for the situation where the ambiguous ETA with TSO Mobile stops might
cause problems, it’s recommended to merge TSO Mobile ETAs with the other
Transitime feed as long as they fall within a time span of up to 1 minute.

For example, let’s say “Stop X” in TSO Mobile’s API says that there are 3
upcoming arrivals: *2m 39s* (A), *9m 12s* (B), and *16m 43s* (C). Now, the
Transitime feed says there are also 3 upcoming arrivals: *2m 37s* (A), *17m 1s*
(C), and *26m 14s* (D). Notice that both list 3 vehicles en route, but this is
not what actually is happening. In reality, there’s 4 vehicles serving the route
at a 7-10 minute frequency. Also notice that Vehicle B is not present with the
Transitime feed, but is on the TSO Mobile feed. Vehicle D is on the Transitime
feed (meaning it’s tracked with MDT equipment) but not on the TSO Mobile feed.
These are all live and running, while Vehicles A and C are tracked by both
systems (thus, overlapping). In this case, proper leeway (about 1-2 minutes)
should be given to merge the ETAs properly, so it appears as (A), (B), (C), (D)
to the user, not (A), (A), (B), (C), (C), (D).

### TSO Public Transportation

The TSO Mobile Public Transit feed is the only URL needed, since all data passed
to it is in the form of parameters. Responses will be returned as string-encoded
JSON that will need to be unescaped and parsed. The following are examples of
real responses from the API:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"{\"stops\":{},\"routes\":{},\"points\":{}}"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
"[]"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### GET Routes

| **Name​** | **Required** | **Description**                    | **Example** |
|----------|--------------|------------------------------------|-------------|
| key      | required     | The method to use for this request | `routes`    |
| id       | required     | Company ID (see reference)         | `21241`     |

##### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://rest.tsoapi.com/PubTrans/GetModuleInfoPublic?key=routes&id=26082
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### Example Response

This is the result after parsing the response. The `ID` property here refers to
the route ID and the `RoutePath` uses standard polyline encoding.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[{
  "ID": "102216",
  "Name": "Blue Express Route",
  "RoutePath": "__m}CdidhNCkBaE|@QCc@g@o@s@U...",
  "LineColor": "#1303FF",
  "StopIcon": "Stop_Small_Blue.png",
  "UnitIcon": "Unit_Blue.png"
}]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### GET Route Stops

| **Name** | **Required** | **Description**                    | **Example**   |
|----------|--------------|------------------------------------|---------------|
| key      | required     | The method to use for this request | `route_stops` |
| id       | required     | Route ID (see GET Routes)          | `102216`      |

##### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://rest.tsoapi.com/PubTrans/GetModuleInfoPublic?key=route_stops&id=102216
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### Example Response

This is the result after parsing the response. The array of stops contains an
`ID` property which refers to the internal stop ID. The `Description` property
refers to the user-facing stop name.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[{
  "ID": "1155598",
  "StopNumber": "1001",
  "Sequence": "1",
  "Street": "19525 Biscayne Boulevard Abigail Road",
  "Description": "Transit Terminal",
  "Latitude": "25.958400",
  "Longitude": "-80.144987",
  "Weelchair": "",
  "RouteID": "102216",
  "DirectionName": "",
  "Direction": ""
}, {
  "ID": "1155599",
  "StopNumber": "1002",
  "Sequence": "2",
  "Street": "2952-2984 Northeast 199th Street Aventura Mall Eckerd",
  "Description": "Publix",
  "Latitude": "25.960051",
  "Longitude": "-80.141868",
  "Weelchair": "",
  "RouteID": "102216",
  "DirectionName": "",
  "Direction": ""
}, {
  "ID": "1155600",
  "StopNumber": "1003",
  "Sequence": "3",
  "Street": "125 Northeast 29th Place",
  "Description": "NE 29 Pl / Walgreens",
  "Latitude": "25.961775",
  "Longitude": "-80.141865",
  "Weelchair": "",
  "RouteID": "102216",
  "DirectionName": "",
  "Direction": ""
}]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### GET Vehicle Positions

| **Name** | **Required** | **Description**                    | **Example**            |
|----------|--------------|------------------------------------|------------------------|
| key      | required     | The method to use for this request | `units_location_route` |
| id       | required     | Route ID (see GET Routes)          | `85862`                |

##### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://rest.tsoapi.com/PubTrans/GetModuleInfoPublic?key=units_location_route&id=85862
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### Example Response

This is the result after parsing the response. The array of vehicles includes an
`ID` property which can be used to uniquely identify the vehicle. The `hea` is
the bearing from 0 to 359. `Tim` is the timestamp when the vehicle’s location
was last updated. `StopA` represents the previous vehicle stop, and `StopB`
represents the next stop. If `RealStopID` is not an empty string, then it means
that the vehicle is currently at that stop.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[{
  "ID": "1057402",
  "ShortName": "MB21",
  "Lat": "25.790623",
  "Lng": "-80.136557",
  "Hea": "358",
  "Tim": "1560632428",
  "SquareID": "13654196",
  "StopA": "1028427",
  "StopB": "1028428",
  "RealStopID": "1028427",
  "RouteID": "85862",
  "AntibunchingCmd": "2",
  "Occupation": "0"
}, {
  "ID": "1057404",
  "ShortName": "MB22",
  "Lat": "25.797689",
  "Lng": "-80.128196",
  "Hea": "205",
  "Tim": "1560632461",
  "SquareID": "13652499",
  "StopA": "1045210",
  "StopB": "1045211",
  "RealStopID": "",
  "RouteID": "85862",
  "AntibunchingCmd": "3",
  "Occupation": "0"
}]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### GET Stop Info

| **Name** | **Required** | **Description**                    | **Example** |
|----------|--------------|------------------------------------|-------------|
| key      | required     | The method to use for this request | `stopinfo`  |
| id       | required     | Stop ID (see GET Route Stops)      | `1067128`   |

##### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://rest.tsoapi.com/PubTrans/GetModuleInfoPublic?key=stopinfo&id=1067128
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### Example Response

For Miami-Dade Transit routes (such as 72A and circulators with a numbered route
short-name like the 212 Sweetwater Trolley), the `Description` property will
exactly match the stop name in the Miami-Dade Transit GTFS file. This is useful
for merging ETAs between the Transitime and TSO Mobile feeds.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[{
  "RouteName": "Route - 72A",
  "ID": "1067128",
  "StopNumber": "98",
  "Description": "SW 72 St & SW 79 CT",
  "Street": "8000-8012 Southwest 72nd Street",
  "Latitude": "25.702708",
  "Longitude": "-80.323340",
  "ETAResult": "1",
  "ETASeconds": "1624"
}]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Metromover, Metrobus, and Metrorail (XML Feed)
----------------------------------------------

Most commonly, the Miami-Dade Transit XML feed is used to fetch data from the
Metrobus, Metrorail, and Metromover. This is done by fetching the `/Buses` and
`/Trains` endpoint on either the verbose or basic feed.

**This data is slow to update for buses (up to 40 minute delays) and may be
prone to errors and inconsistencies. Please use the Transitime feed for the
Metrorail and Metrobus whenever possible.**

### Metromovers

The Miami-Dade Transit XML Feed should only be primarily used for fetching
real-time information about the Metromover system. There are three loop
configurations: **BKL** (Brickell Loop), **INN** (Inner Loop), and **OMN** (Omni
Loop).

#### Vehicle Positions

The location of all Metromovers, with direction and speed. This is updated
roughly every 10-15 seconds. This feed can be fetched by calling `/MoverTrains`
on either the verbose or basic feed as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://www.miamidade.gov/transit/mobile/xml/MoverTrains/
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This would return a response similar to this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<MoverTrains>
    <Train>
        <TrainID>872</TrainID>
        <Cars>46</Cars>
        <Latitude>25.771006</Latitude>
        <Longitude>-80.192551</Longitude>
        <vehDirection>N</vehDirection>
        <vehSpeed>16</vehSpeed>
        <ServiceDirection>NB</ServiceDirection>
        <Service>Northbound</Service>
        <LoopID>BKL</LoopID>
        <LoopName>Brickell</LoopName>
        <Color>F95309</Color>
        <LocationUpdated>3:41:41 PM</LocationUpdated>
        <LocationUpdatedDiff>5 seconds ago</LocationUpdatedDiff>
        <LocationUpdatedDiffBasic>5</LocationUpdatedDiffBasic>
    </Train>
</MoverTrains>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### Station Arrivals

The arrival estimates and some other station information for the Metromovers.
This feed can be fetched by calling `/MoverTracker` on either the verbose or
basic feed as follows:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://www.miamidade.gov/transit/WebServices/MoverTrains/
GET https://www.miamidade.gov/transit/mobile/xml/MoverTracker/?StationID=GVT
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Note:** The verbose feed does return more information, including a fifth
arrival time and a RiderAlert field, but it **requires** the StationID
parameter. The following is an example response of the verbose feed:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~ xml
<?xml version="1.0" encoding="UTF-8"?>
<MoverTracker>
    <Info>
        <StationName>Government Center</StationName>
        <StationID>GVT</StationID>
        <StationAddress>101 NW First Street</StationAddress>
        <RiderAlert></RiderAlert>
        <Facilities></Facilities>
        <firstLoopID>INN</firstLoopID>
        <firstLoopName>Inner</firstLoopName>
        <firstDirection></firstDirection>
        <firstTime1>0:11 sec</firstTime1>
        <firstTime1_Est>11</firstTime1_Est>
        <firstTime1_Arrival>4:03 PM</firstTime1_Arrival>
        <firstTime1_Train>886</firstTime1_Train>
        <firstTime1_ServiceTypeID>0</firstTime1_ServiceTypeID>
        <firstTime2>2:58 min</firstTime2>
        <firstTime2_Est>178</firstTime2_Est>
        <firstTime2_Arrival>4:05 PM</firstTime2_Arrival>
        <firstTime2_Train>901</firstTime2_Train>
        <firstTime2_ServiceTypeID>0</firstTime2_ServiceTypeID>
        <secondLoopID>BKL</secondLoopID>
        <secondLoopName>Brickell</secondLoopName>
        <secondDirection></secondDirection>
        <secondTime1>0:00 sec</secondTime1>
        <secondTime1_Est>0</secondTime1_Est>
        <secondTime1_Arrival>4:02 PM</secondTime1_Arrival>
        <secondTime1_Train>897</secondTime1_Train>
        <secondTime1_ServiceTypeID>0</secondTime1_ServiceTypeID>
        <secondTime2>6:02 min</secondTime2>
        <secondTime2_Est>362</secondTime2_Est>
        <secondTime2_Arrival>4:08 PM</secondTime2_Arrival>
        <secondTime2_Train>874</secondTime2_Train>
        <secondTime2_ServiceTypeID>0</secondTime2_ServiceTypeID>
        <thirdLoopID>OMN</thirdLoopID>
        <thirdLoopName>Omni</thirdLoopName>
        <thirdDirection></thirdDirection>
        <thirdTime1>4:11 min</thirdTime1>
        <thirdTime1_Est>251</thirdTime1_Est>
        <thirdTime1_Arrival>4:07 PM</thirdTime1_Arrival>
        <thirdTime1_Train>887</thirdTime1_Train>
        <thirdTime1_ServiceTypeID>0</thirdTime1_ServiceTypeID>
        <thirdTime2>9:06 min</thirdTime2>
        <thirdTime2_Est>546</thirdTime2_Est>
        <thirdTime2_Arrival>4:11 PM</thirdTime2_Arrival>
        <thirdTime2_Train>882</thirdTime2_Train>
        <thirdTime2_ServiceTypeID>0</thirdTime2_ServiceTypeID>
        <forthLoopID></forthLoopID>
        <forthLoopName></forthLoopName>
        <forthDirection></forthDirection>
        <forthTime1></forthTime1>
        <forthTime1_Est></forthTime1_Est>
        <forthTime1_Arrival></forthTime1_Arrival>
        <forthTime1_Train></forthTime1_Train>
        <forthTime1_ServiceTypeID></forthTime1_ServiceTypeID>
        <forthTime2></forthTime2>
        <forthTime2_Est></forthTime2_Est>
        <forthTime2_Arrival></forthTime2_Arrival>
        <forthTime2_Train></forthTime2_Train>
        <forthTime2_ServiceTypeID></forthTime2_ServiceTypeID>
        <fifthLoopID></fifthLoopID>
        <fifthLoopName></fifthLoopName>
        <fifthDirection></fifthDirection>
        <fifthTime1></fifthTime1>
        <fifthTime1_Est></fifthTime1_Est>
        <fifthTime1_Arrival></fifthTime1_Arrival>
        <fifthTime1_Train></fifthTime1_Train>
        <fifthTime1_ServiceTypeID></fifthTime1_ServiceTypeID>
        <fifthTime2></fifthTime2>
        <fifthTime2_Est></fifthTime2_Est>
        <fifthTime2_Arrival></fifthTime2_Arrival>
        <fifthTime2_Train></fifthTime2_Train>
        <fifthTime2_ServiceTypeID></fifthTime2_ServiceTypeID>
    </Info>
</MoverTracker>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Real-Time Transit (Tri-Rail/SFRTA)
==================================

Commuter Rail and Connector Buses
---------------------------------

Tri-Rail/SFRTA and their connector buses’ AVL data is sent directly to the same
Transitime server as the Metrobus and Metrorail for Miami-Dade Transit. The data
can be fetched as GTFS Realtime by calling the Transitime Tri-Rail feed and
using the `/gtfs-rt` endpoint. **This data is very reliable and accurate.**
There are no known “gotchas” with this system, and the same API applies for
Transitime data. (see Transitime Vehicle Positions and Trip Updates)

Real-Time Transit (Coral Gables Trolley, Tri-Rail/SFRTA)
========================================================

This section deals only with ETA Transit. The same principle of wholly
discarding Coral Gables Trolley data from the Miami-Dade Transit GTFS also
applies here, treated in the same way with the transit systems that are *not*
present on the GTFS (the only exception applies for offline trip planning
purposes). Tri-Rail/SFRTA shares the same system with Coral Gables Trolley, and
both may be treated the same way. However, **it is highly recommended to use the
Transitime feed whenever possible for Tri-Rail/SFRTA**.

API
---

All valid responses should return plain JSON. The API endpoint varies based on
the serviced agency, but generally follows the syntax
`http://<agency>.etaspot.net/service.php`. For calling methods, everything is
sent to the same endpoint as parameters.

### get_vehicles

Returns a JSON object containing an array of all tracked vehicles, which may
also include vehicles not currently running.

#### Parameters

| **Name**           | **Required** | **Description**                                                                                                                                                          | **Example** |
|--------------------|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|
| equipmentID        | optional     | The `equipmentID` of a vehicle, used for tracking a single vehicle                                                                                                       | `502`       |
| includeETAData     | optional     | If set to `1`, a `minutesToNextStops` property will be added to each vehicle containing an array of trip update objects. Setting to `-1`does not return any trip updates | `-1`        |
| minutesToNextStops | optional     | If set to `1`, the `minutesToNextStops` property will have a sorted array                                                                                                | `1`         |

#### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET http://trirailpublic.etaspot.net/service.php?service=get_vehicles&includeETAData=1&orderedETAArray=1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### Example Response

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "get_vehicles": [
    {
      "aID": "354676050503",
      "blockID": 0,
      "capacity": 100,
      "deadHead": "0",
      "direction": "South",
      "directionAbbr": "S.",
      "equipmentID": "503",
      "inService": 1,
      "lat": 25.9222,
      "lng": -80.21625,
      "load": 0,
      "minutesToNextStops": [
        {
          "blockID": 0,
          "direction": "South",
          "directionAbbr": "S.",
          "equipmentID": "503",
          "minutes": 1,
          "patternStopID": "32",
          "routeID": 1,
          "schedule": "01:20PM",
          "scheduleNumber": "P673",
          "status": "On Time",
          "statuscolor": "#39b139",
          "stopID": "14",
          "time": "01:19PM",
          "track": 1
        },
        {
          "blockID": 0,
          "direction": "South",
          "directionAbbr": "S.",
          "equipmentID": "503",
          "minutes": 31,
          "patternStopID": "36",
          "routeID": 1,
          "schedule": "01:50PM",
          "scheduleNumber": "P673",
          "status": "On Time",
          "statuscolor": "#39b139",
          "stopID": "18",
          "time": "01:49PM",
          "track": 2
        }
      ],
      "nextPatternStopID": 32,
      "nextStopETA": 800,
      "nextStopID": 14,
      "onSchedule": 0,
      "patternID": 1,
      "receiveTime": 1550945942000,
      "routeID": 1,
      "scheduleNumber": "P673",
      "trainID": 190,
      "tripID": "P673",
      "vehicleType": "Train"
    }
  ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### get_routes

Returns a JSON object containing an array of route objects.

#### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET http://trirailpublic.etaspot.net/service.php?service=get_routes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#example-response-2>Example Response

The `encLine` property in each route object is a regular polyline, and `vType`
is usually either “Bus” or “Train”.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "get_routes": [{
    "abbr": "TR",
    "color": "#006c86",
    "encLine": "qgm|C`mzhNuONcGLmJLgYNoRNmEH}...",
    "id": 1,
    "name": "Tri-Rail",
    "order": 1,
    "showDirection": true,
    "showPlatform": true,
    "showScheduleNumber": 1,
    "showVehicleCapacity": false,
    "stops": [1, 18, 2, 17, 3, 16, 4, 15, 5, 14, 6, 13, 7, 12, 8, 11, 9, 10],
    "type": "Inbound",
    "vType": "Train"
  }]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### get_patterns

Returns a JSON object containing an array of pattern objects. This endpoint
contains information on route lengths (distance-wise in miles)

#### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#example-request-3>Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET http://trirailpublic.etaspot.net/service.php?service=get_patterns
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#example-response-3>Example Response

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "get_patterns": [{
    "color": "#2E3191",
    "decLine": [],
    "encLine": "qgm|C`mzhNuONcGLmJLgYNoRNmEH}...",
    "extID": "1",
    "id": 1,
    "length": 75,
    "name": "Northbound",
    "routeNames": ["Tri-Rail"],
    "routes": [1],
    "stations": [],
    "type": 1
  }]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#get_stops>get_stops

Returns a JSON object containing an array of stop objects. The stop objects are
not all unique, some overlap. Each stop’s unique identifier is in
the `id` property, which can be used to merge together stop information. Exact
stop locations may vary even for those with the same `id`, make sure to be aware
of this.

#### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#parameters-1>Parameters

| **Name** | **Required** | **Description**                                                                                                      | **Example** |
|----------|--------------|----------------------------------------------------------------------------------------------------------------------|-------------|
| stopIDs  | optional     | A comma-delimited list of stop IDs. If set, it returns only the stops which match this same ID instead of all stops. | `2,5,3`     |

#### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET http://trirailpublic.etaspot.net/service.php?service=get_stops&stopIDs=2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### <https://github.com/cscape/miamitransit-docs/wiki/ETA-Transit-API#example-response-4>Example Response

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "get_stops": [
    {
      "rid": 1,
      "id": 2,
      "name": "West Palm Beach Station",
      "lat": 26.713298797607,
      "lng": -80.062538146973,
      "extID": "",
      "shortName": "WPB"
    },
    {
      "rid": 51,
      "id": 2,
      "name": "West Palm Beach Station",
      "lat": 26.712093353271,
      "lng": -80.062118530273,
      "extID": "2",
      "shortName": "2"
    }
  ]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### get_stop_etas

Returns a JSON object containing an array of ETAs for a stop.

#### Parameters

| **Name** | **Required** | **Description**                                                                                                      | **Example** |
|----------|--------------|----------------------------------------------------------------------------------------------------------------------|-------------|
| stopIDs  | optional     | A comma-delimited list of stop IDs. If set, it returns only the stops which match this same ID instead of all stops. | `12,3`      |

#### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET http://trirailpublic.etaspot.net/service.php?service=get_stop_etas&stopIDs=14
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

#### Example Response

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
{
  "get_stop_etas": [{
    "enRoute": [{
      "blockID": 0,
      "direction": "South",
      "directionAbbr": "S.",
      "equipmentID": "1006",
      "minutes": 6,
      "patternStopID": 32,
      "routeID": 1,
      "schedule": "12:20PM",
      "scheduleNumber": "P671",
      "status": "12:33PM",
      "statuscolor": "#b91d1d",
      "stopID": 14,
      "time": "12:33PM",
      "track": 1
    }],
    "id": 14
  }]
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Real-Time Transit (Brightline)
==============================

This is among the simplest of all systems to implement, since the GTFS file is
well-done and the train status API is simple to work with. However, there is no
known way to track vehicle positions. This is not a major issue since the trains
only board at the stations and the real-time arrival estimates are very
accurate.

Train Arrival and Departure Times
---------------------------------

There exist other APIs, however, the Train Status API is the most relevant one
for trip planning and most usage purposes. This returns an array of all trains
scheduled to run on a certain date, as well as information about each train’s
current status (for example: departed, arrived, arriving, scheduled to depart).
The station IDs here directly correspond to the stop IDs (and parent station) in
the Brightline GTFS.

### Parameters

| **Name**      | **Required** | **Description**                                                                                          | **Example**  |
|---------------|--------------|----------------------------------------------------------------------------------------------------------|--------------|
| arrival       | optional     | The train station ID. If specified, it returns all trains arriving at this station for a certain day.    | `FLL`        |
| departure     | optional     | The train station ID. If specified, it returns all trains departing from this station for a certain day. | `PBI`        |
| departureDate | required     | A `YYYY-MM-DD` formatted date to fetch trains for.                                                       | `2019-06-16` |

### Example Request

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
GET https://luxapi.verbinteractive.com/api/TrainStatus?arrival=PBI&departureDate=2019-06-16
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

### Example Response

The `legs` property for each train status object refers to the individual trips
between stations. It’s useful for getting the ETAs of the same train for
upcoming stations.

The `consistNumber` refers to the color of the train, and it comes in 5
variants: `BRTBLUE` (Blue), `BRTPINK` (Pink), `BRTORANGE` (Orange), `BRTGREEN`
(Green), and `BRTRED` (Red). Brightline employees often announce the train by
its color, such as “Bright Red is now boarding” and “The Bright Green train is
on the left-hand track.” This is useful information for a rider and should be
included (such as a colored indicator next to a station’s ETAs).

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
[{
  "actualArrival": null,
  "actualArrivalPlatform": "1",
  "actualDeparture": "2019-06-16T12:10:00",
  "actualDeparturePlatform": "2",
  "arrival": "2019-06-16T13:25:00",
  "arrivalPlatform": "1",
  "arrivalStatus": "On-Time",
  "arrivalType": "Estimated",
  "bagClaim": "",
  "consistNumber": "BRTORANGE",
  "departure": "2019-06-16T12:10:00",
  "departurePlatform": "2",
  "departureStatus": "Departed",
  "departureType": "Actual",
  "destination": "PBI",
  "estimatedArrival": "2019-06-16T13:25:00",
  "estimatedDeparture": null,
  "fidsArrivalStatus": "Default",
  "fidsDepartureStatus": "Default",
  "legs": [{
      "actualArrival": "2019-06-16T12:42:00",
      "actualArrivalPlatform": "1",
      "actualDeparture": "2019-06-16T12:10:00",
      "actualDeparturePlatform": "2",
      "arrival": "2019-06-16T12:42:00",
      "arrivalPlatform": "1",
      "arrivalStatus": "Arrived",
      "arrivalType": "Actual",
      "bagClaim": "",
      "consistNumber": "BRTORANGE",
      "departure": "2019-06-16T12:10:00",
      "departurePlatform": "2",
      "departureStatus": "Departed",
      "departureType": "Actual",
      "destination": "FLL",
      "estimatedArrival": "2019-06-16T12:42:00",
      "estimatedDeparture": null,
      "fidsArrivalStatus": "Default",
      "fidsDepartureStatus": "Default",
      "inventoryLegId": 43234,
      "isCancelled": false,
      "legStatus": "Normal",
      "origin": "MIA",
      "scheduledArrival": "2019-06-16T12:40:00",
      "scheduledArrivalPlatform": "1",
      "scheduledDeparture": "2019-06-16T12:10:00",
      "scheduledDeparturePlatform": "3",
      "train": "710"
    },
    {
      "actualArrival": null,
      "actualArrivalPlatform": "1",
      "actualDeparture": "2019-06-16T12:45:00",
      "actualDeparturePlatform": "1",
      "arrival": "2019-06-16T13:25:00",
      "arrivalPlatform": "1",
      "arrivalStatus": "On-Time",
      "arrivalType": "Estimated",
      "bagClaim": "",
      "consistNumber": "BRTORANGE",
      "departure": "2019-06-16T12:45:00",
      "departurePlatform": "1",
      "departureStatus": "Departed",
      "departureType": "Actual",
      "destination": "PBI",
      "estimatedArrival": "2019-06-16T13:25:00",
      "estimatedDeparture": "2019-06-16T12:45:00",
      "fidsArrivalStatus": "Default",
      "fidsDepartureStatus": "Default",
      "inventoryLegId": 43233,
      "isCancelled": false,
      "legStatus": "Normal",
      "origin": "FLL",
      "scheduledArrival": "2019-06-16T13:25:00",
      "scheduledArrivalPlatform": "2",
      "scheduledDeparture": "2019-06-16T12:42:00",
      "scheduledDeparturePlatform": "1",
      "train": "710"
    }
  ],
  "origin": "MIA",
  "scheduledArrival": "2019-06-16T13:25:00",
  "scheduledArrivalPlatform": "2",
  "scheduledDeparture": "2019-06-16T12:10:00",
  "scheduledDeparturePlatform": "3",
  "segmentStatus": "On-Time",
  "train": "710"
}]
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
