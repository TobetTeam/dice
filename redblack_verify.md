# Big.game 红黑大战公平性验证说明

验证工具网页请访问：https://big.game/redblack_verify

在使用此网页工具之前，请仔细阅读以下说明。您可以按照如下步骤，自行开发程序验证。

## 一、解析下注记录的memo
在区块链浏览器（如eosflare：https://eosflare.io/ ）查询您下注的转账记录，查看其memo信息。

memo信息是以”-“字符将信息连接起来的字符串,类似“16728-0-1-0-2-5000-3-0-SIG_K1_KddLmqy6kt3VgPw1J1XuvAoerKrLVWtonEkbswb4skMVjyMmpwxipJyTqm46YqNz8KjdP4EtB946j5rZHNfGBMMEuPrntp”:
![eosbiggame55 - transfer截图](https://github.com/biggamerobot/dice/blob/master/red_memo.png) 

其结构如下：
游戏ID-推荐人账号-1-黑方投注金额-2-红方投注金额-3-幸运一击投注金额-本局种子签名

* 第一个信息：游戏ID；
* 第二个信息：推荐人账号，若玩家不存在推荐人则显示为0；
* 第三个信息：1指黑方投注区域；
* 第四个信息：黑方区域的投注金额，若未投注则显示0，若投注则可用数字除以10000即可得到相应的EOS投注金额，比如第四个信息显示5000指的是玩家在黑方投注了0.5 EOS；
* 第五个信息：2指红方投注区域；
* 第六个信息：红方区域的投注金额；
* 第七个信息：3指幸运一击投注区域；
* 第八个信息：幸运一击投注金额；
* 第九个信息：本局种子签名。

## 二、查询开奖记录
在 https://eosflare.io/ 自己账号的记录中找到类型为“eosbiggame55 - reveal”的交易记录，只要“game_id”与自己投注记录memo中的第一个信息相同，即为对应的开奖记录，如下图所示： 

![eosbiggame55 - reveal截图](https://github.com/biggamerobot/dice/blob/master/red_reveal_memo.png) 

## 三、验证签名

验证签名需要使用biggame的公钥“public_key”对开奖记录中的种子seed及其签名“seed_sign”进行ecc签名验证，验证结果通过并且签名等同于memo中的本局种子签名即可证明服务器种子未被篡改。


## 四、计算开奖结果
使用开奖记录的种子“seed”进行 SHA256 运算，得到 A = sha256(seed)（32位长度的uint8数组），使用该数组即可完成发牌。

牌组的排序及相关值如下所示（52位长度的一维数组）：
* 0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08,0x09,0x0A,0x0B,0x0C,0x0D, //方块 A - K
* 0x11,0x12,0x13,0x14,0x15,0x16,0x17,0x18,0x19,0x1A,0x1B,0x1C,0x1D, //梅花 A - K
* 0x21,0x22,0x23,0x24,0x25,0x26,0x27,0x28,0x29,0x2A,0x2B,0x2C,0x2D, //红桃 A - K
* 0x31,0x32,0x33,0x34,0x35,0x36,0x37,0x38,0x39,0x3A,0x3B,0x3C,0x3D  //黑桃 A - K

发牌逻辑，总共需要发六张牌，每张牌在牌组中的位置按照如下公式计算得出，算出结果后将该牌从牌组中移出以便计算下一张牌的位置：

(A[0]*17+A[1]*13+A[2]*11)%剩余牌的张数(52) 作为黑方第一张牌位置，从牌组中取出。

(A[5]*17+A[6]*13+A[7]*11)%剩余牌的张数(51) 作为红方第一张牌位置，从牌组中取出。

(A[10]*17+A[11]*13+A[12])%剩余牌的张数(50) 作为黑方第二张牌位置，从牌组中取出。

(A[15]*17+A[16]*13+A[17])%剩余牌的张数(49) 作为红方第二张牌位置，从牌组中取出。

(A[20]*17+A[21]*13+A[22])%剩余牌的张数(48) 作为黑方第三张牌位置，从牌组中取出。

(A[25]*17+A[26]*13+A[27])%剩余牌的张数(47) 作为红方第三张牌位置，从牌组中取出。

