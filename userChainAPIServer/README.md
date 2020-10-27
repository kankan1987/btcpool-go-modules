# User Chain API Server
由两个模块合并而来，请看具体子模块的介绍（代码已重构，文档需要更新）：
* [Switcher API Server](SwitcherAPIServer.md)
  提供触发 Stratum 切换的API
* [Init User Coin](InitUserCoin.md)
  初始化zookeeper里的用户币种记录

## 构建
```
go get -u github.com/btccom/btcpool-go-modules/userChainAPIServer
```

## 运行
```
cp config.default.json config.json
$GOPATH/bin/userChainAPIServer --config config.json --logtostderr -v 2
```

# Docker

## 构建
```
cd btcpool-go-modules/userChainAPIServer
docker build -t btcpool-user-chain-api-server -f Dockerfile ..
```

## 运行
```
docker run -it --rm --network=host \
  -e AvailableCoins='ubtc,btc,bcc,auto' \
  -e UserListAPI_ubtc='http://localhost:58080/userlist-ubtc.php' \
  -e UserListAPI_btc='http://localhost:58080/userlist-btc.php' \
  -e UserListAPI_bcc='http://localhost:58080/userlist-bch.php' \
  -e ZKBroker='10.0.1.176:2181,10.0.1.175:2181,10.0.1.174:2181' \
  -e ZKSwitcherWatchDir='/stratumSwitcher/btcbcc/' \
  -e EnableAPIServer='true' \
  -e APIUser='switchapi' \
  -e APIPassword='admin' \
  -e ListenAddr='0.0.0.0:8080' \
  -e UserCoinMapURL='http://localhost:58080/usercoin.php' \
  -e UserSubPoolMapURL='http://localhost:58080/usersubpool.php' \
  -e StratumServerCaseInsensitive='true' \
  btcpool-user-chain-api-server:latest -logtostderr -v 2

# 守护进程
docker run -it --name user-chain-api-server --network=host --restart always -d \
  -e AvailableCoins='ubtc,btc,bcc,auto' \
  -e UserListAPI_ubtc='http://localhost:58080/userlist-ubtc.php' \
  -e UserListAPI_btc='http://localhost:58080/userlist-btc.php' \
  -e UserListAPI_bcc='http://localhost:58080/userlist-bch.php' \
  -e ZKBroker='10.0.1.176:2181,10.0.1.175:2181,10.0.1.174:2181' \
  -e ZKSwitcherWatchDir='/stratumSwitcher/btcbcc/' \
  -e EnableAPIServer='true' \
  -e APIUser='switchapi' \
  -e APIPassword='admin' \
  -e ListenAddr='0.0.0.0:8080' \
  -e UserCoinMapURL='http://localhost:58080/usercoin.php' \
  -e UserSubPoolMapURL='http://localhost:58080/usersubpool.php' \
  -e StratumServerCaseInsensitive='true' \
  btcpool-user-chain-api-server:latest -logtostderr -v 2
```

如果需要自动注册功能，可使用如下配置：

```
docker run -it --name user-chain-api-server --network=host --restart always -d \
  -e AvailableCoins='ubtc,btc,bcc,auto' \
  -e UserListAPI_ubtc='http://localhost:58080/userlist-ubtc.php' \
  -e UserListAPI_btc='http://localhost:8000/userlist-autoreg.php' \
  -e UserListAPI_bcc='http://localhost:58080/userlist-bch.php' \
  -e ZKBroker='10.0.1.176:2181,10.0.1.175:2181,10.0.1.174:2181' \
  -e ZKSwitcherWatchDir='/stratumSwitcher/btcbcc/' \
  -e EnableAPIServer='true' \
  -e APIUser='switchapi' \
  -e APIPassword='admin' \
  -e ListenAddr='0.0.0.0:8080' \
  -e UserCoinMapURL='http://localhost:58080/usercoin.php' \
  -e UserSubPoolMapURL='http://localhost:58080/usersubpool.php' \
  -e StratumServerCaseInsensitive='true' \
  -e EnableUserAutoReg="true" \
  -e ZKAutoRegWatchDir="/stratumSwitcher/btcbcc_autoreg/" \
  -e UserAutoRegAPI_IntervalSeconds=10 \
  -e UserAutoRegAPI_URL="http://localhost:8000/autoreg.php" \
  -e UserAutoRegAPI_User="" \
  -e UserAutoRegAPI_Password="" \
  -e UserAutoRegAPI_DefaultCoin="btc" \
  -e UserAutoRegAPI_PostData='{"sub_name": "{sub_name}", "region_name": "all", "currency": "btc"}' \
  btcpool-user-chain-api-server:latest -logtostderr -v 2
```

### 参数说明

* 如没有`UserCoinMapURL`，或者只有一个币种，请将其留空，这样就不会启动拉取`UserCoinMapURL`的任务。注意：用于开关该功能的`EnableCronJob`选项已废弃。
* 如没有`UserSubPoolMapURL`，或者只有一个币种，请将其留空，这样就不会启动拉取`UserSubPoolMapURL`的任务。
* 币种`auto`可选，用于机枪切换，不需要实际配置到`sserver`的`chains`里。`sserver`只需要打开机枪切换功能（`auto_switch_chain`）即可识别币种`auto`。