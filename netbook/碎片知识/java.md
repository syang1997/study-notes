# 碎片知识

## java的new浪费了太多的内存，应该大量的使用static？

对象重要特征是什么？对象重要的特征就是有生命的，有scope的，有生命周期的，这其实是一个很朴素的哲学思想，没有永垂不朽的东西，任何物体都是有生命的。

既然，对象有生命，那就当然有生有死，在java中，对象什么时候死已经无需我们操心，因为有垃圾回收机制，我们程序员只要决定对象什么时候生就可以，也就是对象什么时候创建，以何种方式创建。

回到对象的生命周期上来，使用静态实际就是变态的延长对象的生命周期，虽然也解放了程序员，无需程序员照顾对象创建，但是这是一种错误的解放，是一种虽然简单，但是方向完全错误，可能导致更大性能陷阱的解决方式。因为我们现在的软件是一个多线程环境，如果你使用静态，不但导致非OO系统，到处是长命百岁的对象，系统难于维护；更重要不小心就导致多线程变成单线程系统，也就是单用户系统，某个时刻只能一个用户操作这个系统。换句话说：就是系统缓慢，人操作一多就死机。

## Spring Bean中的成员变量是否会被jvm回收

一个spring bean默认初始化为单例对象，长期会被spring容器保持，容器寄存在servlet的成员变量的servetcontext中，即应用服务器不关闭，引用将一直存在。如开始的成员变量，注入的对象，内存空间一直被引用着，jvm不会回收它们的空间。

对于强引用的对象，比如，定义了一个全局变量：Object obj = new Object，当你希望它被回收时，必须强制将它置为空 ，obj=null, 否则，jvm永远不会去回收一个强引用对象。

所以要想一个spring对象中的成员变量不被回收，在bean被注入到spring时就要强引用它。