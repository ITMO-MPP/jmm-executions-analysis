# Анализ возможных исполнений в модели памяти Java

В рамках данного задания вы проанализируете возможные исполнения нескольких программ с точки зрения модели памяти Java. Перед выполнением будет полезно просмотреть следующие доклады:

+ https://www.youtube.com/watch?v=iB2N8aqwtxc - Прагматика Java Memory Model, Алексей Шипилёв
+ https://www.youtube.com/watch?v=C6b_dFtujKo - Близкие Контакты JMM-степени, Алексей Шипилёв
+ https://2019.hydraconf.com/2019/talks/143pbdxfvijthb8rg3qk6e/ - Weak memory concurrency in C/C++11, Ori Lahav

## Задание

Вася Пупкин разбирается с моделью памяти Java и написал некоторое количество примеров кода. Помогите ему разобраться в том, что может произойти при их запуске. Предлагается ответить на следующие вопросы для каждого примера:

1. Какие значения может напечатать второй поток на экран? Нарисуйте все возможные в модели памяти Java графы исполнения. Напишите короткие комментарии для каждого из возможных исполнений. В качестве примера смотрите слайды с лекции.
2. Опишите, как можно исправить данный код. Докажите, что ваше решение является корректным. 

Еще раз: обязательно нарисуйте все возможные исполнения, которые могут произойти, это самый продуктивный способ понять модели памяти и не запутаться!

### 1. Data race access
В процессе изучения модели памяти, Вася Пупкин написал следующий код и хочет понять, к каким результатам он может привести. Помогите ему разобраться.

```java
class A {
    String s = "abc";
}

A a = null;

Thread 1:
=========
a = new A();


Thread 2:
=========
A ta = a;
if (ta != null)
    print(ta.s)
```

### 2. Volatile array
Василий продолжает изучения и теперь решил усложнить пример и начал использовать массивы. Что может получиться теперь?

```java
class A {
    String s = "abc";
}


volatile A[] arr = new A[1];

Thread 1:
=========
arr[0] = new A();


Thread 2:
=========
A ta = arr[0];
if (ta != null)
    print(ta.s)
```


### 3. Double-checked locking
Неугомонный Вася. Теперь он прочитал про [Double-checked locking](https://en.wikipedia.org/wiki/Double-checked_locking) идиому и решил использовать её у себя в проекте. Однако, во время запуска своего кода из разных потоков он получил неожиданные результаты и решил разобраться в корректности написанного кода. 

```java
class A {
    private static A instance;
    
    private long x = -1;
    
    private A() {}
    
    incX() { x++; }
    getX() { return x; }
    
    static A getInstance() {
        A res = instance;
        if (res == null) {
            synchronized(A.class) {
                res = instance;
                if (res == null) {
                    instance = res = new A();
                }
            }
        }
        return res;
    }
}


Thread 1:
=========
A a1 = A.getInstance();
print(a1.incX());


Thread 2:
=========
A a2 = A.getInstance();
print(a2.getX());
```

### Assignment in constructor

```java
class A {
    volatile int f;
    
    A() {
        f = 42;
        p = this;
    }
}

volatile A p;
A q;


Thread 1:
=========
A a = new A();
q = a;
p = a;


Thread 2:
=========
print(p.f)
print(q.f)
```

А что будет, если строчки во втором потоке поменять местами? А если поменять строчки в конструкторе `A()`?
