---
title: Compile
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-07T21:26:29+08:00
lastmod: 2024-03-14T21:53:43+08:00
tags:
categories:
  - Graph-Computing
weight: 0

password: woshinidaye
message:

draft: false
repost:
  enable: false
  url:

hiddenFromHomePage: false
hiddenFromSearch: false

comment: false
lightgallery: force
---

目前涉及到两个关键的概念：MLIR 和 LLVM[^LLVM]。LLVM 通过引入 IR 的概念，减轻了传统编译器前后端之间的强耦合关系。与此同时也凸显出了模块化的概念，通过 IR 可以自由实现前后端的组合。而MLIR和IR是同一类别的事物，差别在于其更加通用可扩展，可以用于描述和表示程序的结构和语义信息。

如果想要利用MLIR实现图计算的相关编译工作，可行的思路就是将MLIR转换为LLVM IR（LLVM的中间表示形式），然后利用LLVM提供的优化器和代码生成器将LLVM IR编译成目标平台的机器代码，简单来说就是利用LLVM的后端，这样，MLIR可以利用LLVM的强大优化和代码生成能力，为不同的编程模型和应用场景提供高效的编译器支持。

[^LLVM]: [LLVM](https://gaohongy.github.io/blog/posts/compile-link/c-c++-compile-link/#llvm)

Just a guess:

gc: Graph Convolution（图卷积）
gnn: Graph Neural Networks（图神经网络）
gpm: 


在了解 MLIR 过程中发现使用到了一个名为 TableGen 的工具，有一种说法是它是 一种声明式编程语言。似乎通过相关工具，可以将.td文件生成C++代码。粗浅理解似乎是为了能够运用某些公共组件

## 编译原理
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403122106578.png)

### 词法分析（Lexical Analysis）：
将源代码转换为标记（token）序列，识别出代码中的基本单位（如关键字、标识符、常量、运算符等），并移除注释和不必要的空白字符。

### 语法分析（Syntax Analysis）：
根据语法规则检查标记序列是否符合编程语言的语法结构。将标记序列转换为语法树（Parse Tree）或者抽象语法树（Abstract Syntax Tree），以便后续阶段的处理。

### 语义分析（Semantic Analysis）：
检查代码中的语义是否合法，即代码是否符合语言的语义规则。这包括类型检查、作用域检查、语义错误检查等，以确保代码在逻辑上是正确的。

## 对于 MLIR 的理解

1. 背景：在当前编译结构中，各种IR之间转换的效率和可迁移性不高
![](https://pic2.zhimg.com/80/v2-b6e62260b0faf1085f972d1eda6e4bb1_1440w.webp)

2. 引入 MLIR 所期望做到的：使用一种一致性强的方式，为各种DSL提供一种中间表达形式，将他们集成为一套生态系统，编译到特定硬件平台的汇编语言上
![](https://pic2.zhimg.com/80/v2-4b2fa235edc4387378a84b8a3587efd9_1440w.webp)


MLIR 表达式组成:
![](https://pic4.zhimg.com/80/v2-6d75286d07a53555437f2c436f718083_1440w.webp)

MLIR 并不是一个端到端（从计算图到最后可执行程序这个全流程）的框架，只是一个基础架构

## MLIR 编译

1. 如果需要用到[Python Bindings](https://mlir.llvm.org/docs/Bindings/Python/#building)，在[MLIR Getting Started](https://mlir.llvm.org/getting_started/)给出的编译命令之外是需要添加额外的编译选项的，完整示例如下所示：

同时需要注意文档中提到的对于部分python包的依赖，这部分需要在编译之前进行安装

```bash
cmake -G Ninja ../llvm \
    -DLLVM_ENABLE_PROJECTS=mlir \
    -DLLVM_BUILD_EXAMPLES=ON \
    -DLLVM_TARGETS_TO_BUILD="Native" \
	-DCMAKE_BUILD_TYPE=Debug \
	-DLLVM_USE_SPLIT_DWARF=ON \
	-DLLVM_ENABLE_ASSERTIONS=ON \
	-DCMAKE_C_COMPILER=clang \
	-DCMAKE_CXX_COMPILER=clang++ \
	-DLLVM_ENABLE_LLD=ON \
	-DMLIR_ENABLE_BINDINGS_PYTHON=ON \
	-DPython3_EXECUTABLE="/opt/homebrew/bin/python3"
```

2. compiler这个项目所包含的CMakeLists.txt中写死了一些路径，这些都需要进行修改

3. compiler项目在CMakeLists.txt中使用到了一些`find_package()`，涉及到numpy和pybind11，需要通过`CMAKE_PREFIX_PATH`手动指定路径


## MLIR Tutorials

在 llvm 的项目源码中，同MLIR Tutorials相关的需要关注的是两个路径下的文件:

- `/Users/gaohongyu/llvm/mlir/test/Examples/Toy`：存放的是 toy 语言的源程序
- `/Users/gaohongyu/llvm/mlir/examples/toy`：存放的是实现 toy 语言获取 AST，MLIR 表达式等工具的源代码 (产生的工具可执行文件在`build/bin`目录中)

### Chapter 1: Toy Language and AST
这一部分主要内容就是创造了一种新的语言，叫做Toy，然后实现了一个简易的语法分析器，仅仅能够获得toy源程序对应的抽象语法树。所以说这一节的重点就是抽象语法树

### Chapter 2: Emitting Basic MLIR
在第一节中已经生成了 Toy 语言源程序的 AST，这一节就是要根据 AST 结合 Dialect 来生成 MLIR表达式

![](https://mmbiz.qpic.cn/mmbiz_png/SdQCib1UzF3tN9fRfXZhWRgL2OLr400ESibMbgibPJfUrSLDicq855g64h5cz6CHn4lstoRPJ2KjGbG2q43ANqSPmg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

如何理解 Dialect？

Dialects provide a grouping mechanism for abstraction under a unique namespace

MLIR is designed to allow all IR elements, such as attributes, operations, and types,

自定义 Dialect 同 MLIR 结合的方式：

C++实现Dialect：

```cpp
class ToyDialect : public mlir::Dialect {
public:
  explicit ToyDialect(mlir::MLIRContext *ctx);

  static llvm::StringRef getDialectNamespace() { return "toy"; }

  void initialize();
};
```

借助tablegen工具实现Dialect：

```
def Toy_Dialect : Dialect {
  let name = "toy";

  let summary = "A high-level dialect for analyzing and optimizing the "
                "Toy language";

  let description = [{
  }];

  let cppNamespace = "toy";
}
```

因为这一节的内容的主要目的就是探索从 抽象语法树AST 转为 MLIR表达式 的过程，所以说 Ch2 相较于 Ch1 增加的从模块上来将，就将增加 生成MLIR表达式所需的3个模块

具体模块对应的文件是：

- MLIRGen模块：`mlil/MLIRGen.cpp` 和 `include/toy/MLIRGen.h`
- Dialect模块：`mlir/Dialect.cpp` 和 `include/toy/Dialect.h`
- TableGen模块：`include/toy/Ops.td`


1. 从抽象语法树 生成 MLIR表达式

```
# AST 中 toy源程序中 transpose(a) 语句对应的内容

Call 'transpose' [ @codegen.toy:5:10
  var: a @codegen.toy:5:20
]
```

经过 MLIRGen 以下代码的加工处理，将 AST 转变为了 MLIR 表达式，其中的关键在于`if (callee == "transpose")`，即当程序扫描到`transpose`关键词时，就会返回构造出的 transpose 的 MLIR表达式中的节点，即第一步的内容

```cpp
mlir::Value mlirGen(CallExprAST &call) {
  llvm::StringRef callee = call.getCallee();
  auto location = loc(call.loc());

  // Codegen the operands first.
  SmallVector<mlir::Value, 4> operands;
  for (auto &expr : call.getArgs()) {
    auto arg = mlirGen(*expr);
    if (!arg)
      return nullptr;
    operands.push_back(arg);
  }

  // Builtin calls have their custom operation, meaning this is a
  // straightforward emission.
  if (callee == "transpose") {
    if (call.getArgs().size() != 1) {
      emitError(location, "MLIR codegen encountered an error: toy.transpose "
                          "does not accept multiple arguments");
      return nullptr;
    }
    return builder.create<TransposeOp>(location, operands[0]);
  }

  // Otherwise this is a call to a user-defined function. Calls to
  // user-defined functions are mapped to a custom call that takes the callee
  // name as an attribute.
  return builder.create<GenericCallOp>(location, callee, operands);
}
```

不过存在一个问题是 在构造MLIR表达式节点时，利用到了一个 `TransposeOp`类，它应当表示的是源程序和MLIR表达式中的 `transpose`操作，这个类从何而来？

我目前的理解是，所有数据类型都是在td文件中以一种声明式语法来说明的，后续需要用到哪些类型的文件则通过`mlir-tblgen`工具来生成。那么如何理解Dialect模块，我的理解是TableGen模块提供的只是一些基本原料，但是这些原料究竟怎么用到项目中，是由Dialect模块来决定的或者说设置的。所以说我们需要通过td文件对于源程序中涉及到的结构进行抽象。

实际上，从 `mlir-tblgen` 的 help 信息可以看出，tb文件可以生成很多格式的信息文件

```bash
--gen-attr-interface-decls                        - Generate attribute interface declarations
--gen-attr-interface-defs                         - Generate attribute interface definitions
--gen-attr-interface-docs                         - Generate attribute interface documentation

--gen-attrdef-decls                               - Generate AttrDef declarations
--gen-attrdef-defs                                - Generate AttrDef definitions
--gen-attrdef-doc                                 - Generate dialect attribute documentation

--gen-avail-interface-decls                       - Generate availability interface declarations
--gen-avail-interface-defs                        - Generate op interface definitions

--gen-bytecode                                    - Generate dialect bytecode readers/writers

--gen-convertible-llvmir-intrinsics               - Generate list of convertible LLVM IR intrinsics

--gen-dialect-decls                               - Generate dialect declarations
--gen-dialect-defs                                - Generate dialect definitions
--gen-dialect-doc                                 - Generate dialect documentation

--gen-directive-decl                              - Generate declarations for directives (OpenMP/OpenACC etc.)

--gen-enum-decls                                  - Generate enum utility declarations
--gen-enum-defs                                   - Generate enum utility definitions
--gen-enum-from-llvmir-conversions                - Generate conversions of EnumAttrs from LLVM IR
--gen-enum-to-llvmir-conversions                  - Generate conversions of EnumAttrs to LLVM IR

--gen-intr-from-llvmir-conversions                - Generate conversions of intrinsics from LLVM IR

--gen-llvmir-conversions                          - Generate LLVM IR conversions
--gen-llvmir-intrinsics                           - Generate LLVM IR intrinsics

--gen-op-decls                                    - Generate op declarations
--gen-op-defs                                     - Generate op definitions
--gen-op-doc                                      - Generate dialect documentation
--gen-op-from-llvmir-conversions                  - Generate conversions of operations from LLVM IR
--gen-op-interface-decls                          - Generate op interface declarations
--gen-op-interface-defs                           - Generate op interface definitions
--gen-op-interface-docs                           - Generate op interface documentation

--gen-pass-capi-header                            - Generate pass C API header
--gen-pass-capi-impl                              - Generate pass C API implementation
--gen-pass-decls                                  - Generate pass declarations
--gen-pass-doc                                    - Generate pass documentation

--gen-python-enum-bindings                        - Generate Python bindings for enum attributes
--gen-python-op-bindings                          - Generate Python bindings for MLIR Ops

--gen-rewriters                                   - Generate pattern rewriters

--gen-spirv-attr-utils                            - Generate SPIR-V attribute utility definitions
--gen-spirv-avail-impls                           - Generate SPIR-V operation utility definitions
--gen-spirv-capability-implication                - Generate utility function to return implied capabilities for a given capability
--gen-spirv-enum-avail-decls                      - Generate SPIR-V enum availability declarations
--gen-spirv-enum-avail-defs                       - Generate SPIR-V enum availability definitions
--gen-spirv-serialization                         - Generate SPIR-V (de)serialization utilities and functions

--gen-type-interface-decls                        - Generate type interface declarations
--gen-type-interface-defs                         - Generate type interface definitions
--gen-type-interface-docs                         - Generate type interface documentation

--gen-typedef-decls                               - Generate TypeDef declarations
--gen-typedef-defs                                - Generate TypeDef definitions
--gen-typedef-doc                                 - Generate dialect type documentation
```


怎么理解这个方言？我目前的看法就是适配器，方言也同时说明了不同的人有不同的说法，同样的对于transpose来说，细节可能也不同，所以才被称之为方言

但是说通过多层IR逐步降级，这个降级是怎么表现的还是不太明白

根据[This is the C++ definition of a dialect, but MLIR also supports defining dialects declaratively via tablegen.](https://mlir.llvm.org/docs/Tutorials/Toy/Ch-2/#:~:text=This%20is%20the%20C%2B%2B%20definition%20of%20a%20dialect%2C%20but%20MLIR%20also%20supports%20defining%20dialects%20declaratively%20via%20tablegen.)的说法，td文件写的其实就是dialect，这个dialect可以直接通过C++代码进行定义，同时也可以使用tablegen结合td文件生成

### 生成 MLIR 表达式 所需的模块：[^MLIR-Mode]

[^MLIR-Mode]: [MLIR的生产线](https://zhuanlan.zhihu.com/p/102565792)

1. TableGen模块(生产线的零件): 通过.td文件定义了各种操作的类（这部分也叫做Operation Definition Specification (ODS)框架）(我理解这部分也可以通过手动编写C++代码实现，只是说可能写起来比较繁琐，同时在不同的场景下可能存在类似的需求，如果总是手动编写会带来很大的重复工作量，所以说一般通过td文件结合TableGen工具来生成)
2. Dialect模块(生产线的机械臂): 负责定义各种操作和分析，为操作添加相应的类型和操作数的值
3. MLIRGen模块(生产线履带): 遍历抽象语法树(AST)，构造 MLIR 节点

> TableGen模块在编译时向Dialect模块提供支持

我理解着上述这 3 个模块之间的关系，或者说作用流程，大概是 TableGen 模块生成的是一种类似模版的东西，然后 Dialect 像是具体的适配器，为模版添加不同的属性，这样两阶段分离的设计可以做到 模版 和 数据 解耦合，同时做到 模版 的复用，最后再通过 MLIRGen 将添加了具体属性的模版 生成为 MLIR 节点

![](https://pic4.zhimg.com/80/v2-95f6bf7b8482ab5a4d8541d16ba6cf7b_1440w.webp)

关注图中TransposeOp的指向

## 对于项目的理解
从关键技术那张图上，能够了解到CGA_Framework就是那个统一编程框架

王老师说我们做图学习还有编译的部分

我目前对于统一编程框架的理解是：目前项目中的include下面的各种.h文件提供的就是C++接口的功能，而apps则是使用这些接口的示例程序，apps就应当是等价于实际使用这个统一编程框架的用户所编写的代码

统一编程框架位于框架层，要注意到其上还存在一个应用层，应用层和框架层的接口应当是 GraphScope 和 DGL。也就是说用户仍然是利用GraphScope 和 DGL，来完成图计算、图挖掘和图学习的任务，但是开发者需要提供从GraphScope和DGL到统一编程框架的转换机制（**对应着任务中的“用户透明的代码自动转换机制"**)，目前猜测自动代码生成技术就是来负责实现这种转换的

这种转换应该是在存在着统一编程框架和Graph
现在涉及到几个关键词，自动代码生成，领域特定语言DSL，中间表达IR

从领域特定语言的作用机制来说，库从某种意义上来说本身就是一种DSL，如何从DSL转到另一种语言，这里如果借助IR，似乎再通过llvm IR以及llvm backend，那就直接变成了可执行程序，所以没有想到如何实现这种语言的转换，所以猜测的可能是通过automatic code generation，但是还并不了解这个自动代码生成的机制，不过用户透明的代码自动转换机制肯定是通过某种机制来实现的

编译层的作用可能是实现高层次代码转换为中间表达IR，逐层下降逐层优化

根据项目结构图，我对于目前任务的理解是，有三项任务，一是实现统一编程框架、二是实现dgl到统一编程框架的用户无感转换、三是实现统一编程框架的编译层。
对于前两项任务的理解，在项目图中，图学习部分在应用层和框架层之间的联系有两点，一是直接利用统一编程框架，二是首先利用dgl，然后dgl转换到统一编程框架。

所以说我的疑问就是：
1. 在统一编程框架中，和图学习相关的组件应当有哪些，采用什么样的项目模式（对图学习本身就不太清楚，都还不知道图学习都包含什么部分）
2. dgl到统一编程框架的用户无感转换，这个转换依据的是什么，或者说这种转换机制是什么
3. 从框架层到编译层，MLIR是如何发挥的作用，编译层的设计目标是什么

## 如何理解 MLIR
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403110950934.png)

中间表达的意义在于逐层解析，逐层降低抽象而偏向硬件。高级语言的 AST 到 IR 之间存在较大差距，通过在中间架设 IR 可以对高级抽象进行渐进式变换和递降，降低这种差距变化的梯度，降低阶段转换的难度，同时只要提供 IR 就相当于我们可以为任何领域的代码实现设计特定的编译器。

但是带来的一个问题就是，每当增加一种实现时都会出现一种全新的 IR，而实际上可能这些 IR 之间存在一些共性的东西，通过增设 MLIR 这一层类似标准化的层次，可以使得中间的转化流程变得更加规范

根据High Performance GPU Code Generation for Matrix-Matrix Multiplication using MLIR: Some Early Results[^MLIR-Usage]的说法，原有的IR基础设施并不能有效地解决自动生成特定领域库的问题。特别是，很难使用单个IR来表示和转换高，中，低级别的抽象。

[^MLIR-Usage]: [High Performance GPU Code Generation for Matrix-Matrix Multiplication using MLIR: Some Early Results](https://arxiv.org/abs/2108.13191)

MLIR 用于解决编程语言、编译器和硬件之间的交互问题，它的出现是为了应对日益复杂的编程语言和硬件架构

MLIR提供了一个统一的中间表示（IR），可以作为不同编程语言编译器和LLVM后端之间的桥梁。

通过自定义 dialect，可以实现语言扩充 和 实现特定领域的编译优化

## CGA_Framework Project Structure

Graph Generalization:

|   组件   |   包含   |   功能   |
| ---- | ---- | ---- |
|   gc(graph computing)   |   GcComp(graph convolution component), GcProg(graph convolution program), GraphC(graph convolution)   |      |
|   gnn(graph neural network)   |      |   图神经网络   |
|   gpm(graph partitioning method)   |      |   图划分   |
|   mat(matrix)   |      |   各种矩阵格式表示以及矩阵运算   |
|   utils   |      |   小工具   |
|   vec   |      |   像是容器类，会管理实际的数据   |
|   converter.h   |   -   |   数据加载   |


GNN:

共给出了3种图神经网络模型：

- GAT（Graph Attention Network）
- GCN（Graph Convolutional Network）：
- GIN（Graph Isomorphism Network）

在具体的实现上，三种神经网络模型都实现了2个类，分别继承 GnnComp 和 GnnProg


SSA 意为静态单赋值（Static Single Assignment）。SSA 是一种中间表示（IR）的形式，常用于编译器优化和静态分析中。在 SSA 中，每个变量在其定义处只能被赋值一次，并且每个变量必须在使用之前被明确地赋值。这意味着在 SSA 中，变量的赋值是静态的，因此称为“静态单赋值”。

## Reference
> [从零开始学习深度学习编译器](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA4MjY4NTk0NQ==&action=getalbum&album_id=2099721001268740096&scene=173&subscene=&sessionid=svr_76259f2a30f&enterid=1709891420&from_msgid=2247499828&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
