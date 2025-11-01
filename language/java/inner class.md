
- The association between a nonstatic member class instance and its enclosing instance is established when the member class instance is created and cannot be modified thereafter

non-static member class의 인스턴스는 생성될 때 외부 클래스 인스턴스와의 연결(association)이 만들어짐

 참조 관계는 생성 시점에 고정되고, 이후에 다른 외부 인스턴스로 바꿀 수 없습니다.
    
```java
class Outer {
    class Inner {  
        void hello() {
            System.out.println("Hello from inner, outer hash = " + Outer.this.hashCode());
        }
    }
}

Outer a = new Outer();
Outer b = new Outer();

Outer.Inner inner = a.new Inner();  // a와 연결됨
// inner = b.new Inner();  ❌ 기존 inner의 연결을 b로 바꿀 수 없음
```

---

Normally, the association is established automatically by invoking a nonstatic member class constructor from within an instance method of the enclosing class.

일반적으로 이 연결은 외부 클래스의 인스턴스 메서드 내에서 내부 클래스 생성자를 호출함으로써 자동으로 설정된다.

외부 클래스 안에서 내부 클래스를 생성하면 컴파일러가 자동으로 `this`를 넘겨줌


```java
class Outer {
    class Inner { }

    void createInner() {
        Inner i = new Inner(); // 컴파일러가 자동으로 this를 넘김: new Outer.Inner(this)
    }
}
```

---

"It is possible, though rare, to establish the association manually using the expression enclosingInstance.new MemberClass(args)."

```java
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();  // 👈 outer와 연결해서 생성
```

> 드물긴 하지만, `enclosingInstance.new MemberClass(args)` 문법을 사용하여 **직접 수동으로 연결을 설정**하는 것도 가능하다.

    

java

CopyEdit

`Outer outer = new Outer(); Outer.Inner inner = outer.new Inner();  // 👈 outer와 연결해서 생성`

- 위처럼 `.new` 구문을 써야 내부 클래스 생성이 가능합니다.
     

---

### 📌 4. **"As you would expect, the association takes up space in the nonstatic member class instance and adds time to its construction."**

#### 🔍 해석

> 예상할 수 있듯이, 이 연결(association)은 내부 클래스 객체 안에 메모리를 차지하며, 생성 시간도 더 든다.

#### 💡 설명

- 내부 클래스는 내부적으로 **외부 인스턴스를 참조하는 필드**를 갖습니다. (컴파일 시 `Outer this$0;` 같은 필드가 생깁니다.)
    
- 따라서:
    
    - 메모리: 내부 객체마다 외부 객체 포인터 하나 추가됨
        
    - 성능: 생성 시 외부 참조를 설정해야 하므로 약간의 오버헤드 발생
        


| 개념           | 설명                                         |
| ------------ | ------------------------------------------ |
| 내부 클래스 생성 시  | 외부 클래스 인스턴스를 반드시 참조해야 함                    |
| 이 연결은 변경 가능? | ❌ 생성 시점에 고정됨                               |
| 일반적인 생성 방식   | 외부 클래스의 인스턴스 메서드 내에서 `new Inner()`         |
| 수동 생성 방식     | 외부에서 `outer.new Inner()`                   |
| 성능 영향        | 메모리: 외부 인스턴스 참조 필드 포함  <br>시간: 연결 설정 비용 발생 |


--------

```java
// Outer.java
public class Outer {
    private String msg = "hello";

    // static nested class
    public static class StaticNested {
        public void print() {
            System.out.println("StaticNested");
        }
    }

    // non-static inner class
    public class Inner {
        public void print() {
            System.out.println("Inner: " + msg);
        }
    }
}
```

```
◎ javac Outer.java
◎ ls | grep Outer2

// output
Outer2.class
Outer2.java
Outer2$Inner.class
Outer2$StaticNested.class
```


`javap -c Outer2$Inner` 실행 시

```java
Compiled from "Outer2.java"
public class Outer2 {
  public Outer2();
    Code:
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #7                  // String hello
         7: putfield      #9                  // Field msg:Ljava/lang/String;
        10: return

  public static void main(java.lang.String[]);
    Code:
         0: new           #15                 // class Outer2$StaticNested
         3: dup
         4: invokespecial #17                 // Method Outer2$StaticNested."<init>":()V
         7: invokevirtual #18                 // Method Outer2$StaticNested.print:()V
        10: new           #21                 // class Outer2$Inner
        13: dup
        14: new           #10                 // class Outer2
        17: dup
        18: invokespecial #23                 // Method "<init>":()V
        21: invokespecial #24                 // Method Outer2$Inner."<init>":(LOuter2;)V
        24: invokevirtual #27                 // Method Outer2$Inner.print:()V
        27: return
}
```

static inner class 생성은 어떻게?



------