# orclapex-websockets

An Oracle PL/SQL package to work with websockets.

The database cannot transfer data via the [WebSocket](https://en.wikipedia.org/wiki/WebSocket) protocol. It must send data via HTTP to a web server which is capable of using websockets.

## WEBSOCKET_PKG package
* [Data Types](#data-types)
* [Global Variables](#global-variables)
* [REGISTER procedure](#register-procedure)
* [UNREGISTER procedure](#unregister-procedure)
* [EMIT_TO_USER procedure](#emit_to_user-procedure)
* [EMIT_TO_ROOM procedure](#emit_to_room-procedure)

## Data Types

```sql
subtype t_room is varchar2(4000);
subtype t_token is varchar2(4000);
subtype t_event is varchar2(130); -- same as DA Custom Event Name in APEX 5.1
```

## Global Variables

```sql
c_prefix    constant varchar2(20) := 'websocket:';
c_separator constant varchar2(1)  := ':';

-- Predefined websocket event names
c_message   constant t_event := c_prefix || c_separator || 'message';
c_refresh   constant t_event := c_prefix || c_separator || 'refresh';
```

You may use custom event names instead of a predefined event name.

## REGISTER procedure

Register a user session for a websocket room and return JSON Web Token (JWT).
The token can be used by the client later to actually join the room.

```sql
WEBSOCKET_PKG.REGISTER (
  p_room            IN t_room,
  p_user_name       IN VARCHAR2,
  p_session_id      IN VARCHAR2,
  p_token           OUT t_token
);
```

## UNREGISTER procedure

Removes the user access for a room.

```sql
WEBSOCKET_PKG.UNREGISTER (
  p_room            IN t_room,
  p_user_name       IN VARCHAR2,
  p_session_id      IN VARCHAR2
);
```

### Parameters

| Parameter | Description |
| --- | --- |
| p_room | Name of the websocket room to join. |
| p_user_name| Name of a user. |
| p_session_id | A unique session ID. |

## EMIT_TO_USER procedure

Send a message to a single user to trigger a JavaScript event.

```sql
WEBSOCKET_PKG.EMIT_TO_USER (
  p_user_name       IN VARCHAR2,
  p_event           IN t_event,
  p_data            IN CLOB DEFAULT NULL
);
```


### Parameters

| Parameter | Description |
| --- | --- |
| p_user_name | Name of an user. |
| p_event| Name of the websocket event to emit. |
| p_data | Additional data to include in the event. Must be in valid JSON format. The format is checked by [APEX_JSON.PARSE](http://docs.oracle.com/cd/E59726_01/doc.50/e39149/apex_json.htm#AEAPI29747). |

### Example

```sql
-- Send message to one user to trigger a message event
-- This will trigger the websocket:message event for all dynamic actions which use this event
websocket_pkg.emit_to_user (
  p_user_name  => 'DEMO',
  p_event => websocket_pkg.c_message,
  p_data  => '{"message": "Hello World!"}'
);
```

## EMIT_TO_ROOM procedure

Send a message to all users in a websocket room.

### syntax

```sql
WEBSOCKET_PKG.EMIT_TO_ROOM (
  p_room    IN VARCHAR2,
  p_event   IN t_event,
  p_data    IN CLOB DEFAULT NULL
);
```

### Parameters

| Parameter | Description |
| --- | --- |
| p_room | Name of the WebSocket room. If you are creating a lot of rooms in you application, a good name for a room would be 'prefix:entity:id'.  |
| p_event| Name of the WebSocket event to trigger.                     |
| p_data | Additional data to include in the event. Must be in valid JSON format. The format is checked by [APEX_JSON.PARSE](http://docs.oracle.com/cd/E59726_01/doc.50/e39149/apex_json.htm#AEAPI29747). |

### Example

```sql
-- Send message of custom event to room
-- The client can perform any action on the custom event via JavaScript
websocket_pkg.emit_to_room (
  p_room => 'hr:emp:1234',
  p_event => 'RefreshEmployees'
);

-- Send message to show a message to all users in a room
websocket_pkg.emit_to_room (
  p_room  => 'global',
  p_event => 'websocket:message',
  p_data  => '{"message": "Hello World!"}'
);
```

## ACL

You need to be able to access the Node.JS server to make the REST calls.
