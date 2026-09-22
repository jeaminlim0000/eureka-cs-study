# Process · Thread · PCB · Context Switching — 임수환

> Week04 · 2026.07.28 · [Issue #5](https://github.com/jeaminlim0000/eureka-cs-study/issues/5)

프로세스는 실행 중인 프로그램이고, 스레드는 프로세스 안에서 실제 작업을 수행하는 실행 흐름입니다. 스레드는 Code·Data·Heap을 공유하고 Stack은 각자 독립적으로 가집니다.

PCB는 프로세스의 상태, 프로그램 카운터, 레지스터 값 등을 보관합니다. Context Switching은 현재 실행 상태를 저장하고 다음 실행 상태를 복원하는 과정이며, 너무 자주 발생하면 처리량이 낮아질 수 있습니다.
