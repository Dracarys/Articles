# 深入学习 Solidity 之合约结构

Solidity 中 Contract 与面向对象语言中的 Class 非常相似。每个 Contract 都可以包含一下声明，State Variables，Functions, Function Modifiers, Events, Struct Type 和 Enum Types。不仅如此，Contract 还可以继承自其它 Contract。

## State Variables

在Contract中，State variable 被用来永久的存储一些数值。

``` Solidity
pragma solidity ^0.4.0;

contract SimpleStorage {
    uint storedData; // State variable
    // ...
}

```

## Functions

Function 是 Contract 中的可执行代码单元。

``` Solidity
pragma solidity ^0.4.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}
```

Function 既可以在定义它的合约被调用，也可以被其它合约调用，相对外部调用它可以拥有不同的访问级别（更多请参见 Visibility and Getters）。

## Function Modifiers

Function Modifier 可以通过声明的方式，对某些 Function 的语义进行修饰（更多请参见 Contract部分的 Function Modifiers ）

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

Events 是输出 EVM 日志的一个便利接口。

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

欲了解 Event 是如何声明的，以及如何在 DApp 中应用，请参见 Contract 一节。

## Struct Types

Stuct 是自定义数据类型，它可以打包多个变量（请参见 types 部分中的 Structs一节）。

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

## Enum Types

Enum 可以用来创建包含一组有限值的自定义类型（请参见 types 部分中的 Enums 一节）。

``` Solidity
pragma solidity ^0.4.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}

```

