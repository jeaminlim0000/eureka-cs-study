# Week04 Process & Thread

> 일자: 2026.07.28 · 관련 Issue: [#5](https://github.com/jeaminlim0000/eureka-cs-study/issues/5)

## 발표·학습 구성

남은 기록상 1조 임수환, 2조 이서진이 Process, Thread, PCB, Context Switching을 공통으로 설명했습니다.

## 함께 정리한 내용

- 프로세스는 실행 중인 프로그램이고, 스레드는 프로세스 안의 실행 흐름입니다.
- 스레드는 Code·Data·Heap을 공유하고 Stack만 독립적으로 가집니다.
- PCB는 프로세스 상태·프로그램 카운터·레지스터 값 등을 보관합니다.
- Context Switching은 현재 상태를 저장하고 다음 실행 상태를 복원하는 과정이며, 과도하면 처리량을 낮춥니다.

## 질문 기록

멀티스레드는 메모리 공유로 생성·통신 비용이 작지만 동기화 문제가 생깁니다. 멀티프로세스는 격리성이 높아 하나의 장애가 다른 프로세스로 전파될 위험이 작습니다.
