# [public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라]

## public 클래스에서는 public 필드 대신 접근자 메소드를 제공하라.
- 다음 클래스는 데이터 필드에 직접 접근할 수 있으니 캡슐화의 이점을 제공하지 못한다.
- 또한 API를 수정하지 않고는 내부 표현을 바꿀 수 없고, 불변식을 보장할 수 없으며, 외부에서 필드에 접근할 때 부수적인 작업을 수행할 수도 없다.

```JAVA
public class Point {
    public double x; 
    public double y;

    public static void main(String[] args) {
        Point point = new Point();
        point.x = 1;
        point.y = 2;
    }
}
```

- 따라서 다음과 같이 필드들을 모두 private으로 바꾸고 public 접근자를 추가한다.
- 패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공함으로써 클래스 내부 표현 방식을 언제든 바꿀 수 있는 유연성을 얻을 수 있다.

```JAVA
@Getter
public class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public void setX(double x){
        if(this.x <= x){
            this.x = x;
        }
    }
    public void setY(double y){
        if(this.y <= y){
            this.y = y;
        }
    }

    public static void main(String[] args) {
        Point point = new Point(1, 2);
        System.out.println(point.getX); // 1
        System.out.println(point.getY); // 2
    }
}
```


## 접근자 대신 다른 방식을 고려할 수도 있다.

- 하지만 package-private 클래스 혹은 private 중첩 클래스라면 데이터 필드를 노출한다 해도 문제가 없다.
- 이 방식은 클래스 선언 면에서나 이를 사용하는 클라이언트의 코드 면에서나 접근자 방식보다 훨씬 깔끔하다.
- 클라이언트 코드가 이 클래스 내부 표현에 묶이기는 하나, 클라이언트도 어차피 이 클래스를 포함하는 패키지 안에서만 동작하는 코드일 뿐이다. 따라서 패키지 바깥 코드는 전혀 손대지 않고도 데이터 표현 방식을 바꿀 수 있다.

```JAVA
// package-private 클래스에서 데이터 필드 노출. 물론 이렇게 해서는 안된다.
class Point {
    public double x;
    public double y;
    ...
}
```

    package-private를 사용하는 경우에는 패키지를 소유하고 있는 사람들만 사용하기 때문에
    side effect가 제한적이고
    데이터필드에 직접 접근하더라도 public으로 써도 문제는 없다.
    하지만, 그렇다하더라도 메서드로 접근을 해야 개발단계에서 점진적인 변화가 가능하고 코드 변경시 번거로움이 없다.


```JAVA
// private 중첩 클래스에서 데이터 필드 노출. 오직 클래스 내부에서만 데이터 필드에 접근할 수 있다.
public class PointHolder {
    private class Point {
        public double x;
        public double y;
    }
    ...
}
```

## public 클래스의 필드가 불변이라도 public으로 제공하는 것은 좋지 않다.
- public 클래스의 필드가 불변이라면 직접 노출할 때의 단점이 조금은 줄어들지만, 결코 좋은 생각이 아니다.
- API를 변경하지 않고는 표현 방식을 바꿀 수 없고, 필드를 읽을 때 부수 작업을 수행할 수 없다는 단점은 여전하다.
- 단, 불변식은 보장할 수 있게 된다. 예컨대 다음 클래스는 각 인스턴스가 유효한 시간을 표현함을 보장한다.


```JAVA
public final class Time {
    private static final int HOURS_PER_DAY = 24;
    private static final int MINUTES_PER_HOUR = 60;

    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= HOURS_PER_DAY)
            throw new IllegalArgumentException("시간: " + hour);

        if (minute < 0 || minute >= MINUTES_PER_HOUR)
            throw new IllegalArgumentException("분: " + minute);

        this.hour = hour;
        this.minute = minute;
    }
}
    ...

```