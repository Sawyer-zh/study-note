## include与require的区别

include()与require()的功能基本相同，但在用法上有一些不同

在错误处理方面，使用include语句，如果发生包含错误，程序将跳过include语句，虽然会**显示错误信息但是程序还是会继续执行**(显示**warning**)。但是，require语句会提示一个致命错误(显示**fatal**)