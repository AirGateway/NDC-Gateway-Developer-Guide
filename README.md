NDC Gateway Developer Guide
====================

This is a developer guide for our **NDC Gateway**, a platform designed to facilitate the connection to the new [NDC (New Distribution Capability)](http://www.iata.org/whatwedo/airline-distribution/ndc/) model established by IATA.

Our **NDC Gateway** facilitates the task of accessing multiple NDC providers in a one-to-one integration architecture. This makes much easier to interact with the NDC model and transitioning to it.

Sending requests to our NDC Gateway
------------
We use **HTTP/1.1** to respond requests to the gateway. All NDC requests are sent using the **POST** method.

The public endpoint to our **NDC Gateway** which remains inmutable for all NDC requests is this:
> http://prxy.airgateway.net:8080/gtwy/ndc/

We require some mandatory basic HTTP headers:
> **Content-Type**: application/xml
> **Accept**: application/xml

It's necessary to distinguish between two types of requests to send to the **NDC Gateway**. Attending to the architecture, we have two types of requests with significant differences between them:

- MPR (Multiple providers requests)
- SPR (Single provider requests)

----------

MPR (Multiple providers requests)
-------------

This requests are basically restricted to one single NDC method call (**AirShopping**) since it's the only NDC method that makes sense to be requested to multiple NDC providers. 

To execute a MPR you only need send a valid standard NDC **AirShopping** request including an HTTP header like this:
> **AG-Forwarded-Hosts:** *

or define a list of providers using [IATA airline designators](https://en.wikipedia.org/wiki/List_of_airline_codes) like this:
> **AG-Forwarded-Hosts:** WA, FA, GL

This is interpreted by the **NDC Gateway** like a send to all available NDC providers. As said before, it only makes sense to use this feature on **AirShopping** requests.

MPR behave asynchronously when used this HTTP headers:
> **Connection:** Keep-Alive

SPR (Single NDC provider requests)
-------------

To execute a MPR you only need send a valid standard NDC **AirShopping** request including an HTTP header like this:
> **AG-Forwarded-Hosts:** WA

------

Authentication
----
All requests sent to the **NDC Gateway**  are required to be consumer-authenticated with a HTTP header like this:
> **AG-Authorization:** {Client-API-Key}


NDC Formatting
----

We use plain NDC/XML formatting for the messages.

Example of a valid **AirShopping** request properly formatted:
```
<AirShoppingRQ Version="15.2" xmlns="http://www.iata.org/IATA/EDIST" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.iata.org/IATA/EDIST ../AirShoppingRQ.xsd">
	<Document>
		<Name>NDC Wrapper</Name>
		<ReferenceVersion>1.0</ReferenceVersion>
	</Document>
	<Party>
		<Sender>
			<ORA_Sender>
				<AgentUser>
					<Name>Travel Wadus</Name>
					<Type>TravelManagementCompany</Type>
					<PseudoCity>A4A</PseudoCity>
					<IATA_Number>00000000</IATA_Number>
					<AgentUserID>travelwadus</AgentUserID>
				</AgentUser>
			</ORA_Sender>
		</Sender>
	</Party>
	<Parameters>
		<CurrCodes>
			<CurrCode>EUR</CurrCode>
		</CurrCodes>
	</Parameters>
	<Travelers>
		<Traveler>
			<AnonymousTraveler>
				<PTC Quantity="1">ADT</PTC>
			</AnonymousTraveler>
		</Traveler>
	</Travelers>
	<CoreQuery>
		<OriginDestinations>
			<OriginDestination>
				<Departure>
					<AirportCode>LHR</AirportCode>
					<Date>2016-10-01</Date>
				</Departure>
				<Arrival>
					<AirportCode>JFK</AirportCode>
				</Arrival>
			</OriginDestination>
		</OriginDestinations>
	</CoreQuery>
	<Preferences>
	</Preferences>
</AirShoppingRQ>
```
