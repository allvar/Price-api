# Price-api
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
		"address": {
			"street": "Testvägen",
			"number": "100",
			"littera": "",
			"postalCode": "10000",
			"city": "Ankeborg"
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
			"projection": "WGS84"
		},
		"district": "GAMLA STAN",
		"suppliers": [ // "Optional; should only be provided if not heavy too calculate."
			{
				"name": "Supplier",
				"fiberStatus": "IN_REAL_ESTATE", // "'AT_ADDRESS', 'AT_SITE_BOUNDARY'"
				"statusValidationRequired": true // "Indicates if the fiberStatus needs manual validation to assure availability."
			}
		],
 		"relatedPointIds": [
			"CDE678",
			"CDE901"
		]
	},
	...
]
```


# 2. Price Estimate API
### Hämta prisuppskattning på en förbindelse

***Request:***

```http
POST /api/1.0/priceEstimates HTTP/1.1
Content-Type: application/json
```
```javascript
{		
	"from": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"address": { 
			"city" : "Stockholm",
			"street": "Drottninggatan",
			"number": "52",
			"littera": "A",
			"zipCode" : "11134",
			"comment": "",
			"isCustomerNode": true /* This would be specific and only implemented if no other soultion can be found for customer-nodes */
		}	
	},
	"to": {  /* May be set to null if any product only requires one point (address). E.g. Star. */ 
		"pointId": "ABC123",
		"address": { 
			"city" : "Stockholm",
			"street": "Drottninggatan",
			"number": "52",
			"littera": "A",
			"zipCode" : "11134",
			"comment": "",
			"redundancy": {
				"type": "Full", /* "'Normal', 'Full'" */
				"toPointId": "" /* May be set to null */
			}
		}
	},
	"customerType": "Commercial", /* E.g. 'Commercial', 'Residential' */
	"serviceLevel": "Premium", /* E.g. 'Base', 'Gold', 'Premium' */
	"products": ["All"], /* E.g. 'All', 'Point2Point', 'Star' */
	"parameters": [ 
		{
			"name": "ConnectorType",
			"value": "SC/APC"
		},
		{
			"name": "NumberOfRooms",
			"value": "10"
		},
		[...]
	],
	"subProducts": [ /* Will not be implemented by innofactor*/
	...
	],
	"contractPeriodMonths": 12, /* 1, 3 or 5 years (in months). */
	"noOfFibers": 1 /* Number of wanted fiber pairs or single fibers depending on product.  */
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
		"supplier": "supplier",
		"products": [
			{
				"productId" : "8u3-3563-3635-365-ff",
				"name": "Point2Point",
				"actualAddress": { // May be null if exact match on address was found
					"city" : "Stockholm",
					"street": "Drottninggatan",
					"number": "52",
					"littera": "A",
					"zipCode" : "11134"
				},
				"status": "AVAILABLE",
				"comment": "",
				"items": [ 
					{
						"name": "distance",
						"value": "987"
					},
					[...]
				],
				"subProducts": [
					{
						"productId": "51dceb13-8d25-4f5f-adb1-29169d6042e2",
						"name": "APC",
						"price": {
							"oneTimeFee": 1100.0,
							"monthlyFee": 100.0
						}
					},
					[...]
				],
				"price": {
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
								[...]
							],
							"oneTimeFee": 5000.0,
							"monthlyFee": 1000.0
						},
						{
							"name": "Service level Premium",
							"parameters": [...],
							"oneTimeFee": 0.0,
							"monthlyFee": 200.0
						},
						{
							"name": "Connector",
							"parameters": [...],
							"oneTimeFee": 100.0,
							"monthlyFee": 0.0
						},
						{
							"name": "ResidentialNetwork",
							"parameters": [
								{
									"name": "NumberOfRooms",
									"value": "10"
								}
							],
							"oneTimeFee": 10000.0,
							"monthlyFee": 0.0
						},
						[...]
					]
				}
				
			},
			{
				"productId": "2b99bcfb-b563-4b04-8c8a-8fd636f56d67",
				"name": "Star",
				"actualAddress" : { // May be null if exact match on address was found
					"city" : "Stockholm",
					"street": "Drottninggatan",
					"number": "52",
					"littera": "A",
					"zipCode" : "11134",
				},
				"status": "POSSIBLE_WITH_CONDITIONS", // "'NOT_AVAILABLE', 'AVAILABLE'"
				"comment": "Connection to anslutningsnod is necessary for address to be available",
				"items": [],
				"subProducts": [],
				"price": null
			},
			{
				"productId": "87sg-098dsaf-098sudg-098gf",
				"name": "Star",
				"actualAddress" : { // May be null if exact match on address was found
					"city" : "Stockholm",
					"street": "Drottninggatan",
					"number": "52",
					"littera": "A",
					"zipCode" : "11134"
				},
				"status": "POSSIBLE_WITH_CONDITIONS", // "'NOT_AVAILABLE', 'AVAILABLE'"
				"comment": "Connection to anslutningsnod is necessary for address to be available",
				"items": [],
				"subProducts": [],
				"price": null
			},
			[...]
		],
	},
	[...]
]
```

# 3. Offer Inquiry API
### Skicka en offert-förfrågan för en förbindelse av en specifik produkt från en specifik leverantör

***Request:***

```http
POST /api/1.0/offerInquiry HTTP/1.1
Content-Type: application/json
```
```javascript
{
	"supplier": "supplier",
	"product" : {
		"productId": "0d13c5e0-ce23-41a0-87b5-f480479fa71e", 
		"name": "Point2Point",
	},	
	"referenceId": "CH-12345", /* "client own reference for this inquiry, could be empty" */
	"fromPointId": "ABC123"
	"toPointId": "ABC124",
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
			"name": "NoOfRooms",
			"value": "10"
		},
		[...]
	],
	"subProducts": [
		{
			"productId": "512b7502-fb41-496d-86b6-34d4562cd9a6",
			"name": "APC"
		},
	[...]
	],
	"contractPeriodMonths": 12,
	"noOfFibers": 1, /* "Number of wanted fiber pairs (or single fibers depending on product)" */
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
		"createDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	},
	"supplier": "supplier",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "", /* "May be set to the identifier for the connection if that is already generated when inquiry is answered." */
	"deliveryDurationDays": 20, /* "Days from order to delivered connection." */
	"product": {
		"productId": "bd16d8d5-c147-4490-8b7a-93193fce8fb3",
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
				"productId": "5f3e0024-77aa-42df-9faf-2088d328975c",
				"name": "awesomeProduct",
				"price": {
				"oneTimeFee": "1287.99",
				"monthlyFee": "283.00"
				}
			},
			[...]
		],
		"price": {
			"status": "ESTIMATED", /* "'ESTIMATED', 'OFFER'. Estimated price can be delivered in synchronous answer and then be overridden by an offer in an asynchronous answer" */
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
				},
				{
					"name": "Service level Premium",
					"parameters": [...],
					"oneTimeFee": 0.0,
					"monthlyFee": 200.0
				},
				{
					"name": "Connector",
					"parameters": [...],
					"oneTimeFee": 100.0,
					"monthlyFee": 0.0
				},
				{
					"name": "ResidentialNetwork",
					"parameters": [
						{
							"name": "NoOfRooms",
							"value": "10"
						}
					],
					"oneTimeFee": 10000.0,
					"monthlyFee": 0.0
				},
				...
			]
			
		}
	}
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
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"inquiryNumber": "ST-9847598475", 
	"referenceId": "CH-12345",
	"status": {
		"state": "DONE_ASYNC_ANSWER_SUCCESS",
		"message": "",
		"createDateTime": "2016-08-21T08:01:06.000Z",
		"doneDateTime": "2016-08-22T10:15:01.000Z", /* "Should be null if not yet done." */
	},
	"supplier": "supplier",
	"offerValidUntilDate": "2016-01-31",
	"connectionId": "",
	"deliveryDurationDays": 20,
	"product": {
		"productId": "141c78ce-acfa-4f77-aebc-1ab94327872a",
		"name": "Point2Point",
		"status": "AVAILABLE",
		"comment": "",
		"items": [ 
			{
				"name": "distance",
				"value": "1046"
			},
			...
		],
		"subProducts": [
		{
			"productId": "141c78ce-acfa-4f77-aebc-1ab94327872a",
			"name": "awesomeProduct",
			"price": {
				"oneTimeFee": "1287.99",
				"monthlyFee": "283.00"
			}
		}...
		],
		"price": {
			"status": "OFFER",
			"oneTimeFee": 15100.0,
			"monthlyFee": 1500.0,
			"items": [ 
				{
					"name": "Connection based on distance",
					"parameters": [
						{
							"name": "distance",
							"value": "1046"
						},
						...
					],
					"oneTimeFee": 5000.0,
					"monthlyFee": 1300.0
				},
				{
					"name": "Service level Premium",
					"parameters": [...],
					"oneTimeFee": 0.0,
					"monthlyFee": 200.0
				},
				{
					"name": "Connector",
					"parameters": [...],
					"oneTimeFee": 100.0,
					"monthlyFee": 0.0
				},
				{
					"name": "ResidentialNetwork",
					"parameters": [
						{
							"name": "NoOfRooms",
							"value": "10"
						}
					],
					"oneTimeFee": 10000.0,
					"monthlyFee": 0.0
				},
				...
			]
		}
	}
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
		"inquiryNumber": "ST-9847598475", 
		"referenceId": "CH-12345",
		"state": "DONE_ASYNC_ANSWER_SUCCESS",
		"message": "",
		"toAddress": { 
			"city" : "Stockholm",
			"street": "Drottninggatan",
			"number": "68",
			"littera": "A",
			"zipCode": "11244",
			"comment": ""
		},
		"product": "Star",
		"price": {
			"oneTimeFee" : 15000,
			"monthlyFee": 3000
		}
	},
	{
		"inquiryId": "fc4bc754-6a30-11e2-a585-4fc569183432",
		"state": "DONE_ASYNC_ANSWER_FAILED",
		"message": "An error occured",
		"toAddress": { 
			"city" : "Stockholm",
			"street": "Drottninggatan",
			"number": "68",
			"littera": "A",
			"zipCode": "11244",
			"comment": ""
		},
		"product": "Star",
		"price": {
			"oneTimeFee" : 15000,
			"monthlyFee": 3000
		}
	},
	...
]
```

# 4. Order API
### Lägg en beställning på en tidigare mottagen offert och komplettera med kund och fakturainformation.

***Request:***

```http
POST /api/1.0/orders HTTP/1.1
Content-Type: application/json
```
```javascript
{
	"inquiryId": "ec4bc754-6a30-11e2-a585-4fc569183061",
	"responsiblePerson": {
		"firstName": "Anders",
		"lastName": "Larsson",
		"phoneNumber": "46701234567",
		"email": "anders.larsson@comhem.com"
	},
	"endCustomer": {
		"companyName" : "HSB",
		"firstName": "Sven",
		"lastName": "Svensson",
		"phoneNumber": "46702233445",
		"email": "sven.svensson@company.com"
	},
	"invoiceGroup": "IK-12345"
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
	"orderNumber": "ST-9847598475", 
	"referenceId": "CH-12345",
	"supplier": "supplier",
	"product": "Point2Point",
	"status": {
		"state": "ORDERED",
		"message": "",
		"orderDateTime": "2015-01-11T11:34:00.000Z"
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
	"orderNumber": "ST-9847598475", 
	"referenceId": "CH-12345",
	"supplier": "supplier",
	"product": "Point2Point",
	"status": {
		"state": "DELIVERED", /* "ORDERED", "DELIVERED", "REJECTED" */
		"message": "",
		"orderDateTime": "2015-01-11T11:34:00.000Z",
		"doneDateTime": "2015-02-15T14:49:12.000Z"
	},
	"responsiblePerson": {
		"firstName": "Anders",
		"lastName": "Larsson",
		"phoneNumber": "46701234567",
		"email": "anders.larsson@comhem.com"
	},
	"endCustomer": {
		"companyName": "HSB",
		"firstName": "Sven",
		"lastName": "Svensson",
		"phoneNumber": "46702233445",
		"email": "sven.svensson@company.com"
	},
	"invoiceGroup": "IK-12345"
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
		"orderNumber": "ST-9847598475", 
		"referenceId": "CH-12345",
		"state": "DELIVERED",
		"message": ""
	},
	{
		"orderId": "ab6cd754-6a30-11e2-a585-4fc569185643",
		"orderNumber": "ST-9847598475", 
		"referenceId": "CH-12345",
		"state": "REJECTED",
		"message": "The order couldn't be delivered because of no available fibers"
	},
	...
]
```
# 5. Invoice Group
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
		"name": "ComHem",
		"clientNumber": "9834",
		"invoiceGroupid": "47193bdf-7855-499b-96a1-f9d387556a9e"

	},
	...
]
