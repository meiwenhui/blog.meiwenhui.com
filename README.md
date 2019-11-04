### 适配器模式

#### 特点：

 *  适配器继承或依赖已有的对象，实现想要的目标接口
 *  消除由于接口不匹配所造成的类的兼容性问题
 *  类的适配器模式、对象的适配器模式、接口的适配器模式
 *  提高了类的复用，增加了类的透明度

#### 案例
mybatis的log模块

### 装饰器模式

#### 特点：装饰类持有原有类或接口的对象，并调用它的方法
 *  通过一个装饰类对现有类对象动态添加一些功能，同时不改变其结构
 *  动态添加，动态撤销
 *  继承的替代方式，继承只能静态添加
 *  多成装饰产生过多相似对象，复杂且不易排错
 
#### 案例
mybatis的cache


### 代理模式
#### 注意事项：
 *  和适配器模式的区别：适配器模式主要改变所考虑对象的接口，而代理模式不能改变所代理类的接口。
 *  和装饰器模式的区别：装饰器模式为了增强功能，而代理模式是为了加以控制。


### 观察者模式

#### 特点
 *  类和类之间的关系
 *  对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并被自动更新
