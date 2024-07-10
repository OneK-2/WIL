# Introduction

## OSTEP: Operating Systems - Three Easy Pieces

### 1. Virtualization (가상화)

- CPU: Process, CPU Scheduling...
- Memory: Address, Segmentation, Paging...

### 2. Concurrency (병행성)

- Lock, Thread, Semaphores...

### 3. Persistence (영속성)

- FileSystem, Disk...

<br>

## Layered structure of a computer system

![LayeredStructure](https://github.com/OneK-2/WIL/assets/85729858/aef6fe2c-7798-4857-95b6-8dfe0bf96306)

|       구조       |                    예시                     |
| :--------------: | :-----------------------------------------: |
|       User       |                    user1                    |
|   Application    | compiler, asembler, text editor, Ms Word... |
| Operating System |                windows10 pro                |
|     Hardware     |        CPU, DRAM, Disk, Terminal...         |

<br>

## 프로그램 실행 시 발생하는 일

### 1. Fetch and Execute (핵심)

![image](https://github.com/OneK-2/WIL/assets/85729858/564de729-26bc-4533-a1d6-0fc34dca17a3)

CPU 내부에 레지스터들이 존재한다.

- PC: 다음에 수행해야할 명령어 위치 가르킨다.
- IR: fetch한 Instruction이 들어가 있는 위치

```
// Data 영역
int a = 3;
int b = 4;
int c;

// Instruction 영역
c = a + b;

//구체적으로는 3단계
movl a, %eax
addl b, %eax
movl %eax, c
```

<br>

![image](https://github.com/OneK-2/WIL/assets/85729858/34a4c63d-7bbc-4904-9e3f-4c090b9335ec)
AC: 범용 레지스터

#### Step1

- Fetch 작업
- PC가 가르키는 300번지에서 명령어를 IR로 가져온다.
- IR에서 해석, => 940번지에서 데이터를 Load해라

#### Step2

- Excute 작업
- 940번지에서 3을 AC로 로드함
- PC는 다음 수행할 위치인 301로 변경

<br>
위와 같은 작업이 반복된다.

### 2. 추가적인 수 많은 작업들

![image](https://github.com/OneK-2/WIL/assets/85729858/0ee59782-c566-4d60-8e0f-d77f8c77b460)

- Loading : Disk에서 Memory로 load
- Memory management : 메모리 할당이 필요
- Scheduling : 프로그램이 동시에 여러개 수행 됨.
- context switching
- I/O processing
- file management...

<br>

## OS의 정의

### Resource manager (자원 관리자)

> **물리적, 추상적 자원을 관리한다.**

- **Physical Resource**: CPU, DRAM, Disk, Flash, KBD, Network...
- **Virtual Resource**: Process, Thread, Virtual memory, Page, File, Directory, Driver, Protocol, Access control, Security...

### Virtualization (Abstraction)

> **물리적 자원을 추상화한다.**

- CPU -> Process, Thread
- DRAM -> Virtual memory, Page
- Disk -> File, Directory
- KBD(디바이스) -> Driver
- Network -> Protocol

![image](https://github.com/OneK-2/WIL/assets/85729858/e0d44ddb-f44a-4aeb-82db-5d9353096c91)

### System call

> **OS가 사용자에게 제공하는 인터페이스(API) 집합**

<br>

![image](https://github.com/OneK-2/WIL/assets/85729858/be9daffd-799c-4771-957e-8911a9603c12)

![image](https://github.com/OneK-2/WIL/assets/85729858/ed4df736-3f4d-4701-9c2e-643d45a6f4ce)

사용자가 open()을 하려면, 실제로는 커널에서 수행해야함.
<br>
시스템콜은 유저모드에서 커널모드로 모드스위치를 발생시킨다.

## Virtualizing CPU

> **CPU 가상화란?**

실제 가지고 있는 CPU보다 많이 가지고 있는 것처럼 느끼게 한다. <br>
하나의 물리적인 CPU를 여러 개의 CPU로 보이게 하는 것이다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <assert.h>
#include "common.h"

int main(int argc, char *argv[])
{
    if (argc != 2) {
        fprintf(stderr, "usage: cpu <string>\n");
        exit(1);
    }
    char *str = argv[1];
    while (1) {
    Spin(1); // 1초동안 기다리는 함수
    printf("%s\n", str);
    }
    return 0;
}

```

**위 코드에 대한 실행 결과**

```
prompt> ./cpu A & ./cpu B & ./cpu C & ./cpu D &
[1] 7353
[2] 7354
[3] 7355
[4] 7356
A
B
D
C
A
B
D
C
A
```

> **결론: 모든 프로세스들이 자신의 CPU를 가지고 있는 것처럼 추상화를 제공한다.**

<br>

## Virtualizing Memory

> **메모리 가상화?**

물리적으로 메모리는 하나지만 프로세스들마다 마치 자기가 독립적인 메모리를 가지고 있는 것처럼 가상화시킨다.

```c
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include "common.h"

int main(int argc, char *argv[]) {
    if (argc != 2) {
	fprintf(stderr, "usage: mem <value>\n");
	exit(1);
    }
    int *p;
    p = malloc(sizeof(int));
    assert(p != NULL); // 조건이 만족하면 계속 수행
    printf("(%d) addr pointed to by p: %p\n", (int) getpid(), p);
    *p = atoi(argv[1]); // assign value to addr stored in p
    while (1) {
	Spin(1);
	*p = *p + 1;
	printf("(%d) value of p: %d\n", getpid(), *p);
    }
    return 0;
}
```

**위 코드에 대한 실행 결과**

```sh
prompt> ./mem & ./mem &
[1] 24113
[2] 24114
(24113) address pointed to by p: 0x200000
(24114) address pointed to by p: 0x200000
(24113) p: 1
(24114) p: 1
(24114) p: 2
(24113) p: 2
(24113) p: 3
(24114) p: 3
(24113) p: 4
(24114) p: 4


```

-> 주소는 같지만 독립적

<br>

## Concurrency

> **병행성 문제란?**

같은 데이터를 동시에 접근하려고 할 때 발생하는 문제

```c
#include <stdio.h>
#include <stdlib.h>
#include "common.h"
#include "common_threads.h"

volatile int counter = 0;
int loops;

void *worker(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
	counter++;
    }
    return NULL;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
	fprintf(stderr, "usage: threads <loops>\n");
	exit(1);
    }
    loops = atoi(argv[1]);
    pthread_t p1, p2;
    printf("Initial value : %d\n", counter);

    Pthread_create(&p1, NULL, worker, NULL);
    Pthread_create(&p2, NULL, worker, NULL);
    Pthread_join(p1, NULL);
    Pthread_join(p2, NULL);
    printf("Final value   : %d\n", counter);
    return 0;
}

```

```
prompt> ./threads 100000
Initial value : 0
Final value : 143012 // huh??
prompt> ./threads 100000
Initial value : 0
Final value : 137298 // what the??
```

실행횟수가 커지면 원자성이 지켜지지 않는다.

<br>

## Persistence

> **영속성?**

- 사용자는 데이터를 영구적으로 저장하고 싶어 한다.
- DRAM은 휘발성이므로, 스토리지에 저장해야함.

  <br>

- **DRAM**
  - 용량 ↓
  - 속도 ↑
  - Byte 단위 (CPU가 직접 접근 가능)
  - Volatile
- **Disk**
  - 용량 ↑
  - 속도 ↓
  - Sector 단위 (CPU가 직접 접근 불가능)
  - Non-volatile

> **결론: 사용자로 하여금, 데이터가 항상 신뢰성 있고, 휘발되지 않게 관리되어 있다고 생각하게 한다.**

<br>

### Reference

- W. Stalling, “Operating Systems: Internals and Design Principles”
- A. Silberschatz, “Operating system Concept”
