## 父工程步骤
###### 1、New Project
	![[创建工程.png]]
###### 2、聚合总夫工程名字
	![[总父工程名字.png]]
###### 3、字符编码
	![[字符编码.png]]
###### 4、注解生效激活
	![[注解生效.png]]
###### 5、java编译版本选17
	![[编译版本.png]]
###### 6、File Type过滤
	![[过滤.png]]

## 细节复习
###### 1、Maven中的dependencyManagement和dependencies
	![[父工程和子工程.png]]

	这样做的好处就是：如果有多个子项目都引用同一样依赖，则可以避免在每个使用的子项目里都声明一
	个版本号，优势：

	|1|这样当想升级或切换到另一个版本，只需要在顶层父容器里更新，而不需要一个一个子项目的改|
	|2|另外如果某个子项目需要另外的一个版本，只需要声明version就可                     |

	dependencyManagement里面只声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。

	如果不在子项目中声明依赖，是不会从父项目中继承下来的，只有在子项目中写了该依赖项并且没有指
	定具体版本，才会从父项目中继承该项。

	如果子项目中指定了版本号，那么会使子项目中指定的jar版本

###### 2、跳过单元测试
	![[跳过测试.png]]