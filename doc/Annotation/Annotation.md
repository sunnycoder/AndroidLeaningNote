Android 注解详解

-----------

## 概述

    注解如同标签

注解是一种元数据, 可以添加到java代码中. 类、方法、变量、参数、包都可以被注解，注解对注解的代码没有直接影响.

首先, 明确一点: 注解并没有什么魔法, 之所以产生作用, 是对其解析后做了相应的处理. 注解仅仅只是个标记罢了.

<b>有些时候注解不做处理， 只是为了做个标记，参见Android中有很多这种用法</b>

定义注解用的关键字是@interface

## 语法
    
    其实同 classs 和 interface 一样，注解也属于一种类型。它是在 Java SE 5.0 版本中开始引入的概念。  

### 注解的定义

注解通过 @interface 关键字进行定义。
```
public @interface TestAnnotation {
}
```
它的形式跟接口很类似，不过前面多了一个 @ 符号。上面的代码就创建了一个名字为 TestAnnotaion 的注解。

你可以简单理解为创建了一张名字为 TestAnnotation 的标签。

### 注解的应用

解的的使用方法是什么呢？

```
@TestAnnotation
public class Test {
}
```
创建一个类 Test,然后在类定义的地方加上 @TestAnnotation 就可以用 TestAnnotation 注解这个类了。

你可以简单理解为将 TestAnnotation 这张标签贴到 Test 这个类上面。

不过，要想注解能够正常工作，还需要介绍一下一个新的概念那就是元注解。 

## 元注解

元注解是可以注解到注解上的注解，或者说元注解是一种基本注解，但是它能够应用到其它的注解上面。

如果难于理解的话，你可以这样理解。元注解也是一张标签，但是它是一张特殊的标签，它的作用和目的就是给其他普通的标签进行解释说明的。

元标签有 @Retention、@Documented、@Target、@Inherited、@Repeatable 5 种。

### @Retention
Retention 的英文意为保留期的意思。当 @Retention 应用到一个注解上的时候，它解释说明了这个注解的的存活时间。

它的取值如下：
- RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时它将被丢弃忽视。
- RetentionPolicy.CLASS 注解只被保留到编译进行的时候，它并不会被加载到 JVM 中。
- RetentionPolicy.RUNTIME 注解可以保留到程序运行的时候，它会被加载进入到 JVM 中，所以在程序运行时可以获取到它们。

我们可以这样的方式来加深理解，@Retention 去给一张标签解释的时候，它指定了这张标签张贴的时间。@Retention 相当于给一张标签上面盖了一张时间戳，时间戳指明了标签张贴的时间周期。 
```
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
}
```
上面的代码中，我们指定 TestAnnotation 可以在程序运行周期被获取到，因此它的生命周期非常的长。 

### @Documented
顾名思义，这个元注解肯定是和文档有关。它的作用是能够将注解中的元素包含到 Javadoc 中去。

### @Target
Target 是目标的意思，@Target 指定了注解运用的地方。

你可以这样理解，当一个注解被 @Target 注解时，这个注解就被限定了运用的场景。

类比到标签，原本标签是你想张贴到哪个地方就到哪个地方，但是因为 @Target 的存在，它张贴的地方就非常具体了，比如只能张贴到方法上、类上、方法参数上等等。@Target 有下面的取值

    ElementType.ANNOTATION_TYPE 可以给一个注解进行注解
    ElementType.CONSTRUCTOR 可以给构造方法进行注解
    ElementType.FIELD 可以给属性进行注解
    ElementType.LOCAL_VARIABLE 可以给局部变量进行注解
    ElementType.METHOD 可以给方法进行注解
    ElementType.PACKAGE 可以给一个包进行注解
    ElementType.PARAMETER 可以给一个方法内的参数进行注解
    ElementType.TYPE 可以给一个类型进行注解，比如类、接口、枚举


### @Inherited

Inherited 是继承的意思，但是它并不是说注解本身可以继承，而是说如果一个超类被 @Inherited 注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解。
说的比较抽象。代码来解释。

```
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@interface Test {}


@Test
public class A {}


public class B extends A {}
```
注解 Test 被 @Inherited 修饰，之后类 A 被 Test 注解，类 B 继承 A,类 B 也拥有 Test 这个注解。 

### @Repeatable
Repeatable 自然是可重复的意思。@Repeatable 是 Java 1.8 才加进来的，所以算是一个新的特性。

什么样的注解会多次应用呢？通常是注解的值可以同时取多个。

举个例子，一个人他既是程序员又是产品经理,同时他还是个画家。
```
@interface Persons {
  Person[] value();
}

@Repeatable(Persons.class)
@interface Person {
  String role
  default "";
}

@Person(role = "artist")
@Person(role = "coder")
@Person(role = "PM")
public class SuperMan {
}
```

注意上面的代码，@Repeatable 注解了 Person。而 @Repeatable 后面括号中的类相当于一个容器注解。

什么是容器注解呢？就是用来存放其它注解的地方。它本身也是一个注解。

我们再看看代码中的相关容器注解。

```
@interface Persons {
    Person[]  value();
}
```
按照规定，它里面必须要有一个 value 的属性，属性类型是一个被 @Repeatable 注解过的注解数组，注意它是数组。

如果不好理解的话，可以这样理解。Persons 是一张总的标签，上面贴满了 Person 这种同类型但内容不一样的标签。把 Persons 给一个 SuperMan 贴上，相当于同时给他贴了程序员、产品经理、画家的标签。

我们可能对于 @Person(role=”PM”) 括号里面的内容感兴趣，它其实就是给 Person 这个注解的 role 属性赋值为 PM ，大家不明白正常，马上就讲到注解的属性这一块。

## 注解的属性
注解的属性也叫做成员变量。注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无形参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型。

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {

    int id();

    String msg();

}
```
上面代码定义了 TestAnnotation 这个注解中拥有 id 和 msg 两个属性。在使用的时候，我们应该给它们进行赋值。

赋值的方式是在注解的括号内以 value=”” 形式，多个属性之前用 ，隔开。 

```
@TestAnnotation(id=3,msg="hello annotation")
public class Test {

}
```
需要注意的是，在注解中定义属性时它的类型必须是 8 种基本数据类型外加 类、接口、注解及它们的数组。

注解中属性可以有默认值，默认值需要用 default 关键值指定。比如：
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
  public int id() default -1;

  public String msg() default "Hi";
}
```
TestAnnotation 中 id 属性默认值为 -1，msg 属性默认值为 Hi。
它可以这样应用。

```
@TestAnnotation()
public class Test {}
```
因为有默认值，所以无需要再在 @TestAnnotation 后面的括号里面进行赋值了，这一步可以省略。

另外，还有一种情况。如果一个注解内仅仅只有一个名字为 value 的属性时，应用这个注解时可以直接接属性值填写到括号内。

```
public @interface Check {
    String value();
}
```

上面代码中，Check 这个注解只有 value 这个属性。所以可以这样应用。

```
@Check("hi")
int a;
```

这和下面的效果是一样的

```
@Check(value="hi")
int a;
```

最后，还需要注意的一种情况是一个注解没有任何属性。比如

```
public @interface Perform {}
```
那么在应用这个注解的时候，括号都可以省略。
```
@Perform
public void testMethod(){}
```

## Java预置的注解
Java 语言本身已经提供了几个现成的注解。

更多的注解详见javax.inject包

### @Deprecated
这个元素是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。 

```
public class Hero {
  @Deprecated
  public void say() {
    System.out.println("Noting has to say!");
  }

  public void speak() {
    System.out.println("I have a dream!");
  }
}
```

###  @Override
这个大家应该很熟悉了，提示子类要复写父类中被 @Override 修饰的方法

### @SuppressWarnings
阻止警告的意思。之前说过调用被 @Deprecated 注解的方法后，编译器会警告提醒，而有时候开发者会忽略这种警告，他们可以在调用的地方通过 @SuppressWarnings 达到目的。

```
@SuppressWarnings("deprecation")
public void test1(){
    Hero hero = new Hero();
    hero.say();
    hero.speak();
}
```
### @SafeVarargs
参数安全类型注解。它的目的是提醒开发者不要用参数做一些不安全的操作,它的存在会阻止编译器产生 unchecked 这样的警告。它是在 Java 1.7 的版本中加入的。
```
@SafeVarargs // Not actually safe! 
  static void m(List<String>... stringLists) {
    Object[] array = stringLists;
    List<Integer> tmpList = Arrays.asList(42);
    array[0] = tmpList;
// Semantically invalid, but compiles without warnings 
    String s = stringLists[0].get(0);
// Oh no, ClassCastException at runtime! 
  }

```
上面的代码中，编译阶段不会报错，但是运行时会抛出 ClassCastException 这个异常，所以它虽然告诉开发者要妥善处理，但是开发者自己还是搞砸了。

Java 官方文档说，未来的版本会授权编译器对这种不安全的操作产生错误警告。 
### @FunctionalInterface
函数式接口注解，这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。

    函数式接口 (Functional Interface) 就是一个具有一个方法的普通接口。 
```
@FunctionalInterface public interface Runnable { /**
 * When an object implementing interface <code>Runnable</code> is used
 * to create a thread, starting the thread causes the object's
 * <code>run</code> method to be called in that separately executing
 * thread.
 * <p>
 * The general contract of the method <code>run</code> is that it may
 * take any action whatsoever.
 *
 * @see     java.lang.Thread#run()
 */ public abstract void run(); }

```
我们进行线程开发中常用的 Runnable 就是一个典型的函数式接口，上面源码可以看到它就被 @FunctionalInterface 注解。

可能有人会疑惑，函数式接口标记有什么用，这个原因是函数式接口可以很容易转换为 Lambda 表达式。

### @Qualifier
任何人都可以定义一个新的修饰语，一个qualifier注解应该满足如下条件：

    定义的注解类有@Qualifier，@Retention(RUNTIME)和@Documented。
    可以有属性
    可以是公共API的一部分
    可以用@Target注解限定使用范围

## Android中的注解
使用代码检查工具(如Lint)可以帮助您发现问题并改进代码，但是检查工具只能推断这么多。例如，Android资源id使用int来标识字符串、图形、颜色和其他资源类型，因此检查工具无法告诉您何时指定了字符串资源，何时指定了颜色。这种情况意味着您的应用程序可能呈现不正确或根本无法运行，即使您使用代码检查。

注解允许您为Lint等代码检查工具提供提示，以帮助检测这些更微妙的代码问题。它们作为元数据标记添加，您可以将它们附加到变量、参数和返回值，以检查方法返回值、传递的参数、本地变量和字段。当与代码检查工具一起使用时，注释可以帮助您检测问题，比如空指针异常和资源类型冲突。

Android通过注解库支持多种注解。您可以通过android.support访问注解包。

注意：如果模块依赖于注释处理器，则必须使用“annotationProcessor”依赖配置来添加该依赖。要了解更多信息，请阅读使用注释处理器依赖配置。

详见[官方文档](https://developer.android.google.cn/studio/write/annotations)

### Nullness annotations
添加@null和@nonnull注解，以检查给定变量、参数或返回值的null。
- @Nullable注释指示变量、参数或返回值可以为空
-   @NonNull则指示变量、参数或返回值不能为空。

例如，如果一个包含空值的局部变量作为参数传递给一个带有@NonNull注释附加到该参数的方法，则构建代码将生成一个警告，指示非空冲突。另一方面，如果尝试引用@Nullable标记的方法的结果，而不首先检查结果是否为空，则会生成null警告。如果方法的每次使用都应该显式地检查为空，那么您应该只在方法的返回值上使用@Nullable。

下面的示例将@NonNull注释附加到上下文和attrs参数，以检查传递的参数值是否为空。它还检查onCreateView()方法本身是否不返回null
```
import android.support.annotation.NonNull;
...

    /** Add support for inflating the <fragment> tag. **/
    @NonNull
    @Override
    public View onCreateView(String name, @NonNull Context context,
      @NonNull AttributeSet attrs) {
      ...
      }
...
```
### Resource annotations
验证资源类型可能很有用，因为Android引用资源(如drawable和string资源)是作为整数传递的。期望参数引用特定类型资源(例如Drawables)的代码可以传递预期的int引用类型，但实际上引用的是不同类型的资源，比如R。字符串资源。

例如，添加@StringRes注解来检查资源参数是否包含R.string引用，如下所示:
```
public abstract void setTitle(@StringRes int resId)
```
在代码检查期间，如果没有传入R.string类型，注解将生成一个警告。

其他资源类型的注释，如@DrawableRes、@dimensions、@ColorRes和@InterpolatorRes，可以使用相同的注解格式添加，并在代码检查期间运行。如果参数支持多种资源类型，则可以在给定的参数上放置多个注释。使用@AnyRes表示带注释的参数可以是任何类型的R资源。

虽然可以使用@ColorRes指定参数应该是颜色资源，但是颜色整数(RRGGBB或AARRGGBB格式)不能识别为颜色资源。相反，使用@ColorInt注释表明参数必须是一个颜色整数。构建工具将标记传递颜色资源ID(如android.R.color)的不正确代码。对带注释的方法使用黑色，而不是颜色整数。

### Thread annotations
线程注解检查方法是否从特定类型的线程调用。支持以下线程注释:

- @MainThread
- @UiThread
- @WorkerThread
- @BinderThread
- @AnyThread

    
    注意：构建工具将@MainThread和@UiThread注释视为可互换的，因此您可以从@MainThread方法调用@UiThread方法，反之亦然。然而，对于在不同线程上具有多个视图的系统应用程序，UI线程可能与主线程不同。因此，您应该用@UiThread注释与应用程序视图层次结构相关联的方法，并且只注释与应用程序生命周期相关联的方法。

如果类中的所有方法都共享相同的线程需求，那么可以向类中添加单个线程注释，以验证类中的所有方法都是从相同类型的线程调用的。

线程注释的一个常见用法是验证AsyncTask类中的方法重写，因为该类只在UI线程上执行后台操作并发布结果。

### Value constraint annotations
使用@IntRange、@FloatRange和@Size注解验证传递参数的值。
当将@IntRange和@FloatRange应用于用户可能会弄错范围的参数时，它们都非常有用。

@IntRange注释验证整数或长参数值在指定范围内。
下面的示例确保alpha参数包含一个从0到255的整数值:
```
public void setAlpha(@IntRange(from=0,to=255) int alpha) { ... }
```

@Size注释检查集合或数组的大小以及字符串的长度。@Size注释可用于验证以下尺寸:
最小尺寸(如@Size(min=2))
最大尺寸(如@Size(max=2))
准确尺寸(如@Size(2))
大小必须是倍数的数字(例如@Size(倍数=2))

例如，@Size(min=1)检查集合是否为空，@Size(3)验证数组是否恰好包含三个值。下面的示例确保location数组至少包含一个元素:
```
void getLocation(View button, @Size(min=1) int[] location) {
    button.getLocationOnScreen(location);
}
```

### Permission annotations
使用@RequiresPermission注释验证方法调用者的权限。要检查列表中的单个权限(有效权限)，请使用anyOf属性。要检查一组权限，请使用allOf属性。下面的示例注释setWallpaper()方法，以确保方法的调用者具有权限。SET_WALLPAPERS许可:
```
@RequiresPermission(Manifest.permission.SET_WALLPAPER)
public abstract void setWallpaper(Bitmap bitmap) throws IOException;
```

这个例子要求copyFile()方法的调用者同时具有对外部存储的读写权限:
```
@RequiresPermission(allOf = {
    Manifest.permission.READ_EXTERNAL_STORAGE,
    Manifest.permission.WRITE_EXTERNAL_STORAGE})
public static final void copyFile(String dest, String source) {
    //...
}
```
对于意图的权限，将权限要求放在定义意图动作名称的字符串字段上:
```
@RequiresPermission(android.Manifest.permission.BLUETOOTH)
public static final String ACTION_REQUEST_DISCOVERABLE =
            "android.bluetooth.adapter.action.REQUEST_DISCOVERABLE";
```

对于内容提供程序的权限，您需要单独的读写权限，将每个权限需求包装在一个@RequiresPermission.Read或@RequiresPermission.Write注解:
```
@RequiresPermission.Read(@RequiresPermission(READ_HISTORY_BOOKMARKS))
@RequiresPermission.Write(@RequiresPermission(WRITE_HISTORY_BOOKMARKS))
public static final Uri BOOKMARKS_URI = Uri.parse("content://browser/bookmarks");
```

### Return value annotations
使用@CheckResult注释验证方法的结果或返回值是否实际使用。

与其用@CheckResult注释每一个非void方法，不如添加注释来澄清可能令人困惑的方法的结果。例如，新的Java开发人员经常错误地认为<string>.trim()从原始字符串中删除了空格。</string>使用@CheckResult标志注释方法时，使用<string>.trim()，调用方不会对方法的返回值做任何事情。</string>

下面的示例注释checkPermissions()方法，以确保实际引用了该方法的返回值。它还将cepermission()方法命名为建议开发人员作为替代的方法:
```
@CheckResult(suggest="#enforcePermission(String,int,int,String)")
public abstract int checkPermission(@NonNull String permission, int pid, int uid);
```

### CallSuper annotations
使用@CallSuper注释验证重写方法调用方法的超级实现。
下面的示例注释了onCreate()方法，以确保任何重写方法实现都调用super.onCreate():
```
@CallSuper
protected void onCreate(Bundle savedInstanceState) {
}
```

### Typedef annotations
使用@IntDef和@StringDef注释，这样您就可以创建整数和字符串集的枚举注释来验证其他类型的代码引用。
Typedef注释确保特定的参数、返回值或字段引用一组特定的常量。它们还允许代码完成自动提供允许的常量。

Typedef注释使用@interface来声明新的枚举注释类型。@IntDef和@StringDef注释以及@Retention注释新注释，对于定义枚举类型是必要的。@Retention(RetentionPolicy.SOURCE)注释告诉编译器不要将枚举的注释数据存储在.class文件中。

下面的示例演示了创建注释的步骤，该注释确保作为方法参数传递的值引用定义的常量之一:
```
import android.support.annotation.IntDef;
//...
public abstract class ActionBar {
    //...
    // Define the list of accepted constants and declare the NavigationMode annotation
    @Retention(RetentionPolicy.SOURCE)
    @IntDef({NAVIGATION_MODE_STANDARD, NAVIGATION_MODE_LIST, NAVIGATION_MODE_TABS})
    public @interface NavigationMode {}

    // Declare the constants
    public static final int NAVIGATION_MODE_STANDARD = 0;
    public static final int NAVIGATION_MODE_LIST = 1;
    public static final int NAVIGATION_MODE_TABS = 2;

    // Decorate the target methods with the annotation
    @NavigationMode
    public abstract int getNavigationMode();

    // Attach the annotation
    public abstract void setNavigationMode(@NavigationMode int mode);
}
```
在构建此代码时，如果模式参数不引用定义的常量(NAVIGATION_MODE_STANDARD、NAVIGATION_MODE_LIST或NAVIGATION_MODE_TABS)，就会生成警告。

您还可以组合@IntDef和@IntRange，以指示整数可以是给定的常量集或范围内的值。

#### Enable combining constants with flags
如果用户可以将允许常数与标志(如| & ^,等等),您可以定义一个注释标记属性来检查是否一个参数或返回值引用一个有效的模式。下面的示例使用有效的DISPLAY_常量列表创建DisplayOptions注释:
```
import android.support.annotation.IntDef;
...

@IntDef(flag=true, value={
        DISPLAY_USE_LOGO,
        DISPLAY_SHOW_HOME,
        DISPLAY_HOME_AS_UP,
        DISPLAY_SHOW_TITLE,
        DISPLAY_SHOW_CUSTOM
})
@Retention(RetentionPolicy.SOURCE)
public @interface DisplayOptions {}

...
```
当您使用注释标志构建代码时，如果修饰参数或返回值没有引用有效模式，就会生成警告。

### Keep annotation
@Keep注释确保在构建时代码缩减时不会删除带注释的类或方法。
这个注释通常被添加到通过反射访问的方法和类中，以防止编译器将代码视为未使用的代码。

有关如何缩小代码并指定哪些代码不应该删除的详细信息，请参阅缩小代码和资源。

### Code visibility annotations
使用以下注释表示代码的特定部分的可见性，例如方法、类、字段或包。

#### Make visible for testing
@VisibleForTesting注释表明，要使方法可测试，带注释的方法比通常需要的更可见。这个注释有一个可选的否则参数，可以让您指定方法的可见性，如果不需要使其对测试可见的话。Lint使用否则参数来增强预期的可见性。

在下面的示例中，myMethod()通常是私有的，但是对于测试，它是包私有的。使用以下可见测试。如果该方法是从私有访问允许的上下文(例如从不同的编译单元)之外调用，lint将显示一条消息。
```
@VisibleForTesting(otherwise = VisibleForTesting.PRIVATE)
void myMethod() { ... }
```
您还可以指定@VisibleForTesting(否则= visiblefortest . none)，以指示只存在用于测试的方法。这个表单与使用@RestrictTo(测试)相同。它们都执行相同的棉绒检查。

#### Restrict an API
@RestrictTo注释指示对带注释的API(包、类或方法)的访问受到以下限制。

##### Subclasses
使用注释表单@RestrictTo(RestrictTo.scope.subclasses)来限制API对子类的访问。

只有扩展带注释的类的类才能访问这个API。Java protected修饰符的限制还不够，因为它允许从同一包中不相关的类访问。此外，在某些情况下，您可能希望为将来的灵活性而保留一个方法public，因为您永远不能使以前受保护和覆盖的方法public，但是您希望提供一个提示，即该类仅用于类内或子类中的用法。

##### Libraries
使用注释表单@RestrictTo(RestrictTo.scope.group_id)来限制对库的API访问。

只有库代码才能访问带注释的API。这不仅允许您将代码组织到您想要的任何包层次结构中，还允许您在一组相关库之间共享代码。这个选项已经对支持库可用，这些支持库有很多实现代码，这些代码不是用于外部使用的，但必须是公共的，以便在各种互补的支持库之间共享。

##### Testing
使用注释表单@RestrictTo(RestrictTo.scope.tests)以防止其他开发人员访问您的测试api。

只有测试代码才能访问带注释的API。这可以防止其他开发人员将api用于您仅用于测试目的的开发。

## 注解与反射
    要想正确检阅注解，离不开一个手段，那就是反射。 

注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解
```
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
然后通过 getAnnotation() 方法来获取 Annotation 对象。
```
 public <A extends Annotation> A getAnnotation(Class<A> annotationClass) {}
```
或者是 getAnnotations() 方法。
```
public Annotation[] getAnnotations() {}
```
前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解。

如果获取到的 Annotation 如果不为 null，则就可以调用它们的属性方法了。比如
```
@TestAnnotation()
public class Test {
  public static void main(String[] args) {
    boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);
    if (hasAnnotation) {
      TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
      System.out.println("id:" + testAnnotation.id());
      System.out.println("msg:" + testAnnotation.msg());
    }
  }
}

```

程序的运行结果是：
```
id:-1
msg:
```

这个正是 TestAnnotation 中 id 和 msg 的默认值。

上面的例子中，只是检阅出了注解在类上的注解，其实属性、方法上的注解照样是可以的。同样还是要假手于反射。 
```
@TestAnnotation(msg = "hello")
public class Test {
  @Check(value = "hi")
  int a;

  @Perform
  public void testMethod() {
  }

  @SuppressWarnings("deprecation")
  public void test1() {
    Hero hero = new Hero();
    hero.say();
    hero.speak();
  }

  public static void main(String[] args) {
    boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);
    if (hasAnnotation) {
      TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
//获取类的注解 
      System.out.println("id:" + testAnnotation.id());
      System.out.println("msg:" + testAnnotation.msg());
    }
    try {
      Field a = Test.class.getDeclaredField("a");
      a.setAccessible(true);
//获取一个成员变量上的注解 
      Check check = a.getAnnotation(Check.class);
      if (check != null) {
        System.out.println("check value:" + check.value());
      }
      Method testMethod = Test.class.getDeclaredMethod("testMethod");
      if (testMethod != null) {
        // 获取方法中的注解 
        Annotation[] ans = testMethod.getAnnotations();
        for (int i = 0; i < ans.length; i++) {
          System.out.println("method testMethod annotation:" + ans[i].annotationType().getSimpleName());
        }
      }
    } catch (NoSuchFieldException e) { // TODO Auto-generated catch block 
      e.printStackTrace();
      System.out.println(e.getMessage());
    } catch (SecurityException e) {
      // TODO Auto-generated catch block 
      e.printStackTrace();
      System.out.println(e.getMessage());
    } catch (NoSuchMethodException e) {
      // TODO Auto-generated catch block 
      e.printStackTrace();
      System.out.println(e.getMessage());
    }
  }
}
```
它们的结果如下：
```
id:-1
msg:hello
check value:hi
method testMethod annotation:Perform
```

需要注意的是，如果一个注解要在运行时被成功提取，那么 @Retention(RetentionPolicy.RUNTIME) 是必须的。


## 注解的使用场景
注解到底有什么用呢？ 

我们先看看官方最严谨的文档中的描述

    注解是一系列元数据，它提供数据用来解释程序代码，但是注解并非是所解释的代码本身的一部分。注解对于代码的运行效果没有直接影响。

    注解有许多用处，主要如下：
    - 提供信息给编译器： 编译器可以利用注解来探测错误和警告信息
    - 编译阶段时的处理： 软件工具可以用来利用注解信息来生成代码、Html文档或者做其它相应处理。
    - 运行时的处理： 某些注解可以在程序运行的时候接受代码的提取 

注解主要针对的是编译器和其它工具软件(SoftWare tool)。

当开发者使用了Annotation 修饰了类、方法、Field 等成员之后，这些 Annotation 不会自己生效，必须由开发者提供相应的代码来提取并处理 Annotation 信息。这些处理提取和处理 Annotation 的代码统称为 APT（Annotation Processing Tool)。 

注解有什么用？给谁用？给 编译器或者 APT 用的。

## 注解应用实例

### JUnit
JUnit 这个是一个测试框架，典型使用方法如下：
```
public class ExampleUnitTest {
    @Test
    public void addition_isCorrect() throws Exception {
        assertEquals(4, 2 + 2);
    }
}
```
@Test 标记了要进行测试的方法 addition_isCorrect(). 

### ButterKnife
ButterKnife 是 Android 开发中大名鼎鼎的 IOC 框架，它减少了大量重复的代码。

```
public class MainActivity extends AppCompatActivity {
  @BindView(R.id.tv_test)
  TextView mTv;

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    ButterKnife.bind(this);
  }
}

```

### Dagger2
也是一个很有名的依赖注入框架。 

### Retrofit
很牛逼的 Http 网络访问框架
```
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
  Retrofit retrofit = new Retrofit.Builder().baseUrl("https://api.github.com/").build();
  GitHubService service = retrofit.create(GitHubService.class);

```

## 总结

- 如果注解难于理解，你就把它类同于标签，标签为了解释事物，注解为了解释代码。
- 注解的基本语法，创建如同接口，但是多了个 @ 符号。
- 注解的元注解。
- 注解的属性。
- 注解主要给编译器及工具类型的软件用的。
- 注解的提取需要借助于 Java 的反射技术，反射比较慢，所以注解使用时也需要谨慎计较时间成本。


## 参考文档

[Android进阶之自定义注解](https://www.jianshu.com/p/a13c6326671d)

[秒懂，Java 注解 （Annotation）你可以这样学](https://blog.csdn.net/briblue/article/details/73824058)

[Android官方注解文档](https://developer.android.google.cn/studio/write/annotations)