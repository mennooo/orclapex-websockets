# orclapex-websockets-plsql

An Oracle PL/SQL package to work with websockets.

The database cannot transfer data via the [WebSocket](https://en.wikipedia.org/wiki/WebSocket) protocol to APEX users. It must send data via HTTP to a web server which is capable of using websockets.

## APEX_WEBSOCKETS package
* [Data Types]()
* [EMIT_TO_USER procedure](#EMIT_TO_USER procedure)
* [EMIT_TO_ROOM procedure](#EMIT_TO_ROOM procedure)

## Data Types

* t_event

## Global Variables

```sql
c_prefix    constant varchar2(20) := 'websocket:';
c_separator constant varchar2(1)  := ':';

c_message   constant t_event := c_prefix || c_separator || 'message';
c_refresh   constant t_event := c_prefix || c_separator || 'refresh';
```

## EMIT_TO_USER procedure

Send a message to a single APEX user to trigger a JavaScript event.

```sql
APEX_WEBSOCKETS.EMIT_TO_USER (
  p_user            IN VARCHAR2,
  p_event           IN t_event,
  p_data            IN CLOB DEFAULT NULL
);
```


### Parameters

| Parameter | Description |
| - | - |
| p_user | Name of an APEX user. |
| p_event| Name of the JavaScript event to trigger in APEX. The same event name has to be used to trigger a Dynamic Action in APEX |
| p_data | Additional data to include in the event. Must be in valid JSON format. The format is checked by [APEX_JSON.PARSE](http://docs.oracle.com/cd/E59726_01/doc.50/e39149/apex_json.htm#AEAPI29747). |

### Example

```sql
-- Send message to one user to trigger a message event
-- This will trigger the websocket:message event for all dynamic actions which use this event
apex_websockets.emit_to_gbrk (
  p_room  => 'DEMO',
  p_event => apex_websockets.c_message,
  p_data  => '{"message": "Hello World!"}'
);
```

## EMIT_TO_ROOM procedure

Send a message to all APEX users in a websocket room.

### syntax

```sql
APEX_WEBSOCKETS.EMIT_TO_ROOM (
  p_room    IN VARCHAR2,
  p_event   IN t_event,
  p_data    IN CLOB DEFAULT NULL
);
```

### Parameters

| Parameter | Description |
| - | - |
| p_room | Name of the websocket room. A good name for a room would be 'prefix:entity:id'. |
| p_event| Name of the JavaScript event to trigger in APEX. |
| p_data | Additional data to include in the event. Must be in valid JSON format. The format is checked by [APEX_JSON.PARSE](http://docs.oracle.com/cd/E59726_01/doc.50/e39149/apex_json.htm#AEAPI29747). |

### Example

```sql
-- Send message to all users in a room to trigger a refresh
-- for an item/ region which is associated with employee data
apex_websockets.emit_to_room ('hr:emp:1234', apex_websockets.c_refresh, 'employees');

-- Send message to show a message to all users in a room
apex_websockets.emit_to_room (
  p_room  => 'hr:emp:1234',
  p_event => apex_websockets.c_message,
  p_data  => '{"message": "Hello World!"}'
);
```

## ACL
