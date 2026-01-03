## Requirements
You have to install the following packages:

[getopt](https://linux.die.net/man/3/getopt)<br>
[jq](https://stedolan.github.io/jq/)<br>
[git](https://git-scm.com/)<br>
[tput](https://command-not-found.com/tput)<br>
[bc](https://www.gnu.org/software/bc/)<br>
[curl](https://curl.se/download.html)<br>
[parallel (version > 20220515)](https://www.gnu.org/software/parallel/)<br>
[shuf](https://www.gnu.org/software/coreutils/)

## How to run
### 1. clone

```shell
[~]>$ git clone https://github.com/MortezaBashsiz/CFScanner.git
```

### 2. Change directory and make them executable

```shell
[~]>$ cd CFScanner/bash
[~/CFScanner/bash]> chmod +x ../bin/*
```
In the config file the variables are
```shell
{
	"id": "User's UUID",
	"Host": "Host address which is behind Cloudflare",
	"Port": "Port which you are using behind Cloudflare on your origin server",
	"path": "Websocket endpoint like api20",
	"serverName": "SNI",
  "subnetsList": "https://raw.githubusercontent.com/MortezaBashsiz/CFScanner/main/config/cf.local.iplist"
}
```

### 3. Execute it

You have following switches to define the arguments 

-k: XRAY or V2RAY, you can define which core you want to use

-x: Custom xray/v2ray client config template (see "Custom config template (-x)" below)

-v: YES or NO, you are able to define for script to test with your vmess or not

-m: SUBNET or IP, Choose one of them for scanning subnets or single IPs

-t: DOWN or UP or BOTH, Choos one of them for download and upload test

-o: Port to scan when VPN mode is NO (default 443)

-p: This is an integer number that defines the parallel threads count

-n: This is an integer to define how many times you like to check an IP

-s: This is the filter that you can define to list the IPs based on download speed. The value is in KBPS (Kilo Bytes Per Second). For example, if you set it to 50, it means that you will only list the IPs which have a download speed of more than 50 KB/S.

-f: This is an optional argument which is a file address if you want to execute only some specific subnets. Then put your subnets in a file and pass the file as an argument to the command.

-L: Comma-separated subnet list for SUBNET mode. Keeps the given order.

-R: YES or NO to disable remote subnet download (use local config/cf.local.iplist instead).

-S: YES or NO to auto-skip subnet ranges after 5 working IPs OR 10% scanned OR 3 minutes.

-r: This is an integer that specifies randomness. Instead of testing all IPs in a subnet, a random sample of size ``d`` will be tested.

-d: This is the threshold to download succeed count. With this option you can filter to show you only the IPs which have successfully download count more than or equal the amount you specified. This will be AND with -u

-u: This is the threshold to upload succeed count. With this option you can filter to show you only the IPs which have successfully upload count more than or equal the amount you specified. This will be AND with -d

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -k <XRAY/V2RAY> -x <custom-config-file> -v <YES/NO> -m <SUBNET/IP> -t <DOWN/UP/BOTH> -o <port> -p <int> -n <int> -r <int> -s <int> -d <int> -u <int> -f <Custome Subnet File> -L <subnet list> -R <YES/NO> -S <YES/NO>
```

Notes for using `-x`:
- `-x` only affects VPN mode. Use it with `-v YES` (with `-v NO` the custom config is ignored).
- `-x` is required when you run with `-v YES`.
Notes for using `-o`:
- `-o` is only available with `-v NO`.

### Custom config template (-x)

The custom config file is a full xray/v2ray client config. The script replaces these placeholders:

- `IP.IP.IP.IP` → each candidate Cloudflare IP (outbound)
- `PORTPORT` → local SOCKS inbound port (inbound)

Because of these placeholders, the template is not valid JSON until the script replaces them.

Sample template: [`bash/custom-config.sample.json`](custom-config.sample.json)

#### EXAMPLE: Use a custom config template

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -k XRAY -x custom-config.sample.json -m SUBNET -t DOWN -p 8 -n 1 -s 100
```
#### EXAMPLE: Download test without custom subnet

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t DOWN -p 8 -n 1 -s 100
```

#### EXAMPLE: Upload test without custom subnet

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t UP -p 8 -n 1 -s 100
```

#### EXAMPLE: Upload and Download test without custom subnet

##### in Linux:

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 1 -s 100
```

#### EXAMPLE: Use your custom subnet file

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 1 -s 100 -f custom.subnets
```

#### EXAMPLE: Use a custom subnet list (ordered)

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 1 -s 100 -L "5.226.179.0/24,203.89.5.0/24"
```

#### EXAMPLE: Disable remote subnet download and enable auto-skip

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 1 -s 100 -R YES -S YES
```

Which the `custom.subnets` is like as follows. You can edit this file and add your subnets in each line.

```shell
[~/CFScanner/bash]>$ cat custom.subnets 
5.226.179.0/24
203.89.5.0/24
[~/CFScanner/bash]>$
```

#### EXAMPLE: Use random count in each subnet

In this example script will select only 5 random IPs from each subnet.

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 1 -s 100 -r 5 -f custom.subnets
```

#### EXAMPLE: Use upload and download threshold

In this example script will try 5 time for each IP and the IPs which have more than or equal 5 successful download AND more than or equal 3 successful upload will be select as OK

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m SUBNET -t BOTH -p 8 -n 8 -s 100 -d 5 -u 3 -f custom.subnets
```

#### EXAMPLE: Use your custom ip file

```shell
[~/CFScanner/bash]>$ bash cfScanner.sh -v YES -m IP -t BOTH -p 8 -n 1 -s 100 -f ip.list
```

Which the `ip.list` is like as follows. You can edit this file and add your IPs in each line.

```shell
[~/CFScanner/bash]>$ cat ip.list
23.227.37.250 
23.227.37.252 
23.227.37.253 
23.227.37.254 
23.227.37.255 
23.227.38.1 
23.227.38.8 
23.227.38.2 
23.227.38.3 
23.227.38.6 
23.227.38.14 
23.227.38.11 
23.227.38.9 
23.227.38.4 
23.227.38.10 
23.227.38.7 
[~/CFScanner/bash]>$
```

### 4. Result

It will generate a file in datetime format in the result directory.

```shell
[~/CFScanner/bash]>$ ls result/
20230120-203358-result.cf
[~/CFScanner/bash]>$
```
## Video Guide
You can find a video guide for this script on [youtube](https://youtu.be/BKLRAHolhvM "youtube") and [youtube](https://youtu.be/4xJvWYdGuV8 "youtube").
