# consul-e2e-tests-crashes

Very simple (as simple as I can make) consul end to end tests on various enviornments and any crashes associated with those tests.
Results and log files are listed here for bug tickets written against Consul.


## Test 1

Running consul bootstrap mode on Windows 2000 R2 (w2kr2) with a simple curl script which puts random strings into the K/V data store over
long periods of time will cause consul to begin giving the errors:

```
2016/07/22 21:08:06 [ERR] raft: Failed to commit logs: file resize error: truncate C:\Program Files\Logrhythm\Data Indexer\consul\data\raft\raft.db: The requested operation cannot be performed on a file with a user-mapped section open.
2016/07/22 21:08:06 [INFO] raft: Node at 192.168.253.10:8303 [Follower] entering Follower state
2016/07/22 21:08:06 [ERR] consul.kvs: Apply failed: file resize error: truncate C:\Program Files\Logrhythm\Data Indexer\consul\data\raft\raft.db: The requested operation cannot be performed on a file with a user-mapped section open.
```

On reboot, your consul instance will not be operational again until you delete the file of:
```
consul\data\raft\raft.d
```

The test is simply a non-ending loop of:
```
curl -X PUT -d '!_RndAlphaNum!' http://localhost:8500/v1/kv/stress/random/!_RndAlphaNum!
```

You can see and run the test batch file at:

[consul-v0.6.4/soaktest/push-data.bat](consul-v0.6.4/soaktest/push-data.bat)

I ran the above batch file in ~10 simultaneous instances over the period of 5 hours with once shutting down the service overnight.

First errors that show up:

[consul-v0.6.4/soaktest/crashed-instances/consul-1/consul-firsterrors.log](consul-v0.6.4/soaktest/crashed-instances/consul-1/consul-firsterrors.log?raw=true)

On reboot, database cannot be resized and consul cannot elect a leader.

[consul-v0.6.4/soaktest/crashed-instances/consul-1/consul.log](consul-v0.6.4/soaktest/crashed-instances/consul-1/consul.log?raw=true)


### Potential fix

The root of the issue has been reported here to consul:

[https://github.com/hashicorp/consul/issues/2203](https://github.com/hashicorp/consul/issues/2203)

And it appears to be from the 3rd party dependency boltdb:
[https://github.com/boltdb/bolt/issues/504](https://github.com/boltdb/bolt/issues/504)

boltdb has been fixed and patched.  I performed an upgrade of consul v0.6.4 with that dependency here:
[https://github.com/FrankHassanabad/consul/tree/v0.6.2-boltdb-upgrade](https://github.com/FrankHassanabad/consul/tree/v0.6.2-boltdb-upgrade)

I re-ran the same tests and consul was able to soak test for 12 hours and did not report resize issues.
