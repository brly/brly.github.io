---
layout: post
title:
tags: [x86]
---

redis-benchmark で遊んで見る。

ryzen をほったらかしにしていたので、ちょっとベンチマークプログラムで遊んでみたくなった。

ところでベンチをとるにはベンチをとられるスペックを予め控えて置くことが重要ですよね。

```
CPU: Ryzen7 1700: 8 core 16 Thread:
L1 8x(32KB I Cache + 64KB D Cache)
L2 8x512KB L3 2x8MB
ベースクロック: 3GHz, ブーストクロック: 3.7GHz
```

L3 が 2x8MB となっているのは, ryzen が CCX (Core Complex?) という単位でダイにあることから来ていて、
CCX に物理 4 CPU があるらしく、このモデルは CCX が 2つ搭載されていることになっており、
L3 Cache は CCX につき 1 つとなるため、こうなっている。それより小さなキャッシュはコアごとに存在している。

Memory: 32GB
DDR4-2133MHz, 8GBx4

redis-server: シングルスレッドで動作している。ように見える。詳しくないので知らず。
conf はなにもいじらず。

実行結果.

```
====== PING_INLINE ======
  100000 requests completed in 0.71 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
141643.06 requests per second

====== PING_BULK ======
  100000 requests completed in 0.57 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
176366.86 requests per second

====== SET ======
  100000 requests completed in 0.65 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
153846.16 requests per second

====== GET ======
  100000 requests completed in 0.75 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
132450.33 requests per second

====== INCR ======
  100000 requests completed in 0.62 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
162074.56 requests per second

====== LPUSH ======
  100000 requests completed in 0.52 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
190476.20 requests per second

====== LPOP ======
  100000 requests completed in 0.54 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
186915.88 requests per second

====== SADD ======
  100000 requests completed in 0.51 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
195312.48 requests per second

====== SPOP ======
  100000 requests completed in 0.68 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
146627.56 requests per second

====== LPUSH (needed to benchmark LRANGE) ======
  100000 requests completed in 0.74 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

100.00% <= 0 milliseconds
134589.50 requests per second

====== LRANGE_100 (first 100 elements) ======
  100000 requests completed in 1.40 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

99.73% <= 1 milliseconds
100.00% <= 1 milliseconds
71581.96 requests per second

====== LRANGE_300 (first 300 elements) ======
  100000 requests completed in 3.69 seconds
  50 parallel clients
  3 bytes payload
  keep alive: 1

  81.65% <= 1 milliseconds
  99.97% <= 2 milliseconds
  100.00% <= 2 milliseconds
  27100.27 requests per second

  ====== LRANGE_500 (first 450 elements) ======
    100000 requests completed in 5.22 seconds
    50 parallel clients
    3 bytes payload
    keep alive: 1

  0.15% <= 1 milliseconds
  99.78% <= 2 milliseconds
  99.93% <= 3 milliseconds
  99.98% <= 4 milliseconds
  100.00% <= 4 milliseconds
  19164.43 requests per second

  ====== LRANGE_600 (first 600 elements) ======
    100000 requests completed in 7.20 seconds
    50 parallel clients
    3 bytes payload
    keep alive: 1

  0.04% <= 1 milliseconds
  95.78% <= 2 milliseconds
  99.85% <= 3 milliseconds
  99.94% <= 4 milliseconds

  ====== MSET (10 keys) ======
    100000 requests completed in 0.80 seconds
    50 parallel clients
    3 bytes payload
    keep alive: 1

  99.85% <= 1 milliseconds
  100.00% <= 1 milliseconds
  124843.95 requests per second
```

#### Ping

まず最初の ping は PING_INLINE と PING_BULK があるけど、違いがわからない。
そしてみなあまり関心がないのか、検索しても出てこない。payload も 3bytes とあるので、違いもわからず...。
https://redis.io/commands/ping
ちなみに ping とはおそらくこれのことを指しているはず、だと思われる。

#### Set/Get

SET/GET は実行時オプションの `-d` で要素サイズを指定出来るが、デフォルトでは 2byte。
2byte の要素を SET/GET するリクエストを 100k くらいやる, やつを 50 parallel clients で.. とかやるので計算しにくいな..。
マルチスレッドを考えないといけないのが。と思ったが CPU 使用率を見た感じではシングルスレッドで動作していた。
50 parallel と言っているのはグリーンスレッドみたいなことだろうか。

SET: 153846.16 requests per second
GET: 132450.33 requests per second

2byte の書き込みは秒間 150k リクエストをさばけるが読み込みは 130k に下がるのがなんだか不思議だ。
いや、普通に？考えてみると single client の場合はうまくキャッシュのハードウェアプリフェッチがあたりそうだが、
平行に読み出すとプリフェッチが逆に悪さをして性能を阻害していそうな気もしなくもない。いや、さすがにそれくらい redis では気にしているか...。
毎回 uncache (non_temporal) で読み込みするようにし、redis 内部で cache 機構を持っているような気もする。まぁソース読まないとわからんけど。

```
Performance counter stats for 'redis-benchmark -t get -n 100000':

        523.496255      task-clock (msec)         #    0.983 CPUs utilized          
               248      context-switches          #    0.474 K/sec                  
                 4      cpu-migrations            #    0.008 K/sec                  
                74      page-faults               #    0.141 K/sec              


Performance counter stats for 'redis-benchmark -t set -n 100000':

        490.863560      task-clock (msec)         #    0.997 CPUs utilized          
                60      context-switches          #    0.122 K/sec                  
                 0      cpu-migrations            #    0.000 K/sec                  
                73      page-faults               #    0.149 K/sec
```

perf で見てみる。なんだか set のほうが context-switch が低いな？いや誤差か。

そして以外なことに数値を上げていってもあまり性能が下がらない。8KBの書き込みでも 150K rps をいけるが

```
$ redis-benchmark -t set -n 100000 -d 8192
====== SET ======
  100000 requests completed in 0.66 seconds
  50 parallel clients
  8192 bytes payload
  keep alive: 1

99.98% <= 1 milliseconds
100.00% <= 1 milliseconds
151515.14 requests per second

$ redis-benchmark -t set -n 100000 -d 16384
====== SET ======
  100000 requests completed in 1.01 seconds
  50 parallel clients
  16384 bytes payload
  keep alive: 1

99.64% <= 1 milliseconds
100.00% <= 1 milliseconds
98814.23 requests per second
```

8KB を超えると性能が下がってきた。ここくらいまで大きくなってくると TLB ミスの回数とかいろいろ見たくなるのだけど、perf で
ぱぱっと見られないので困った。いや、Ryzen のマニュアルに載ってるんだけど、前頑張ってみようとして折れたのだよね。

ちなみに get のほうもサイズを変えてやってみようとしたところ

```
$ redis-benchmark -t get -d 2
====== GET ======
  100000 requests completed in 1.10 seconds
  50 parallel clients
  2 bytes payload
  keep alive: 1

99.35% <= 1 milliseconds
100.00% <= 1 milliseconds
90909.09 requests per second
```

なぜか同じパラメータでもかなりパフォーマンスが低下していた。環境によって変わるものだろうけど set に比べ get はかなり変わっているようだった。

つかれたのでここまで。
