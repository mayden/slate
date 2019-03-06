---
title: ZeroSix.ai API

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: Shell
  - php: PHP


toc_footers:
  - <a href='https://www.zerosix.ai'>ZeroSix.ai</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:


search: true
---

# Introduction

Welcome to ZeroSix.ai API. You can use our API to access ZeroSix.ai database, and retrieve information about products, orders and customers.

With our API, you can easily integrate our products, and place an order in your website. Detailed information on each endpoint is listed below.

If you have any questions regarding using this API, or if you encountered with problem - please feel free to contact us [here](https://zerosix.ai/contact/).

## Formats

The default response format is JSON. Requests with a message-body use plain JSON to set or update resource attributes. Successful requests will return a 200 OK HTTP status.
Some general information about responses:

- Dates are returned in ISO8601 format: YYYY-MM-DDTHH:MM:SS or in timestamp format. You can choose.
- Resource IDs are returned as integers.
- Any decimal monetary amount, such as prices or totals, will be returned as strings with two decimal places.
- Other amounts, such as item counts, are returned as integers.
- Blank fields are generally included as null or emtpy string instead of being omitted.

## Errors

Occasionally you might encounter errors when accessing the REST API. There are four possible types:
 
Error Code | Error Type 
--------- | ------- 
`400 Bad Request` | Invalid request, e.g. using an unsupported HTTP method
`401 Unauthorized` | Authentication or permission error, e.g. incorrect API keys
`404 Not Found` | Requests to resources that don't exist or are missing
`500 Internal Server Error` | Server error

Errors return both an appropriate HTTP status code and response object which contains a `code`, `message` and `data` attribute.

## Parameters

Almost all endpoints accept optional parameters which can be passed as a HTTP query string parameter, e.g. `GET /bookings?status=completed`. All parameters are documented along each endpoint.

# Authentication

## REST API Keys
Pre-generated keys can be used to authenticate use of the REST API endpoints. 

ZeroSix uses API keys to allow access to our API. For now, you can't automatically create your own API keys. If you want to start using our API, please contact us.

ZeroSix expects for the API keys to be included in all API requests to the server in a header that looks like the following:

## OAuth

You must use OAuth 1.0a "one-legged" authentication to ensure REST API credentials cannot be intercepted by an attacker. Typically you will use any standard OAuth 1.0a library in the language of your choice to handle the authentication, or generate the necessary parameters by following the following instructions.

> Example of using OAuth library to authenticate:

```php
<?php
// URL of listing the products
$url = "https://www.zerosix.ai/wp-json/api/v1/products";

$config = array(
    'consumer_key' => 'YOUR_CUSTOMER_KEY',
    'consumer_secret' => 'YOUR_CUSTOMER_SECRET',
);

$oauth = new OAuth($config['consumer_key'], $config['consumer_secret'], OAUTH_SIG_METHOD_HMACSHA1, OAUTH_AUTH_TYPE_URI);

$requestInfo = $oauth->getLastResponseInfo();

$oauth->fetch($url);
$data = $oauth->getLastResponse();

var_dump($data);
?>

```
> You should use the OAuth library.

### Creating a signature
### Collect the request method and URL

First you need to determine the HTTP method you will be using for the request, and the URL of the request.

The **HTTP method** will be `GET` in our case.

The **Request URL** will be the endpoint you are posting to, e.g. `https://www.zerosix.ai/wp-json/api/v1/products`

###Collect parameters
Collect and normalize your parameters. This includes all `oauth_*` parameters except for the `oauth_signature` itself.

These values need to be encoded into a single string which will be used later on. The process to build the string is very specific:

1. [Percent encode](https://dev.twitter.com/oauth/overview/percent-encoding-parameters) every key and value that will be signed.
2. Sort the list of parameters alphabetically by encoded key.
3. For each key/value pair:
    - Append the encoded key to the output string.
    - Append the = character to the output string.
    - Append the encoded value to the output string.
    - If there are more key/value pairs remaining, append a `&` character to the output string.

When percent encoding in PHP for example, you would use `rawurlencode()`.

When sorting parameters in PHP for example, you would use `uksort( $params, 'strcmp' ).`

> Paramaters example:

```php
oauth_consumer_key=abc123&oauth_signature_method=HMAC-SHA1
```



### Create the signature base string
The above values collected so far must be joined to make a single string, from which the signature will be generated. This is called the signature base string in the OAuth specification.

To encode the HTTP method, request URL, and parameter string into a single string:

1. Set the output string equal to the uppercase **HTTP Method**.
2. Append the & character to the output string.
3. [Percent encode](https://dev.twitter.com/oauth/overview/percent-encoding-parameters) the URL and append it to the output string.
4. Append the & character to the output string.
5. [Percent encode](https://dev.twitter.com/oauth/overview/percent-encoding-parameters) the parameter string and append it to the output string.




> Example signature base string:

```php
GET&https%3A%2F%2Fwww.zerosix.ai%2Fwp-json%2Fapi%2Fv1%2Fproducts&oauth_consumer_key%3Dabc123%26oauth_signature_method%3DHMAC-SHA1
```

###Generate the signature
Generate the signature using the *signature base* string and your consumer secret key with a `&` character with the HMAC-SHA1 hashing algorithm.

In PHP you can use the [hash_hmac](http://php.net/manual/en/function.hash-hmac.php) function.

HMAC-SHA1 or HMAC-SHA256 are the only accepted hash algorithms.


### OAuth Tips
- The OAuth parameters may be added as query string parameters or included in the Authorization header.
- Note there is no reliable cross-platform way to get the raw request headers in our system, so query string should be more reliable in some cases.
- The required parameters are: `oauth_consumer_key`, `oauth_timestamp`, `oauth_nonce`, `oauth_signature`, and `oauth_signature_method`. `oauth_version` is not required and should be omitted.
- The OAuth timestamp should be the unix timestamp at the time of the request. The REST API will deny any requests that include a timestamp outside of a 15 minute window to prevent replay attacks.
- Twitter has great instructions on [generating signatures](https://dev.twitter.com/docs/auth/creating-signature) with OAuth 1.0a, but remember tokens are not used with this implementation.
- Note that the request body is *not* signed as per the OAuth spec, see [Google's OAuth 1.0 extension](https://oauth.googlecode.com/svn/spec/ext/body_hash/1.0/oauth-bodyhash.html) for details on why.
- If including parameters in your request, it saves a lot of trouble if you can order your items alphabetically.

# Products

The products API allows you to view individual or a batch of products.

## Product properties

### Product
Attribute | Type | Description
--------- | ------- | -----------
`id` | `integer` | Unique identifier for the product.
`title` | `string` | Product name
`slug` | `string` | Product slug
`description` | `string` | Product description
`permalink` | `string` | Product URL
`hourly-price` | `integer` | Product price
`currency` | `string` | Default is: `USD`.
`attributes` | `array` | Listed below.


### Product Attributes

The options for the attribute is listed from the lower efficiency to higher.

Attribute | Description
--------- | -----------
`cpu` | Options are: `intel-celeron`, `amd-fx-8320`, `intel-i3`, `amd-ryzen-5`, `intel-i5`, `intel-i7`, `xeon-e5-2667`
`geo` | Options are: `asia-pacific`, `australia`, `europe`, `united-states` or `global` (Worldwide)
`gpu` | Options are: `nvidia-1060`, `nvidia-1060ti`, `nvidia-1070`, `nvidia-1070ti`, `nvidia-1080`, `nvidia-1080ti`, `nvidia-titan-xp`, `nvidia-tesla-v`.
`network-speed` | Options are: `50mbps`, `50-100mbps`, `100-200mbps`, `500-600mbps`
`ram` | Options are: `8gb`, `12gb`, `16gb`, `32gb`, `192gb`


## List all Products

```php
  <?php
  
    $GET_URL = "https://www.zerosix.ai/wp-json/api/v1/products";
    $data = array(
              
      );
  
    $options = array(
      'http' => array(
        'header'  => "Content-type: application/json\r\n",
        'method'  => 'GET',
        'content' => json_encode($data)
             )
        );
    $context  = stream_context_create($options);
    
    $result = file_get_contents($GET_URL, false, $context);

  ?>
```


```shell
  curl  https://www.zerosix.ai/wp-json/api/v1/products  \
        -u consumer_key:consumer_secret
```



> The above command returns JSON structured like this:

```json
[
 {
        "id": 1,
        "title": "NVIDIA 1060, 100 GB SSD",
        "slug": "nvidia-1060-100-gb-ssd",
        "description": "NVIDIA 1060 with 100 GB SSD ",
        "permalink": "product/nvidia-1060-100-gb-ssd/",
        "hourly-price": "0.25",
        "currency": "USD",
        "attributes": {
            "cpu": "intel-i3",
            "geo": "asia-pacific",
            "network-speed": "50-100mbps",
            "ram": "8gb",
            "gpu": "nvidia-1060"
        }
    },
    {
        "id": 2,
        "title": "NVIDIA 1080ti, 100GB SSD",
        "slug": "nvidia-1080ti-100gb-ssd",
        "description": "NVIDIA 1080ti with 250 GB SSD",
        "permalink": "product/nvidia-1080ti-100gb-ssd/",
        "hourly-price": "0.5",
        "currency": "USD",
        "attributes": {
            "cpu": "intel-i3",
            "ram": "8gb",
            "gpu": "nvidia-1080ti",
            "geo": "europe",
            "network-speed": "50mbps"
        }
    }   
]
```

This API helps you to view all the products.

### HTTP Request

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/products</h6>
    </div>
</div>


### Query Parameters

*none.*


## Filter Products by attributes

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/products/</h6>
    </div>
</div>

```json
/* For example: we want to search a machine
  GREATER than nvidia-1070
  with at least 16GB RAM
  that located in Europe 
  and the hourly-price is between 0.3 and 0.8
*/
{
	"relation": "greater",
	"attributes": 
	{
		"gpu": "nvidia-1070",
		"geo": "europe",
		"ram": "16gb"
	},
	"min_price": 0.3,
	"max_price": 0.8
}
```

> The response of this JSON will be:

```json
[
    {
        "id": 1,
        "title": "NVIDIA TitanXP",
        "slug": "nvidia-titanxp",
        "description": "NVIDIA TitanXp with 250 GB SSD ",
        "permalink": "/product/nvidia-titanxp/",
        "hourly-price": "0.7",
        "currency": "USD",
        "attributes": {
            "cpu": "intel-i7",
            "ram": "16gb",
            "gpu": "nvidia-titan-xp",
            "geo": "europe",
            "network-speed": "50mbps"
        }
    }
]

```

### HTTP Requests

Parameter | Description
--------- | -----------
`relation` | Relation of the attributes. Must be `greater`, `equal` or `less`. 
`attributes` | JSON Object of the required attributes. See example.
`min_price` | Minimum hourly-price per product
`max_price` | Maximum hourly-price per product

### HTTP Responses

Parameter | Status | Description
--------- | ----------- | -----------
`products_invalid_relation` | `400` | Invalid relation type. Please use: greater, equal or less.
`products_invalid_prices` | `400` | Invalid min or max price type. Please valid your input to numbers only
`products_invalid_prices` | `400` | min price must be less than max price.
`products_invalid_attribute` | `400` | Please input valid attributes according to the attributes table.


## Retrieve a Product

```shell
  curl  https://www.zerosix.ai/wp-json/api/v1/products/1  \
        -u consumer_key:consumer_secret \
        -H "Content-Type: application/json" 
```

> The above command returns JSON structured like this:

```json
{
 
        "id": 1,
        "title": "NVIDIA 1060, 100 GB SSD",
        "slug": "nvidia-1060-100-gb-ssd",
        "description": "NVIDIA 1060 with 100 GB SSD ",
        "permalink": "product/nvidia-1060-100-gb-ssd/",
        "hourly-price": "0.25",
        "currency": "USD",
        "attributes": {
            "cpu": "intel-i3",
            "geo": "asia-pacific",
            "network-speed": "50-100mbps",
            "ram": "8gb",
            "gpu": "nvidia-1060"
        }
    
}
```

This API lets you retrieve and view a specific product by ID.


### HTTP Request


<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/products/(ID)</h6>
    </div>
</div>

### Query Parameters

Parameter | Description
--------- | -----------
`ID` | The ID of the product to retrieve


# Orders
This API endpoints allows you to create new order with multiple machines , retrieve information about specific order and more.

## Order properties

Attribute | Type | Description
--------- | ------- | -----------
`order_id` | `integer` | Unique identifier for the order.
`order_key` | `string` | Order key
`order_status` | `string` | Order status. Options: `unlimited`, `pending`, `processing`, `on-hold`, `completed`, `cancelled`, `refunded`, `failed` and `trash`. Default is `pending`.
`date_created` | `DateTime` | The date the order was created, in the site's timezone. Properties are: `date`, `timezone_type` and `timezone`.
`hourly-price` | `integer` | Hourly price for this order
`currency` | `string` | The currency. Default is: `USD`.
`product` | `array` | Product properties: <br /> `id` - The ID of the product. <br /> `name` - Name of the ordered product.
`img_file` | `string` | Img file URL to be loaded initially on the rented machines.
`Bookings` | `array` | Booking properties: <br /> `id`, `status`, `start_date`, `end_date`, `machine` and `rented_hours`.

## Create new order


```php
  <?php
  
    $ORDER_URL = "https://www.zerosix.ai/wp-json/api/v1/orders";
    $data = array(
      'product_id' => 2,
      'instances' => 3,
      'img_file' => "https://ww.zerosix.ai/blabla.tar.gz"
      );
  
    $options = array(
      'http' => array(
        'header'  => "Content-type: application/json\r\n",
        'method'  => 'PUT',
        'content' => json_encode($data)
             )
        );
    $context  = stream_context_create($options);
    
    $result = file_get_contents($ORDER_URL, false, $context);

  ?>
```


```shell
  curl  https://www.zerosix.ai/wp-json/api/v1/products  \
        -u consumer_key:consumer_secret
```



> The above command returns JSON structured like this:

```json
[
 {
        "order_id": 1,
        "order_key": "wc_order_randomchars",
        "order_status": "unlimited",
        "date_created": {
            "date": "2019-01-01 00:00:00.000000",
            "timezone_type": 3,
            "timezone": "America/Los_Angeles"
        },
        "hourly-price": "0.5",
        "currency": "USD",
        "product": {
            "id": 2,
            "name": "NVIDIA 1080ti, 100GB SSD"
        },
        "img_file": "https://ww.zerosix.ai/blabla.tar.gz",
        "bookings": [
            {
                "id": 10,
                "status": "wc-unlimited",
                "start_date": "January 1, 2019, 00:00 am",
                "end_date": "",
                "machine": "machine details",
                "rented_hours": "1"
            },
            {
                "id": 11,
                "status": "wc-unlimited",
                "start_date": "January 1, 2019, 00:00 am",
                "end_date": "",
                "machine": "machine details",
                "rented_hours": "1"
            },
            {
                "id": 12,
                "status": "wc-unlimited",
                "start_date": "January 1, 2019, 00:00 am",
                "end_date": "",
                "machine": "machine details",
                "rented_hours": "1"
            },
        ]
    }
]
```

You can place an order with multiple instances. 

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-put">PUT</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/orders</h6>
    </div>
</div>

### HTTP Request

Parameter | Description
--------- | -----------
`product_id` | The product ID we want to book
`instances` | Number of instances to start with. 
`img_file` | Optional. URL of the img file to be loaded on the machines. 


### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`order_invalid_product_id` | `400` | Invalid ProductID. 
`order_invalid_instances` | `400` | Invalid number of instances
`order_not_available_resources` | `200` | Number of requested instances unavailable at this moment.


## Complete an order

Once you completed your order, all the machines (bookings) will be marked as `completed`, and the total cost of your rental time will be updated accordingly. 

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-del">DEL</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/orders/(ID)</h6>
    </div>
</div>


### HTTP Requests

Parameter | Description
--------- | -----------
`order_id` | `integer` | Unique identifier for the order.


### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`order_invalid_id` | `400` | Invalid Order ID. 
`order_invalid_permissions` | `401` | You are not authorized to modify this order.
`order_invalid_status` | `404` | Order has been marked already as completed. You cant delete this order. 

## List all Orders

List all the orders made by you. 

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/orders/</h6>
    </div>
</div>


### HTTP Requests

Parameter | Description
--------- | -----------


### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------


## Retrieve an Order


<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/orders/(ID)</h6>
    </div>
</div>


### HTTP Requests

Parameter | Description
--------- | -----------
`id` | The ID of the order we want to retrieve. 

### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`order_invalid_id` | `400` | Invalid Order ID. 
`order_invalid_permissions` | `401` | You are not authorized to modify this order.


#Bookings

This API endpoints allows you to create new booking, retrieve information about specific booking, see available booking dates and more. 

## Booking properties

Attribute | Type | Description
--------- | ------- | -----------
`id` | `integer` | Unique identifier for the booking.
`order_id` | `integer` | Unique identifier for the order.
`order_key` | `string` | Order key
`booking_status` | `string` | Booking status. Options: `unpaid`, `pending-confirmation`, `confirmed`, `paid`, `cancelled`, `complete`.
`order_status` | `string` | Order status. Options: `pending`, `processing`, `on-hold`, `completed`, `cancelled`, `refunded`, `failed` and `trash`. Default is `pending`.
`date_created` | `DateTime` | The date the order was created, in the site's timezone. Properties are: `date`, `timezone_type` and `timezone`.
`date_modified` | `DateTime` | The date the order was modified, in the site's timezone. Properties are: `date`, `timezone_type` and `timezone`.
`start_time` | `date` | The start time of the booking. Options: `date` and `timestamp`.
`end_time` | `date` | The end time of the booking. Options: `date` and `timestamp`.
`discount_total` | `integer` | Total discount amount for the order.
`discount_tax` | `integer` | Total discount tax amount for the order.
`total` | `integer` | Grand total
`total_tax` | `integer` | Sum of all taxes
`currency` | `string` | Currency the order was created with, in ISO format. default is `USD`.
`customer_id` | `integer` | Customer ID who created the order. 
`client_info` | `array` | Options: `billing` and `shipping` address **OR** If the order is made trough your `customer_id`, so the `client_info` contains all your sub-client information.
`transaction_id` | `string` |  Unique transaction ID.
`date_paid` | `DateTime` |  The date the order was paid, in the site's timezone.
`date_completed` | `DateTime` |  The date the order was completed, in the site's timezone.
`payment_url` | `string` | Payment URL
`items_booked` | `array` | Booked items. Properties are: `product_id`, `product_name`, `quanity` and `tax_status`.
`coupon_lines` | `array` | Coupon lines. Properties are: `id`, `code`, `discount`, `discount_tax` and `meta_data`



### Client Info

`billing` and `shipping` properties:

Attribute | Type | Description
--------- | ------- | -----------
`first_name` | `string` | First name
`last_name` | `string` | Last name
`company` | `string` | Company name
`address_1` | `string` | Address line 1
`address_2` | `string` | Address line 2
`city` | `string` | City name
`state` | `string` | ISO code or name of the state, province or district.
`postcode` | `string` | Postal Code
`country` | `string` | Country code in ISO 3166-1 alpha-2 format.
`email` | `string` | Email address
`phone` | `string` | Phone number



If client made an order trough your API, so you can attach custom details to `client_info`. The properties are:


Attribute | Type | Description
--------- | ------- | -----------
`internal_id` | `integer` | Unique identifier for your custom client id. (Not related to `customer_id`)
`first_name` | `string` | First name
`last_name` | `string` | Last name
`company` | `string` | Company name
`address_1` | `string` | Address line 1
`address_2` | `string` | Address line 2
`city` | `string` | City name
`state` | `string` | ISO code or name of the state, province or district.
`postcode` | `string` | Postal Code
`country` | `string` | Country code in ISO 3166-1 alpha-2 format.
`email` | `string` | Email address
`phone` | `string` | Phone number


## Create new booking

> To create new booking:

```php
<?php

// Limited duration
$data = [
  'product_id': 1,
  'start_date': "2018-12-28 14:00",
  'end_date': "2018-12-28 18:00",
  'client_id': 193
  'client_first_name': "John",
  'client_last_name': "Doe",
  'client_address_1': "123 Market",
  'client_address_2': "",
  'client_city': "San Francisco",
  'client_state': "CA",
  'client_postcode': "94103",
  'client_country': "US",
  'client_email': "john.doe@example.com",
  'client_phone': "(555) 555-5555"
  ];
  
// Unlimited duration. Start immediately.  
$data = [
  'product_id': 1,
  'unlimited': 1
  // Client info
  ];
  
```

```shell
curl -X POST https://www.zerosix.ai/wp-json/api/v1/bookings \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
    -d '{
  "product_id": 1,
  "start_date": "2018-12-28 14:00",
  "end_date": "2018-12-28 18:00",
  "client_id": 193
  "client_first_name": "John",
  "client_last_name": "Doe",
  "client_address_1": "123 Market",
  "client_address_2": "",
  "client_city": "San Francisco",
  "client_state": "CA",
  "client_postcode": "94103",
  "client_country": "US",
  "client_email": "john.doe@example.com",
  "client_phone": "(555) 555-5555"
  }
  
```

> JSON response example:

```json
{
    "id": 321,
    "order_id": 320,
    "order_key": "wc_order_somerandomkey",
    "booking_status": "unpaid",
    "order_status": "pending",
    "date_created": {
        "date": "2018-12-28 12:22:00.000000",
        "timezone_type": 1,
        "timezone": "+00:00"
    },
    "date_modified": {
        "date": "2018-12-28 12:22:00.000000",
        "timezone_type": 1,
        "timezone": "+00:00"
    },
    "start_time": {
        "date": "28-12-2018 14:00:00",
        "timestamp": 1546005600
    },
    "end_time": {
        "date": "28-12-2018 18:00:00",
        "timestamp": 1546020000
    },
    "discount_total": "0",
    "discount_tax": "0",
    "total": "2.00",
    "total_tax": "0",
    "currency": "USD",
    "customer_id": 2,
    "client_info": {
        "internal_id": "193",
        "first_name": "John",
        "last_name": "Doe",
        "company": "",
        "address_1": "123 Market",
        "address_2": "",
        "city": "San Francisco",
        "state": "CA",
        "postcode": "94103",
        "country": "US",
        "email": "John.doe@example.com"
        },
    "transaction_id": "",
    "date_paid": null,
    "date_completed": null,
    "payment_url": "/order-pay/320/?pay_for_order=true&key=wc_order_somerandomkey",
    "items_booked": {
        "product_id": 1,
        "product_name": "NVIDIA 1080ti, 100GB SSD",
        "quanity": 1,
        "tax_status": "taxable"
    },
    "coupon_lines": []
}
```
This API helps you to create a new booking.

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-put">PUT</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings</h6>
    </div>
</div>

### HTTP Request


Parameter | Description
--------- | -----------
`product_id` | The product ID we want to book
`start_date` | The start date and time we want to book. format is: `YYYY-MM-DD H:i`
`end_date` | The end date and time we want to book. format is: `YYYY-MM-DD H:i`
`unlimited` | If `unlimited = 1`, then the booking **starts immediately with unlimited duration**. Default is `unlimited = 0`.
`client_id` | Optional. Your client id. 
`client_first_name` | Optional. Your client first name.
`client_last_name` | Optional. Your client last name.
`client_company` | Optional. Your client company.
`client_address_1` | Optional. Your client address line 1.
`client_address_2` | Optional. Your client address line 2.
`client_city` | Optional. Your client city.
`client_state` | Optional. Your client state.
`client_postcode` | Optional. Your client postcode. 
`client_country` | Optional. Your client country.
`client_email` | Optional. Your client email.


### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`booking_invalid_dates` | `404` | Invalid start or end dates. Please follow the format of YYYY-MM-DD H:i.
`Error` | `400` | `"This block cannot be booked"`, This booking is not available on this dates. Please try different times.

## Complete Booking

> To complete booking:

```php
<?php

    $DEL_URL = "https://www.zerosix.ai/wp-json/api/v1/bookings/<ID>";
    $data = array();
  
    $options = array(
      'http' => array(
        'header'  => "Content-type: application/json\r\n",
        'method'  => 'DELETE',
        'content' => json_encode($data)
             )
        );
    $context  = stream_context_create($options);
    
    $result = file_get_contents($GET_URL, false, $context);

  ?>
  

  
```

```shell
curl -X DELETE https://www.zerosix.ai/wp-json/api/v1/bookings/<ID> \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
 
  
```

> JSON response example:
*None*

When user wants to finish his currently booking process **with unlimited plan**, a request should be made in order to complete the order. 


<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-del">DEL</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings/(ID)</h6>
    </div>
</div>

### HTTP Request


Parameter | Description
--------- | -----------
`id` | The booking id we want to complete.



### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`invalid_booking_id` | `404` | Invalid booking Id
`invalid_booking_permissions` | `401` |  You are not authorized to modify this booking
   | `204` |  No content. Everything went fine.


## Check available dates

> To check available dates:

```php

$data = [
  'product_id': 1,
  'start_date': "2018-12-28 18:00",
  'end_date': "2018-12-28 22:00",
  ];
  
```

```shell
curl -X GET https://www.zerosix.ai/wp-json/api/v1/bookings/available \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
    -d '{
  "product_id": 1,
  "start_date": "2018-12-28 18:00",
  "end_date": "2018-12-28 22:00"
      }
  
```


> JSON response example:

```json
[
    "28/12/2018 18:00",
    "28/12/2018 19:00",
    "28/12/2018 20:00",
    "28/12/2018 21:00",
    "28/12/2018 22:00"
]
```


In order to place a booking with limited duration, you should check first the available dates and times. 

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>... zerosix.ai/wp-json/api/v1/bookings/available</h6>
    </div>
</div>

### Query Parameters

Parameter | Description
--------- | -----------
`product_id` | The ID of the product we want to check availability.
`start_date` | Start date of the booking
`end_date` | end date of the booking
`timestamp` | Optional. You can retrieve the dates in timestamp. Set `timestamp` flag to 1. Default is 0.

<aside class="notice">
Please notice that our bookings times is only at <strong>rounded</strong> times, and the response contains available hours by blocks.
</aside>




## Calculate the cost

> To calculate the cost:

```php

$data = [
  'product_id': 1,
  'start_date': "2018-12-28 18:00",
  'end_date': "2018-12-28 22:00",
  ];
  
```

```shell
curl -X GET https://www.zerosix.ai/wp-json/api/v1/bookings/cost \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
    -d '{
  "product_id": 1,
  "start_date": "2018-12-28 18:00",
  "end_date": "2018-12-28 22:00"
        }
  
```


> JSON response example:

```json
{
    "cost": 2
}
```

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings/cost</h6>
    </div>
</div>

### Query Parameters

Parameter | Description
--------- | -----------
`product_id` | The ID of the product we want to check availability.
`start_date` | Start date of the booking
`end_date` | end date of the booking
`hours` | Optional. You can calculate the cost by `hours` parameter. (`Integer`)


## List all bookings

This API helps you to view all the bookings.

> To list all the bookings made by you:

```php

$data = [

  ];
  
```

```shell
curl -X GET https://www.zerosix.ai/wp-json/api/v1/bookings/ \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
    -d '{}

  
```


> JSON response example:

```json
[
    {
        "id": 329,
        "order_id": 328,
        "order_key": "wc_order_randomKey",
        "booking_status": "unpaid",
        "order_status": "pending",
        "date_created": {
            "date": "2018-12-13 12:34:44.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "date_modified": {
            "date": "2018-12-13 12:34:44.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "start_time": {
            "date": "30-12-2018 18:00:00",
            "timestamp": 1546192800
        },
        "end_time": {
            "date": "30-12-2018 22:00:00",
            "timestamp": 1546207200
        },
        "discount_total": "0",
        "discount_tax": "0",
        "total": "2.00",
        "total_tax": "0",
        "currency": "USD",
        "customer_id": 2,
        "client_info": {
            "internal_id": "456",
            "first_name": "John",
            "last_name": "Doe",
            "company": "Company",
            "address_1": "Address 1",
            "address_2": "",
            "city": "Santa Barbara",
            "state": "CA",
            "postcode": "12354",
            "country": "US",
            "email": "John.doe@example.com",
            "phone": "123456789"
        },
        "transaction_id": "",
        "date_paid": null,
        "date_completed": null,
        "payment_url": "order-pay/328/?pay_for_order=true&key=wc_order_5c1251e4a3884",
        "items_booked": {
            "product_id": 1,
            "product_name": "NVIDIA 1080ti, 100GB SSD",
            "quanity": 1,
            "tax_status": "taxable"
        },
        "coupon_lines": []
    },
    {
        "id": 327,
        "order_id": 326,
        "order_key": "wc_order_randomKey",
        "booking_status": "unpaid",
        "order_status": "pending",
        "date_created": {
            "date": "2018-12-13 12:33:07.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "date_modified": {
            "date": "2018-12-13 12:33:07.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "start_time": {
            "date": "30-12-2018 14:00:00",
            "timestamp": 1546178400
        },
        "end_time": {
            "date": "30-12-2018 18:00:00",
            "timestamp": 1546192800
        },
        "discount_total": "0",
        "discount_tax": "0",
        "total": "2.00",
        "total_tax": "0",
        "currency": "USD",
        "customer_id": 2,
        "client_info": {
            "internal_id": "456",
            "first_name": "John",
            "last_name": "Doe",
            "company": "Company",
            "address_1": "Address 1",
            "address_2": "",
            "city": "",
            "state": "IL",
            "postcode": "CA",
            "country": "US",
            "email": "John.doe@example.com",
            "phone": ""
        },
        "transaction_id": "",
        "date_paid": null,
        "date_completed": null,
        "payment_url": "/order-pay/326/?pay_for_order=true&key=wc_order_randomKey",
        "items_booked": {
            "product_id": 1,
            "product_name": "NVIDIA 1080ti, 100GB SSD",
            "quanity": 1,
            "tax_status": "taxable"
        },
        "coupon_lines": []
    }
 ]
```

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings/</h6>
    </div>
</div>

### Query Parameters

*none* for now.

## Retrieve a booking

This API lets you retrieve and view a specific booking made by you.

> To retrieve booking:

```php

$data = [

  ];
  
```

```shell
curl -X GET https://www.zerosix.ai/wp-json/api/v1/bookings/327 \
    -u consumer_key:consumer_secret \
    -H "Content-Type: application/json" \
    -d '{}
  
```


> JSON response example:

```json
    {
        "id": 327,
        "order_id": 326,
        "order_key": "wc_order_randomKey",
        "booking_status": "unpaid",
        "order_status": "pending",
        "date_created": {
            "date": "2018-12-13 12:33:07.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "date_modified": {
            "date": "2018-12-13 12:33:07.000000",
            "timezone_type": 1,
            "timezone": "+00:00"
        },
        "start_time": {
            "date": "30-12-2018 14:00:00",
            "timestamp": 1546178400
        },
        "end_time": {
            "date": "30-12-2018 18:00:00",
            "timestamp": 1546192800
        },
        "discount_total": "0",
        "discount_tax": "0",
        "total": "2.00",
        "total_tax": "0",
        "currency": "USD",
        "customer_id": 2,
        "client_info": {
            "internal_id": "456",
            "first_name": "John",
            "last_name": "Doe",
            "company": "Company",
            "address_1": "Address 1",
            "address_2": "",
            "city": "",
            "state": "IL",
            "postcode": "CA",
            "country": "US",
            "email": "John.doe@example.com",
            "phone": ""
        },
        "transaction_id": "",
        "date_paid": null,
        "date_completed": null,
        "payment_url": "/order-pay/326/?pay_for_order=true&key=wc_order_randomKey",
        "items_booked": {
            "product_id": 1,
            "product_name": "NVIDIA 1080ti, 100GB SSD",
            "quanity": 1,
            "tax_status": "taxable"
        },
        "coupon_lines": []
    }
```

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings/(id)</h6>
    </div>
</div>

### Query Parameters

*none* for now.


# Credentials
In order to receive the credentials of the order, a request should be made. 



## Receive Credentials

> The response of credentials:

```json
{
    "order_id": 1,
    "order_key": "wc_order_randomkey",
    "bookings": [
        {
            "BOOKING_ID": 2,
            "SSH_KEY": "URL",
            "SSH_HOSTNAME": "renter@127.0.0.1",
            "SSH_PORT": "80"
        },
        {
            "BOOKING_ID": 3,
            "SSH_KEY": "URL",
            "SSH_HOSTNAME": "renter@127.0.0.2",
            "SSH_PORT": "80"
        },
}
```

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/credentials/(order_key)</h6>
    </div>
</div>


### HTTP Request


Parameter | Description
--------- | -----------
`order_key` | Must mention an `order_key` to receive the credentials. 



### HTTP Respones

Parameter | Status | Description
--------- | ----------- | -----------
`credentials_invalid_order_key` | `400` | Invalid Order Key
`credentials_invalid_id` | `400` |  Problem with order_key. Please mention valid order_key
