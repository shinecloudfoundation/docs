# 安装全节点
### 准备工作
准备一台2核(最好4核)，4G内存，初始硬盘200G(以后每年100G递增)的机器，安装好ubuntu linux 18.04系统（必须，不可以用别的版本）
### 执行脚本
`wget https://raw.githubusercontent.com/shinecloudfoundation/shinecloudnet-deploy/master/mainnet/fullnode/deploy.sh -O deploy.sh && sh deploy.sh myScloudNode 0.01uscds v1.2.0`

# 安装轻节点
### 获取安装程序
下载地址：
`https://github.com/shinecloudfoundation/shinecloudnet-binary/raw/master/shinecloudnet-mainnet/binary/v1.2.0/scloudcli`

### 启动
`nohup ./scloudcli rest-server --chain-id shinecloudnet --trust-node --node [全节点IP]:26657 --laddr tcp://0.0.0.0:1317 > lcd.log 2>&1 &`

> 1. `--chain-id shinecloudnet` 指定接入的网络chain-id
> 2. `--laddr tcp://0.0.0.0:1317` 参数指定绑定当前轻节点服务器的公共IP，`0.0.0.0`代表RPC接口可以被其它机器访问，假如希望RPC接口只能本机程序来访问，需要改成`127.0.0.1`。
> 
**注意事项**

1. 提供的安装程序是在ubuntu 18.04上编译通过的，所以服务器操作系统务必是ubuntu 18.04；
2. 由于轻节点上存有私钥，所以轻节点不能被外网访问；
3. 对于准备废弃轻节点服务器，务必执行删除私钥的操作；
4. 账号的私钥默认存在$HOME/.scloudcli目录下，一旦确定不使用轻节点了，请删除这个目录，删除前，注意备份好私钥/助记词；

# 接口介绍
## 1. 创建新账户
 **接口名称：** `/keys`
 
 **执行示例：**
`curl -X POST "http://{轻节点IP}:1317/keys" -H "accept: application/json" -H "Content-Type: application/json" -d "{ \"name\": \"my-wallet-demo123\", \"password\": \"shinecloudnet\"}"`
>name账户名称，password是密码

 **返回内容：**
```
{
  "name": "my-wallet-demo123",
  "type": "local",
  "address": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5",
  "pub_key": "scloudpub1addwnpepqgsdzltw2z29jz60t5lhecdp9k63e3xaj5g53cp22h2a6twp35gfcne65yz",
  "mnemonic": "stand smart quality casual tunnel husband enable doctor almost foster wink someone"
}
```
>address字段是新账户的地址，mnemonic是新账户的助记词

## 2. 使用助记词导入老账户
**接口名称：** /keys/{name}/recover

**执行示例：**
`curl -X POST "http://{轻节点IP}:1317/keys/my-demo-wallet/recover" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"password\": \"shinecloudnet\",  \"mnemonic\": \"elephant point wonder float face cool blame laundry fault moon twist muffin\”}"`
>password是密码，mnemonic是你自己的助记词

**返回内容：**
```
{
  "name": "my-demo-wallet",
  "type": "local",
  "address": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5",
  "pub_key": "scloudpub1addwnpepqdj3umzkzq3fl4rfm7vfhrzkylhq6evvp7z6avel9lxzp4xa2e8d5zt8m69"
}
```
>name账户名称
>address是账户地址

## 3. 获取账号信息接口
**接口名称：** /auth/accounts/{address}

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/auth/accounts/scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5" -H  "accept: application/json"`

**返回内容：**
```
{
  "height": "329547",
  "result": {
    "type": "cosmos-sdk/Account",
    "value": {
      "address": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5",
      "coins": [
        {
          "denom": "uscds",
          "amount": “1000000000" 
        }
      ],
      "public_key": null,
      "account_number": "55",
      "sequence": "0"
    }
  }
}
```
>coins是个数组结构，我们只需要denom=“uscds”的数据；

>amount是当前账户的余额，需要除以```10^6```，才是展示给用户看到的SCDS数量；

>account_number 与 sequence 参数是留给转账接口使用的，所以在每次转账之前，都需要调用这个接口获取这两个参数，然后再调用转账接口。

## 4. 转账接口
**接口名称：** /bank/accounts/{address}/transfers

**执行示例：**
`curl -X POST "http://{轻节点IP}:1317/bank/accounts/scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5/transfers" -H  "accept: application/json" -H  "Content-Type: application/json" -d "{  \"base_req\": {    \"from\": \"scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860\",    \"password\": \"shinecloudnet\",    \"memo\": \"first transfer to a account\",    \"chain_id\": \"shinecloudnet\",    \"account_number\": \"55\",    \"sequence\": \"0\",    \"gas\": \"100000\",    \"gas_adjustment\": \"1.2\",    \"fees\": [      {        \"denom\": \"uscds\",        \"amount\": \"1000\"      }    ],    \"broadcast_mode\": \"sync\",    \"simulate\": false  },  \"amount\": [    {      \"denom\": \"uscds\",      \"amount\": \"22000000\"    }  ]}"`
>address：收款方地址；

>account_number：账户编号，一旦创建了账户，这个值是不变的；

>sequence：每次转账都需要从/auth/accounts/{address}接口获取最新的值；

>from：发送方地址；

>broadcast_mode：转账模式，可选的参数sync（阻塞），async（不阻塞）；

>amount：转账数量，SCDS公链的所有数值，通过接口上链的时候，需要在1个SCDS单位的基础上乘以 ```10^6```, 从接口中拿到的数据也要除以 ```10^6``` 后展示给用户，比如你想转20个SCDS，则amount的值如下
```math
20 * 10^{6} = 20,000,000
```


**返回内容：**
```
{
  "result": {
    "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
    "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
    "height": "0",
    "logs": [
      {
        "success": true,
        "log": "",
        "msg_index": 0
      }
    ]
  },
  "height" : "0"
}
```
> success字段表明交易成功提交到网络，不能通过这个字段来判断交易是否成功；

> txhash是交易的hash值，需要通过/txs{hash}接口来查询交易的详细信息；

## 5. 根据转账返回的hash值，查询单条交易
**接口名称：** `/txs/{hash}`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs/9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0" -H  "accept: application/json"`

**返回内容：**
```
{
  "height": "329655",
  "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
  "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
  "logs": [
    {
      "msg_index": 0,
      "success": true,
      "log": ""
    }
  ],
  "gas_wanted": "100000",
  "gas_used": "39320",
  "events": [
    {
      "type": "message",
      "attributes": [
        {
          "key": "action",
          "value": "send"
        },
        {
          "key": "sender",
          "value": "scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
        },
        {
          "key": "module",
          "value": "bank"
        }
      ]
    },
    {
      "type": "transfer",
      "attributes": [
        {
          "key": "recipient",
          "value": "scloud1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug"
        },
        {
          "key": "amount",
          "value": "22000000uscds"
        }
      ]
    }
  ],
  "tx": {
    "type": "cosmos-sdk/StdTx",
    "value": {
      "msg": [
        {
          "type": "cosmos-sdk/MsgSend",
          "value": {
            "from_address": "scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860",
            "to_address": "scloud1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug",
            "amount": [
              {
                "denom": "uscds",
                "amount": "22000000"
              }
            ]
          }
        }
      ],
      "fee": {
        "amount": [
          {
            "denom": "uscds",
            "amount": "1000"
          }
        ],
        "gas": "100000"
      },
      "signatures": [
        {
          "pub_key": {
            "type": "tendermint/PubKeySecp256k1",
            "value": "A2UebFYQIp/Uad+Ym4xWJ+4NZYwPha6zPy/MINTdVk7a"
          },
          "signature": "FntwNT0aVDp+fqGXThmjplWdki5s5JyNhcdIsnnfJ5IQaoxrF398wWsKOwAWcFeGzGxtbiQdhGPXBc6wPhRcXg=="
        }
      ],
      "memo": "first transfer to a account"
    }
  },
  "timestamp": "2019-11-26T09:00:25Z"
}
```
>success：转账是否成功；

>from_address：发送方地址；

>to_address：接收方地址；

>tx->value->msg->value-amount->amount：交易的金额，需要除以```10^6```后展示给客户；

>memo：交易备注，通过这个来区分用户；

>timestamp：交易发生的时间，注意这是0时区的时间，可以根据客户端的业务需求格式化为东8区；
>

## 6. 根据发送方地址，查询交易：
**接口名称：** /txs

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs?message.action=send&message.sender=scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5&page=1&limit=10" -H  "accept: application/json"`

**返回内容：**
```
{
  "total_count": "1",
  "count": "1",
  "page_number": "1",
  "page_total": "1",
  "limit": "10",
  "txs": [
    {
      "height": "329655",
      "txhash": "9CE5BE6A700A6199811204B473188BE94089963D5CBEB39E3493F92B87E3CCA0",
      "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
      "logs": [
        {
          "msg_index": 0,
          "success": true,
          "log": ""
        }
      ],
      "gas_wanted": "100000",
      "gas_used": "39320",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "send"
            },
            {
              "key": "sender",
              "value": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5"
            },
            {
              "key": "module",
              "value": "bank"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "scloud1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug"
            },
            {
              "key": "amount",
              "value": "22000000uscds"
            }
          ]
        }
      ],
      "tx": {
        "type": "cosmos-sdk/StdTx",
        "value": {
          "msg": [
            {
              "type": "cosmos-sdk/MsgSend",
              "value": {
                "from_address": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5",
                "to_address": "scloud1csl7cm802trp2lqdky3ngpl9xka8av7y28s9ug",
                "amount": [
                  {
                    "denom": "uscds",
                    "amount": "22000000"
                  }
                ]
              }
            }
          ],
          "fee": {
            "amount": [
              {
                "denom": "uscds",
                "amount": "1000"
              }
            ],
            "gas": "100000"
          },
          "signatures": [
            {
              "pub_key": {
                "type": "tendermint/PubKeySecp256k1",
                "value": "A2UebFYQIp/Uad+Ym4xWJ+4NZYwPha6zPy/MINTdVk7a"
              },
              "signature": "FntwNT0aVDp+fqGXThmjplWdki5s5JyNhcdIsnnfJ5IQaoxrF398wWsKOwAWcFeGzGxtbiQdhGPXBc6wPhRcXg=="
            }
          ],
          "memo": "first transfer to a account"
        }
      },
      "timestamp": "2019-11-26T09:00:25Z"
    }
  ]
}
```
>需要关注的字段与`/txs/{hash}`接口相同

## 7. 根据接收方地址，查询交易：
**接口名称：** `/txs`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/txs?message.action=send&transfer.recipient=scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860&page=1&limit=10" -H  "accept: application/json"`

**输出内容：**
```
{
  "page_total": "1",
  "page_number": "1",
  "txs": [
    {
      "height": "328473",
      "txhash": "56CEBE1522F737F0A5F2AF1EF322A282C2ACA231C70F466A33FE31356B812C2B",
      "logs": [
        {
          "success": true,
          "log": "",
          "msg_index": 0
        }
      ],
      "gas_used": "42633",
      "events": [
        {
          "type": "message",
          "attributes": [
            {
              "key": "action",
              "value": "send"
            },
            {
              "key": "sender",
              "value": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5"
            },
            {
              "key": "module",
              "value": "bank"
            }
          ]
        },
        {
          "type": "transfer",
          "attributes": [
            {
              "key": "recipient",
              "value": "scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
            },
            {
              "key": "amount",
              "value": "1000000000uscds"
            }
          ]
        }
      ],
      "gas_wanted": "200000",
      "tx": {
        "type": "cosmos-sdk\/StdTx",
        "value": {
          "signatures": [
            {
              "pub_key": {
                "type": "tendermint\/PubKeySecp256k1",
                "value": "A0uAEtIIQHLNw0Hqed9fRc8lhGo7oBnTu1WJKzSBp2rg"
              },
              "signature": "VR4r9nK2i9pa\/\/zMhuEcWJejBZZz0BjOnEeZzUhxfAMHrPpgi37fftbNaydGmfANDP3QaITDr9NHvkClgd4zEA=="
            }
          ],
          "memo": "Sent token from light node",
          "msg": [
            {
              "type": "cosmos-sdk\/MsgSend",
              "value": {
                "from_address": "scloud1hrst75w8mkpfm55wn6xcwjl7usknhv2gvzsjh5",
                "amount": [
                  {
                    "denom": "uscds",
                    "amount": "1000000000"
                  }
                ],
                "to_address": "scloud1ufpp68vql0jd58anvlau97cs7ck4lxj58gn860"
              }
            }
          ],
          "fee": {
            "amount": [
              {
                "denom": "uscds",
                "amount": "1000000"
              }
            ],
            "gas": "200000"
          }
        }
      },
      "timestamp": "2019-11-26T07:20:30Z",
      "raw_log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]"
    }
  ],
  "total_count": "1",
  "count": "1",
  "limit": "10"
}
```
>需要关注的字段与`/txs/{hash}`接口相同
>

## 8. 获取最新块信息
**接口名称：** `/blocks_results/latest`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/blocks_results/latest" -H  "accept: application/json"`
>解释信息见下一条
>

## 9. 根据块高度，查询块信息
**接口名称：** `/blocks_results/{height}`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317/blocks_results/1000" -H  "accept: application/json"`

**返回内容：**
```
{
  "height": "5339054",
  "results": {
    "deliver_tx": [
      {
        "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
        "gas_wanted": "100000",
        "gas_used": "40569",
        "events": [
          {
            "type": "message",
            "attributes": [
              {
                "key": "action",
                "value": "send"
              },
              {
                "key": "sender",
                "value": "scloud1qg7xe8azyhvjkvxnh9seqqcp2wvm2qk0zr7ej0"
              },
              {
                "key": "module",
                "value": "bank"
              }
            ]
          },
          {
            "type": "transfer",
            "attributes": [
              {
                "key": "recipient",
                "value": "scloud1eg83rfelg0auqrxkncesjtk05zqf7l3mwxpv7y"
              },
              {
                "key": "amount",
                "value": "49000000uscds"
              }
            ]
          }
        ]
      }
    ],
    "end_block": {
    },
    "begin_block": {
    }
  }
}
```
>deliver_tx字段是一个数组字段，包含了当前block里的所有tx记录

## 10. 给定高度值，查询整个区块的的所有数据：
**接口名称：** `/blocks_results/{height}`

**执行示例：**
`curl -X GET "http://{轻节点IP}:1317//blocks_results/12165" -H  "accept: application/json"`

**输出内容：**
```
{
  "height": "12165",
  "results": {
    "deliver_tx": [
      {
        "log": "[{\"msg_index\":0,\"success\":true,\"log\":\"\"}]",
        "gas_wanted": "100000",
        "gas_used": "41817",
        "events": [
          {
            "type": "message",
            "attributes": [
              {
                "key": "action",
                "value": "send"
              },
              {
                "key": "sender",
                "value": "scloud1r35exs2t0jt5u9g9t2xt6k4kdyzvrwmy4ehqkl"
              },
              {
                "key": "module",
                "value": "bank"
              }
            ]
          },
          {
            "type": "transfer",
            "attributes": [
              {
                "key": "recipient",
                "value": "scloud1j3ly90f6rm6x0xv8rh5g425745qqa5lwkwdgxf"
              },
              {
                "key": "amount",
                "value": "100000000uscds"
              }
            ]
          }
        ]
      }
    ],
    ......
  }
}
```
> a. 输出内容太多，删除了对集成没有价值的信息，目前这个结构里，只需要关心deliver_tx数组里面的记录，每个数组项是一个交易记录，然后再查看events里的数据，通过type=message 与 type=transfer来判定转出方、转入方、转账数额；

> b. 在应用层，通过一个定时程序，区块链网络大约每5.5秒出一个新块，所以应用程序可以每5.5秒调用一下这个接口，分析好数据，在应用层重建一个交易数据库。然后在应用层程序里，可以通过查看这个交易数据库，来判断一个交易是否成功；

# 三. 如何集成
>主要说明如何集成充值与提币，建议通过tx的memo字段来区分不同的用户，所以需要在系统层面为每个客户设置一个唯一的ID标识，一般用数字来表示

## 1.充值
> a. 在充值页面需要给用户展示官方钱包的充值地址与需要用户填入的memo内容。系统需要查询自己地址的转入交易，然后通过查看tx的memo字段来确定是哪一位用户执行了充币，在区块链交易量非常大的情况，这个方法可能会存在遗漏交易的可能，最好采用第二种；

> b. 采用第8个接口重建本地交易数据库的方式，重建本地交易数据库的方式的好处就是，一定不会遗漏交易，效率更高；
&nbsp; &nbsp; 
## 2.提币
在提币页面需要让用户输入提币地址与memo（假如客户从当前交易所提到别的交易所，则需要填写memo字段；若是提到自己的去中心钱包，则不需要填写memo字段）。

#### 提币需要用到转账接口，转账的基本调用逻辑：
> a. 先调用获取账号信息的接口，提取account_number与sequence参数；

> b. 把上面两个参数放到转账接口中，编排好接口的body参数的json结构，然后执行；
