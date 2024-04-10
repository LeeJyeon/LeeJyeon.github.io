---
layout: post
title: 디자인 패턴 구현하기
categories:
  - Dev
tags:
- 디자인 패턴
- Spring boot
date: 2024-04-10
toc: true
comment: true


---

예제를 곁들인 디자인 패턴 구현하기
{: .message}

## 디자인 패턴
목적은 중복을 없애고, 커스텀한 관심사에 열정을 쏟는다..!

상황에 따라서 바뀔 수 있는 부분을 제외하고는 템플릿형태로 제공하자!

템플릿 메서드 / 전략패턴 / 템플릿콜백

이름만 거창하지 대충 하는 일이 비슷한 친구들이다!!

- 템플릿 메서드 패턴
1. 추상 클래스와 메서드를 이용해서 템플릿을 만들고
2. 추상 클래스를 상속받아 추상메서드dp 세부기능을 구현한다.
3. 전체적인 템플릿의 흐름대로 바깥에서 사용한다.

- 전략패턴
1. 전략을 Interface 로 구현해 둔다
2. 전략을 사용하는 템플릿을 구성해 둔다.
3. 해당 템플릿을 사용하는 바깥에서, 템플릿을 인스턴스화 할때 상황에 따라 전략을 주입한다..!
4. 이때 전략의 구현체를 받아도 되고~ 람다식으로 받아도 된다
5. 다만 람다식으로 받을때는 인터페이스에 1개의 메서드만 있어야 한다 ^^

- 템플릿 콜백패턴
1. 전략패턴과 명칭만 조금 다른 부분이 있을뿐 형태는 똑같다
2. 전략패턴에서 템플릿을 인스턴스화 하는 부분이 다르다. 
  - 전략패턴에서는 전략을 주입한 템플릿을 사용하는 반면, 템플릿콜백패턴에서는 템플릿은 인스턴스화하고, 메서드에서 주입받는다!


## 코드로 봐보자

요구사항은 아래와 같다.
- Dummy, Dumdummy Entity List 를 insert 한다!
- 근데 제약사항이 있다.
- Dummy.id 가 있는데, 사전에 주어진 Set 에 부합하게 저장해야한다.
  - Set = { "A", "B" } 이렇게 주어진다면, A 와 B 로 세팅된 Dummy List 가 저장되어야 한다.
  - "A" 만 저장하거나, 엉뚱한 "C" 가 저장되면 안된다.
- Dumdummy.degree 가 도 동일한 요구사항이지만, Integer 타입의 degree 를 검증해야한다.
  - Set = { 1, 2, 3 } 이렇게 기준값이 주어진다.
  - 마찬가지로, dumdummy.degree 는 1, 2, 3 을 포함되게끔 저장되어야 한다.
  - 1, 1, 1 → 실패 (2, 3이 없다)
  - 1, 1, 1, 2, 3 → 성공
  - 1, 2, 2, 2 → 실패 (3이 없다)
  - 1, 2, 3, 4 → 실패 (엉뚱한 4가 왔다)

뭔가 템플릿으로 동작할 수 있어보인다...!


### 전략패턴

전략을 정의한다. (람다식을 사용하기 위해 단일 메서드로 만들었다.)
```java
// ValidationCode → 해당 부분을 상황에 따라 String , Integer 로 지정한다.
public interface InsertStrategy<Entity, ValidationCode> {
    ValidationCode insert(Entity entity);
}
```

전략을 사용할 "템플릿" 을 정의한다. (전략의 insert 메서드를 잘 구현해 넣어주면 된다.)
```java
@RequiredArgsConstructor
public class InsertTemplate<Entity, ValidationCode> {

    // 템플릿을 인스턴스화 할때 생성자에 알맞은 전략을 넣어줘야 한다.
    private final InsertStrategy<Entity, ValidationCode> insertStrategy;

    // 저장할 Entity 리스트와, 기준이 되는 Set 을 인자로 받는다.
    public List<Entity> insertAndVerifyEnoughCode(List<Entity> sourceList
            , Set<ValidationCode> criteriaSet) {

        // 전략의 insert 에서 id(String), degree(Integer) 를 잘 리턴해주면 된다.
        Set<ValidationCode> codeSet =
                sourceList.stream()
                        .map(insertStrategy::insert)
                        .collect(Collectors.toSet());

        // 기준과 동일하게 저장이 되었는지 확인한다.
        if (!criteriaSet.equals(codeSet)) {
            throw new IllegalArgumentException("!");
        }

        return sourceList;
    }
}

```

바깥에서 템플릿을 사용해 보자

```java
@Service
@RequiredArgsConstructor
public class PatternService {

    private final DummyRepository dummyRepository;
    private final DumDummyRepository dumDummyRepository;

    public void patternInsert(List<DummyEntity> dummyEntityList
            , Set<String> codes
            , List<DumDummyEntity> dumDummyEntityList
            , Set<Integer> degrees) {

        List<Integer> sms = new ArrayList<>();

        // 템플릿을 생성하는데, 이때 전략을 람다식으로 구현해 넣었다.
        InsertTemplate<DummyEntity, String> dummyInsertTemplate =
                new InsertTemplate<>(
                        (DummyEntity dummy) -> {
                            dummyRepository.insert(dummy);
                            return dummy.getId();
                        });

        // 템플릿을 생성하는데, 이때 전략을 람다식으로 구현해 넣었다.
        InsertTemplate<DumDummyEntity, Integer> dumDummyInsertTemplate = 
                new InsertTemplate<>(
                        (DumDummyEntity dumDummy) -> {
                            dumDummyRepository.insert(dumDummy);
                            sms.add(dumDummy.getDegree());
                            return dumDummy.getDegree();
                            });

        List<DummyEntity> insertedDummy = 
                dummyInsertTemplate.insertAndVerifyEnoughCode(dummyEntityList, codes);
        List<DumDummyEntity> insertedDumDummy = 
                dumDummyInsertTemplate.insertAndVerifyEnoughCode(dumDummyEntityList, degrees);

        sms.forEach(System.out::println);
    }
}
```

### 템플릿 콜백패턴

전략을 정의한다. (전략패턴과 동일하다.)
```java
// ValidationCode → 해당 부분을 상황에 따라 String , Integer 로 지정할 생각이다.
public interface InsertStrategy<Entity, ValidationCode> {
    ValidationCode insert(Entity entity);
}
```

템플릿을 작성한다.
```java
public class InsertTemplate<Entity, ValidationCode> {

    // 멤버로 주입받는 전략이 없다
    // 메서드를 사용할때 클라이언트한테 주입받아 버린다...!
    public List<Entity> insertAndVerifyEnoughCode(
            List<Entity> sourceList
            , Set<ValidationCode> criteriaSet
            , InsertStrategy<Entity, ValidationCode> insertStrategy) {

        // 사용하는 insertStrategy.insert 부분을 클라이언트 쪽 코드에 의존한다. (이래서 콜백인듯)
        Set<ValidationCode> codeSet =
                sourceList.stream()
                        .map(insertStrategy::insert)
                        .collect(Collectors.toSet());

        if (!criteriaSet.equals(codeSet)) {
            throw new IllegalArgumentException("!");
        }
        return sourceList;
    }
}
```

바깥에서 템플릿을 사용해보자

```java
@Service
@RequiredArgsConstructor
public class PatternService {

    private final DummyRepository dummyRepository;
    private final DumDummyRepository dumDummyRepository;

    public void patternInsert(
            List<DummyEntity> dummyEntityList
            , Set<String> codes
            , List<DumDummyEntity> dumDummyEntityList
            , Set<Integer> degrees) {

        List<Integer> sms = new ArrayList<>();

        // 템플릿은 그냥 만들어버린다.
        InsertTemplate<DummyEntity, String> insertTemplate = new InsertTemplate<>();

        // 호출할때 전략을 직접 만들어준다.
        List<DummyEntity> insertedDummy =
                insertTemplate.insertAndVerifyEnoughCode(
                        dummyEntityList
                        , codes
                        , (DummyEntity dummy) -> {
                            dummyRepository.insert(dummy);
                            return dummy.getId();
                        });

        // 기존에 만들어진 템플릿을 사용해도 되지만, 검증하는 타입이 달라 별도로 만들었다. (dumdummy 는 Integer 타입으로 검증)
        InsertTemplate<DumDummyEntity, Integer> insertTemplate2 = new InsertTemplate<>();

        List<DumDummyEntity> insertedDumDummy =
                insertTemplate2.insertAndVerifyEnoughCode(
                        dumDummyEntityList
                        , degrees
                        , (DumDummyEntity dumDummy) -> {
                            dumDummyRepository.insert(dumDummy);
                            sms.add(dumDummy.getDegree());
                            return dumDummy.getDegree();
                        });
    }
}
```


## 결론
만들어 보면 별거없다.
공통 관심사를 잘 "템플릿" 으로 만들면 된다!