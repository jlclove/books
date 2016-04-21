### 异常的一些思考

异常是对正常执行流程的中断，是一种正向的反馈。

俗语说，就怕不吭不响、不知不觉、不明不白就崩溃了，Java 使用异常这种通知机制告知我们的代码执行已超出了逻辑边界：出大事了。

可以换一种说法，异常是正常代码流程的副产品，也会对代码可读性产生某种干扰。

 比如以下代码：
     
 ``` java
     // 正常流程
   public  void doSomeLogic(){
     //1,  查询
   	A  a=  search(); 
    //发送消息 
    send(a); 
    }
 ``` 
 出于健壮性考虑，添加异常后：
 ``` java
     // 异常处理
   public  void doSomeLogic(){
     //1,  查询
      try{
        A  a=  search(); 
      }catch(Exception e){
        //handle excetpion
      }
    //发送消息 
    try{
       send(a); 
    }catch(Excetpion e){
       // handle excetpion
    }
  }
 ``` 
代码可读性降低，也不够简洁了。

对于以上代码，我们可以在search()或send()方法内部try-catch异常。

### 异常的一些特殊用途

1， 将异常堆栈转换为字符串，以便发送报警邮件。
```java
StringWriter sw = new StringWriter();
PrintWriter pw = new PrintWriter(sw);
t.printStackTrace(pw);
sw.toString(); // stack trace as a string
```

2，堆栈元素StackTraceElement包含方法名、文件名、行号，可以找到当前程序启动类的main方法
```java
  private Class<?> deduceMainApplicationClass() {
		try {
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			for (StackTraceElement stackTraceElement : stackTrace) {
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// Swallow and continue
		}
		return null;
	}
```

