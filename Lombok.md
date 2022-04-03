# 简介

Project Lombok 是可以自动往编辑器和构建工具中插入的一个java库，为您的 java 增添趣味。
永远不要再额外编写 getter 或 equals 方法，仅需要使用一个注释，您的类就有一个功能齐全的构建器、自动化您的日志变量等等。

# 对比

没有使用Lombok之前：

```java
public class Person {
    private Integer id;
    private String name;
    public Integer getId() {
        return id;
    }
    public void setId(Integer id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

使用Lombok之后：

```java
@Getter
@Setter
public class Person {
    private Integer id;
    private String name;
}
```

# 特性

## 1. @Getter / @Setter

[官方文档](https://projectlombok.org/features/GetterSetter)

您可以使用 @Getter 和/或 @Setter 注释任何字段，以让 lombok 自动生成默认的 getter/setter。默认的 getter 只是简单地返回该字段。

除非明确指定 AccessLevel，否则生成的 getter/setter 方法将是公共的。合法访问级别为 PUBLIC、PROTECTED、PACKAGE 和 PRIVATE。

您还可以在类上放置@Getter 和/或@Setter 注释。在这种情况下，就好像您使用注解对该类中的所有非静态字段进行了注解。

```java
import lombok.AccessLevel;
import lombok.Getter;
import lombok.Setter;

public class GetterSetterExample {
  @Getter
  @Setter 
  private int age = 10;

  @Setter(AccessLevel.PROTECTED)
  private String name;
}
```

生成的代码：

```java
public class GetterSetterExample {
  private int age = 10;

  private String name;

  public int getAge() {
    return age;
  }

  public void setAge(int age) {
    this.age = age;
  }

  protected void setName(String name) {
    this.name = name;
  }
}
```

## 2. @ToString

[官方文档](https://projectlombok.org/features/ToString)

任何类定义都可以用 @ToString 注释，让 lombok 生成 toString() 方法的实现。默认情况下，它将按顺序打印每个字段，以逗号分隔。

通过将 includeFieldNames 参数设置为 true，输出时将包含每一个字段的名字。

默认情况下，将打印所有非静态字段。如果要跳过某些字段，可以使用@ToString.Exclude 注释这些字段。或者，您可以使用@ToString(onlyExplicitlyIncluded = true) 准确指定您希望使用的字段，然后使用@ToString.Include 标记您想要包含的每个字段。

通过将 callSuper 设置为 true，您可以将超类中 toString 的输出包含到本输出中。

您还可以在 toString 中包含方法调用的输出。只能包含不带参数的实例（非静态）方法。为此，请使用@ToString.Include 标记该方法。

您可以使用@ToString.Include(name = "some other name") 更改用于标识成员的名称，并且可以通过@ToString.Include(rank = -1) 更改打印成员的顺序。没有rank的成员被认为具有rank=0，首先打印更高等级的成员，并且相同等级的成员按照它们在源文件中出现的顺序打印。

```java
import lombok.ToString;

@ToString
public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  @ToString.Exclude
  private int id;

  public String getName() {
    return this.name;
  }

  @ToString(callSuper=true, includeFieldNames=true)
  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

生成的代码：

```java
import java.util.Arrays;

public class ToStringExample {
  private static final int STATIC_VAR = 10;
  private String name;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;

  public String getName() {
    return this.name;
  }

  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }

    @Override
    public String toString() {
      return "Square(super=" + super.toString() + ", width=" + this.width + ", height=" + this.height + ")";
    }
  }

  @Override
  public String toString() {
    // 没有成员变量id
    return "ToStringExample(" + this.getName() + ", " + this.shape + ", " + Arrays.deepToString(this.tags) + ")";
  }
}
```

## 3. @EqualsAndHashCode

[官方文档](https://projectlombok.org/features/EqualsAndHashCode)

任何类定义都可以使用 @EqualsAndHashCode 进行注释，以让 lombok 生成 equals(Object other) 和 hashCode() 方法的实现。默认情况下，它将使用所有non-static、non-transient字段，但您可以通过使用 @EqualsAndHashCode.Include 或@EqualsAndHashCode.Exclude来准确指定您希望使用的字段或方法。

如果将 @EqualsAndHashCode 应用于扩展类，则需要格外注意。通常，为扩展类自动生成 equals 和 hashCode 方法是一个坏主意，因为超类还定义了字段，这些字段也需要 equals/hashCode 代码，但不会生成此代码。通过将 callSuper 设置为 true，您可以在生成的方法中包含超类的 equals 和 hashCode 方法。对于扩展类的hashCode方法，超类的hashCode()的结果包含在hash算法中，而对于扩展类的equals方法，如果超类的equals方法实现认为不等于传入的对象，扩展类生成的方法会返回false。请注意，并非所有 equals 实现都能正确处理这种情况。但是，lombok 生成的 equals 实现确实可以正确处理这种情况，因此如果您的超类也有 lombok 生成的 equals 方法，您可以安全地调用它。如果你有一个显式的超类，你必须为 callSuper 提供一些值来确认你已经考虑过它，不这样做会导致警告。

当你不扩展任何东西（你扩展 java.lang.Object）时将 callSuper 设置为 true 会产生一个编译时错误。扩展类时不将 callSuper 设置为 true 会生成警告。

```java
import lombok.EqualsAndHashCode;

@EqualsAndHashCode
public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  @EqualsAndHashCode.Exclude
  private Shape shape = new Square(5, 10);
  private String[] tags;
  @EqualsAndHashCode.Exclude
  private int id;

  public String getName() {
    return this.name;
  }

  @EqualsAndHashCode(callSuper=true)
  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```

生成的代码：

```java
import java.util.Arrays;

public class EqualsAndHashCodeExample {
  private transient int transientVar = 10;
  private String name;
  private double score;
  private Shape shape = new Square(5, 10);
  private String[] tags;
  private int id;

  public String getName() {
    return this.name;
  }

  @Override
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof EqualsAndHashCodeExample)) return false;
    EqualsAndHashCodeExample other = (EqualsAndHashCodeExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (Double.compare(this.score, other.score) != 0) return false;
    if (!Arrays.deepEquals(this.tags, other.tags)) return false;
    return true;
  }

  @Override
  public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.score);
    result = (result*PRIME) + (this.name == null ? 43 : this.name.hashCode());
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.tags);
    return result;
  }

  protected boolean canEqual(Object other) {
    return other instanceof EqualsAndHashCodeExample;
  }

  public static class Square extends Shape {
    private final int width, height;

    public Square(int width, int height) {
      this.width = width;
      this.height = height;
    }

    @Override
    public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Square)) return false;
      Square other = (Square) o;
      if (!other.canEqual((Object)this)) return false;
      if (!super.equals(o)) return false;
      if (this.width != other.width) return false;
      if (this.height != other.height) return false;
      return true;
    }

    @Override
    public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + super.hashCode();
      result = (result*PRIME) + this.width;
      result = (result*PRIME) + this.height;
      return result;
    }

    protected boolean canEqual(Object other) {
      return other instanceof Square;
    }
  }
}
```

## 4. @NoArgsConstructor/@RequiredArgsConstructor/@AllArgsConstructor

这三个注解都作用于类上，都将生成构造函数。

- @NoArgsConstructor 将生成一个没有参数的构造函数。 
  
  如果无法生成，则会导致编译器错误，除非使用 @NoArgsConstructor(force = true)，否则所有 final 字段都将初始化为 0 / false / null。

- @RequiredArgsConstructor 将生成带参数的构造函数。
  
  ****所有未初始化的 final 字段，以及任何标记为 @NonNull 且未声明的字段都将成为参数。**** 
  
  对于标有@NonNull 的字段，还会生成显式空检查。 如果标记有 @NonNull 的字段包含 null，则构造函数将抛出 NullPointerException。
  
  参数的顺序与字段在类中出现的顺序相匹配。

- @AllArgsConstructor 将生成包含类中每一个参数的构造函数。
  
  标有@NonNull 的字段会对这些参数进行空检查。

```java
import lombok.AccessLevel;
import lombok.RequiredArgsConstructor;
import lombok.AllArgsConstructor;
import lombok.NonNull;

@RequiredArgsConstructor(staticName = "of")
@AllArgsConstructor(access = AccessLevel.PROTECTED)
public class ConstructorExample<T> {
  private int x, y;
  @NonNull
  private T description;

  @NoArgsConstructor
  public static class NoArgsExample {
    @NonNull
    private String field;
  }
}
```

生成的代码：

```java
public class ConstructorExample<T> {
  private int x, y;
  @NonNull
  private T description;

  private ConstructorExample(T description) {
    if (description == null) throw new NullPointerException("description");
    this.description = description;
  }

  public static <T> ConstructorExample<T> of(T description) {
    return new ConstructorExample<T>(description);
  }

  @java.beans.ConstructorProperties({"x", "y", "description"})
  protected ConstructorExample(int x, int y, T description) {
    if (description == null) throw new NullPointerException("description");
    this.x = x;
    this.y = y;
    this.description = description;
  }

  public static class NoArgsExample {
    @NonNull
    private String field;

    public NoArgsExample() {
    }
  }
}
```

## 5. @NonNull

@NonNull 会判断是否为 null ，如果为 null ，则会抛出 java.lang.NullPointerException。

作用于字段，方法，参数，局部变量等。

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    this.name = person.getName();
  }
}
```

生成的代码：

```java
import lombok.NonNull;

public class NonNullExample extends Something {
  private String name;

  public NonNullExample(@NonNull Person person) {
    super("Hello");
    if (person == null) {
      throw new NullPointerException("person is marked @NonNull but is null");
    }
    this.name = person.getName();
  }
}
```

## 6. @Data

@Data = @Setter + @Getter + @RequiredArgsConstructor + @ToString + @EqualsAndHashCode。

```java
import lombok.AccessLevel;
import lombok.Setter;
import lombok.Data;
import lombok.ToString;

@Data
public class DataExample {
  private final String name;
  @Setter(AccessLevel.PACKAGE)
  private int age;
  private double score;
  private String[] tags;

  @ToString(includeFieldNames=true)
  @Data(staticConstructor="of")
  public static class Exercise<T> {
    private final String name;
    private final T value;
  }
}
```

生成的代码：

```java
import java.util.Arrays;

public class DataExample {
  private final String name;
  private int age;
  private double score;
  private String[] tags;

  public DataExample(String name) {
    this.name = name;
  }

  public String getName() {
    return this.name;
  }

  void setAge(int age) {
    this.age = age;
  }

  public int getAge() {
    return this.age;
  }

  public void setScore(double score) {
    this.score = score;
  }

  public double getScore() {
    return this.score;
  }

  public String[] getTags() {
    return this.tags;
  }

  public void setTags(String[] tags) {
    this.tags = tags;
  }

  @Override
  public String toString() {
    return "DataExample(" + this.getName() + ", " + this.getAge() + ", " + this.getScore() + ", " + Arrays.deepToString(this.getTags()) + ")";
  }

  protected boolean canEqual(Object other) {
    return other instanceof DataExample;
  }

  @Override
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof DataExample)) return false;
    DataExample other = (DataExample) o;
    if (!other.canEqual((Object)this)) return false;
    if (this.getName() == null ? other.getName() != null : !this.getName().equals(other.getName())) return false;
    if (this.getAge() != other.getAge()) return false;
    if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
    if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
    return true;
  }

  @Override
  public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final long temp1 = Double.doubleToLongBits(this.getScore());
    result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
    result = (result*PRIME) + this.getAge();
    result = (result*PRIME) + (int)(temp1 ^ (temp1 >>> 32));
    result = (result*PRIME) + Arrays.deepHashCode(this.getTags());
    return result;
  }

  public static class Exercise<T> {
    private final String name;
    private final T value;

    private Exercise(String name, T value) {
      this.name = name;
      this.value = value;
    }

    public static <T> Exercise<T> of(String name, T value) {
      return new Exercise<T>(name, value);
    }

    public String getName() {
      return this.name;
    }

    public T getValue() {
      return this.value;
    }

    @Override
    public String toString() {
      return "Exercise(name=" + this.getName() + ", value=" + this.getValue() + ")";
    }

    protected boolean canEqual(Object other) {
      return other instanceof Exercise;
    }

    @Override
    public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof Exercise)) return false;
      Exercise<?> other = (Exercise<?>) o;
      if (!other.canEqual((Object)this)) return false;
      if (this.getName() == null ? other.getValue() != null : !this.getName().equals(other.getName())) return false;
      if (this.getValue() == null ? other.getValue() != null : !this.getValue().equals(other.getValue())) return false;
      return true;
    }

    @Override
    public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      result = (result*PRIME) + (this.getName() == null ? 43 : this.getName().hashCode());
      result = (result*PRIME) + (this.getValue() == null ? 43 : this.getValue().hashCode());
      return result;
    }
  }
}
```

## 7. @Builder

@Builder使用构造器模式生成对象。作用于类、方法和构造器上。

调用方法类似于：

```java
Person.builder().name("Adam Savage").city("San Francisco").job("Mythbusters").job("Unchained Reaction").build();import lombok.Builder;
```

@Builder.Default

如果在构建期间从未设置某个字段/参数，则它始终为 0 / null / false。 如果您将@Builder 放在类（而不是方法或构造函数）上，则可以直接在字段上指定默认值，并使用@Builder.Default 注释字段。

@Singular

通过使用@Singular 注解注解其中一个参数（如果使用@Builder 对方法或构造函数进行注解）或字段（如果使用@Builder 对类进行注解），lombok 会将该构建器节点视为一个集合，并生成 2个adder方法而不是一个setter方法。 一种将单个元素添加到集合中，另一种将另一个集合的所有元素添加到集合中。

```java
import lombok.Singular;
import java.util.Set;

@Builder
public class BuilderExample {
  @Builder.Default
  private long created = System.currentTimeMillis();
  private String name;
  private int age;
  @Singular
  private Set<String> occupations;
}
```

生成的代码：

```java
import java.util.Set;

public class BuilderExample {
  private long created;
  private String name;
  private int age;
  private Set<String> occupations;

  BuilderExample(String name, int age, Set<String> occupations) {
    this.name = name;
    this.age = age;
    this.occupations = occupations;
  }

  private static long $default$created() {
    return System.currentTimeMillis();
  }

  public static BuilderExampleBuilder builder() {
    return new BuilderExampleBuilder();
  }

  public static class BuilderExampleBuilder {
    private long created;
    private boolean created$set;
    private String name;
    private int age;
    private java.util.ArrayList<String> occupations;

    BuilderExampleBuilder() {
    }

    public BuilderExampleBuilder created(long created) {
      this.created = created;
      this.created$set = true;
      return this;
    }

    public BuilderExampleBuilder name(String name) {
      this.name = name;
      return this;
    }

    public BuilderExampleBuilder age(int age) {
      this.age = age;
      return this;
    }

    public BuilderExampleBuilder occupation(String occupation) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }

      this.occupations.add(occupation);
      return this;
    }

    public BuilderExampleBuilder occupations(Collection<? extends String> occupations) {
      if (this.occupations == null) {
        this.occupations = new java.util.ArrayList<String>();
      }

      this.occupations.addAll(occupations);
      return this;
    }

    public BuilderExampleBuilder clearOccupations() {
      if (this.occupations != null) {
        this.occupations.clear();
      }

      return this;
    }

    public BuilderExample build() {
      // complicated switch statement to produce a compact properly sized immutable set omitted.
      Set<String> occupations = ...;
      return new BuilderExample(created$set ? created : BuilderExample.$default$created(), name, age, occupations);
    }

    @java.lang.Override
    public String toString() {
      return "BuilderExample.BuilderExampleBuilder(created = " + this.created + ", name = " + this.name + ", age = " + this.age + ", occupations = " + this.occupations + ")";
    }
  }
}
```

## 8. @Value

@Value类似于@Data，但是它生成的所有字段都是private和final的。

```java
import lombok.AccessLevel;
import lombok.experimental.NonFinal;
import lombok.experimental.Value;
import lombok.experimental.With;
import lombok.ToString;

@Value
public class ValueExample {
  String name;
  @With(AccessLevel.PACKAGE)
  @NonFinal
  int age;
  double score;
  protected String[] tags;

  @ToString(includeFieldNames=true)
  @Value(staticConstructor="of")
  public static class Exercise<T> {
    String name;
    T value;
  }
}
```

生成的代码：

```java
import java.util.Arrays;

public final class ValueExample {
  private final String name;
  private int age;
  private final double score;
  protected final String[] tags;

  @java.beans.ConstructorProperties({"name", "age", "score", "tags"})
  public ValueExample(String name, int age, double score, String[] tags) {
    this.name = name;
    this.age = age;
    this.score = score;
    this.tags = tags;
  }

  public String getName() {
    return this.name;
  }

  public int getAge() {
    return this.age;
  }

  public double getScore() {
    return this.score;
  }

  public String[] getTags() {
    return this.tags;
  }

  @java.lang.Override
  public boolean equals(Object o) {
    if (o == this) return true;
    if (!(o instanceof ValueExample)) return false;
    final ValueExample other = (ValueExample)o;
    final Object this$name = this.getName();
    final Object other$name = other.getName();
    if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
    if (this.getAge() != other.getAge()) return false;
    if (Double.compare(this.getScore(), other.getScore()) != 0) return false;
    if (!Arrays.deepEquals(this.getTags(), other.getTags())) return false;
    return true;
  }

  @java.lang.Override
  public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final Object $name = this.getName();
    result = result * PRIME + ($name == null ? 43 : $name.hashCode());
    result = result * PRIME + this.getAge();
    final long $score = Double.doubleToLongBits(this.getScore());
    result = result * PRIME + (int)($score >>> 32 ^ $score);
    result = result * PRIME + Arrays.deepHashCode(this.getTags());
    return result;
  }

  @java.lang.Override
  public String toString() {
    return "ValueExample(name=" + getName() + ", age=" + getAge() + ", score=" + getScore() + ", tags=" + Arrays.deepToString(getTags()) + ")";
  }

  ValueExample withAge(int age) {
    return this.age == age ? this : new ValueExample(name, age, score, tags);
  }

  public static final class Exercise<T> {
    private final String name;
    private final T value;

    private Exercise(String name, T value) {
      this.name = name;
      this.value = value;
    }

    public static <T> Exercise<T> of(String name, T value) {
      return new Exercise<T>(name, value);
    }

    public String getName() {
      return this.name;
    }

    public T getValue() {
      return this.value;
    }

    @java.lang.Override
    public boolean equals(Object o) {
      if (o == this) return true;
      if (!(o instanceof ValueExample.Exercise)) return false;
      final Exercise<?> other = (Exercise<?>)o;
      final Object this$name = this.getName();
      final Object other$name = other.getName();
      if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
      final Object this$value = this.getValue();
      final Object other$value = other.getValue();
      if (this$value == null ? other$value != null : !this$value.equals(other$value)) return false;
      return true;
    }

    @java.lang.Override
    public int hashCode() {
      final int PRIME = 59;
      int result = 1;
      final Object $name = this.getName();
      result = result * PRIME + ($name == null ? 43 : $name.hashCode());
      final Object $value = this.getValue();
      result = result * PRIME + ($value == null ? 43 : $value.hashCode());
      return result;
    }

    @java.lang.Override
    public String toString() {
      return "ValueExample.Exercise(name=" + getName() + ", value=" + getValue() + ")";
    }
  }
}
```

## 9. @Synchronized

作用于方法上，注解锁定了一个名为`$lock`的字段，该字段是私有的，如果不存在，将会自动创建。如果注解在静态方法上，则将锁定名为`$LOCK`的静态字段。

```java
import lombok.Synchronized;

public class SynchronizedExample {
  private final Object readLock = new Object();

  @Synchronized
  public static void hello() {
    System.out.println("world");
  }

  @Synchronized
  public int answerToLife() {
    return 42;
  }

  @Synchronized("readLock")
  public void foo() {
    System.out.println("bar");
  }
}
```

生成的代码：

```java
public class SynchronizedExample {
  private static final Object $LOCK = new Object[0];
  private final Object $lock = new Object[0];
  private final Object readLock = new Object();

  public static void hello() {
    synchronized($LOCK) {
      System.out.println("world");
    }
  }

  public int answerToLife() {
    synchronized($lock) {
      return 42;
    }
  }

  public void foo() {
    synchronized(readLock) {
      System.out.println("bar");
    }
  }
}
```

## 10. @Cleanup

作用于局部变量，自动清理资源。比如文件流关闭时调用Close()。

```java
import lombok.Cleanup;
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    @Cleanup
    InputStream in = new FileInputStream(args[0]);
    @Cleanup
    OutputStream out = new FileOutputStream(args[1]);
    byte[] b = new byte[10000];
    while (true) {
      int r = in.read(b);
      if (r == -1) break;
      out.write(b, 0, r);
    }
  }
}
```

生成的代码：

```java
import java.io.*;

public class CleanupExample {
  public static void main(String[] args) throws IOException {
    InputStream in = new FileInputStream(args[0]);
    try {
      OutputStream out = new FileOutputStream(args[1]);
      try {
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      } finally {
        if (out != null) {
          out.close();
        }
      }
    } finally {
      if (in != null) {
        in.close();
      }
    }
  }
}
```

## 11. @SneakyThrows

作用于方法和构造器，用于抛出指定的异常，无需手动捕获异常，然后抛出。

```java
import lombok.SneakyThrows;

public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }
  
  @SneakyThrows
  public void run() {
    throw new Throwable();
  }
}
```

生产的代码：

```java
import lombok.Lombok;

public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }
  
  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      throw Lombok.sneakyThrow(t);
    }
  }
}
```

## 12. @Log

作用于类上，用于生成日志，需要导入相关的依赖库。

- @CommonsLog
  private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog(LogExample.class);

- @Flogger
  private static final com.google.common.flogger.FluentLogger log = com.google.common.flogger.FluentLogger.forEnclosingClass();

- @JBossLog
  private static final org.jboss.logging.Logger log = org.jboss.logging.Logger.getLogger(LogExample.class);

- @Log
  private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());

- @Log4j
  private static final org.apache.log4j.Logger log = org.apache.log4j.Logger.getLogger(LogExample.class);

- @Log4j2
  private static final org.apache.logging.log4j.Logger log = org.apache.logging.log4j.LogManager.getLogger(LogExample.class);

- @Slf4j
  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExample.class);

- @XSlf4j
  private static final org.slf4j.ext.XLogger log = org.slf4j.ext.XLoggerFactory.getXLogger(LogExample.class);

- @CustomLog
  private static final com.foo.your.Logger log = com.foo.your.LoggerFactory.createYourLogger(LogExample.class);

```java
import lombok.extern.java.Log;
import lombok.extern.slf4j.Slf4j;

@Log
public class LogExample {
  
  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

@Slf4j
public class LogExampleOther {
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

@CommonsLog(topic="CounterLog")
public class LogExampleCategory {

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

生成的代码：

```java
public class LogExample {
  private static final java.util.logging.Logger log = java.util.logging.Logger.getLogger(LogExample.class.getName());
  
  public static void main(String... args) {
    log.severe("Something's wrong here");
  }
}

public class LogExampleOther {
  private static final org.slf4j.Logger log = org.slf4j.LoggerFactory.getLogger(LogExampleOther.class);
  
  public static void main(String... args) {
    log.error("Something else is wrong here");
  }
}

public class LogExampleCategory {
  private static final org.apache.commons.logging.Log log = org.apache.commons.logging.LogFactory.getLog("CounterLog");

  public static void main(String... args) {
    log.error("Calling the 'CounterLog' with a message");
  }
}
```

## 13. @Val

作用于局部变量声明的类型。声明的局部变量类型为final。

```java
import java.util.ArrayList;
import java.util.HashMap;
import lombok.val;

public class ValExample {
  public String example() {
    val example = new ArrayList<String>();
    example.add("Hello, World!");
    val foo = example.get(0);
    return foo.toLowerCase();
  }
  
  public void example2() {
    val map = new HashMap<Integer, String>();
    map.put(0, "zero");
    map.put(5, "five");
    for (val entry : map.entrySet()) {
      System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
    }
  }
}
```

生成的代码：

```java
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Map;

public class ValExample {
  public String example() {
    final ArrayList<String> example = new ArrayList<String>();
    example.add("Hello, World!");
    final String foo = example.get(0);
    return foo.toLowerCase();
  }
  
  public void example2() {
    final HashMap<Integer, String> map = new HashMap<Integer, String>();
    map.put(0, "zero");
    map.put(5, "five");
    for (final Map.Entry<Integer, String> entry : map.entrySet()) {
      System.out.printf("%d: %s\n", entry.getKey(), entry.getValue());
    }
  }
}
```

## 14. @Var

var与val类似，只是局部变量不是final。在JDK 9中已经集成了该语法。


