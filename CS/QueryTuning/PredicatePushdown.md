# Predicate Pushdown

- 조건절을 더 아래 단계의 쿼리에서 처리하도록 pushdown 해서 최적화하는 쿼리 튜닝 방법

## 실제 튜닝 사례(26-04-13)

> [Query Tuning] Predicate Pushdown 수동 적용을 통한 성능 개선(70s -> 0.1s)

```SQL
-- pseudo code
-- [AS-IS] 병합 후 필터링 (Full Scan)
SELECT * FROM (
    SELECT ... FROM table1
    UNION
    SELECT ... FROM table2
) AS inline_view
WHERE condition_expression
GROUP BY condition_expression; -- 병합된 Temporary Table 전체 스캔

-- [TO-BE] 필터링 후 병합 (Index Scan)
SELECT * FROM (
    -- 인덱스 활용
    SELECT ... FROM table1 
    WHERE condition_expression 
    GROUP BY condition_expression
    UNION
    -- 인덱스 활용
    SELECT ... FROM table2 
    WHERE condition_expression 
    GROUP BY condition_expression
) AS inline_view;
```

1. 개요
- 중첩된 View와 UNION 연산이 포함된 특정 쿼리에서 발생하는 성능 저하(70s)를 분석하고, 옵티마이저가 처리하지 못하는 Predicate Pushdown을 수동으로 적용하여 0.1s로 개선

2. 현상 및 환경
- 환경: 로컬 구동 및 특정 권한 데이터 캐싱 시점
- 현상: 단순 조회성 쿼리임에도 불구하고 약 70초의 응답 지연 발생
- 구조: FROM절이 인라인 뷰로 구성되어 있었으며, 테이블이 UNION으로 묶여 있는 View 구조

3. 원인 분석 (Execution Plan)
- 추상화의 부작용: 비즈니스 로직 캡슐화를 위해 사용된 View가 옵티마이저의 최적화(Predicate Pushdown)를 방해함
- 물리적 병목: 실행 계획 확인 결과, UNION으로 결합된 모든 데이터를 메모리에 올려 Temporary Table을 생성한 뒤 외부 조건절을 필터링
- 결과: 인덱스가 존재함에도 불구하고 활용되지 못하고 임시 테이블에 대한 Full Scan이 발생

4. 해결 방안: 수동 최적화
- "합리적인 I/O 범위 내에서 데이터 규모 대비 이 정도의 지연이 발생할 수 없다"는 전제하에 쿼리 구조를 재설계
- 옵티마이저의 판단에 의존하지 않고, 인라인 뷰 내부의 개별 서브쿼리 단계로 조건절을 직접 전진 배치하여 인덱스 스캔을 강제

5. 결과
- 성능: 70s → 0.1s