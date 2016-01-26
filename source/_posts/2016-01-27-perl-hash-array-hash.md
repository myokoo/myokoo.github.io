---
title: perlで「hashの配列のhash」にアクセスする方法
date: 2016-01-26 22:27:17
tags:
 - perl
---

perlで「配列のhash」や「hashの配列」を扱う方法について詳しく載っているサイトは多いけど、「hashの配列hash」とか1階層深いものへのアクセス方法とかが載っているサイトが見つからなかったので備考禄的な目的で残しておきます。
(色々書き方はあると思うのであくまで今回やった方法を載せます)

やりたかったこと
----------------------------------------------------------------------

複数のアカウントが行っている処理のログを元にアカウント別でデータをグループ化して、アカウント別でゴニョゴニョしたかった。
正確には、引っ張ってきたデータを元に別のログとの突き合わせをして要素追加したりしたけど掲題とは関係のでバッサリカット。。。

<!-- more -->

「hashの配列のhash」の想定データ構造
----------------------------------------------------------------------
下記のような構造をもつhashを想定

 ```perl
%smtp_lists = (
    account_aaa  => [
      {day => '2016-01-01', time => '00:00:00', from => 'xyp@aaaa.com', to => 'bbb@bbbb.com', src_ip => '210.188.22.21'},
      {day => '2016-01-01', time => '00:01:00', from => 'bji@aaaa.com', to => 'aaa@cccc.com', src_ip => '210.18.213.133'},
      {day => '2016-01-01', time => '00:02:00', from => 'zzb@cccc.com', to => 'aaa@dddd.com', src_ip => '111.42.22.12'},
    ],
    account_bbb  => [
      {day => '2016-01-01', time => '00:01:00', from => 'aaa@hoge.com', to => 'kru@hoge.com', src_ip => '221.16.130.13'},
      {day => '2016-01-01', time => '00:01:00', from => 'aaa@hoge.com', to => 'swu@hoge.com', src_ip => '221.16.130.13'},
    ],
    account_ccc  => [
      {day => '2016-01-01', time => '00:00:33', from => 'sef@bal.com', to => 'aaea@aol.com', src_ip => '210.221.44.111'},
      {day => '2016-01-01', time => '01:00:00', from => 'sef@bal.com', to => 'aaas@aol.com', src_ip => '210.221.44.111'},
      {day => '2016-01-01', time => '02:00:00', from => 'sef@bal.com', to => 'aava@aol.com', src_ip => '210.221.44.111'},
      {day => '2016-01-01', time => '02:31:00', from => 'info@hhh.com', to => 'abc@eeee.com', src_ip => '196.28.101.87'},
    ],
);
 ```

「hashの配列のhash」の作り方
----------------------------------------------------------------------

前提として、下部のような配列を用意済みとして、
 ```perl
@log_data = (
    '2016-01-01 00:00:00 account_aaa xyp@aaaa.com bbb@bbbb.com 210.188.22.21',
    '2016-01-01 00:01:00 account_aaa bji@aaaa.com aaa@cccc.com 210.18.213.133',
    '2016-01-01 00:01:00 account_bbb aaa@hoge.com kru@hoge.com 221.16.130.13',
    '2016-01-01 00:01:00 account_bbb aaa@hoge.com swu@hoge.com 221.16.130.13',
    '2016-01-01 00:02:00 account_aaa zzb@cccc.com aaa@dddd.com 111.42.22.12',
    '2016-01-01 00:00:33 account_ccc sef@bal.com aaea@aol.com 210.221.44.111',
    '2016-01-01 01:00:00 account_ccc sef@bal.com aaas@aol.com 210.221.44.111',
    '2016-01-01 02:00:00 account_ccc sef@bal.com aava@aol.com 210.221.44.111',
    '2016-01-01 02:31:00 account_ccc info@hhh.com abc@eeee.com 196.28.101.87',
);
 ```

下記の用な感じで `%lists` が想定しているデータ構造の「hashの配列のhash」になります。
 ```perl
my %lists;
for my $data (@log_data) {
    my ( $day, $time, $account, $from, $to, $ip ) = split(/ /, $data);
    my %hash = (day => $day, time => $time, from => $from, to => $to, src_ip => $ip);
    push( @{$lists{$account}}, \%hash );
}
 ```

* 2行目で配列を1行ずつループさせる
* 3行目でデータを半角スペース区切りで変数に格納
* 4行目で一番深い部分のhashを作成

    5行目でpushされるのはあくまで `%hash` のリファレンスとなるため、4行目の `%hash` を loop外(1行目の下とか)で宣言してしまうと、`@{lists{$account}}` の配列内に格納されるデータは全て同じ `%hash` (のデータ)を参照してしまいます。※説明が間違ってたらすいません。
その辺に関しては http://perldoc.jp/docs/perl/5.20.1/perldsc.pod に詳しく説明が載っています。

* 5行目でデータ内にあったアカウントをkeyとした「配列のhash」に4行目で作成したhashを追加

すると、 `%lists` が「hashの配列のhash」のデータ構造をもつhashになります。非常にお手軽。

ちなみに上記 `%lists` を `Data::Dumper` で確認すると下記のような感じになります。
 ```perl
$VAR1 = 'account_bbb';
$VAR2 = [
          {
            'day' => '2016-01-01',
            'from' => 'aaa@hoge.com',
            'ip' => '221.16.130.13',
            'to' => 'kru@hoge.com',
            'time' => '00:01:00'
          },
          {
            'to' => 'swu@hoge.com',
            'time' => '00:01:00',
            'ip' => '221.16.130.13',
            'from' => 'aaa@hoge.com',
            'day' => '2016-01-01'
          }
        ];
$VAR3 = 'account_ccc';
$VAR4 = [
          {
            'time' => '00:00:33',
            'to' => 'aaea@aol.com',
            'ip' => '210.221.44.111',
            'from' => 'sef@bal.com',
            'day' => '2016-01-01'
          },
          {
            'from' => 'sef@bal.com',
            'day' => '2016-01-01',
            'to' => 'aaas@aol.com',
            'time' => '01:00:00',
            'ip' => '210.221.44.111'
          },
          {
            'ip' => '210.221.44.111',
            'time' => '02:00:00',
            'to' => 'aava@aol.com',
            'day' => '2016-01-01',
            'from' => 'sef@bal.com'
          },
          {
            'ip' => '196.28.101.87',
            'to' => 'abc@eeee.com',
            'time' => '02:31:00',
            'day' => '2016-01-01',
            'from' => 'info@hhh.com'
          }
        ];
$VAR5 = 'account_aaa';
$VAR6 = [
          {
            'from' => 'xyp@aaaa.com',
            'day' => '2016-01-01',
            'ip' => '210.188.22.21',
            'to' => 'bbb@bbbb.com',
            'time' => '00:00:00'
          },
          {
            'to' => 'aaa@cccc.com',
            'time' => '00:01:00',
            'ip' => '210.18.213.133',
            'from' => 'bji@aaaa.com',
            'day' => '2016-01-01'
          },
          {
            'ip' => '111.42.22.12',
            'to' => 'aaa@dddd.com',
            'time' => '00:02:00',
            'from' => 'zzb@cccc.com',
            'day' => '2016-01-01'
          }
        ];
 ```

 「hashの配列のhash」へのアクセス方法
 ----------------------------------------------------------------------
`%lists` に格納されている各要素を取り出して出力してみます。

 ```perl
for my $hash_key (keys %lists){
    print 'account: ' . $hash_key . "\n";
    for my $account_data (@{$lists{$hash_key}}) {
        print '  day: '  . $account_data->{day} .
              ', time: ' . $account_data->{time} .
              ', from: ' . $account_data->{from} .
              ', to: '   . $account_data->{to} .
              ', ip: '   . $account_data->{ip} .
              "\n";
     };
};
 ```

* 1行目で一番外側のhashのkeyを `$hash_key` に取り出す
* 3行目で `@{$lists{$hash_key}}` の配列を1行ずつ取り出す
* 4行目以降で `$account_data` に格納されたデータを `$account_data->{<key>}` でとり出す。

出力結果は下記のような感じになります

```bash
account: account_ccc
  day: 2016-01-01, time: 00:00:33, from: sef@bal.com, to: aaea@aol.com, ip: 210.221.44.111
  day: 2016-01-01, time: 01:00:00, from: sef@bal.com, to: aaas@aol.com, ip: 210.221.44.111
  day: 2016-01-01, time: 02:00:00, from: sef@bal.com, to: aava@aol.com, ip: 210.221.44.111
  day: 2016-01-01, time: 02:31:00, from: info@hhh.com, to: abc@eeee.com, ip: 196.28.101.87
account: account_aaa
  day: 2016-01-01, time: 00:00:00, from: xyp@aaaa.com, to: bbb@bbbb.com, ip: 210.188.22.21
  day: 2016-01-01, time: 00:01:00, from: bji@aaaa.com, to: aaa@cccc.com, ip: 210.18.213.133
  day: 2016-01-01, time: 00:02:00, from: zzb@cccc.com, to: aaa@dddd.com, ip: 111.42.22.12
account: account_bbb
  day: 2016-01-01, time: 00:01:00, from: aaa@hoge.com, to: kru@hoge.com, ip: 221.16.130.13
  day: 2016-01-01, time: 00:01:00, from: aaa@hoge.com, to: swu@hoge.com, ip: 221.16.130.13
```

`for my $hash_key (keys %lists){` を `for my $hash_key (sort keys %lists){` にすればアカウントの文字列順で並べてくれます。

書いていると何に詰まったんだっけ・・・と思うも、そうなるのは俄ゆえんか。精進せねば。
