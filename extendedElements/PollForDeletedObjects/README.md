In Cloud Elements one of the many strengths is the ability to [Extend Elements](https://docs.cloud-elements.com/home/extend-elements) with new resources to make the catalog element work to meet your use cases. The ability to extend an element goes beyond using that resource for direct calls, and these new resources can be used for other purposes such as to enhance your experience with polling if a given vendor supports associated endpoints for deleted or related records. In this example we will be looking at the extendable element [Zoho CRM V2](https://docs.cloud-elements.com/home/zoho-crm) and how it can be extended to utilize additional vendor functionality so that you are not limited to only Created and Updated events.

Polling Zoho CRM V2 for Created and Updated events is as simple as setting your element instance to poll the desired object (e.g. /accounts or /leads), but if you need to also be notified of Deleted events through a Zoho element instance then different endpoints must be leveraged.

In the Zoho CRM V2 APIs there is no parameter that can be used to return deleted records using the GET /{module} endpoints and instead deleted records are accessible through a separate endpoint GET /{module}/deleted. What this means is that to retrieve a list of deleted records a new resource must be added to the Zoho CRM V2 element for /{objectName}/deleted and then the poller must be set to execute this API for the desired object/module. Let’s go through an example for polling deleted accounts:

1. First, create a new resource in your Zoho element that can be used to retrieve deleted records from any of the supported Zoho modules. Execute this POST with the provided resource definition:

```
curl -X POST \
  https://staging.cloud-elements.com/elements/api-v2/elements/zohocrmv2/resources \
  -H 'Authorization: User XXXXX, Organization XXXXX' \
  -H 'Content-Type: application/json' \
  -H 'accept: application/json'
```

2. Confirm that you have a new resource /{objectName}/deleted in your Zoho CRM V2 element instances, and that you can make GET calls against this resource such as GET /accounts/deleted. Also note the configuration of this new resource in the Resources tab. https://cl.ly/ec8c190a4466

3. Add this new resource to your Poller Configuration so that you start polling for deleted ‘account’ events in addition to your existing poller configurations: (https://cl.ly/bdc6453ef62a)

```json
{
  "deleted": {
    "url": "/accounts/deleted",
    "idField": "id",
    "datesConfiguration": {
      "updatedDateField": "deleted_time",
      "updatedDateFormat": "yyyy-MM-dd'T'HH:mm:ssXXX",
      "createdDateField": "deleted_time",
      "createdDateFormat": "yyyy-MM-dd'T'HH:mm:ssXXX",
      "updatedDateTimezone": "GMT",
      "createdDateTimezone": "GMT"
    },
    "pageSize": 200
  }
}
```


### Notes:
* Because the Zoho CRM V2 /{module}/deleted endpoints only support "If-Modified-Since" in the request header but no other date query/filtering it makes more sense for us to leverage the polling framework to filter new events instead of modifying the request header in this use case. By setting the datesConfiguration in your poller appropriately for the desired resource the Cloud Elements polling framework will filter the vendor responses and only send event notifications for records processed since the last poll interval.

* The same approach can be used for other resources/modules that Zoho supports /deleted for including: leads, accounts, contacts, deals, campaigns, tasks, cases, events, calls, solutions, products, vendors, pricebooks, quotes, salesorders, purchaseorders, invoices, custom, and notes.

* The only caveat with this approach is that your polling events will appear as Created events, but so long as you process events based on both the objectType (e.g. ‘deleted’) and the eventType then this solution can be used to poll Zoho CRM V2 for deleted records.
