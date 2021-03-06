# Price-api

# 0. Autentisering
### Begär en token för vidare anrop
Alla anrop till API:t behöver åtföljas av ett headerattribut (Authorization) som innehåller "Bearer" + den token som erhölls i Auth-anropet

***Request:***

```http
GET /api/1.0/auth HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
{
	"accessToken": "", // Json Web Token att skicka in till efterföljande anrop
	"expiresIn": "", // Number by client
	"impersonating": "47193bdf-7855-499b-96a1-f9d387556a9e" // Om autentiserad användare impersonerar annan
}
```


# 1. Feasibility API
### Hämta alla levererbara platser

***Request:***

```http
GET /api/1.0/feasibility HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Sun, 10 Jan 2016 12:03:28 GMT
Content-Type: application/json
```
```javascript
[
	{
		"pointId": "NORR1234", // Name of feasability point
		"address": {
			"street": "Testvägen",
			"number": "100",
			"littera": "",
			"postalCode": "10000",
			"city": "Ankeborg",
			"countryCode" : "SE"
		},
		"building": {
			"distinguisher": "", // "Set if something is needed to distinguish a specific building on the address."
			"type": "MDU" // "'MDU' = apartment building, 'SDU' = villa"
		},
		"realEstate": {
			"label": "PENGABINGEN 1",
			"municipality": "ANKEBORG"
		},
		"coordinate": {
			"latitude": 6581619.085,
			"longitude": 1628539.32,
			"projection": "" // Can be WGS84, RT90 or SWEREF90 or variants of these
		},
		"district": "GAMLA STAN",
		"suppliers": [ // "Optional; should only be provided if not heavy too calculate."
			{
				"name": "STOKAB",
				"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
				"statusValidationRequired": true // "Indicates if the fiberStatus needs manual validation to assure availability."
			}
		],
 		"relatedPointIds": [
			"CDE678",
			...
		],
		"pointInfo": {
			"name": "NORR1234", // Name of point, same as pointId
			"type": "", // Node type, can be either: N, H or HK
			"aNode": "", // Name of connection node if available
			"oNode": "", // Name of area node if available
		}
	},
	...
]
```


# 2.1 Availability API - Med punktid
### Hämta detaljerad information och leverantörers fiberstatus på en specifik plats med punktid där enbart en eller ingen träff kan förekomma.

***Request:***

Sökning med punktid efter specifik resurs

```http
GET /api/1.0/availability/{pointId} HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
{
	"pointId": "ABC123",
	"address": {
		"street": "Testvägen",
		"number": "100",
		"littera": "",
		"postalCode": "10000",
		"city": "Ankeborg",
		"countryCode": "SE"
	},
	"building": {
		"distinguisher": "", // "set if something is needed to distinguish a specific building on the address"
		"type": "MDU" // "'MDU' = apartment building, 'SDU' = villa"
	},
	"realEstate": {
		"label": "PENGABINGEN 1",
		"municipality": "ANKEBORG"
	},
	"coordinate": {
		"latitude": 6581619.085,
		"longitude": 1628539.32,
		"projection": "" // Can be WGS84, RT90 or SWEREF90 or variants of these
	},
	"district": "GAMLA STAN",
	"suppliers": [
		{
			"name": "STOKAB",
			"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
			"statusValidationRequired": true // "indicates if the fiberStatus needs manual validation to assure availability"
		}
	],
	"relatedPointIds": [ // "list of other nearby points (addresses) which could be used instead of the searched point (address)"
		"CDE678",
		...
	]
}
```

# 2.2 Availability API - Med adress
### Hämta detaljerad information och leverantörers fiberstatus på en eller flera platser utifrån en adressökning

***Request:***

Sökning med punktid efter specifik resurs

```http
GET /api/1.0/availability/?city=&street=&number=&littera= HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"pointId": "ABC123",
		"address": {
			"street": "Testvägen",
			"number": "100",
			"littera": "",
			"postalCode": "10000",
			"city": "Ankeborg",
			"countryCode": "SE"
		},
		"building": {
			"distinguisher": "", // "set if something is needed to distinguish a specific building on the address"
			"type": "MDU" // "'MDU' = apartment building, 'SDU' = villa"
		},
		"realEstate": {
			"label": "PENGABINGEN 1",
			"municipality": "ANKEBORG"
		},
		"coordinate": {
			"latitude": 6581619.085,
			"longitude": 1628539.32,
			"projection": "" // Can be WGS84, RT90 or SWEREF90 or variants of these
		},
		"district": "GAMLA STAN",
		"suppliers": [
			{
				"name": "STOKAB",
				"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
				"statusValidationRequired": true // "indicates if the fiberStatus needs manual validation to assure availability"
			}
		],
		"relatedPointIds": [ // "list of other nearby points (addresses) which could be used instead of the searched point (address)"
			"CDE678",
			...
		]
	},
	...
]
```

# 3. Price Estimate API
### Hämta prisuppskattning på en förbindelse

***Request:***

```http
POST /api/1.0/priceEstimate HTTP/1.1
Content-Type: application/json
```
```javascript
{
	"invoiceGroupId": "", // GUID of invoice group for price estimate
	"frameworkAgreementId": "", // GUID of framework agreement to use for price estimate
	"from": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"comment": ""
	},	
	"to": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"comment": ""
	},
	"redundancy": {
		"type": "Full", /* "'Normal', 'Full'" */
		"toPointId": "" /* May be set to null */
	},
	"customerType": "Commercial", /* E.g. 'Commercial', 'Residential' */
	"serviceLevel": "Premium", /* E.g. 'Base', 'Gold', 'Premium' */
	"products": "All", /* E.g. 'All', 'Point2Point', 'Star' */
	"productId": "", // GUID of product if price estimate is to be done on one specific product 
	"parameters": [ 
		{
			"name": "ConnectorType",
			"value": "SC/APC"
		},
		{
			"name": "NumberOfRooms",
			"value": "10"
		},
		...
	],
	"subProducts": [ 	/* Will not be implemented by Stokab */
		{
			"productId": "",
			"name": "",
			"parameters": [ 
				{
					"name": "ConnectorType",
					"value": "SC/APC"
				},
				{
					"name": "NumberOfRooms",
					"value": "10"
				},
				...
			]
		}
		...
	],
	"contractPeriodMonths": 12, /* 1, 3 or 5 years (in months). */
	"noOfSingleFibers": 1, /* Number of wanted single fibers */
	"NumberOfFiberPairs": 1 /* Number of wanted fiber pairs */
}
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"supplier": "STOKAB",
		"products": [
			{
				"productId" : "8u3-3563-3635-365-ff",
				"name": "Point2Point",
				"status": "AVAILABLE",
				"comment": "",
				"items": [ 
					{
						"name": "distance",
						"value": "987"
					},
					...
				],
				"subProducts": [
					{
						"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
						"name": "APC",
						"price": {
							"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
							"oneTimeFee": 1100.0,
							"monthlyFee": 100.0
							"items": [ 
								{
									"name": "Connection based on distance",
									"parameters": [
										{
											"name": "distance",
											"value": "987"
										},
										...
									],
									"oneTimeFee": 5000.0,
									"monthlyFee": 1000.0
								}
							]
						}
					},
					...
				],
				"price": { // NOTE: Pricing on sub products not implemented by Stokab
					"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
					"oneTimeFee": 15100.0,
					"monthlyFee": 1200.0,
					"items": [ 
						{
							"name": "Connection based on distance",
							"parameters": [
								{
									"name": "distance",
									"value": "987"
								},
								...
							],
							"oneTimeFee": 5000.0,
							"monthlyFee": 1000.0
						}
						...
					]
				},
				"pricingMessage": "",
				"optionalProducts": [
					{
						"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
						"name": "APC",
						"price": { // NOTE: Pricing on optional products not implemented by Stokab
							"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
							"oneTimeFee": 1100.0,
							"monthlyFee": 100.0
							"items": [ 
								{
									"name": "Connection based on distance",
									"parameters": [
										{
											"name": "distance",
											"value": "987"
										},
										...
									],
									"oneTimeFee": 5000.0,
									"monthlyFee": 1000.0
								}
							]
						}
					}
					...
				],
			}
			...
		]
	}
	...
]
```


# 4. Offer Inquiry API
### Skicka en offert-förfrågan för en förbindelse av en specifik produkt från en specifik leverantör

***Request:***

```http
POST /api/1.0/offerInquiry HTTP/1.1
Content-Type: application/json
```
```javascript
{
	"responsiblePersonEmail": "anders.larsson@comhem.com",
	"invoiceGroupId": "", // GUID of invoice group for price estimate
	"frameworkAgreementId": "", // GUID of framework agreement to use for price estimate
	"supplier": "STOKAB",
	"product" : {
		"productId": "0d13c5e0-ce23-41a0-87b5-f480479fa71e", 
		"name": "Point2Point",
	},	
	"referenceId": "CH-12345", /* "client own reference for this inquiry, could be empty" */
	"from": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"comment": ""
	},	
	"to": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"comment": ""
	},
	"comment": "Lorem ipsum", /* When comment is present the request will be async */
	"redundancy": {
		"type": "Full", /* 'NoReduncancy', 'Normal'. (Full redundancy not availible from price-api.) */
		"toPointId": "CBA123"
	},	
	"customerType": "Commercial", /* "e.g. 'Commercial', 'Residential'" */
	"serviceLevel": "Premium", /* "e.g. 'Base', 'Gold', 'Premium'" */
	"parameters": [  
		{
			"name": "ConnectorType",
			"value": "SC/APC"
		},
		{
			"name": "NoOfRooms",
			"value": "10"
		}
		...
	],
	"subProducts": [
		{
			"productId": "",
			"name": "",
			"parameters": [ 
				{
					"name": "ConnectorType",
					"value": "SC/APC"
				},
				{
					"name": "NumberOfRooms",
					"value": "10"
				},
				...
			]
		}
		...
	],
	"contractPeriodMonths": 12,
	"noOfFiberPairs": 1, /* "Number of wanted fiber pairs */
	"noOfSinglePairs": 1, /* "Number of wanted single fibers */
	"asyncAnswerAllowed": true /* "If asychronous answer is ok." */
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Location: /api/1.0/inquiries/ec4bc754-6a30-11e2-a585-4fc569183061
Content-Type: application/json
```
```javascript
{
	"inquiryId": "47193bdf-7855-499b-96a1-f9d387556a9e",
	"inquiryNumber": "SP-892383", /* Incident-number in CRM. */
	"referenceId": "CH-12345",
	"status": {
		"state": "WAIT_ASYNC_ANSWER", /* "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'" */
		"message": "",
		"orderDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	},
	"supplier": "STOKAB",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "", /* "May be set to the identifier for the connection if that is already generated when inquiry is answered." */
	"deliveryDurationDays": 20, /* "Days from order to delivered connection." */
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
		"name": "Point2Point",
		"price": {
			"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
			"oneTimeFee": 15100.0,
			"monthlyFee": 1200.0,
			"items": [
				{
					"name": "Connection based on distance",
					"parameters": [
						{
							"name": "distance",
							"value": "987"
						},
						...
					],
					"oneTimeFee": 5000.0,
					"monthlyFee": 1000.0
				}
				...
			]
		}
	},
	"asyncStatusMessage": ""
}
```

### Hämta uppdatering på en skickad offert-förfrågan

***Request:***

```http
GET /api/1.0/inquiries/ec4bc754-6a30-11e2-a585-4fc569183061 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Mon, 11 Jan 2015 12:03:28 GMT
Content-Type: application/json
```
```javascript
{
	"inquiryId": "47193bdf-7855-499b-96a1-f9d387556a9e",
	"inquiryNumber": "SP-892383", /* Incident-number in CRM. */
	"referenceId": "CH-12345",
	"status": {
		"state": "WAIT_ASYNC_ANSWER", /* "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'" */
		"message": "",
		"orderDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	},
	"supplier": "STOKAB",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "", /* "May be set to the identifier for the connection if that is already generated when inquiry is answered." */
	"deliveryDurationDays": 20, /* "Days from order to delivered connection." */
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
		"name": "Point2Point",
		"price": {
			"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
			"oneTimeFee": 15100.0,
			"monthlyFee": 1200.0,
			"items": [
				{
					"name": "Connection based on distance",
					"parameters": [
						{
							"name": "distance",
							"value": "987"
						},
						...
					],
					"oneTimeFee": 5000.0,
					"monthlyFee": 1000.0
				}
				...
			]
		}
	},
	"asyncStatusMessage": ""
}
```

### Hämta slutförda förfrågningar som krävde asynkront svar

Anropet skall returnera samtliga förfrågningar som skett sedan since. Om en förfrågan har exakt angiven tidpunkt som doneDateTime så skall den ordern inte ingå i svaret.
Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.
Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på förfrågningar, som kan ändra status, utan endast innefattar sådana förfrågningar som har nått en slutstatus.
Listan över förfråningar skall vara stigande i kronologisk ordning (senaste förfrågan som har förändrats sist). Klienten kan då använda det sista lästa förfrågan som since-parameter i nästföljande anrop.
Enbart förfrågningar som har status DONE_ASYNC_ANSWER_SUCCESS eller DONE_ASYNC_ANSWER_FAILED får förekomma i svaret.
Om inte since skickas med så skall alla slutförda förfrågningar returneras.
För att hämta detaljerad information om respektive förfrågan, använd Hämta uppdatering på en skickad förfrågan.

***Request:***

```http
GET /api/1.0/inquiries?since=2016-08-22T15:01:02.000Z
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
		"orderId": "ec4bc754-6a30-11e2-a585-4fc569183061",
		"supplier": "STOKAB",
		"product": {
			"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
			"name": "Point2Point",
			"price": {
				"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
				"oneTimeFee": 15100.0,
				"monthlyFee": 1200.0,
				"items": [
					{
						"name": "Connection based on distance",
						"parameters": [
							{
								"name": "distance",
								"value": "987"
							},
							...
						],
						"oneTimeFee": 5000.0,
						"monthlyFee": 1000.0
					}
					...
				]
			}
		},
		"status": {
			"state": "WAIT_ASYNC_ANSWER", /* "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'" */
			"message": "",
			"orderDateTime": "2016-08-21T08:01:06.000Z",
			"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
		}
	},
	...
]
```


# 5. Order API
### Lägg en beställning på en tidigare mottagen offert och komplettera med kund och fakturainformation.

***Request:***

```http
POST /api/1.0/orders HTTP/1.1
Content-Type: application/json
```
```javascript
{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"responsiblePersonEmail": "anders.larsson@comhem.com",
	"endCustomer": {
		"companyName" : "HSB",
		"firstName": "Sven",
		"lastName": "Svensson",
		"phoneNumber": "46702233445",
		"email": "sven.svensson@company.com"
	}
}
```

***Response:***

```http
HTTP/1.1 201 CREATED
Last-Modified: Mon, 11 Jan 2015 12:05:28 GMT
Location: /api/1.0/orders/fc6cd754-6a30-11e2-a585-4fc569185689
Content-Type: application/json
```
```javascript
{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"supplier": "STOKAB",
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
		"name": "Point2Point",
		"price": {
			"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
			"oneTimeFee": 15100.0,
			"monthlyFee": 1200.0,
			"items": [
				{
					"name": "Connection based on distance",
					"parameters": [
						{
							"name": "distance",
							"value": "987"
						},
						...
					],
					"oneTimeFee": 5000.0,
					"monthlyFee": 1000.0
				}
				...
			]
		}
	},
	"status": {
		"state": "WAIT_ASYNC_ANSWER", /* "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'" */
		"message": "",
		"orderDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	}
}
```

### Hämta uppdatering på en lagd order

***Request:***

```http
GET /api/1.0/orders/fc6cd754-6a30-11e2-a585-4fc569185689 HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Last-Modified: Tue, 15 Feb 2015 15:12:53 GMT
Content-Type: application/json
```
```javascript
{
	"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"inquiryId": "fc6cd754-6a30-11e2-a585-4fc569185689",
	"supplier": "STOKAB",
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
		"name": "Point2Point",
		"price": {
			"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER' */
			"oneTimeFee": 15100.0,
			"monthlyFee": 1200.0,
			"items": [
				{
					"name": "Connection based on distance",
					"parameters": [
						{
							"name": "distance",
							"value": "987"
						},
						...
					],
					"oneTimeFee": 5000.0,
					"monthlyFee": 1000.0
				}
				...
			]
		}
	},
	"status": {
		"state": "WAIT_ASYNC_ANSWER", /* "'DONE_SUCCESS', 'DONE_FAILED', 'DONE_ASYNC_ANSWER_SUCCESS', 'DONE_ASYNC_ANSWER_FAILED'" */
		"message": "",
		"orderDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	}
}
```

### Hämta slutförda ordrar

Anropet skall returnera samtliga ordrar som skett sedan since. Om en order har exakt angiven tidpunkt som doneDateTime så skall den ordern inte ingå i svaret.
Listan är oföränderlig. Vid två tidpunkter, t1 och t2, där t2 inträffar efter t1, skall tjänstens svar vid t2 alltid omfatta hela t1. Svaret kan alltså enbart växa, inte förändras.
Av denna anledningen är det viktigt att tjänstens svar inte är en projektion på ordrar, som kan ändra status, utan endast innefattar sådana ordrar som har nått en slutstatus.
Listan över ordrar skall vara stigande i kronologisk ordning (senaste order som har förändrats sist). Klienten kan då använda det sista lästa orderns  som since-parameter i nästföljande anrop.
Enbart ordrar som har status DELIVERED eller REJECTED får förekomma i svaret.
Om inte since skickas med så skall alla slutförda ordrar returneras.
För att hämta detaljerad information om respektive order, använd Hämta uppdatering på en lagd order.

***Request:***

```http
GET /api/1.0/orders?since=2016-08-22T10:09:23.000Z
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"orderId": "fc6cd754-6a30-11e2-a585-4fc569185689",
		"state": "DELIVERED",
		"message": ""
	},
	...
]
```


# 6. Invoice Group
### Hämta alla fakturamottagare för specifik kund

***Request:***

```http
GET /api/1.0/invoiceGroup HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"name": "Client",
		"clientNumber": "9834", // Number by client
		"invoiceGroupid": "47193bdf-7855-499b-96a1-f9d387556a9e",
		"invoiceGroupNumber" : "1122" // Unique number per invoice group
	},
	...
]
```


# 7. Framework Agreement
### Hämta alla ramavtal för specifik kund

***Request:***

```http
GET /api/1.0/frameworkAgreement HTTP/1.1
```

***Response:***

```http
HTTP/1.1 200 OK
Content-Type: application/json
```
```javascript
[
	{
		"name": "Client", // Name of the framework agreement
		"isStandard": true, // If the default framework agreement, true or false
		"frameworkAgreementId": "47193bdf-7855-499b-96a1-f9d387556a9e",  // Unique number per invoice group
		"masterSystemId" : "47193bdf-7855-499b-96a1-f9d387556a9e" // Internal id
	},
	...
]
