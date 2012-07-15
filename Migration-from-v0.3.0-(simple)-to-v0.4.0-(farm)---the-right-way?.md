## FAQ

1. Can a node v0.4.0 join a cluster v0.3.0?
No. join(440) joining node sent a message with the wrong protocol version
2. Can sheep start after update from v0.3.0 to v0.4.0?
Yes, but all vdi's are gone. First convert simple to farm!
3. Is online migration possible?
No. All nodes must be offline.

## HowTo Migration from v0.3.0 (simple) to v0.4.0 (farm)

1. shutdown all nodes v.0.3.0 and update to v0.4.0
2. Convert on all nodes from simple to farm with script simple2farm
3. Startup all nodes with v0.4.0