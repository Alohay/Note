- 策略模式

  - 将一个类的行为抽象成一个接口，通过构造函数或者`setter`实例化，如果某个行为需要修改，增加一个相应的实现类即可
  - 如果使用原来继承的方式，则需要继承接口，将其他不变的行为再重新写一遍一模一样的，这代码重复度就很高了，而且修改次数多了就会变得难以维护
  - 这个模式说明`has a` 比`is a` 更好，也就是要面向接口编程，多用组合，少用继承

- 观察者模式
  - 定义了对象之间一对多的依赖，当一个对象状态改变的时候，他的所有依赖者会受到通知并自动更新
  - 以重置密码举例，该事件可以作为一个主题，有短信通知，邮件通知，站内信通知的观察者，修改密码成功后调用`notifyObservers`，就会以三种形式通知用户

- 装饰者模式
  - 以继承的方式活用组合，多个装饰者通过继承统一对象类型，然后自己添加行为方法，达到装饰的目的，这样也是一种组合，如果不需要某个行为就减少一层包装，同样如果要加一种行为，再包装一层装饰者
  - 可以在不修改底层代码的情况下，增加新职责
  - 缺点是容易创建非常多的类

- 工厂模式
  - 简单工厂 `// 这个不能说是工厂模式，说是一种编码习惯更为恰当`

    - 将`new`对象的代码从方法中抽出来，实现一个工厂方法，通过参数区分返回不同的对象，代码维护起来更容易，适用于用户不关心对象的创建过程

  - 工厂模式

    - 与简单模式不同，工厂模式针对每一个不同的对象要有不同的工厂，每个对象都要一个与之对应的工厂，由子类来决定实例化的对象

    - ``` java
      public interface Reader {
          void read();
      }
      public class JpgReader implements Reader {
          @Override
          public void read() {
              System.out.print("read jpg");
          }
      }
      public class PngReader implements Reader {
          @Override
          public void read() {
              System.out.print("read png");
          }
      }
      
      public interface ReaderFactory {
          Reader getReader();
      }
      public class JpgReaderFactory implements ReaderFactory {
          @Override
          public Reader getReader() {
              return new JpgReader();
          }
      }
      public class PngReaderFactory implements ReaderFactory {
          @Override
          public Reader getReader() {
              return new PngReader();
          }
      }
      
      
      // Factory factory=new JpgReaderFactory();
      ReaderFactory factory=new PngReaderFactory();
      Reader  reader=factory.getReader();
      reader.read();
      
      /*
      作者：Knight_Davion
      链接：https://www.jianshu.com/p/83ef48ce635b
      來源：简书
      简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
      */
      ```

  - 抽象工厂

    - 抽象工厂模式是工厂方法的仅一步深化，在这个模式中的工厂类不单单可以创建一个对象，而是可以创建一组对象 
    -  `https://www.jianshu.com/p/83ef48ce635b` 讲的很好

- 