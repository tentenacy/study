# 3장 스프링 컨텍스트: 빈 작성

빈 작성 → 빈 간 관계 설정

  

## 구성 파일에서 정의된 빈 간 관계 구현

### 1\. @Bean 메서드 직접 호출

```java
@Configuration
public class ProjectConfig {

    @Bean
    public Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    public Person person() {
        Person p = new Person();
        p.setName("Elia");
        p.setParrot(parrot());
        return p;
    }
}
```

- 빈 메서드를 직접 호출함에도 불구하고, 스프링은 빈의 인스턴스를 여전히 하나가 되도록 유지함
    - 스프링 컨텍스트에 이미 해당 빈이 있다면 그대로 반환
    - 스프링 컨텍스트에 해당 빈이 없다면 빈 메서드를 호출한 반환값을 반환

  

### 2\. @Bean 메서드의 매개변수로 빈 와이어링

```bash
@Configuration
public class ProjectConfig {

    @Bean
    public Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }

    @Bean
    public Person person(Parrot parrot) {
        Person p = new Person();
        p.setName("Elia");
        p.setParrot(parrot);
        return p;
    }
}
```

- 스프링은 `person()`  메서드의 `Parrot`  매개변수에 `parrot`  빈을 주입한다.

  

### 3\. @Autowired 애너테이션을 사용한 빈 주입

- @Autowired 애너테이션을 사용하면 스프링이 컨텍스트에서 값을 주입하기 원하는 객체 속성 표시
- 객체 간 관계를 더 쉽게 확인 가능

  

@Autowired 애터테이션을 사용하는 3가지 방법

1. 필드 주입

```bash
@Component
public class Person {

    private String name = "Ella";

    @Autowired
    private Parrot parrot;

    ...
}
```

  

2. 생성자 주입

```bash
@Component
public class Person {

    private String name = "Ella";

    private final Parrot parrot;

    @Autowired
    public Person(Parrot parrot) {
        this.parrot = parrot;
    }

    ...
}
```

- 유일하게 final field 사용 가능

  

3. 세터 주입

```bash
@Component
public class Person {

    private String name = "Ella";

    private Parrot parrot;

    ...

    @Autowired
    public void setParrot(Parrot parrot) {
        this.parrot = parrot;
    }
}
```

  

## 순환 의존성 다루기

객체 A, B가 있고, 서로가 서로에 대한 의존성을 갖는다면 스프링은 교착 상태에 빠짐

```bash
Caused by: org.springframework.beans.factory.BeanCurrentlyInCreationException: Error creating bean with name 'parrot': Requested bean is currently in creation: Is there an unresolvable circular reference?
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.beforeSingletonCreation(DefaultSingletonBeanRegistry.java:355)
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:227)
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:324)
```

- `BeanCurrentlyInCreationException` 
    - 이런 예외를 발견할 때마다 지정된 클래스를 찾아 순환 의존성 제거

  

## 스프링 컨텍스트에서 여러 빈 중 선택하기

`@Primary`  혹은 `@Qualifier`  사용

  

#Spring #스프링교과서