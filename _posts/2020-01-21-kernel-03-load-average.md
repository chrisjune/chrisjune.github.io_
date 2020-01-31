---
layout: post-sidebar
date: 2020-01-31
title: "Linux Command 3 - Load Average와 시스템 부하"
categories: linux
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 15
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
**Load Average와 시스템 부하**

## 3.1 Load Average의 정의
* `man proc | grep loadavg`
    * R(Running)과 D(Uninterruptible waiting) 상태인 프로세스의 평균개수
    * 1분, 5분, 15분마다 위의 상태인 프로세스의 평균값
* `cat /proc/loadavg`
    * 이 값은 `uptime` 명령어의 결과와 동일하다
    * 첫 세개의 값은 1,5,15분 동안의 실행중이거나 대기중인 프로세스 수
    * 네번째 값은 실행가능한 커널 스케줄링 엔티티(프로세스, 스레드)의 수 / 시스템에 존재하는 커널 스케줄링 엔티티의 수
    * 다섯번째 값은 가장 최근에 생성된 프로세스의 PID
    ```sh
    > cat /proc/loadavg
    0.52 0.58 0.59 1/5 312
    ```
* `loadaverage와 CPU 코어 관계`
    * Load average 값은 실행중이거나 대기중인 프로세스의 개수이기 때문에
    * CPU코어가 많을 수록 Load average가 높더라도 많은 프로세스가 실행중이므로 큰 걱정은 없지만,
    * CPU코어가 낮을 수록 대기중인 프로세스가 많다는 의미기 때문에 환경에 따라 상대적으로 해석할수있다.
*  `strace -s 65535 -f -t -o uptime_dump uptime`
    ```sh
   openat(AT_FDCWD, "/proc/loadavg", O_RDONLY) = 4
    lseek(4, 0, SEEK_SET)                   = 0
    read(4, "0.52 0.58 0.59 1/6 441\n", 8191) = 23
    fstat(1, {st_mode=S_IFCHR|0660, st_rdev=makedev(4, 1), ...}) = 0
    22:08:42 up 29 min,  0 users,  load average: 0.52, 0.58, 0.59
    ```
   * uptime 명령어는 /proc/loadavg 파일을 그대로 보여주는 명령어이다.
## 3.2 Load Average 계산과정
* 내부 구현은 여러 함수를 통해서 감추어져 복잡해보이지만,
* 실제로는 Timer가 돌면서 실행중, 대기중인 프로세스 수를 수집하는 역할을 하고,
* Timer가 수집된 값들을 1, 5, 15분마다의 평균값을 계산하여 넣어준다.
## 3.3 CPU Bound vs I/O Bound
* 높은 Load Average는 CPU에 대한 부하 뿐만 아니라, I/O의 병목상황을 나타낼 수 있다.
    * nr_running, nr_uninterruptible
    ```python
    # CPU bound
    test = 0
    while True:
        test += 1
    ```
    ```python
    # I/O bound
    while True:
        f = open('./io_test.txt', 'w')
        f.write('test')
        f.close()
    ```
    * Load Average는 높아지지만 원인은 다르기 때문에 부하를 줄이기 위한 방법도 달라진다.
## 3.4 vmstat을 부하의 정체 확인하기
* `vmstat`
    * Virtual memory의 통계값을 출력해준다
    * Procs
        * r: 실행중인 프로세스의 수
        * b: IO대기중인 프로세스의 수
    * Memory
        * swpd: 사용중인 가상메모리
        * free: 상태가 idle인 메모리
        * buff: buffer로 사용중인 메모리
        * cache: cache로 사용중인 메모리
        * inact: inactive한 메모리
        * active: active한 메모리
    * Swap
        * si: 디스크로부터 swap된 메모리
        * so: 디스크에 swap된 메모리
    * IO
        * bi: block device에서 받은 블록들
        * bo: block device에 보낸 블록들
    * System
        * in: 초당 interrupts의 수
        * cs: 초당 context switches 수
    * CPU
        * 항목들의 총 합은 100%가 됩니다
        * us: non-kernel 코드를 실행한 시간 (user time, nice time)
        * sy: kernel 코드를 실행한 시간 (system time)
        * id: idle 시간
        * wa: IO 대기시간
            * Linux 2.54.1버전 이하에서는 id와 wa가 합쳐져서 표시되었다
        * st: 가상머신에 사용된 시간
    * r/b: 1~2의 값이더라도 지속적으로 부하를 일으키는 프로세스가 존재한다는 의미이기 때문에 확인을 해야한다
        * 특히 b값이 높다면 I/O처리에 문제가 있는지 확인해야한다
## 3.5 Load Average가 시스템에 끼치는 영향
* CPU bound vs I/O Bound
    * CPU bound 작업이 많으면, CPU연산이 필요한 작업이 추가될 때 시간이 오래걸린 다는 것을 의미한다
    * 왜냐하면 CPU자원에 대한 경합으로 인하여 응답속도가 늦어지기 때문이다.
    * I/O bound 작업은, D상태에 빠지기 때문에 CPU 경합이 위보다 훨씬 덜하기 때문에 보다 빠른 응답을 한다.
* `/proc/sched_debug`
    * vmstat 툴보다 더욱 자세한 정보를 제공한다
    * nr_running, runnable tasks항목에서 CPU에 할당된 프로세스의 수와 PID정보 확인을 할 수 있다.
