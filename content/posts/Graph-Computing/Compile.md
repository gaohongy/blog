---
title: Compile
subtitle:
description:
keywords:
summary:
license:
date: 2024-03-07T21:26:29+08:00
lastmod: 2024-03-24T23:16:53+08:00
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

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403191134543.png)

### Chapter 1
> Toy Language and AST

这一部分主要内容就是创造了一种新的语言，叫做Toy，然后实现了一个简易的语法分析器，仅仅能够获得toy源程序对应的抽象语法树。所以说这一节的重点就是抽象语法树

需要使用到的命令：`~/llvm/build/bin/toyc-ch1 /Users/gaohongyu/llvm/mlir/test/Examples/Toy/Ch1/ast.toy --emit=ast`

意思是通过`toyc-ch1`程序生成`ast.toy`程序的抽象语法树

### Chapter 2

> Emitting Basic MLIR

在第一节中已经生成了 Toy 语言源程序的 AST，这一节就是要根据 AST 结合 Dialect 来生成 MLIR表达式

![](https://mmbiz.qpic.cn/mmbiz_png/SdQCib1UzF3tN9fRfXZhWRgL2OLr400ESibMbgibPJfUrSLDicq855g64h5cz6CHn4lstoRPJ2KjGbG2q43ANqSPmg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

Ch2/toyc.cpp 相较于 Ch1/toy.cpp相比，有一个区别就是程序接收的参数多了一个：

这是Ch1/toy.cpp中和程序参数相关的内容：
```cpp
static cl::opt<enum Action>
    emitAction("emit", cl::desc("Select the kind of output desired"),
               cl::values(clEnumValN(DumpAST, "ast", "output the AST dump")));
```

这是Ch2/toy.cpp中和程序参数相关的内容：
```cpp
static cl::opt<enum Action> emitAction(
    "emit", cl::desc("Select the kind of output desired"),
    cl::values(clEnumValN(DumpAST, "ast", "output the AST dump")),
    cl::values(clEnumValN(DumpMLIR, "mlir", "output the MLIR dump")));
```

区别就在于编译生成的toyc-ch2不仅能够生成toy源程序的抽象语法树，还能够生成对应的mlir表达式

#### 关于 Ops.td 是如何同 Dialect模块（Dialect.h，Dialect.cpp）相结合的？**

通过Ops.td可以生成以下这些.inc文件:

1. Dialect.h.inc: Dialect Declarations
2. Dialect.cpp.inc: Dialect Definitions
3. Ops.h.inc: Op Declarations 
4. Ops.cpp.inc: Op Definitions

而 Dialect 模块则使用到了这些文件：

1. Dialect.h 负责引入 .h.inc，即 Declarations, 包括 Dialect.h.inc 和 Ops.h.inc
2. Dialect.cpp 负责引入 .cpp.inc，即 Definitions，包括 Dialect.cpp.inc 和 Ops.cpp.inc

所以说，关于 dialect 本身 和 其 操作 等内容，都是在.td文件中说明，然后通过 tablegen 工具结合选项生成所需要的内容

Dialect.h 负责引入头文件这件事情并不难理解，那么如何理解 Dialect.cpp 和 Ops.cpp.inc 都具备的对于 Op Definition 功能这件事情？

以 `TransposeOp::build` 为例:

Ops.cpp.inc 中的相关内容为：

```cpp
void TransposeOp::build(::mlir::OpBuilder &odsBuilder, ::mlir::OperationState &odsState, ::mlir::Type resultType0, ::mlir::Value input) {
  odsState.addOperands(input);
  odsState.addTypes(resultType0);
}

void TransposeOp::build(::mlir::OpBuilder &odsBuilder, ::mlir::OperationState &odsState, ::mlir::TypeRange resultTypes, ::mlir::Value input) {
  odsState.addOperands(input);
  assert(resultTypes.size() == 1u && "mismatched number of results");
  odsState.addTypes(resultTypes);
}

void TransposeOp::build(::mlir::OpBuilder &, ::mlir::OperationState &odsState, ::mlir::TypeRange resultTypes, ::mlir::ValueRange operands, ::llvm::ArrayRef<::mlir::NamedAttribute> attributes) {
  assert(operands.size() == 1u && "mismatched number of parameters");
  odsState.addOperands(operands);
  odsState.addAttributes(attributes);
  assert(resultTypes.size() == 1u && "mismatched number of return types");
  odsState.addTypes(resultTypes);
}
```

Dialect.cpp 中的相关内容为：

```cpp
void TransposeOp::build(mlir::OpBuilder &builder, mlir::OperationState &state,
                        mlir::Value value) {
  state.addTypes(UnrankedTensorType::get(builder.getF64Type()));
  state.addOperands(value);
}
```

目前还未搞懂这两者之间有何关联，不过 Ops.cpp.inc 还有额外的工作，就是在 Dialect.cpp 最开始的 dialect 初始化时为 dialect 指定操作：

```cpp
void ToyDialect::initialize() {
  addOperations<
#define GET_OP_LIST
#include "toy/Ops.cpp.inc"
      >();
}
```

#### 关于创建一个 Op 所需要用到的类

以 Transpose Op 为例，根据 Ops.cpp.inc 的内容，和 transpose op 相关的有以下内容: 

1. TransposeOpGenericAdaptorBase
2. TransposeOpGenericAdaptor
3. TransposeOpAdaptor
4. TransposeOp

#### 随想疑问
一直在思考，toy tutorial 给出的这个示例到底对于实际的应用场景有何启发作用？

实际上toy是一门新创造出的语言，是为了展示MLIR的各项特点而出现的，获取抽象语法树，获取MLIR表达式的工具都是项目自行实现的。

如果考虑真实的场景，例如用C++实现的代码，哪怕是框架，通过cmake都可以完成编译，引入MLIR难道就是为了替换掉整个编译流程中的前端和中端，用人工实现MLIR的方式逐层降级？首先编译层的上层是框架层，在我的理解中，我们可以把这个框架和Tensorflow或者Pytorch等价来看，但是在我看来，无论是不是新的框架，只要不是新的语言，理论上都可以用现有的编译工具完成编译，非要做这个编译层，非要用MLIR逐层降级只是为了提高性能，优化性能，使得每一层的优化都能够做到极致，而不是仅仅依靠现有编译器采用的那些优化。

那么有一个问题是：我们要站在哪个起点来分析这件事。toy是一门新的语言，所以说最基础的解析器都需要全新实现，也就是说面对toy这个场景就应当是一无所有的。但是现有项目是用C++实现的，MLIRGen这个工具对于现在的场景是存在的，肯定需要做的是写Dialect，那结合Dialect生成MLIR表达式这个过程是谁负责的？

突然感觉我们的重点就是写Dialect，结合编程框架中的一些数据结构，在Dialect中把需要完成的内容完成，考虑一层一层是怎么降级的。我觉得像：

1.是怎么从源程序解析得到的AST？(这个问题不考虑，暂时不重要）
2.MLIRGen是怎么结合Dialect生成的一层表达（每一次Dialect的加入都会形成一层中间层），主要是类和类之间是怎么关联起来的
3. .td文件通过tablegen能生成很多东西，这些东西都怎么用？ 
4. .td结合tablegen生成的一大堆东西 和 Dialect.cpp又是什么关系，或者说Dialect.cpp到底负责干什么的？

1. 我现在比较纠结的就是各个部分的联动关系，我不知道要写的内容到底要怎么发挥出它的作用
2. 不太理解如果想要完成一层降级都需要做些什么，只知道要写dialect，但是这dialect是怎么对降级起到作用的？通过生成MLIR表达式或许是，所以说就是让dialect为生成MLIR表达式做出贡献

如果对toy ch2这个内容来说，是否可以假设现在有MLIRGen的工具，但是还没有相关的dialect，任务就是编写dialect实现MLIR表达式的生成。（因为后续教程的内容是例如表达式优化和降级的内容，如果让dialect在生成MLIR表达式这个过程中发挥作用我觉得是整个任务的基础）

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

#### Chapter 2 展现出的问题
以下是通过 `toyc-ch2 codegen.toy --emit=mlir`生成的 mlir表达式

```
module {
  toy.func @multiply_transpose(%arg0: tensor<*xf64> loc("codegen.toy":4:1), %arg1: tensor<*xf64> loc("codegen.toy":4:1)) -> tensor<*xf64> {
    %0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64> loc("codegen.toy":5:10)
    %1 = toy.transpose(%arg1 : tensor<*xf64>) to tensor<*xf64> loc("codegen.toy":5:25)
    %2 = toy.mul %0, %1 : tensor<*xf64> loc("codegen.toy":5:25)
    toy.return %2 : tensor<*xf64> loc("codegen.toy":5:3)
  } loc("codegen.toy":4:1)
  toy.func @main() {
    %0 = toy.constant dense<[[1.000000e+00, 2.000000e+00, 3.000000e+00], [4.000000e+00, 5.000000e+00, 6.000000e+00]]> : tensor<2x3xf64> loc("codegen.toy":9:17)
    %1 = toy.reshape(%0 : tensor<2x3xf64>) to tensor<2x3xf64> loc("codegen.toy":9:3)
    %2 = toy.constant dense<[1.000000e+00, 2.000000e+00, 3.000000e+00, 4.000000e+00, 5.000000e+00, 6.000000e+00]> : tensor<6xf64> loc("codegen.toy":10:17)
    %3 = toy.reshape(%2 : tensor<6xf64>) to tensor<2x3xf64> loc("codegen.toy":10:3)
    %4 = toy.generic_call @multiply_transpose(%1, %3) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64> loc("codegen.toy":11:11)
    %5 = toy.generic_call @multiply_transpose(%3, %1) : (tensor<2x3xf64>, tensor<2x3xf64>) -> tensor<*xf64> loc("codegen.toy":12:11)
    toy.print %5 : tensor<*xf64> loc("codegen.toy":13:3)
    toy.return loc("codegen.toy":8:1)
  } loc("codegen.toy":8:1)
} loc(unknown)
```

可以看出这个层次下的 mlir表达式 是严格按照代码逐句翻译后的结果，例如下面两句是 源代码 和 对应的mlir表达式，实际上从 tensor<*xf64> transpose到 tensor<*xf64> 显然是没有意义的，这是一步冗余的操作，所以 Chapter 3 就着手通过 表达式优化 来解决这个问题

```
var a<2, 3> = [[1, 2, 3], [4, 5, 6]];

%0 = toy.transpose(%arg0 : tensor<*xf64>) to tensor<*xf64> loc("codegen.toy":5:10)
```

### Chapter 3

> High-level Language-Specific Analysis and Transformation

关于表达式变形消除冗余在整个流程中的位置：

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403191137970.png)

Chapter 3 的展开逻辑本质上是遵循了 Chapter 2 遗留下来的问题，不过其重新提出了一个 `transpose(transpose(X)) -> X` 的问题，讲述如何解决这个问题。需要注意的是本节一共举了为两个操作添加优化方法的示例，它们采用了不同的方法， transpose操作 采用了 C++ 实现(对应ToyCombine.cpp)，reshape操作 采用了 DRR模块(对应ToyCombine.td)

这一节中，共涉及到整体架构的三部分；
![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403201132203.png)

这一节对应到的是 MLIR 的 Pattern Rewrite机制，用于对IR做一些通用变换优化，还负责Op的规范化以及Dialect间以及Dialect内部的Op转换，我感觉就是架构图中提到的 transformation 和 canonicalization

#### C++ 实现 canonicalization

表达式的变形优化 在 MLIR 的场景下被称为 rewrite

想要实现 rewrite 需要进行以下几点：

1. 实现一个 RewritePattern，即一个继承了`mlir::OpRewritePattern<>`的类

```cpp
struct SimplifyRedundantTranspose : public mlir::OpRewritePattern<TransposeOp>
```

> 当继承这个`OpRewritePattern`类后，需要重写`matchAndRewri`方法

2. rewrite能够被应用，需要通过 canonicalization pass。需要做的工作一方面是让 规范化器知道有这个 rewrite 的存在，另一方面就是让其运用上这一 rewrite。所以这一步就是实现注册工作

```cpp
void TransposeOp::getCanonicalizationPatterns(RewritePatternSet &results,
                                              MLIRContext *context) {
  results.add<SimplifyRedundantTranspose>(context);
}
```

不过有一点疑问是，这里只是类方法的定义，并没有找到对此方法的调用，那么是什么时候真正执行的注册?

关于这个问题，官方文档中有提到一个 canonicalization framework，虽然在第2点中说注册是为了让 规范化器 知道有这个rewrite的存在，但是本质上可能是为了让这个 优化框架 了解到这个rewrite的存在。有一种可能就是只要我们为一个操作定义了`getCanonicalizationPatterns`方法，那么 MLIR 在面对这个操作类的时候，就一定能检测到它是具备这个优化方法的，那么在下一步就能够真正实现这种优化。

> 真实的实现机制需要去看 MLIR 实现的源码是怎么处理的，我们暂且可以认为这样做就是满足 MLIR对于实现一个方法的优化 所做出的规定，也就是说 MLIR 提供了一种标准化的模版，我们只需要按照模版的要求去做，那么就可以为这个操作实现这种优化（实际上 Chapter 4 给出了支持这种看法的证据，详见Chapter 4）

3. 规范化器 的 代码原型 就是 `mlir::PassManager`，即我们需要利用它来真正运用上第一步实现的 rewrite

```cpp
mlir::PassManager pm(module.get()->getName());
pm.addNestedPass<mlir::toy::FuncOp>(mlir::createCanonicalizerPass());
```

#### 使用 DRR模块 实现 transformation

相较于C++实现，使用 DRR模块 的优点似乎就在于省去了后两步的注册和应用到规范化器的两项操作，只需要实现一个继承关系即可

```cpp
class Pattern<
    dag sourcePattern, list<dag> resultPatterns,
    list<dag> additionalConstraints = [],
    dag benefitsAdded = (addBenefit 0)>;

def ReshapeReshapeOptPattern : Pat<(ReshapeOp(ReshapeOp $arg)),
                                   (ReshapeOp $arg)>;
``` 

不过教程中还额外给出了另外两种实现方式，那两种只是说在基础之上，额外添加了其他信息：

1. 增加了参数限制：只需要实现一个`Constraint`类的子类，并且将该子类在创建模式时添加到模版参数之中
2. 当限制条件比较复杂时，可以通过实现一个`NativeCodeCall`的子类添加原生的C++语句实现，同样在创建模式时将其添加到模版参数之中

#### 对 Chapter 3 的理解和感悟

这样来说，是不是可以理解为 rewrite 就是一种 pass

通过这一部分内容，更加加深了对于 MLIR 似乎就和 SpringBoot 那种东西一样，我们似乎完全可以把他们当作普通的框架，为操作添加优化就是必须要为操作类定义`getCanonicalizationPatterns`方法，这就好像在其他那些框架中所要求做的类似


#### compiler pattern-match transformations 分类
1. local

有两种方法用于实现transformation：

- Imperative, C++ pattern-match and rewrite
- Declarative, rule-based pattern-match and rewrite(Declarative Rewrite Rules (DRR)模块)(此时要求之前的op定义是通过ods模块实现的，否则只能采用C++代码手动实现transformation了)

到这里，架构图中涉及到的两个模块就都出现了

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403191146250.png)

2. global

#### 关于 Pattern Rewrite机制 的疑问

在这个机制下，存在一些名称极为类似暂时还不清楚用途的类，例如
- `RewriterBase`
- `PatternRewriter`

以上两者关系见

[![](https://mlir.llvm.org/doxygen/classmlir_1_1RewriterBase__inherit__graph.png)](https://mlir.llvm.org/doxygen/classmlir_1_1RewriterBase.html#details)

- `RewritePattern`
- `OpRewritePattern`

上面这 4 个类是不同的，区别在于前者核心是 rewriter，后者核心是 pattern

这两个 pattern 之间的继承关系如下：

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403211147117.png)


### Chapter 4

> Enabling Generic Transformation with Interfaces


在 Chapter 3 的 C++ 实现 transformation 一节中留下了一个疑问，即在给一个 operation 应用一种 模式patter，或者说希望重写它时，我们是定义了`TransposeOp::getCanonicalizationPatterns`这一方法，当时我们疑问在于代码中只是定义了但是并没有在任何位置看到调用它的代码，那么其是如何发挥作用的？

在 Chapter 4 开头给出，`getCanonicalizationPatterns` 实际是一个钩子函数，作用于operations之上，通过钩子函数实现为 operation 添加额外功能的方法并不难理解，这也是软件开发中常见的一种方法

不过钩子函数从何而来，是需要我们自行创建（如果是这样的话，又需要在哪个环节进行创建），还是 MLIR 本身对于 operation 就声明了一些固定的钩子函数？

关于这个问题，我们进行了如下的探究：

1. 关于 `TransposeOp`，源头应当位于 `Ops.td`，所以我们回到这一文件

文件中主要包含 2 部分内容，一是对于 `Dialect` 的说明，二是对于 operation 的说明

对于 `Dialect` 的说明：
```
def Toy_Dialect : Dialect {
  let name = "toy";
  let cppNamespace = "::mlir::toy";
}
```

对于 operation 的说明：
```
class Toy_Op<string mnemonic, list<Trait> traits = []> :
    Op<Toy_Dialect, mnemonic, traits>;

def TransposeOp : Toy_Op<"transpose">{

}
```

由此可见，关于 operation 的继承关系是 TransposeOp -> Toy_Op -> Op

2. 由于 td 只是声明性语言，并不代码最终的代码实现，所以我们对 cpp 文件下的实际的 TransposeOp类 进行了追踪，发现其实际声明位于 Ops.h.inc，同时声明内容为

```cpp
class TransposeOp : public ::mlir::Op<> {
  static void getCanonicalizationPatterns(::mlir::RewritePatternSet &results, ::mlir::MLIRContext *context);
}
```

由于 Ops.h.inc 是有 Ops.td 生成的，但是在 Ops.td 中并没有显示给出 getCanonicalizationPatterns 的声明，说明这一方法应当是通过继承关系得到的(在llvm源码中，确实有找到 和 getCanonicalizationPatterns 相关的内容，不过还未完全分析透彻逻辑关系）,说明只要是继承了 mlir::Op 的子类，都应当是能够使用这个钩子函数的

官方定义的毕竟是有限且不够灵活的，所以 MLIR 就提出了 interface 的概念，用户可以自定义实现各种钩子函数（感觉这就是一个组件化工具正常的发展流程，官方提供一些，然后官方开放接口，第三方可自行实现接入）

本 Chapter 提及的也是 Dialect 的一个子部分

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403201134960.png)

引入的情景是 tensor 的形状推导

目前感觉 Chapter 4 所介绍的 Interface 只是对于 Chapter 3 所介绍的用于表达式优化的 pattern 在功能性上的一种扩充，所以说并不需要把其当作一种新的技术点

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

1. 在 Toy Tutorial Chapter 3 给出了 基于DRR模块实现 transformation，在 td中声明 Pat 的子类。但是/Users/gaohongyu/Graduate/Graph-Calculation/project/compiler/compiler/include/compiler/Dialect/Graph/Transforms/Passes.td 都是继承 Pass 的子类，暂时还不清楚继承 Pass 的子类作用

两层IR表示 Graph IR 和 Matrix IR，对应 IR 分别在 `include/compiler/Dialect/Graph` 和 `include/compiler/Dialect/Matrix`

1. 完整表示 IR
2. 自动编译优化（表达式优化）
3. 图-矩阵转换（lowering），最后要转到 LLVM IR

[GraphOps.td 结构](https://www.mubu.com/doc/CuigbBIo7Q)

首先说，lib 存放的就是对各种东西的定义，都是各种实现。
`lib/Dialect/Interfaces/` 这里虽然说给起的名字是 interface，但是目前的代码中并没有使用到 MLIR interface 的代码，以 For.cpp 为例，
主要内就 2 项：

1. 对于 operation of For 的定义
2. 表达式优化，重写，代码层面即为 继承 `OpRewritePattern` 类的子类实现


看代码过程中遇到一个新的概念 SSA，Static Single Assignment，即静态单变量。编译器会将同名变量分解为两个变量，便于进行各类优化


2024/03/21组会疑问解答；

1. 关于 统一编程框架 和 编译层 之间的联动关系，是谁决定谁，可以把编译层那个东西看为是一个第三方的库，为统一编程框架提供服务
2. 之前疑问的是编译层需要对图操作进行描述，怎么知道图操作有什么，或者说编译层都需要写哪些东西？这个应该要和上一个问题联系一下，既然是实现第三方库，那么所实现的操作就应当是尽可能全面的，不是说统一编程框架需要什么编译层提供什么，而是说统一编程框架要跟着编译层走，所以我理解着就得尽可能多想一下有哪些需要的操作


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

## buddy-mlir

### Build

在第一步构建过程中遇到几个坑点：

1. The target building platform of MLIR is uncompleted，because [MLIR Getting Started](https://mlir.llvm.org/getting_started/) asks that we can use the build option `-DLLVM_TARGETS_TO_BUILD="Native;NVPTX;AMDGPU"`, but the buddy-mlir needs the `RISCV` platform data, so we need to recompile the MLIR

2. I learn about the process of building buddy-compiler needs the mlir compiling data from the option `-DMLIR_DIR=$PWD/../llvm/build/lib/cmake/mlir` and `-DLLVM_DIR=$PWD/../llvm/build/lib/cmake/llvm`, so I think a idea that create a soft link of llvm project for the buddy-compiler. But when I build the buddy-compiler, I get the error message that cmake cannot find some files about RISCV, until I get some tips from the slack 

![](https://img2024.cnblogs.com/blog/1898659/202403/1898659-20240324100837179-389572672.png)

执行之后就能够正常build了，但是里面不确定的因素有二，其一是 我手动创建软链接确实也起到了增加submodule的功能，因为执行 git submodule update --init 之后并没有重复 git clone llvm- project，但是不太清楚执行之后到底产生了什么其他的额外影响(主要怀疑会不会执行后修改了什么变量），使得 cmake 就能够找到相关文件了；其二是 submodule 并不仅仅只有 llvm 这一个，还存在另外一个，不确定是不是因为之前缺少这个子模块从而导致build失败（不过这一点是可以验证的，只需要注释掉这部分，只 git submodule llvm-project 那部分，查看是否可以完成 build 就可以，按理说应该不太行）

### Structure

![](https://cdn.jsdelivr.net/gh/gaohongy/cloudImages@master/202403241200921.png)

目前的困惑在于 buddy-mlir 是否可以看为是对 mlir 的一种封装，如果是的话，封装了啥，如果不是，那它相较于 mlir 又有何区别或者说设计的意义在哪里

## Reference
> [从零开始学习深度学习编译器](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzA4MjY4NTk0NQ==&action=getalbum&album_id=2099721001268740096&scene=173&subscene=&sessionid=svr_76259f2a30f&enterid=1709891420&from_msgid=2247499828&from_itemidx=1&count=3&nolastread=1#wechat_redirect)
> [深度学习编译器资料总结](https://github.com/BBuf/tvm_mlir_learn)
