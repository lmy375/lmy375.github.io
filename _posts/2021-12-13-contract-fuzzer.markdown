---
lng_pair: id_contract_fuzzer
title: 智能合约 Fuzzer 调研
category: 智能合约
tags: [Solidity]
img: ":moon4.webp"
date: 2021-12-13 10:00:00 +0800

---

# 先说结论

## 开发向：合约开发自测

开发时，需要对合约功能要有明确的建模。在可以代码中适当位置编写 assert 进行检查验证合约运行是否满足最初模型的约束条件。

在条件允许的情况下，使用 `0.8.3` 以上版本的 solidity 编译器，使用内置的[约束求解 checker](https://docs.soliditylang.org/en/v0.8.11/smtchecker.html)进行检查。
```
solc target.sol --model-checker-engine all --model-checker-targets all
```

合约内编写 `echidna_` 函数（[参考](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/how-to-test-a-property.md)）进行某些固定的状态检查，并通过 echidna 工具进行 fuzz 测试。

部署时可移除 assert 及 `echidna_` 函数以节约 gas。

在无需修改代码的情况下，可使用 Smartian 和 ConFuzzius 检查合约语言层面的各种漏洞。

## 安全向：合约漏洞挖掘

fuzzer 是一种不错的漏洞挖掘手段。

智能合约 fuzzer 至少需要解决如下问题：
- 支持合约调用序列、合约调用参数的变异。
- 支持对合约调用的 context （sender/gas/blocknumber/timestamp 等）的变异。
- 支持预设的 code, storage, balance 等，尤其对于 DeFi 项目来说，能模拟链上的情况非常重要。
- 独立的 EVM 执行模块，保证执行效率，且可提供丰富的执行过程信息，最好可以支持自定义的 hook，进行一些定制化的操作。
- 内置检测规则（如 reentrancy, int-overflow 等），最好可以支持自定义检测规则（如经济模型上的套利漏洞）。
- 代码风格好，模块化好，易于功能定制化和二次开发。

已有工具：
- 只有 echidna 自行编写检测函数进行检测。
- 检测 assert 和其他语言层面的漏洞类型，Smartian 和 ConFuzzius 比较好。
- ConFuzzius 最易于二次开发。

--------

以下是一些工具的简单分析。注：学术界使用 bug oracles 一词表示漏洞模式，与预言机没有关系。

# manticore

- 源码（python 编写）：https://github.com/trailofbits/manticore
- wiki：https://github.com/trailofbits/manticore/wiki
- doc：https://manticore.readthedocs.io/en/latest/verifier.html
- 文章 [Manticore: Symbolic execution for humans](https://blog.trailofbits.com/2017/04/27/manticore-symbolic-execution-for-humans/)
- ASE 2019 视频 https://www.youtube.com/watch?v=o6pmBJZpKAc
- 论文：[Manticore: A User-Friendly Symbolic Execution Framework for Binaries and Smart Contracts](https://arxiv.org/pdf/1907.03890.pdf)
- EVM API 教程 https://github.com/trailofbits/manticore/wiki/Getting-Started-EVM
- 使用 demo:
    - manticore-verifier micro tutorial https://asciinema.org/a/xd0XYe6EqHCibae0RP6c7sJVE
    - Manticore 0.1.6 Ethereum tool https://asciinema.org/a/154012

trailofbits 的工具。2017年发布，至年仍在积极维护。EVM, WASM, ELF Binary **符号执行**工具。最初思路源于 [pysymemu]( https://github.com/feliam/pysymemu)。默认使用 z3 作求解器。开放 API 接口。

命令行工具：
- manticore 进行路径搜索
- manticore-verifier 支持写一些订制的检查函数，并搜索触发输入。


工具集成在 `eth-security-toolbox` docker 中，主机 pip 安装失败或者运行报错的话，可以通过 docker 试用。
```
docker pull trailofbits/eth-security-toolbox
docker run -ti -v ~/Desktop/code/:/code trailofbits/eth-security-toolbox
```

实际测试测试执行了一下对 ERC20 合约 [WAVES](https://etherscan.io/address/0x1cf4592ebffd730c7dc92c1bdffdfc3b9efcf29a#code) 的执行会最报错，应该是求解时出错导致的。
```
manticore test.sol --contract WAVES
```
测试示例小程序还是可以正常运行的。

总结来说，即使是智能合约这类较为简单的程序，符号执行技术仍要面对消耗大量算力、内存，或者无法求解的情况。对于比较简单的合约或许有效，但在目前 DeFi 合约的复杂程度下，仍然有很多问题需要解决。

另外在约束求解这一方法上，0.8.3 版本及以上可以尝试 solidity 编译器内置的约束求解器。其以 require 语句作为约束；以 assert, 整数溢出，数字越界等作为求解目标。可以打印出触发问题的交易序列。参考： https://docs.soliditylang.org/en/v0.8.11/smtchecker.html

命行如下
```
solc target.sol --model-checker-engine all --model-checker-targets all
```


# echidna

- 源码（haskell 编写）： https://github.com/crytic/echidna
- binary 下载 https://github.com/crytic/echidna/releases/
- 文章：
    - [HOW TO USE ECHIDNA TO TEST SMART CONTRACTS](https://ethereum.org/en/developers/tutorials/how-to-use-echidna-to-test-smart-contracts/)
    - [Smart Contract Fuzzing How to find edge cases with echidna](https://medium.com/coinmonks/smart-contract-fuzzing-d9b88e0b0a05)
    - 使用教程 https://github.com/crytic/building-secure-contracts/tree/master/program-analysis/echidna#echidna-tutorial

特性：
- 使用 Slither 在 fuzzing 前做扫描
- 支持覆盖率引导
- 基于 [hevm](https://github.com/dapphub/dapptools/tree/master/src/hevm)
- 依赖 [crytic_compile](https://github.com/crytic/crytic-compile) 进行项目编译
- 支持自写的 property（即一种检查函数，在合约内以 `echidna_` 开头，[参考](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/how-to-test-a-property.md)）


工具集成在 `eth-security-toolbox` docker 中，也直接在 docker 中使用（如果是 Mac Docker Desktop 要设置内存大一些，否则程序无法运行起来。）。也可直接下载预先编译好的 binary（有 Mac aarch64 版本，但报错缺少某些库。运行 x64 版本正常）。

下面测试检查 assert，[参考](https://github.com/crytic/building-secure-contracts/blob/master/program-analysis/echidna/assertion-checking.md)：
```
$ cat assert.sol
contract Incrementor {
  uint private counter = 2**200;

  function inc(uint val) public returns (uint){
    uint tmp = counter;
    counter += val;
    assert (tmp <= counter);
    return (counter - tmp);
  }
}
$ cat assert.yaml
checkAsserts: true
$ echidna-test assert.sol --config assert.yaml
Analyzing contract: /code/git/building-secure-contracts/program-analysis/echidna/example/assert.sol:Incrementor
assertion in inc: failed!?
  Call sequence:
    inc(70596428801274647407482784864924342840619596841871813884518646479937257252430)
    inc(45197891668523020478856642220863704844510115297279180220639739244301234526221)


Unique instructions: 121
Unique codehashes: 1
Corpus size: 1
Seed: -5514501489610611866
```
可以看到 fuzzer 成功发现了可以触发 assert 的交易序列。

小结：

优势：
- 开箱即用。
- 编写 property 方式比较直观（ `assert` + `echidna_` 函数）
- 支持测试函数黑名单
- 覆盖率引导
- 自动精简样本

不足：
- 不支持自定义的变异、生成策略
- 由于是 haskell 编写，自行定制化修改会比较困难
- 程序运行速度偏慢（haskell 语言有关？）


# Consensys Diligence Fuzzing

Fuzzer 部分未开源：
- 主页 https://consensys.net/diligence/fuzzing/
- Harvey 论文，FSE 2020  https://mariachris.github.io/Pubs/FSE-2020-Harvey.pdf
- Targeted Greybox Fuzzing with Static Lookahead Analysis, ICSE 2020 https://mariachris.github.io/Pubs/ICSE-2020.pdf
- Fuzzing 论文 Learning Inputs in Greybox Fuzzing
- 文档 https://fuzzing-docs.diligence.tools/

Scribble 源码插装工具开源，可以在智能合约上添加注释语言作为 assertion，注释插装后的合约可更方便的用于 fuzz。（但感觉这个工具意义不大，不如直接修改合约更容易添加一些逻辑复杂的约束，而且没有学习 Scribble 注释语言的成本。）
- 文档 https://docs.scribble.codes/
- 安装 `npm install -g eth-scribble`

# ContractFuzzer

- 论文 [ContractFuzzer: Fuzzing Smart Contracts for Vulnerability Detection](https://arxiv.org/pdf/1807.03932.pdf) ASE 2018
    - 北航姜博老师 https://jiangbo.buaa.edu.cn/
- 源码（go 语言）：https://github.com/gongbell/ContractFuzzer

论文中 `Defining Testing Oracles for Vulnerabilities of Smart Contracts` 一节对几种漏洞类型进行了建模，可参考（但有些细节其实不是很准确）。

整体结构：
- 抓取大量合约并部署到私有链上；
- 私有链 geth EVM 添加检查漏洞模式的代码；
- 外部根据合约的 ABI 生成调用，发送给 geth 执行交易。

代码实现还是比较粗糙
- go 解析合约，根据 ABI 生成调用数据，发送给 node.js 写的 contract_tester
- node.js 使用 web 连接 geth 私有节点执行这些交易。
- geth 私有结点自己进行 patch，添加 evm 执行监测相关的功能（见 `go-ethereum/core/vm/hacker_***.go` 文件 和 `go-ethereum/core/vm/evm.go` 文件）
    - `hacker_contractcall.go` 记录 EVM 执行时的调用栈、指令 trace 情况。
    - `hacker_oracle.go` 根据调用栈，指令 trace 去匹配特定的一些漏洞模式。

优势：
- 可以测试合约间调用产生的问题（但是样本空间太大，实际效果应该不明显）

不足：
- 解耦做得不好，系统过于复杂，难以扩展定制化。
- 对单一合约来讲变异深度不够。
- 使用 geth 私有链进行测试，效率较差。


# Smartian

- 论文 ASE 2021 [SMARTIAN: Enhancing Smart Contract Fuzzing with Static and Dynamic Data-Flow Analyses](https://softsec.kaist.ac.kr/~jschoi/data/ase2021.pdf)
    - 注：韩国 kaist 大学近年在二进制安全成果很多，发布了很多开源工具
- 视频 https://www.youtube.com/watch?v=tnKOED6NWS8
- 源码（F# 语言） https://github.com/SoftSec-KAIST/Smartian
- 实验数据 https://github.com/SoftSec-KAIST/Smartian-Artifact 只包括 v0.4.25 的合约。安装 ILF, sFuzz, manticore, mythril 的 Docker 脚本。

基本原理：在合约执行前进行 bytecode 层的数据流分析（define-use chain），根据分析结果生成交易序列（比如 x 函数写了变量 a, y 函数读了变量 a，则可以生成先调用 x 再调用 y 的 txs）。在执行过程也会根据动态污点分析，来进行 bug oracle 的判定。

论文 Table II 对比了现有的静态分析、符号执行、fuzzer 工具，可参考。

实现：
- 主体代码使用 F#，是 .net 的一款函数式编程语言。
- evm code 静态分析在 EVMAnalysis 中实现，基于 [B2R2](https://github.com/B2R2-org/B2R2)。
- EVM 执行基于 [nethermind](https://github.com/NethermindEth/nethermind)
- fuzz 策略基于 [Eclipser](https://github.com/SoftSec-KAIST/Eclipser)

使用
```
# 1. 安装 .net 环境。https://dotnet.microsoft.com/download
# 要安装 5.0 版本，注意只有 x64，6.x 版本才有 arm 版本
# ARM 版本安装在 /usr/local/share/dotnet/dotnet
# x64 版本安装在 /usr/local/share/dotnet/x64/dotnet
# dotnet --info 查看安装的 SDK 版本信息、目录信息

# 下载（注意 --recursive ）
git clone https://github.com/SoftSec-KAIST/Smartian --recursive

# 编译
dotnet build -c Release -o ./build

# 运行 fuzzer
dotnet_x64 build/Smartian.dll fuzz -p ./examples/bc/AF.bin -a ./examples/abi/AF.abi -t 1000 -o ./fuzz_output
# 很快就可以看到 fuzzer 发现了触发 assertion 的 txs

# 复现
dotnet_x64 build/Smartian.dll replay -p ./examples/bc/AF.bin -i ./fuzz_output/bug
```

由于源码是 F# 编写的，没有深入分析。但从试用上看效果不错，而且工具封装的很好，开箱即用。fuzz 只需要提供 bytecode 和 abi，因此不受 solc 编译器版本的影响。

目前只支持固定的几种漏洞类型，因为本身代码是 F# 编写的，二次开发也有一定难度。而且由于 Fuzzer 收集的信息在 bytecode 级，只适用于检查编程语言层面的漏洞，对于经济模型类的套利问题，此 fuzzer 应该不会明显优势。

# sFuzz

- 论文 [sFuzz: An Efficient Adaptive Fuzzer for Solidity Smart Contracts](https://arxiv.org/pdf/2004.08563.pdf) ICSE 2020
- 视频 https://www.youtube.com/watch?v=jDLplO_elyw
- 源码 https://github.com/duytai/sFuzz C++ 编写，主要代码 2018年11月开发
- 在线 https://contract.guardstrike.com/#/scan
    - 这个页面对语言类合约问题总结的很全，且有代码示例 https://contract.guardstrike.com/#/knowledge
- 作者来自 Singapore Management University https://sunjun.site/analyzing-contracts/

使用 AFL 的思路来 fuzz 智能合约，额外添加了 branch distance 作为引导。

代码使用 C++ 编写。
- 文档 https://github.com/duytai/sFuzz/blob/master/AFL.md
- 并没有复用 AFL 代码，自己实现了一套，代码风格还可以。将交易序列 encode 成 bytes，然后由 AFL 进行 bytes 层面的变异。
- 使用 aleth 的 EVM。fuzz 对象是 EVM bytecode。目录结构与 https://github.com/ethereum/aleth 一致。主要添加 `fuzzer` `libfuzzer` `liboracle` 文件夹。
- 支持的 oralce 见 `liboracle/OracleFactory.cpp`

小结：没有明显的创新点，只是把二进制 fuzz 的一些思路应用到智能合约上。



# ConFuzzius

- slides [ConFuzzius: A Data Dependency-Aware Hybrid Fuzzer for Smart Contracts](https://www.ieee-security.org/TC/EuroSP2021/slides/Christof%20Ferreira%20Torres%20-%20Christof%20Ferreira%20Torres-ConFuzzius%20A%20Data%20Dependency%20Aware%20Hybrid%20Fuzzer%20For%20Smart%20Contracts.pdf)
- 论文 EuroS&P 2021 [CONFUZZIUS: A Data Dependency-Aware Hybrid Fuzzer for Smart Contracts](https://arxiv.org/pdf/2005.12156.pdf)
- 源码（python实现） https://github.com/christoftorres/ConFuzzius

碰瓷 confucius

可通过 docker 使用
```
# 拉取 docker
docker pull christoftorres/confuzzius
docker run -it -v ~/Desktop/code:/code christoftorres/confuzzius

# 运行 fuzzer
python3 ~/fuzzer/main.py -s AF.sol -c AssertionFailure --solc v0.4.26 --evm byzantium -t 100
```

- 代码实现不错。
- 有污点分析和符号执行，z3 求解
- 使用 pyevm 运行得到 trace，用符号执行引擎分析 trace 匹配漏洞模式。关键代码 `fuzzer/engine/analysis/execution_trace_analysis.py`