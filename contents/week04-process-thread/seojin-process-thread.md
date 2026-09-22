# Process · Thread · PCB · Context Switching — 이서진

> Week04 · 2026.07.28 · [Issue #5](https://github.com/jeaminlim0000/eureka-cs-study/issues/5)

프로세스는 OS로부터 자원을 할당받은 실행 중인 프로그램이고, 스레드는 프로세스 내부의 실행 단위입니다. 멀티스레드는 메모리 공유로 통신 비용이 작지만 동기화 문제가 생길 수 있습니다. 멀티프로세스는 메모리가 분리되어 한 프로세스의 장애가 다른 프로세스로 전파될 위험이 작습니다.
