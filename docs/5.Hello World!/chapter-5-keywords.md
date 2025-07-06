# chapter-5-keywords

## 📌 `ecall` 명령어

✅ **개념**

- `ecall`은 environment call → 상위 환경에게 호출
- 상위 권한 수준에게 요청할 때 사용하는 명령어
- 사용자 모드 → 커널 || 커널 → 펌웨어

✅ 사용 예시

- 사용자 모드 → OS 커널 : 사용자 프로그램이 시스템콜을 요청할 때
- OS 커널 → 펌웨어 : 커널이 SBI 호출을 할 때

🤖 예시: SBI 호출 (OS → OpenSBI)

- SBI 호출을 받으면 OpenSBIsms M-mode에서 실행됨 → 그러고 ecall에 응답

```c
register int a7 asm("a7") = 0; // SBI call 번호
__asm__ volatile ("ecall");
```

## 📌 `sbi_call` 설명

✅ SBI 호출 함수 정의

- 즉, 이 함수는 특정 기능을 수행하지 않고 SBI 호출 규약에 맞게 인자를 설정하고 `ecall`을 실행하는 함수

🤖 `sbi_call` 코드

```c
struct sbiret sbi_call(long arg0, long arg1, long arg2, long arg3,
                       long arg4, long arg5, long fid, long eid) {
    register long a0 __asm__("a0") = arg0;
    register long a1 __asm__("a1") = arg1;
    register long a2 __asm__("a2") = arg2;
    register long a3 __asm__("a3") = arg3;
    register long a4 __asm__("a4") = arg4;
    register long a5 __asm__("a5") = arg5;
    register long a6 __asm__("a6") = fid;
    register long a7 __asm__("a7") = eid;

    __asm__ __volatile__("ecall"
                         : "=r"(a0), "=r"(a1)
                         : "r"(a0), "r"(a1), "r"(a2), "r"(a3), "r"(a4), "r"(a5),
                           "r"(a6), "r"(a7)
                         : "memory");
    return (struct sbiret){.error = a0, .value = a1};
}
```

👉 `fid`, `eid`

- `eid` : Extension ID : SBI 안에서 어떤 그룹(extension)을 사용할지 결정하는 번호
- `fid` : Fucntion ID : 해당 extension 안에서 어떤 함수를 사용할지 결정하는 번호

👉 `register long a0 __asm__("a0") = arg0;`

- C 변수 `arg0`을 RISC-V의 `a0` 레지스터에 직접 연결하는 코드
- 이런 방식으로 `a0` 레지스터 ~ `a7` 레지스터에 변수 연결

👉 `a0` 레지스터부터 `a7` 레지스터의 의미는 `SBI 호출 규약`에 따른 것

- `a0 ~ a5` : 함수 인자
- `a6` : 함수 번호 (Function ID)
- `a7` : 확장 ID (Extension ID)

👉 `__asm__ __volatile__("ecall" ...)`

- `ecall` 명령으로 상위 권한에 시스템 호출 요청(여기서는 커널이 OpenSBI에게 시스템콜 요청)

👉 출력 오퍼랜드 `=r(a0), =r(a1)`

- SBI 응답 결과

👉 `return (struct sbiret){.error = a0, .value = a1};`

- 호출 결과를 구조체(`sbiret`)로 반환

## 📌 `putchar`

✅ 문자 출력 함수

- OpenSBI의 콘솔 출력 기능을 시스템콜로 요청하는 함수
- 이 함수의 번호는 `eid =1, fid =0`
- 함수의 규칙 : 문자 하나를 `a0`에 넣고 `a1`-`a5`는 `0`으로 세팅
- 이 함수는 출력 전용 → ∴ `sbiret`이 없음 → ∴ `void`

```c
void putchar(char ch) {
    sbi_call(ch, 0, 0, 0, 0, 0, 0, 1);  // fid=0, eid=1: Console Putchar
}
```

## 📌 `kernel_main()`

👉 문자열 출력

- 문자열을 `putchar()` 함수로 한 글자씩 출력

👉 `__asm__ __volatile__("wfi");`

- 무한 루프 블록의 코드
- `wfi` : Wait For Interrupt → CPU를 저전력 상태로 전환
- 여기서는 커널 종료 방지 용도

## 📌 **트랩 핸들러 (Trap Handler)**

✅ 개념

- 예외(Exception)나 시스템콜이 발생했을 때 CPU가 자동으로 호출하는 특정 함수
- 즉, 예외 상황을 처리하는 중앙 함수

✅ 예시

- OpenSBI의 `sbi_trap_handler()` 함수는 커널에서 `ecall` 명령을 실행했을 때 호출됨
- 여기서 어떤 SBI 기능인지 판단하고, 알맞은 처리 함수로 연결

## 📌 **`mtvec` 레지스터 (Machine Trap-Vector Register)**

✅ 개념

- `mtvec` 레지스터 (Machine Trap-Vector Register)
- RISC-V에서 트랩(예외, 인터럽트)이 발생했을 때 점프할 주소를 저장하는 레지스터
- M mode에서만 사용 가능한 레지스터

✅ 예시

- OpenSBI는 부팅 시 `mtvec`에 트랩 핸들러 주소를 등록
- 이후 커널에서 `ecall`이 실행되면 → CPU는 `mtvec`이 가리키는 M-mode 코드로 점프

## 📌 **UART 드라이버**

✅ 개념

- UART HW 장치(시리얼 통신용 칩)을 제어하기 위한 SW
- 특정 주소로 데이터를 쓰면 문자 1개가 전송됨

✅ 예시

- OpenSBI의 `uart8250_putchar()` 함수는 UART 레지스터에 1바이트 데이터를 써서 문자 출력

## 📌 **UART8250 에뮬레이션 장치**

✅ 개념

- QEMU가 제공하는 가상의 UART 하드웨어 장치
- 실제 UART 칩(8250 모델)을 소프트웨어로 구현해 시리얼 입출력을 흉내냄

✅ 예시

- 커널이 `sbi_console_putchar()`를 호출하면
- → OpenSBI가 UART8250 장치를 통해 QEMU로 문자 1개 전송
- → 터미널에 출력됨

## 📌 `printf()`

✅ 전체 흐름 요약

1. 포맷 문자열(fmt)을 한 글자씩 읽음

2. `%` 문자가 나오면 → 다음 문자로 포맷 종류(`s,d,x`) 판단

3. 포맷 종류에 따라 `va_arg`로 인자 추출 후 출력

4. 일반 문자면 그대로 출력

5. 모든 문자 출력 완료 후 `va_end`로 종료

👉 예시 : 포맷 문자열과 가변 인자

```c
printf("1 + %d = %d", 2, 3);
```

- 포맷 문자열 : `"1 + %d = %d"`
- 가변 인자 : `2, 3`

👉  `va_list`, `va_start`, `va_arg`, `va_end`

- 가변 인자(variadic arguments)를 처리하는 C 매크로들

✅ 코드 상세 설명

```c
#include "common.h"

void putchar(char ch);
```

👉 `putchar` : 문자 출력용 함수

---

```c
void printf(const char *fmt, ...) {
    va_list vargs;
    va_start(vargs, fmt);
```

👉 `*fmt`

- 포맷 문자열
- 예를 들면, `printf(”1 + %d = %d”, 2, 3);` 에서 `”1 + %d = %d”` 가 `fmt`

👉 `...` (세 개의 점)

- 가변 인자
- 즉, 포맷 문자열 뒤에 몇 개의 인자가 와도 상관 없음

👉  `va_list vargs;`

- 가변 인자에 접근하려면 도구가 필요함
- `va_list`는 가변 인자를 처리하기 위한 포인터 타입 같은 변수

👉 `va_start(vargs, fmt);`

- 이 함수는 `vargs`를 `fmt` 다음에 오는 가변 인자들의 시작점으로 초기화
- 이렇게 하면, `vargs`를 통해 `fmt` 뒤에 오는 `2`, `3` 등을 꺼내서 사용할 수 있음

---

```c
while (*fmt) {
    if (*fmt == '%') {
        fmt++; // '%'은 건너뜀 -> fmt에는 % 뒤에 나오는 문자가 참조됨
```

- 포맷 문자열을 한 글자씩 읽음
- `%`가 나오면 포맷 명령 처리 시작

---

`✅ %` 이후 문자에 따라 동작 분기

```c
        switch (*fmt) {
            case '\0':
                putchar('%');
                goto end;

```

- 문자열 끝에 `%`만 덩그러니 있으면 그냥 `%` 출력

```c
            case '%':
                putchar('%');
                break;

```

- `%%`는 `%` 하나 출력

---

### `%s`: 문자열 출력

```c
            case 's': {
                const char *s = va_arg(vargs, const char *);
                while (*s) {
                    putchar(*s);
                    s++;
                }
                break;
            }
```

- 가변 인자에서 문자열 포인터 가져와 한 글자씩 출력
- 문자열 끝에 도달하면(null 문자, `\0`) while문 종료

---

### `%d`: 10진수 정수 출력

```c
            case 'd': {
		            // 1. 인자 꺼내기
                int value = va_arg(vargs, int);

                // 2. 음수 처리
                unsigned magnitude = value;
                if (value < 0) {
                    putchar('-'); // - 출력
                    magnitude = -magnitude; // 자릿수 계산을 위해 음수의 절댓값 필요
                }

								// 3. 가장 큰 자릿수 계산
                unsigned divisor = 1;
                while (magnitude / divisor > 9)
                    divisor *= 10;

								// 4. 한 자리씩 출력
                while (divisor > 0) {
                    putchar('0' + magnitude / divisor); // 숫자를 문자로 출력
                    magnitude %= divisor;
                    divisor /= 10;
                }

                break;
            }
```

### `%x`: 16진수 정수 출력

```c
            case 'x': {
                unsigned value = va_arg(vargs, unsigned);
                for (int i = 7; i >= 0; i--) {
                    unsigned nibble = (value >> (i * 4)) & 0xf;
                    putchar("0123456789abcdef"[nibble]);
                }
            }

```

### 일반 문자 출력

```c
    } else {
        putchar(*fmt);
    }
```

- `%`가 아닌 일반 문자면 그대로 출력

---

### 종료 처리

```c
    fmt++;
}

end:
    va_end(vargs);
}

```

- 다음 문자로 이동
- 끝나면 `va_end`로 가변 인자 종료
