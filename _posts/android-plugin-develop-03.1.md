---
title: 代理插件化基础-什么是代理模式
date: 2017-03-07 18:42:01
categories: Android
tags:
- Android
- 插件化
- 组件化
---
<Excerpt in index | 首页摘要> 
> 代理插件化基础-代理模式
>
> <!-- more -->
> <The rest of contents | 余下全文>  



####  概述  ####

我们需要增强某个对象的功能

但是又不能够去直接重写增强的时候

我们就需要用到代理模式了。



java中最著名的代理模式当属

java.io下的类了

比如：

InputStream 和 BufferedInputStream



####  实例-静态代理  ####

比如当前我有个动物的类

```java
interface Run{
  void run();
}

class Animal{
  public void run(){
    System.out.print("会跑");
  }
}
```

现在我需要增强它的run方法

让其变为狗会跑

```java
class Dog implements Run{
  private Run animal;
  public Dog(Run animal){
    this.animal = animal;
  }
  public void run(){
    System.out.print("狗");
    animal.run();
  }
}
```

然后调用起来

```java
public static void main(String[] args){
  Run animal = new Animal();
  Run dog = new Dog(animal);
  dog.run();
}
```



这样我们在不直接修改Animal的这个类的情况下增强了Animal的run方法



####  实例-动态代理  ####

可是我们往往是不知足的，

比如我们要增强的类有1000个 我们就需要去写这样1000个增强对象。

这多麻烦。



于是从jdk1.2开始。java提供了动态代理的方案



直接看代码:

```java
public static void main(String[] args){
  Run animal = new Animal();
  final Run finalAnimal = animal;
  animal = (Run)Proxy.newProxyInstance(Run.class.getClassLoader(),
                        Animal.class.getInterfaces(),
                        new InvocationHandler(){
                          	public Object invoke(Object proxy, Method method, Object[] args){
                              if("run".equals(method.getName())){
                                Systen.out.print("狗");
                                finalAnimal.run();
                              }
                              return null;
                          	}
                        });
  animal.run();
}

interface Run{
  void run();
}

public static class Animal implements Run{
  public void run(){
    System.out.print("会跑");
  }
}
```





####  在Android中应用  ####

了解了代理模式，

那么我们现在来试试看在Android中如何使用这种模式



比如，我们现在需要增强Activity的startActivity方法

希望在启动的时候，打印我们启动的参数等等信息..



创建一个新的model里面包含一个MainActivity

布局里面是一个按钮,id为mBtn

```java
public class MainActivity extends AppCompatActivity{
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(saveInstanceState);
    setContentView(R.layout.activity_main);
    findViewById(R.id.mBtn).setOnClickListener(new OnClickListener(){
      public void onClick(View v){
        startActivity(new Intent(MainActivity.this, MainActivity.class));
      }
    });
    
    //我们去查看看startActivity的代码
    //发现最后调用的是startActivityForResult
    //最后真正执行是由Activity的一个成员变量mInstrumentation
    //的execStartActivity方法在启动
    //那么我们就需要增强这个类的这个方法
    //我们发现Instrumentation没有实现任何借口,所以我们需要继承它然后重写它的execStartActivity方法
    //代码在尾部
    //然后我们修改Activity的这个成员变量
    try{
      Field InstrumentationField = Activity.Class.getDeclaredField("mInstrumentation");
      InstrumentationField.setAccessible(true);
      Instrumentation mInstrumentation = InstrumentationField.get(this);
      
      //修改为增强它
      mInstrumentation = new ExtInstrumentation(mInstrumentation);
      
      mInstrumentation.set(this, mInstrumentation);
    } catch(Exception e){
    }
  }
  
  class ExtInstrumentation extends Instrumentation{
    private Instrumentation target;
    public ExtInstrumentation(Instrumentation target){
      this.target = target;
    }
    
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {

        // Hook之前, XXX到此一游!
        Log.d("TAG", "\n执行了startActivity, 参数如下: \n" + "who = [" + who + "], " +
                "\ncontextThread = [" + contextThread + "], \ntoken = [" + token + "], " +
                "\ntarget = [" + target + "], \nintent = [" + intent +
                "], \nrequestCode = [" + requestCode + "], \noptions = [" + options + "]");

        // 开始调用原始的方法, 调不调用随你,但是不调用的话, 所有的startActivity都失效了.
        // 由于这个方法是隐藏的,因此需要使用反射调用;首先找到这个方法
        try {
            Method execStartActivity = Instrumentation.class.getDeclaredMethod(
                    "execStartActivity",
                    Context.class, IBinder.class, IBinder.class, Activity.class, 
                    Intent.class, int.class, Bundle.class);
            execStartActivity.setAccessible(true);
            return (ActivityResult) execStartActivity.invoke(target, who, 
                    contextThread, token, target, intent, requestCode, options);
        } catch (Exception e) {
            // 出现问题了 需要手动适配
            throw new RuntimeException("do not support!!! pls adapt it");
        }
    }
  }
}
```

