# chrome 开发工具内置变量
chorme devtool 本身也有不少内置变量，今天我们看下哪些变量在平常中使用

## $_ 
* 英文解释 returns the value of the most recently evaluated expression.
* 中文解释 返回最近求值的表达式的值。

![]($_.png)

## $0 - $4
* 英文解释 The $0, $1, $2, $3 and $4 commands work as a historical reference to the last five DOM elements inspected within the Elements panel or the last five JavaScript heap objects selected in the Profiles panel.$0 returns the most recently selected element or JavaScript object, $1 returns the second most recently selected one, and so on.
* 中文解释 $0，$1，$2，$3 和 $4 命令用作对在Elements面板中检查的最后五个DOM元素或在Profiles面板中选择的最后五个JavaScript堆对象的历史参考。$0 返回最近选择的元素或JavaScript对象，$1 返回第二个最近选择的元素，依此类推

![]($1.png)

## $(selector, [startNode]) 
* 英文解释 this function is an alias for the document.querySelector() function.
* 中文解释 document.querySelector 函数别名

![]($().png)

## $$(selector, [startNode])
* 英文解释 $$(selector) returns an array of elements that match the given CSS selector. This command is equivalent to calling document.querySelectorAll().
* 中文解释 document.querySelectorAll 函数别名

![]($$().png)

## $x(path, [startNode])
* 英文解释 $x(path) returns an array of DOM elements that match the given XPath expression.
* 中文解释 $x(path) 返回与给定XPath表达式匹配的DOM元素数组。

  例如 $("//p") 返回所有的p元素，$("//p[a]") 返回所有包含a元素的p元素
![]($x.png)