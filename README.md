# Power Email 

## db design

![](email_schema_diagram.png?raw=true)

Just realized really want a CreatedOn and ModifiedOn time for the sent\_email and received\_email. Really good for bookkeeping and updating records after pulled back in from ML pipeline. Perhaps even CreatedBy and ModifiedBy also useful for auditing.

### Basic idea

#### Start with relational database

Taking postgres as example, to speed up text searching within the email bodies. An index like postgres GIN with a tsvector that contains a word list should speed up searching a great deal. If pattern matching would have to change to something like GIST. The search would need to include a tsquery to make use of the index.

#### Move to distributed microservice solution 

Elasticsearch which is built mainly around text searching and a great addition for performant queries. This scales really well horizontally and would work with something the scale of GMAIL. A traditional relational database will start to struggle with millions of concurrent requests. But can be mitigated to an extent with replicas, caching as well as sharding. Designing for the biggest scale initially is putting the cart before the horse though.

## Pipeline to machine learning models

### Step 1: picking solution

* custom code
  * scripts & scheduler
  * api
  * rpc
* etl service
  * airflow
  * glue
  * etc

The benefit of a solution is that engineers don't have to learn many different code bases and ways of moving data around.

### Step 2: prepare and move data

This will require knowing what solution is chosen. If there is a message queue as intake then clean up the data with whatever transformations need to take place and move into queue. If it's into a data lake then move there.

### Step 3: prepare intake for ML results

Depends on solution and very similar to step 2. 

## API

Route format: http.method URL

### send\_email

#### route

POST /email

#### required data

```json
{
    "from": "user@email.com",
    "to": "anotheruser@anotheremail.com",
    "header": "email headers",
    "body": "this is my message"
}
```

The from field will likely need to be validated by authorization.

#### returns

Success, Unauthorized, Invalid (bad request)

### get\_emails

#### routes

GET /emails

GET /email

GET /email/{id}

### returns

```json
{
    "from": "user@email.com",
    "to": "anotheruser@anotheremail.com",
    "header": "email headers",
    "body": "this is my message"
}
```

/email routes return list

### search\_emails

#### route

GET /emails?search={search}&to={to}&from={from}

search required

to, from optional

### returns

```json
[
{
    "from": "user@email.com",
    "to": "anotheruser@anotheremail.com",
    "header": "email headers",
    "body": "this is my message"
},
...
]
```

### get\_email\_stats

#### route

GET /emails/stats

GET /email/stats

### returns

```json
{
    "emails_sent": 100,
    "emails_received": 2000,
    "spam_blocked": 500
}
```
