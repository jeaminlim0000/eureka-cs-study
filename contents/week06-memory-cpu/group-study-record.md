# Memory · Paging · Cache · System Call — 공동 기록

> Week06 · 2026.08.11 · [Issue #7](https://github.com/jeaminlim0000/eureka-cs-study/issues/7)

개인별 발표 배정 기록이 확인되지 않아 공동 발표 기록으로 남깁니다.

가상 메모리는 프로세스별 독립 주소 공간을 제공하고, Paging은 고정 크기 페이지 단위로 메모리를 관리합니다. Page Fault는 필요한 페이지가 RAM에 없어 적재할 때 발생하며, 반복되면 응답 지연과 Thrashing으로 이어질 수 있습니다.

CPU Cache와 OS Page Cache는 RAM·디스크 접근 비용을 줄입니다. System Call은 애플리케이션이 파일·네트워크 I/O처럼 커널 기능을 요청하는 경계입니다.
