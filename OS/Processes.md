# Processes

## Process

### Process의 정의

| **CPU와 메모리를 가지고, 실행중인 프로그램**

프로그램은 디스크상에 존재하는, 수동적인 생명이 없는 존재 <br>
프로그램이 수행이 되면 프로세스가 된다.

### 어떻게 CPU를 가상화?

| **시분할 시스템 (Time Sharing System)**

#### 1. Mechanism

문맥교환: 한 프로세스의 수행을 멈추고, 다른 프로세스를 수행 시킨다.

#### 2. Policy

스케줄링 정책: 수행중인 프로세스 중 어떤것을 먼저 수행시킬 것인가에 대한 정책

### Process structure

| **프로세스는 크게 3가지 부분으로 구분**

1. **CPU**

   어떤 레지스터들이 어떤 값을 유지하는지 관리

2. **Memory**

   - **Text**
   - **Data**
   - **Stack**
   - **Heap**

![image](https://github.com/user-attachments/assets/16299579-3c05-4c80-ab73-2e6e5abec2e7)

3. I/O information

## Process Execution

![image](https://github.com/user-attachments/assets/540b56ce-b19a-41f3-905b-2f7334789e6b)

### 1.Load

- 코드와 정적 데이터를 주소 공간에 올린다.
- 실행가능한 포맷에 맞게 올린다.
  - Linux -> ELF (Executable Linking Format)
  - Window -> PE (Portable Executable)

### 2. Dynamic allocation

- Stack
- 파라미터 초기화
- Heap (필요하다면)

### 3. Initialization

- 파일 디스크립터 (0, 1, 2)
  - 어떤 프로세스가 I/O를 하기 위해서 미리 오픈해둔 디스크립터가 있다.
  - 0: stdin
  - 1: stdout
  - 2: stderror
- I/O or signal

### 4. Jump to the entry point: main()

## Process States

![image](https://github.com/user-attachments/assets/caccc10b-ac38-4cbc-8e8c-7c3b62b67a3d)

### State (상태)

#### 1. new (created, embryo)

- 새로 프로세스가 만들어짐

#### 2. ready

- 스케줄링의 대상이 되는 상태.

#### 3. running

- CPU에 올라간 상태

#### 4. waiting (blocked)

- 사건을 기다려야 할 상태

#### 5. terminated (zombie)

- 수행이 끝난 상태

| **상태가 2개 더 있을수 있다.**

- 메모리상에 있던 프로세스가 메모리가 부족해지면, 공간 확보를 위해 자주 사용되지 않는 프로세스를 디스크상의 스왑공간으로 내릴수 있다. -> suspend

#### 6.suspend ready

#### 7.suspend wait

### Transition (전이)

#### 1. admitted

- new -> ready
- 새로 프로세스가 만들어지면 바로 수행이 될 수는 없다.
- 로딩, 동적할당 등 스케줄링 할 준비가 되어있으면 전이

#### 2. dispatch(schedule)

- ready -> run <br>
- 수행가능한 상태에서 실제 cpu를 잡고 수행하는 상태

#### 3. timeout(preemptive, descheduled)

- run -> ready
- 할당된 시간을 다 사용함.

#### 4. wait (sleep)

- run -> wait
- 이벤트를 기다리게 한다.

#### 5. wakeup

- 기다리던 사건이 끝남

#### 6. exit

#### 7. suspend

- 스왑 공간으로 프로세스 내림
- running 상태는 suspend 될 수 없다.
- 주로 ready 상태에 있는 프로세스를 내린다

#### 8. resume

- 스왑공간에서 다시 메모리로 돌아감

### Example

![image](https://github.com/user-attachments/assets/a244732a-c6e5-46eb-9092-917b7336a2f7)

1. Process0이 수행되다가 I/O를 수행함. I/O는 시간이 오래 걸리는 작업이기 때문에, Blocked 상태로 전이
2. Ready 상태 중 하나인 Process1을 run상태로 전이
3. 기다리던 사건인 I/O가 끝나면 Process0이 Ready상태로 돌아감
4. Process1이 종료되면 다시 Process0이 실행됨

## Data Structure

### PCB (Process Control Block)

| **프로세스와 관련된 정보들을 관리**

- **Process state**
- **Process ID (pid)**
  - 프로세스의 고유한 id
- **Program counter, CPU registers**
  - 문맥교환, 아키텍쳐 확인 시 사용
- **CPU 스케줄링 정보**
- **Memory 관련 정보**
- **오픈한 파일**
- **I/O 관련 정보**
- **Accounting 정보**
  - cpu, memory 얼마나 사용했는지

OS에서 프로세스를 관리하기 위해서 PCB라는 객체를 사용한다. <br>
PCB는 OS마다 다른 이름으로 구현될 수 있다.

Ex) Linux에서는 PCB가 task structure라는 이름으로 구현되어있음.

원래 Process는 task와 thread들의 집합이다.
<br>
최근에는 Process와 task는 같은 개념으로 사용한다.

# Process API

## system call

### fork()

- **새로운 프로세스 생성: parent, child**
- **리턴 value가 2개**
  - 부모는 항상 0보다 큰값을 리턴값으로 받음
  - 자식은 0을 리턴값으로 받음
- **Non-determinism**
  - 부모가 먼저 수행될지, 자식이 먼저 수행될지 알수없다.

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else {
        // parent goes down this path (original process)
        printf("hello, I am parent of %d (pid:%d)\n",
	       rc, (int) getpid());
    }
    return 0;
}
```

| 제어의 흐름이 둘로 나누어진다. 새로운 프로세스를 생성했기 때문.

### wait()

- **이 API를 호출한 프로세스를 멈추는 역할**
- **deterministic하게 해준다.** -> **synchronization**
  - 순서 관계를 정하는 것

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
	sleep(1);
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
	       rc, wc, (int) getpid());
    }
    return 0;
}
```

| **자식이 수행이 완료될때까지 멈춘다. 자식이 먼저 수행되는것을 보장한다.**

### exec()

- **never return**
- **6 variations**
  - execve -> 이거만 시스템콜이고 나머지는 라이브러리 형태
  - execl
  - execlp
  - execle
  - execv
  - execvp

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
        // child (new process)
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc");   // program: "wc" (word count)
        myargs[1] = strdup("p3.c"); // argument: file to count
        myargs[2] = NULL;           // marks end of array
        execvp(myargs[0], myargs);  // runs word count
        printf("this shouldn't print out");
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
	       rc, wc, (int) getpid());
    }
    return 0;
}
```

### 초기 UNIX에서 fork()와 exec()를 분리한 이유

| **확장성을 제공하기 위해서**

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <assert.h>
#include <sys/wait.h>

int
main(int argc, char *argv[])
{
    int rc = fork();
    if (rc < 0) {
        // fork failed; exit
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) {
	// child: redirect standard output to a file
	close(STDOUT_FILENO);
	open("./p4.output", O_CREAT|O_WRONLY|O_TRUNC, S_IRWXU);

	// now exec "wc"...
        char *myargs[3];
        myargs[0] = strdup("wc");   // program: "wc" (word count)
        myargs[1] = strdup("p4.c"); // argument: file to count
        myargs[2] = NULL;           // marks end of array
        execvp(myargs[0], myargs);  // runs word count
    } else {
        // parent goes down this path (original process)
        int wc = wait(NULL);
	assert(wc >= 0);
    }
    return 0;
}

```

최근에는 UNIX계열에서 fork(), exec() 합친 system사용

### 기타 APIs

- **getpid()**
- **kill()**
- **signal()**
- **스케줄링 관련...**
