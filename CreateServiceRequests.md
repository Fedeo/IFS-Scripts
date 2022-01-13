```cs

Eval DateString(Now()) Into Today

//Check if CustomerId already exists
Query CustomerHandling.svc/CustomerInfoSet Select "CustomerId" Filter With Json Into CustomerIdExists 
{
   "CustomerId": "{$input.CustomerId}"
}  

//Create customer json
Create CustomerHandling.svc/CustomerInfoSet When CustomerIdExists.value.ItemsCount() == 0
{
	"B2bCustomer":false,
	"Country":"{$input.Country}",
	"CreationDate":"{$Today}",
	"CustomerCategory":"Customer",
	"DefaultLanguage": "en",
	"DefaultDomain":true,
	"Name":"{$input.Name}",
	"PartyType":"Person",
	"CustomerId":"{$input.CustomerId}"
}



//Create Customer Address

Eval "LOC-" + "{$input.CustomerId}" + "-ADD1" into AddressId
Eval "{$input.Address1}" + " " + "{$input.Address2}" into FullAddress

//Check if $AddressId already exists
Query CustomerHandling.svc/CustomerInfoSet(CustomerId='{$input.CustomerId}')/CustomerInfoAddresses Select "AddressId" Filter With Json Into AddressIdExists 
{
   "AddressId":"{$AddressId}"
}  

Create CustomerHandling.svc/CustomerInfoSet(CustomerId='{$input.CustomerId}')/CustomerInfoAddresses When AddressIdExists.value.ItemsCount() == 0
{
	"AddressId":"{$AddressId}",
	"Country":"{$input.Country}",
	"CustomerId":"{$input.CustomerId}",
	"DefaultDomain":true,
	"PartyType":"Person",
	"Name":"{$input.Name}",
	"Address":"{$FullAddress}",
	"Address1":"{$input.Address1}",
	"Address2":"{$input.Address2}",
	"ZipCode":"{$input.ZipCode}",
	"City":"{$input.City}"

}

//Create Base Location
Eval "LOC-" + "{$input.CustomerId}"  into LocationId
Eval "Location for customer: " + "{$input.Name}" into LocationName

//Check if LocationId already exists
Query LocationHandling.svc/LocationSet  Select "LocationId" Filter With Json Into LocationIdExists 
{
   "LocationId": "{$LocationId}"
}  

// If Location dooesn't exist create it
Create LocationHandling.svc/LocationSet  When LocationIdExists.value.ItemsCount() == 0
{
    "LocationId": "{$LocationId}",
    "Name": "{$LocationName}",
    "SchedulingExist": true,
    "AvailabilityExist": true
}

//Add Customer Address to Location

//Check if $AddressId already exists
Query LocationHandling.svc/LocationSet(LocationId='{$LocationId}')/LocationAddressArray   Select "AddressId" Filter With Json Into AddressIdExists 
{
   "AddressId":"{$AddressId}"
}  

Create LocationHandling.svc/LocationSet(LocationId='{$LocationId}')/LocationAddressArray  When AddressIdExists.value.ItemsCount() == 0
{
    "DeliveryAddress": false,
    "LocationId": "{$LocationId}",
    "PartyType": "Customer",
    "PositionAddress": true,
    "PrimaryAddress": true,
    "VisitAddress": true,
    "Identity": "{$input.CustomerId}",
    "AddressId": "{$AddressId}",
    "Description": "LOCATION ADDRESS"
}


//Add Map
Eval "LOCATION_ID=" + "{$LocationId}" +  "^" into keyRef

//Check if keyRef already exists
Query MapPositionsHandling.svc/MapPositionSet   Select "KeyRef" Filter With Json Into KeyRefExists 
{
   "KeyRef":"{$keyRef}"
}  


Create MapPositionsHandling.svc/MapPositionSet When KeyRefExists.value.ItemsCount() == 0
{
    "DefaultPosition": true,
    "Longitude": "{$input.long}",
    "Latitude": "{$input.lat}",
    "KeyRef":"{$keyRef}",
    "LuName":"Location"
}


Eval  {%input.ReportedObjectId} into myReportedObjectId 
Eval  {%input.ReportedObjectSite} into myReportedObjectSite
Eval  {%input.ContractId} into myContractId 
Eval  {%input.LineNo} into myLineNo 

Eval "" into myReportedObjectId when myReportedObjectId == null
Eval "" into myReportedObjectSite when myReportedObjectSite == null
Eval "" into myContractId when myContractId == null
Eval "0" into myLineNo when myLineNo == null

Create RequestHandling.svc/SrvRequestVirtualSet into ServiceRequestResponse
{
    "CustomerId": "{$input.CustomerId}",
    "Description": "{$input.requestDescr}",
    "ItemId": "{$input.requestType}",
    "OrganizationId": "{$input.OrganizationId}",
    "Revision": "1",
    "IsApplicableForAll": true,
    "IsServiceGroup": false,
    "IsAppointmentBookingAllowedFinal": true,
    "LocationId": "{$LocationId}",
    "AddressId": "{$AddressId}",
    "ReferenceId":"{$input.ReferenceId}",
    "ReportedObjectId":"{$myReportedObjectId}",
    "ReportedObjectSite":"{$myReportedObjectSite}",
    "ContractId":"{$myContractId}",
    "LineNo":"{$myLineNo}"
}


Eval ServiceRequestResponse.Objkey into ObjkeyParam

Create RequestHandling.svc/SrvRequestVirtualSet(Objkey='{$ObjkeyParam}')/IfsApp.RequestHandling.SrvRequestVirtual_Finish into RequestResponse
{
}

Eval RequestResponse.NewReqId into RequestId

Get RequestHandling.svc/SrvRequestSet(ReqId='{$RequestId}') into RequestDetails

Post RequestHandling.svc/SrvRequestSet(ReqId='{$RequestId}')/IfsApp.RequestHandling.SrvRequest_Release using RequestDetails into Releaseresponse
{
}

```