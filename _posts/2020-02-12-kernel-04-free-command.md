---
layout: post-sidebar
date: 2020-02-12
title: "Linux Command 4 - free 명령이 숨기고 있는 것들
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
**free 명령이 숨기고 있는 것들**

* CPU는 연산과정에 필요한 리소스라면, 메모리는 연산에 필요한 공간을 제공해주는 리소스이다.
* 메모리에 연산에 필요한 함수와 변수를 담아둔다.
## 4.1 메모리 사용량 확인하기
* `free`
    * 메모리의 전체적인 현황을 가장 빠르게 확인
    ```sh
    root@139ed5b27ba1:/# free -m
                  total        used        free      shared  buff/cache   available
    Mem:           1999         297        1110           0         591        1548
    Swap:          1023           0        1023
    ```
    * 옵션, -b:byte, -k:KB, -m:MB, -g:GB 단위 출력
    * total: 전체 메모리양
    * used: 시스템에서 사용하고 있는 메모리양
    * free: 시스템에서 사용하고 있지 않는 메모리양
    * shared: 프로세스 사이에서 공유하는 메모리양
    * buffer: dentry, inode와 같이 커널에서 시스템향상을 위하여 사용하는 영역
    * cache: Page cache로서 블럭디바이스와의 통신을 빠르게 진행하기 위한 영역
    * swap: 메모리가 부족할 때, 오랫동안 참조되지 않아 잠시 swap영역으로 빠지게된 메모리양
## 4.2 buffers와 cached영역
* 블록 디바이스
    * 커널은 하드디스크와 같은 블록디바이스에서 데이터를 읽거나 저장한다
    * 블록 디바이스 드라이버
        * 데이터를 블록단위로 처리하는 보조기억장치를 다룬다
    * 디바이스 파일
        * 드라이버가 파일시스템을 통해 직접 디바이스에 접근하지 않고, 실제로는 파일 형태로 간접 접근한다
        * /dev/hda, /dev/hdb
    * 커널은 느린 디스크에 대한 요청을 빠르게 하기 위하여 한번 읽은 부분은 메모리에 캐싱영역으로 할당한다
* Page Cache
    * `커널이 데이터 파일을 읽을 때` bio 구조체를 만들고 해당 구조체에 page cache로 할당한 메모리영역을 연결한다
    * bio구조체는 디바이스 드라이버와 통신하여 디스크에서 읽은 데이터를 캐쉬에 담는다
* Buffer Cache
    * `커널이 파일시스템을 관리하기 위한 메타데이터를 읽을 때` _get_bulk와 같은 내부함수를 이용한다.
    * 이후 블록디바이스와 통신하여 가져온 블록 내용을 캐쉬에 담는다
* 시간 흐름에 따른 메모리 변화
    * (가용영역                    )(사용영역)
    * (가용영역             )(CACHE)(사용영역)
    * (가용영역      )(CACHE)(사용영역       )
    * (가용영역)(CACHE)(사용영역             )
    * 프로세스에서 사용하는 영역이 커지게 되면, cache로 사용하는 메모리 영역을 사용영역으로 반환된다
    * 메모리가 더 이상 여유가 없을 때 swap이라는 영역을 사용하고 시스템 성능이 저하된다.
## 4.3 /proc/meminfo
* `cat /proc/meminfo`
    * 자세한 메모리 현황 제공
    ```sh
    root@139ed5b27ba1:/# cat /proc/meminfo/
    cat: /proc/meminfo/: Not a directory
    root@139ed5b27ba1:/# cat /proc/meminfo
    MemTotal:        2047132 kB
    MemFree:         1134908 kB
    MemAvailable:    1586032 kB
    Buffers:          111364 kB
    Cached:           447940 kB
    SwapCached:            0 kB
    Active:           476568 kB
    Inactive:         338920 kB
    Active(anon):     249856 kB
    Inactive(anon):     7172 kB
    Active(file):     226712 kB
    Inactive(file):   331748 kB
    Unevictable:           0 kB
    Mlocked:               0 kB
    SwapTotal:       1048572 kB
    SwapFree:        1048572 kB
    Dirty:                28 kB
    Writeback:             0 kB
    AnonPages:        254212 kB
    Mapped:           137684 kB
    Shmem:               848 kB
    Slab:              70112 kB
    SReclaimable:      48932 kB
    SUnreclaim:        21180 kB
    KernelStack:        6640 kB
    PageTables:         2364 kB
    NFS_Unstable:          0 kB
    Bounce:                0 kB
    WritebackTmp:          0 kB
    CommitLimit:     2072136 kB
    Committed_AS:    2717084 kB
    VmallocTotal:   34359738367 kB
    VmallocUsed:           0 kB
    VmallocChunk:          0 kB
    AnonHugePages:      2048 kB
    ShmemHugePages:        0 kB
    ShmemPmdMapped:        0 kB
    HugePages_Total:       0
    HugePages_Free:        0
    HugePages_Rsvd:        0
    HugePages_Surp:        0
    Hugepagesize:       2048 kB
    DirectMap4k:       26028 kB
    DirectMap2M:     2070528 kB
    DirectMap1G:           0 kB
    ```
    * SwapCached: 메모리 부족으로 swap으로 빠졌다가 다시 메모리로 돌아온 영역
        * 메모리에 있던 부분을 swap영역으로 옮길 때 IO가 발생하기 때문에 성능저하가 발생한다
        * swap영역에서 메모리로 다시 돌아올 때, 또 다시 메모리 부족현상이 발생할 것을 대비하여 swap에서 해당 메모리 내용을 삭제하지 않는다
    * Active(anon): PageCache 영역을 제외한 메모리 영역
        * 주로 프로세스들이 사용하는 메모리 영역
        * 그중에서도 비교적 최근에 참조되어 swap영역으로 이동하지 않을 영역
    * Inactive(anon): 참조된지 오래되어 swap영역으로 이동가능한 영역
    * Active(file): 커널이 IO향상을 위하여 사용하는 Cache영역(buffers, page cache)     
        * 그중에서도 비교적 최근에 참조되어 swap영역으로 이동하지 않을 영역
    * Inactive(file): 참조된지 오래되어 swap영역으로 이동가능한 영역
    * Dirty: 블록 디바이스에 실제로 쓰여질 영역, 일정량이 될 때까지 모았다가 한번에 작업함
    * Swap 대상
        * 메모리를 LRU기반의 리스트로 관리한다
        * 자주 사용되는 영역은 active, 아닌 영역은 inactive list영역이 된다
        * 프로세스가 메모리 할당을 요청하면 해당 메모리의 페이지가 active리스트에 연결된다
    * Active -> Inactive로 이동하는 시점
        * 시간이 지난다고 이동하는 것이 아님
        * 메모리 부족 또는 메모리 할당이 실패하여 커널이 메모리를 해제해야할 메모리를 찾아야할 때 LRU리스트를 살펴본다
    * `sysctl -w vm.min_free_bkytes=6553500`
        * 시스템에서 유지해야할 최소 free 메모리의 양을 설정
        * 이 값을 높게 설정하면 `kswapd`이 실행되면서 active영역에서 오래된 페이지들을 inactive로 옮긴후 메모리를 해제한다
        * 해제된 메모리의 대부분 swap으로 이동한다
## 4.4 Slab 메모리 영역
* 커널 또한 프로세스이기 때문에 메모리를 필요로 한다. Slab이 바로 커널이 사용하는 영역이다.
```sh
Slab:              70112 kB
SReclaimable:      48932 kB
SUnreclaim:        21180 kB
```
* Slab: 커널이 직접 사용하는 영역, dentry cache, inode cache등 정보를 저장한다
* SReclaimable: 말 그대로 재사용될 수 있는 영역, 캐쉬용도로 사용하는 메모리가 포함된다. 따라서 메모리가 부족하면 해제되어 프로세스에 할당가능한영역.
* SUnreclaim: 재사용불가능한 영역, 커널이 현재 사용중이며 해제하여 다른 용도로 사용할 수 없다
* `slabtop -o`
    * slabtop명령어로 현재 커널에서 사용중인 slab의 정보를 살펴볼 수 있다
    ```sh
    root@139ed5b27ba1:/# slabtop -o
    Active / Total Objects (% used)    : 299311 / 310928 (96.3%)
    Active / Total Slabs (% used)      : 16695 / 16706 (99.9%)
    Active / Total Caches (% used)     : 78 / 140 (55.7%)
    Active / Total Size (% used)       : 63626.76K / 66470.81K (95.7%)
    Minimum / Average / Maximum Object : 0.02K / 0.21K / 4096.00K

    OBJS ACTIVE  USE OBJ SIZE  SLABS OBJ/SLAB CACHE SIZE NAME
    92235  92187   0%    0.10K   2365       39      9460K buffer_head
    63819  60149   0%    0.19K   3039       21     12156K dentry
    26240  25789   0%    0.06K    410       64      1640K kmalloc-64
    21576  21295   0%    0.03K    174      124       696K kmalloc-32
    19362  18781   0%    0.57K   2766        7     11064K inode_cache
    18156  18112   0%    0.12K    534       34      2136K kernfs_node_cache
    ```
* 일반 프로세스에서 사용하는 메모리와 커널에서 사용하는 메모리 영역이 다른 이유?
    * 일반 프로세스 메모리 할당자인 버디 시스템은 4KB(4*1024byte) 단위로 메모리를 할당한다.
    * 커널은 이렇게 큰 영역을 받을 필요가 없기 때문에 Slab할당자를 통하여 메모리를 할당받는다
    * (할당영역    (사용영역)) - 이렇게 사용영역이 커지면 메모리 단편화 현상이 생기기 때문에 영역을 달리하여 다른 할당자를 사용한다
    * 메모리 기본단위인 4KB를 할당받아 캐쉬 목적별로 나누어 사용한다.
        * dentry: 디렉터리의 계층관계를 저장
        * inode_cache: 파일의 inode에 대한 정보를 저장
        * 파일에 자주 접근하고 디렉터리의 생성과 삭제가 빈번하면 해당 영역이 높아진다
* slab할당자는 free에서 used로 계산된다. 캐쉬영역이기 때문에 buffer/cache로 포함되지 않는다
* 프로세스들의 메모리 총 사용양이 used와 맞지 않으면 slab메모리 누수일 수 있다.
    
## 4.5 Case Study - Slab 메모리 누수

            
    

        
      
     
