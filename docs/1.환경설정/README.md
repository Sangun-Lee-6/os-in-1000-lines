# 1. 환경 설정

## **1. 개발 툴 설치**

> 🖥️ 개발 환경 : macOS 기준

1. Homebrew 설치
2. 터미널에서 다음 명령어를 실행하여 필요한 프로그램들을 설치

```
brew install llvm lld qemu
```

1. OpenSBI 펌웨어 다운로드

```
curl -LO https://github.com/qemu/qemu/raw/v8.0.4/pc-bios/opensbi-riscv32-generic-fw_dynamic.bin
```

⚠️ 주의

- QEMU를 실행할 때, `opensbi-riscv32-generic-fw_dynamic.bin` 파일이 현재 디렉터리에 반드시 있어야 합니다. 파일이 없으면 아래와 같은 오류가 발생

```
qemu-system-riscv32: Unable to load the RISC-V firmware "opensbi-riscv32-generic-fw_dynamic.bin"
```

## 2. **Git 레포지토리 설정**

🤖 `.gitignore`

```
/disk/*
!/disk/.gitkeep
*.map
*.tar
*.o
*.elf
*.bin
*.log
*.pcap
```
