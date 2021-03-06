# 骰宝

## 概述
骰宝是由各閒家向庄家下注的游戏。每次下注前，庄家先把三颗骰子放在有盖的器皿内摇晃，待閒家们下注完毕，庄家便打开器皿并派彩。最常见的玩法是买骰子点数的大小（总点数为3至10称作小，11至18为大，围股庄家通杀）

## 公平
游戏採用EOS区块链技术，由庄家生成三个随机数，并採用哈希算法开奖与派彩，所有人都能在每局结束后验证本局游戏的公平性。由于区块链技术与哈希算法两者任何人都不可逆转，所以能保证游戏结果不为任何人所控制。

## 算法
### 简述
庄家开盘前会生成一串随机数（32字节）并保密，同时公开展示该数的哈希值作为签名。签名本局游戏中不会变，以保证庄家不会作弊，随后玩家开始下注。开奖时庄家提供该随机数给游戏，游戏通过签名验证该随机数，并使用随机数生成三个骰子，然后自动派奖。玩家可以事后验证该随机数与签名公正性。

### 伪代码

```
if (签名 == sha256(随机数[32])) {
    骰子1 = (随机数[0] + 随机数[1] + 随机数[2] + 随机数[3] + 随机数[4] + 随机数[5]) % 6 + 1  
    骰子2 = (随机数[6] + 随机数[7] + 随机数[8] + 随机数[9] + 随机数[10]+ 随机数[11])% 6 + 1  
    骰子3 = (随机数[12]+ 随机数[13]+ 随机数[14]+ 随机数[15]+ 随机数[16]+ 随机数[17])% 6 + 1
} else {
    验证不过，游戏出错
}
```

### 示例

玩家理解算法需要一些C语言的基础知识，数值均为十六进制。

在游戏开奖前，玩家可以获取庄家事先展示的签名，本局假设签名为：
763f42ad937c1a268a17dfbdbb3b8e8bce12bc7cef5ac94f3ab3110ee78f52ca
在游戏结束后，庄家会展示该签名所对应的哈希原值（随机数），原值如下：
c6793186e4a0c748118a5070faf7a4ffdd07ea9d07d9f6ca25c4d59b88c6e58a
套用上方公式可以计算得出

```
commitment = 763f42ad937c1a268a17dfbdbb3b8e8bce12bc7cef5ac94f3ab3110ee78f52ca
source = c6793186e4a0c748118a5070faf7a4ffdd07ea9d07d9f6ca25c4d59b88c6e58a
if (commitment == sha256(source)) {
    3 = (c6 + 79 + 31 + 86 + e4 + a0) % 6 + 1
    1 = (c7 + 48 + 11 + 8a + 50 + 70) % 6 + 1
    5 = (fa + f7 + a4 + ff + dd + 07) % 6 + 1
}
```
3, 1, 5便是本次开奖结果

### 第三方验证
查询一些公示信息可以通过任何第三方网站，比如：[https://bloks.io/](https://bloks.io/)

签名，开奖前就能查询：  
search - > 庄家名 -> Account Transactions -> Contract -> Tables -> Step 2 - Select Table -> game -> Load Table -> 当前游戏编号 -> commitment  
原值，开奖后庄家公开：  
search - > 庄家名 -> Account Transactions -> Contract -> Tables -> Step 2 - Select Table -> game -> Load Table -> 当前游戏编号 -> source

计算sha256，这裡以linux操作系统为例。注意，sha256的计算这裡是整形数值，而不是字符串：  

```bash
>> echo -n 'c6793186e4a0c748118a5070faf7a4ffdd07ea9d07d9f6ca25c4d59b88c6e58a' | xxd -r -p | sha256sum -b | awk '{print $1}'

>> 763f42ad937c1a268a17dfbdbb3b8e8bce12bc7cef5ac94f3ab3110ee78f52ca
```


