```
redis-cli -h 10.8.169.176 keys "kline*" | xargs redis-cli -h 10.8.169.176 del
```
