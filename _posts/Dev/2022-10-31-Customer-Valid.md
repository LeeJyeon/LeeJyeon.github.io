---
layout: post
title: Custmoer Valid
categories:
  - Dev
tags:
- Valid
- Annotation
- Spring boot
date: 2022-10-31
toc: true
comment: true


---

@Valid Customize in Spring Boot
{: .message}

## @Valid

Request parm / body 에 대한 검증 + 커스텀마이징 애노테이션


## Gradle 의존성 추가

Validation annotation 사용을 위한 의존성 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```


## RestController

```java
    @PostMapping(value = "/valid/test")
    public ResponseEntity<String> postValidator(@RequestBody @Valid ValidationEntity validationEntity) {
        log.info(validationEntity.toString());
        return new ResponseEntity<String>("OK", HttpStatus.OK);
    }
```
RequestBody 로 받는 객체에 @Valid 체크 애노테이션을 붙여준다!

RequestBody 에서 입력값으로 받는 객체(ValidationEntity)는 다음과 같이 되어 있다.

```java
public class ValidationEntity {

    @Min(1) // 1이상의 값을 입력받음
    private int customerSequence;

    @Size(min = 10, max = 20) // 길이가 10~20 사이
    private String customerId;

    @Positive // 양수여야 함
    private double balance;

    @NotBlank // NotBlank , NotNull, NotEmpty 등..
    @RequestValid(typeCode = "UD-TYPE-VALID") // Customizing annotation
    private String upDownCode;

    @RequestValid(typeCode = "LR-TYPE-VALID") // Customizing annotation
    private String leftRightCode;

}
```

주석과 같이 Controller 에 진입하면, RequestBody 를 위와 같은 조건으로 검증한다.

## Customizing Valid Annotation

사전에 주어진 검증 조건이 아닌, 직접 만들어 사용할 수 있다.

아래는 typeCode 라는 값을 받는, @RequestValid 이다.

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = RequestValidator.class)
public @interface RequestValid {
    String message() default "";

    Class<?>[] groups() default {};

    Class<? extends Payload>[] payload() default {};

    public String typeCode(); // annotation 으로 받을 입력값

    @Target(ElementType.FIELD)
    @Retention(RetentionPolicy.RUNTIME)
    @interface LIST {
        RequestValid[] value();
    }
}
```

@Constraint(validatedBy = RequestValidator.class)

와 같이, 해당 애노테이션이 붙어있으면, RequestValidator 의 로직으로 검증한다.

```java
public class RequestValidator implements ConstraintValidator<RequestValid, String> {

    private String typeCode;

    @Override
    public void initialize(RequestValid constraintAnnotation) {
        typeCode = constraintAnnotation.typeCode();
    }

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {

        boolean isNormal = true;
        String message = "";


        if (typeCode.equals("LR-TYPE-VALID")) {
            if ("LEFT".equals(value) || "RIGHT".equals(value)) {
                isNormal = true;
            } else {
                isNormal = false;
                message = "LEFT or RIGHT is only allowed";
            }
        } else if (typeCode.equals("UD-TYPE-VALID")) {
            if ("UP".equals(value) || "DOWN".equals(value)) {
                isNormal = true;
            } else {
                isNormal = false;
                message = "UP or DOWN is only allowed";
            }
        }

        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(message).addConstraintViolation();
        
        return isNormal;
    }
}
```
ConstraintValidator 의 구현체이며, initialize 와 isValid 를 구현해야한다.

- initialize

애노테이션을 통해, 입력받은 typeCode 를 전역변수로 저장하고. (싱글톤 관련된 문제가 있을지는 확인해 봐야겠다. - https://zgundam.tistory.com/27 )

- isValid

검증을 진행한다. typeCode 에 따라, RequestBody 로 받은 field 의 검증 로직을 다르게 설정했다.

유효하지 않는 값이라면, false 를 return 한다. 

추가로 ConstraintValidatorContext 의 인스턴스인 context 의 메소드도 호출해준다.

context.buildConstraintViolationWithTemplate 같은 경우 Junit Test code 에서도 활용된다.


## Exception Handler

유효하지 않을경우 "MethodArgumentNotValidException" 이 발생한다.

@ControllerAdice 를 통해, Controller 에서 발생하는 예외를 캐치해 처리한다.

```java
@ControllerAdvice
public class DefaultExceptionAdvice {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    protected ResponseEntity<Object> handleException(final MethodArgumentNotValidException exception) {
        Map<String, Object> result = new HashMap<>();
        List<FieldError> errors = exception.getBindingResult().getFieldErrors();

        for (FieldError error : errors) {
            result.put("Error", "[ " + error.getField() + " ] invalid!!");
        }
        return new ResponseEntity<>(result, null, HttpStatus.OK);
    }
}
```

## Test Code

테스트 코드 작성시, Valid 애노테이션을 기준으로 어느정도 검증이 가능한 객체들을 자동생성해 주는 기능이 있다.


```java
@Test
public void validationTest() {

    final Validator validator = Validation.buildDefaultValidatorFactory().getValidator();

    ValidationEntity validationEntity = ValidationEntity.builder()
            .customerId("123").leftRightCode("").upDownCode("").build();

    // 팩토리를 통해 애노테이션에 대한 유효케이스를 생성한다.
    Set<ConstraintViolation<ValidationEntity>> testSet = validator.validate(validationEntity);

    Iterator<ConstraintViolation<ValidationEntity>> iterator = testSet.iterator();

    List<String> violations = new ArrayList<>();

    while (iterator.hasNext()) {
        ConstraintViolation<ValidationEntity> next = iterator.next();
        // 케이스를 추가한다.
        violations.add(next.getMessage());
        System.out.println(next.getMessage());
    }

    Assert.assertEquals(6, violations.size());

}
```

위와 같이 작성할 경우, 직접만든 애노테이션은 자동케이스가 생성이 안된다.

그럴경우 팩토리에 넣기전에, validationEntity.setField() 를 통해 직접 케이스를 추가후 아래 코드를 진행하면 된다!

- 정상 케이스
![OK](/assets/img/Validation-annotation-OK.png)

- 실패 케이스 1
![Fail-1](/assets/img/Validation-annotation-fail-1.png)

- 실패 케이스 2
![Fail-2](/assets/img/Validation-annotation-fail-2.png)