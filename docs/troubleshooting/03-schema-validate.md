# 03. 스키마 변경 후 기동 실패 (`ddl-auto: validate`)

| 항목 | 내용 |
| --- | --- |
| 발생 | 2026-05-13 |
| 계기 | RDS에 신규 테이블 추가 (목표 `purpose`, 일정 `calendar` 등) |

## 증상

```
java[213593]: at org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean.createNativeEntityManagerFactory(...)
systemd[1]: nextplan.service: Main process exited, code=exited, status=1/FAILURE
systemd[1]: nextplan.service: Scheduled restart job, restart counter is at 27.
```

## 확인 순서

1. 테이블 존재 여부 (pgAdmin)
   ```sql
   SELECT schemaname, tablename FROM pg_tables WHERE tablename = ai_analysis_records;
   -- schemaname: public
   ```
2. DB는 정상 → Hibernate `validate` 단계에서 엔티티와 실제 테이블 정의 불일치로 실패
3. 엔티티·DDL 정합성 맞춘 뒤 재시작 → 정상 기동

## 배운 점

- `validate`는 운영 DB를 앱이 임의로 바꾸지 못하게 지켜주는 대신, **DDL 적용과 앱 배포의 순서**를 사람이 관리해야 함
- 개선안: Flyway 마이그레이션 파일을 레포에 두고 배포 파이프라인에서 실행 → 스키마 변경도 코드 리뷰·이력 관리 대상으로
