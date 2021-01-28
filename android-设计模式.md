# Android技能之设计模式
## 设计模式的分类
### 设计模式共23种，按照目的的准则分类，分为三大类：
- 创建型（5）：单例、工厂方法、抽象工厂、建造者、原型
- 结构型（7）：适配器、装饰、代理、外观、桥接、组合、享元
- 行为型（11）：策略、模板方法、观察者、迭代器、责任链、命令、备忘录、状态、访问者、中介者、解释器
## 设计模式的六大原则
- 单一职责原则：一个类应该只有一处引起它变化的地方，即一个类不应该承担过多的职责，降低耦合性
- 开放封闭原则：类、函数、模块对扩展开放，修改封闭
- 里式替换原则：引用父类的地方，使用父类对象，运行时指定具体的子类对象
- 依赖倒置原则：高层模块与低层模块、模块间应通过接口或抽象类产生依赖
- 迪米特原则：最少知识原则，软件实体尽可能少地与其他实体相互作用
- 接口隔离原则：类与类之间的依赖建立在最小的接口上
## 常用的设计模式
### ==1. 单例模式==
- 饿汉模式
        
```
    /**
     * 在类加载的时候就完成实例化，如果从未使用过该单例，会造成资源的浪费
     */
    public class Singleton {
        pivate static Singleton instance = new Singleton();
        
        private Singleton() {
            
        }
        
        public static Singleton getInstance() {
            return instance;
        }
    }
```
- 懒汉模式（线程安全）

```
    /**
     * 为了处理并发，每次调用getInstance方法时都需要进行同步，会有不必要的同步开销
     */
    public class Singleton() {
        private static Singleton instance;
        
        private Singleton() {
            
        }
        
        public static synchronized Singleton getInstance() {
            if(instance == null) {
                instance = new Singleton();
            }
            return instance;
        }
    }
```
- 双重检查锁模式（DCL）

```
    /**
     * DCL在一定程度上解决了资源的消耗和多余的同步、线程安全等问题，但是在某些情况下会失效
     */
   public class Singleton {
        //volatile保证instance具有可见性，每次都是从主存中获取
       private static volatile Singleton instance;
       
       private Singleton() {
           
       }
       
       public static Singleton getInstance() {
           if(instance == null) {//第一次判空，省去不必要的同步
               synchronized(Singleton.class) {
                   if(instance == null) {//第二次判空，当instance为空时才去创建实例
                       //不是原子操作，分为三步，
                       //第一步给instance的实例分配内存，
                       //第二步调用Singleton()构造函数，初始化成员字段，
                       //第三步，将instance对象指向分配的内存空间（此时instance就不是null了），
                       //但java编译器允许处理器乱序执行，则顺序1，2，3可以是1，3，2，
                       //则当线程A执行到3后，线程B（线程都有各自的私有存储空间），获取instance则不为空，但成员变量都为空，
                       //JDK1.5之后，且引入volatile可在并发不是很复杂的情况下解决此问题
                       instance = new Singleton();
                   }
               }
           }
           return instance;
       }
   } 
```
- 静态内部类模式

```
    /**
     * 第一次调用getInstance方法时虚拟机才加载SingletonHolder并初始化instance，保证了线程安全和实例的唯一性
     */
    public class Singleton {
        private Singleton() {
            
        }
        
        public static Singleton getInstance() {
            return SingletonHolder.instance;
        }
        
        private static class SingletonHolder {
            private static final Singleton instance = new Singleton();
        }
    }
```
- 枚举单例
    
```
    /**
     * 默认枚举实例的创建是线程安全的，并且在任何情况下都是单例，简单、可读性不高
     */
    public enum Singleton {
        INSTANCE;
        
        public void doSomething() {
            
        }
    }
```
注意：：上面的几种单例模式创建的单例对象被反序列化时会重新创建实例，可以重写readReslove方法返回当前的单例对象。
### ==2.简单工厂模式==
也成为静态工厂方法模式，由工厂对象决定创建出哪一种产品类的实例。

有如下角色：
- 工厂类：核心，负责创建所有实例的内部逻辑，由外部直接调用
- 抽象产品类：创建所有对象的抽象父类，并描述所有实例的共有接口
- 具体产品类：要创建的产品

1. 抽象产品类

```
    public abstract class Computer {
        public abstract void start();
    }
```
2. 具体产品类

```
    public class LenovoComputer extends Computer {
    
        @Override
        public void start() {
            
        }
    }
    
    public class HpComputer extends Computer {
    
        @Override
        public void start() {
            
        }
    }
    ....
```

3. 工厂类

```
    public class ComputerFactory {
        public static Computer createComputer(String type) {
            Computer computer = null;
            switch(type) {
                case "lenovo":
                    computer = new LenovoComputer();
                break;
                case "hp":
                    computer = new HpComputer();
                break;
            }
            return computer;
        }
    }
```
- 需要知道所有的产品类，因此只适合工厂类负责创建的对象比较少的情况
- 避免直接实例化，降低耦合性
- 增加新产品需要修改工厂类，违背开放封闭原则
### ==3. 工厂方法模式==
定义一个用于创建对象的接口，使类的实例化延迟到子类。

有如下角色：
- 抽象产品类：同简单工厂模式
- 具体产品类：同简单工厂模式
- 抽象工厂类：返回一个泛型的产品对象
- 具体工厂类：返回一个具体的产品对象
1. 抽象产品类

```
    public abstract class Computer {
        public abstract void start();
    }
```
2. 具体产品类

```
    public class LenovoComputer extends Computer {
    
        @Override
        public void start() {
            
        }
    }
    
    public class HpComputer extends Computer {
    
        @Override
        public void start() {
            
        }
    }
    ....
```
3. 抽象工厂类

```
    public abstract class ComputerFactory {
        public abstract <T extends Computer> T createComputer(Class<T> clz);
    }
```

4. 具体工厂类

```
    public class GDComputerFactory extends ComputerFactory {
        
        @Override
        public <T extends Computer> T createComputer(Class<T> clz) {
            Computer computer = null;
            String className = clz.getName();
            try {
                computer = (Computer) Class.foName(className).newInstance();
            } catch (Exception e){
                e.printStackTrace();
            }
            return (T) computer;
        }
    }
```
使用：

```
    GDComputerFactory gDComputerFactory = new GDComputerFactory();
    LenovoComputer lenovoComputer = gDComputerFactory.createComputer(LenovoComputer.class);
```

相比简单工厂，如果需要新增产品类，无需修改工厂类，直接创建产品即可。
### ==4. 建造者模式==
将复杂对象的构建和它的表示进行分离，使得同样的构建过程有不同的表示。
有如下角色：
- 导演类：负责安排建造的顺序，通知建造者开始建造
- 抽象建造者：抽象Builder类，用于规范产品的建造
- 具体建造者：实现抽象Builder类的所有方法，并返回建造好的产品
- 产品类：具体使用的产品类
1. 产品类

```
    public class Computer {
    private String cpu;
    private String mainBoard;
    private String ram;

    public void setCpu(String cpu) {
        this.cpu = cpu;
    }

    public void setMainBoard(String mainBoard) {
        this.mainBoard = mainBoard;
    }

    public void setRam(String ram) {
        this.ram = ram;
    }
}
```

2. 抽象建造者

```
    public abstract class Builder {
    
    public abstract void buildCpu(String cpu);

    public abstract void buildMainBoard(String mainBoard);

    public abstract void buildRam(String ram);

    public abstract Computer create();
}
```

3. 具体建造者

```
    public class HpComputerBuilder extends Builder {

        private Computer computer = new Computer();
    
        @Override
        public void buildCpu(String cpu) {
            computer.setCpu(cpu);
        }
    
        @Override
        public void buildMainBoard(String mainBoard) {
            computer.setMainBoard(mainBoard);
        }
    
        @Override
        public void buildRam(String ram) {
            computer.setRam(ram);
        }
    
        @Override
        public Computer create() {
            return computer;
        }
    }
```

4. 导演类

```
    public class Director {
        private Builder builder;
    
        public Director(Builder builder) {
            this.builder = builder;
        }
    
        public Computer createComputer(String cpu, String mainBoard, String ram) {
            builder.buildCpu(cpu);
            builder.buildMainBoard(mainBoard);
            builder.buildRam(ram);
            return builder.create();
        }
    }
```
使用：

```
    HpComputerBuilder hpComputerBuilder = new HpComputerBuilder();
    Director director = new Director(hpComputerBuilder);
    Computer computer = director.createComputer("cpu","mainBoard","ram");
```

- 屏蔽产品内部组成细节
- 具体建造者类之间相互独立，容易扩展
- 会产生多余的建造者对象和导演类

### ==5. 适配器模式==
将一个接口转换为另一个需要的接口。
有如下角色：
- 要转换的接口
- 要转换的接口的实现类
- 转换后的接口
- 转换后的接口的实现类
- 适配器类
1. 要转换的接口(火鸡)

```
    public interface Turkey {
        //吞噬
        public void gobble();
        
        public void fly();
    }
```

2. 要转换的接口的实现类(野火鸡)

```
    public class WildTurkey implements Turkey {
        
        @Override
        public void gobble() {
            ...
        }
        
        @Override
        public void fly() {
            ...
        }
    }
```

3. 转换后的接口(鸭子)

```
    public interface Duck {
        //嘎嘎
        public void quack();
        
        public void fly();
    }
```

4. 转换后的接口的实现类(绿头鸭)

```
   public class MallardDuck implements Duck {
   
        @Override
        public void quack() {
            ...
        }
        
        @Override
        public void fly() {
            ...
        }
       
   }
```

5. 适配器类

```
    public class TurkeyAdapter implements Duck {
        private Turkey turkey;
        
        public TurkeyAdapter(Turkey turkey) {
            this.turkey = turkey;
        }
        
        @Override
        public void quack() {
            turkey.gobble();
        }
        
        @Override
        public void fly() {
            // 火鸡没有鸭子飞的远，因此多飞几次，达到适配鸭子fly的作用
            for(int i = 0; i < 5; i++) {
                turkey.fly();
            }
        }
    }
```
使用：

```
    WildTurkey wildTurkey = new WildTurkey();
    TurkeyAdapter turkeyAdapter = new TurkeyAdapter(wildTurkey);
    turkeyAdapter.quack();
    turkeyAdapter.fly();
```
- 注重适度使用即可。


### ==6. 装饰模式==
动态地给一个对象添加额外的职责。

有如下角色：
- 抽象组件：接口/抽象类，被装饰的最原始的对象
- 具体组件：被装饰的具体对象
- 抽象装饰者：扩展抽象组件的功能
- 具体装饰者：抽象装饰者的具体实现
1. 抽象组件

```
    //剑客
    public abstract class Swordsman {
    
        public abstract attackMagic();
    }
```

2. 具体组件

```
    public class YangGuo extends Swordsman {
        
        @Override
        public attackMagic() {
            ...
        }
    }

```

3. 抽象装饰者

==抽象装饰者必须持有抽象组件的引用==，以便扩展功能。

```
    public class Master extends Swordsman {
        private Swordsman swordsman;
        
        public Master(Swordsman swordsman) {
            this.swordsman = swordsman;
        }
        
        @Override
        public attackMagic() {
            swordsman.attackMagic();
        }
    }

```

4. 具体装饰者

```
    public class HongQiGong extends Master {
    
        public HongQiGong(Swordsman swordsman) {
            this.swordsman = swordsman;
        }
        
        public void teachAttackMagic() {
            ...
        }
        
        @Override
        public attackMagic() {
            super.attackMagic();
            teachAttackMagic();
        }
    }
```
使用：

```
    YangGuo yangGuo = new YangGuo();
    HongQiGong hongQiGong = new HongQiGong(yangGuo);
    hongQiGong.attackMagic();
```
- 使用组合动态地扩展对象的功能，在运行时能够使用不同的装饰器实现不同的行为。
- 比继承更易出错，旨在必要时使用。


### ==7. 代理模式==
为其他对象提供一种代理以控制这个对象的访问。
有如下角色：
- 抽象主题类：声明真实主题和代理的共同接口方法
- 真实主题类：抽象主题类的具体实现
- 代理类：==持有对真实主题类的引用==
- 客户端类：
#### 静态代理：
1. 抽象主题类

```
    public interface IShop {
    
        public void buy();
    }
```

2. 真实主题类

```
    public class Weimo implements IShop {
        
        @Override
        public void buy() {
            ...
        }
    }
```

3. 代理类

```
    public class Purchasing implements IShop {
        private IShop iShop;
    
        public Purchasing(IShop iShop) {
            this.iShop = iShop;
        }
        
        @Override
        public void buy() {
            iShop.buy();
        }
    }
```

4. 客户端类

```
    public class Client {
    
        public static void main(String[] args) {
            IShop weimo = new Weimo();
            IShop purchasing = new Purchasing(weimo);
            purchasing.buy();
        }
    }
```
#### 动态代理：
在代码运行时通过==反射==来动态地生成代理类对象，并确定到底来代理谁。
1. 动态代理类

```
    public class DynamicPurchasing implements InvocationHandler {
        private Object obj;
        
        public DynamicPurchasing(Object obj) {
            this.obj = obj;
        }
        
        @Overrdie
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            return method.invoke(obj, args);
        }
    }
```

2. 客户端类

```
    public class Client {
    
        public static void main(String[] args) {
            IShop weimo = new Weimo();
            DynamicPurchasing dynamicPurchasing = new DynamicPurchasing(weimo);
            ClassLoader cl = weimo.getClass.getClassLoader();
            IShop purchasing = Proxy.newProxyInstance(cl, new Class[]{IShop.class}, dynamicPurchasing);
            purchasing.buy();
        }
    }
```
- 真实主题类发生变化时，由于它实现了公用的接口，因此代理类不需要修改。


### ==8. 外观模式（门面模式）==
一个子系统的内部和外部通信必须通过一个统一的对象进行，即提供一个高层的接口，方便子系统更易于使用。
有如下角色：
- 外观类：将客户端的请求代理给适当的子系统对象
- 子系统类：可以有一个或多个子系统，用于处理外观类指派的任务。注意子系统不包含外观类的引用
1. 子系统类（这个有三个子系统）

```
    public class ZhaoShi {
        public void TaiJiQuan() {
            ...
        }
    
        public void QiShanQuan() {
            ...
        }
    
        public void ShengHuo() {
            ...
        }
    }

    public class NeiGong {
        public void JiuYang() {
            ...
        }
        
        public void QianKun() {
            ...
        }
    }

    public class JingMai {
        public void JingMai() {
            ...
        }
    }
```

2. 外观类

```
    public class ZhaoWuJi {
        private ZhaoShi zhaoShi;
        private JingMai jingMai;
        pirvate Neigong neiGong;
    
        public ZhangWuJi() {
            zhaoShi = new ZhaoShi();
            jingMai = new JingMai();
            neiGong = new NeiGong();
        }
        
        public void qianKun() {
            jingMai.JingMai();
            neiGong.QianKun();
        }
        
        public void qiShang() {
            jingMai.JingMai();
            neiGong.JiuYang();
            zhaoShi.QiShangQuan();
        }
    }
```
使用：

```
    ZhaoWuJi zhaoWuJi = new ZhaoWuJi();
    zhaoWuJi.qianKun();
    zhaoWuJi.qiShang();
```
- 对子系统的依赖转换为对外观类的依赖
- 对外部隐藏子系统的具体实现
- 这种外观特性增强了安全性


### ==9. 享元模式==
使用共享对象有效支持大量细粒度（性质相似）的对象。

额外的两个概念：
- 1、内部状态：共享信息，不可改变
- 2、外部状态：依赖标记，可以改变

有如下角色：
- 抽象享元角色：定义对象内部和外部状态的接口
- 具体享元角色：实现抽象享元角色的任务
- 享元工厂：管理对象池及创建享元对象
1. 抽象享元角色

```
    public interface IGoods {
    
        public void showGoodsPrice(String type);
    }
```

2. 具体享元角色

```
    public class Goods implements IGoods {
        private String name;
        
        public Goods(String name) {
            this.name = name;
        }
        
        @Override
        public void showGoodsPrice(String type) {
            ...
        }
    }
```

3. 享元工厂

```
    public class GoodsFactory {
        private static Map<String,Goods> pool = new HashMap<String,Goods>();
        
        public static Goods getGoods(String name) {
            if(pool.containsKey(name) {
                return pool.get(name);
            } else {
                Goods goods = new Goods(name);
                pool.put(name,goods);
                return goods;
            }
        }
    }
```
使用：

```
    //goods1为新创建的对象，其他的都是从对象池中取出的缓存对象
    Goods goods1 = GoodsFactory.getGoods("Android进阶之光");
    good1.showGoodsPrice("普通版");
    Goods goods2 = GoodsFactory.getGoods("Android进阶之光");
    good2.showGoodsPrice("签名版");
    Goods goods3 = GoodsFactory.getGoods("Android进阶之光");
    goods3.showGoodsPrice("定制版");
```

### ==10. 策略模式==
定义一系列的算法，将每一种算法都封装起来，并可相互转换，使得算法可独立于使用者而单独变化。

有如下角色：
- 上下文角色：用来操作策略使用的上下文环境。屏蔽了高层模块对策略和算法的直接访问。
- 抽象策略角色：
- 具体策略角色：
1. 抽象策略角色

```
    public interface FightingStrategy {
    
        public void fighting();
    }
```

2. 具体策略角色

```
    public class WeakRivalStrategy implements FightingStrategy {
        
        @Override
        public void fighting() {
            ...
        }
    }
    
    public class CommonRivalStrategy implements FightingStrategy {
        
        @Override
        public void fighting() {
            ...
        }
    }
    
    public class StrongRivalStrategy implements FightingStrategy {
        
        @Override
        public void fighting() {
            ...
        }
    }
```

3. 上下文角色

```
    public class Context {
        private FightingStrategy fightingStrategy;
        
        public Context(FightingStrategy fightingStrategy) {
            this.fightingStrategy = fightingStrategy;
        }
        
        public void fighting() {
            fightingStrategy.fighting();
        }
    }
```
使用：

```
    Context context;
    context = new Context(new WeakRivalStrategy());
    context.fighting();
    context = new Context(new CommonRivalStrategy());
    context.fighting();
    context = new Context(new StrongRivalStrategy());
    context.fighting();
```
- 隐藏具体策略算法中的实现细节
- 避免使用多重条件语句
- 易于扩展
- 每一个策略都是一个类，复用性小
- 上层模块必须知道有哪些策略类，违背迪米特原则

### ==11. 模板方法模式==
定义一套算法框架，将某些步骤交给子类去实现，使得子类不需改变框架结构即可重写算法中的某些步骤。

有如下角色：
- 抽象类：定义一套算法框架
- 具体实现类：抽象类的具体实现
1. 抽象类

```
    public abstract class AbstractSwordsman {

        public final void fighting() {
            neigong();
            
            jingmai();
    
            if (hasWeapons()) {
                weapons();
            }
    
            moves();
    
            hook();
        }
    
        //空方法
        protected void hook() {
    
        }
    
        protected abstract void neigong();
    
        protected abstract void weapons();
    
        protected abstract void moves();
    
        //具体方法
        public void jingmai() {
            //....
        }
    
        protected boolean hasWeapons() {
            return true;
        }
    }
```

2. 具体实现类

```
    public class ZhangWuJi extends AbstractSwordsman {
    
        @Override
        protected void neigong() {
    
        }
    
        @Override
        protected void weapons() {
            //没有武器，不做处理
        }
    
        @Override
        protected void moves() {
    
        }
    
        @Override
        protected boolean hasWeapons() {
            return false;
        }
    }
    
    public class ZhangSanFeng extends AbstractSwordsman {

        @Override
        protected void neigong() {
    
        }
    
        @Override
        protected void weapons() {
    
        }
    
        @Override
        protected void moves() {
    
        }
    
        @Override
        protected void hook() {
            super.hook();
            //额外处理
        }
    }
```
使用：

```
    ZhangWuJi zhangWuJi = new ZhangWuJi();
    zhangWuJi.fighting();
    ZhangSanFeng zhangSanFeng = new ZhangSanFeng();
    zhangSanFeng.fighting();
```
- 可以实现子类对父类的反向控制
- 可以把核心或固定的逻辑搬移到父类，其它细节交给子类实现
- 每个不同的实现类都需要定义一个子类，复用性小

### ==12. 观察者模式（发布-订阅模式）==
定义对象间一对多的依赖关系，每当这个对象的状态发生改变时，其他的对象都会接收到通知并被自动更新。

有如下角色：
- 抽象被观察者：将所有已注册的观察者对象保存在一个集合中
- 具体被观察者：当内部状态发生改变时，将会通知所有已注册的观察者
- 抽象观察者：定义了一个更新接口，当被观察者状态改变时更新自己
- 具体观察者：实现抽象观察者的更新接口
1. 抽象观察者

```
    public interface observer {
    
        public void update(String message);
    }
```

2. 具体观察者

```
    public class WeiXinUser implements observer {
        private String name;
        
        public WeiXinUser(String name) {
            this.name = name;
        }
        
        @Override
        public void update(String message) {
            ...
        }
    }
```

3. 抽象被观察者

```
    public interface observable {
    
        public void addWeiXinUser(WeiXinUser weiXinUser);
        
        public void removeWeiXinUser(WeiXinUser weiXinUser);
        
        public void notify(String message);
    }

```

4. 具体被观察者

```
    public class Subscription implements observable {
        private List<WeiXinUser> mUserList = new ArrayList();
        
        @Override
        public void addWeiXinUser(WeiXinUser weiXinUser) {
            mUserList.add(weiXinUser);
        }
        
        @Override
        public void removeWeiXinUser(WeiXinUser weiXinUser) {
            mUserList.remove(weiXinUser);
        }
        
        @Override
        public void notify(String message) {
            for(WeiXinUser weiXinUser : mUserList) {
                weiXinUser.update(message);
            }
        }
    }

```
5.使用

```
    Subscription subscription = new Subscription();
    WeiXinUser weiXinUserA = new WeiXinUser("AAA");
    WeiXinUser weiXinUserB = new WeiXinUser("BBB");
    WeiXinUser weiXinUserC = new WeiXinUser("CCC");
    subscription.addWeiXinUser(weiXinUserA);
    subscription.addWeiXinUser(weiXinUserB);
    subscription.addWeiXinUser(weiXinUserC);
    subscription.notify("ahhhhhhh");
    
```
- 实现了观察者和被观察者之间的抽象耦合，容易扩展
- 有利于建立一套触发机制
- 一个被观察者卡顿，会影响整体的执行效率，采用异步机制可解决此类问题