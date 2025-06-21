# chapter-1-keywords

<br>

## 📌 Homebrew

✅ 개념 : macOS 패키지 관리자

- 패키지를 터미널에서 간편하게 설치하고 관리할 수 있음

✅ 예시

- 이번 프로젝트에서는 QEMU, LLVM, LLD 등의 툴을 `brew install` 명령어로 간편하게 설치 가능

<br>

## 📌 LLVM

✅ **개념**

- C, C++, Rust 등을 위한 모듈형 컴파일러 인프라
- 플랫폼 독립적인 중간 표현(IR)을 사용하여 코드 분석 및 최적화에 유리
- `clang`, `rustc`, `swiftc` 등 여러 언어의 컴파일러가 LLVM 위에 구현됨

✅ 예시

- OS 프로젝트에서 C 코드를 컴파일하기 위해 `clang`을 사용했으며, 이는 LLVM 기반
- Rust의 `rustc`도 내부적으로 LLVM을 통해 머신 코드 생성

<br>

## 📌 LLD

✅ 개념

- LLVM 프로젝트의 링커

✅ 예시

- OS 프로젝트에서 컴파일된 목적파일을 실행 파일(`.elf`)로 연결
- `.elf`는 QEMU가 실행할 커널 바이너리 파일

<br>

## 📌 QEMU

✅ 개념

- HW 가상화 시뮬레이터
- 커널을 작성하면, 이를 실행할 HW가 필요함 → QEMU가 이 역할을 해줌
- RISC-V, ARM, x86 등 다양한 CPU 아키텍처를 지원

✅ 예시

- OS 프로젝트에서 커널 생성(.elf) → 이 파일을 실행할 가상의 RISC-V 컴퓨터

<br>

## 📌 **OpenSBI**

✅ 개념

- RISC-V 시스템의 펌웨어
- RISC-V HW를 초기화하고 커널에 제어권을 넘겨줌
- PC의 BIOS/UEFI 역할
- 커널이 HW에 접근할 수 있는 시스템 인터페이스(System Binary Interface) 제공

✅ 예시

- QEMU가 실행되면(즉, 컴퓨터의 전원을 ON) OpenSBI가 가장 먼저 실행됨
- HW 설정 후 `kernel.elf`에 제어권을 넘김

👉 PC의 BIOS/UEFI

- PC 전원을 켰을 때 가장 먼저 실행되는 프로그램
- HW 초기화 및 부트 로더 실행
