# 04. 외부 AI 서버(Colab + ngrok) 연동 안정화

## 구조

```
Spring Boot ──HTTPS POST /api/analyze──▶ ngrok 고정 도메인 ──▶ Colab (uvicorn)
     │                                                        │
     └─ S3 PutObject (Private 버킷) ◀── Presigned GET URL(30분) ─┘
```

## 문제와 조치

| 문제 | 조치 |
| --- | --- |
| Colab 무료 세션 약 12시간 → 재실행 시 ngrok URL 변경 → EC2 환경변수 수정 + 재시작 반복 | ngrok **고정 도메인** 사용, `AI_SERVER_URL`은 한 번만 설정 |
| AI 서버가 살아있는지 백엔드 쪽에서 판단할 방법 부재 | 점검 절차 정리 (아래) |
| 분석이 길어지면 업로드 요청 전체가 대기 → 프록시/엣지 타임아웃 위험 | 업로드는 S3 저장 후 **분석 ID 즉시 반환**, 분석은 트랜잭션 커밋 이후 **비동기** 처리 (타임아웃 3분), 상태는 `PROCESSING`으로 기록 후 갱신 |
| AI 서버에 파일을 어떻게 넘길지 | 파일 자체 대신 **30분 만료 Presigned URL** 전달 → 버킷 비공개 유지 |

## 점검 절차 (EC2에서)

```
$ curl -I https://<ngrok-domain>/api/analyze
HTTP/2 405
allow: POST
server: uvicorn
```
`405 Allow: POST` = AI 서버 정상 기동 (GET이라 거절된 것). 연결 오류·ngrok 오프라인 페이지면 Colab 재실행.

```
$ sudo tail -f /var/log/nginx/access.log     # 업로드 요청이 EC2까지 왔는지
$ sudo journalctl -u nextplan -f              # 앱 → AI 서버 호출 결과
```

## 한계와 개선 방향

- Colab은 세션 만료·단일 노드 → 운영용 아님
- 개선: AI 서버 컨테이너화 → EC2 GPU 인스턴스 / ECS / SageMaker Endpoint, 내부 통신은 VPC 안에서 처리하고 ngrok 제거
