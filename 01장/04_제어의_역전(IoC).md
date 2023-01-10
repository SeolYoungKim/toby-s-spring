# 1.4.1 오브젝트 팩토리

## 팩토리

팩토리는 객체의 생성 방법을 결정하고 그렇게 만들어진 오브젝트를 반환한다.
예제의 `UserDao`는 아래와 같이 팩토리를 만들어 볼 수 있다.
```java
public class DaoFactory {
    public UserDao userDao() {
        // UserDao의 ConnectionMaker 구현체를 결정한다.
        ConnectionMaker cm = new DConnectionMaker();
        UserDao userDao = new UserDao(cm);
        return userDao;
    }
}
```

덕분에 테스트 코드에선 `UserDao`를 어떻게 만드는지, 초기화 되는지 신경쓰지 않아도 된다.

테스트 코드에선 다음과 같이 사용한다.
```java
public class UserDaoTest {
    public static void main(String[] args) throws ~~ {
        UserDao dao = new DaoFactory().userDao();
        ...
    }
}
```

## 설계도로서의 팩토리

UserDao와 ConnectionMaker는 애플리케이션의 핵심적인 데이터와 로직을 담당하는 컴포넌트,  
DaoFactory는 이런 컴포넌트들의 관계를 정의하는 설계도 역할을 담당한다고 볼 수 있다.

![image](https://user-images.githubusercontent.com/7973448/211448421-e5b640ed-c120-4b54-8d47-7a422d6e5e35.png)

이제 이 애플리케이션을 변경하거나 확장하려 할때 설계도를 수정해주면 된다.

# 1.4.2 오브젝트 팩토리의 활용

DAO가 많아져 `DaoFactory`를 수정해야 한다고 해보자.

```java
public class DaoFactory {
    public UserDao userDao() {
        return new UserDao(new DConnectionMaker());
    }

    public AccountDao accountDao() {
        return new AccountDao(new DConnectionMaker());
    }

    public MessageDao messageDao() {
        return new MessageDao(new DConnectionMaker());
    }
}
```

중복되는 `new DConnectionMaker()`를 메서드로 분리하자.  
`ConnectionMaker` 구현체 클래스를 결정하는 코드를 분리한다고 볼 수도 있다.

```java
public class DaoFactory {
    public UserDao userDao() {
        return new UserDao(connectionMaker());
    }

    public AccountDao accountDao() {
        return new AccountDao(connectionMaker());
    }

    public MessageDao messageDao() {
        return new MessageDao(connectionMaker());
    }

    private ConnectionMaker connectionMaker() {
        return new DConnectionMaker();
    }
}
```

# 1.4.3 제어권의 이전을 통한 제어관계 역전

`제어의 역전`은 프로그램의 제어 흐름 구조가 뒤바뀐 것이다.

예컨데 일반적으로 자바 프로그램은 `main()`에서 시작되어 오브젝트 결정 한 뒤 생성하고, 메소드를 호출해서 다시 다음에 사용할 오브젝트를 결정하는 작업의 구성이 반복된다.

모든 오브젝트가 능동적으로 자신이 사용할 클래스를 결정하고, 오브젝트를 어떻게 만들지 언제 생성할지를 결정한다.

제어의 역전에서는 오브젝트가 자신이 사용할 오브젝트를 선택하지 않는다, 생성하지도 않고 심지어 자신이 어디서 만들어지고 사용되는지 알 수 없게 한다.  
다른 대상에게 이 일을 위임하기 때문이다.

## 📝 예시

제어의 역전은 이미 폭넓게 사용되고 있다.

서블릿은 서블릿 컨테이너에서 서블릿 오브젝트를 생성하고 메서드를 호출한다.  

서블릿 내에 `main()`이 있는 것도 아니고 서블릿 객체에서 실행을 제어할 수 도 없다.  
서블릿 컨테이너에게 서블릿 오브젝트의 생성과 사용을 위임한 제어의 역전 개념이 사용되었다고 볼 수 있다.

우리가 만든 `DaoFactory`도 제어의 역전을 적용한 예시라고 볼 수 있다.  

원래 `ConnectionMaker`의 구현 클래스를 결정하고 오브젝트를 만드는 제어권은 `UserDao`가 가지고 있었다.  
새롭게 만든 `DaoFactory`에게 이 역할을 위임했으니 이제 `UserDao`는 수동적인 존재가 되어 자신이 사용할 `ConnectionMaker` 구현체를 정할 수 없다.  
`UserDaoTest`도 같은 맥락으로 `UserDao`가 어떻게 만들어 지는 지 정할 수 없다.  
바로 이것이 제어의 역전이 일어난 상황이다.

프레임워크도 제어의 역전 개념이 사용되었다.

프레임워크는 단지 유용한 라이브러리의 집합이 아니다.  
프레임워크 자체가 애플리케이션 코드를 사용하고 애플리케이션 클래스를 제어한다.  
라이브러리는 애플리케이션 코드가 직접 호출해 흐름을 제어하니 프레임워크라고 할 수 없다.

## 💬 소감

그동안 제어의 역전이 좋다느니 스프링이 곧 제어의 역전이라느니 같은 소리만 들어봤지 이것이 의미하는 뜻을 잘 몰랐다.

이번 주제에서 다루면서 제어의 역전에 대한 개념을 짚고 갈 수 있었고, 앞으로 리펙토링을 할 때 생각할 수 있는 주요 개념으로 잘 써먹을 수 있으리라 기대한다.