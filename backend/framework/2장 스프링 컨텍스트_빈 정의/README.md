# 2장 스프링 컨텍스트: 빈 정의

스프링의 인스턴스(빈; bean) 관리를 위해 반드시 **컨텍스트**에 인스턴스를 추가해야 함

컨텍스트: 프레임워크가 관리할 모든 객체 인스턴스를 추가하는 앱의 메모리 공간

  

## 메이븐 프로젝트 생성하기

### 빌드 도구?

사용하는 프레임워크에 관계없이 앱의 빌드 프로세스를 쉽게 관리하는 데 사용하는 도구

- Apache Maven
- Gradle

  

**앱 빌드에 자주 포함되는 작업**

- 앱에 필요한 의존성 내려받기
- 테스트 실행
- 구문이 정의한 규칙 준수 여부 검증
- 보안 취약점 확인
- 앱 컴파일
- 실행 가능한 아카이브에 앱 패키징

  

### 프로젝트 구성

- src 폴더: 소스 폴더라고도 하며, 앱에 속한 모든 것을 넣을 수 있다.
- **pom.xml 파일**: 새 종속성 추가처럼 메이븐 프로젝트 구성을 작성하는 파일이다.

  

## 스프링 컨텍스트에 새로운 빈 추가하기

- 외부 라이브러리 종속성의 빈도 추가 가능

  

### 1\. @Bean 애너테이션 사용

1. `@Configuration`  애너테이션이 지정된 프로젝트 구성 클래스 정의

```
@Configuration
public class ProjectConfig {
}
```

  

2. 객체 인스턴스를 반환하고 `@Bean`  애너테이션 지정된 메서드 정의

```
@Configuration
public class ProjectConfig {
    @Bean
    Parrot parrot() {
        Parrot p = new Parrot();
        p.setName("Koko");
        return p;
    }
}
```

- 기본적으로 빈 이름은 메서드 이름과 동일
- `@Bean()`  인자로 value나 name에 이름 지정 가능

  

3. 스프링이 구성 클래스를 사용하도록 설정

```
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Parrot p = context.getBean(Parrot.class);
        System.out.println("p.getName() = " + p.getName());
    }
}
```

  

### 2\. 스테레오타입 사용

1. @Component 애너테이션이 지정된 인스턴스 클래스 정의
    - “클래스를 컴포넌트로 표시한다”

```
@Component
public class Parrot {

    private String name;

    @PostConstruct
    public void init() {
        this.name = "Kiwi";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- `@Bean`  애너테이션을 사용하는 방식과는 다르게 public 메서드 호출이 불가하므로 `@PostConstruct`  사용

  

  

2. 스프링이 컴포넌트의 위치를 찾을 수 있도록 구성 클래스 정의
    - 이 찾는 과정을 **컴포넌트 스캔**이라고 함

```
@Configuration
@ComponentScan(basePackages = "org.example.main")
public class ProjectConfig {
}
```

  

  

3. 스프링이 구성 클래스를 사용하도록 설정

```
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Parrot p = context.getBean(Parrot.class);
        System.out.println(p);
        System.out.println("p.getName() = " + p.getName());
    }
}
```

  

### 3\. 프로그래밍 방식 사용

- 유연성이 가장 뛰어남
- 조건에 따라 분기하여 빈 생성 가능

  

1. 인스턴스 클래스 정의

```
public class Parrot {

    private String name;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

  

2. 비어있는 구성 클래스 정의

```
@Configuration
public class ProjectConfig {
}
```

  

3. 스프링 컨텍스트에 빈을 직접 등록

```
public class Main {
    public static void main(String[] args) {
        var context = new AnnotationConfigApplicationContext(ProjectConfig.class);
        Parrot x = new Parrot();
        x.setName("Kiki");

        Supplier<Parrot> parrotSupplier = () -> x;

//        context.registerBean("parrot1", Parrot.class, parrotSupplier);
        context.registerBean(
                "parrot1",
                Parrot.class,
                parrotSupplier,
                bc -> bc.setPrimary(true)
        );
        Parrot p = context.getBean(Parrot.class);
        System.out.println("p.getName() = " + p.getName());
    }
}
```

  

## 빈 생성 방식 선택하기

**우선순위**

1. 스테레오타입 사용
    - 특정 애너테이션이 있는 애플리케이션 클래스만을 위한 빈 생성 가능
    - 상용구 코드가 적고, 편하게 읽을 수 있으므로 가장 선호됨
2. @Bean 애너테이션 사용
    - 스테레오타입 사용하는 방식보다 유연함
    - 상용구 코드가 다소 있음
3. 프로그래밍 방식 사용
    - 가장 유연하지만, 사용이 번거롭고 상용구 코드가 있음
    - 스프링 컨텍스트에 빈을 추가하는 로직을 재정의하여 구현할 수 있음

  

#스프링교과서 #Spring