# BrokerWS
## _BrokerWS is a Trading Interface for AmiBroker using Json &amp; WebSocket_

[![Build Status](https://raw.githubusercontent.com/ideepcoder/BrokerWS/0e9daa2d2d7c53653b90e0475e46b92167e60a74/images/unknown.svg)](https://github.com/ideepcoder/BrokerWS)
Doc version: 1.o, Build: 1.4.0

## Features
- Bi-directional websocket communication
- Easy to integrate Json message structure
- Derived from AmiBroker Automatic Trading Interface

### USAGE INSTRUCTIONS
The WS Controller (BrokerWS.EXE) is a separate application that acts as a buffer between AmiBroker and your CLIENT-APP (See the WsRtd project for details).
Copy the executable and XML file to the AmiBroker Installation directory.
**Important** "BrokerWS.exe" > Run as Administrator, for the first time to register itself and create the necessary file & Windows Registry entries.

**This is a draft, any format,field,code is subject to change**

### https://www.amibroker.com/at/
Kindly familiarize with AmiBroker Auto-Trading Interface described here, as BrokerWS emulates the same design and flow. Repition of controller interface examples is omitted here.

### AFL usage
Create object in your AFL file like this:
Orders are auto transmitted for now, your Client-APP can choose what to do.

```sh
wsc = GetTradingInterface( "WS" );
if( wsc.IsConnected() )		// check if connection to IB was successfull
{
   wsc.PlaceOrder( "MSFT", "BUY", 100, "MKT", 0, 0, "DAY", 1 );
}
```

### AmiBroker Place Order Window
https://www.amibroker.com/guide/w_placeorder.html

# REQUEST STRUCTURES

## 1) Request IDs
Request order status by orderId

|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |reqIds| |
|numIds| int |10000| |

```sh
{"ti":"reqIds","numIds":10000}
```

## 2) Request Auto Open Orders
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |reqAutoOpenOrders| |
|bAutoBind| int |true| ? unkn |

```sh
{"ti":"reqAutoOpenOrders","bAutoBind":true}
```

## 3) Request Account Updates
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |reqAccountUpdates||
|accountCode| string | | | |

```sh
{"ti":"reqAccountUpdates","accountCode":""}
```

## 4) Request All open Orders
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |requestAllOpenOrders| |

```sh
{"ti":"requestAllOpenOrders"}
```

## 5) Cancel Order
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |cancelOrder||
|orderId| int |1005| |

```sh
{"ti":"cancelOrder","orderId":1005}
```

## 6) Place or Modify Order
See response structure #6 for description
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti| string |placeOrder| |
|orderId| int |1000|
|symbol| string |ZZZ1| |
|secType| string |STK| |
|expiry| string | | |
|strike| int |0.0|
|exchange| string |NSE| |
|currency| string |INR| |
|action| string |BUY| |
|quantity| int |100|
|orderType| string |LIMIT| |
|lmtPrice| int |25.28|
|auxPrice| int |25.28|
|tif| string |DAY||
|parentId| int |0| |

```sh
{
"ti":"placeOrder","orderId":1000,"symbol":"ZZZ1","secType":"STK","expiry":"",
"strike":0.0,"exchange":"NSE","currency":"INR","action":"BUY","quantity":100,
"orderType":"LIMIT","lmtPrice":25.28,"auxPrice":25.28,"tif":"DAY","parentId":0
}
```


# RESPONSE STRUCTURES

## 1) Account Info Tab
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string|ACCT_VALUE| response type|
|key|			string| key|			Map new order to this account, OPEN_ORDER.account|
|val|			string| value|		Account value (cash) numerical |
|curr|			string| currency|    INR, USD etc |
|accountName|	string| accountName|   Account Holder name |

Sample json:

```sh
{
"ti":			"ACCT_VALUE",
"key":			"key01",
"val":			"1,602,480",
"curr":			"INR",
"accountName":	"NSM"
}
```

## 2) Order ID for new orders
Prevents orderId conflict if client-app is using same incrementally.
orderID starts from 1000
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |NEXT_VALID_ID| |
|orderId|		int |orderId       | for all orders placed |

Sample Json:

```sh
{
"ti":			"NEXT_VALID_ID",
"orderId":		1005
}
```

## 3) Portifolio or Holding details for "Portfolio Tab"
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |PORTFOLIO_VALUE||
|conId| 		int |conId| Contract Id, important foreign key |
|exchange| 	string |exchange| |
|expiry| 		string |expiry		| contract expiry |
|strike| 		double |strike 		| options strike price |
|currency| 	string |currency| |
|secType| 		string |secType		| security type |
|localSymbol| 	string |localSymbol| |
|symbol| 		string |symbol		|Symbol long format |
|right| 		string |right 		| P / C, PE / CE, options |
|position| 	int |position| |
|marketPrice| 	double |marketPrice| |
|marketValue| 	double |marketValue| |
|averageCost| 	double |averageCost| |
|unrealizedPNL|double |unrealizedPNL | |
|realizedPNL| 	double |realizedPNL| |
|accountName| 	string |accountName	| ACCT_VALUE.accountName |

Sample Json:

```sh
{
"ti":			"PORTFOLIO_VALUE",
"conId": 		9001,
"exchange": 	"NSE",
"expiry": 		"",
"strike": 		0,
"currency": 	"INR",
"secType": 		"STK",
"localSymbol": 	"ZZZ1",
"symbol": 		"NSE-ZZZ1-EQ",
"right": 		""
"position": 	1000,
"marketPrice": 	25,
"marketValue": 	25000,
"averageCost": 	20010,
"unrealizedPNL":1000, 
"realizedPNL": 	3000,
"accountName": 	"NSM"
}
```

## 5) Contract (Symbol) Information
Details about each Symbol
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |CONTRACT_DATA| |
|conId| 		int |conId				|Important ContractId, primary key |
|localSymbol| 	string |localSymbol	| displayed name |
|symbol| 		string |symbol		| long format |
|secType| 		string |secType		| security type |
|expiry| 		string |expiry		| contract expiry |
|strike| 		double |strike 		| Strike price |
|right| 		string |right 		| P / C, PE / CE, options |
|exchange| 	string |exchange| |
|currency| 	string |currency| |
|marketName| 	string |marketName| |
|tradingClass| string |tradingClass| |
|minTick| 		double |minTick			| minimum tick size |
|multiplier| 	string |multiplier		| ? |
|orderTypes| 	string |orderTypes| to-do |
|validExchanges| string |validExchanges| |
|priceMagnifier| int |priceMagnifier| |

Sample Json:

```sh
{
"ti":			"CONTRACT_DATA",
"symbol": 		"ZZZ1",
"secType": 		"STK",
"expiry": 		"",
"strike": 		0,
"right": 		"r",
"exchange": 	"NSE",
"currency": 	"INR",
"localSymbol": 	"ZZZ1",
"marketName": 	"EQ",
"tradingClass": "CLASS",
"conId": 		9001,
"minTick": 		0.05,
"multiplier": 	"5",
"orderTypes": 	"LIMIT",
"validExchanges": "",
"priceMagnifier": 1
}
```

## 6) Place new or Modify Order
when a new order is placed, or existing is modified
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |OPEN_ORDER||
|orderId|		int |orderId| unique order id to modify, 0 for new order |
|conId|		int |conId			| Contract ID, foreign key |
|secType|		string |secType		| Security Type, eg. STK FUT ... |
|localSymbol| 	string |localSymbol | Displayed Symbol name |
|expiry| 		string |expiry		| Contract Expiry |
|currency|		string |currency| |
|symbol| 		string |symbol| |
|exchange|		string |exchange| |
|action| 		string |action		| BUY / SELL |	
|totalQuantity|int |totalQuantity
|orderType| 	string |orderType	| LMT, MKT, STP, STP, LMT, TRAIL, ...|
|lmtPrice| 	double |lmtPrice	| Limit price |
|auxPrice| 	double |auxPrice	| SL-order price |
|tif| 			string |tif			| time_in_force: DAY, GTC, GTD, ... |
|goodAfterTime|string |goodAfterTime	| YYYYMMDD hh:mm:ss TZ |
|goodTillDate| string |goodTillDate	| YYYYMMDD hh:mm:ss TZ |
|parentId| 	int |parentId			| For Exit legs of Bracket order etc. |
|ocaGroup|		string |ocaGroup| |		
|ocaType| 		int |ocaType | |
|account| 		string |account			| ACCT_VALUE.KEY |
|whatIf| 		bool | whatIf | |
|status| 		string |status		| Filled, Submitted, PreSubmitted, Cancelled, PartiallyFilled, Error, ... |
|maintMargin| 	string |maintMargin | minimum amount of equity that must be maintained |
|warningText| 	string |warningText	| details if order held |

Sample Json:

```sh
{
"ti":			"OPEN_ORDER",
"orderId":		1000,
"conId":		9001,
"secType":		"STK",
"localSymbol": 	"ZZZ1",
"expiry": 		"",
"currency":		"INR",
"symbol": 		"NSE-ZZZ1",
"exchange":		"NSE",
"action": 		"BUY",
"totalQuantity":50,
"orderType": 	"LIMIT",
"lmtPrice": 	22,
"auxPrice": 	0,
"tif": 			"DAY",
"goodAfterTime":"DAY",
"goodTillDate": "DAY",
"parentId": 	0,
"ocaGroup":		"Grp1",
"ocaType": 		1,
"account": 		"key01",
"whatIf": 		false,
"status": 		"partial",
"maintMargin": 	"10",
"warningText": 	"testwarn"
}
```

## 7) Order status after order is opened
Status of Order after it is placed by OPEN_ORDER as the state changes
|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |ORDER_STATUS||
|orderId| 		int |orderId| unique order id |
|status|		string |status | Filled, Submitted, PreSubmitted, Cancelled, PartiallyFilled, Error, ... |
|filled|		int |filled				| qty that is filled |
|remaining|	int |remaining			| qty yet to be filled |
|avgFillPrice|	double |avgFillPrice| |
|permId|		int |permId				| unique broker order id |
|parentId|		int |parentId			| 0(zero) if not Bracket Order, else use entry orderID for SL/Profit legs |
|lastFillPrice|double |lastFillPrice| |
|clientId|		int |clientId| |
|whyHeld|		string |whyHeld			| warn msg if order not processed |

Sample Json:

```sh
{
"ti":			"ORDER_STATUS",
"orderId": 		1000,
"status":		"partial",
"filled":		10,
"remaining":	50,
"avgFillPrice":	22,
"permId":		101000,
"parentId":		0,
"lastFillPrice":22.1,
"clientId":		12345,
"whyHeld":		"whyheld"
}
```

## 8) Error messages
**If id == orderID, then it updates specific orderID.** don't use standard errorCode for this

**standard codes**

errorCode 504, 502: 	if client disconnected

errorCode 505: 		fatal error

errorCode 800:		clear Account / Orders list. Request fresh

reserved: 321, 721, 724


|key  |type |value|description|
| ------ | ------ |------ |------ |
|ti|			string |ERR_MSG| |
|id|			int |id			| read comments above |
|errorCode|	int |errorCode| |
|errorMsg|		string |errorMsg| |

Sample Json:

```sh
{
"ti":			"ERR_MSG",
"id":			1000,
"errorCode":	20,
"errorMsg":		"err_20"
}
```
