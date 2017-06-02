v0.5.1

AirGateway NDC Gateway Developer Guide
====================

This is a developer guide intended for our **NDC Gateway** aggregation service. Our platform is designed to facilitate the connection to the new [NDC (New Distribution Capability)](http://www.iata.org/whatwedo/airline-distribution/ndc/) model established by IATA in a multiple-provider world.

Our **NDC Gateway** facilitates the task of accessing multiple NDC providers in a one-to-one integration architecture. This makes easier to interact with the NDC architecture and travel agents transitioning into it.

Our **NDC Gateway** works as a "router" delivering and collecting  concurrently shopping offers from multiple NDC providers (airlines) and deliver an asynchronous real-time aggregation isolating consumers from a lot of cases they won't need to consider such as message wrapping (SOAP, REST...), Authentication issues, host architecture issues, and ancillaries normalization.
Also transitioning among different NDC versions/implementations will be a  much easier process to handle relying our **NDC Gateway**.

Sending requests to our NDC Gateway
------------
We use **HTTP/1.1** to respond requests to the gateway. All NDC requests are sent using the **POST** HTTP method.

The public endpoint to our **NDC Gateway** which remains inmutable for all NDC requests is this:
> http://gtwy.airgateway.net:4000/gtwy/ndc/

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

MPRs get several NDC messages queued responses separated by an XML comment including some useful info regarding provider request like this:

    <!-- AG-Info: ProviderCode: WA | Status: ok | ResponseTime: 114 | ProcessingTime: 0 -->

This can be used to "split" responses before processing them.

**Important note for MPR:**
 
MPRs behave asynchronously when used this HTTP headers:
> **Connection:** keep-alive
> **Keep-Alive:** timeout=xx (where xx is an integer number of seconds)

At this time, this asynchronously mode is the only available mode for **AirShopping** requests.

SPR (Single provider requests)
-------------

To execute a SPR you only need send a valid standard NDC **AirShopping** request including an HTTP header detailing a specific provider like this:
> **AG-Providers:** WA

Authentication
----
All requests sent to the **NDC Gateway**  are required to be consumer-authenticated with a couple of HTTP headers:
> **AG-Consumer**:  {Consumer-Key}
> **AG-Authorization**: {Proxy-key}

----------

AG-Authorization:** {Client-API-Key}


Note:

    We will provide very soon further details on how NDC Authentication (IATA number, Agency ID in NDC messages Party block) must be provided since this could be related to final providers.

NDC Messages
----

We expect plain NDC/XML formatting for the messages. No SOAP or any other wrapping is necessary nor allowed.

This is a list of NDC valid requests to send to our gateway:

- AirShopping
- FlightPrice
- SeatAvailability
- OrderCreate
- OrderRetrieve
- OrderChange
- OrderCancel
- ServiceList
- ServicePrice


Request Samples
---

**MPR request sample (AirShopping)** 

RQ:
```
<AirShoppingRQ xmlns="http://www.iata.org/IATA/EDIST" Version="15.2">
	<Document>
		<Name>NDC Wrapper</Name>
		<ReferenceVersion>1.0</ReferenceVersion>
	</Document>
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
---

RS:
```
<!-- AG-Info: ProviderName: FA | Status: ok | ResponseTime: 115 | ProcessingTime: 0 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <Document>
        <Name>OpenNDC Sandbox</Name>
        <MessageVersion>15.2</MessageVersion>
    </Document>
    <Success/>
    <AirShoppingProcessing/>
    <ShoppingResponseIDs>
        <Owner>FA</Owner>
        <ResponseID>0a366290c7ff480033dda7d402c3864c70171640</ResponseID>
    </ShoppingResponseIDs>
    <OffersGroup>
        <AirlineOffers>
            <TotalOfferQuantity>0</TotalOfferQuantity>
            <Owner>FA</Owner>
        </AirlineOffers>
    </OffersGroup>
</AirShoppingRS>
<!-- AG-Info: ProviderName: IA | Status: ok | ResponseTime: 114 | ProcessingTime: 0 -->
<?xml version="1.0"?>
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <Document>
        <Name>OpenNDC Sandbox</Name>
        <MessageVersion>15.2</MessageVersion>
    </Document>
    <Success/>
    <AirShoppingProcessing/>
    <ShoppingResponseIDs>
        <Owner>IA</Owner>
        <ResponseID>0a366290c7ff480033dda7d402c3864c70171640</ResponseID>
    </ShoppingResponseIDs>
    <OffersGroup>
        <AirlineOffers>
            <TotalOfferQuantity>0</TotalOfferQuantity>
            <Owner>IA</Owner>
        </AirlineOffers>
    </OffersGroup>
</AirShoppingRS>
<!-- AG-Info: ProviderName: WA | Status: ok | ResponseTime: 387 | ProcessingTime: 1 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
    <Document>
        <Name>OpenNDC Sandbox</Name>
        <MessageVersion>15.2</MessageVersion>
    </Document>
    <Success/>
    <AirShoppingProcessing/>
    <ShoppingResponseIDs>
        <Owner>WA</Owner>
        <ResponseID>0a366290c7ff480033dda7d402c3864c70171640</ResponseID>
    </ShoppingResponseIDs>
    <OffersGroup>
        <AirlineOffers>
            <TotalOfferQuantity>2</TotalOfferQuantity>
            <Owner>WA</Owner>
            <AirlineOffer>
                <OfferID Owner="WA">OFFER-3</OfferID>
                <TimeLimits>
                    <OfferExpiration Timestamp="2016-07-15T11:44:57+00:00"/>
                </TimeLimits>
                <TotalPrice>
                    <DetailCurrencyPrice>
                        <Total Code="EUR">1440.00</Total>
                        <Details>
                            <Detail>
                                <SubTotal Code="EUR">1200.00</SubTotal>
                                <Application>Base Fare</Application>
                            </Detail>
                        </Details>
                        <Taxes>
                            <Total Code="EUR">240.00</Total>
                        </Taxes>
                    </DetailCurrencyPrice>
                </TotalPrice>
                <PricedOffer>
                    <OfferPrice OfferItemID="acdad2da29a48348d4bff4f3c31eed9e">
                        <RequestedDate>
                            <PriceDetail>
                                <TotalAmount>
                                    <SimpleCurrencyPrice Code="EUR">1440.00</SimpleCurrencyPrice>
                                    <BaseAmount Code="EUR">1200.00</BaseAmount>
                                    <Taxes>
                                        <Total Code="EUR">240.00</Total>
                                    </Taxes>
                                </TotalAmount>
                            </PriceDetail>
                            <Associations>
                                <AssociatedTraveler>
                                    <TravelerReferences>SH1</TravelerReferences>
                                </AssociatedTraveler>
                                <AssociatedService>
                                    <BundleReference>SB1</BundleReference>
                                </AssociatedService>
                            </Associations>
                            <FareDetail>
                                <FareComponent>
                                    <FareBasis>
                                        <FareBasisCode>
                                            <Code>EFO</Code>
                                        </FareBasisCode>
                                    </FareBasis>
                                </FareComponent>
                            </FareDetail>
                        </RequestedDate>
                    </OfferPrice>
                    <Associations>
                        <ApplicableFlight>
                            <FlightSegmentReference ref="WA0005">
                                <ClassOfService>
                                    <Code>Y</Code>
                                </ClassOfService>
                            </FlightSegmentReference>
                        </ApplicableFlight>
                    </Associations>
                </PricedOffer>
            </AirlineOffer>
            <AirlineOffer>
                <OfferID Owner="WA">OFFER-4</OfferID>
                <TimeLimits>
                    <OfferExpiration Timestamp="2016-07-15T11:44:57+00:00"/>
                </TimeLimits>
                <TotalPrice>
                    <DetailCurrencyPrice>
                        <Total Code="EUR">3870.00</Total>
                        <Details>
                            <Detail>
                                <SubTotal Code="EUR">3225.00</SubTotal>
                                <Application>Base Fare</Application>
                            </Detail>
                        </Details>
                        <Taxes>
                            <Total Code="EUR">645.00</Total>
                        </Taxes>
                    </DetailCurrencyPrice>
                </TotalPrice>
                <PricedOffer>
                    <OfferPrice OfferItemID="acdad2da29a48348d4bff4f3c31eed9e">
                        <RequestedDate>
                            <PriceDetail>
                                <TotalAmount>
                                    <SimpleCurrencyPrice Code="EUR">3870.00</SimpleCurrencyPrice>
                                    <BaseAmount Code="EUR">3225.00</BaseAmount>
                                    <Taxes>
                                        <Total Code="EUR">645.00</Total>
                                    </Taxes>
                                </TotalAmount>
                            </PriceDetail>
                            <Associations>
                                <AssociatedTraveler>
                                    <TravelerReferences>SH1</TravelerReferences>
                                </AssociatedTraveler>
                                <AssociatedService>
                                    <BundleReference>SB1</BundleReference>
                                </AssociatedService>
                            </Associations>
                            <FareDetail>
                                <FareComponent>
                                    <FareBasis>
                                        <FareBasisCode>
                                            <Code>EFO</Code>
                                        </FareBasisCode>
                                    </FareBasis>
                                </FareComponent>
                            </FareDetail>
                        </RequestedDate>
                    </OfferPrice>
                    <Associations>
                        <ApplicableFlight>
                            <FlightSegmentReference ref="WA0005">
                                <ClassOfService>
                                    <Code>J</Code>
                                </ClassOfService>
                            </FlightSegmentReference>
                        </ApplicableFlight>
                    </Associations>
                </PricedOffer>
            </AirlineOffer>
        </AirlineOffers>
    </OffersGroup>
</AirShoppingRS>
<!-- AG-Info: ProviderName: BA | ResponseTime: 3555 | ProcessingTime: 0 -->
<AirShoppingRS xmlns="http://www.iata.org/IATA/EDIST">
    <Document>
        <MessageVersion>1.1.3</MessageVersion>
    </Document>
    <AirShoppingProcessing>
        <Status>COMPLETE</Status>
    </AirShoppingProcessing>
    <ShoppingResponseIDs>
        <ResponseID>2016-07-14T11:45:00.274Z</ResponseID>
    </ShoppingResponseIDs>
    <OffersGroup>
        <AirlineOffers>
            <TotalOfferQuantity>52</TotalOfferQuantity>
            <Owner>BA</Owner>
            <AirlineOffer RequestedDateInd="true">
                <OfferID Owner="BA">OFFER1</OfferID>
                <TotalPrice>
                    <SimpleCurrencyPrice Code="EUR">1418.72</SimpleCurrencyPrice>
                </TotalPrice>
                <PricedOffer>
                    <OfferPrice OfferItemID="1">
                        <RequestedDate>
                            <PriceDetail>
                                <TotalAmount>
                                    <SimpleCurrencyPrice Code="EUR">1418.72</SimpleCurrencyPrice>
                                </TotalAmount>
                                <BaseAmount Code="EUR">1204.00</BaseAmount>
                                <Taxes>
                                    <Total Code="EUR">214.72</Total>
                                </Taxes>
                            </PriceDetail>
                            <Associations>
                                <AssociatedTraveler>
                                    <TravelerReferences>SH1</TravelerReferences>
                                </AssociatedTraveler>
                            </Associations>
                        </RequestedDate>
                    </OfferPrice>
                    <Associations>
                        <ApplicableFlight>
                            <FlightSegmentReference ref="BA2273">
                                <ClassOfService>
                                    <Code>Y</Code>
                                    <MarketingName>World Traveller</MarketingName>
                                </ClassOfService>
                            </FlightSegmentReference>
                        </ApplicableFlight>
                    </Associations>
                </PricedOffer>
            </AirlineOffer>
            <AirlineOffer RequestedDateInd="true">
        </AirlineOffers>

		(...)

    </OffersGroup>
    <DataLists>
        <AnonymousTravelerList>
            <AnonymousTraveler ObjectKey="SH1">
                <PTC>ADT</PTC>
            </AnonymousTraveler>
        </AnonymousTravelerList>
        <DisclosureList>
            <Disclosures ListKey="EuroTraveller">
                <Description>
                    <Metadata MetadataToken="British Airways"/>
                    <Text>Food and bar service</Text>
                </Description>
                <Description>
                    <Metadata MetadataToken="British Airways"/>
                    <Text>30-31" legroom</Text>
                </Description>
                <Description>
                    <Metadata MetadataToken="British Airways"/>
                    <Text>1 x 23kg checked baggage allowance</Text>
                </Description>
				
				(...)
				
            </Disclosures>
        </DisclosureList>
        <FlightSegmentList>
            <FlightSegment SegmentKey="BA2273">
                <Departure>
                    <AirportCode>LGW</AirportCode>
                    <Date>2016-10-01</Date>
                    <Time>16:40</Time>
                    <AirportName>Gatwick (London)</AirportName>
                    <Terminal>
                        <Name>N</Name>
                    </Terminal>
                </Departure>
                <Arrival>
                    <AirportCode>JFK</AirportCode>
                    <Date>2016-10-01</Date>
                    <Time>19:30</Time>
                    <AirportName>John F Kennedy (NY) (New York)</AirportName>
                    <Terminal>
                        <Name>7</Name>
                    </Terminal>
                </Arrival>
                <MarketingCarrier>
                    <AirlineID>BA</AirlineID>
                    <Name>British Airways</Name>
                    <FlightNumber>2273</FlightNumber>
                </MarketingCarrier>
                <OperatingCarrier>
                    <AirlineID>BA</AirlineID>
                    <Name>British Airways</Name>
                </OperatingCarrier>
                <Equipment>
                    <AircraftCode>777</AircraftCode>
                    <Name>Boeing 777 jet</Name>
                </Equipment>
                <FlightDetail>
                    <FlightDuration>
                        <Value>PT7H50M</Value>
                        <Application>FlightTime</Application>
                    </FlightDuration>
                    <Stops>
                        <StopQuantity>0</StopQuantity>
                    </Stops>
                </FlightDetail>
            </FlightSegment>
            <FlightSegment SegmentKey="BA0456">
                <Departure>
                    <AirportCode>LHR</AirportCode>
                    <Date>2016-10-01</Date>
                    <Time>06:15</Time>
                    <AirportName>Heathrow (London)</AirportName>
                    <Terminal>
                        <Name>5</Name>
                    </Terminal>
                </Departure>
                <Arrival>
                    <AirportCode>MAD</AirportCode>
                    <Date>2016-10-01</Date>
                    <Time>09:40</Time>
                    <AirportName>Madrid</AirportName>
                    <Terminal>
                        <Name>4S</Name>
                    </Terminal>
                </Arrival>
                <MarketingCarrier>
                    <AirlineID>BA</AirlineID>
                    <Name>British Airways</Name>
                    <FlightNumber>0456</FlightNumber>
                </MarketingCarrier>
                <OperatingCarrier>
                    <AirlineID>BA</AirlineID>
                    <Name>British Airways</Name>
                </OperatingCarrier>
                <Equipment>
                    <AircraftCode>767</AircraftCode>
                    <Name>Boeing 767 jet</Name>
                </Equipment>
                <FlightDetail>
                    <FlightDuration>
                        <Value>PT2H25M</Value>
                        <Application>FlightTime</Application>
                    </FlightDuration>
                    <Stops>
                        <StopQuantity>0</StopQuantity>
                    </Stops>
                </FlightDetail>
            </FlightSegment>
            
            (...)
            
        </FlightSegmentList>
        <OriginDestinationList>
            <OriginDestination OriginDestinationKey="OD1">
                <DepartureCode>LON</DepartureCode>
                <ArrivalCode>NYC</ArrivalCode>
            </OriginDestination>
        </OriginDestinationList>
    </DataLists>
    <Metadata>
        <Other>
            <OtherMetadata>
                <CurrencyMetadatas>
                    <CurrencyMetadata MetadataKey="EUR">
                        <Decimals>2</Decimals>
                    </CurrencyMetadata>
                </CurrencyMetadatas>
            </OtherMetadata>
        </Other>
    </Metadata>
</AirShoppingRS>
```


**MPR request sample (FlightPrice)**

RQ:
```
<FlightPriceRQ xmlns="http://www.iata.org/IATA/EDIST" Version="15.2">
  <Document>
    <MessageVersion>1.1.3</MessageVersion>
  </Document>
  <Travelers>
    <Traveler>
      <AnonymousTraveler ObjectKey="SH1">
        <PTC>ADT</PTC>
      </AnonymousTraveler>
    </Traveler>
  </Travelers>
  <Query>
    <OriginDestination>
      <Flight>
        <Departure>
          <AirportCode>LHR</AirportCode>
          <Date>2017-11-20</Date>
        </Departure>
        <Arrival>
          <AirportCode>AMS</AirportCode>
          <Date>2017-11-20</Date>
        </Arrival>
        <MarketingCarrier>
          <AirlineID>BA</AirlineID>
          <FlightNumber>0430</FlightNumber>
        </MarketingCarrier>
        <Equipment>
          <AircraftCode>744</AircraftCode>
        </Equipment>
        <ClassOfService>
          <Code>H</Code>
          <MarketingName>Euro Traveller</MarketingName>
        </ClassOfService>
      </Flight>
    </OriginDestination>
    <OriginDestination>
      <Flight>
        <Departure>
          <AirportCode>AMS</AirportCode>
          <Date>2017-11-27</Date>
        </Departure>
        <Arrival>
          <AirportCode>LHR</AirportCode>
          <Date>2017-11-27</Date>
        </Arrival>
        <MarketingCarrier>
          <AirlineID>BA</AirlineID>
          <FlightNumber>0429</FlightNumber>
        </MarketingCarrier>
        <Equipment>
          <AircraftCode>744</AircraftCode>
        </Equipment>
        <ClassOfService>
          <Code>H</Code>
          <MarketingName>Euro Traveller</MarketingName>
        </ClassOfService>
      </Flight>
    </OriginDestination>
  </Query>
  <Metadata>
    <Other>
      <OtherMetadata>
        <LanguageMetadatas>
          <LanguageMetadata MetadataKey="Display">
            <Application>Display</Application>
            <Code_ISO>en</Code_ISO>
          </LanguageMetadata>
        </LanguageMetadatas>
      </OtherMetadata>
    </Other>
  </Metadata>
</FlightPriceRQ>
```

RS:


