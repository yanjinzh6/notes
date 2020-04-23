---
title: Protocol Buffer proto 3 指南
date: 2020-01-17 16:00:00
tags: 'Proto3'
categories:
  - ['开发', 'Protocol']
permalink: protocol-buffer-development-guide
---

## 简介

> 本文是对中文文档的整理, 大部分内容都是引用其他一些作者的优质翻译, 需要进行更新优化

这篇指南描述如何使用 protocol buffer 语言来组织你的 protocol buffer 数据, 包括 `.proto` 文件的语法规则以及如何通过 `.proto` 文件来生成数据访问类代码.

<!-- more -->

## Defining A Message Type (定义一个消息类型)

```
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

语法说明

- `syntax` 前只能是空行或者注释
- 每个字段由字段限制, 字段类型, 字段名和编号四部分组成

### Specifying Field Types (指定字段类型)

在上面的例子中, 该消息定义了三个字段, 两个 `int32` 类型和一个 `string` 类型的字段

### Assigning Tags (赋予编号)

消息中的每一个字段都有一个独一无二的数值类型的编号. 1 到 15 使用一个字节编码, 16 到 2047 使用 2 个字节编码, 所以应该将编号 1 到 15 留给频繁使用的字段.
可以指定的最小的编号为 1, 最大为 2^{29}-1 或 536, 870, 911. 但是不能使用 19000 到 19999 之间的值, 这些值是预留给 protocol buffer 的.

### Specifying Field Rules (指定字段限制)

- `required`: 必须赋值的字段
- `optional`: 可有可无的字段
- `repeated`: 可重复字段 (变长字段)

### Adding More Message Types (添加更多消息类型)

一个 `.proto` 文件可以定义多个消息类型:

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

### Adding Comments (添加注释)

`.proto` 文件也使用 `C/C++` 风格的注释语法 `//`

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

### What's Generated From Your `.proto`？ (编译 `.proto` 文件)

对于 C++, 每一个 `.proto` 文件经过编译之后都会对应的生成一个 .h 和一个 .cc 文件.

## Scalar Value Types (类型对照表)

| `.proto` Type | Notes                                                                                                                                           | C++ Type |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | -------- |
| double      | double                                                                                                                                          | double   |
| float       | float                                                                                                                                           | float    |
| int32       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint32 instead. | int32    |
| int64       | Uses variable-length encoding. Inefficient for encoding negative numbers – if your field is likely to have negative values, use sint64 instead. | int64    |
| uint32      | Uses variable-length encoding.                                                                                                                  | uint32   |
| uint64      | Uses variable-length encoding.                                                                                                                  | uint64   |
| sint32      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int32s.                            | int32    |
| sint64      | Uses variable-length encoding. Signed int value. These more efficiently encode negative numbers than regular int64s.                            | int64    |
| fixed32     | Always four bytes. More efficient than uint32 if values are often greater than 2^28                                                             | uint32   |
| fixed64     | Always eight bytes. More efficient than uint64 if values are often greater than 2^56                                                            | uint64   |
| sfixed32    | Always four bytes.                                                                                                                              | int32    |
| sfixed64    | Always eight bytes.                                                                                                                             | int64    |
| bool        | bool                                                                                                                                            | boolean  |
| string      | A string must always contain UTF-8 encoded or 7-bit ASCII text.                                                                                 | string   |
| bytes       | May contain any arbitrary sequence of bytes.                                                                                                    | string   |

## Default Values (缺省值)

如果没有指定默认值,则会使用系统默认值,

对于 string 默认值为空字符串,
对于 bytes 默认值为空字符,
对于 bool 默认值为 false,
对于数值类型默认值为 0,
对于 enum 默认值为定义中的第一个元素,
对于 repeated 默认值为空.

## Enumerations (枚举)

```
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

通过设置可选参数 allow_alias 为 true, 就可以在枚举结构中使用别名 (两个值元素值相同)

```
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

由于枚举值采用 varint 编码, 所以为了提高效率, 不建议枚举值取负数. 这些枚举值可以在其他消息定义中重复使用.

### Reserved Fields (预留字段)

如果通过完全删除枚举或将其注释掉来更新枚举类型, 则以后进行更新时可以重用数值, 如果与旧版本的 proto 文件一起加载, 便会导致冲突, 包括数据损坏, 隐私错误等. 一种避免此类问题的方法就是指明这些删除的字段是保留的. 如果有用户使用这些字段的编号, protocol buffer 编译器会发出告警. 可以使用 to max 关键字指定保留数值范围到最大值

```
enum Foo {
  reserved 2, 15, 9 to 11, 40 to max;
  reserved "FOO", "BAR";
}
```

## Using Other Message Types (使用其他消息类型)

可以使用一个消息的定义作为另一个消息的字段类型.

```
message SearchResponse {
  repeated Result results = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

### Importing Definitions (导入定义)

就像 `C++` 的头文件一样, 你还可以导入其他的 `.proto` 文件

```
import "myproject/other_protos.proto";
```

如果想要移动一个 `.proto` 文件, 但是又不想修改项目中 `import` 部分的代码, 可以在文件原先位置留一个空 `.proto` 文件, 然后使用 `import public` 导入文件移动后的新位置:

```
// new.proto
// All definitions are moved here
// old.proto
// This is the proto that all clients are importing.
import public "new.proto";
import "other.proto";
// client.proto
import "old.proto";
// You use definitions from old.proto and new.proto, but not other.proto
```

### 使用 proto2 消息类型

可以导入 proto2 消息类型并在 proto3 消息中使用, 反之亦然, 但是不能在 proto3 语法中直接使用 proto2 的枚举

## Nested Types (嵌套类型)

在 protocol 中可以定义如下的嵌套类型

```
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result results = 1;
}
```

如果在另外一个消息中需要使用 `Result` 定义, 则可以通过 `Parent.Type` 来使用.

```
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

protocol 支持更深层次的嵌套和分组嵌套, 但是为了结构清晰起见, 不建议使用过深层次的嵌套.

```
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

## Updating A Message Type (更新一个数据类型)

在实际的开发中会存在这样一种应用场景, 既消息格式因为某些需求的变化而不得不进行必要的升级, 但是有些使用原有消息格式的应用程序暂时又不能被立刻升级, 这便要求我们在升级消息格式时要遵守一定的规则, 从而可以保证基于新老消息格式的新老程序同时运行. 规则如下:

- 不要修改已经存在字段的标签号.
- 任何新添加的字段必须是 `optional` 和 `repeated` 限定符, 否则无法保证新老程序在互相传递消息时的消息兼容性.
- 在原有的消息中, 不能移除已经存在的 `required` 字段, `optional` 和 `repeated` 类型的字段可以被移除, 但是他们之前使用的标签号必须被保留, 不能被新的字段重用.
- `int32`, `uint32`, `int64`, `uint64` 和 `bool` 等类型之间是兼容的, `sint32` 和 `sint64` 是兼容的, `string` 和 `bytes` 是兼容的, `fixed32` 和 `sfixed32`, 以及 `fixed64` 和 `sfixed64` 之间是兼容的, 这意味着如果想修改原有字段的类型时, 为了保证兼容性, 只能将其修改为与其原有类型兼容的类型, 否则就将打破新老消息格式的兼容性.
- `optional` 和 `repeated` 限定符也是相互兼容的.

## 未知字段

## Any (任意消息类型)

`Any` 类型是一种不需要在 `.proto` 文件中定义就可以直接使用的消息类型, 使用前 `import google/protobuf/any.proto` 文件即可.

```
import  "google/protobuf/any.proto";

message ErrorStatus  {
  string message =  1;
  repeated google.protobuf.Any details =  2;
}
```

`C++` 使用 `PackFrom()` 和 `UnpackTo()` 方法来打包和解包 `Any` 类型消息.

```
// Storing an arbitrary message type in Any.
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// Reading an arbitrary message from Any.
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.Is<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

## Oneof (其中一个字段类型)

有点类似 `C++` 中的联合, 就是消息中的多个字段类型在同一时刻只有一个字段会被使用, 使用 `case()` 或 `WhichOneof()` 方法来检测哪个字段被使用了.

### Using Oneof (使用 Oneof)

```
message SampleMessage  {
  oneof test_oneof {
    string name =  4;
    SubMessage sub_message =  9;
  }
}
```

你可以添加除 `repeated` 外任意类型的字段到 `Oneof` 定义中

### Oneof Features (Oneof 特性)

`oneof` 字段只有最后被设置的字段才有效, 即后面的 `set` 操作会覆盖前面的 `set` 操作

```
SampleMessage message;
message.set_name("name");
CHECK(message.has_name());
message.mutable_sub_message();   // Will clear name field.
CHECK(!message.has_name());
```

- `oneof` 不可以是 `repeated` 的
- 反射 API 可以作用于 `oneof` 字段
- 如果使用 `C++` 要防止内存泄露, 即后面的 `set` 操作会覆盖之前的 `set` 操作, 导致前面设置的字段对象发生析构, 要注意字段对象的指针操作

```
SampleMessage message;
SubMessage* sub_message = message.mutable_sub_message();
message.set_name("name");      // Will delete sub_message
sub_message->set_...            // Crashes her
```

如果使用 `C++` 的 `Swap()` 方法交换两条 `oneof` 消息, 两条消息都不会保存之前的字段

```
SampleMessage msg1;
msg1.set_name("name");
SampleMessage msg2;
msg2.mutable_sub_message();
msg1.swap(&msg2);
CHECK(msg1.has_sub_message());
CHECK(msg2.has_name());
```

### Backwards-compatibility issues (向后兼容)

添加或删除 `oneof` 字段的时候要注意, 如果检测到 `oneof` 字段的返回值是 `None/NOT_SET`, 这意味着 `oneof` 没有被设置或者设置了一个不同版本的 `oneof` 的字段, 但是没有办法能够区分这两种情况, 因为没有办法确认一个未知的字段是否是一个 `oneof` 的成员.

#### Tag Reuse Issues (编号复用问题)

- 删除或添加字段到 `oneof`: 在消息序列化或解析后会丢失一些信息, 一些字段将被清空
- 删除一个字段然后重新添加: 在消息序列化或解析后会清除当前设置的 `oneof` 字段
- 分割或合并字段: 同普通的删除字段操作

## Maps (表映射)

protocol buffers 提供了简介的语法来实现 `map` 类型:

```
map<key_type, value_type> map_field = N;
```

`key_type` 可以是除浮点指针或 `bytes` 外的其他基本类型, `value_type` 可以是任意类型

```
map<string,  Project> projects =  3;
```

- `Map` 的字段不可以是重复的 (`repeated`)
- 线性顺序和 `map` 值的的迭代顺序是未定义的, 所以不能期待 `map` 的元素是有序的
- `maps` 可以通过 `key` 来排序, 数值类型的 `key` 通过比较数值进行排序
- 线性解析或者合并的时候, 如果出现重复的 `key` 值, 最后一个 `key` 将被使用. 从文本格式来解析 `map`, 如果出现重复 `key` 值则解析失败.

### Backwards compatibility (向后兼容)

`map` 语法下面的表达方式在线性上是等价的, 所以即使 protocol buffers 没有实现 `maps` 数据结构也不会影响数据的处理:

```
message MapFieldEntry  {
  key_type key =  1;
  value_type value =  2;
}
repeated MapFieldEntry map_field = N;
```

## 包

类似 `C++` 的命名空间, 用来防止名称冲突

```
package foo.bar;
message Open { ... }
```

你可以使用包说明符来定义你的消息字段:

```
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

## 定义服务

如果想在 RPC 系统中使用消息类型, 就需要在 `.proto` 文件中定义 RPC 服务接口, 然后使用编译器生成对应语言的存根.

```
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

## JSON 映射

Proto3 支持 JSON 格式的编码. 编码后的 JSON 数据的如果没有值或值为空, 解析时 protocol buffer 将会使用默认值, 在对 JSON 编码时可以节省空间.

| proto3                 | JSON          | JSON example                            | Notes                                                                                                                                                                                                                                                           |
| ---------------------- | ------------- | --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| message                | object        | {"fBar": v, "g": null, …}               | Generates JSON objects. Message field names are mapped to lowerCamelCase and become JSON object keys. null is accepted and treated as the default value of the corresponding field type.                                                                        |
| enum                   | string        | "FOO_BAR"                               | The name of the enum value as specified in proto is used.                                                                                                                                                                                                       |
| map< K,V>              | object        | {"k": v, …}                             | All keys are converted to strings.                                                                                                                                                                                                                              |
| repeated V             | array         | [v, …]                                  | null is accepted as the empty list [].                                                                                                                                                                                                                          |
| bool                   | true, false   | true, false                             |
| string                 | string        | "Hello World!"                          |
| bytes                  | base64 string | "YWJjMTIzIT8kKiYoKSctPUB+"              |
| int32, fixed32, uint32 | number        | 1, -10, 0                               | JSON value will be a decimal number. Either numbers or strings are accepted.                                                                                                                                                                                    |
| int64, fixed64, uint64 | string        | "1", "-10"                              | JSON value will be a decimal string. Either numbers or strings are accepted.                                                                                                                                                                                    |
| float, double          | number        | 1.1, -10.0, 0, "NaN", "Infinity"        | JSON value will be a number or one of the special string values "NaN", "Infinity", and "-Infinity". Either numbers or strings are accepted. Exponent notation is also accepted.                                                                                 |
| Any                    | object        | {"@type": "url", "f": v, … }            | If the Any contains a value that has a special JSON mapping, it will be converted as follows: {"@type": xxx, "value": yyy}. Otherwise, the value will be converted into a JSON object, and the "@type" field will be inserted to indicate the actual data type. |
| Timestamp              | string        | "1972-01-01T10:00:20.021Z"              | Uses RFC 3339, where generated output will always be Z-normalized and uses 0, 3, 6 or 9 fractional digits.                                                                                                                                                      |
| Duration               | string        | "1.000340012s", "1s"                    | Generated output always contains 0, 3, 6, or 9 fractional digits, depending on required precision. Accepted are any fractional digits (also none) as long as they fit into nano-seconds precision.                                                              |
| Struct                 | object        | { … }                                   | Any JSON object. See struct.proto.                                                                                                                                                                                                                              |
| Wrapper types          | various types | 2, "2", "foo", true, "true", null, 0, … | Wrappers use the same representation in JSON as the wrapped primitive type, except that null is allowed and preserved during data conversion and transfer.                                                                                                      |
| FieldMask              | string        | "f.fooBar,h"                            | See fieldmask.proto.                                                                                                                                                                                                                                            |
| ListValue              | array         | [foo, bar, …]                           |
| Value                  | value         |                                         | Any JSON value                                                                                                                                                                                                                                                  |
| NullValue              | null          | JSON null                               |

### JSON 选项

## 选项

Protocol Buffer 允许我们在 `.proto` 文件中定义一些常用的选项, 这样可以指示 Protocol Buffer 编译器帮助我们生成更为匹配的目标语言代码. Protocol Buffer 内置的选项被分为以下三个级别:

- 文件级别, 这样的选项将影响当前文件中定义的所有消息和枚举.
- 消息级别, 这样的选项仅影响某个消息及其包含的所有字段.
- 字段级别, 这样的选项仅仅响应与其相关的字段.

下面将给出一些常用的 Protocol Buffer 选项.

- `optimize_for` (文件选项): 可以设置的值有 `SPEED`, `CODE_SIZE` 或 `LITE_RUNTIME`, 不同的选项会以下述方式影响 `C++` 代码的生成 (`option optimize_for = CODE_SIZE;`) .
  - `SPEED` (default): protocol buffer 编译器将会生成序列化,语法分析和其他高效操作消息类型的方式 .这也是最高的优化选项 .确定是生成的代码比较大 .
  - `CODE_SIZE`: protocol buffer 编译器将会生成最小的类,确定是比 `SPEED` 运行要慢
  - `LITE_RUNTIME`: protocol buffer 编译器将会生成只依赖 "lite" runtime library (`libprotobuf-lite instead of libprotobuf`) 的类. lite 运行时库比整个库更小但是删除了例如 `descriptors` 和 `reflection` 等特性 . 这个选项通常用于手机平台的优化 .
- `cc_enable_arenas` (文件选项): 生成的 `C++` 代码启用 `arena allocation` 内存管理
- `deprecated`(文件选项):

## 参考资料

- [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3)
- [Protocol Buffer 使用简介](http://www.jianshu.com/p/b1f18240f0c7)
- [Protocol Buffer 技术详解 (语言规范)](http://www.cnblogs.com/stephen-liu74/archive/2013/01/02/2841485.html)
