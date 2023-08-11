# [try-finally 보다는 try-with-resources를 사용하라]
## 개요
    자바 라이브러리에서 close 메서드를 호출해 직접 닫아줘야하는 자원들이 있습니다. 
    ex)InputStream, OutputStream, java.sql.Connection 등등...
    이러한 자원들을 닫아주는 것은 클라이언트가 놓치기 쉬워 예측할 수 없는 성능문제로 이어지기도 합니다.
    상당수가 안전망으로 finalizer를 활용하고는 있지만 아이템8에서 이야기하듯 사용하는 것이 좋지 않습니다.
    자바에는 close를 호출해 직접 닫아줘야 하는 자원이 있습니다.

### close 를 못닫는 예시
```JAVA
public class Main {
    public static void main(String[] args) throws IOException {
        try{
            inputString();
        } catch(Exception e){
            System.out.println(e.getMessage());
            throw new IOException();
        }
    }

    public static String inputString() throws IOException {
        BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
        String result = br.readLine();              //만약 이곳에서 에러가 난다면...
        br.close();
        return result;
    }
}
```

### 자원이 한개인 상황
```JAVA
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```
- 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try-finally 가 쓰인다
- 나쁘지 않지만, 자원을 하나 더 사용한다면 어떨까?

### 자원이 두개인 상황
```JAVA
public class Main {
    private final static Integer BUFFER_SIZE = 30;
    
    public static void main(String[] args) throws IOException {
        String src = "Jeong";
        String dst = "Seok";
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dst);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
            } finally {
                log.error(error_test);      //이곳에서 에러가 발생한다면?
                out.close();                //여기까지 도달하지 못한다.
            }
        } finally {
            in.close();
        }

    }
}
```
- 위 코드 처럼 여러개가 된다면 굉장히 지저분해지게 될 수 있다.
- 하지만 문제는 try 블록을 실행하던 도중 기기에 문제가 생긴다면 정상적으로 실행되지 못하고 예외를 던지게 되고, 같은 이유로 finally 블록의 close 메서드도 예외를 던지게 된다.

## 해결책

    자바7 부터는 위 같은 문제에 대한 해결책으로 try-with-resources 가 도입되었다.
    try-with-resources를 사용하기 위해서는 AutoCloseable 인터페이스를 구현해야 한다.
### AutoCloseable 인터페이스 구현
```JAVA
public interface AutoCloseable {
    void close() throws Exception;
}
```

### try-with-resources방식
```JAVA
static String firstLineOfFile(String path) throws IOException {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
        return br.readLine();
    }
}
```
try-finally방식은 br.readLine()에 대한 스택 추적 내역은 남지 않고 br.close()의 스택 추적 내역만 남는다.

## 실제 예시

### try-with-resources방식
```JAVA
public class SimpleResource implements AutoCloseable {

    public void use() {
        System.out.println("Using the resource!");
    }

    @Override
    public void close() {
        System.out.println("Resource has been closed!");
    }
}
public class Main {
    public static void main(String[] args) {
        try (SimpleResource resource = new SimpleResource()) {
            resource.use();
        } // 여기에서 SimpleResource의 close() 메서드가 자동으로 호출됩니다.
    }
}
public interface AutoCloseable {
    void close() throws Exception;
}
```
try-with-resources 문을 사용하면, 
try 블록이 종료될 때 AutoCloseable 인터페이스 (또는 그 하위 인터페이스인 Closeable)의 close() 메서드가 
자동으로 호출되는데 이는 Java 언어 및 JVM에 의해 제공되는 기능입니다.

### try-with-resources와 catch
```JAVA
public static String inputString() {
    try (BufferedReader br = new BufferedReader(new InputStream(System.in))) {
        return br.readLine();
    } catch (IOException e) {
        return "IOException 발생";
    }
}
```
try-with-resources 구조 역시 기존 try-finally 처럼 catch를 병용해서 사용할 수 있다.

## 결론
close를 통해 회수해야 하는 자원을 다룰 때는 try-finally를 사용하는 대신 반드시 try-with-resources를 사용하자. 
예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다.


### 참고
BufferedReader를 try-with-resources 문장으로 사용하면,
BufferedReader는 Closeable 및 AutoCloseable 인터페이스를 구현하기 때문에 close() 메서드가 있어
해당 리소스는 try 블록이 종료되었을 때 자동으로 close()가 호출됩니다.

```JAVA
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    String line;
    while ((line = br.readLine()) != null) {
        System.out.println(line);
    }
}
// 여기에서 BufferedReader의 close() 메서드가 자동으로 호출됩니다.
```

```JAVA
public void close() throws IOException {
    synchronized (lock) {
        if (in == null)
            return;
        try {
            in.close();
        } finally {
            in = null;
            cb = null;
        }
    }
}
```