# Reverse HTTP Tunnel #

### public http/s gateway (central gateway) ###
* installed on public server
* run as web component on rest url

### public http/s endpoint (remote endpoint) ###
* installed on public web farm
* act as web agent or proxy
* can support nginx reverse proxy
* run as web component on rest url

### protected http/s service (local service) ### 
* general http service installed in protected zone

### protected web proxy (proxy) ###
* general http or socket or no proxy

### protected work agent (local agent) ###
* run in protected zone
* perform initial http connection setup via proxy or direct tunnel channels
* http connection use
  - long poll get connection and receive remote command
  - use server sent events to get server send back command
  - use web socket to get server side command
  - try to leverage HTTP2

### case design: agent to gateway connection setup ###
* local agent handshake to gateway with agent id, password
* get configured domain and settings
* setup virtual idle work connections, and virtual status/monitor/control connection

### case design: client data flow ###
* browser send http request to public url
* nginx receive request and proxy to public endpoint
* endpoint receive request and use library to check if requested service is available, otherwise send error response
* endpoint send wrapped request as command to gateway library (may via remote interface or local api call)
* endpoint receive response from gateway library

### case design: library to gateway ###
* pick connection to agent
* gateway reply response with wrapped command with command uuid
* gateway wait for next request from agent for command uuid post
* the gateway reply result to library caller

## Components ##

### Local agent ###
* agent app
* agent web app
* agent library
  - handshake
  - agent id and password
  - get supported domain and settings
  - info set by remote
  - data encrypted
  - communication to domain server use ssl defined in control data
  - smart select channel
  - stream send reply with perf/track info
* outbound channel
  - direct tcp socket
  - existing pipeline
  - http/s connection
    - long poll
    - server sent events
    - web socket
  - http proxy connection
  - socket proxy connection
* virtual streams
  - single connection
  - multiple connections
  - http2
* can split web hook for request and agent listen on listen server (like notification server), then perform another request to gateway or specified server in notification.
* monitor, control
  - exported local interface
  - imported remote interface
  - collected performance

### Control server ###
* interface with resource style
* http implementation
* store and config agent settings
* provide connected agent and history info

### Gateway server ###
* interface with resource style
* http implementation
* open access for connection from agent