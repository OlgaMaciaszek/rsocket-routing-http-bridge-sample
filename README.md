# Spring Cloud Broker RSocket Sample

## Test RSocket Http Bridge Sample
- Run `BrokerApplication`, `BridgeApplication` and `ServiceApplication`.

Use the Bridge to communicate with the RSocket-based `ServiceApplication`, through RSocket Broker, 
using an HTTP client.

Try the following commands to test the 4 RSocket Interaction modes (`rr`- `request - response`, `rs` - `request - stream`, `rc` - `request - channel`, `ff` - `fire and forget`):
 - `http POST localhost:9080/rr/service/service-rr body=test`
 - `http POST localhost:9080/rs/service/service-rs body=test`
 - `http POST localhost:9080/rc/service/service-rc body=test`
 - `http POST localhost:9080/ff/service/service-ff body=test`

Use the following command to test the default function resolution (`request-response` unless otherwise specified in `spring.cloud.function.definition`):

- `http POST localhost:9080/service/service-ff body=test`



## TODO:

- [X] Move ping and pong to use Spring Framework RSocket Messaging rather than raw RSocket
- [ ] Multiple pong servers to highlight gateway load balancing.
- [ ] Golang ping requester to highlight RSocket polyglot
- [ ] JS ping requester from browser. (Likely needs changes in gateway)

## Succesful log messages

When ping and pong are communicating correctly, you should see logs like the following in pong:
```
2019-08-09 11:32:19.477  INFO 16726 --- [tor-tcp-epoll-1] o.s.c.r.s.pong.PongApplication$Pong      : received ping1(1) in Pong
2019-08-09 11:32:20.186  INFO 16726 --- [tor-tcp-epoll-1] o.s.c.r.s.pong.PongApplication$Pong      : received ping1(2) in Pong
```

And in ping you should see logs like:
```
2019-08-09 11:32:19.480  INFO 16077 --- [tor-tcp-epoll-1] o.s.c.r.s.ping.PingApplication$Ping      : received pong(1) in Ping1
2019-08-09 11:32:20.189  INFO 16077 --- [tor-tcp-epoll-1] o.s.c.r.s.ping.PingApplication$Ping      : received pong(2) in Ping1
```

## Direct Mode: No Broker

Run pong with `spring.profiles.active=server`. Then run ping.

## Single Broker Mode

Run gateway first. Then run ping. 

You should see backpressure logs like:
```
2019-08-09 11:30:59.812  INFO 15199 --- [     parallel-2] o.s.c.r.s.ping.PingApplication$Ping      : Dropped payload ping1
2019-08-09 11:31:00.811  INFO 15199 --- [     parallel-2] o.s.c.r.s.ping.PingApplication$Ping      : Dropped payload ping1
```

Run pong. 

## Broker Cluster Mode

Run broker for first node. The run another broker with `spring.profiles.active=broker2`

You should see logs like this in 2nd broker node:
```
2019-08-09 11:36:12.524 DEBUG 19644 --- [tor-tcp-epoll-1] o.s.c.broker.rsocket.registry.Registry  : Registering RSocket: [Metadata@5015196c name = 'ping', properties = map['id' -> 'pingproxy1']]
2019-08-09 11:36:12.526 DEBUG 19644 --- [tor-tcp-epoll-1] o.s.c.g.rsocket.registry.RegistryRoutes  : Created Route for registered service [Route@57801e07 id = 'ping', targetMetadata = [Metadata@5015196c name = 'ping', properties = map['id' -> 'pingproxy1']], order = 0, predicate = org.springframework.cloud.broker.rsocket.registry.RegistryRoutes$$Lambda$536/302508515@57d6f132, brokerFilters = list[[empty]]]
```

And in the first broker node:
```
2019-08-09 11:36:12.573 DEBUG 19475 --- [or-http-epoll-2] o.s.c.broker.rsocket.registry.Registry  : Registering RSocket: [Metadata@318e483 name = 'pong', properties = map['id' -> 'broker21']]
2019-08-09 11:36:12.575 DEBUG 19475 --- [or-http-epoll-2] o.s.c.g.rsocket.registry.RegistryRoutes  : Created Route for registered service [Route@6796b8e4 id = 'pong', targetMetadata = [Metadata@318e483 name = 'pong', properties = map['id' -> 'broker21']], order = 0, predicate = org.springframework.cloud.broker.rsocket.registry.RegistryRoutes$$Lambda$523/976465559@11bf03ce, brokerFilters = list[[empty]]]
2019-08-09 11:36:12.576 DEBUG 19475 --- [or-http-epoll-2] o.s.c.g.r.s.SocketAcceptorFilterChain    : filter chain completed with success
```

Run ping, you should see backpressure logs as above.

Run pong with `spring.profiles.active=broker2`.

You should see successful log messages in ping and pong.

## Additional ping client

During any mode, you can run another ping client with `spring.profiles.active=ping2`. The default ping client uses the 'request channel' RSocket method, where ping 2 uses 'request reply'.

The logs in pong will now show additional client pings such as:
```
2019-08-09 11:43:58.309  INFO 22645 --- [tor-tcp-epoll-1] o.s.c.r.s.pong.PongApplication$Pong      : received ping1(280) in Pong
2019-08-09 11:43:58.449  INFO 22645 --- [tor-tcp-epoll-1] o.s.c.r.s.pong.PongApplication$Pong      : received ping2(281) in Pong
```

## Profile Specific Configuration

To see what each profile is setting, see the `application.yml` for each individual project. Each profile is another yaml document.