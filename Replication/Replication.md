REASON FOR REPLICATION (fault-taularant, durabililty):
	- Scalability  
			- (Processings more req)
	- Latency 
			- Place read replicas closer to user.

1.	Cross GEO replication DB
		- Keep READ-DB at different GEO
2.	Asyc & Sync Replication
		- we do not do sync replication. if any one node dies whole req will get rejected
		- async will not immediate consistent node same as master
		- we prefer semi-sync.
3.	Setting-up new followers
		- take last snapshot and then use binlog(after seq. no of snapshot) of master
4.	Handling Node Outage:
		- Follower dies:
				->	add new follower and then use point 3
		- Master dies : 
				->	how to know master died? => timeout
				->	how would you know diff b/w slow and dead master? => timeout
				->	how will you select a new master?
				->	what will happen when old master resurrect? (split brain problem)
				->	Always use immediate consistent node as master otherwise primary
					key constaint might get voilated.
5.	Implementation of Replication Logs:
		- Statement-based
				-> Breaks at non-deterministic functions like RANDOM() and NOW()
				-> It doesn't ensure Ordering.
		- Write-Ahead Log
				-> Low level at disk layer. Also keeps disk segment.
						DIKKAT=> when version of DB or OS changes. Log will break.
		- Row-based
				-> Hybrid of WAL & statement.(MIDDLE)
				-> Primary key and its row value.
6.	Problem with Replication Lag:
		- Read your own writes:
				-> own User-Profile should hit master for others hit slave.
				-> if whole application is user-editable tab??
						=> redirect all the queries to master till eg. 1 minute of last update of that parameter.
				-> client will keep most recent write timestamp. and will check if that replica is consistent till this TS otherwise it will be rerouted to most recent replica or master.
				-> how will you ensure cross-device read-after-write consistency? => distributed cache(we thought !?)
		- Monotonic Read:
				-> sees data goes back in time.(non-decreasing)
				-> always route client req to same replica by Hashing USER_ID.(may cause HOT SHARD)
		- Consistent Prefix Read:
				-> If some writes happens in some order then anyone reading them will always see them in the same order.
				-> usually happens in sharded DB(partitioning)
				-> we will correct casual dependency writes! ( or write them in the same partition if it is possible)



