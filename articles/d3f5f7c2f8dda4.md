---
title: "RubyでIPアドレス範囲を確認する"
emoji: "🥤"
type: "tech"
topics: ["ruby", "ipv6", "ipv4"]
published: true
---

これは[Ruby Advent Calendar 2024](https://qiita.com/advent-calendar/2024/ruby)の8日目の記事です。書いたのは日本時間の20日で空いているスロットに潜り込ませてもらいます。前日分は「[ActivityPub RelayというActivityPubリレーサーバをRails 8を使って作った](https://gamelinks007.hatenablog.com/entry/developing-activity-pub-relay-with-rails-8)」、翌日分は「[Tips: ファイルの作成日時をファイル名のソート順と同じ順序になるように変換するRubyスクリプト](https://qiita.com/getty104/items/748b3141d7e73fef41e6)」でした。

筆者は[PaaSプロバイダのサポートとして働いていて](https://zenn.dev/zunda/books/0f89b5da809f49)、プライベートなネットワークどうしを接続するVPC PeeringやVPNの設定を確認することがあります。例えば、`192.168.1.0/29`と接続されてるはずなのに`192.168.1.15`と通信できないのはどうしてか調査を依頼されることがあります。

ここで、`192.168.1.0/29`は[Class Inter-Domain Routing](https://ja.wikipedia.org/wiki/Classless_Inter-Domain_Routing) (CIDR) 記法というものです。`/`のあとの数字(10進数)がネットワークのプレフィックス長で、`/`の左側のIPアドレスの最初の、プレフィックス長分のビット数が一致するIPアドレスの範囲をあらわします。この例では、32ビットのIPv4アドレスのうち、最初の29ビットが一致する、つまり最後の3ビットが任意のIPアドレス範囲、つまり`192.168.1.0`から`192.168.1.7`までのIPアドレス範囲をあらわすことになります。

ここまで考察すると、`192.168.1.15`と通信できない理由は、このIPアドレスが、`192.168.1.0/29`のIPアドレス範囲に含まれないことだとわかります。

が、即座に正確にここまでの考察をするのはたいへんですよね。こういう場合に便利なのが、Rubyです。

```ruby
$ irb
> require 'ipaddr'
> IPAddr.new("192.168.1.0/29").include?(IPAddr.new("192.168.1.15"))
=> false
```

IPv6アドレスでも同様に確認できます。ありがたい。

```ruby
$ irb
> require 'ipaddr'
> IPAddr.new("::ffff:192.168.1.0/125").include?(IPAddr.new("::ffff:192.168.1.15"))
=> false
```
