当业务出现问题时，我们需要知道应该去哪个日志查看堆栈异常信息，这里对tomcat下的几种日志进行说明。

tomcat日志默认使用jdk的log，提供了自定义的logging.properties文件，在该文件中指定了不同的日志级别和格式(console/file)的日志输出。

## catalina.out

1. 主要是 tomcat中标准输出（stdout）和标准出错（stderr）信息，这个是在tomcat启动脚本中指定的，如果没有修改，那么stdout和stderr会重定向到这里。
2. 我们在应用中使用system.out打印的信息也会输出到这里。
3. 应用中使用了第三方日志框架，并且配置了console输出，则信息也会输出到这里。

## catalina.{yyy-MM-dd}.log

catalina.log存放了tomcat自身运行的一些日志。通过在tomcat/conf/logging.properties中配置的。

## localhost.{yyyy-MM-dd}.log

主要是应用初始化（listener，filter，servlet等）未处理的异常，最后被tomcat捕获而输出的日志。

一个tomcat对应一个engine，一个engine中有多个host（virtual host)，一个host里面可以有多个context，我们常常将应用部署在webapps的某个应用目录下，那么这个应用目录就是一个context。

这其中Engine对应着tomcat里的StandardEngine类，Host对应着StandardHost类，而Context对应着StandardContext。这几个类都是从ContainerBase派生。这些类里打的一些跟应用代码相关的日志都是使用ContainerBase里的getLogger，而这个这个logger的logger name就是: org.apache.catalina.core.ContainerBase.[current container name].[current container name]...

而我们一个webapp里listener, filter, servlet的初始化就是在StandardContext里进行的，比如ROOT里有一个listener初始化出异常了，打印日志则logger name是org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/]。这其中Catalina和localhost是上面xml片段里的Engine和Host的name，而[/]是ROOT对应的StandardContext的name。所以listener, filter, servlet初始化时的日志是需要看localhost.{yyyy-MM-dd}.log这个日志的。比如现在我们使用Spring，Spring的初始化我们往往是使用Spring提供的一个listener进行的，而如果Spring初始化时因为某个bean初始化失败，导致整个应用没有启动，这个时候的异常日志是输出到localhost中的，而不是cataina.out中。所以有的时候我们应用无法启动了，然后找catalina.out日志，但最后也没有定位根本原因是什么，就是因为我们找的日志不对。但有的时候catalina.out里也有我们想要的日志，那是因为我们的应用或使用的一些组件自己捕获了异常，然后将其打印了，这个时候如果恰好这些日志被我们配置成输出到console，则这些日志也会在catalina.out里出现了。



[参考链接](http://wiki.jikexueyuan.com/project/tomcat/logging.html)



