---
layout: post
title: Linux perf
tags: [Linux, system] 
comments: true
---  


# Install

- build 
  kernel source의 root directory에서 tool/perf로 이동하여 make  
  ~~(단, kernel source는 현재 시스템의 커널버전과 동일해야함)~~  
  디렉토리 이동 없이 쓰고 싶으면 환경변수에 추가해주면 됨  
- apt-get  
  apt로 설치 가능  
~~(추후 찾아봐야할듯, perf가 설치되지않은 시스템에 perf치면 apt command 알려줌)~~

```bash
$ perf
WARNING: perf not found for kernel 4.15.0-135

  You may need to install the following packages for this specific kernel:
    linux-tools-4.15.0-135-generic
    linux-cloud-tools-4.15.0-135-generic

  You may also want to install one of the following packages to keep up to date:
    linux-tools-generic
    linux-cloud-tools-generic
```

# Event

- perf가 측정하는 이벤트들로 하드웨어, 소프트웨어 구분  
~~(하드웨어 이벤트들의 경우, VM처럼 시스템에서 지원이 안되면 측정이 안됨 → 해결방안은 아직 모르겟음)~~
- 추가적으로 특정 함수나, 객체에 대한 tracing point도 설정하여 측정 가능~~(이건 안써봄)~~
- user-level 에서 성능 저하를 측정할 때는 cpu-clock을 사용  
~~(task-clock도 있던데 구글링 해보니 차이점 없다함)~~
- perf -e [event name]으로 특정 이벤트에 대한 성능 측정 가능  
- 지원하는 이벤트 목록을 보고싶으면 perf list로 확인 가능  
~~(event list에 대해서 찾아봐야 할듯)~~  

# Do it !

- perf에서 성능을 측정하는 command는 stat와 record로 구분
- stat :  주요 이벤트를 측정하여 해당 이벤트의 수행횟수 및 프로세스 실행시간을 화면에 출력
  - -e 옵션을 통해 측정하고자 하는 이벤트만 측정 가능

```bash
$ perf stat -p [pid] //현재 실행중인 프로세스의 성능 측정
$ perf stat [executable file & option] // 실행파일과 해당 실행파일의 인자를 줘서 성능측정
// 프로세스 종료 후 결과가 실시간으로 나옴 

//example : perf stat으로 ls 성능 측정
$ perf stat ls

Performance counter stats for 'ls':

              0.77 msec task-clock                #    0.663 CPUs utilized          
                10      context-switches          #    0.013 M/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
                84      page-faults               #    0.109 M/sec                  
   <not supported>      cycles                                                      
   <not supported>      instructions                                                
   <not supported>      branches                                                    
   <not supported>      branch-misses                                               

       0.001157733 seconds time elapsed

       0.001301000 seconds user
       0.000000000 seconds sys
```

- record : 측정된 결과를 별도의 파일로 출력(default output : perf.data, 이것도 이름 지정가능)
  - e 옵션을 통해 측정하고자 하는 이벤트를 선택할 수 있으며, 기본은 cpu-clock  
  - record로 측정된 결과를 보기 위해서는 script나 report command를 사용해야 함  
    - script : 측정된 결과를 모두 보여줌  
    - report : 측정된 결과를 잘 정리하여 보여줌  

```bash
$ perf record -p [pid] // 현재 실행중인 프로세스의 성능 측정
$ perf record [executable file & option] // 실행파일과 해당 실행파일의 인자를 줘서 성능측정

//example : perf record로 ls의 성능 측정
$ perf record ls 
[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.009 MB perf.data (2 samples) ]
$ ls 
perf.data
// perf script로 확인
$ perf script
ls  3271  2903.562735: 250000 cpu-clock:pppH:ffffffff99454863 xas_start+0x63 (/lib/modules/5.5.7/build/vmlinux)
ls  3271  2903.562984: 250000 cpu-clock:pppH:ffffffff98c11ea6 filemap_map_pages+0x286 (/lib/modules/5.5.7/build/vmlinux)

//perf report로 확인
$ perf report
# Samples: 2  of event 'cpu-clock:pppH'
# Event count (approx.): 500000
#
# Overhead  Command  Shared Object     Symbol               
# ........  .......  ................  .....................
#
    50.00%  ls       [kernel.vmlinux]  [k] filemap_map_pages
    50.00%  ls       [kernel.vmlinux]  [k] xas_start

#
# (Tip: List events using substring match: perf list <keyword>)
```

### Advanced

- LD_PRELOAD로 후킹하여 실행하는 프로그램의 성능 측정(without pid)

```bash
$ perf stat env LD_PRELOAD=[libaray path] [binary & option ...]
$ perf record env LD_PRELOAD=[libaray path] [binary & option ...]
```

- 측정 시 유용한 옵션들~~(내가 많이 사용하는 옵션들)~~
- perf record
1. -g or —call-graph
   성능을 측정하며, 성능저하가 많이 발생하는 부분의 call stack을 표현해줌
   대신, 해당 옵션을 넣어주게 되면 측정과정에서 성능저하가 발생할 수 있음
2. -s
   멀티쓰레드 바이너리일 때 사용하며, 쓰레드 별로 overhead를 분석하고 싶을 때 사용
   perf record를 이용하여 측정하게 되면 perf report를 할 때 가장 마지막에 pid와 tid를 보여줌

```bash
$ perf report -n -T | tail
...
#  PID   TID   cycles:ppp
  3602  3604  52758679502
  3602  3603    487183790

// tid 3604 쓰레드의 성능 저하 분석
$ perf report -T -tid 3604 -n
     8.28%         25657  h264dec  h264dec      [.] decode_cabac_residual_nondc
     7.12%         35880  h264dec  h264dec      [.] put_h264_qpel8_hv_lowpass
     6.19%         31430  h264dec  h264dec      [.] put_h264_qpel8_v_lowpass
     5.87%         28874  h264dec  h264dec      [.] h264_v_loop_filter_luma_c
     2.82%         10105  h264dec  h264dec      [.] ff_h264_decode_mb_cabac
     1.19%          4525  h264dec  h264dec      [.] get_cabac_noinline
```

3. -v
   이건 성능저하 지점을 좀더 자세하게 표현하여 함수단위로 보여줌


- perf stat
   1. -t —tid=[tid] 
      각 쓰레드 별로 stat 결과를 보고싶을 때 사용
      ~~(단, tid를 알아야되서 측정할 때 좀 귀찮음)~~


- 주로 측정에 사용 되는 이벤트
   - cpu-clock[Software event] : default event이며, 측정하는 프로세스에서 특정 함수가 cpu의 점유시간 
   ~~(task-clock하고 차이를 모르겟음)~~
   - cache-misses [Hardware event] : 캐시 미스가 발생한 횟수를 측정
   - dTLB-load-misses, iTLB-load-misses [Hardware Event]: 같이 쓰며 TLB miss가 발생한 횟수 측정
   - page-faults or faults[software event] : 프로세스에서 page-fault가 발생한 횟수 측정


- 성능 측정 후 별도의 output file을 생성하고 싶을때 ? 
   -o or —output [file_name]으로 사용 가능(stat와 record 모두 가능)
- report 시 overhead가 적은 애들은 제외하고 싶을때는 ?

```bash
$ perf report --percent-limit=[Wanted]
```



last update : 2021.03.14