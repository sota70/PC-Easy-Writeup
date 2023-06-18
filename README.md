# PC(Easy)
今回はHackTheBoxのPCマシーンを解いたので、その流れを解説します。<br>
<br>
<br>
## 情報収集
まずはポートスキャンを行い、アクセスできるサービスがないかを探します。<br>
![portscan](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/01c3fb71-aa95-4030-a776-2752a469d81d)
そうすると、22番ポート(SSH)と50051番ポートが開いているのか分かります。<br>
50051番ポートはgRPCという、Proto RequestとProto Responseの２つのメッセージで通信を行う<br>
プロトコルのデフォルトポートです。<br>
<br>
<br>
### gRPCへアクセス
gRPCが怪しいので、gRPCUIというツール(<https://github.com/fullstorydev/grpcui>)を使って<br>
中身を見てみます。<br>
![gRPCUI](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/7068923a-60b2-4b55-8b17-1b608df5576f)
そうすると、gRPCプロトコル経由でアカウントの登録、ログイン、アカウント情報取得ができることが分かります。<br>
ここでは、既にadminアカウントがadminというパスワードで登録されていました。<br>
sqlmapを使いアカウント登録、ログイン、アカウント情報取得のどれかでsql injectionできないか試してみます。<br>
![sqlmap_batch](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/fd9b4de9-3ef0-4870-87d1-d40ac0724c32)
そうすると、アカウント情報取得だけsql injectionの脆弱性があることが分かりました。<br>
<br>
<br>
## 侵入
このアカウント情報取得ページからテーブルの内容をsql injectionを使って表示できそうなので<br>
実際にやってみます。<br>
![sqldump](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/3d99dd12-37c3-4ea8-a10c-8179372cc841)
そうすると、sauというアカウント情報がパスワードとともに出てきました。このアカウントでSSH接続できそうなので、やってみます。<br>
sauのアカウントでSSH接続することができました。<br>
![ssh](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/e99119dc-772a-480d-80bd-82cd255ae401)
ちなみに、ログインするとuser.txtというファイルがあり、その中に1つ目のフラグが入っていました。<br>
<br>
<br>
## 情報収集2
まず、suidがついた実行ファイルが無いか確認します。PATH変数を利用した権限昇格ができないか試すためです。<br>
ですが、利用できるファイルが見つからなかったので、次はローカルで動いてるサービスが無いか確認してみます。<br>
![ps](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/5722c2bf-e0d8-484d-8999-7a2e17c10bfa)
8000番ポートはpyLoadのデフォルトポートなので、このサービスがrootで動いているか確かめてみます。<br>
![netstat](https://github.com/sota70/PC-Easy-Writeup/assets/46929379/58a16427-e087-4288-bdc1-27a853262f66kl)
netstatで確認したところ、rootで動いているみたいです。<br>
これは権限昇格に使えそうです。このサービス内で利用できそうな脆弱性を探してみると<br>
command injectionの脆弱性が見つかりました(<https://github.com/bAuh0lz/CVE-2023-0297_Pre-auth_RCE_in_pyLoad>)<br>
<br>
<br>
## 攻撃スクリプトの作成
pyLoadの脆弱性を利用した攻撃では、reverse-shellを使います。今回は２つのスクリプトを作ります。<br>
1つ目のスクリプトはpyLoadに/tmp/exploit.shを実行させるスクリプトです。<br>
２つ目のスクリプトは自分のPCで動かしているC2サーバーにアクセスさせるスクリプトです。<br>
今回はC2サーバーにVillain(<https://github.com/t3l3machus/Villain>)を使っています。<br>
<br>
<br>
## 侵入2
実際に作成したスクリプトを実行してみましょう。<br>
そうすると無事セッションが確立できました。<br>
rootディレクトリに行ってみると２個目のフラグがあったので、これでPCは終わりです。
