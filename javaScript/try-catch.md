# try catch

> 执行 JS 的引擎是单线程的，当抛出异常时，会终止 JS 的执行，除非，我们主动捕获异常
>
> 使用 `try/catch/finally` 主动捕获异常，只要不让异常抛给 JS 引擎，就不会终止 JS 的执行

## throw

- 如果 try 抛了异常但没有被 catch 捕获（即没有 catch 代码块），或者 catch/finally 抛了异常，那么异常会被抛到外部并终止代码的执行

- catch 中的异常会覆盖 try 中的异常
- finally 中的异常会覆盖 try/catch 中的异常

## return 

try 可以有 return，catch/finally 也可以有 return，如果这个抛异常那个 return 会怎么样呢?

- catch 中的异常会覆盖 try 中的异常和 return (前提是try中出错)
- catch 中的 return 会覆盖 try 中的异常和 return (前提是try中出错)
- finally 中的异常会覆盖 try/catch 中的异常和 return
- finally 中的 return 会覆盖 try/catch 中的异常和 return
- try/catch/finally 块中异常或 return 之后的代码不会被执行