# Task 以及 Function



> 前言：这是一篇翻译文章，希望对你有一些帮助，原文在这里：https://documentation-rp-test.readthedocs.io/en/latest/tutorfpga07.html



function 和 task 是用来减少重复代码的利器！不仅仅如此，如果你想要你的代码变得更具有「可读性」不妨多用用 function 和 task。





## 0X00 Task



Task 是可以在该 Task 被定义的模块中任意时刻调用的子例程。但是可以在其他文件中定义它们，并将文件 include 到一个 module 中。



Task 有以下几种特性：



+ Task 可以包括 delay，但是 function 不能。



+ Task 可以有任意数量的输入和输出，但是 function 只能有一个输出



+ 在 Task 中定义的「变量」不能在 Task 外被使用



+ Task 是用语句调用的，不能在表达式中使用，function 可以在表达式中使用



+ Task 可以使用全局变量

















