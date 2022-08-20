返回有关函数和库的信息。

您可以使用可选的`LIBRARYNAME`参数来指定用于匹配库名称的模式。
可选`WITHCODE`修饰符将导致服务器在回复中包含库源实现。

为响应中的每个库提供了以下信息：

*   **library_name：**库的名称。
*   **发动机：**库的引擎。
*   **功能：**库中的函数列表。
    每个函数都具有以下字段：
    *   **名字：**函数的名称。
    *   **描述：**函数的描述。
    *   **标志：**数组[函数标志](/docs/manual/programmability/functions-intro/#function-flags).
*   **library_code：**库的源代码（当给定`WITHCODE`修饰符）。

欲了解更多信息，请参阅[Redis 函数简介](/topics/functions-intro).

@return

@array回复
