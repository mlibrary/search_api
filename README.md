# Search API
Documentation for the API between Pride and Spectrum

## General Notes

When the type of a value is STRING it should not contain any HTML. The client is responsible for escaping HTML entities, so any HTML tags in STRING values will be mangled.

## Objects from the Server

### Responses

#### Basic Format

```javascript
{
  'messages': [
    // If there are no messages it should be an empty array.
    // Each message is formatted like this:
    {
      'class': 'error' | 'warning' | 'success' | 'info'
        // 'error'   Something went seriously wrong and either no results were
        //           produced or the content of this response should be
        //           considered invalid.
        //
        // 'warning' There wasn't a serious problem, but the user should be
        //           notified of a potential issue (upcoming shutdowns, error
        //           parsing part of the request, etc).
        //
        // 'success' The request was successful and here are details about what
        //           the server did.
        //
        // 'info'    A generic notification, probably unrelated to the search
        //           result, such as libraries closed due to weather or a new
        //           service is available.

      'summary': STRING
        // A short summary of what when wrong. Only a couple of sentences.
        // Examples: 'Database is not responding' or 'Upcoming shutdown'.

      'details': HTML
        // A message that can be displayed to the user to give further
        // explanation or help. The HTML shouldn't have any attributes applied,
        // just basics tags such as <p>, <ol>, <ul>, <strong>, <em>, <etc>.
    }
  ]

  'request': OBJECT
    // A copy of the object sent to the server to generate this request.
    // If no object was sent, this field may be excluded.

  'response': ARRAY | @
    // When there is no result of the request or an error occurred the array
    // or object should empty.
    // Each request type should only be able to return an array or an object,
    // not either. Example: datastore queries return an array, while 
}
```

#### Datastore Query Responses

```javascript
{
  // All the usual data ('messages', 'request', etc) is included. See above for
  // how this information is organized.

  'total_available': INT | UNDEFINED
    // How many total records are available for this particular request.
    // If left undefined it is assumed

  'datastore': OBJECT
    // The object for the datastore that is responding.
    // This is sent in the response since it may have new default facet values
    // based on the search you just did.
    // Note: this may not be the same datastore that you sent the query to. In
    // the case of a multisearch you may have been sent to a new datastore.

  'new_request': OBJECT
    // If the client wants to redo the search with a different pagination, they
    // should send this version of the request object with modified start and
    // count values.
    // This is used for things like multisearches, where initially the datastore
    // object given was a simplified version, but the datastore object returned
    // by the query is more complicated and may use different formatting.

  // The 'response' field should contain an array of record objects. See below
  // for how those are formatted.
}
```

#### Facet Query Responses

```javascript
{
  // All the usual data ('messages', 'request', etc) is included. See above for
  // how this information is organized.

  'new_request': OBJECT
    // If the client wants to redo the search with a different pagination, they
    // should send this version of the request object with modified start and
    // count values.

  'total_available': INT | UNDEFINED
    // How many total records are available for this particular request.

  facet: OBJECT
    // The object for the datastore that is responding.
    // This is sent in the response since it may have new default facet values
    // based on the search you just did.
    // Note: this may not be the same datastore that you sent the query to. In
    // the case of a multisearch you may have been sent to a new datastore.

  // The 'response' field should contain an array of record objects. See below
  // for how those are formatted.
}
```

### Datastore Objects

Datastore Objects contain all the information necessary to create an interface for a specific search and to generate a query. "Bento box" style searches are represented the same way as any other datastore, but they give multiple URLs to send the query to.

```javascript
{
  'uid': STRING
    // A machine readable identifier for this datastore.
    // Only contains word characters (letters, numbers, underscores).

  'metadata': {
    'name': STRING
      // A human readable name for the datastore.

    'short_desc': STRING
      // A single sentence description of what this datastore contains.
  }

  'url': URL
    // The URL where query objects can be sent.

  'sorts': ARRAY
    // An array containing the various ways the search results can be sorted
    // such as alphabetical, by relevance, by date, etc.
    // If there are no sorts available, this array should be empty.
    // See below for how sort objects are formatted.

  'default_sort': STRING
    // The 'uid' for the default sort type.

  'fields': [
    // Array of field objects which look like this:
    {
      'uid': STRING
        // A machine readable identifier for this datastore.
        // Only contains word characters (letters, numbers, underscores).
        // There should almost always be a field with a 'uid' of 'all_fields'.
        // The 'all_fields' is considered the default field which searches
        // everything.

      'metadata' : {
        'name': STRING
          // A human readable name for the field.

        'short_desc': STRING
          // A one to three sentence description of what this field is.
      }

      'required': BOOLEAN
        // Whether or not this field must be set in a query.

      'fixed': BOOLEAN
        // Whether or not this field can be edited by users.

      'default_value': STRING
        // The value that this field starts out with.
        // Normally this is just set to an empty string.
    }
  ]

  'facets': ARRAY
    // An array of facet objects (see below).

        'settings': [
          // Array of objects which tell you what settings are available on this
          // datastore.
    {
      'uid': STRING
        // A machine readable identifier for this particular setting.

      'description':
        // A human readable description for this setting.
    }
  ]
}
```

### Facet Objects

```javascript
{
  'uid': STRING
    // A machine readable identifier for this facet.
    // Only contains word characters (letters, numbers, underscores).

  'metadata' : {
    'name': STRING
      // A human readable name for the facet.

    'short_desc': STRING
      // A one to three sentence description of what this facet is.
  }

  'default_value': ANYTHING
    // The value that will be chosen by default if this facet is not set in
    // a query.

  'default_sort': STRING
    // The 'uid' for the default sort type.

  'values': [
    // Array of possible values for this facet formatted as follows:
    {
      'value': ANYTHING
        // The value that should be sent to the server.

      'name': STRING
        // A human readable name for this particular value.

      'count': INT
        // The number of results which have this value.
        // If not known, do not create the 'count' key/value pair.
    }
  ]

  'fixed': BOOLEAN
    // Whether or not this facet can be edited by users.

  'required': BOOLEAN
    // Whether or not this field must be set to something other than NULL in
    // a query.

  'more': URL | false
    // If 'more' is set to a URL, there are more facet values. If it is set
    // to false then the list of values is complete.
    // A facet value request object (see below) is sent to this URL to get
    // more values for this facet.

  'sorts': ARRAY
    // An array containing the various ways the facet values can be sorted
    // such as alphabetical, by result count, etc.
    // If there are no sorts available, this array should be empty.
    // See below for how sort objects are formatted.
}
```

### Sort Objects

```javascript
{
  'uid': STRING
    // A machine readable identifier for this sort order.
    // Only contains word characters (letters, numbers, underscores).

  'metadata': {
    'name': STRING
      // A human readable name for the sort order.

    'short_desc': STRING
      // A one to three sentence description of what this sort order is.
  }

  'group': STRING
    // If this sort belongs to a group, provide a machine readable (only
    // alphanumeric and underscores) name that will also be applied to every
    // other sort object in that group.
    // Example: if there is an ascending sort and a descending sort, you create
    // one sort object for each and put the same 'group' value in each object.
}
```

### Record Objects

```javascript
{
  'type': STRING
    // A machine readable 'uid' made out of word characters.
    // Identifies the type of record so the client has a hint about
    // what fields to expect.
    // Examples: 'catalog', 'summon', 'deep_blue'
    // Not used to control rendering, nor is it metadata to show
    // to the user. It is just to help the client understand what fields
    // to expect.

  'source': URL
    // A URL that returns the complete record for this object.
    // This URL can be used as a unique identifier for this record.

  'complete': BOOLEAN
    // Whether or not there is more info available on this record.

  'names': [
    // An array of human readable names for this record.
    // ALL elements are either HTML or STRING, you can not mix the two.
    ]

  'names_have_html': BOOLEAN
    // Whether or not the items in the 'names' array contain HTML.
    // Used to determine if those values need to be escaped.

  'fields': [
    // An array of objects that give the information about this record.
    // Each object is formatted as follows:

    {
      'uid': STRING
        // A machine readable identifier for this particular field.

      'name':
        // A human readable name for this field.

      'value': ANYTHING
        // Usually this will be a string which is safe to show to users, such as
        // a title, but it could be anything.

      'value_has_html': BOOLEAN
        // Whether or not the name contains HTML.
        // Used to determine if it this value needs to be escaped.
    }
  ]
}
```



## OBJECTS FROM THE CLIENT

The client only needs to know one URL. That is the URL which they can send a GET request to to get all the datastores the

### Queries

#### Datastore Queries

```javascript
{
  'uid': STRING
    // A machine readable identifier for the datastore you are querying.

  'request_id': ANYTHING
    // Some data that the server will not modify.
    // The 'request' object in the response will have an exact copy of what was
    // sent in this field.
    // The 'new_request' object in the response will not contain this value
    // because the 'new_request' object may not be related to this search such
    // as in the case of a multisearch search.

  'start': INT
    // The index of the item you want to start with.
    // 0 means you want the first set of results, while any other number means
    // you want to load another page of results.

  'count': INT
    // The number of results you are requesting.
    // Note: the server should never send more results than the client
    // requested. However, the server may send fewer results than requested.

  'field_tree': OBJECT
    // A tree that describes the fields being searched, the values to be placed
    // into those fields, and how to combine them together.
    // An empty object should be sent when not sending any values.
    // See below for how this tree is formated

  'facets': OBJECT
    // An object which contains the facets that should be applied to the search.
    // Each facet is represented as a key/value pair, where the key is the 'uid'
    // of the facet as defined by the datastore object and where the value is
    // the value given by the server, which could be any form of data (boolean,
    // string, object, array, number, etc).

  'sort': STRING
    // The 'uid' for the sort you want to use.

  'settings': OBJECT
    // An object of additional settings for how you want results formated.
    // Each settings should be set as a key/value pair, where the key is the
    // 'uid' for that setting and the value is whatever you want to set it to.
}
```

#### Facet Queries

```javascript
{
  // Same as a datastore query except for the following additions:

  facet_uid: STRING;
  // The 'uid' for the facet that you are requesting.
}
```

### Field Tree Nodes

```javascript
{
  'type': 'field_boolean' | 'field'   |
          'value_boolean' | 'tag'     |
          'literal'       | 'special'
    // A 'field_boolean' can have children that are 'field_boolean' or 'field'.
    // A 'field' or 'value_boolean' or 'tag' can have children of any type
    // except for 'field_boolean' and 'field'.
    // A 'literal' or 'special' node can not have any children.


  'value': STRING
    // In the case of a 'field_boolean' or 'value_boolean' the literal can be:
    //   'AND' | 'OR' | 'NOT'
    // For 'field' type nodes the value is the 'uid' for the field.
    // For 'literal' or 'special' nodes the value is a string, but in the case
    // of special nodes the string may have special meaning like it does for *
    // For 'tag' nodes the value is a string which assigns a special meaning
    // to its children, like the way + and - are used in some searches to say
    // that the following is or isn't mandatory.

  'children': ARRAY
    // The children of this node.
    // 'AND' and 'OR' booleans assume that all children are joined with that
    // particular boolean.
    // 'NOT' booleans assume that all children are joined together with 'AND',
    // then that group of nodes has the NOT on it. For example, if the children
    // of a 'NOT' are "something", "else" and "blue" then the serialization is:
    // NOT("something" AND "else" AND "blue")
}
```
