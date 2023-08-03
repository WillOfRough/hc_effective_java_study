# [private 생성자나 열거 타입으로 싱글턴임을 보증하라]

## 싱글턴이란?

    싱글턴 디자인 패턴은 소프트웨어 설계에서 사용되는 디자인 패턴 중 하나로, 
    클래스의 인스턴스를 하나만 생성하도록 설계하는 패턴입니다. 
    이는 공유 리소스를 사용할 때 유용하며, 주로 로그, 데이터베이스 연결, 파일 시스템 등에서 사용됩니다. 
    싱글턴 패턴을 사용하면, 한 클래스의 인스턴스가 시스템 내에서 한 번만 생성되고, 
    그 인스턴스를 여러 곳에서 공유하여 사용할 수 있습니다.

### 특징
    private 생성자는 public static final 필드인 CONNECT_INSTANCE 를 초기화 할때
    딱 한번만 호출되고 public 이나 protected 생성자가 없으므로 인스턴스가 전체 시스템에서 
    하나뿐임을 보장한다.

## public static final 방식의 싱글턴
```JAVA
public class DatabaseInfo {
    String ip;
    int port;
    public static final DatabaseInfo CONNECT_INSTANCE = new DatabaseInfo("172.3.30.32",5432);

    private DatabaseInfo(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}

public class Main {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {

        //DatabaseInfo 의 생성자는 private 이기 때문에 new 를 이용해서 새로 생성할 수 없으며 오직 메모리에 저장되어있는 인스턴스를 가져올 수 밖에 없음
        DatabaseInfo devDatabase = DatabaseInfo.CONNECT_INSTANCE;

        System.out.println(devDatabase.ip);     //172.3.30.32
        System.out.println(devDatabase.port);   //5432

        Constructor[] constructors = DatabaseInfo.class.getDeclaredConstructors();
        for (Constructor constructor : constructors) {
            constructor.setAccessible(true);
            DatabaseInfo instance2 = (DatabaseInfo) constructor.newInstance("255.255.255.255",8081);
            System.out.println(instance2.ip);       //255.255.255.255
            System.out.println(instance2.port);     //8081
            break;
        }
    }
}
```
### 장점
    1. 해당 클래스가 싱글턴임이 API 에 명백하게 드러남
    2. 간결함

## 정적 팩터리 방식의 싱글턴
```JAVA
public class DatabaseInfo {
    String ip;
    int port;
    //static 을 이용해 메소드 영역에 위치하게 만들고 final 을 이용해서 변경할 수 없도록 설정함
    private static final DatabaseInfo CONNECT_INSTANCE = new DatabaseInfo("172.3.30.32",5432);

    private DatabaseInfo(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }

    public static DatabaseInfo getInstance() {
        return CONNECT_INSTANCE;
    }
}
```
### 장점
    1. API 를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 점
    2. 원한다면 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다
    3. 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다는점
        ex. DatabaseInfo::getInstance 를 Supplier<DatabaseInfo> 로 사용

### API 를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다는 예시
```JAVA
public class DatabaseInfo {
    String ip;
    int port;
    private static final DatabaseInfo CONNECT_INSTANCE = new DatabaseInfo("172.3.30.32",5432);

    private DatabaseInfo(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
    public static DatabaseInfo getInstance() {
        return CONNECT_INSTANCE;
    }
}
```

### 정적 팩터리를 제네릭 싱글턴 팩터리로 만들 수 있다는 예시
```JAVA
public class SingletonFactory {
    private static final Map<String, Object> instances = new HashMap<>();

    private SingletonFactory() {
        // Prevent instantiation
    }

    public static synchronized <T> T getInstance(String key, Supplier<T> supplier) {
        if (!instances.containsKey(key)) {
            instances.put(key, supplier.get());
        }
        return (T) instances.get(key);
    }
}

public class DatabaseInfo {
    private String ip;
    private int port;

    public DatabaseInfo(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
    
    //이곳에서 재네릭을 이용한 싱글턴 인스턴스를 동적으로 생성
    public static DatabaseInfo getInstance(String ip, int port) {
        String key = ip + ":" + port;
        return SingletonFactory.getInstance(key, () -> new DatabaseInfo(ip, port));
    }
}

public class Main {
    public static void main(String[] args) {
        DatabaseInfo dbInfo1 = DatabaseInfo.getInstance("172.3.30.32", 5432);
        DatabaseInfo dbInfo2 = DatabaseInfo.getInstance("192.168.1.1", 3306);
    }
}
```

### 특징
    둘 중 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Serializable 을 구현한다고 선언
    하는것만으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve
    메서드를 제공해야한다. 이렇게 하지 않으면 직렬화된 인스턴스를 역직렬화할 때마다 새로운 인스턴스가
    만들어진다.

* 클래스 직렬화 : 싱글턴 클래스의 인스턴스가 여러 JVM 간에 전송되어야 하거나, 장기간에 걸쳐 상태를 유지해야 하는 경우에 직렬화를 사용하게 됩니다.


### 싱글턴임을 보장해주는 readResolve 메서드
```JAVA
//Serializable 인터페이스는 Java에서 객체의 직렬화를 가능하게 하는 표시(marker) 인터페이스
public class DatabaseInfo implements Serializable {
    String ip;
    int port;
    public static final DatabaseInfo CONNECT_INSTANCE = new DatabaseInfo("172.3.30.32",5432);

    private DatabaseInfo(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
    
    //역직렬화시 readResolve() 메소드가 있다면 해당 메소드를 이용해서 역직렬화함
    private Object readResolve() throws ObjectStreamException {
        return CONNECT_INSTANCE;
    }
}
```

* 역직렬화 과정에서는 직렬화 데이터를 기반으로 새로운 인스턴스가 생성됩니다. 이는 새로운 인스턴스를 생성하는 것이지만, 생성자는 호출되지 않습니다. 이러한 특성 때문에 역직렬화는 싱글턴 객체에 문제를 일으킬 수 있습니다.
### 역직렬화 예시
```JAVA
public class Main {
public static void main(String[] args) {

        //싱글턴 클래스 직렬화
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.ser"))) {
            out.writeObject(DatabaseInfo.getInstance());
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
        DatabaseInfo databaseInfo1;

        //싱글턴 클래스 역직렬화
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.ser"))) {
            databaseInfo1 = (DatabaseInfo) in.readObject();
        } catch (IOException e) {
            throw new RuntimeException(e);
        } catch (ClassNotFoundException e) {
            throw new RuntimeException(e);
        }

        DatabaseInfo databaseInfo2 = DatabaseInfo.getInstance();
        //만약에 readResolve() 메소드가 없었더라면 아래 값이 false 가 되어있을것이다.
        System.out.println(databaseInfo1 == databaseInfo2);
    }
}
```
### 열거 타입 방식의 싱글턴 - 바람직한 방법

```JAVA
public enum DatabaseInfoEnum {
    CONNECT_INSTANCE("172.3.30.32",5432);

    String ip;
    int port;

    DatabaseInfoEnum(String ip, int port) {
        this.ip = ip;
        this.port = port;
    }
}

public class Main {
    public static void main(String[] args) {
        //항상 같은 인스턴스를 참조함
        String ip = DatabaseInfoEnum.CONNECT_INSTANCE.ip;
        int port = DatabaseInfoEnum.CONNECT_INSTANCE.port;
    }
}
```

### 특징
    1. public 필드 방식과 비슷하지만 더 간결함
    2. 추가 노력 없이 직렬화 할 수 있음
    3. 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아줌
    4. 조금 부자연스러워 보일 수 있으나 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

* 단 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.

### TMI : Enum 특징
1. 값의 제한: enum은 주어진 값들 중 하나로 제한됩니다. 이는 다른 클래스와는 대조적으로, 일반 클래스는 어떤 종류의 상태도 가질 수 있습니다.
2. 인스턴스 생성의 제한: enum은 그들의 값들을 정의하는 순간에 인스턴스가 생성됩니다. new 키워드를 사용해 추가적인 인스턴스를 만들 수 없습니다. 반면에 일반 클래스는 new 키워드를 사용해 언제든지 새로운 인스턴스를 생성할 수 있습니다.
3. 싱글턴의 기능: 각 enum 값은 자연스럽게 싱글턴입니다. 같은 enum 타입에서 동일한 값은 항상 같은 인스턴스를 참조합니다.
4. 메서드와 필드의 추가: enum은 메서드와 필드를 가질 수 있으며, 인터페이스를 구현할 수도 있습니다. 이를 통해 enum 값이 각자의 행동을 가지도록 할 수 있습니다.
5. 스위치 문에서의 사용: enum은 switch 문에서 사용될 수 있습니다.
6. 자동 toString 메서드: enum은 각 값의 이름을 반환하는 toString 메서드를 자동으로 제공합니다. 이는 오버라이드할 수 있습니다.
7. 직렬화의 안전성: enum은 직렬화와 역직렬화가 안전하게 이루어집니다. 직렬화와 역직렬화 과정에서 새로운 인스턴스가 생성되는 일이 없습니다.