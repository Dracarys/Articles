# 【译文】深入学习 Solidity 之合约结构

Solidity 中合约（Contract）与面向对象语言中的类（Class）非常相似。每个合约都可以包含以下声明，状态变量（State Variables），函数（Function）, 函数修饰器（Function Modifiers）, 事件（Events）, 结构体（Struct Type）和枚举（Enum Types）。不仅如此，合约还可以继承自其它合约。

## 状态变量（State Variables）

在合约中，状态变量被用来存储一些数值。

``` Solidity
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData; // State variable
    // ...
}

```

## 函数(Functions)

函数是合约中的可执行代码单元。

``` Solidity
pragma solidity ^0.4.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}
```

函数既可以在定义它的合约被调用，也可以被其它合约调用，相对与内部调用，外部调用可以拥有不同的访问级别（更多请参见 Visibility and Getters）。

## 函数修饰器（Function Modifiers）

函数修饰器可以通过声明的方式，对某些函数的语义进行修饰（更多请参见 Contract部分的 Function Modifiers ）

``` Solidity
pragma solidity ^0.4.11;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(msg.sender == seller);
        _;
    }

    function abort() public onlySeller { // Modifier usage
        // ...
    }
}
```

## Events

事件是输出 EVM 日志的一个便利接口。

``` Solidity
pragma solidity ^0.4.0;

contract SimpleAuction {
    event HighestBidIncreased(address bidder, uint amount); // Event

    function bid() public payable {
        // ...
        HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
```

欲了解事件是如何声明的，以及如何在 DApp 中应用，请参见 Contract 一节。

## 结构体类型（Struct Types）

结构体是自定义数据类型，它可以打包多个变量（请参见 types 部分中的 Structs一节）。

``` Solidity
pragma solidity ^0.4.0;

contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

## 枚举类型（Enum Types）

枚举可以用来创建包含一组有限值的自定义类型（请参见 types 部分中的 Enums 一节）。

``` Solidity
pragma solidity ^0.4.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}

```