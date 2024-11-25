---
title: bcc
weight: 2
---
### bcc 설치

### execsnoop
- 새 프로세스를 `exec()` syscall로 추적
```
$ sudo execsnoop-bpfcc
PCOMM            PID    RET ARGS
supervise        9660     0 ./run
supervise        9661     0 ./run
mkdir            9662     0 /bin/mkdir -p ./main
run              9663     0 ./run
[...]
```
### opensnoop
- 프로세스의 `open()` syscall 추적
```
$ sudo opensnoop-bpfcc
```
### ext4slower (or btrfs*, xfs*, zfs*)
- IO 입출력 확인에 사용
```
$ sudo ext4slower-bpfcc
```

### biolatency
- 디스크 IO 대기시간의 지연을 측정
```bash
ubuntu@ip-172-31-38-88:~$ sudo biolatency-bpfcc
Tracing block device I/O... Hit Ctrl-C to end.
^C
     usecs               : count     distribution
         0 -> 1          : 0        |                                        |
         2 -> 3          : 0        |                                        |
         4 -> 7          : 0        |                                        |
         8 -> 15         : 0        |                                        |
        16 -> 31         : 0        |                                        |
        32 -> 63         : 0        |                                        |
        64 -> 127        : 0        |                                        |
       128 -> 255        : 22       |                                        |
       256 -> 511        : 1674     |*********                               |
       512 -> 1023       : 7138     |****************************************|
      1024 -> 2047       : 2921     |****************                        |
      2048 -> 4095       : 782      |****                                    |
      4096 -> 8191       : 635      |***                                     |
      8192 -> 16383      : 162      |                                        |
     16384 -> 32767      : 2        |                                        |
```
- 구간 설명
  - 출력의 왼쪽 구간 `usecs`은 특정 I/O 요청의 대기 시간(µs) 범위
  - `count`는 해당 대기 시간 범위 내에서 발생한 I/O 요청의 개수
  - `distribution`은 비율에 따른 시각적 표현
- 주요 구간 해석
  - 128 -> 255 µs: 22개의 요청이 이 범위에서 완료됨.
  - 256 -> 511 µs: 1674개의 요청. 상당한 수의 요청이 이 구간에서 발생.
  - 512 -> 1023 µs: 7138개의 요청으로 가장 많은 비율을 차지함.
  - 1024 -> 2047 µs: 2921개의 요청. 여전히 대기 시간이 상대적으로 적은 편.
  - 2048 -> 4095 µs 및 그 이상: 일부 요청은 더 높은 대기 시간을 가짐.
- 결과 해석
  - 대부분의 I/O 요청은 256 -> 1023 µs 범위에 분포되어 있으며, 이는 비교적 적당한 대기 시간으로 볼 수 있습니다.   
    다만, 4096 µs 이상의 대기 시간을 가진 요청도 존재하여 특정 조건에서 디스크 성능이 제한되고 있을 가능성이 있습니다.
  - 16384 ~ 32767 µs 구간에서 2개 IO 요청 처리하는데 걸린 대기시간이 가장 높음

### biosnoop
- `biosnoop-bpfcc` 출력은 블록 디바이스의 개별 I/O 요청을 추적하여 상세 정보를 제공합니다. 각 요청은 시간, 프로세스 정보, 디스크 장치, 작업 유형, 섹터, 바이트 크기, 지연 시간 등을 포함
```bash
ubuntu@ip-172-31-38-88:~$ sudo biosnoop-bpfcc
TIME(s)     COMM           PID     DISK      T SECTOR     BYTES  LAT(ms)
0.000000    kworker/u30:5  235     xvda      W 10419440   4096      0.76
0.000016    kworker/u30:5  235     xvda      W 12843344   12288     0.75
0.000019    kworker/u30:5  235     xvda      W 16288320   4096      0.75
0.000022    kworker/u30:0  1789    xvda      W 16440280   8192      0.71
0.000024    kworker/u30:0  1789    xvda      W 16440312   8192      0.70
```
- kworker/u30:5 커널의 프로세스ID 235가 xvda 디스크의 10419440 섹터에서 쓰기 요청을 4096 BYTES의 크기로 IO에 보내 0.76ms 뒤에 완료됨
- kworker는 커널 워커 스레드로, 시스템에서 백그라운드 작업을 처리함. 주로 파일 시스템 또는 디스크 캐시 플러시 작업일 수 있음
- 섹터 번호(SECTOR)를 통해 디스크의 특정 부분에 과도한 I/O가 집중되는지 확인할 수 있습니다. 만약 특정 섹터에 집중된 요청이 많다면 디스크의 조각화 문제나 특정 파일 접근이 원인일 수 있음
- xvda는 AWS의 EBS 볼륨
  - gp2/gp3: 일반 SSD. IOPS와 처리량 제한이 있음.
  - io2/io1: 고성능 SSD. 높은 IOPS 제공.
  - st1/sc1: HDD. 대기 시간이 길어질 수 있음.
```
TIME(s)	스크립트 시작 후 경과 시간(초).
COMM	요청을 생성한 커널 또는 사용자 공간 프로세스 이름.
PID	    요청을 생성한 프로세스의 PID(프로세스 ID).
DISK	요청이 발생한 디스크 이름. 여기서는 xvda (아마 AWS EC2의 EBS 볼륨).
T	    작업 타입. W는 쓰기 요청, R은 읽기 요청.
SECTOR	I/O 요청이 발생한 디스크의 시작 섹터 번호.
BYTES	요청 크기(바이트).
LAT(ms)	요청이 완료되기까지 걸린 시간(밀리초).
```

### cachestat
- cachestat 출력은 파일 시스템 캐시 사용 및 성능을 보여줍니다. 이는 리눅스 페이지 캐시에 대한 상태를 실시간으로 추적하여 파일 시스템 I/O와 캐시 효율성을 이해하는 데 유용합니다.
```
필드	설명
HITS	    요청이 캐시에서 처리된 횟수.
MISSES	    캐시에 데이터가 없어 디스크로부터 읽어야 했던 요청 수.
DIRTIES	    캐시에 기록되었으며 아직 디스크로 플러시되지 않은 데이터 수.
HITRATIO	전체 요청 중 캐시에서 처리된 요청의 비율 (%).
BUFFERS_MB	커널 버퍼 캐시에 사용 중인 메모리 (MB).
CACHED_MB	파일 시스템 캐시로 사용 중인 메모리 (MB).
```