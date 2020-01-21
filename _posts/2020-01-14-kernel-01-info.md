---
layout: post-sidebar
date: 2020-01-14
title: "Linux Command 1 - 시스템 구성 정보 확인하기"
categories: linux
author_name : Tomas
author_url : /author/tom
author_avatar: tom
show_avatar : false
read_time : 10
feature_image: feature-wolf
show_related_posts: true
square_related: recommend-fire
---
**리눅스 시스템에서 가장 중요한 커널의 구성정보를 확인해본다**

## 1.1 커널 정보 확인하기
* `uname`
    * 커널 이름 출력
    ```sh
    Linux
    ```
* `uname -a`
    * 커널 버전 상세정보 출력
    ```sh
    root@bea55500dc5a:/# uname -a
    Linux bea55500dc5a 4.9.184-linuxkit #1 SMP Tue Jul 2 22:58:16 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
    ```
* `dmseg`
    * 커널의 디버그메시지 출력
    * 부팅메시지, 운영메시지 출력
    ```sh
    root@bea55500dc5a:/# dmesg
    [    0.000000] Linux version 4.9.184-linuxkit (root@a8c33e955a82) (gcc version 8.3.0 (Alpine 8.3.0) ) #1 SMP Tue Jul 2 22:58:16 UTC 2019
    [    0.000000] Command line: BOOT_IMAGE=/boot/kernel console=ttyS0 console=ttyS1 page_poison=1 vsyscall=emulate panic=1 root=/dev/sr0 text
    [    0.000000] x86/fpu: Supporting XSAVE feature 0x001: 'x87 floating point registers'
    [    0.000000] x86/fpu: Supporting XSAVE feature 0x002: 'SSE registers'
    [    0.000000] x86/fpu: Supporting XSAVE feature 0x004: 'AVX registers'
    [    0.000000] x86/fpu: xstate_offset[2]:  576, xstate_sizes[2]:  256
    [    0.000000] x86/fpu: Enabled xstate features 0x7, context size is 832 bytes, using 'standard' format.
    [    0.000000] e820: BIOS-provided physical RAM map:
    ```
    * /proc 파일 시스템에서 확인가능
    * 메모리 정보는 Free의 값과 다른데, 이유는 부팅과정에서 사용하고 반환하기 때문
    * intel_idel.max_cstate, crashkernel, biosdevname, console 파라미터 눈도장
    * crashkernel: 커널 패닉시 crashkernel을 로딩하고 디버깅정보를 저장하여, 문제 원인 분석이 가능해짐
    * intel_idle.max_cstate: IDLE CPU를 잠자기모드로 전환하는 기능을 끄는 설정
* `cat /boot/config-'uname-r'`
    * 커널 컴파일 정보
    * 커널 기능중 컴파일 옵션에 포함되어하는 것이 있기 때문에 체크필요
## 1.2 CPU정보 확인하기
* `dmidecode`
    * 하드웨어 정보 출력
    * 중요 키워드
        * bios, system, baseboard, chassis, processor, memory, cache, connector, slot
        ```sh
        dmidecode -t bios
        dmidecode -t system
        dmidecode -t process
        ```
        * 소켓: 물리적인 CPU의 개수
        * 코어: 물리적인 CPU안에 컴퓨팅 코어의 개수
        * 2개 소켓, 6개 코어 = 총 12개 코어
* `cat /proc/cpuinfo`
    * 물리적 CPU 정보 확인
    ```sh
    root@bea55500dc5a:/# cat /proc/cpuinfo
    processor	: 0
    vendor_id	: GenuineIntel
    cpu family	: 6
    model		: 158
    model name	: Intel(R) Core(TM) i7-7700HQ CPU @ 2.80GHz
    stepping	: 9
    cpu MHz		: 2800.000
    cache size	: 6144 KB
    ```
* `lscpu`
    * 물리적 CPU 정보 확인
    * NUMA 관련 정보 출력
    ```sh
    root@bea55500dc5a:/# lscpu
    Architecture:        x86_64
    CPU op-mode(s):      32-bit, 64-bit
    Byte Order:          Little Endian
    CPU(s):              2
    On-line CPU(s) list: 0,1
    Thread(s) per core:  1
    Core(s) per socket:  1
    Socket(s):           2
    Vendor ID:           GenuineIntel
    ```
## 1.3 메모리정보 확인하기
* `dmidecode -t memory`
    * 각 메모리 슬롯의 정보를 출력
* `dmidecode -t memory | grep -i size`
    * 각 메모리별 크기 출력
* `free -m`
    * 위 명령어의 메모리합과 free와 같지 않으면 시스템 메모리 인식에 이슈가 있음
    ```sh
    root@bea55500dc5a:/# free -m
                  total        used        free      shared  buff/cache   available
    Mem:           1999         769         297           1         931        1084
    Swap:          1023           0        1023
    ```
## 1.4 디스크 정보 확인하기
* `df -h`
    * 디스크 파티션 정보 출력
    ```sh
    root@bea55500dc5a:/# df -h
    Filesystem      Size  Used Avail Use% Mounted on
    overlay          59G   23G   34G  41% /
    tmpfs            64M     0   64M   0% /dev
    tmpfs          1000M     0 1000M   0% /sys/fs/cgroup
    shm              64M     0   64M   0% /dev/shm
    /dev/sda1        59G   23G   34G  41% /etc/hosts
    ```
    * hda: IDE 하드 타입
    * sda: SATA, SCSI, SAS 하드 타입
    * vda: 가상서버 디스크타입
* `smartctl`
    * 물리적인 디스크 정보 출력
    * `-a`: 논리적 파티션
    * `-d`: RAID 컨트롤러 드라이버 정보(lsmod 로 확인), 디스크베이번호
    ```sh
    smartctl -a /dev/sda -d cciss,0
    ```
## 1.5 네크워크 정보 확인하기
* `lspci`
    * 네트워크 카드 정보 출력
* `ethtool eth0`
    * 연결 상태를 출력 
    * speed: 현재 연결속도
    * link detected: 연결 정상 여부
* `ethtool -g eth0` 
    * Ring Buffer의 크기를 확인
    * 패킷을 버퍼에 저장 후 커널로 복사함, 따라서 크키조절이 중요
    * maxinum과 current값이 동일하도록 세팅필요
* `ethtool -G eth0 rx 255`
    * Ring Buffer의 값을 설정
* `ethtool -k eth0`
    * 네트워크 성능 확인
* `ethtool -K eth0 generic-receive-offload on`
    * 네트워크 성능의 값 설정
    * generic-receive-offload: tcp-offload기능, 패킷 분할 작업을 CPU가 아니라 네트워크 카드가 처리
    * 대여폭이 넓은 네트워크에서는 이슈가 되므로 off 한다
* `ethtool -i eth0`
    * 네트워크 카드 커널 드라이버 정보 출력
    * 드라이버 버전, 펌웨어 정보 확인가능


## After study
* CPU
    * C-state
        * CPU의 상태를 나타내며, G0(ON) -> G1(MID) -> G2(OFF)
        * G0 안에서 C0~C7...순으로 Active한 상태에서 조금씩 멀어진다
    * Socket -> Core
        * 코어는 각 소켓마다 존재하는 컴퓨팅 코어 개수 
        * 코어는 Intel에서 지원하는 하이퍼스레드 기술을 활용하여 1코어를 2코어 처럼 활용할 수 있도록
        * 효율을 높이는 기술을 사용한다.
* Network
    * RingBuffer
        * 네트워크 카드가 패킷을 받으면 바로 저장하는 저장소
        * 이후에 소켓 버퍼로 담기기 때문에, Ring buffer가 병목의 지점이 될 수 있다
        * 네트워크 속도 최적화를 위해서는, 버퍼의 Max값과 Current값을 같게해주어야 한다.
        * tcp-offload: 네트워크 카드가 직적 패킷을 분해하는 작업을 하는데, off해주어야 패킷 유실을 막을 수 있다.

