# Common Issues

## General

### Endpoints Accessibility

Most of platforms are passive, means there's no notification to connector when data changed from their sides. Check CRUD accessibility of platform interface before design integration solution. This will strongly affect how [Sequence Diagram](technical-specification.md) looks like.

### Fields Type/Length Limitation and Validation

Be aware of platform fields limitation, some might limit type as an integer or have limited length. Some fields will be validated before it goes into platform (email, phone number, address), it can lead to syncing failure or incomplete data.

### Database Connection Limitation

The whole integration can end up to have 5 \~ 6 (or more when decentralized requests applied) cron jobs running at the same time. Raise maximum database connections to allow this happened.

### Change of Source of Truth

Make sure data should be always managed from source of truth, this can be indicated in [Sequence Diagram.](technical-specification.md) If the condition is broken, there's risk that data will not be sync or will be overwritten.

### Data transparency

Sometimes different terms are used to refer data in business team and in technical team. The data, fields, naming demonstrate in client portal might be different from platform API. Make sure developer team and client window reach consensus or build a tool to easily interpret from one to the other (e.g. shopify-to-ap21 product ID lookup).

### **Incremental Data Size**

When data size become larger and larger as business grow, processing time of single task would increase as well. Ideally, big dataset should be broken down into small ones, and handling requests should be sent from different threads. Every single dataset thus would be handled independently.

## Platform Limitation

### Shopify

#### API Calls ([Official Documentation](https://help.shopify.com/api/getting-started/api-call-limit))

| Plan                     | Bucket Size | Leak Rate |
| ------------------------ | ----------- | --------- |
| Shopify Standard/Advance | 40          | 2/sec     |
| Shopify Plus             | 80          | 4/sec     |

#### Endpoint

* Search endpoint didn't well function yet , it only functions when login as an admin in browser
* line item id mapping is needed when refund or partial fulfillment function is required

#### Metafields

* Metafields has it's own endpoint and won't be return in parent object's endpoint. It's not included in search index neither.
* Besides native fields, connector needs to ensure required metafields are also brought to the other platform if needed, and it needs additional API calls

#### **Pagination**

Up to 250 resources per page

#### Webhook Response Time

Shopify will resend the payload to webhooks if it didn't get response within 5 sec. Make sure the webhooks respond to Shopify to avoid unnecessary duplicate request.
