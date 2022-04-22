<div align="center">
  <a href="https://liamcobb.com/">
    <img alt="starknet logo" src="https://ethereum.org/static/28214bb68eb5445dcb063a72535bc90c/9019e/hero.webp" >
  </a>
  <h1 align="center">zksync-contract front-end</h1>
  <p align="center">
    <a href="https://github.com/hedgezhu/zksync-demo-front-end/graphs/contributors">
      <img alt="GitHub contributors" src="https://img.shields.io/github/contributors/hedgezhu/zksync-demo-front-end">
    </a>
    <a href="https://www.typescriptlang.org/">
      <img alt="TS" src="https://img.shields.io/badge/--3178C6?logo=typescript&logoColor=ffffff">
    </a>
    <a href="http://makeapullrequest.com">
      <img alt="pull requests welcome badge" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat">
    </a>
  </p>
</div>

# zksync-contract-front-end

安装依赖文件
```
yarn install
```

启动项目
```
yarn serve
```

# 介绍

连接[zksync network](https://v2-docs.zksync.io/dev/testnet/metamask.html)，使用metamask provider和signer与节点通信、并签发交易

1. 通过vue实现了一个简单的合约调用页面，连接钱包后可以收到合约中存储的消息

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter.png)

2. 选择token时，自动调用getBalance、getFee计算钱包余额和交易费用

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter2.png)

3. 更进一步，输入新的greeter，调用合约更新，这里我输入的是 “跨越长城，走向世界”，来自1987年9月14日从中国发出的[第一封电子邮件](https://en.wikipedia.org/wiki/Internet_in_China)，代表中国民众中追求进步的力量

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter3.png)

4. 合约调用成功，可以看到合约内存储的消息变为了 “跨越长城，走向世界”

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter4.png)

---

# zksync-contract-front-end

Install dependencies
```
yarn install
```

Startup project
```
yarn serve
```

# introduce

Connect to the [zksync network](https://v2-docs.zksync.io/dev/testnet/metamask.html), use the metamask provider and signer to communicate with nodes and sign transactions

1. A simple contract calling page is implemented through vue. After connecting to the wallet, you can receive the messages stored in the contract

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter.png)

2. When a token is selected, getBalance and getFee are automatically called to calculate wallet balance and transaction fees

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter2.png)

3. Going a step further, enter a new greeter to invoke contract update, here I typed "跨越长城，走向世界", from [the first email](https://en.wikipedia.org/wiki/Internet_in_China) sent from China on September 14, 1987, representing the progressive force among the Chinese people

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter3.png)

4. The contract is successfully invoked, and you can see that the message stored in the contract has changed to "Across the Great Wall we can reach every corner in the world"

![image](https://github.com/hedgezhu/zksync-demo-front-end/blob/main/img/greeter4.png)
