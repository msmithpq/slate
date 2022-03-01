---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell: CuRL
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the BMS API
---

# Overview 

 The primary function of the Book Meta Data Service (hereto forth, BMS) is to allow a client program to access the information that Bowker maintains on books and other media.

The client can obtain a list of records that satisfy a request or it can obtain details about a single record. The client can indicate what information it wants for each record, or it can simply state that it wants all information available for each record.

For example, to obtain the title and author associated with an ISBN, a client could issue a request like this one: 

 http://bms.bowker.com/rest/books/isbn/0-262-19502-X?fields=title,author

If the client wanted the result as an HTML page, it could tack a *format* parameter to the above request, like this:

http://bms.bowker.com/rest/books/isbn/0-262-19502-X?fields=title,author&format=html 

# Authentication

 The BMS uses HTTP Basic Authentication. The client program must provide credentials using Basic Authentication. The Perl reference client described later shows how to easily authenticate in Perl.

Based on the ID of an authenticated user, the BMS will provide the user with access to one or more markets and will allow the user to request specific fields. A user with the highest access level would be able to view records in every market and would have unrestricted access to fields so that, for a given record, all the information available to the BMS would be available also to the user.

A user with a low access level might, for example, be able to view records in the USA market and might be able to retrieve only Title, Subtitle, Author, and ISBN information for any record. 

# Identificaion of Records 

 Because the BMS tracks multiple records with the same ISBN, an ISBN alone is not sufficient to identify a record uniquely. And some records don't even have an ISBN. The BMS provides a special ID that identifies a record uniquely, the BMSID. When a client issues a query that includes a BMSID, the query is guaranteed to return a single record.

Where a record identified by a BMSID contains an ISBN, ISBN-13, EAN, or UPC, the BMSID can be considered immutable; it will always point to the same book. Most of the records that Bowker maintains (by far) fall into this catetgory. However, there are some records for which Bowker doesn't have a formal identifier. For these records, the BMSID is not guaranteed to always point to the same book. However, when the BMS uses a BMSID to reference one of these records, the reference will work for some time (most likely a long time). 
# Application Programming Interface

## Overview of API

The BMS uses a RESTful interface to expose book meta data. URIs are meaningful and for the most part static. They are easy to share. Consuming data is simple for any kind of client, including Web browsers, Perl, C#, Python, Ruby, XQuery, and even JavaScript. 

## Requests

The BMS allows you to obtain information about books in several ways. You can request information by providing a BMSID (which is a number that Bowker assigns to each book that it tracks -- see the section "Resource Representation," ahead), a standard ISBN, a DOI, a title, an author, a subject, or a more complex search that involves one or more of ISBN, DOI, Title, Author, or Subject. The general URI format for these requests is as follow: 


* ID: http://bms.bowker.com/rest/books/id/[number]
* One field: http://bms.bowker.com/rest/books/isbn/[number]
* Muiltiple fields: http://bms.bowker.com/rest/books/search?title=[text]&subject=[text]


Strings that you provide for search parameters should be URL encoded

## String Searches

If you provide more than one of the search parameters (ISBN, DOI, Title, Author, Subject), the BMS returns only records that meet all the supplied criteria. (The search values are logically ANDed). Also, when you provide a search string for one of the parameters title, author, or subject, the BMS tokenizes the search string (splits it into words) so that the string will match records where the words are out of order. This allows the search **title=things little** to match the record with title "All the Little Things." However, you when you enclose a string in quotes, the string is processed as a single word. Thus, the search **title="things little"** will not match the aforementioned title.

## Responses

The BMS responds with two types of resources when successful:

* search results -- A list of resources that satisfy the client's request
* record details -- Detailed information about a specific record (like a book, for example)

The BMS always returns a search results resource except when the query includes a BMSID, in which case the BMS understands that only one record could ever satisfy the query and the BMS returns a record details resource. 

The search results resource contains one or more records that, by default, consist only of a reference to a record details resource. Using the fields parameter, however, clients can specify the *fields* that the BMS should include in each record of a search results resource. 
 
The record details resource also allows the client to specify the fields that the BMS should provide. 

Rather than listing fields individually for the field parameter, the client can provide the value 'all', which the BMS will interpret depending on whether the BMS is returning a search results resource or a record details resource. When returning a search results resource, the BMS interprets the *fields* value 'all' to mean that the user wants all the fields that are not too long. So *annotations, chapter1,* and other such fields will be excluded from the records in the search results resource. However, the client can override this behavior by explicitly listing the fields that the BMS should display for records in the search results resource. 
When returning a record details resource, the BMS interprets the fields value 'all' to mean exactly that, all, and the BMS returns every field that the client has access to and that is available for each record, including long fields.

## Resource Representation

Each record details resource or search results resource that the BMS exposes to clients is inactuality the result of a search. The BMS performs the search using the parameters that the client provides. The BMS then formats the result from the Mark Logic server and possibly excludes certain fields depending on the client's access rights. 

Clients that request XHTML-formatted resources have the option of using CSS to format the resources in any way they see fit for consumption by end users. When the BMS returns a resource in XHTML format (pursuant to the client's format parameter submission), the XHTML code is tagged so that creating a CSS style sheet for the XHTML fragment is easy. Every XHTML tag contains a class attribute that is fairly unique and that can be referenced in CSS. 

Bowker references each unique resource that resolves to a record in BIPOL with a Book Meta Data Service ID (BMSID). A BMSID is a number that uniquely identifies records in Bowker's database. A given title, like "Cryptonomicon," might have a dozen records in our database; a BMSID can identify only one of them. 

## Resource URIs

The resources listed here support only the HTTP GET method. In essence, you can request a record list or the details for a record with one of the following types of URIs:

* [/rest/books/id/[number]] (#rest-books-id-number)
* [/rest/books/isbn/[number]] (#rest-books-isbn-number)
* /rest/books/upc/[number]
* /rest/books/oclc/[number]
* /rest/books/anyof?isbn=[number]&upc=[number]&oclc=[number]
* /rest/books/doi/[number]
* /rest/books/title/[text]
* /rest/books/author/[text]
* /rest/books/subject/[text]
* /rest/books/binding/[text]
* /rest/books/status/[text]
* /rest/books/publisher/[text]
* /rest/books/search?query_string
* /rest/books/extract/[text]?resume=[text]


## /rest/books/id/[number]

This resource consists of one element: details about a unique item. The number is a BMSID.

The resource accepts the following parameters:

* Results Details

  * fields
     * A comma-separated list of fields that you want in the response. The section fields list valid values for this parameter

  * filterdata
     * See the section The fileterdata parameter

  * userinfo
     * When the client sets this parameter to the value 1, the BMS returns user-information attributes in the parent tag

* Result Format

  * format
     * Valid formats are *html_fragment, html_page, xml,* and *js*. If the BMS doesn't see this parameter within the query string, the BMS assumes the value *xml* for this parameter

## /rest/books/isbn/[number]

This resource consists of book records that share the specified ISBN. The URI returns a list with one or more elements. 

Each item in the result list represents a specific resource (a unique book) and includes a reference to the details for that resource. 

The list resource accepts the following parameters: 

* Limiters
  * isbn
  * upc
  * doi
  * title
  * author
  * binding
  * status
  * publisher
  * qtext
* Results Details
  * descending
     * When you sort search results with the sort parameter, the BMS sorts in ascending order by default. If you provide the parameter descending=true, the BMS will sort results in descending order instead. 
  * fields 
     * A comma-separated list of fields that you want in the response. The section Fields lists valid values for this parameter. Each element in the response list contains contain these fields. Although it is possible to obtain all the information for each record in the list, Bowker advises that clients obtain a search-results list first, then use the BMSID provided with each record in the list to obtain details of the record in a separate call. In most cases, details for a record should be presented to the end user as one page.
filterdata


```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

