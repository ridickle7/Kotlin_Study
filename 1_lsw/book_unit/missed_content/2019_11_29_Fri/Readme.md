# [Properties and Fields](https://kotlinlang.org/docs/reference/properties.html)

## 프로퍼티 선언
코틀린 클래스 내의 프로퍼티는 수정 가능한 var 키워드 또는 읽기 전용인 val 키워드로 선언할 수 있다.

```
class Address {
    var name: String = "Holmes, Sherlock"
    var street: String = "Baker"
    var city: String = "London"
    var state: String? = null
    var zip: String = "123456"
}
```
프로퍼티를 사용하기 위해 단순하게 이름으로 참조가 가능합니다.

```
fun copyAddress(address: Address): Address {
    val result = Address() // 코틀린에서는 new 키워드를 사용하지 않습니다.
    result.name = address.name // 접근자가 호출됨 (이름으로 호출)
    result.street = address.street
    // ...
    return result
}
```

여기에서 [접근자](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.reflect/-k-property/-accessor/index.html)는 **getter setter** 와 같은 것을 말합니다.  

## Getters and Setters
프로퍼티를 선언하는 전체 구문은 아래와 같습니다.

```
var <propertyName>[: <PropertyType>] [= <property_initializer>]
    [<getter>]
    [<setter>]
```

초기화 변수 및 getter, setter는 있어도 되고 없어도 됩니다.
초기화 변수를 통해 타입 추론이 가능한 경우, 프로퍼티 타입은 적어도 되고 안 적어도 됩니다. 
(또는 getter 리턴 타입으로부터 추론이 가능한 경우 (아래 example 참고))

### Examples:

```
var allByDefault: Int?  // error: 기본 getter, setter 에 암시할 수 있는 명시적 초기화 값 필요 (ex> null or int value)
var initialized = 1     // Int형태로 타입 추론, 기본 getter, setter를 가짐
```

### 읽기 전용 프로퍼티 선언(val)은, 수정 가능한 변수(var)와 두가지 차이점이 있습니다.

1. var 대신 val로 시작하며 setter을 허용하지 않는다.

```
val simple: Int?      // Int형 타입을 가지며, 기본 getter을 가짐, 생성자 부분에서 초기화 시켜주어야 함.
val inferredType = 1  // Int 형 타입을 가지며 기본 getter을 가짐 
```

2. 우리는 커스텀 접근자를 프로퍼티에서 선언할 수 있습니다.
만약 우리가 커스텀 getter을 선언한다면, 우리가 프로퍼티에 접근할 때마다 불려질 것입니다.
(커스텀 getter는 우리로 하여금 계산된 프로퍼티를 구현할 수 있도록 합니다.)

#### 여기 커스텀 getter의 예제는 아래와 같습니다 : 

```
val isEmpty: Boolean
    get() = this.size == 0
```
만약 우리가 커스텀 getter을 선언한다면, 이는 우리가 프로퍼티에 할당한 값대로 항상 불려질 것입니다.

#### 커스텀 setter의 예제는 아래와 같습니다 : 

```
var stringRepresentation: String
    get() = this.toString()
    set(value) {
        setDataFromString(value) // parses the string and assigns values to other properties
    }
```
일반적으로 setter 매개 변수의 이름은 value이지만, 원하는 경우 다른 이름을 사용할 수 있습니다.  
  
### Kotlin 1.1 버전 이후, 만약 getter에서 타입 추론이 가능한 경우, 프로퍼티의 타입을 생략할 수 있습니다.

```
val isEmpty get() = this.size == 0  // Boolean 타입
```

### 만약 접근자의 가시성을 조정하거나, 주석을 달아야하지만 기본 값을 바꿀 필요가 없는 경우, 몸체의 선언 없이 접근자를 선언할 수 있다. (아래와 같이)
```
var setterVisibility: String = "abc"
    private set // setter는 private하고 기본 값을 가지고 있습니다.
​
var setterWithAnnotation: Any? = null
    @Inject set // Inject와 함께 setter에 주석을 처리합니다?
```

## Backing Fields
필드는 코틀린 클래스에 직접적으로 사용될 수 없습니다.  
코틀린은 필드라는 식별자를 통해 접근할 수 있는 Automatic Backing field 를 제공합니다.
Backing field 는 필드 식별자를 활용해 접근자에서만 참조될 수 있습니다.

```
var counter = 0 // Note: 초기 값을 backing field를 직접 할당합니다.
    set(value) {
        if (value >= 0) field = value
    }
```
필드 식별자는 프로퍼티의 식별자 내에서만 사용이 가능합니다.

### Backing Field는 만약 최소 하나의 접근자에서 프로퍼티로 만들어질 것입니다.

1. 프로퍼티 중 최소 1개 이상의 접근자의 기본 구현 방식을 사용하거나,
2. 커스텀 접근자가 필드 식별자를 참조하는 경우  

**Backing Field 가 생성됩니다. (both 1 and 2)**  

```
val isEmpty: Boolean
    get() = this.size == 0
```

위의 경우는 Backing Field가 생성되지 않습니다.
(2번 위배)

## Backing Properties
만약 암시적 Backing Field 문법이 맞지 않는 경우, Backing Property를 가질 수 있다.

```
private var _table: Map<String, Int>? = null
public val table: Map<String, Int>
    get() {
        if (_table == null) {
            _table = HashMap() // Type parameters are inferred
        }
        return _table ?: throw AssertionError("Set to null by another thread")
    }
```
JVM 위에서: 일반 getter와 setter 접근자를 통해 private property들에 접근하는 경우  
함수 호출 오버 헤드가 발생하지 않도록 최적화되었습니다.  
**-> 왠지 여기에서 오늘 내가 제대로 답변하지 못했던 내용에 대한 실마리가 있는 것 같기도 하고...?**

## Compile-Time Constants
const 를 활용하여 Compile-Time Constants로 표현할 수 있습니다.
이 property 들은 아래의 요구사항들을 이행할 필요가 있습니다.

상위 레벨에 있거나, object의 멤버이거나, companion object 이거나
String 또는 primitive 타입 변수로 초기화 된 경우  
custom Getter 가 없는 경우 

이 Property들은 아래처럼 활용될 수 있습니다.

```
const val SUBSYSTEM_DEPRECATED: String = "This subsystem is deprecated"
@Deprecated(SUBSYSTEM_DEPRECATED) fun foo() { ... }
```

## Late-Initialized Properties and Variables

일반적으로 porperty들은 not null 데이터로 선언되며 반드시 초기화를 시켜주어야 합니다.
그러나 이는 불편할 때가 있습니다. 예를 들어 property들이 의존성 주입을 통해 초기화 될 경우 또는 unit test에서 SetUp 함수를 사용할 때 등의 경우가 있습니다.
이런 경우 생성자에 null이 아닌 초기값을 제공해줄 수 없지만, 클래스 본문 내에서 Property 를 참조 할 때 null 검사를 피하고 싶습니다.

이를 핸들링하기 위해 우리는 lateinit 을 활용할 수 있습니다.
```
public class MyTest {
    lateinit var subject: TestSubject

    @SetUp fun setup() {
        subject = TestSubject()
    }

    @Test fun test() {
        subject.method()  // dereference directly
    }
}
```

lateinit 제약사항
1. 클래스 본문 내 var property에서 사용됨
2. (Kotlin 1.2 버전 이후) 상위 레벨 프로퍼티와 로컬 변수에 사용 가능
2. 기본 생성자에서 선언된 property는 불가능
3. 오직 커스텀 getter와 setter을 가지지 않는 property 만 가능

초기화 되지 않은 lateinit property에 접근하면 exception이 발생합니다.

### lateinit 변수가 초기화 됬는지 확인하는 법 (Kotlin 1.2 버전 이후)

"프로퍼티.isInitialized" 를 통해 확인 가능  
```
if (foo::bar.isInitialized) {
    println(foo.bar)
}
```
이 확인은 오직 property에 접근이 가능한 경우에 대해서만 가능합니다.
즉, 동일한 타입 내에서의 선언 또는 outer 타입 중 하나 내에서의 선언 또는 동일한 파일 안에서의 상위 레벨에서 접근 가능한 경우에 대해서만 가능합니다.

============================================ 여기서부터 다시 시작 ============================================

Delegated Properties
The most common kind of properties simply reads from (and maybe writes to) a backing field. On the other hand, with custom getters and setters one can implement any behaviour of a property. Somewhere in between, there are certain common patterns of how a property may work. A few examples: lazy values, reading from a map by a given key, accessing a database, notifying listener on access, etc.

Such common behaviours can be implemented as libraries using delegated properties.
