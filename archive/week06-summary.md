# Week06 OS 메모리와 CPU

> 일자: 2026.08.11 · 관련 Issue: [#7](https://github.com/jeaminlim0000/eureka-cs-study/issues/7)

## 발표 기록 상태

노션에는 6주차 제목만 남아 있어 개인별 발표 배정은 확인되지 않습니다. 아래 내용은 당시 스터디에서 다룬 공통 키워드를 요약합니다.

## 함께 정리한 내용

- 가상 메모리는 프로세스마다 독립된 주소 공간을 제공하고, Paging은 고정 크기 페이지 단위로 메모리를 관리합니다.
- Page Fault는 필요한 페이지가 RAM에 없어 디스크에서 적재해야 할 때 발생합니다. 반복되면 응답 지연과 Thrashing으로 이어질 수 있습니다.
- CPU Cache와 OS Page Cache는 RAM·디스크 접근 비용을 줄입니다. JVM Heap만 크게 잡지 말고 전체 메모리 여유도 고려해야 합니다.
- System Call은 애플리케이션이 파일 I/O·네트워크 I/O 등 커널 기능을 요청하는 경계입니다.
