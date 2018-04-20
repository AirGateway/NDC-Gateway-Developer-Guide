v0.2.1

AirGateway NDC Gateway Developer Guide
====================

This is a developer guide intended to provide insights on a technical integration with AirGateway's **NDC JSON API** aggregation service.

Our platform is designed to facilitate the interaction with the new distribution model brought by [NDC (New Distribution Capability)](http://www.iata.org/whatwedo/airline-distribution/ndc/) established by IATA and adopted by a growing number of the most relevant airlines in the World.

Our **NDC Aggregation Platform** facilitates the interaction with multiple NDC providers in a one-to-one architecture. This makes much easier to interact with distributed airline NDC APIs and therefore facilitates transitioning into this model.

Our **NDC Aggregation Platform** works as a real-time routing machine delivering and collecting concurrently shopping offers from multiple NDC providers (airlines) and deliver an asynchronous real-time aggregation isolating consumers from cases they won't need to consider such as message wrapping (SOAP, REST...), Authentication issues, API architectural issues, and ancillaries normalization.
Also transitioning among different NDC versions/implementations will be a  much easier process to handle relying on our **NDC Aggregation Platform**.


NDC Messages available
----

This is a list of NDC valid requests in our **v1** version of the API.

- AirShopping
- FlightPrice
- SeatAvailability
- OrderCreate
- OrderRetrieve
- OrderChange
- OrderCancel
- ServiceList
- ServicePrice
- ItinReShop


> We are already working on a version upgrade to **v1.5** based in NDC
> **v17.2** that will replace **FlightPrice** with **OfferPrice** and adding
> new methods.

## Authentication

To be able to access our platform you will have to get the proper credentials at our [Developer Portal](https://dev.airgateway.net/). After registration, you would be able to apply for an Authentication key [here](https://dev.airgateway.net/apis/).

----
All requests sent to the **NDC Aggregation Platform**  are required to be consumer-authenticated with a two HTTP headers pair:

- **Authorization**: {Authorization-key} (obtained from our Dev Portal)
- **AG-Consumer**:  {Consumer-ID}  (a three chars. code)
----------

Note regarding Authentication:

> We will provide further details on how NDC Participant authentication
> (IATA number, Agency ID in NDC messages Party block) must be provided
> since this could be related to final production stage.


Sending requests to our NDC Gateway
------------
We use **HTTP/1.1** as communication protocol to accept requests to the gateway. All NDC requests are sent using the **POST** HTTP method.

The public endpoint to our **NDC Gateway** remains immutable for all NDC requests and it is:
> http://proxy.airgateway.net/ndc/v1/

We require some mandatory basic HTTP headers intended for message formatting:
> **Content-Type**: application/json
> **Accept**: application/json

It's very relevant to distinguish between two types of requests to send to the **NDC Gateway**. Attending to the architecture, we have two types of requests with significant differences between them:

- **MPR** (Multiple provider requests)
- **SPR** (Single provider requests)


MPR (Multi-Provider requests)
-------------

This requests are basically restricted to one single NDC method call (**AirShopping**) since it's the only NDC method that makes sense to be concurrently requested to multiple NDC providers.

To execute a MPR you only need send a valid standard NDC **AirShopping** request including an HTTP header like this:
> **AG-Providers:** *
This is interpreted by the **NDC Gateway** like a send to all available NDC providers for the consumer.

Alternatively, you can define a list of providers using a list of comma-separated standard [IATA airline designators](https://en.wikipedia.org/wiki/List_of_airline_codes).
> **AG-Providers:** BA, AA, LH

**MPR Responses**

Multi-Provider responses

MPRs behave asynchronously when used this HTTP headers:
> **Connection:** keep-alive


MPR response formats:

Our MPR responses (only applies to AirShopping seraches) has two modes: **Streaming** or **Standard HTTP**.

To enabled streaming mode (strongly recommended for GUI Applications) it's required to send the following HTTP header:

 - **AG-Connection**: keep-alive

And optionally, for the streaming mode a timeout in the server-side can be set upon request of the client.

 - **Keep-Alive:** timeout=xx (where xx is an integer number in seconds)


**Important note**

> Please, keep in mind streaming mode responses use a non-standard JSON notation wrapping called [JSON Streaming](https://en.wikipedia.org/wiki/JSON_streaming) used to streamline a collection of valid JSON objects instead of a resulting JSON-valid notation.


SPR (Single provider requests)
-------------
All non-AirShopping NDC requests are considered SPR. This means:

- FlightPrice
- SeatAvailability
- OrderCreate
- OrderRetrieve
- OrderChange
- OrderCancel
- ServiceList
- ServicePrice


To execute a SPR you need send a valid standard NDC request including an HTTP header detailing a specific provider like this:
> **AG-Providers:** BA

SPR responses don't have special any special wrapping since only one response is delivered in synchronous mode.


Cabin Preferences
-----------

List of cabin preferences values accepted by provider:

| Provider      | 1 | 2 | 3	| 4 | 5 |	6 |	7 |
| ------------- |:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| AA            | YES | YES |  |  | YES |  |  |
| BA            | YES | YES |  | YES | YES |  |  |
| EK            | YES | YES |  |  | YES |  |  |
| IB            | YES | YES |  | YES | YES |  |  |
| LH            | YES | YES |  |  | YES |  |  |
| S7            |  | YES |  |  | YES |  |  |
| WS            | YES | YES |  |  | YES |  |  |

1. First
2. Business
3. Lowest
4. Premium
5. Economy
6. Discounted
7. All

Those with "-" are still in integration process.

Fare Preferences
-----------

List of fare preferences values accepted by provider:

| Provider      | 70E | 70F | 70G	| 70H | 70I |	70M |	749 |	754 | 758 | 759 |
| ------------- |:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| AA            | YES | YES |   |   |   |   |   |   |   |   |
| BA            |   |   |   |   |   |   |   | YES | YES | YES |
| EK            | YES | YES |   |   |   |   |   |   |   |   |
| IB            |   |   |   |   |   |   |   |   | YES |   |
| LH            | YES | YES |   |   |   |   |   |   |   |   |
| S7            |   |   |   |   |   |   |   |   |   |   |
| WS            | YES | YES |   |   |   |   |   |   |   |   |

Those with "-" are still in integration process.


References and Links
-----------
We use a self-documented API Framework that produces always-up to date Swagger Documentation. This live-documentation can be found here.

> [NDC JSON API reference](https://airgateway.github.io/ndc-json-api/)

For futher questions/comments or just quickly getting in touch with our technical team you are welcome to use our Gitter Support Chat.

> https://gitter.im/AirGateway/Lobby
