# 压力测试

1. 压力测试50并发

```bat
redis-benchmark.exe -h 127.0.0.1 -p 6379 -a 123456 -c 50 -n 1000
pause
```

2. 压力测试100并发

```bat
redis-benchmark.exe -h 127.0.0.1 -p 6379 -a 123456 -c 100 -n 1000
pause
```

3. 压力测试500并发

```bat
redis-benchmark.exe -h 127.0.0.1 -p 6379 -a 123456 -c 500 -n 1000
pause
```

4. 压力测试1000并发

```bat
redis-benchmark.exe -h 127.0.0.1 -p 6379 -a 123456 -c 1000 -n 1000
pause
```
