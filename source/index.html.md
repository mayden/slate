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

Attribute | Type | Description
--------- | ------- | -----------
`id` | `integer` | Unique identifier for the product.
`title` | `string` | Product name
`slug` | `string` | Product slug
`description` | `string` | Product description
`permalink` | `string` | Product URL
`price` | `integer` | Product price
`related_ids` | `array` | List of related products IDs
`categories` | `array` | List of categories. Each category has `category_id` and `category_name`
`image` | `array` | Product image. `url`, `width`, `height` and `is_thumbail` properties. 

## List all Products

```php
  <?php
  //Initialize cURL.
  $ch = curl_init();
   
  //Set the URL that you want to GET by using the CURLOPT_URL option.
  curl_setopt($ch, CURLOPT_URL, 'https://www.zerosix.ai/wp-json/api/v1/products');
  
  //Set CURLOPT_RETURNTRANSFER so that the content is returned as a variable.
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
   
  //Set CURLOPT_FOLLOWLOCATION to true to follow redirects.
  curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
   
  //Execute the request.
  $data = curl_exec($ch);
   
  // $data contains all the products information
  print_r($data);
  
  //Close the cURL handle.
  curl_close($ch);
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
      "title": "NVIDIA 1080ti, 100GB SSD",
      "slug": "nvidia-1080ti-100gb-ssd",
      "description": "NVIDIA 1080ti with 250 GB SSD,",
      "permalink": "https://www.zerosix.ai/product/nvidia-1080ti-100gb-ssd/",
      "price": "0.5",
       "related_ids":
        [
          "2"
        ],
       "categories": 
       [
         {
          "category_id": 10,
          "category_name": "1080"
          },
          {
           "category_id": 11,
           "category_name": "1080ti"
          } 
       ],
       "image": 
       [
           "https://www.zerosix.ai/uploads/2018/11/1080ti_store-150x150.jpg",
           150,
           150,
           true
       ]
     },
     {
       "id": 2,
       "title": "NVIDIA 1080ti, 250 GB SSD",
       "slug": "nvidia-1080ti-250gb-ssd",
       "description": "NVIDIA 1080ti with 250 GB SSD",
       "permalink": "https://www.zerosix.ai/product/1080ti/",
       "price": "0.6",
        "related_ids": 
        [
          "1"
        ],
        "categories": 
        [
          {
            "category_id": 11,
            "category_name": "1080ti"
          }
        ],
          "image": [
            "https://www.zerosix.ai/uploads/2018/11/1080ti_store-150x150.jpg",
            150,
            150,
            true
             ]
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


## Retrieve a Product

```shell
  curl  https://www.zerosix.ai/wp-json/api/v1/products/1  \
        -u consumer_key:consumer_secret
```

> The above command returns JSON structured like this:

```json
{
      "id": 1,
      "title": "NVIDIA 1080ti, 100GB SSD",
      "slug": "nvidia-1080ti-100gb-ssd",
      "description": "NVIDIA 1080ti with 250 GB SSD,",
      "permalink": "https://www.zerosix.ai/product/nvidia-1080ti-100gb-ssd/",
      "price": "0.5",
       "related_ids":
        [
          "2"
        ],
       "categories": 
       [
         {
          "category_id": 10,
          "category_name": "1080"
          },
          {
           "category_id": 11,
           "category_name": "1080ti"
          } 
       ],
       "image": 
       [
           "https://www.zerosix.ai/uploads/2018/11/1080ti_store-150x150.jpg",
           150,
           150,
           true
       ]
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


#Bookings

This API endpoints allows you to create new order, retrieve information about specific order, see available booking dates and more. 

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
        <i class="label label-post">POST</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings</h6>
    </div>
</div>

### HTTP Request


Parameter | Description
--------- | -----------
`product_id` | The product ID we want to book
`start_date` | The start date and time we want to book. format is: `YYYY-MM-DD H:i`
`end_date` | The end date and time we want to book. format is: `YYYY-MM-DD H:i`
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
`Error` | `400` | `"This block cannot be cooked"`, This booking is not available on this dates. Please try different times.


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


In order to place a booking, you should check first the available dates and times. 

<div class="api-endpoint">
    <div class="endpoint-data">
        <i class="label label-get">GET</i>
        <h6>https://www.zerosix.ai/wp-json/api/v1/bookings/available</h6>
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
