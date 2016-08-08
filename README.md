# 工作准备 - 刷题"一"  	
###1、为什么delegate中的property属性使用的是assign  
	  assign修饰的对象是弱引用(在arc中使用weak),不会造成循环
	引用的问题。
	eq:在使用tableView的时候，我们通常会把tableView的代理对
	象设置成viewController对象，但是同时viewcontroller持有
	这tableView，这个时候如果tableView的属性不是弱引用而是强
	引用的话，就会造成循环引用，这样ViewController和
	tableVie都不会被释放，这样就造成了内存泄露。
###2、retain、copy、assign的区别
	1.assgin修饰的都是基本对象类型，而不是OC中常用的NS对象类型，
	 assgin修饰的对象，就是直接进行赋值，如果需要使用某一个协议
	 需要明确指出某一个标记。
	2.retain仅仅是释放旧的对象，而降旧的对象的值赋值给输入对
	象，再增加输入对象的引用计数+1。即release旧值，retain新
	值。
	3.copy是建立一个引用计数为1的对象，然后释放旧的对象。
	
	解释一:Copy其实是建立了一个相同的对象，而retain不是： 
	比如一个NSString对象，地址为0×1111，内容为@"STR"，
	Copy到另外一个NSString之后，地址为0×2222，内容相同，新
	的对象retain为1，旧有对象没有变化；retain到另外一个
	NSString之后，地址相同（建立一个指针，指针拷贝），内容当
	然相同，这个对象的retain值+1
	
	其中有一个例子能很好说明assign和retain的区别，和引用计数的
	作用:那么假设你用malloc分配了一块内存，并且把它的地址赋值给
	了指针a，后来你希望指针b也共享这块内存，于是你又把a赋值给
	（assign）了b。此时a和b指向同一块内存，请问当a不再需要这块
	内存，能否直接释放它？答案是否定的，因为a并不知道b是否还在使
	用这块内存，如果a释放了，那么b在使用这块内存的时候会引起程序
	crash掉。
	了解到上述assign的问题，那么如何解决？最简单的一个方法就是
	使用引用计数（reference counting），还是上面的那个例子，
	我们给那块内存设一个引用计数，当内存被分配并且赋值给a时，引
	用计数是1。当把a赋值给b时引用计数增加到2。这时如果a不再使用
	这块内存，它只需要把引用计数减1，表明自己不再拥有这块内存。b
	不再使用这块内存时也把引用计数减1。当引用计数变为0的时候，代
	表该内存不再被任何指针所引用，系统可以把它直接释放掉。
	所以assign只是试用基本数据类型，因为不会引起加一问题的出现
	，而retain则会引起引用计数加一。
###3、深copy、浅copy的问题
	伪拷贝:
	- (id)copyWithZone:(NSZone *)zone{
    return [self retain];
    }
    相当于让让其引用计数加一
    
    浅拷贝:
    - (id)copyWithZone:(NSZone *)zone{
    Person *person = [[Person allocWithZone:zone]init];
    person.name = self.name;
    person.gender = self.gender;
    person.mArr = self.mArr;
    return person;
    }
    浅拷贝只是对地址的拷贝，改变其中一个对象的值，另一个对象的值
    也会发生变化。如果改对象指向的是常量区，那个改变器对象的值就
    相当于重指向了。并不对改变常量区里面的值。
    
    深拷贝:
    -(id)copyWithZone:(NSZone *)zone{
    Person *person = [[Person allocWithZone:zone]init];
    person.name = [self.name mutableCopy];
    person.gender = [self.name mutableCopy];
    return person;
    }
    深拷贝的结果是两份对象，两个值。改变其中一份值，另一份的值
    不会发生变化。
  
###4、非原子特性和原子特性
　　原子操作主要决定了编译器生成的getter和setter是否是原子操作。</br>
　　多线程下原子操作的setter方法会被加锁。

	 {lock}
     if (property != newValue) { 
        [property release]; 
        property = [newValue retain]; 
        }
        {unlock}
　　atomic是Objc使用的一直线程保护技术，是为了防止在某一个内容未完成的时候被另一个线程所读取，造成数据错误，但是这个机制十分消耗系统资源，所以在iPhone这种小型的设备上，一般是使用nonatomic。  
指出访问器不是原子操作，而默认地，访问器是原子操作。这也就是说，在多线程环境下，解析的访问器提供一个对属性的安全访问，从获取器得到的返回值或者通过设置器设置的值可以一次完成，即便是别的线程也正在对其进行访问。如果你不指定 nonatomic ，在自己管理内存的环境中，解析的访问器保留并自动释放返回的值，如果指定了 nonatomic ，那么访问器只是简单地返回这个值。
###5、类目和延展
####类目:
　　对一个类方法的扩展(允许添加方法，不允许添加变量)
类目的好处:1.团队开发的时候，如果想使用同一个类，而又不想相互影响就可以使用类目2.由于iOS是不开源的，所以不能修改原有的类，这个时候就可以使用类目扩展系统类里面的方法了。而且在调用方法的时候使用的是原有的类去调用方法。
####延展:
　　给一个类添加私有的实例变量和方法，在.m文件中添加一个私有的接口文件，声明方法:@interface 类名() @end