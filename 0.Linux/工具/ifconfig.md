- 无参数，查看所有激活的接口的状态

  ```shell
  $ ifconfig
  ens33: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
          inet 192.168.0.80  netmask 255.255.255.0  broadcast 192.168.0.255
          ether 00:0c:29:56:7c:17  txqueuelen 1000  (以太网)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
          device interrupt 19  base 0x2000

  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          inet6 ::1  prefixlen 128  scopeid 0x10<host>
          loop  txqueuelen 1000  (本地环回)
          RX packets 6522  bytes 470549 (470.5 KB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 6522  bytes 470549 (470.5 KB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  ```

- `interface`，查看指定接口状态

    ```shell
    $ ifconfig ens33
    ens33: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
            ether 00:0c:29:56:7c:17  txqueuelen 1000  (以太网)
            RX packets 0  bytes 0 (0.0 B)
            RX errors 0  dropped 0  overruns 0  frame 0
            TX packets 0  bytes 0 (0.0 B)
            TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
            device interrupt 19  base 0x2000
    ```

- `-a`，显示所有接口，即使被关闭

- `s`，简化输出的信息

- `up`，激活接口

- `down`，关闭接口

- `mut N`，设置接口的 MUT

- `addr`，接口配置静态 ip

  ```shell
  $ sudo ifconfig ens33 192.168.0.1
  $ ifconfig ens33
  ens33: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
          inet 192.168.0.1  netmask 255.255.255.0  broadcast 192.168.0.255
          ether 00:0c:29:56:7c:17  txqueuelen 1000  (以太网)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
          device interrupt 19  base 0x2000
  ```

- `name newname`，修改接口名
