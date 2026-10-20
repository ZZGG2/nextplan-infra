# 02. systemd 재시작 무한 루프

| 항목 | 내용 |
| --- | --- |
| 발생 | 2026-05-05 (nohup → systemd 전환 당일) |
| 증상 | API 504, 그런데 `systemctl status`는 `active (running)` |

## 관찰

```
$ sudo systemctl status nextplan
● nextplan.service - NextPlan Spring Boot App
     Loaded: loaded (/etc/systemd/system/nextplan.service; enabled; preset: enabled)
     Active: active (running) since Tue 2026-05-05 10:39:45 UTC; 1s ago
   Main PID: 61308 (java)
     Memory: 45.7M
systemd[1]: nextplan.service: Scheduled restart job, restart counter is at 29.
systemd[1]: Started nextplan.service - NextPlan Spring Boot App.
```

`active (running)`은 "방금 다시 켜졌다"는 뜻일 뿐. 핵심은 `1s ago`와 `restart counter is at 29`.

## 원인 확인

```
$ sudo journalctl -u nextplan -n 20 | grep "ERROR\|Caused by"
Caused by: org.springframework.util.PlaceholderResolutionException:
  Could not resolve placeholder app.cors.allowed-origins in value "${app.cors.allowed-origins}"
```

- YAML 리스트로 정의한 CORS 허용 Origin을 `@Value`로 단일 값처럼 주입 → 기동 단계에서 예외
- `Restart=always`가 10초마다 계속 다시 띄우면서 장애가 "살아있는 것처럼" 보임

## 조치

1. 설정 바인딩 방식 수정 후 재배포
2. 새 JAR 반영 여부 확인
   ```
   $ ls -lt /home/ubuntu/app/
   -rw-r--r-- 1 ubuntu ubuntu 95314904 May  5 10:35 NextPlan-0.0.1-SNAPSHOT.jar
   ```
3. 기동 로그에서 `Tomcat started on port 8080` 확인 후 종료

## 배운 점

- 상태 확인은 `active` 여부가 아니라 **가동 시간 · 재시작 횟수 · 기동 로그** 세 가지로
- `Restart=always` 단독 사용 시 크래시 루프를 숨김 → `StartLimitBurst`/`StartLimitIntervalSec`로 횟수 제한 + 알림 연동이 필요
