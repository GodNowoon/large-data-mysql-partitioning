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

## 고찰
- ㅇㅇ
---
