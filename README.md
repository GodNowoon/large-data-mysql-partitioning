# large-data-mysql-partitioning
 > 📊 대용량 Stack Overflow 설문 데이터를 MySQL에 적재하고, 파티셔닝을 적용하여 RDBMS 성능 실습을 진행한 프로젝트입니다.

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
- **목표**: Stack Overflow의 개발자 설문 데이터를 MySQL에 적재하고, 파티셔닝을 적용하여 대용량 데이터 처리 성능을 실습하고 분석합니다.
- **핵심 내용**: 파티셔닝 전/후 성능 비교, 쿼리 효율성 실험, 실시간 데이터 탐색성 향상

## 📂 데이터셋
- **출처**: [Stack Overflow Developer Survey](https://insights.stackoverflow.com/survey)
- **형식**: CSV
- **선택 컬럼**: user_id, Age, Country, LanguageHaveWorkedWith, CompTotal 등
- **전처리 과정**:
  - 불필요한 열 제거 + 3개년치 데이터 병합

## 🧱 시스템 구성
- **Database**: MySQL
- **개발 환경**: Google Colab, DBeaver
- **기술 스택**: SQL, MySQL Partitioning, CSV Import

## 🧪 파티셔닝 전략
- **기법 적용**

- **파티셔닝 기준 선정 이유**


## ⚡ 성능 테스트
- **테스트 시나리오**
- **비교 항목**


## ✅ 데이터 수집 및 병합

- 2022, 2023, 2024년 설문 결과 CSV 파일을 수집하고, `Age`, `Country`, `CompTotal`, `LanguageHaveWorkedWith`, `DatabaseHaveWorkedWith`, `RemoteWork`, `MainBranch`, `DevType` 컬럼만 선택
- `year` 컬럼을 추가한 후 `pd.concat()`으로 병합
- 병합된 데이터에 대해 `user_id`를 부여하여 고유 식별자 설정

```python
# 병합 및 ID 부여
merged_df = pd.concat([df_2022, df_2023, df_2024], ignore_index=True)
merged_df.insert(0, 'user_id', range(1, len(merged_df) + 1))

```

---

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

## ✅ 회고

- `;`로 나뉜 다중 응답 컬럼은 정규화를 통해 1:N 관계 테이블로 분리
- 데이터 정제 후 SQL을 통해 단순 빈도 분석을 수행했으며, 연도별로도 확장 가능
- 향후에는 연도별 변화 추이 분석, 국가별 연봉 비교 등 추가 분석 가능

---

