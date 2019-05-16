# 【译】Solidity源文件结构

源文件中可以定义任意数量的合约，及若干 include 和 pragma。

## 版本标注（Version Pragma）

源文件中可以（也应该）使用版本标注来指明版本，一面将来更新版本的编辑器在编译器时发生异常。 We try to keep such changes to an absolute minimum and especially introduce changes in a way that changes in semantics will also require changes in the syntax, but this is of course not always possible. Because of that, it is always a good idea to read through the changelog at least for releases that contain breaking changes, those releases will always have versions of the form `0.x.0` 或者 `x.0.0`。

版本标注的使用方法如下：

```Solidity
pragma solidity ^0.4.19;
```
如此该源文件将无法被低于0.4.0的编译器编译，同时它也将不能在高于0.5.0的编译器上工作（该限制是通过`^`来指定）。如此一来在0.5.0版本之前都不会有什么大的变动，以保证代码按照我们的预先设想被编译。大版本虽然不变，但小版本的bug修复还是会持续不断的。

除此之外，还可以为编译器版本制定更多规则，详细方法请参见 [npm](https://docs.npmjs.com/misc/semver) 的使用。

## 引用其它源文件（Importing other Source Files）

### 语法和语义（Syntax and Semantics）

Solidity与 JavaScritp ES6 类似支持 import声明，但 Solidity 没有 “default export”的概念。

在全局作用范围内，可以通过一下方式进行 import声明：

```Solidity
import "filename";
```

下面的声明会将“filename”中的所有全局符号引入到当前作用范围内。

```Solidity
import * as symbolName from "filename";
```
...同时创建一个包涵 `filename` 文件中所有全局符号的新的全局符号 `symbolName` 。

```Solidity
import {symbol1 as alias, symbol2} from "filename";
```

...创建2个新的全局符号`alias`和`symbol2`，他们分别引用自`filename`文件中的`symbol1`和`symbol2`。

另一种语法虽然ES6种没有，但是相当便利：

```Solidity
import "filename" as symbolName;
```
它等同于`import * as symbolName from "filename";`

### 路径（Paths）

如前文所示，`filename`总是被当作以`/`作为分割符的路径来处理，`.`表示当前目录，`..`表示上一级目录。一旦`.`或`..`后面跟一个`／`以外的字符，它将不再表示当前或者上一级目录。除非以`.`或者`..`开头，否则所有路径都将视为绝对路径。

可以这样`import "./x" as x;`来引入统计目录中的`x`文件。如果你这样写`import "x" as x;`，那么将有可能引入其它同名文件（例如位于“include”目录的同名文件）。

如何解析路径完全依赖于编译器。通常，在本地的文件系统中你不需要严格地表示出完整的路径层级关系，此外它还支持发现：ipfs，http以及git。

### （Use in Actual Compilers）
When the compiler is invoked, it is not only possible to specify how to discover the first element of a path, but it is possible to specify path prefix remappings so that e.g. `github.com/ethereum/dapp-bin/library` is remapped to `/usr/local/dapp-bin/library` and the compiler will read the files from there. If multiple remappings can be applied, the one with the longest key is tried first. This allows for a “fallback-remapping” with e.g. "" maps to `"/usr/local/include/solidity"`. Furthermore, these remappings can depend on the context, which allows you to configure packages to import e.g. different versions of a library of the same name.

#### solc:

For solc (the commandline compiler), these remappings are provided as `context:prefix=target` arguments, where both the `context: `and the `=target` parts are optional (where target defaults to prefix in that case). All remapping values that are regular files are compiled (including their dependencies). This mechanism is completely backwards-compatible (as long as no filename contains = or :) and thus not a breaking change. All imports in files in or below the directory `context` that import a file that starts with prefix are redirected by replacing `prefix` by `target`.

So as an example, if you clone `github.com/ethereum/dapp-bin/` locally to `/usr/local/dapp-bin`, you can use the following in your source file:

```
import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;
```

and then run the compiler as

```
solc github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ source.sol
```

As a more complex example, suppose you rely on some module that uses a very old version of dapp-bin. That old version of dapp-bin is checked out at /usr/local/dapp-bin_old, then you can use

```
solc module1:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin/ \
     module2:github.com/ethereum/dapp-bin/=/usr/local/dapp-bin_old/ \
     source.sol
```

so that all imports in `module2` point to the old version but imports in `module1` get the new version.

Note that solc only allows you to include files from certain directories: They have to be in the directory (or subdirectory) of one of the explicitly specified source files or in the directory (or subdirectory) of a remapping target. If you want to allow direct absolute includes, just add the remapping `=/`.

If there are multiple remappings that lead to a valid file, the remapping with the longest common prefix is chosen.

#### Remix:

*Remix* provides an automatic remapping for github and will also automatically retrieve the file over the network: You can import the iterable mapping by e.g. 

```
import "github.com/ethereum/dapp-bin/library/iterable_mapping.sol" as it_mapping;
```

Other source code providers may be added in the future.

## 注释（Comments）

单行注释`//`和多行注释`/*...*/`都是允许的。

```Solidity
// 这是一个单行注释

/*
这是一个
多行注释
*/
```
此外，还有一种注释被称为natspec注释，关于它的文档尚未完成。该注释以三个斜杠`///`或者双星号`/**...*/`开始，该注释应该直接应用在函数声明前。可以在注释中插入Doxygen标记，以便用户更好的理解该函数。

在下面的例子中，我们通过注释说明了智能合约的title以及2个输出参数和2个返回值。

``` Solidity
pragma solidity ^0.4.0;

/** @title Shape calculator. */
contract shapeCalculator {
    /** @dev 通过计算得到矩形的面积和周长。
      * @param w 矩形的宽度。
      * @param h 矩形的高度。
      * @return s 计算得到的面积。
      * @return p 计算得到的周长。
      */
    function rectangle(uint w, uint h) returns (uint s, uint p) {
        s = w * h;
        p = 2 * (w + h);
    }
}
```