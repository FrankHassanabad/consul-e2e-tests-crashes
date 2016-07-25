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

You can run the test batch file at:
[push-data.bat](consul-v0.6.4/soaktest/push-data.bat)

Log files of the crash are located at:

First errors that show up
[consul-firsterrors.log](consul-v0.6.4/soaktest/consul-1/conf/consul-firsterrors.log)

On reboot, database cannot be resized
[consul.log](consul-v0.6.4/soaktest/consul-1/conf/consul.log)


### Potential fix
