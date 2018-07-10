# CPU

## 查看机器CPU信息

- 查看CPU信息(型号)

```shell
-> cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
->      8  Intel(R) Xeon(R) CPU E5-2650 v2 @ 2.60GHz
```

- 查看物理CPU个数

```
cat /proc/cpuinfo | grep "physical id" | sort | uniq | wc -l
2
```

- 查看每个物理CPU中core的个数(核数)

```shell
cat /proc/cpuinfo | grep "cpu cores" | uniq
```

- 查看逻辑CPU个数

```shell
cat /proc/cpuinfo | grep "processor" | wc -l 
```



## 参考

[参考](https://www.cnblogs.com/bugutian/p/6138880.html)

