
### 【写在前面】


CMake 可以通过属性来存储信息。它就像是一个变量，但它被附加到一些其他的实体上，像是一个目录或者是一个目标。例如一个全局的属性可以是一个有用的非缓存的全局变量。


在 CMake 的众多属性中，目标属性 ( **Target Properties** ) 扮演着尤为重要的角色，它们直接关联到最终生成的可执行文件、库文件等构建产物。


更直观一点，如果把目标类比为 **类 ( Class )**，那么目标属性则类似 **类成员 ( Class  Member )**。



```
class Target {
    string target_property;
};

```



---


### 【正文开始】


生成目标的方式有三种：



```
#生成可执行文件目标test
add_executable(test test.cpp)
#生成共享库目标test_lib
add_library(test_lib SHARED test.cpp)
#生成静态库目标test_lib
add_library(test_lib STATIC test.cpp)

```

#### 定义目标属性：


给目标定义属性的命令为：



```
define_property(TARGET | SOURCE |
                 TEST | VARIABLE | CACHED_VARIABLE>
                 PROPERTY  [INHERITED]
                 [BRIEF_DOCS  [docs...]]
                 [FULL_DOCS  [docs...]]
                 [INITIALIZE_FROM_VARIABLE ])

```


> 在范围内定义一个属性，用于 [set\_property()](https://github.com "set_property()") 和 [get\_property()](https://github.com "get_property()") 命令。它主要用于定义属性的初始化或继承方式。从历史上看，该命令还将文档与属性相关联，但这不再被视为主要用例。


示例：



```
# 定义一个目标属性 TEST_TARGET，带有简短和详细描述 
define_property(TARGET PROPERTY TEST_TARGET 
    BRIEF_DOCS "A test property"
    FULL_DOCS "A long description of this test property"
)

```

#### 设置目标属性：


给目标设置属性的命令为：



```
set_property(]           |
              TARGET    [ ...]   |
              SOURCE    [ ...]
                        [DIRECTORY  ...]
                        [TARGET_DIRECTORY  ...] |
              INSTALL   [ ...]     |
              TEST      [ ...]     |
              CACHE     [ ...]    >
             [APPEND] [APPEND_STRING]
             PROPERTY  [ ...])

```


> 在范围的零个或多个对象上设置一个属性。
> 
> 
> 如果给出 `APPEND` 选项，列表将附加到任何现有的属性值（除了忽略和不附加空值）。如果给出 `APPEND_STRING` 选项，字符串将作为字符串附加到任何现有属性值，即它会产生更长的字符串而不是字符串列表。当使用 `APPEND` 或 `APPEND_STRING` 以及定义为支持 `INHERITED` 行为的属性时（请参阅 `:command:define_property`），在找到要附加到的初始值时不会发生继承。如果该属性尚未在指定范围内直接设置，则该命令的行为就好像没有给出 `APPEND` 或 `APPEND_STRING` 一样。


其中，有一个专用于设置目标属性命令：



```
set_target_properties(target1 target2 ...
                      PROPERTIES prop1 value1
                      prop2 value2 ...)

```


> 设置目标的属性。该命令的语法是列出您要更改的所有目标，然后提供您接下来要设置的值。您可以使用任何您想要的 prop 值对，稍后使用 [get\_property()](https://github.com "get_property()") 或 [get\_target\_property()](https://github.com "get_target_property()") 命令提取它。


示例：



```
# 设置目标 TEST_TARGET 的属性 P_1 和 P_2 的值
set_target_properties(TEST_TARGET PROPERTIES
    P_1 "这是属性P_1的值"
    P_2 "这是属性P_2的值" 
)

```

#### 获取目标属性：


获取目标的属性的命令为：



```
get_property(
             ]  |
              TARGET    <target> |
              SOURCE    
                        [DIRECTORY  | TARGET_DIRECTORY <target>] |
              INSTALL   <file>   |
              TEST      <test>   |
              CACHE       |
              VARIABLE           >
             PROPERTY 
             [SET | DEFINED | BRIEF_DOCS | FULL_DOCS])

```


> 从范围内的一个对象获取一个属性。
> 
> 
> 如果给出了 `SET` 选项，变量将被设置为一个布尔值，指示该属性是否已被设置。如果给出了 `DEFINED` 选项，变量将被设置为一个布尔值，指示该属性是否已被定义，例如使用 `define_property` 命令。 如果给出了`BRIEF_DOCS` 或 `FULL_DOCS`，那么该变量将被设置为一个字符串，其中包含所请求属性的文档。如果为尚未定义的属性请求文档，则返回 “`NOTFOUND`”。


其中，有一个专用于获取目标属性命令：



```
get_target_property( target property)

```


> 从目标获取属性。属性的值存储在变量“”中。如果未找到目标属性，则行为取决于它是否已被定义为 `INHERITED` 属性（请参阅:command:define\_property）。非继承属性会将设置为`-NOTFOUND`，而继承属性将搜索相关的父范围，如 [define\_property()](https://github.com "define_property()") 命令所述，如果仍然找不到属性  将被设置为空字符串。
> 
> 
> 使用 [set\_target\_properties()](https://github.com "set_target_properties()"):[豆荚加速器](https://yirou.org) 设置目标属性值。属性通常用于控制目标的构建方式，但有些属性会查询目标。此命令可以获得迄今为止创建的任何目标的属性。目标不需要位于当前的 CMakeLists.txt 文件中。


示例：



```
# 获取目标 TEST_TARGET 的属性 P_1 的值，并将其存储在变量 TEST_TARGET_P1 中
get_target_property(TEST_TARGET_P1 TEST_TARGET P_1) 

```

最后完整测试一遍：



```
# 要求 CMake 最低版本为 3.16
cmake_minimum_required(VERSION 3.16)

# 定义项目名称为 PROPERTY_TEST，版本号为 1.0，使用 C++ 语言
project(PROPERTY_TEST VERSION 1.0 LANGUAGES CXX)

# 添加一个名为 TEST_TARGET 的可执行文件，其源文件为 test.cpp
add_executable(TEST_TARGET test.cpp)

# 定义一个目标属性 TEST_TARGET，带有简短和详细描述 
define_property(TARGET PROPERTY TEST_TARGET 
    BRIEF_DOCS "A test property"
    FULL_DOCS "A long description of this test property"
)

# 设置目标 TEST_TARGET 的属性 P_1 和 P_2 的值
set_target_properties(TEST_TARGET PROPERTIES
    P_1 "这是属性P_1的值"
    P_2 "这是属性P_2的值" 
)

# 获取目标 TEST_TARGET 的属性 P_1 的值，并将其存储在变量 TEST_TARGET_P1 中
get_target_property(TEST_TARGET_P1 TEST_TARGET P_1) 

# 打印变量 TEST_TARGET_P1 的值
message("TEST_TARGET_P1: ${TEST_TARGET_P1}") 


```

CMake 输出如下：


![image](https://img2024.cnblogs.com/blog/802097/202410/802097-20241008201403243-1366374655.png)




---


### 【结语】


最后，属性在 CMake 中是可继承的。这意味着，如果一个目标继承了另一个目标的属性，那么它将自动获得父目标的属性值。这种继承关系在构建大型项目时非常有用，因为它减少了重复的配置。


CMake 的属性系统是一个强大的工具，它提供了细粒度的控制，允许我们定制构建过程。通过合理使用属性，我们可以创建更加灵活和可维护的构建系统。掌握属性的使用，是每个 CMake 用户进阶的必经之路，希望这篇文章能够帮助你更好地理解和使用 CMake 的属性系统。


项目链接(多多star呀..⭐\_⭐)：


Github 地址：[https://github.com/mengps/LearnCMake](https://github.com)


 ![](https://github.com/blog/802097/202404/802097-20240417231514692-194340162.png)    - **本文作者：** [梦起丶](https://github.com)
 - **本文链接：** [https://github.com/mengps/p/18452409](https://github.com)
 - **关于博主：** 🎉🎉🎉专注于 C / C\+\+ / Qt / JS / Python 编程技巧🎉🎉🎉

✨个人微信✨：MenPenS612 (交流合作都行\~)

✨交流群✨：83986890 ヾ(￣▽￣)欢迎来玩\~

✨公众号✨：程序梦 (扫左侧二维码)
 - **版权声明：** 本博客所有文章除特别声明外，均采用 [BY\-NC\-SA](https://github.com "BY-NC-SA") 许可协议。转载请注明出处！
 - **声援博主：** 如果您觉得文章对您有帮助，可以点击文章右下角**【[推荐](javascript:void(0);)】**一下。
     
