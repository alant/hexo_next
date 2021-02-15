---
title: 一个简单的以太坊智能合约开发练习
date: 2018-04-22 14:40:19
tags:
---

以太坊 V 神在一个采访中说过， 世界上现在分成两种人， 一种是知道什么是比特币的， 一种是不知道的。 对于知道什么是比特币的人， 解释以太坊是什么要相对容易一些。 就是比特币基础上再加了一个智能合约。 对于不知道什么是比特币的， 可以先在网上科普一下。 

最近几天把 hackernoon 上面的一篇以太坊智能合约开发指南教程跑了一遍。 开发的例子灵感是来自于一部电影[时间规划局](https://movie.douban.com/subject/4924142/)。 一部褒贬不一的电影， 但是我挺喜欢的， 题材不错。 就是未来世界，钱就是时间。 所有人的身体都可以定格在 25 岁， 能活多久取决于左手上的计时器。 穷人就得每天打工赚取活下去的时间， 富人就有很多时间 / 钱。 

说回智能合约， 这个智能合约就是很简单的比大小。 两个玩家， 在任何一轮玩家一号打入智能合约的钱是玩家二号打入的两倍或以上的话， 游戏结束，合约里所有的钱就归玩家一号。 反之亦然。 如两倍以内， 则进入下一轮。 每轮双方只能打一次钱进入合约。

我比较喜欢这个教程的第一点是， 一上来不要求装一大堆框架和 SDK，折腾半天也没开始开发。 简单粗暴， 先来智能合约代码：

``` solidity
pragma solidity ^0.4.18;

contract Wrestling {
  address public playerA;
  address public playerB;

  bool public playerAPlayed;
  bool public playerBPlayed;

  uint private playerADeposit;
  uint private playerBDeposit;

  bool public gameOver;
  address public theWinner;
  uint public prize;

  constructor() public {
    playerA = msg.sender;
  }

  function registerAnOpponent() public {
    require(playerB == address(0));
    playerB = msg.sender;
  }

  function wrestle() public payable {
    require(!gameOver && (msg.sender == playerA || msg.sender == playerB));

    if (msg.sender == playerA) {
      require(!playerAPlayed);
      playerAPlayed = true;
      playerADeposit = playerADeposit + msg.value;
    } else {
      require(!playerBPlayed);
      playerBPlayed = true;
      playerBDeposit = playerBDeposit + msg.value;
    }

    if (playerAPlayed && playerBPlayed) {
      if (playerADeposit > 2 * playerBDeposit) {
        endGame(playerA);
      } else if (playerBDeposit > 2 * playerADeposit) {
        endGame(playerB);
      } else {
        endRound();
      }
    }
  }

  function endGame(address winner) internal {
    gameOver = true;
    theWinner = winner;
    prize = playerADeposit + playerBDeposit;
  }

  function endRound() internal {
    playerAPlayed = false;
    playerBPlayed = false;
  }

}
```

第一行 pragma solidity ^0.4.18; 是给编译器看的， 意思是用 0.4.18 版本以上的 solidity 编译器， 基本可以忽略。 后面的智能合约，如有任何现代语言的编程基础的话还是比较好懂的。 谁通过创造器（constructor）创造了合约谁就是一号玩家 playerA， playerA 和 playerB 的类型都是 address，根据 solidity 的[官方文档](https://solidity.readthedocs.io/en/develop/types.html#address)，address 是一个 20 个字节放以太坊地址的类型。

合约创造以后，二号玩家可以调用 registerAnOpponent() 来注册。 require(playerB == address(0)); 的意思是要求二号玩家还没有被注册过（地址为 0）。合约的主体是 wrestle()，payable 的意思是这个方法可以接受付款。 后面的代码就是实现上面说的游戏的逻辑。 语法和现代语言 （java，javascript，python）基本没有什么区别。 

至此， 智能合约的开发第一步就完成了， 神奇吧。 你可以把以上的代码复制粘贴到 solidity 的 IDE 中：[http://remix.ethereum.org/](http://remix.ethereum.org/)。然后点击 start to compile，一切顺利的话，你能看到 Wrestling 是绿色的， 表示编译没问题。如下图所示：

![](remix.jpg)

到这里其实合约还有一个问题。 就是如果赢了的话，奖金如何取出来？我们需要加一个取钱的方法：
<!--more-->
``` solidity
  function withDraw() public {
    require(gameOver && msg.sender == theWinner);
    uint amount = prize;
    prize = 0;
    msg.sender.transfer(amount);
  }
```

这里用了一个为了安全的写法， 先把 prize 赋值为 0， 再把钱发给赢家。 在这个 DApp 不是那么关键， 毕竟钱都是给赢家的。 但是如果合约里的钱是要分批给不同的人这个写法就非常重要了， 如果不这么写，有人可以利用一些手段反复调用这个方法，直到合约里的钱都被取光，还是很可怕的。

有了这些以后，就可以将合约部署到测试网络上试用一下了。 

先下载一个模拟测试网络的工具 Ganache， 在终端任何目录下执行以下命令均可，全局安装 ganache-cli：

``` bash
npm install -g ganache-cli
```

下一步将写好的合约放到测试网络上，看了好几种方法， 比较下来我认为比较合适的是使用 Truffle 这个框架。 直接用 Node / Web3.js 对我来说还是太原始了一点。 Node / js 大牛可以直接跳过剩下的内容， 移步 [Full Stack Hello World Voting Ethereum Dapp Tutorial — Part 1](https://medium.com/@mvmurthy/full-stack-hello-world-voting-ethereum-dapp-tutorial-part-1-40d2d0d807c2) 有详细的直接用 Node / Web3.js 调用智能合约的介绍。 

先安装 Truffle：

``` bash
npm install -g truffle
```

为项目创建一个目录。 进入这个目前是空的目录，执行：

``` bash
truffle init
```

Truffle 会创建几个目录。 在 contracts 目录下创建文件 Wrestling.sol，将上面的合约内容粘贴到 Wrestling.sol 中。 到这里，这个文件内容应该是这样子的：

``` solidity
pragma solidity ^0.4.18;

contract Wrestling {
  address public playerA;
  address public playerB;

  bool public playerAPlayed;
  bool public playerBPlayed;

  uint private playerADeposit;
  uint private playerBDeposit;

  bool public gameOver;
  address public theWinner;
  uint public prize;

  constructor() public {
    playerA = msg.sender;
  }

  function registerAnOpponent() public {
    require(playerB == address(0));
    playerB = msg.sender;
  }

  function wrestle() public payable {
    require(!gameOver && (msg.sender == playerA || msg.sender == playerB));

    if (msg.sender == playerA) {
      require(!playerAPlayed);
      playerAPlayed = true;
      playerADeposit = playerADeposit + msg.value;
    } else {
      require(!playerBPlayed);
      playerBPlayed = true;
      playerBDeposit = playerBDeposit + msg.value;
    }

    if (playerAPlayed && playerBPlayed) {
      if (playerADeposit > 2 * playerBDeposit) {
        endGame(playerA);
      } else if (playerBDeposit > 2 * playerADeposit) {
        endGame(playerB);
      } else {
        endRound();
      }
    }
  }

  function endGame(address winner) internal {
    gameOver = true;
    theWinner = winner;
    prize = playerADeposit + playerBDeposit;
  }

  function endRound() internal {
    playerAPlayed = false;
    playerBPlayed = false;
  }
}
```

下一步在 “migrations” 目录下建文件 “2_deploy_contracts.js” 用来将合约部署到区块链上。 代码如下：

``` javascript
const Wrestling = artifacts.require("./Wrestling.sol")

module.exports = function(deployer) {
	deployer.deploy(Wrestling);
};
```



接下来的步骤我没有在 windows 上面验证过， mac 的话需要在项目根目录下修改 “truffle.js” 文件，内容如下：

``` javascript
module.exports = {
  // See <http://truffleframework.com/docs/advanced/configuration>
  // for more about customizing your Truffle configuration!
  networks: {
    development: {
      host: "127.0.0.1",
      port: 7545,
      network_id: "*" // Match any network id
    }
  }
};
```

据说 windows 下需要删掉“truffle.js” 将以上内容放入 “truffle-config.js”。好了， 下面就要发车了。

先启动模拟测设网络的工具：

``` bash
ganache-cli -p 7545
```

会产生 10 个以太坊账号，每个里面有 100 个 Ether。 效果如下：

![](terminal-ganache.jpg)

然后编译和部署合约到上面这个假以太坊网络中去，保留上面的终端， 开一个命令在项目的根目录处执行：

``` bash
truffle compile
truffle migrate --network development
```

第一句话是编译合约，作用和上面 remix IDE 里有的编译功能是一样的， 第二句话是将编译好的合约部署到测试网上。测试网的参数在 ”truffle.js" 上配好了。

部署后执行 truffle 的终端显示如下：

![](terminal-truffle-migrate.jpg)

ganache 的终端显示如下：

![](terminal-ganache2.jpg)

值得注意的是上面两个终端里 Wresling: 0x… 和 contract created: 0x… 的地址是一致的。

细心的同学会注意到 Gas usage，以太坊上的运算和交易都需要付旷工费。具体 gas 怎么算可以上网查一下， 有不少相关的中文资料。

接下来可以和部署好的合约互动了。 在 truffle 的终端输入以下命令：

``` bash
truffle console --network development
```

第一步可以查一下账户的余额：

``` bash
web3.fromWei(web3.eth.getBalance(web3.eth.accounts[1]))
```

![](terminal-truffle-console.jpg)

合约是默认 accounts[0] 部署的，ganache 默认的 gas price 是 100,000,000,000 或者 100 gwei，部署这个合约花掉了 1,068,066 的 gas。加上余额，就能对上每个账号默认 100 个 ether 的量了。

接下来就可以开始调用合约的方法了：

``` bash
playerA = web3.eth.accounts[0]
playerB = web3.eth.accounts[1]
Wrestling.deployed().then(inst => { WrestlingInstance = inst })
WrestlingInstance.registerAsAnOpponent({from: playerB})
WrestlingInstance.wrestle({from: playerA, value: web3.toWei(2, "ether")})
WrestlingInstance.wrestle({from: playerB, value: web3.toWei(3, "ether")})
WrestlingInstance.wrestle({from: playerA, value: web3.toWei(5, "ether")})
WrestlingInstance.wrestle({from: playerB, value: web3.toWei(20, "ether")})
```

默认合约的调用者是 accounts[0]，根据合约的代码就是 playerA 了。 这里玩了两轮。 第二轮符合游戏结束的条件。 

到这里这个合约基本就开发完成了。 下一步是加上几个事件的代码。 并配上一个前端网页显示相关的信息。 
