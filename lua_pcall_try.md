# lua pcall

**2020-12-6**

[https://answer.uwa4d.com/question/5fc4bcbd10a17c6c2b09d3ca](https://answer.uwa4d.com/question/5fc4bcbd10a17c6c2b09d3ca)

事情的起因是这样的。在写代码时，使用 try-catch 可以很方便的防止异常抛出，导致程序报错卡住的情况。在 lua 中也想引入这种机制来写 lua 脚本。特此提出链接中的问题来确定想法的可行性。

但是，在完成封装、测试，最后应用到项目之前，我还是放弃了为 lua 引入的 try-catch 的特性。

原因是，我们是否真的需要这种异常捕获机制，以及这种机制是否会被滥用。

我认为绝大多数情况，我们并不需要这种这种异常的捕获机制。当一个函数的输入在可预知的范围内时，是不需要异常捕获的。作为一个合格的程序员，有义务对函数的输入做全面的合法性检测，并确保在合法的函数输入时，实际运行的逻辑代码不会出现异常。当检测到无法处理的非法的输入时，应该通过断言等方式，将错误信息反馈到控制台，这是开发环境中。在生产环境中，当检测到无法处理的非法输入时，要尽可能的还原发生错误前的现场，来确保其他模块不受影响的正常运行，并上报错误信息。在这里，我并不推荐使用函数返回值的方式来提供错误码，主要是因为一个函数内部的异常原因太多，调用者如果要判断函数的各种各样的返回值，无疑会增加相当大的心智负担，最后调用者的代码会变得冗长。

对于异常捕获机制的滥用，也是我不想把他引入项目的重要的原因。它会导致经验不足的开发者疏于考虑各种边界情况，把本该在开发阶段暴露出来的问题隐藏在层层嵌套的 try-catch 中。