---
template: BlogPost
path: /spot
mockup: 
thumbnail:
date: 2021-09-21
name: Building a Simple Bond Trading System in C++
title: In this trading system implementation - we develop a core bond trading system
category: Quantitative Finance
description: 'Building bond trading system from scratch.'
tags:
  - Bond 
  - Trading
  - Fixed Income
  - Quantitative Finance
---

## Bond Trading System

We will develop a bond trading system for US Treasuries with six securities: 2Y, 3Y, 5Y, 7Y, 10Y, and 30Y. Look up the CUSIPS, coupons, and maturity dates for each security. Ticker is T.

We have a new definition of a Service in soa.hpp, with the concept of a ServiceListener and Connector also defined. A ServiceListener is a listener to events on the service where data is added to the service, updated on the service, or removed from the service. A Connector is a class that flows data into the Service from some connectivity source (e.g. a socket, file, etc) via the Service.OnMessage() method. The Publish() method on the Connector publishes data to the connectivity source and can be invoked from a Service. Some Connectors are publish-only that do not invoke Service.OnMessage(). Some Connectors are subscribe-only where Publish() does nothing. Other Connectors can do both publish and subscribe.

Using the base classes in tradingsystem.zip attached to this thread we define the following bond specific classes:

BondTradeBookingService BondPositionService

BondRiskService
 BondPricingService
 BondMarketDataService
 BondExecutionService
 BondStreamingService
 BondAlgoExecutionService (no base class – we create this from scratch) BondAlgoStreamingService (no base class – we create this from scratch) GUIService (no base class –we create this from scratch) BondInquiryService

BondHistoricalDataService

The following services below should read data from a file – we create sample data as outlined below. We then read data from the file via a Connector subclass. The Connector should flow data from the file into the Service via the Service.OnMessage() method. Note that we use fractional notation for US Treasuries prices when reading from the file and writing back to a file in the BondHistoricalDataService (with the smallest tick being 1/256th). Example of fractional is 100-xyz, with xy being from 0 to 31 and z being from 0 to 7 (we can replace z=4 with +). The xy number gives the decimal out of 32 and the z number gives the remainder out of 256. So 100-001 is 100.00390625 in decimal and 100-25+ is 100.796875 in decimal.

Note that output files should have timestamps on each line with millisecond precision.

**BondPricingService**

This should read data from prices.txt. Create 1,000,000 prices for each security (so a total of 6,000,000 prices across all 6 securites). The file should create prices which oscillate between 99 and 101 (bearing in mind that US Treasuries trade in 1/256th increments). The bid/offer spread should oscillate between 1/128 and 1/64.

**BondTradeBookingService**

This should read data from trades.txt. Create 10 trades for each security (so a total of 60 trades across all 6 securities) in the file with the relevant trade attributes. Positions should be across books TRSY1, TRSY2, and TRSY3. The BondTradeBookingService should be linked to a BondPositionService via a ServiceListener and send all trades there via the AddTrade() method (note that the BondTradeBookingService should not have an explicit reference to the BondPositionService though or vice versa – link them through a ServiceListener). Trades for each security should alternate between BUY and SELL and cycle from 1000000, 2000000, 3000000, 4000000, and 5000000 for quantity, and then repeat back from 1000000. The price should oscillate between 99.0 (BUY) and 100.0 (SELL).

**BondPositionService**

The BondPositionService does not need a Connector since data should flow via ServiceListener from the BondTradeBookingService. The BondPositionService should be linked to a BondRiskService via a ServiceListenher and send all positions to the BondRiskService via the AddPosition() method (note that

the BondPositionService should not have an explicit reference to the BondRiskService though or versa – link them through a ServiceListener).

**BondRiskService**

The BondRiskService does not need a Connector since data should flow via ServiceListener from the BondPositionService.

**BondMarketDataService**

This should read data from marketdata.txt. Create 1,000,000 order book updates for each security (so a total of 6,000,000 prices) each with 5 orders deep on both bid and offer stacks. The top level should have a size of 10 million, second level 20 million, 30 million for the third, 40 million for the fourth, and 50 million for the fifth. The file should create mid prices which oscillate between 99 and 101 (bearing in mind that US Treasuries trade in 1/256th increments) with a bid/offer spread starting at 1/128th on the top of book (and widening in the smallest increment from there for subsequent levels in the order book). The top of book spread itself should widen out on successive updates by 1/128th until it reaches 1/32nd, and then decrease back down to 1/128th in 1/128th intervals (i.e. spread of 1/128th at top of book, then 1/64th, then 3/128th, then 1/32nd, and then back down again to 1/128th, and repeat).

**BondAlgoExecutionService**

This should be keyed on product identifier with value an AlgoExecution object. The AlgoExecution should be a class with a reference to an ExecutionOrder object. BondAlgoExecutionService
 should register a ServiceListener on the BondMarketDataService and aggress the top of the book, alternating between bid and offer (taking the opposite side of the order book to cross the spread) and only aggressing when the spread is at its tightest (i.e. 1/128th) to reduce the cost of crossing the spread. It
 should send this order to the BondExecutionService via a ServiceListener and the ExecuteOrder() method. Execute on the entire size on the market data for the right side we are executing against (remember, we are *crossing* the spread).

**BondAlgoStreamingService**

This should be keyed on product identifier with value an AlgoStream object. The AlgoStream should be a class with a reference to a PriceStream object. BondAlgoStreamingService
 should register a ServiceListener on the BondPricingService and send the bid/offer prices to the BondStreamingService via a ServiceListener and the PublishPrice() method. Alternate visible sizes between 1000000 and 2000000 on subsequent updates for both sides. Hidden size should be twice the visible size at all times.

**GUIService**

This should be keyed on product identifier with value a Price object (from PricingService). The GUIService is a GUI component that listens to streaming prices that should be throttled. Define the GUIService with a 300 millisecond throttle. We only need to print the first 100 updates. It should register a ServiceListener on the BondPricingService with the specified throttle, which should notify back to the GUIService at that throttle interval. The GUIService should output those updates with a timestamp with millisecond precision to a file gui.txt.

**BondExecutionService**

The BondExecutionService does not need a Connector since data should flow via ServiceListener from the BondAlgoExecutionService. Each execution should result in a trade into the BondTradeBookingService via ServiceListener on BondExectionService – cycle through the books above in order TRSY1, TRSY2, TRSY3.

**BondStreamingService**

The BondStreamingService does not need a Connector since data should flow via ServiceListener from the BondAlgoStreamingService.

**BondInquiryService**

We read inquiries from a file called inquiries.txt with attributes for each inquiry (with state of RECEIVED). We create 10 inquiries for each security (so 60 in total across all 6 securities). Then we register a ServiceListener on the BondInquiryService which sends back a quote when the inquiry is in the RECEIVED state. The BondInquiryService should send a quote of 100 back to a Connector via the Publish() method. The Connector should transition the inquiry to the QUOTED state and send it back to the BondInquiryService via the OnMessage method with the supplied price. It should then immediately send an update of the Inquiry object with a DONE state. Then it moves on to the next inquiry from the file and we repeat the process.

**BondHistoricalDataService**

This service should register a ServiceListener on the following: BondPositionService, BondRiskService, BondExecutionService, BondStreamingService, and BondInquiryService. It should persist objects it receives from these services back into files positions.txt, risk.txt, executions.txt, streaming.txt, and allinquiries.txt via special Connectors for each type with a Publish() method on each Connector. There should be a BondHistoricalDataService corresponding to each data type. When persisting positions, we should persist each position for a given book as well as the aggregate position. When persisting risk, we should persist risk for each security as well as for the following bucket sectors: FrontEnd (2Y, 3Y), Belly (5Y, 7Y, 10Y), and LongEnd (30Y). Use realistic PV01 values for each security.

Please look at the tree structure - this is how all files will be placed inside the project folder

##File Structure
.
├── AlgoExecution.hpp
├── AlgoStream.hpp
├── BondAlgoExecutionService.hpp
├── BondAlgoExecutionServiceListener.hpp
├── BondAlgoStreamingService.hpp
├── BondAlgoStreamingServiceListener.hpp
├── BondExecutionService.hpp
├── BondExecutionServiceConnector.hpp
├── BondExecutionServiceListener.hpp
├── BondHistoricalDataService.hpp
├── BondInformation.h
├── BondInquiryService.hpp
├── BondInquiryServiceConnector.hpp
├── BondMarketDataService.hpp
├── BondMarketDataServiceConnector.hpp
├── BondPositionService.hpp
├── BondPositionServiceListener.hpp
├── BondPricingService.hpp
├── BondPricingServiceConnector.hpp
├── BondProductService.hpp
├── BondRiskService.hpp
├── BondRiskServiceListener.hpp
├── BondStreamingService.hpp
├── BondStreamingServiceConnector.hpp
├── BondStreamingServiceListener.hpp
├── BondTradeBookingService.hpp
├── BondTradeBookingServiceListener.hpp
├── BondTradingBookingServiceConnector.hpp
├── Debug
│   ├── allinquiries.txt
│   ├── executions.txt
│   ├── gui.txt
│   ├── inquiries.txt
│   ├── main.cpp.o
│   ├── main.cpp.o.d
│   ├── marketdata.txt
│   ├── positions.txt
│   ├── prices.txt
│   ├── risk.txt
│   ├── streaming.txt
│   ├── trades.txt
│   └── tradingsystem
├── GUIService.hpp
├── GUIServiceConnector.hpp
├── GUIServiceListener.hpp
├── HistoricalAllInquiriesConnector.hpp
├── HistoricalAllInquriesListener.hpp
├── HistoricalExecutionConnector.hpp
├── HistoricalExecutionListener.hpp
├── HistoricalPositionConnector.hpp
├── HistoricalPositionListener.hpp
├── HistoricalRiskConnector.hpp
├── HistoricalRiskListener.hpp
├── HistoricalSteamingListener.hpp
├── HistoricalStreamingConnector.hpp
├── InquriesGenerator.hpp
├── Makefile
├── MarketdataGenerator.hpp
├── PricesGenerator.hpp
├── README.md
├── TradesGenerator.hpp
├── TradingSystem.workspace
├── executions.txt
├── executionservice.hpp
├── gui.txt
├── historicaldataservice.hpp
├── inquiries.txt
├── inquiryservice.hpp
├── main.cpp
├── marketdata.txt
├── marketdataservice.hpp
├── positions.txt
├── positionservice.hpp
├── prices.txt
├── pricingservice.hpp
├── products.hpp
├── risk.txt
├── riskservice.hpp
├── soa.hpp
├── streaming.txt
├── streamingservice.hpp
├── tradebookingservice.hpp
├── trades.txt
├── tradingsystem.mk
├── tradingsystem.project
└── tradingsystem.txt

```c++
#AlgoExecution.hpp
/**
 * AlgoExecution.hpp
 * defines AlgoExecution, to be key of BondAlgoExecutionService
 */ 
#pragma once

#ifndef ALGOEXECUTION_HPP
#define ALGOEXECUTION_HPP

using namespace std;

#include "BondMarketDataService.hpp"
#include "executionservice.hpp"
#include <string>
#include <random>

class AlgoExecution {
private:
	static int order_ID;
	ExecutionOrder<Bond> executionOrder;
	
	// append c in front of v to make the length equal to width
	string FillCast(string v, int width, char c);
public:
	// ctor
	AlgoExecution(ExecutionOrder<Bond> _executionOrder);

	// run algo to update priceStream according to bond order book
	void RunAlgo(OrderBook<Bond> orderBook);

	// get the executionOrder
	ExecutionOrder<Bond> GetExecutionOrder() const;
};

int AlgoExecution::order_ID=0;

string AlgoExecution::FillCast(string v, int width, char c){
	string result;
	std::stringstream inter;
	inter << std::setw(width) << std::setfill(c) << v;
	inter >> result;
	return result;
}

AlgoExecution::AlgoExecution(ExecutionOrder<Bond> _executionOrder)
	:executionOrder(_executionOrder) {}

void AlgoExecution::RunAlgo(OrderBook<Bond> orderBook){
	auto bond=orderBook.GetProduct();
	// not this executionOrder to update
	if(bond.GetProductId() != executionOrder.GetProduct().GetProductId()) return;

    auto orderID = FillCast(to_string(order_ID),8,'0');
	
	OrderType orderType;
	int rd=rand()%5;
	switch(rd){
		case 0:orderType=FOK;break;
		case 1:orderType=IOC;break;
		case 2:orderType=MARKET;break;
		case 3:orderType=LIMIT;break;
		case 4:orderType=STOP;break;
	}
	PricingSide pricingSide=(rand()%2==0?BID:OFFER);
	
	
	auto bidOrder=orderBook.GetBidStack().begin();
	auto askOrder=orderBook.GetOfferStack().begin();
	
	double price;
	long visiableQ=0,hiddenQ;
	if(pricingSide==BID){
		price=bidOrder->GetPrice();
		if(askOrder->GetPrice()-bidOrder->GetPrice()<2.5/256.0)
			visiableQ=bidOrder->GetQuantity();
		hiddenQ=2*visiableQ;
	}
	else{
		price=askOrder->GetPrice();
		if(askOrder->GetPrice()-bidOrder->GetPrice()<2.5/256.0)
			visiableQ=askOrder->GetQuantity();
		hiddenQ=2*visiableQ;
	}
    
    string parentID = "P"+orderID ;

    executionOrder = ExecutionOrder<Bond>(bond,pricingSide,orderID,orderType,price,visiableQ,hiddenQ,parentID,true);
	
	++order_ID;
	return;
}

ExecutionOrder<Bond> AlgoExecution::GetExecutionOrder() const{
	return executionOrder;
}
#endif
```

```c++
#AlgoStream.hpp
/**
 * AlgoStream.hpp
 * Defines AlgoStream class, to be used as key of BondAlgoStreamingService
 * 
 */
#pragma once

#ifndef ALGOSTREAM_HPP
#define ALGOSTREAM_HPP

#include "streamingservice.hpp"
#include "pricingservice.hpp"
#include "products.hpp"
#include <iostream>

using namespace std;

class AlgoStream{
private:
	PriceStream<Bond> priceStream;
public:
	// ctor
	AlgoStream(const PriceStream<Bond>& _priceStream);
	
	// run algo to update priceStream according to bond price
	void RunAlgo(Price<Bond> price);
	
	// return *priceStream
	PriceStream<Bond> GetPriceStream() const;
};


AlgoStream::AlgoStream(const PriceStream<Bond>& _priceStream):priceStream(_priceStream){
}

void AlgoStream::RunAlgo( Price<Bond> price){
	auto bond = price.GetProduct();
	// not this PriceStream to update
	if(bond.GetProductId() != priceStream.GetProduct().GetProductId()) return;
    
	float mid=price.GetMid();
	float spread=price.GetBidOfferSpread();
	float bid=mid-spread/2.0;
	float offer=mid+spread/2.0;
	
	// if spread is tight
	if(spread<2.5/256.0){
		PriceStreamOrder order_bid(bid, 1000000, 2000000, BID);
		PriceStreamOrder order_ask(offer, 1000000, 2000000, OFFER);
		priceStream = PriceStream<Bond>(bond, order_bid, order_ask);
	}
	// if spread is not tight 
    else{
		PriceStreamOrder order_bid(bid, 0, 0, BID);
		PriceStreamOrder order_ask(offer, 0, 0, OFFER);
		priceStream = PriceStream<Bond>(bond, order_bid, order_ask);
	}
	
	return;
}


PriceStream<Bond> AlgoStream::GetPriceStream() const{
	return priceStream;
}
#endif 
```

```c++
#BondAlgoExecutionService.hpp
/*
 * BondAlgoExecutionService.hpp
 * defines BondAlgoExecutionService to received datafrom BondMarketDataService
 * and pass to BondExecutionService
 */ 
#pragma once

#ifndef BONDALGOEXECUTIONSERVICE_HPP
#define BONDALGOEXECUTIONSERVICE_HPP

using namespace std;

#include "AlgoExecution.hpp"

class BondAlgoExecutionService: public Service<string, AlgoExecution>{
private:
	map<string,AlgoExecution> algoMap;
	vector<ServiceListener<AlgoExecution >*> listenerList;      
	
public:
    // ctor
    BondAlgoExecutionService();

    // override pure virtual functions in base class 
	// get algoExecution by id 
    AlgoExecution & GetData(string id);       
	
	// called by conector, in this case, no connector
    void OnMessage(AlgoExecution &);                
	
	// add a listener. in this case, BondExecutionServiceListener
    void AddListener(ServiceListener<AlgoExecution> *listener);  
	
	// get listeners. in this case, BondExecutionServiceListener
    const vector<ServiceListener<AlgoExecution> *>& GetListeners() const;  
	
	// add a order book and call RunAlgo in corresponding AlgoExecution
	// called by BondAlgoExecutionServiceListener
    void AddOrderBook(OrderBook<Bond>& orderBook);        
};

BondAlgoExecutionService::BondAlgoExecutionService(){
	algoMap=map<string, AlgoExecution>();
}

AlgoExecution& BondAlgoExecutionService::GetData(string id){
	return algoMap.at(id);
}

void BondAlgoExecutionService::OnMessage(AlgoExecution & a){
	return;
}

void BondAlgoExecutionService::AddListener(ServiceListener<AlgoExecution> *listener){
	listenerList.push_back(listener);
	return;
}

const vector<ServiceListener<AlgoExecution> *>& BondAlgoExecutionService::GetListeners() const{
	return listenerList;
}

void BondAlgoExecutionService::AddOrderBook(OrderBook<Bond>& orderBook){
	auto id=orderBook.GetProduct().GetProductId();
	auto it=algoMap.find(id);
	// if already exist, update price by RunAlgo
	if(it!=algoMap.end()){
		(it->second).RunAlgo(orderBook);
	}
	
	// add an empty AlgoStream as value. now as id already in map,
	// call AddPrice again to update price by RunAlgo
	else{
		auto eo=ExecutionOrder<Bond>(orderBook.GetProduct(),BID,"0",FOK,0,0,0,"0",true);	
		algoMap.insert(pair<string,AlgoExecution >(id,AlgoExecution(eo)));
		AddOrderBook(orderBook);
		return;
	}
	
	// notify the listeners
	for(auto& listener:listenerList){
		listener->ProcessAdd(it->second);
	}

	return;
}
#endif 
```

```c++
#BondAlgoExecutionServiceListener.hpp
/**
 * BondAlgoExectuionServiceListener.hpp
 * define listener to pass data from BondMarketDataService
 * to BondAlgoExecutionService
 */ 
#pragma once 

#ifndef BONDALGOEXECUTIONSERVICELISTENER_HPP
#define BONDALGOEXECUTIONSERVICELISTENER_HPP

using namespace std;

#include "BondAlgoExecutionService.hpp"
#include "soa.hpp"

class BondAlgoExecutionServiceListener: public ServiceListener<OrderBook<Bond> >{
private:
    BondAlgoExecutionService* bondAlgoExecutionService;
public:
    //ctor
    BondAlgoExecutionServiceListener(BondAlgoExecutionService* _bondAlgoExecutionService);

    // override the pure virtual functions in base class
	// called by BondMarketDataService and call AddOrderBook in BondAlgoExecutionService
    virtual void ProcessAdd(OrderBook<Bond> &data) override; 
	
	// no use in this case
    virtual void ProcessRemove(OrderBook<Bond> &data) override{};  
	
	// no use in this case
    virtual void ProcessUpdate(OrderBook<Bond> &data) override{};  
};

BondAlgoExecutionServiceListener::BondAlgoExecutionServiceListener(BondAlgoExecutionService* _bondAlgoExecutionService){
	bondAlgoExecutionService=_bondAlgoExecutionService;
}

void BondAlgoExecutionServiceListener::ProcessAdd(OrderBook<Bond> &data){
	bondAlgoExecutionService -> AddOrderBook(data);
	return;
}

#endif
```

```c++
#BondAlgoStreamingService.hpp
/**
 * BondAlgoStreamingService.hpp
 * Defines BondAlgoStreamingService class. Receive data from BondPricingService
 * and send data tp BondStreamingService
 * 
 */ 
#pragma once

#ifndef BONDALGOSTREAMINGSERVICE_HPP
#define BONDALGOSTREAMINGSERVICE_HPP

#include "AlgoStream.hpp"
#include "products.hpp"
#include <map>
#include <vector>
#include <string>
#include "soa.hpp"

using namespace std;

class BondAlgoStreamingService: public Service<string, AlgoStream>{
private:
	map<string, AlgoStream > algoMap; 
	vector<ServiceListener<AlgoStream>* > listenerList;  
public:
	// ctor
    BondAlgoStreamingService();

    // override pure virual functions in Service
	// get AlgoStream given id 
    virtual AlgoStream& GetData(string id) override;              
	
	// in this case, no connector call OnMessage method
    virtual void OnMessage(AlgoStream &) override;                   
	
	// add a listener. In this case, listener is BondStreamingServiceListener
    virtual void AddListener(ServiceListener<AlgoStream> *listener) override;  
	
	// get listeners. In this case, listener is BondStreamingServiceListener
    virtual const vector<ServiceListener<AlgoStream> *>& GetListeners() const override;  
	
	// add price to update/initialize a new AlgoStream in algomap
	// called by BondAlgoStreamingServiceListener to pass a new price from BondPricingService
    void AddPrice(const Price<Bond>& price);       
		
};

BondAlgoStreamingService::BondAlgoStreamingService(){
	algoMap=map<string, AlgoStream>();
}

AlgoStream& BondAlgoStreamingService::GetData(string id){
	return algoMap.at(id);
}

void BondAlgoStreamingService::OnMessage(AlgoStream & algo){
}

void BondAlgoStreamingService::AddListener(ServiceListener<AlgoStream> *listener){
	listenerList.push_back(listener);
}

const vector<ServiceListener<AlgoStream> *>& BondAlgoStreamingService::GetListeners() const{
	return listenerList;
}

void BondAlgoStreamingService::AddPrice(const Price<Bond>& price){
	auto id=price.GetProduct().GetProductId();
	auto it=algoMap.find(id);
	// if already exist, update price by RunAlgo
	if(it!=algoMap.end()){
		(it->second).RunAlgo(price);
	}
	// add an empty AlgoStream as value. now as id already in map,
	// call AddPrice again to update price by RunAlgo
	else{
		PriceStreamOrder p1(0, 0, 0, BID);
		PriceStreamOrder p2(0, 0, 0, OFFER);
		PriceStream<Bond> ps(price.GetProduct(), p1, p2);
		algoMap.insert(pair<string,PriceStream<Bond> >(id,ps));
		AddPrice(price);
		return;
	}
	
	// notify the listeners
	for(auto& listener:listenerList){
		listener->ProcessAdd(it->second);
	}

	return;
}
#endif
```

```c++
#BondAlgoStreamingServiceListener.hpp
/*
 * BondAlgoStreamingServiceListener.hpp
 * defines BondAlgoStreamingServiceListener to pass data from
 * BondPricingService to BondAlgoStreamingService
 */
#pragma once

#ifndef BONDALGOSTREAMINGSERVICELISTENER_HPP
#define BONDALGOSTREAMINGSERVICELISTENER_HPP

#include "soa.hpp"
#include "BondAlgoStreamingService.hpp"

using namespace std;

class BondAlgoStreamingServiceListener: public ServiceListener<Price<Bond> >{
private:
	BondAlgoStreamingService* bondAlgoStreamingService;
public:
	// ctor
	BondAlgoStreamingServiceListener(BondAlgoStreamingService* _bondAlgoStreamingService);
	
	// override the pure virtual functions in ServiceListener
	// when BondPricingService pass a new price, call AddPrice method in
	// BondAlgoStreamingService
	virtual void ProcessAdd(Price<Bond>& price) override; 
	
	// in this case, do not need this method
	virtual void ProcessRemove(Price<Bond>&) override;  
	
	// in this case, do not need this method
	virtual void ProcessUpdate(Price<Bond>&) override;  
};

BondAlgoStreamingServiceListener::BondAlgoStreamingServiceListener(BondAlgoStreamingService* _bondAlgoStreamingService){
	bondAlgoStreamingService=_bondAlgoStreamingService;
}

void BondAlgoStreamingServiceListener::ProcessAdd(Price<Bond>& price){
	bondAlgoStreamingService->AddPrice(price);
	return;
}

void BondAlgoStreamingServiceListener::ProcessRemove(Price<Bond>& data){
	return;
}

void BondAlgoStreamingServiceListener::ProcessUpdate(Price<Bond>& data){
	return;
}

#endif
```

```c++
#BondExecutionService.hpp
/**
 * BondExecutionService.hpp
 * defines BondExecutionService to receive data from BondAlgoExecutionService
 * and pass to connector to publish
 */ 

#pragma once

#ifndef BONDEXECUTIONSERVICE_HPP
#define BONDEXECUTIONSERVICE_HPP

using namespace std;

#include "BondAlgoExecutionService.hpp"
#include "executionservice.hpp"

class BondExecutionServiceConnector;

class BondExecutionService: public ExecutionService<Bond> {
private:
	map<string, ExecutionOrder<Bond> > executionMap;   
	vector<ServiceListener<ExecutionOrder<Bond> >*> listenerList;      
    BondExecutionServiceConnector* bondExecutionServiceConnector;  
public:
    // ctor
    BondExecutionService(BondExecutionServiceConnector* _bondExecutionServiceConnector);

    // override pure virtual functions in base class
	// get execution order by id
    virtual ExecutionOrder<Bond> & GetData(string id) override;     
	
	// no connector in this case
    virtual void OnMessage(ExecutionOrder<Bond> &) override;      
	
	// add a listener, in this case, should be BondHistoricalDataServiceListener
    virtual void AddListener(ServiceListener<ExecutionOrder<Bond> > *listener) override;  
	
	// get listeners, in this case, should be BondHistoricalDataServiceListener
    virtual const vector<ServiceListener<ExecutionOrder<Bond> > *>& GetListeners() const override; 
	
	// pass execution order to connector
    virtual void ExecuteOrder(ExecutionOrder<Bond> &order, Market market) override;   
	
	// add algo to AlgoMap and notify listeners
	// also be called by BondExecutionServiceListener
    void AddAlgoExecution(const AlgoExecution& algo);         
};

BondExecutionService::BondExecutionService(BondExecutionServiceConnector* _bondExecutionServiceConnector){
	bondExecutionServiceConnector=_bondExecutionServiceConnector;
}

ExecutionOrder<Bond> & BondExecutionService::GetData(string id){
	return executionMap.at(id);
} 
	
void BondExecutionService::OnMessage(ExecutionOrder<Bond> &){
	return;
}     

void BondExecutionService::AddListener(ServiceListener<ExecutionOrder<Bond> > *listener){
	listenerList.push_back(listener);
	return;
} 
	
const vector<ServiceListener<ExecutionOrder<Bond> > *>& BondExecutionService::GetListeners() const{
	return listenerList;
}

	
// define void BondExecutionService::ExecuteOrder in BondExecutionServiceConnector.hpp

	
void BondExecutionService::AddAlgoExecution(const AlgoExecution& algo){
	auto executionOrder = algo.GetExecutionOrder();
    string id = executionOrder.GetProduct().GetProductId();
	
	if(executionMap.find(id)!=executionMap.end())
		executionMap.erase(id);
	executionMap.insert(pair<string,ExecutionOrder<Bond> >(id,executionOrder));

    for(auto& listener :listenerList) {
        listener->ProcessAdd(executionOrder );
    }
	
	return;
}
#endif
```

```c++
#BondExecutionServiceConnector.hpp
/**
 * BondExecutionServiceConnector.hpp
 * receive data from BondExecutionService and print it out
 */ 
#pragma once

#ifndef	BONDEXECUTIONSERVICECONNECTOR_HPP
#define BONDEXECUTIONSERVICECONNECTOR_HPP

using namespace std;

#include "BondExecutionService.hpp"
#include "soa.hpp"
#include "products.hpp"
#include <string>
#include <iostream>

class BondExecutionServiceConnector:public Connector<ExecutionOrder<Bond> >{
public:
    // ctor
    BondExecutionServiceConnector();

    // override pure virtual functions in base class
	// print ExecutionOrder. called by BondExecutionService
    virtual void Publish(ExecutionOrder<Bond>& data) override;   
	
	// publish only, no subsribe
    virtual void Subscribe() override;   
};

BondExecutionServiceConnector::BondExecutionServiceConnector(){}

void BondExecutionServiceConnector::Publish(ExecutionOrder<Bond>& data){
	auto bond=data.GetProduct();
	string oderType;
	switch(data.GetOrderType()) {
		case FOK: oderType = "FOK"; break;
		case MARKET: oderType = "MARKET"; break;
		case LIMIT: oderType = "LIMIT"; break;
		case STOP: oderType = "STOP"; break;
		case IOC: oderType = "IOC"; break; 
	}
	cout<<bond.GetProductId()<<" OrderId: "<<data.GetOrderId()<< "\n"
		<<"    PricingSide: "<<(data.GetSide()==BID? "Bid":"Ask")
		<<" OrderType: "<<oderType<<" IsChildOrder: "<<(data.IsChildOrder()?"True":"False")
		<<"\n"
		<<"    Price: "<<data.GetPrice()<<" VisibleQuantity: "<<data.GetVisibleQuantity()
		<<" HiddenQuantity: "<<data.GetHiddenQuantity()<<"\n"<<endl;
		
	return;
}

void BondExecutionServiceConnector::Subscribe(){
	return;
}

void BondExecutionService::ExecuteOrder(ExecutionOrder<Bond> &order, Market market){
	bondExecutionServiceConnector->Publish(order);
} 
#endif
```

```c++
#BondExecutionServiceListener.hpp
/**
 * BondExecutionServiceListener.hpp
 * defines listener to pass data from BondAlgoExecutionService
 * to BondExecutionService
 */ 
#pragma once 

#ifndef BONDEXECUTIONSERVICELISTENER_HPP
#define BONDEXECUTIONSERVICELISTENER_HPP

using namespace std;

#include "BondAlgoExecutionService.hpp"
#include "soa.hpp"
 
class BondExecutionServiceListener: public ServiceListener<AlgoExecution>{
private:
    BondExecutionService* bondExecutionService;

public:
    // ctor
    BondExecutionServiceListener(BondExecutionService* _bondExecutionService);

    // override pure virtual functions in base class
	// called by BondAlgoExecutionService and call BondExecutionService::AddAlgoExecution
	// and BondExecutionService::ExecuteOrder
    virtual void ProcessAdd(AlgoExecution &data) override; 
	
	// no use in this case
    virtual void ProcessRemove(AlgoExecution &data) override{};
	
	// no use in this case
    virtual void ProcessUpdate(AlgoExecution &data) override{}; 
	
};

BondExecutionServiceListener::BondExecutionServiceListener(BondExecutionService* _bondExecutionService){
	bondExecutionService=_bondExecutionService;
}

void BondExecutionServiceListener::ProcessAdd(AlgoExecution &data){
	auto order=data.GetExecutionOrder();
	bondExecutionService->AddAlgoExecution(data);
	bondExecutionService->ExecuteOrder(order,BROKERTEC);
	return;
}


#endif
```

```c++
#BondHistoricalDataService.hpp
#pragma once

#ifndef BONDHISTORICALDATASERVICE_HPP
#define BONDHISTORICALDATASERVICE_HPP

#include <iostream>
#include "historicaldataservice.hpp"
#include "GUIServiceConnector.hpp"
#include "BondStreamingService.hpp"
#include "BondRiskService.hpp"
#include "BondPositionService.hpp"
#include "BondExecutionService.hpp"
#include "BondInquiryService.hpp"

using namespace std;

class HistoricalPositionConnector;
class HistoricalRiskConnector;
class HistoricalExecutionConnector;
class HistoricalStreamingConnector;
class HistoricalAllInquiriesConnector;


class BondHistoricalPositionService: public HistoricalDataService<Position<Bond> > {
private:
	HistoricalPositionConnector* connector;
	map<string, Position<Bond> > dataMap;
public:
	// ctor
	BondHistoricalPositionService(HistoricalPositionConnector* _p):connector(_p){}

	// override pure virtual functions in base class
	// get data given id
	virtual Position<Bond>  & GetData(string id) override {return dataMap.at(id);}

	// no use
	virtual void OnMessage(Position<Bond>  & bond) override {}

	// no use
	virtual void AddListener(ServiceListener<Position<Bond>  > *listener) override {}

	// no use
	virtual const vector<ServiceListener<Position<Bond>  > *>& GetListeners() const override {}

	// called by listeners, call correspoding connector to publish
	virtual void PersistData(string persistKey, Position<Bond>& data) override;
};


class BondHistoricalRiskService: public HistoricalDataService<PV01<Bond> > {
private:
	HistoricalRiskConnector* connector;
	map<string, PV01<Bond> > dataMap;
public:
	// ctor
	BondHistoricalRiskService(HistoricalRiskConnector* _p):connector(_p){}

	// override pure virtual functions in base class
	// get data given id
	virtual PV01<Bond>  & GetData(string id) override {return dataMap.at(id);}

	// no use
	virtual void OnMessage(PV01<Bond>  & bond) override {}

	// no use
	virtual void AddListener(ServiceListener<PV01<Bond>  > *listener) override {}

	// no use
	virtual const vector<ServiceListener<PV01<Bond>  > *>& GetListeners() const override {}

	// called by listeners, call correspoding connector to publish
	virtual void PersistData(string persistKey, PV01<Bond>& data) override;
};


class BondHistoricalExecutionService: public HistoricalDataService<ExecutionOrder<Bond> > {
private:
	HistoricalExecutionConnector* connector;
	map<string, ExecutionOrder<Bond> > dataMap;
public:
	// ctor
	BondHistoricalExecutionService(HistoricalExecutionConnector* _p):connector(_p){}

	// override pure virtual functions in base class
	// get data given id
	virtual ExecutionOrder<Bond> & GetData(string id) override {return dataMap.at(id);}

	// no use
	virtual void OnMessage(ExecutionOrder<Bond> & bond) override {}

	// no use
	virtual void AddListener(ServiceListener<ExecutionOrder<Bond>  > *listener) override {}

	// no use
	virtual const vector<ServiceListener<ExecutionOrder<Bond>  > *>& GetListeners() const override {}

	// called by listeners, call correspoding connector to publish
	virtual void PersistData(string persistKey, ExecutionOrder<Bond>& data) override;
};


class BondHistoricalStreamingService: public HistoricalDataService<PriceStream<Bond> > {
private:
	HistoricalStreamingConnector* connector;
	map<string, PriceStream<Bond> > dataMap;
public:
	// ctor
	BondHistoricalStreamingService(HistoricalStreamingConnector* _p):connector(_p){}

	// override pure virtual functions in base class
	// get data given id
	virtual PriceStream<Bond> & GetData(string id) override {return dataMap.at(id);}

	// no use
	virtual void OnMessage(PriceStream<Bond> & bond) override {}

	// no use
	virtual void AddListener(ServiceListener<PriceStream<Bond>  > *listener) override {}

	// no use
	virtual const vector<ServiceListener<PriceStream<Bond>  > *>& GetListeners() const override {}

	// called by listeners, call correspoding connector to publish
	virtual void PersistData(string persistKey, PriceStream<Bond>& data) override;
};


class BondHistoricalAllInquiriesService: public HistoricalDataService<Inquiry<Bond> > {
private:
	HistoricalAllInquiriesConnector* connector;
	map<string, Inquiry<Bond> > dataMap;
public:
	// ctor
	BondHistoricalAllInquiriesService(HistoricalAllInquiriesConnector* _p):connector(_p){}

	// override pure virtual functions in base class
	// get data given id
	virtual Inquiry<Bond> & GetData(string id) override {return dataMap.at(id);}

	// no use
	virtual void OnMessage(Inquiry<Bond> & bond) override {}

	// no use
	virtual void AddListener(ServiceListener<Inquiry<Bond>  > *listener) override {}

	// no use
	virtual const vector<ServiceListener<Inquiry<Bond> > *>& GetListeners() const override {}

	// called by listeners, call correspoding connector to publish
	virtual void PersistData(string persistKey, Inquiry<Bond>& data) override;
};


#endif
```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

```c++

```

