# large-data-mysql-partitioning
 > 📊 대용량 Stack Overflow 설문 데이터를 MySQL에 적재하고, 파티셔닝을 적용하여 RDBMS 성능 비교 실습을 진행한 프로젝트입니다.

## 👥팀원
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/Hyunsoo1998">
        <img src="https://github.com/Hyunsoo1998.png" width="100px;" alt="menzzi"/><br />
        <sub><b>김현수</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/shin-kibeom">
        <img src="https://github.com/shin-kibeom.png" width="100px;" alt="shin-kibeom"/><br />
        <sub><b>신기범</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/GodNowoon">
        <img src="https://github.com/GodNowoon.png" width="100px;" alt="shinjunsuuu"/><br />
        <sub><b>이노운</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/hyewon8245">
        <img src="https://github.com/hyewon8245.png" width="100px;" alt="GIHYUN-LEE"/><br />
        <sub><b>홍혜원</b></sub>
      </a>
    </td>
  </tr>
</table>

## 📌 프로젝트 개요
- **목표**: Stack Overflow의 개발자 설문 데이터를 MySQL에 적재하고, 파티셔닝을 적용하여 대용량 데이터 처리하고 성능 분석
- **핵심 내용**: 파티셔닝 전/후 성능 비교, 쿼리 효율성 실험, 데이터 탐색성 향상

## 📂 데이터셋
- **출처**: [Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey)
- **형식**: CSV
- **선택 컬럼**: user_id, Age, Country, LanguageHaveWorkedWith, CompTotal 등
- **전처리 과정**:
  - 불필요한 열 제거 + 3개년치 데이터 병합

## 🧱 시스템 구성
- **Database**: MySQL
- **개발 환경**: Google Colab, DBeaver
  


# 파티셔닝 전 / 후 메모리 5MB일때 소요 시간.

실제 운영 환경에서는 훨씬 더 큰 자원으로 서비스를 운영하지만, 제한된 자원으로 운영하는 상황이 발생하고, 파티셔닝 전/후 비교를 명확히 확인하기 위해서, mysql에 할당되는 메모리 크기를 하한선까지 설정한 후, 결과 비교를 진행했습니다.

mysql에서는 메모리 하한선은 5MB이며, 그 이하로 설정해도 Mysql에서 확인 시, 5MB는 확보되는 것을 확인할 수 있습니다. 

### OS에서 mysql의 메모리 값을 5MB 미만으로 설정 후, mysql 내부에서 메모리 리소스 확인 결과

OS에서 mysql의 메모리 사이즈를 1M로 설정해도 mysql에서 약 5M로 나오는 것을 확인할 수 있습니다. 

**(innodb_buffer_pool_size =  MySQL 서버의 RAM(메모리)에 할당되는 공간)**

<img width="1288" height="205" alt="image" src="https://github.com/user-attachments/assets/69f9173f-899f-42fe-ba27-2ca0879ce8ed" />


<img width="1288" height="232" alt="image 1" src="https://github.com/user-attachments/assets/027fa8b4-6bc7-489e-86a8-af8177a61b1b" />


## OS에서 mysql 메모리 값 설정 확인

<img width="1288" height="141" alt="image 2" src="https://github.com/user-attachments/assets/222fe8a4-1bd5-4f55-b4c9-b6b6a34d174b" />

-----------------------------------------------------------

## 파티션 테이블 생성 및 데이터 삽입

```sql
CREATE TABLE survey_partitioned (
  id INT NOT NULL AUTO_INCREMENT,
  Age VARCHAR(512),  -- ← 수정됨
  Country VARCHAR(512),
  CompTotal double,
  LanguageHaveWorkedWith varchar(512),
  DatabaseHaveWorkedWith varchar(512),
  RemoteWork VARCHAR(100),
  MainBranch VARCHAR(128),
  DevType TEXT,
  PRIMARY KEY (id, Country)
)
PARTITION BY LIST COLUMNS(Country) (
  PARTITION p_USA VALUES IN ('United States of America'),
  PARTITION p_India VALUES IN ('India'),
  PARTITION p_Germany VALUES IN ('Germany'),
  PARTITION p_Others VALUES IN ('Others')
);			-- 파티션 생성

UPDATE merged_survey
SET Country = 'Others'
WHERE Country NOT IN (
  'United States of America', 'India', 'Germany'
);			-- 미국, 인도, 독일 제외한 다른 나라들 Others로 설정

INSERT INTO survey_partitioned (
  Age, Country, CompTotal, LanguageHaveWorkedWith,
  DatabaseHaveWorkedWith, RemoteWork, MainBranch, DevType
)
SELECT
  Age, Country, CompTotal, LanguageHaveWorkedWith,
  DatabaseHaveWorkedWith, RemoteWork, MainBranch, DevType
FROM merged_survey;	-- 파티션에 데이터 삽입
```

원본 테이블을 조회 후 성능 확인

```sql
-- 원본
EXPLAIN ANALYZE
SELECT *
FROM merged_survey
WHERE Country = 'United States of America' and age = 'Under 18 years old';
```

파티션이 적용된 테이블의 Country를 기준으로 조회 후 성능 확인 

```sql

-- 파티션
EXPLAIN ANALYZE
SELECT *
FROM survey_partitioned
WHERE Country = 'United States of America' and age = 'Under 18 years old';

```
## 📈 쿼리 성능 비교

### ✅ 원본 테이블 (merged_survey)
```sql
-> Filter: ((merged_survey.Age = 'Under 18 years old') 
         and (merged_survey.Country = 'United States of America'))  
   (cost=23706 rows=2245) (actual time=2.112..1186 rows=2171 loops=1)
    -> Table scan on merged_survey  
       (cost=23706 rows=224547) (actual time=0.0607..904 rows=227889 loops=1)

-> Filter: ((merged_survey.Age = 'Under 18 years old') 
         and (merged_survey.Country = 'United States of America'))  
   (cost=23706 rows=2245) (actual time=1.79..415 rows=2171 loops=1)
    -> Table scan on merged_survey  
       (cost=23706 rows=224547) (actual time=0.0984..357 rows=227889 loops=1)

**파티셔닝을 하기 전에는 최대 2.11초까지 소요되었고 평균 1.7~1.8초정도 소요되었습니다**
```


### ✅ 파티션 테이블

```
-> Filter: ((survey_partitioned.Age = 'Under 18 years old') 
         and (survey_partitioned.Country = 'United States of America'))  
   (cost=4439 rows=429) (actual time=0.442..138 rows=2171 loops=1)
    -> Table scan on survey_partitioned  
       (cost=4439 rows=42866) (actual time=0.0748..107 rows=43285 loops=1)

-> Filter: ((survey_partitioned.Age = 'Under 18 years old') 
         and (survey_partitioned.Country = 'United States of America'))  
   (cost=4439 rows=429) (actual time=0.351..134 rows=2171 loops=1)
    -> Table scan on survey_partitioned  
       (cost=4439 rows=42866) (actual time=0.0544..105 rows=43285 loops=1)


파티셔닝을 진행한 후, 소요시간이 약 1.4초 줄어들어 평균 0.4초 정도 소요되는 것을 확인할 수 있었습니다.



```
## ✅ 데이터 수집 및 병합

- 2022, 2023, 2024년 설문 결과 CSV 파일을 수집하고, `Age`, `Country`, `CompTotal`, `LanguageHaveWorkedWith`, `DatabaseHaveWorkedWith`, `RemoteWork`, `MainBranch`, `DevType` 컬럼만 선택
- `year` 컬럼을 추가한 후 `pd.concat()`으로 병합
- 병합된 데이터에 대해 `user_id`를 부여하여 고유 식별자 설정

```python
# 병합 및 ID 부여
merged_df = pd.concat([df_2022, df_2023, df_2024], ignore_index=True)
merged_df.insert(0, 'user_id', range(1, len(merged_df) + 1))


## ✅ 제1정규화: 1:N 관계 분리

- `DevType`, `LanguageHaveWorkedWith`, `DatabaseHaveWorkedWith` 컬럼은 `;`로 구분된 다중값이 존재
- 이를 `explode()`로 분해하여 각각 정규화된 테이블(`use_devtype`, `use_language`, `use_database`)로 분리

```python
# 예시: Language 정규화
lang_df = df[['user_id', 'LanguageHaveWorkedWith']].dropna()
lang_df['LanguageHaveWorkedWith'] = lang_df['LanguageHaveWorkedWith'].str.split(';')
lang_df = lang_df.explode('LanguageHaveWorkedWith')
lang_df['LanguageHaveWorkedWith'] = lang_df['LanguageHaveWorkedWith'].str.strip()
lang_df = lang_df.rename(columns={'LanguageHaveWorkedWith': 'language'})

```
<img width="717" height="562" alt="image" src="https://github.com/user-attachments/assets/44f74fce-0c70-4a02-a5ba-ee2174c96a4b" />

---

## ✅ 주요 통계 분석 결과

### 💾 가장 많이 사용된 데이터베이스 (최근 3개년)

```sql
SELECT database, COUNT(*) AS count
FROM use_database
GROUP BY database
ORDER BY count DESC;

```

📊 결과 시각화:

<img width="402" height="528" alt="image 1" src="https://github.com/user-attachments/assets/7e13f991-9e42-4ffe-97f2-cd5be7c70b54" />


---

### 🧑‍💻 가장 많이 사용된 프로그래밍 언어

```sql
SELECT language, COUNT(*) AS count
FROM use_language
GROUP BY language
ORDER BY count DESC;

```

📊 결과 시각화:

<img width="413" height="551" alt="image 2" src="https://github.com/user-attachments/assets/3f877b7b-c8a6-4339-b718-aac7d54c3cb9" />


---

### 👩‍💻 가장 많이 선택된 개발자 유형

```sql
SELECT devtype, COUNT(*) AS count
FROM use_devtype
GROUP BY devtype
ORDER BY count DESC;

```

📊 결과 시각화:

<img width="532" height="678" alt="image 3" src="https://github.com/user-attachments/assets/eabe9aeb-45d3-45be-bae6-c0f9bfe47c6f" />

---
## ✅ 결론

<br>

특히, 파티션 기준 컬럼(Country)을 WHERE 조건에 활용하는 경우, 빠른 파티션 프루닝이 이루어져 속도 개선이 두드러집니다.

실서비스에서는 훨씬 높은 성능을 가진 자원에서 운영되므로, 파티셔닝 효과는 더욱 극대화될 수 있습니다.




## ✅ 회고

- `;`로 나뉜 다중 응답 컬럼은 정규화를 통해 1:N 관계 테이블로 분리
- 데이터 정제 후 SQL을 통해 단순 빈도 분석을 수행했으며, 연도별로도 확장 가능
- 향후에는 연도별 변화 추이 분석, 국가별 연봉 비교 등 추가 분석 가능

---

