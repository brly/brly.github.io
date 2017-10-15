---
layout: post
title: gen_fsm から gen_statem への移行
tags: [Erlang]
---

最近のおしごとであったので.

とあるモジュールが gen_fsm で実装されており OTP20 では deprecated なため, 将来のことを考えて移行する必要が出てきた.

gen_statem とかは http://erlang.org/doc/design_principles/statem.html にあるような雰囲気で, ある状態に対してイベントが発生するとそれに対するアクションと次の状態が得られるやつのようで.

例として gen_fsm でとある状態遷移のやつを作ったあとに gen_statem に移す場合の例を書いてみる.

まぁ公式の http://erlang.org/doc/man/gen_fsm.html にも migration to statem みたいな感じで移行例みたいなのがあって, それに準じているのだけれど.

#### モデル

モデル化する対象として, 何かよいアイデアが思いつけばよかったのだけれど思いつかないので, 状態遷移を検索キーワードとしてよく出てきたストップウォッチさんを対象にすることに.

<img src="/assets/posts/2017/10-14/sw.png">

#### gen_fsm

https://github.com/brly/blog-coding/blob/master/2017/10-14-gen_statem/sw_fsm.erl

まず, 簡単にモジュールの説明を書くと start_link/0 で起動した後に push_hoge でイベントを起こすようにしていて,
push_hoge メソッドが状態遷移図の "Push start" とかに相当している.

gen_fsm ではイベントに対して同期的と非同期的に対応する場合は関数が分かれており, Module:StateName/2 が非同期で Module:StateName/3 が同期的となる. アリティ3 の場合は From が引数に増えている.

さらに状態を問わず発生させたいイベントを処理するためのコールバックとして Module:handle_event/3 と Module:handle_sync_event/4 を用意する必要もある. これはさらに引数として現在の状態を表す StateName が増え, StateName に合わせて処理を変えて ~ のような感じになるはず.

今回はそのような global なイベントとして, 内部で保持している数値を出力させるためのイベントをつくった.

また, くっそ雑な実装として計測中(measuring)状態の時は timeout イベントを発行するようにして, 保持している整数値をインクリメントするようにしている.
これはストップウォッチの計測中画面をイメージしたものになっている.

#### gen_statem

マイグレーションする. https://github.com/brly/blog-coding/blob/master/2017/10-14-gen_statem/sw_statem.erl

gen_statem に行くと, 先ほどまであった Module:StateName/2 やら Module:handle_event/3, Module:handle_sync_event/4 が消えていて, さらによくわからないが callback_mode/0 が増えており, そこで指定するやりようによっては全て "Module:StateName/3 で処理するように." という感じになっている.

gen_fsm では状態とは関数名(=atom)であるという制限があったが, gen_statem ではパワーアップして状態を任意のtermとして扱えるようになっており,
その場合は Module:handle_event/4 で処理するようになっている.

そうでなく, gen_fsm 形式を選ぶ場合は Module:StateName/3 で処理できるようだった.

どちらを選ぶかを表明するのが callback_mode/0 で `callback_mode() = state_functions | handle_event_function` のどちらかの atom を返すことによって行える.

今回はとりあえず gen_fsm からの移行なので state_functions にしておく.

Module:StateName/3 しかないのにどうやって非同期イベントに対応するのかというと, 最初の引数で event_type() なるものを取り, これが cast だったり {call, From} だったり info だったり timeout だったりと様々で, これで場合分けして対応する方式になるようで.

また gen_fsm 時にあった handle_sync_event 的なやつはなくなっているので, 各状態で対応する形になるのだが,
今回は面倒くさいので各状態でそのような global なイベントに対応する時はもとの handle_sync_event を残しておいて, それを呼ぶように変更した.

若干面倒くさい点として, reply やら timeout の形式が変わっており, timeout は `{next_state...` でやりたい時は末尾に整数値を加えるだけでよいのだが, reply の形式が変わっており, 新しく `action()` という形式でやる必要があり

```
%% sw_fsm.erl
handle_sync_event(show, _From, StateName, Data) ->
  Number = maps:get(display, Data, undefined),
  Reply = [{state, StateName}, {display, Number}],
  {reply, Reply, StateName, Data, 1000};
```

このように `{reply..` から始まる形では使えずに, reply は action として渡す必要があり,

```
%% sw_statem.erl
handle_sync_event(show, From, StateName, Data) ->
  Number = maps:get(display, Data, undefined),
  Reply = [{state, StateName}, {display, Number}],
  {keep_state, Data, [{reply, From, Reply}, {timeout, 1000, []}]};
```

こんな感じで, action は `action()` の配列を渡せるので `reply_action()` と `event_timeout() = enter_action()` として渡すように.
面倒くさくなったけど, 機能的にはリッチになってるか.

適当に遊んでいるようす. OTP20 で実行.

<img src="/assets/posts/2017/10-14/console.png">

マズイ点があったら教えてもらえると.
