#  IOC2RPZ - turns your threat intelligence into RPZ feeds.

## How to start ioc2rpz service

## Configuration
1. AXFR update time - full Zone update and rebuild if MD5 is different. IOC added as insert_new (zone, ioc) -> to have an ability support IXFR
2. IXFR update time - incremental zone update
### Config file

[:AXFR:] = url,
[:FDateTime:] = "2017-10-13 13:13:13", [:FDateTimeZ:] = "2017-10-13T13:13:13Z", [:FTimestamp:] = 1507946281
[:ToDateTime:] = "2017-10-13 13:13:13", [:ToDateTimeZ:] = "2017-10-13T13:13:13Z", [:ToTimestamp:] = 1507946281

### Constants - ioc2rpz.hrl


### How the Full (AXFR) and Incremental(IXFR) caches are updated
- AXFR cache always contains prebuilt zones w/o SOA/NS/TSIG records. Prebuilt means all records are splitted by packets and labels were shortened/zipped.
- If server recieve an AXFR request it retrieve packets from the AXFR cache, add SOA/NS records and if needed TSIG.
- AXFR zones update should be considered as a clean up procedure, which should periodicaly take place. Just to be sure that there is no desynchronization between the sources and the cache.
- For large zones, AXFR updates should be scheduled infrequently to minimize impact on server's performance and ammount of transferred data to all clients.
- All changes if it is possible should be done via incremental zone updates. In that case the AXFR cache will be rebuilt only in case if a zone was updated.
- [TODO] Due to an optimization, only last packet will be rebuilt for new IOCs and relevant and accordant packets for the expired IOCs.
- IXFR cache contains only IOCs and expiration dates. [TODO] and packets ID's (to make it possible rebuild the zone fast).
- RPZ record contains current zone Serial and Serial_IXFR. Serial_IXFR serve as a minimum incremental zone serial which is available for an incremental zone transfer.
- IXFR cache is flushed after full zone update (AXFR). Serial_IXFR = Serial. Clients will recieve full zone update in any case, this is why it is important to have AXFR zone transfer infrequently.
- When IXFR is updated, AXFR cache must be rebuilt.
- If a zone does not support IXFR updates -> it doesn't saved in the IXFR table

### Hot cache
IXFR is not cached in the hot cache

## TODO features
- [x] IOC individual expiration time - update time will be used as minimum time for full zone refresh + check the SOA
- [x] Logs: Good timestamp, IP into normal format
- [ ] TSIG Found Key ... 2017-10-24 10:58:58 Bad timestamp ... Valid MAC
- [X] Cache for IXFR (save IOC rules with Exp.Time) and rebuild AXFR cache
-- [X] check AXFR zone update with new records
-- [X] check zone update with expired records
- [ ] Support DNS UDP IXFR/SOA request
- [ ] Incremental zone transfer/IXFR
- [ ] Save "cache config"
- [ ] DNS Notify messages
- [ ] Add source RPZ
- [ ] Add source MySQL
- [ ] http/https/ftp errors handling - source status in the record
- [ ] Reread CFG by a signal/TCP DNS request from localhost
- [ ] Refresh a zone by a signal/TCP DNS request from localhost
- [ ] Mnesia for storage (and automatic creation)
- [ ] Distributed configuration based on mnesia
- [ ] Fix IPv6 reversing "cleanup"
- [ ] Fool proof cfg
- [ ] Hot cache for IXFR IOC
- [ ] source based on files check by mod. date and size

## Other/optimization TODO
- [ ] Clean up the code & add comments
- [X] Move connectors to a new file
- [ ] Documentation
- [ ] IOC to lowercase - check performance impact
- [x] Do not save in IXFR cache zonex w/o IXFR
- [ ] Memory optimization for huge zones

## Bugs
- [x] Live zone - ;; Got bad packet: bad compression pointer
- [x] Updated by IXFR zone - +2 answer записи в заголовке ;; Warning: Message parser reports malformed message packet.
- [x] После AXFR запустился IXFR ---- похоже это не баг. просто время IXFR пришло, а AXFR не было
2017-10-24 00:45:39 Zone "last50.ioc2rpz" serial 1508830899, refresh time 3600 current status ready
2017-10-24 00:45:40 Zone "last50.ioc2rpz", Last packet ACOUNT 24, packets 1
2017-10-24 00:45:40 Zone "last50.ioc2rpz" updated in 1 seconds, new serial 1508831139

### References

Packets
https://tools.ietf.org/html/rfc6895

Records
https://www.ietf.org/rfc/rfc1035.txt

IXFR
https://tools.ietf.org/html/rfc1995

DNS Headers
https://github.com/blackberry/Erlang-OTP/blob/master/lib/kernel/src/inet_dns.hrl

RPZ
https://tools.ietf.org/html/draft-ietf-dnsop-dns-rpz-00

TSIG etc
https://tools.ietf.org/html/rfc2845
https://tools.ietf.org/html/rfc2104
https://tools.ietf.org/html/rfc4635

DNS over TCP
https://tools.ietf.org/html/rfc5966

DNS Notify
https://tools.ietf.org/html/rfc1996