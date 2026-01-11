# Java-Stream
**<h1>Java-Stream API 사용 가이드 및 실측 보고서</h1>**


이 문서는 Java Stream API를 효율적이고 안전하게 사용하기 위한 7가지 핵심 원칙을 실습해보고 실습 결과를 다룹니다.


---

**<h2>? Stream API 사용 시 주의해야 할 7가지 </h2>**

**1. 재사용 불가능 (One-time Use)**: 종단 연산 후 스트림은 닫힙니다. 필요 시 다시 생성하세요.

**2. 무분별한 병렬 스트림 지양**: 스레드 관리 비용으로 인해 데이터가 적으면 오히려 느려질 수 있습니다.

**3. 부작용(Side Effect) 피하기**: 외부 변수 수정은 지양하고 collect()를 사용하세요. (실험 섹션 참고)

**4. 메서드 참조 활용**: 람다가 길어지면 메서드로 분리하여 가독성을 높이세요.

**5. 무한 스트림 주의**: iterate(), generate() 사용 시 반드시 limit()을 설정하세요.

**6. 박싱(Boxing) 오버헤드**: IntStream 등 기본형 스트림을 사용하여 성능을 최적화하세요.

**7. 전통적 Loop 고려**: 가독성과 성능 면에서 단순 for-loop가 유리할 때도 있습니다.


<h2> ? 실증 테스트 1: 연산의 복잡도와 병렬화의 상관관계 </h2>

**병렬 처리는 데이터를 쪼개고(Split) 합치는(Merge) 과정에서 오버헤드가 발생합니다.**

<h3>실험 과정</h3>
<h4>1. 가벼운 연산 : 100만 개 데이터에 대해 단순 제곱(n * n) 수행. </h4>

# Java-Stream

<h1>Java-Stream API 사용 가이드 및 실측 보고서</h1>

이 문서는 Java Stream API를 효율적이고 안전하게 사용하기 위한 7가지 핵심 원칙을 실습해보고 그 결과를 다룹니다.

---

<h2>? Stream API 사용 시 주의해야 할 7가지 </h2>

1. **재사용 불가능 (One-time Use)**: 종단 연산 후 스트림은 닫힙니다. 필요 시 다시 생성하세요.
2. **무분별한 병렬 스트림 지양**: 스레드 관리 비용으로 인해 데이터가 적으면 오히려 느려질 수 있습니다.
3. **부작용(Side Effect) 피하기**: 외부 변수 수정은 지양하고 `collect()`를 사용하세요. (실험 섹션 참고)
4. **메서드 참조 활용**: 람다가 길어지면 메서드로 분리하여 가독성을 높이세요.
5. **무한 스트림 주의**: `iterate()`, `generate()` 사용 시 반드시 `limit()`을 설정하세요.
6. **박싱(Boxing) 오버헤드**: `IntStream` 등 기본형 스트림을 사용하여 성능을 최적화하세요.
7. **전통적 Loop 고려**: 가독성과 성능 면에서 단순 for-loop가 유리할 때도 있습니다.

---

<h2> ? 실증 테스트 1: 연산의 복잡도와 병렬화의 상관관계 </h2>

**병렬 처리는 데이터를 쪼개고(Split) 합치는(Merge) 과정에서 오버헤드가 발생합니다.**



<h3>실험 과정</h3>

<h4>1. 가벼운 연산 : 100만 개 데이터에 대해 단순 제곱(n * n) 수행. </h4>

<details><summary>1번 실험 코드 펼치기/접기</summary>

```java
package lab02;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class LargeDatasetPerformanceTest {

    public static void main(String[] args) {
        // 프로그램 실행 시 측정 메서드 호출
        measureLargeTask();
    }

    public static void measureLargeTask() {
        // 100만 개의 데이터 생성
        List<Integer> numbers = IntStream.rangeClosed(1, 1_000_000)
                                         .boxed()
                                         .collect(Collectors.toList());

        // [비교군] 직렬 스트림 실행 시간
        long startSerial = System.nanoTime();
        List<Integer> serialResult = numbers.stream()
                                            .map(n -> n * n)
                                            .collect(Collectors.toList());
        long endSerial = System.nanoTime();

        // [대조군] 병렬 스트림 실행 시간
        long startParallel = System.nanoTime();
        List<Integer> parallelResult = numbers.parallelStream()
                                              .map(n -> n * n)
                                              .collect(Collectors.toList());
        long endParallel = System.nanoTime();

        System.out.println("Serial Stream Time: " + (endSerial - startSerial) / 1_000_000.0 + " ms");
        System.out.println("Parallel Stream Time: " + (endParallel - startParallel) / 1_000_000.0 + " ms");
    }
}

```
</details>

<img width="827" alt="실험1 결과" src="images/image1.png" />


<h4> 2. 무거운 연산 : 복잡한 수학 연산(Math.sqrt, sin, cos) 루프를 100회 추가 </h4>
 <details><summary>2번 실험 코드 펼치기/접기</summary>

``` java
package lab02;

import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class LargeDatasetPerformanceTest {

    public static void main(String[] args) {
        measureLargeTask();
    }

    public static void measureLargeTask() {
        // 100만 개의 데이터 생성
        List<Integer> numbers = IntStream.rangeClosed(1, 1_000_000)
                                         .boxed()
                                         .collect(Collectors.toList());

        // [비교군] 직렬 스트림 실행 시간
        long startSerial = System.nanoTime();
        List<Double> serialResult = numbers.stream()
                                            // .map(n -> n * n) // 기존 가벼운 연산
                                            .map(LargeDatasetPerformanceTest::heavyCompute) // 무거운 연산 추가
                                            .collect(Collectors.toList());
        long endSerial = System.nanoTime();

        // [대조군] 병렬 스트림 실행 시간
        long startParallel = System.nanoTime();
        List<Double> parallelResult = numbers.parallelStream()
                                              // .map(n -> n * n) // 기존 가벼운 연산
                                              .map(LargeDatasetPerformanceTest::heavyCompute) // 무거운 연산 추가
                                              .collect(Collectors.toList());
        long endParallel = System.nanoTime();

        System.out.println("Serial Stream Time: " + (endSerial - startSerial) / 1_000_000.0 + " ms");
        System.out.println("Parallel Stream Time: " + (endParallel - startParallel) / 1_000_000.0 + " ms");
    }

    // CPU 부하를 시뮬레이션하기 위한 무거운 연산 메서드
    private static double heavyCompute(Integer n) {
        double result = 0;
        // 각 숫자마다 100번의 복잡한 수학 연산을 수행
        for (int i = 0; i < 100; i++) {
            result += Math.sqrt(Math.sin(i) * Math.cos(i) + Math.PI);
        }
        return result + n;
    }
}
```
</details>

<img width="827" alt="실험2 결과" src="images/image2.png" />

결과: 무거운 연산에서는 병렬(Parallel) 스트림이 훨씬 효율적입니다.



**최종 요약**
- 데이터가 적거나 연산이 단순하면? 직렬 스트림 사용.

- 데이터가 많고 연산이 복잡하면? 병렬 스트림 고려.

---


<h3> 추가 실험 내용 </h3>
객체 생성 비용을 최소화하여 병렬 처리의 성능을 극대화하는 실험을 진행했습니다.

<h4>1. boxed() 메소드와 오버헤드</h4>
`boxed()` 메소드는 기본형 스트림(Primitive Stream)을 객체형 스트림(Object Stream)으로 변환합니다. Java의 `IntStream`, `LongStream` 등은 메모리 효율을 위해 기본형 데이터를 직접 다루지만, 일반 `Stream<Integer>`는 래퍼 클래스 객체로 감싸는 '박싱' 과정이 필요하며 이 과정에서 추가적인 자원이 소모됩니다.

<details> <summary>최적화 실험 코드 펼치기/접기</summary>

```java
package lab02;

import java.util.stream.IntStream;

public class LargeDatasetPerformanceTest1 {

    public static void main(String[] args) {
        measureLargeTask();
    }

    public static void measureLargeTask() {
        int size = 1_000_000;

        // [비교군] 직렬 기본형 스트림
        long startSerial = System.nanoTime();
        double serialSum = IntStream.rangeClosed(1, size)
                                    .mapToDouble(LargeDatasetPerformanceTest1::heavyCompute)
                                    .sum(); 
        long endSerial = System.nanoTime();

        // [대조군] 병렬 기본형 스트림
        long startParallel = System.nanoTime();
        double parallelSum = IntStream.rangeClosed(1, size)
                                      .parallel()
                                      .mapToDouble(LargeDatasetPerformanceTest1::heavyCompute)
                                      .sum();
        long endParallel = System.nanoTime();

        System.out.println("Serial Stream (Primitive) Time: " + (endSerial - startSerial) / 1_000_000.0 + " ms");
        System.out.println("Parallel Stream (Primitive) Time: " + (endParallel - startParallel) / 1_000_000.0 + " ms");
    }

    private static double heavyCompute(int n) {
        double result = 0;
        for (int i = 0; i < 100; i++) {
            result += Math.sqrt(Math.sin(i) * Math.cos(i) + Math.PI);
        }
        return result + n;
    }
}
```

</details>

<img width="827" alt="실험3 결과" src="images/image3.png" />

---

<h3>3. 실험 결과 및 분석</h3>

**분석**: 기본형 스트림을 사용함으로써 객체 생성 및 관리 오버헤드를 줄였고, 병렬 처리 시 코어의 연산 능력을 더욱 순수하게 활용할 수 있었습니다.

**이유**: parallelStream()은 스레드 생성, 데이터 분할(Split), 결과 합치기(Combine) 과정에서 자원(Overhead)이 소모됩니다. 따라서 연산 자체가 매우 가벼우면 병렬화 준비 시간이 더 길어질 수 있으나, 본 실험처럼 무거운 연산과 기본형 스트림이 결합될 때 최적의 성능을 냅니다.