[back](./epoch_api.md)
# Oracle WebSocket API

The WebSocket API provides five different actions:
 * [Register an Oracle](#register-an-oracle)
 * [Post a query for an Oracle](#post-a-query-for-an-oracle)
 * [Answer an Oracle query](#answer-an-oracle-query)
 * [Register for post query events](#register-for-post-query-events) (for a particular Oracle)
 * [Register for query response events](#register-for-query-response-events) (for a particular Oracle query)

## General Request/Response types
A request has the format:

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| target | string | what component to target | Yes |
| action | string | what is the action | Yes |
| payload | object | data for action | Yes |

A response (and an event) has the format:

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| origin | string | what component did the action | Yes |
| action | string | what was the action | Yes |
| payload | object | data from the action | Yes |

## Register an Oracle
### Request
 * **target:** `oracle`
 * **action:** `register`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | type | `OracleRegisterTxObject` | | Yes |
  | vsn | number | | No |
  | query_format | string |  | Yes |
  | response_format | string |  | Yes |
  | query_fee | number |  | Yes |
  | ttl | object - [TTL](#ttl) |  | Yes |
  | fee | number |  | Yes |

### Response
 * **origin:** `oracle`
 * **action:** `register`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | result | string | `ok` or error reason | Yes |
  | oracle_id | string | Hash for created oracle | Yes on success |

### Example
```
Request:
  {"target":"oracle",
   "action":"register",
   "payload":{"type":"OracleRegisterTxObject",
              "vsn":1,
              "query_format":"the query spec",
              "response_format":"the response spec",
              "query_fee":4,
              "ttl":{"type":"delta", "value":50},
              "fee":5}
  }

Response:
  {"action":"register",
   "origin":"oracle",
   "payload":{"result":"ok",
              "oracle_id":"ok$3jzZyCLFtHVD7yVdEhGJFM3LjeXrKqWxnHbCYzhnrrR4DkdFtaJuxQvrR8VbbXExDPkCHFAei5q969JA6EayQpb8z5C3Mf"}
  }
```

## Post a query for an Oracle
### Request
 * **target:** `oracle`
 * **action:** `query`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | type | `OracleQueryTxObject` | | Yes |
  | vsn | number | | No |
  | oracle_pubkey | string |  | Yes |
  | query | string |  | Yes |
  | query_fee | number |  | Yes |
  | query_ttl | object - [TTL](#ttl) |  | Yes |
  | response_ttl | object - [RelativeTTL](#relativettl) |  | Yes |
  | fee | number |  | Yes |

### Response
 * **origin:** `oracle`
 * **action:** `query`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | result | string | `ok` or error reason | Yes |
  | query_id | string | Hash for created query | Yes on success |

### Example
```
Request:
  {"target":"oracle",
   "action":"query",
   "payload":{"type":"OracleQueryTxObject",
              "vsn":1,
              "oracle_pubkey":"ok$3jzZyCLFtHVD7yVdEhGJFM3LjeXrKqWxnHbCYzhnrrR4DkdFtaJuxQvrR8VbbXExDPkCHFAei5q969JA6EayQpb8z5C3Mf",
              "query_fee":4,
              "query_ttl":{"type":"delta", "value":10},
              "response_ttl":{"type":"delta", "value":10},
              "fee":7,
              "query":"How are you?"
              }
  }

Response:
  {"action":"query",
   "origin":"oracle",
   "payload":{"result":"ok",
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}
  }
```

### Answer an Oracle query
### Request
 * **target:** `oracle`
 * **action:** `response`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | type | `OracleResponseTxObject` | | Yes |
  | vsn | number | | No |
  | query_id | string |  | No |
  | response | string |  | No |
  | fee | long |  | No |

### Response
 * **origin:** `oracle`
 * **action:** `response`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | result | string | `ok` or error reason | Yes |
  | query_id | string | Hash for the answered query | Yes on success |

### Example
```
Request:
  {"target":"oracle",
   "action":"response",
   "payload":{"type":"OracleResponseTxObject",
              "vsn":1,
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr",
              "fee":4,
              "response":"I am fine, thanks!"}
  }

Response:
  {"origin":"oracle",
   "action":"response",
   "payload":{"result":"ok",
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}
  }
```

### Register for post query events
### Request
 * **target:** `oracle`
 * **action:** `subscribe`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | type | `query` | | Yes |
  | oracle_id | string | | Yes |

### Response
 * **origin:** `oracle`
 * **action:** `subscribe`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | result | string | `ok` or error reason | Yes |
  | subscribed_to | object - query subscribe payload | | Yes on success |

### Event
 * **origin:** `node`
 * **action:** `new_oracle_query`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | sender | string | hash of query sender | Yes |
  | query | string | | Yes |
  | query_id | string | hash that identifies the query | Yes |

### Example
```
Request:
  {"target":"oracle",
   "action":"subscribe",
   "payload":{"type":"query",
              "oracle_id":"ok$3jzZyCLFtHVD7yVdEhGJFM3LjeXrKqWxnHbCYzhnrrR4DkdFtaJuxQvrR8VbbXExDPkCHFAei5q969JA6EayQpb8z5C3Mf"}
  }

Response:
  {"origin":"oracle",
   "action":"subscribe",
   "payload":{"result":"ok",
              "subscribed_to":{"type":"query",
                               "oracle_id":"ok$3jzZyCLFtHVD7yVdEhGJFM3LjeXrKqWxnHbCYzhnrrR4DkdFtaJuxQvrR8VbbXExDPkCHFAei5q969JA6EayQpb8z5C3Mf"}}
  }

Event:
  {"origin":"node",
   "action":"new_oracle_query",
   "payload":{"sender":"ak$3jzZyCLFtHVD7yVdEhGJFM3LjeXrKqWxnHbCYzhnrrR4DkdFtaJuxQvrR8VbbXExDPkCHFAei5q969JA6EayQpb8z5C3Mf",
              "query":"How are you?",
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}
  }
```

### Register for query response events
### Request
 * **target:** `oracle`
 * **action:** `subscribe`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | type | `response` | | Yes |
  | query_id | string | | Yes |

### Response
 * **origin:** `oracle`
 * **action:** `subscribe`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | result | string | `ok` or error reason | Yes |
  | subscribed_to | object - response subscribe payload | | Yes on success |

### Event
 * **origin:** `node`
 * **action:** `new_oracle_response`
 * **payload:**

  | Name | Type | Description | Required |
  | ---- | ---- | ----------- | -------- |
  | query_id | string | hash that identifies the query | Yes |
  | query | string | | Yes |

### Example
```
Request:
  {"target":"oracle",
   "action":"subscribe",
   "payload":{"type":"response",
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}
  }

Response:
  {"origin":"oracle",
   "action":"subscribe",
   "payload":{"result":"ok",
              "subscribed_to":{"type":"response",
                               "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}}
  }

Event:
  {"origin":"node",
   "action":"new_oracle_response",
   "payload":{"response":"I am fine, thanks!",
              "query_id":"oq$4RZoMEkm8QuuhJiiq53dd5pE4VstCthYRjBHgUKdhAhe7rLEr"}
  }
```

## Additional types
### TTL

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| type | string | `block` or `delta` | Yes |
| value | number |  | Yes |

### RelativeTTL

| Name | Type | Description | Required |
| ---- | ---- | ----------- | -------- |
| type | `delta` |  | Yes |
| value | number |  | Yes |


**Current limitation:** Since there is currently no interface for offline signing
(all signing is done by a node - and that node is associated with exactly one
account) it is somewhat limited what you can do with oracles. I.e. in practice
you need a node per account - obviously this is a limitation we should try to
lift as soon as possible.
