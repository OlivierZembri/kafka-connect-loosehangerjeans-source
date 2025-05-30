# kafka-connect-loosehangerjeans-source

Kafka Connect source connector used for generating simulated events for demos and tests.

It produces messages simulating the following events:

| **Topic name**    | **Description**                                       |
|-------------------|-------------------------------------------------------|
| `DOOR.BADGEIN`    | An employee using their id badge to go through a door |
| `CANCELLATIONS`   | An order being cancelled                              |
| `CUSTOMERS.NEW`   | A new customer has registered on the website          |
| `ORDERS.NEW`      | An order has been placed                              |
| `SENSOR.READINGS` | A sensor reading captured from an IoT sensor          |
| `STOCK.MOVEMENT`  | Stock shipment received by a warehouse                |
| `ORDERS.ONLINE`   | An online order has been placed                       |
| `STOCK.NOSTOCK`   | A product has run out-of-stock                        |
| `PRODUCT.RETURNS` | A return request has been issued                      |
| `PRODUCT.REVIEWS` | A product review has been posted                      |
| `TRANSACTIONS`    | A transaction has been posted                         |


Avro schemas and sample messages for each of these topics can be found in the `./doc` folder.


## Config

### Minimal

By default, minimal config is needed. Specifying that keys are strings and payloads are json is enough to start it running.

```yaml
apiVersion: eventstreams.ibm.com/v1beta2
kind: KafkaConnector
metadata:
  name: kafka-datagen
  labels:
    eventstreams.ibm.com/cluster: kafka-connect-cluster
spec:
  class: com.ibm.eventautomation.demos.loosehangerjeans.DatagenSourceConnector
  tasksMax: 1
  config:
    key.converter: org.apache.kafka.connect.storage.StringConverter
    key.converter.schemas.enable: false
    value.converter: org.apache.kafka.connect.json.JsonConverter
    value.converter.schemas.enable: false
```

### Custom

Config overrides are available to allow demos based on different domains or industries.

Example config is listed below with all possible options, each shown with their default values.

```yaml
apiVersion: eventstreams.ibm.com/v1beta2
kind: KafkaConnector
metadata:
  name: kafka-datagen
  labels:
    eventstreams.ibm.com/cluster: kafka-connect-cluster
spec:
  class: com.ibm.eventautomation.demos.loosehangerjeans.DatagenSourceConnector
  tasksMax: 1
  config:
    #
    # format of messages to produce
    #
    key.converter: org.apache.kafka.connect.storage.StringConverter
    key.converter.schemas.enable: false
    value.converter: org.apache.kafka.connect.json.JsonConverter
    value.converter.schemas.enable: false

    #
    # name of the topics to produce to
    #
    topic.name.orders: ORDERS.NEW
    topic.name.cancellations: CANCELLATIONS
    topic.name.stockmovements: STOCK.MOVEMENT
    topic.name.badgeins: DOOR.BADGEIN
    topic.name.newcustomers: CUSTOMERS.NEW
    topic.name.sensorreadings: SENSOR.READINGS
    topic.name.onlineorders: ORDERS.ONLINE
    topic.name.outofstocks: STOCK.NOSTOCK
    topic.name.returnrequests: PRODUCT.RETURNS
    topic.name.productreviews: PRODUCT.REVIEWS
    topic.name.transactions: TRANSACTIONS

    #
    # startup behavior
    #
    # if true, the connector will generate one week of historical
    #  events when starting for the first time
    startup.history.enabled: false

    #
    # format of timestamps to produce
    #
    #    default is chosen to be suitable for use with Event Processing,
    #    but you could modify this if you want to demo how to reformat
    #    timestamps to be compatible with Event Processing
    #
    #    NOTE: sensor readings topic is an exception. Events on this topic
    #           ignore this config option
    #
    formats.timestamps: yyyy-MM-dd HH:mm:ss.SSS
    # format of timestamps with local time zone (UTC time in ISO 8601 format)
    #    NOTE: this format is used by default for online orders
    formats.timestamps.ltz: yyyy-MM-dd'T'HH:mm:ss.SSS'Z'

    #
    # how often events should be created
    #
    # 'normal' random orders
    timings.ms.orders: 30000              # every 30 seconds
    # cancellations of a large order followed by a small order of the same item
    timings.ms.falsepositives: 600000     # every 10 minutes
    # repeated cancellations of a large order followed by a small order of the same item
    timings.ms.suspiciousorders: 3600000  # every hour
    # stock movement events
    timings.ms.stockmovements: 300000     # every 5 minutes
    # door badge events
    timings.ms.badgeins: 600              # sub-second
    # new customer events
    timings.ms.newcustomers: 543400       # a little over 9 minutes
    # sensor reading events
    timings.ms.sensorreadings: 27000      # every 27 seconds
    # sensor reading events
    #  from a single sensor that periodically generates very high and increasing readings (before returning to a normal range)
    timings.ms.highsensorreadings: 18000  # every 18 seconds
    # online orders
    timings.ms.onlineorders: 30000        # every 30 seconds
    # return requests
    timings.ms.returnrequests: 300000     # every 5 minutes
    # product reviews
    timings.ms.productreviews: 60000      # every 1 minute
    # transactions
    timings.ms.transactions: 20000        # every 20 seconds

    #
    # how much of a delay to introduce when producing events
    #
    #    this is to simulate events from systems that are slow to
    #    produce to Kafka
    #
    #    events with a delay will be produced to Kafka a short
    #    time after the timestamp contained in the message payload
    #
    #    the result is that the timestamp in the message metadata
    #    will be later than the message in the message payload
    #
    #    because the delay will be random (up to the specified max)
    #    the impact of this is that messages on the topic will be
    #    slightly out of sequence (according to the timestamp in
    #    the message payload)
    #
    # orders
    eventdelays.orders.secs.max: 0              # payload time matches event time by default
    # cancellations
    eventdelays.cancellations.secs.max: 0       # payload time matches event time by default
    # stock movements
    eventdelays.stockmovements.secs.max: 0      # payload time matches event time by default
    # door badge events
    eventdelays.badgeins.secs.max: 180          # payload time can be up to 3 minutes (180 secs) behind event time
    # new customer events
    eventdelays.newcustomers.secs.max: 0        # payload time matches event time by default
    # sensor readings events
    eventdelays.sensorreadings.secs.max: 300    # payload time can be up to 5 minutes (300 secs) behind event time
    # online orders
    eventdelays.onlineorders.secs.max: 0        # payload time matches event time by default
    # out-of-stock events
    eventdelays.outofstocks.secs.max: 0         # payload time matches event time by default
    # return requests
    eventdelays.returnrequests.secs.max: 0      # payload time matches event time by default
    # product reviews
    eventdelays.productreviews.secs.max: 0      # payload time matches event time by default
    # transactions
    eventdelays.transactions.secs.max: 0        # payload time matches event time by default

    #
    # how many events should be duplicated
    #
    #   this is to simulate events from systems that offer
    #   at-least once semantics
    #
    #   messages will occasionally be duplicated, according
    #   to the specified ratio
    #   between 0.0 and 1.0 : 0.0 means events will never be duplicated,
    #                         0.5 means approximately half of the events will be duplicated
    #                         1.0 means all events will be duplicated
    #
    # orders
    duplicates.orders.ratio: 0              # events not duplicated
    # cancellations
    duplicates.cancellations.ratio: 0       # events not duplicated
    # stock movements
    duplicates.stockmovements.ratio: 0.1    # duplicate roughly 10% of the events
    # door badge events
    duplicates.badgeins.ratio: 0            # events not duplicated
    # new customer events
    duplicates.newcustomers.ratio: 0        # events not duplicated
    # sensor reading events
    duplicates.sensorreadings.ratio: 0      # events not duplicated
    # online orders
    duplicates.onlineorders.ratio: 0        # events not duplicated
    # out-of-stock events
    duplicates.outofstocks.ratio: 0         # events not duplicated
    # return requests
    duplicates.returnrequests.ratio: 0      # events not duplicated
    # product reviews
    duplicates.productreviews.ratio: 0      # events not duplicated
    # transactions
    duplicates.transactions.ratio: 0        # events not duplicated

    #
    # product names to use in events
    #
    #    these are combined into description strings, to allow for
    #    use of Event Processing string functions like regexp extracts
    #    e.g. "XL Stonewashed Bootcut Jeans"
    #
    #    any or all of these can be modified to theme the demo for a
    #    different industry
    products.sizes: XXS,XS,S,M,L,XL,XXL
    products.materials: Classic,Retro,Navy,Stonewashed,Acid-washed,Blue,Black,White,Khaki,Denim,Jeggings
    products.styles: Skinny,Bootcut,Flare,Ripped,Capri,Jogger,Crochet,High-waist,Low-rise,Straight-leg,Boyfriend,Mom,Wide-leg,Jorts,Cargo,Tall
    products.name: Jeans

    #
    # prices to use for individual products
    #
    #    prices will be randomly generated between the specified range
    prices.min: 14.99
    prices.max: 59.99
    # prices following large order cancellations will be reduced by a random value up to this limit
    prices.maxvariation: 9.99

    #
    # number of items to include in an order
    #
    # "normal" orders will be between small.min and large.max
    #   (i.e. between 1 and 15, inclusive)
    #
    # a "small" order is between 1 and 5 items (inclusive)
    orders.small.quantity.min: 1
    orders.small.quantity.max: 5
    # a "large" order is between 5 and 15 items (inclusive)
    orders.large.quantity.min: 5
    orders.large.quantity.max: 15

    #
    # controlling when orders should be cancelled
    #
    # how many orders on the ORDERS topic should be cancelled (between 0.0 and 1.0)
    cancellations.ratio: 0.005
    # how long after an order should the cancellation happen
    cancellations.delay.ms.min: 300000   # 5 minutes
    cancellations.delay.ms.max: 7200000  # 2 hours
    # reason given for a cancellation
    cancellations.reasons: CHANGEDMIND,BADFIT,SHIPPINGDELAY,DELIVERYERROR,CHEAPERELSEWHERE

    #
    # suspicious orders
    #
    #  these are the events that are looked for in lab 5 and lab 6
    #
    # how quickly will the large order will be cancelled
    suspicious.cancellations.delay.ms.min: 900000    # at least 15 minutes
    suspicious.cancellations.delay.ms.max: 1800000   # within 30 minutes
    # how many large orders will be made and cancelled
    suspicious.cancellations.max: 3   # up to three large orders
    # customer names to be used for suspicious orders will be selected from this
    #  list, to make it easier in lab 5 and 6 to see that you have created the
    #  flow correctly, and to make it easier in lab 4 to see that there are false
    #  positives in the simplified implementation
    suspicious.cancellations.customernames: Suspicious Bob,Naughty Nigel,Criminal Clive,Dastardly Derek

    #
    # new customers
    #
    #  these events are intended to represent new customers that
    #   have registered with the company
    #
    # how many new customers should quickly create their first order
    #  between 0.0 and 1.0 : 0.0 means new customers will still be created, but they will
    #                           never create orders,
    #                         1.0 means all new customers will create an order
    newcustomers.order.ratio: 0.22
    # if a new customer is going to quickly create their first order, how long
    #  should they wait before making their order
    newcustomers.order.delay.ms.min: 180000     # wait at least 3 minutes
    newcustomers.order.delay.ms.max: 1380000    # order within 23 minutes

    #
    # online orders
    #
    #  these events are intended to represent orders for several products,
    #   illustrating the use of complex objects and primitive arrays
    #
    # number of products to include in an online order: between 1 and 5 (inclusive)
    onlineorders.products.min: 1
    onlineorders.products.max: 5
    # number of emails for the customer who makes an online order: between 1 and 2 (inclusive)
    onlineorders.customer.emails.min: 1
    onlineorders.customer.emails.max: 2
    # number of phones in an address for an online order: between 0 and 2 (inclusive)
    #    NOTE: in case of 0 phone number, `null` is generated in the events as value for the `phones` property
    onlineorders.address.phones.min: 0
    onlineorders.address.phones.max: 2
    # how many online orders use the same address as shipping and billing address
    #  between 0.0 and 1.0 : 0.0 means no online order will use the same address as shipping and billing address
    #                        1.0 means all online orders will use the same address as shipping and billing address
    onlineorders.reuse.address.ratio: 0.55
    # how many online orders have at least one product that runs out-of-stock after the order has been placed
    #  between 0.0 and 1.0 : 0.0 means no online order has some product that runs out-of-stock
    #                        1.0 means all online orders have products that run out-of-stock
    onlineorders.outofstock.ratio: 0.22

    #
    # out-of-stocks
    #
    #  these events are intended to represent products that run out-of-stock in online orders
    #
    # how long after an out-of-stock should the restocking happen (in days)
    outofstocks.restocking.delay.days.min: 1  # 1 day
    outofstocks.restocking.delay.days.max: 5  # 5 days
    # how long after an online order should the out-of-stock happen (in milliseconds)
    outofstocks.delay.ms.min: 300000   # 5 minutes
    outofstocks.delay.ms.max: 7200000  # 2 hours

    #
    # return requests
    #
    #  these events are intended to represent return requests for several products,
    #   illustrating the use of complex objects and complex arrays
    #
    # number of products to include in a return request: between 1 and 4 (inclusive)
    returnrequests.products.min: 1
    returnrequests.products.max: 4
    # quantity for each product to include in a return request: between 1 and 3 (inclusive)
    returnrequests.product.quantity.min: 1
    returnrequests.product.quantity.max: 3
    # number of emails for the customer who issued a return request: between 1 and 2 (inclusive)
    returnrequests.customer.emails.min: 1
    returnrequests.customer.emails.max: 2
    # number of phones in an address for a return request: between 0 and 2 (inclusive)
    #    NOTE: in case of 0 phone number, `null` is generated in the events as value for the `phones` property
    returnrequests.address.phones.min: 0
    returnrequests.address.phones.max: 2
    # how many return requests use the same address as shipping and billing address
    #  between 0.0 and 1.0 : 0.0 means no return request will use the same address as shipping and billing address
    #                        1.0 means all return requests will use the same address as shipping and billing address
    returnrequests.reuse.address.ratio: 0.75
    # reason given for a product return
    returnrequests.reasons: CHANGEDMIND,BADFIT,SHIPPINGDELAY,DELIVERYERROR,CHEAPERELSEWHERE,OTHER
    # how many return requests have at least one product that has a review that is posted after the return request has been issued
    #  between 0.0 and 1.0 : 0.0 means no return request has some product that has a review that is posted
    #                        1.0 means all return requests have products that have a review that is posted
    returnrequests.review.ratio: 0.32
    # how many products have a size issue in a return request
    #  between 0.0 and 1.0 : 0.0 means no product has a size issue in a given return request
    #                        1.0 means all products have a size issue in a given return request
    returnrequests.product.with.size.issue.ratio: 0.22

    #
    # product reviews
    #
    #  these events are intended to represent reviews for products returned in return requests.
    #
    # number of products that have a size issue for product reviews
    productreviews.products.with.size.issue.count: 10
    # how many product reviews have a size issue for products that are supposed to have a size issue
    #  between 0.0 and 1.0 : 0.0 means no review with a size issue is generated for products that are supposed to have a size issue
    #                        1.0 means all generated reviews have a size issue for products that are supposed to have a size issue
    productreviews.review.with.size.issue.ratio: 0.75
    # how long after a return request should the product review happen (in milliseconds)
    productreviews.delay.ms.min: 300000   # 5 minutes
    productreviews.delay.ms.max: 3600000  # 1 hour

    #
    # locations that are referred to in generated events
    #
    locations.regions: NA,SA,EMEA,APAC,ANZ
    # countries in each region
    #  NA   : CA (Canada), US (United States), MX (Mexico)
    #  SA   : BR (Brazil), PY (Paraguay), UY (Uruguay)
    #  EMEA : BE (Belgium), FR (France), CH (Switzerland), GB (United Kingdom), DE (Germany), ES (Spain)
    #  APAC : ID (Indonesia), SG (Singapore), BN (Brunei), PH (Philippines)
    #  ANZ  : AU (Australia), NZ (New Zealand)
    locations.regions.countries: NA:CA,US,MX;SA:BR,PY,UY;EMEA:BE,FR,CH,GB,DE,ES;APAC:ID,SG,BN,PH;ANZ:AU,NZ
    locations.warehouses: North,South,West,East,Central

    #
    # transactions
    #
    #  these events are intended to represent transaction requests
    #
    transactions.max.ids: 5
    # minimum amount of a transaction
    transactions.amount.min: 100.0
    # maximum amount of a transaction
    transactions.amount.max: 1000.0
    # ratio of the transactions that should be a complete sequence of
    #  STARTED -> PROCESSING -> PROCESSING -> COMPLETED
    transactions.valid.ratio: 0.8  # 80% of transactions are complete
                                   # 20% of transactions omit an event
```

For example, if you want to theme the demo to be based on products in a different industry, you could adjust product sizes/materials/styles/name to match your demo (the options don't need to actually be "sizes", "materials" or "styles" - they just need to be lists that will make sense when combined into a single string).

You may also want to modify the prices.min and prices.max values to match the sort of products in your demo.


## Build

```sh
mvn package
```

