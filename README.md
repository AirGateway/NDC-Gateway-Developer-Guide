v0.1.2

AirGateway NDC Gateway Developer Guide
====================

This is a developer guide intended for our **NDC Gateway** aggregation service. Our platform is designed to facilitate the connection to the new [NDC (New Distribution Capability)](http://www.iata.org/whatwedo/airline-distribution/ndc/) model established by IATA in a multiple-provider world.

Our **NDC Gateway** facilitates the task of accessing multiple NDC providers in a one-to-one integration architecture. This makes easier to interact with the NDC architecture and travel agents transitioning into it.

Our **NDC Gateway** works as a "router" delivering and collecting  concurrently shopping offers from multiple NDC providers (airlines) and deliver an asynchronous real-time aggregation isolating consumers from a lot of cases they won't need to consider such as message wrapping (SOAP, REST...), Authentication issues, host architecture issues, and ancillaries normalization.
Also transitioning among different NDC versions/implementations will be a  much easier process to handle relying our **NDC Gateway**.


NDC Messages available
----

We expect plain NDC/XML formatting for the messages. No SOAP or any other wrapping is necessary nor allowed.

This is a list of NDC valid requests to send to our gateway that will be forwarded to the enabled providers:

- AirShopping
- FlightPrice
- SeatAvailability
- OrderCreate
- OrderRetrieve
- OrderChange
- OrderCancel
- ServiceList
- ServicePrice


Authentication
----
All requests sent to the **NDC Gateway**  are required to be consumer-authenticated with a couple of HTTP headers:

> **AG-Consumer**:  {Consumer-Key}
> **AG-Authorization**: {Proxy-key}

----------

Note regarding Authentication:

	 We will provide further details on how NDC Participant authentication (IATA number, Agency ID in NDC messages Party block) must be provided since this could be related to final providers.


Sending requests to our NDC Gateway
------------
We use **HTTP/1.1** to respond requests to the gateway. All NDC requests are sent using the **POST** HTTP method.

The public endpoint to our **NDC Gateway** remains inmutable for all NDC requests and it is:
> http://prxy.airgateway.net:8181/gtwy/ndc/

We require some mandatory basic HTTP headers intended for message formatting:
> **Content-Type**: application/xml
> **Accept**: application/xml

It's very relevant to distinguish between two types of requests to send to the **NDC Gateway**. Attending to the architecture, we have two types of requests with significant differences between them:

- **MPR** (Multiple provider requests)
- **SPR** (Single provider requests)


MPR (Multiple provider requests)
-------------

This requests are basically restricted to one single NDC method call (**AirShopping**) since it's the only NDC method that makes sense to be concurrently requested to multiple NDC providers.

To execute a MPR you only need send a valid standard NDC **AirShopping** request including an HTTP header like this:
> **AG-Providers:** *
This is interpreted by the **NDC Gateway** like a send to all available NDC providers for the consumer.

Alternatively, you can define a list of providers using a list of comma-separated [IATA airline designators](https://en.wikipedia.org/wiki/List_of_airline_codes).
> **AG-Providers:** BA, AA, LH

**MPR Responses**
 
MPRs behave asynchronously when used this HTTP headers:
> **Connection:** keep-alive
> **Keep-Alive:** timeout=xx
> (where xx is an integer number of seconds)

This asynchronously mode is the only available mode for MPR requests.

AirShopping responses are delivered asynchronously (ASAP) wrapped  in XML comments blocks providing some extra info such as:

- ProviderLabel
- NDCMethod
- ResponseStatus*
- ResponseTime*
(* from provider to our gateway)


MPR response format:

```
<!-- AG-Info: ProviderName: S7 | Status: ok | NDCMethod: AirShopping | ResponseTime: 5374 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    (...)
</AirShoppingRS>
<!-- AG-EOM -->
<!-- AG-Info: ProviderName: LH | Status: ok | NDCMethod: AirShopping | ResponseTime: 7082 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    (...)
</AirShoppingRS>
<!-- AG-EOM -->
<!-- AG-Info: ProviderName: AA | Status: ok | NDCMethod: AirShopping | ResponseTime: 7231 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    (...)
</AirShoppingRS>
<!-- AG-EOM -->
<!-- AG-Info: ProviderName: IB | Status: ok | NDCMethod: AirShopping | ResponseTime: 9376 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    (...)
</AirShoppingRS>
<!-- AG-EOM -->
<!-- AG-Info: ProviderName: BA | Status: ok | NDCMethod: AirShopping | ResponseTime: 12658 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    (...)
</AirShoppingRS>
<!-- AG-EOM -->
```

All Responses from providers are normalized to an optimized "easier consumable format"


SPR (Single provider requests)
-------------

To execute a SPR you only need send a valid standard NDC **AirShopping** request including an HTTP header detailing a specific provider like this:
> **AG-Providers:** BA