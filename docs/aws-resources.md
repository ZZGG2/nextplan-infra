# AWS 리소스 구성

리소스는 프로젝트 종료 후 모두 삭제했다. 코드·설정 파일·작업 기록에서 확인되는 값만 정리했다.

| 리소스 | 구성 | 근거 |
| --- | --- | --- |
| Region | `ap-northeast-2` (서울) | application.yml, 워크플로 |
| CloudFront | 단일 Distribution. Default Behavior → S3(정적), `/api/*` → EC2 | 프론트 `VITE_API_BASE_URL`이 CloudFront 도메인, 트러블슈팅 기록 |
| AWS WAF | CloudFront 생성 시 함께 만들어진 Web ACL, AWS Managed Core Rule Set. `SizeRestrictions_BODY`만 Count | WAF Sampled requests 기록 |
| CloudFront 요금제 | flat-rate Free plan (Web ACL 연결 해제 불가 제약 확인) | 콘솔 메시지 |
| S3 (정적) | Vite 빌드 결과물, `aws s3 sync --delete`로 배포 | frontend-deploy.yml |
| S3 (이력서) | `nextplan-resume`, 비공개. Presigned GET URL 30분 | application.yml, 업로드 서비스 코드 |
| EC2 | Ubuntu, VPC 서브넷 `10.0.1.0/24` 대역, Nginx + Spring Boot(systemd) | 셸 프롬프트 `ip-10-0-1-176`, 유닛 파일 |
| RDS | PostgreSQL, DB명 `nextplan`, 포트 5432 | JDBC URL |
| IAM | 프론트 배포: Access Key(GitHub Secrets). 백엔드 S3 접근: SDK 기본 Credential Chain | 워크플로, 코드 |
| AI 서버 | Google Colab + ngrok 고정 도메인, uvicorn | curl 응답 헤더 |

<!-- 기억나는 대로 보완: EC2 인스턴스 타입, RDS 서브넷(Public/Private), 보안 그룹 인바운드 규칙, S3 정적 Origin 접근 방식(OAC 여부) -->
