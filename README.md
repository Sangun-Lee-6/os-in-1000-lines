# os-in-1000-lines

**RISC-V 기반 최소 운영체제 구현 프로젝트**  
1000줄 미만의 코드로 운영체제를 직접 구현하며, 컴퓨터 시스템 구조와 운영체제 개념을 실습합니다.

---

## 📚 진행 현황

| 챕터 | 제목                          | 상태    |
| ---- | ----------------------------- | ------- |
| ch01 | 환경 설정 및 QEMU 준비        | ✅ 완료 |
| ch02 | RISC-V 구조 및 명령어 입문    | ✅ 완료 |
| ch03 | 커널 기능 및 구조 개요        | ✅ 완료 |
| ch04 | 부트 로더 및 커널 진입점 구현 | ✅ 완료 |
| ch05 | Hello World 출력              | ✅ 완료 |
| ch06 | C 표준 라이브러리             | ⏳ 예정 |

---

## 🖥️ 실행 시연 (Ch4: 커널 부팅 과정)

아래는 `run.sh`를 통해 QEMU에서 커널을 부팅한 실제 화면입니다.  
`boot` → `kernel_main` → 무한 루프 진입까지의 과정을 확인할 수 있습니다.

<video src="docs/4.커널 부팅 과정/video/video1.mov" controls width="720"></video>

- 커널이 정상적으로 로딩되어 실행되고 있음을 `QEMU`와 `objdump`, `info registers`로 확인함
- pc → `kernel_main`의 무한 루프 주소(`0x80200048`)
- sp → 링커 스크립트의 `__stack_top` 주소(`0x8022004c`)로 설정됨

<br>

## 🖥️ 실행 시연 (Ch5: Hello World!)

커널에서 문자열을 직접 출력한 결과입니다.<br>
OpenSBI의 Console Putchar 기능을 통해 `Hello World!` 메시지가 터미널에 출력됩니다.

![image.png](./docs/5.Hello%20World!/img/image2.png)

- 커널의 printf() 호출 → putchar() → sbi_call() → ecall 실행 흐름을 통해 문자열 출력
- QEMU는 8250 UART 장치를 에뮬레이션하며 문자를 수신하여 표준 출력으로 전달
- Hello World! 외에도 정수 출력(%d), 16진수 출력(%x), 문자열 포맷(%s)까지 구현됨

---

## 🔧 실행 방법

```bash
# 커널 빌드 및 실행
./run.sh
```
