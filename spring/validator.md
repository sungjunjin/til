# Spring Validator & Java Bean Validation

## Spring Validator
스프링에서 제공하는 Validator는 어플리케이션에서 사용하는 객체의 데이터를 검증하는 인터페이스이다.

### Validator 인터페이스 구현
```java
public class BookValidator implements Validator {
    @Override
    public boolean supports(Class<?> clazz) {
        return Book.class.equals(clazz);
    }

    @Override
    public void validate(Object target, Errors errors) {
        // ValidationUtils 사용
        ValidationUtils.rejectIfEmptyOrWhitespace(errors,"title", "notEmpty", "empty title is not allowed");

        Book book = (Book) target;

        // 커스텀 validation 로직 구현
        if(book.getTitle().length() >= 1000) {
            errors.reject("titleLength", "title length is more than 1000");
        }
    }
}
```

Validator 인터페이스의 supports, validate 메소드를 구현해준다. 각 메소드의 역할은 다음과 같다.

- supports : 매개변수로 넘어온 클래스 인스턴스(clazz)가 검증 대상인지 판별한다. 주로 클래스 타입을 비교한다. 
- validate : 매개변수로 넘어온 인스턴스(target)에 대한 데이터를 검증한다. 만약 검증에 오류가 발생하면 매개변수로 넘어온 error 객체에 등록 된다.
 - ValidationUtils를 활용해 객체 내부의 필드를 검증할 수도 있다
 - validation 로직을 사용자가 직접 커스텀 할 수 있다. 

### Validator 인터페이스 사용 예시
```java
@Component
public class AppRunner implements ApplicationRunner {

    @Override
    public void run(ApplicationArguments args) {
        Book book = new Book();
        book.setTitle("1230401283798123");

        BookValidator validator = new BookValidator();
        Errors bindingResult = new BeanPropertyBindingResult(book, "book");

        if(validator.supports(book.getClass())) {
            validator.validate(book, bindingResult);
        }


        System.out.println(bindingResult.hasErrors());

        bindingResult.getAllErrors().forEach(System.out::println);
    }
}
```

구현한 Validator는 일반 객체로 사용할 수 있고 빈으로 등록해서 사용할 수 있다.

## Java Bean Validation
Java Bean Validation은 JAVA EE 표준으로 자바에서 제공하는 validator 이다. 어노테이션을 기반으로 객체 내부의 데이터에 대한 검증을 실행한다.

