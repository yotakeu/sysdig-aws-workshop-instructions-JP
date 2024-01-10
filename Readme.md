# Sysdig ハンズオン EKS セキュリティワークショップ

Sysdigのハンズオンワークショップへようこそ。このワークショップでは、Kubernetes/EKSのセキュリティ上の課題を実際に体験していただき、Sysdigがどのようにお役に立てるかをご紹介します。

私たちは、皆さんのために個別のEKSクラスタとEC2インスタンス（jumpboxとして機能する）をプロビジョニングしました。ブラウザからAWS SSM Session Managerを経由してジャンプボックスに接続し、EKSクラスタと対話します。ジャンプボックスには本日のラボで作業するために必要なすべてのツールがプリロードされています。

また、Sysdig Secure内にユーザーをプロビジョニングしました。このSysdig SaaSのテナントは本日のワークショップに参加する全員で共有されますが、あなたのログインユーザーはチームに紐付けられており、あなたのEKSクラスタ/環境に関する情報のみが表示されるようにフィルタリングされています。

## 環境へのログイン

### AWS 環境

ファシリテーターから IAM ユーザー名とパスワードを受け取っているでしょう。自分の環境にサインインするには

1. ウェブブラウザを開き、https://sysdig-sales-engineering.signin.aws.amazon.com/console/ にアクセスします。
1. プロンプトが表示されたら、AWS Account IDに **sysdig-sales-engineering** と入力されていることを確認します。
1. 提供された IAM ユーザー名とパスワードを入力し、**Sign in** ボタンをクリックします。
    1. ![](instruction-images/awslogin.png)
1. コンソール右上のドロップダウンから **Sydney** リージョンを選択します。
    1. ![](instruction-images/region.png)
1. EC2サービスのコンソールに移動します（上部の検索ボックスにEC2と入力し、検索結果のEC2サービスをクリックできます）。
1. Resourcesの下にある **Instances (running)** リンクをクリックし、実行中のEC2インスタンスのリストに移動します。
    1. ![](instruction-images/instances1.png)
1. **Find instance by attribute or tag** 検索ボックスに**AttendeeXX**（XXはユーザー名の末尾の出席者番号）と入力し、エンター(リターン）を押します。
1. ジャンプボックスの隣にあるボックスにチェックを入れ、上部の**Connect**ボタンをクリックします。
    1. ![](instruction-images/instances2.png)
1. **Session Manager**タブを選択し、**Connect**ボタンをクリックします。
    1. ![](instruction-images/connect.png)
1. ターミナルウィンドウが開いたら、`sudo bash` と入力してから、`cd /root` と入力します。
    1. **注意:** セッションマネージャーセッション（ターミナルウィンドウ）を閉じて再度開くと、rootユーザーとそのホームディレクトリに戻るために、これら2つのコマンドを再度実行する必要があります。
1. `kubectl get pods -A`と入力すると、EKSクラスタ内の実行中のPodの一覧が表示されます。

> **注意:** ワークショップを通してGitHub上のサンプルファイルをいくつか紹介しますが、実行に必要なものはすべてジャンプボックスの/rootにあらかじめインストールされています。GitHubから何かをコピー＆ペーストしたり、**git clone**したりする必要はありません。

### Sysdig環境

講師から Sysdig へのログイン名とパスワードを受け取っているはずです（パスワードはAWSログインのパスワードと同じです）。自分の環境にサインインします：

1. ウェブブラウザを開き、https://app.au1.sysdig.com/secure/ にアクセスします。
1. 自身に提供されたメールアドレスとパスワードを入力し、**Log in**ボタンをクリックします。
1. Sysdig エクスペリエンスのカスタマイズ画面が表示されたら、右下の **Get into Sysdig** ボタンをクリックし、**Home** 画面に移動します。
    1. ![](instruction-images/sysdigexperience.png)

## モジュール 1 - ランタイム脅威の検知と防御 (Workload/Kubernetes)

最初のモジュールでは、ランタイム脅威の検知と防御に関する Sysdig の機能について説明します。

攻撃者がどのように侵入してくるかにかかわらず、攻撃者の行動は多くの点で共通しています。予測可能な一連の行動は、[MITRE ATT&CK Framework](https://attack.mitre.org/)によって詳細に説明されています。Sysdigの脅威リサーチチームは、世界中に大規模なハニーポットを設置し、攻撃者が侵入後にどのような行動を取るかを直接学んでいます。そして、すべてのお客様に代わって、[Rules](https://docs.sysdig.com/en/docs/sysdig-secure/policies/threat-detect-policies/manage-rules/) (何を検知べきか)と[Managed Policies](https://docs.sysdig.com/en/docs/sysdig-secure/policies/threat-detect-policies/manage-policies/) (見つけたときに何をすべきか)のライブラリを継続的に更新しています。また、ご希望であれば、弊社が提供するものを超えて、お客様独自のカスタム（Falco）ルールおよび(または)ポリシーを作成することも可能です - これは完全に透過的であり、魔法のブラックボックスではなく、オープンソースのツール/標準に基づいています！

当社のエージェントは、お客様の**ポリシー**で定義された様々な活動に対して、継続的に**ルール**と照らし合わせ、**イベント**を見つけたときに関連するすべてのコンテキストとともにリアルタイムでトリガーします。監視対象として下記以外にも、他の一般的なクラウド/SaaSサービスなどが近日中にさらに追加される予定です（GitHub、Oktaなど）：
* ノード/コンテナのLinuxカーネルシステムコール
* Kubernetesの監査証跡
* AWS、Azure、GCPの監査証跡

伝統的な "ルール/ポリシー "ベースのアプローチに加え、脅威検知/防止機能を強化する3つの機能があります：
* [コンテナ・ドリフト](https://docs.sysdig.com/en/docs/sysdig-secure/policies/threat-detect-policies/manage-policies/drift-control/) - コンテナ・イメージに含まれていない実行可能ファイルの実行を検知し、オプションでその実行をブロックすることができます。
* [クリプトマイニングML検出](https://docs.sysdig.com/en/docs/sysdig-secure/policies/threat-detect-policies/manage-policies/machine-learning/) - クリプトマイニングの検出に特化した機械学習モデルを採用しています。
* マルウェア（プレビュー） - 実行しようとするマルウェア（私たちがウォッチしているいくつかの脅威フィードで定義されている）を検知することができます。

### Sysdig 内でイベントを生成するための攻撃のシミュレーション

1. Sysdig UI で左側の **Insights** をクリックします。
    1. この画面はおそらく空であるため（新しい環境の疑わしいアクティビティを見たことがない）、**Welcome to Insights** プレースホルダ画面が表示されます。
1. それでは、イベントを生成してみましょう！
    1. このリンクをクリックして、新しいタブでクラスタ上のsecurity-playgroundサービスの（単純で安全でない）コードを開いてください - https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/app.py
        1. この Python アプリは、単純な **curl** コマンドに応答して、ファイルシステム上の任意のファイルの内容を返したり、ファイルシステム に任意のファイルを書き込んだり、ファイルシステム上の任意のファイルを実行したりする **非常に** 安全でない REST API を提供します。
            1. そして、それらを組み合わせて、例えば、ファイルをダウンロード/書き込みし、それを実行することができます。
        1. これは、非常に悪いリモートコード実行(RCE)の脆弱性をシミュレートしています。
            1. 1.脆弱性が悪用された時に何が起こるかを検出することに、私たちは興味があります。
    1. ジャンプボックスのセッションマネージャ端末のブラウザタブに戻ってください。
    1. security-playground サービスに対して実行するいくつかの **curl** コマンド例を含むスクリプトを見るために、**cat ./example-curls.sh** と入力してください：
        1. 機密パスの読み取り **/etc/shadow**
        1. ファイルを**/bin**に書き込み、**chmod +x**して実行する。
        1. apt**から**nmap**をインストールし、ネットワークスキャンを実行する。
        1. nsenter**コマンドを実行して、コンテナLinuxのネームスペースをホストに「ブレークアウト」する。
        1. Nodeのコンテナランタイムに対して**crictl**コマンドを実行する（KubernetesとKubeletをバイパスして直接管理する） 1.
        1. crictl**コマンドを使用して、同じNode上の別のPodからKubernetesシークレットを取得する（実行時に環境変数に復号化されたもの）
        1. crictl**コマンドを使用して、同じNode上の別のPod内でPostgres CLI **psql**を実行し、機密データを流出させる。
        1. KubernetesのCLIである**kubectl**を使って、別の悪意のあるワークロードを起動する（セキュリティプレイグラウンドのためにオーバープロビジョニングされたKubernetesのServiceAccountを活用する） 1.
        1. security-playground PodからNodeのAWS EC2 Instance Metadataエンドポイントに対して**curl**コマンドを実行する。
        1. 最後に、xmrig crypto minerを実行する。
    1. 先に進み、**./example-curls.sh**と入力してスクリプトを実行し、攻撃者の視点から返されるすべての出力を見てください。
    1. 暗号マイナー（xmrig）がこのコンテナに設定されている制限よりも多くのメモリを使用しようとするためです（PodSpecにこのような制限を設けるのが良いアイデアであるもう1つの理由を示しています！）。
    1. 次に、Sysdig UIタブに戻り、ブラウザのタブを更新します。
        1. どのクラスタ、名前空間、Pod から実行時イベントが発生しているかを示す円形の視覚化/ヒートマップが左側に表示されます。
        1. また、右側の **Summary** タブにはそれらのイベントのサマリーが、**Events** タブにはそれらのイベントの完全なタイムラインが表示されます。
        1. ![](instruction-images/insights2.png)
    1. 右側の「イベント」タブを選択します。
    1. ご覧のように、Sysdigがリアルタイムでピックアップしたイベントが多数あります！
        1.[](インストラクション画像/インサイト3.png)
    1. 一番上の**Detect outbound connections to common miner pools**をクリックしてからスクロールすると、プロセス、ネットワーク、AWSアカウント、Kubernetesクラスタ/名前空間/デプロイメント、ホスト、コンテナの詳細を含む、そのイベントのすべてのコンテキストを見ることができます。
       1. 特にプロセスツリービューを見ると、Pythonアプリ(gunicorn)が暗号マイナーのxmrigを起動するシェルを起動していることがわかります！
       1. ![](instruction-images/processtree.png)
       1. Exploreをクリックすると、このプロセスツリーの詳細とこの環境内の履歴を見ることができます。
       1.![](インストラクション画像/explore.png)
       1. このビューは、右側にある実行ファイル(xmrig)に関連する他の全てのイベントを表示するだけでなく、apt-get、nmap、nsenterなど、起こっている他の全てのことを表示します。
       1. ![](インストラクション画像/explore2.png)
1. イベントを理解する
    1. 最も古い/最初のイベントまでスクロールダウンし、それぞれの詳細/コンテキストをすべて明らかにするために、それぞれをクリックしてください。ここでピックアップしたものは以下の通り：
        1. **信頼できない機密ファイルの読み取り** - ウェブサービスが行うべきでない**/etc/shadow**ファイルの読み取り。
        1. **ドリフト検知** - 実行時に実行ファイルがコンテナに追加され（イメージにはなかった）、それが実行されるたびに
            1. 実行時にコンテナに変更を加えるのはベストプラクティスではない。むしろ、新しいイメージをビルドし、不変パターンでサービスを再デプロイすべきである。
        1. **コンテナでパッケージ管理プロセスを起動する** - **ドリフト検出**と同様に、実行中のコンテナでapt/yum/dnfを使用してパッケージを追加または更新すべきではありません。
        1. その代わりに、コンテナイメージのビルドプロセスの一部として、**Dockerfile**で実行してください。
        1. **docker クライアントがコンテナ内で実行される** - これは **docker** CLI だけでなく、**crictl** や **kubectl** といった他のコンテナ CLI でも実行される。
            1. コンテナがKubernetesクラスタ上のコンテナランタイム/ソケットに直接話そうとするのは珍しいことです！
            1. プロセスセクションを展開すると、データを流出させた**psql**のような実行されたコマンドが表示されます。
            1. ![](命令画像/psql.png)
        1. **コンテナからEC2インスタンスメタデータサービスに問い合わせる** - EKS Podsは、[IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) などの他の手段を使ってAWSとやり取りする必要があります。その代わりにノードを経由して認証情報を使用するのは疑わしい。
        1. **マルウェア検出** - 私たちは脅威フィードから多くのマルウェアのファイル名とハッシュを探します。
            1. マルウェアの実行をブロックすることもできます！
        1. **一般的なマイナー・プール・ポートへのアウトバウンド接続を検出する** - ネットワーク・トラフィックを（レイヤー3で）調べ、宛先が暗号マイナー・プールや[Tor](https://www.torproject.org/)エントリー・ノードのような不審なものである場合に検出します。

これは、サービスの一部としてすぐに利用できるルールのほんの一部です！

(オプション) **example-curls.sh**をコピーして、Sysdigがあなたが試したいと思う他の様々なものをピックアップするかどうかを確認したい場合は、自由に独自のカールを生成して遊んでみてください！

(オプション）すべてのマネージドポリシー（左側の **Policies** に移動し、次に **Runtime Policies** に移動します）とルールライブラリ（**Policies** に移動し、**Rules** のニンジンメニューを展開して **Rules Library** を選択します）を見てください。Falco YAML を掘り下げてください（これは "魔法のブラックボックス "ではないので、独自の Rules と Policies を書くことができることに注意してください）。この例では、PoliciesとRulesに焦点を当てています。

### アクティビティ監査
上記で見たことに加えて、アクティビティ監査では、実行されたすべての対話型コマンドと、関連する ファイルとネットワークのアクティビティもキャプチャします。

これはすべて同じタイムラインに集約され（もちろんフィルタリングすることも可能です）、マシン間を行き来する人々の横方向の動きを確認するのに役立ちます。

SysdigエージェントはどのLinuxマシンにもインストールすることができ（Windowsにもすぐにインストールできるようになります！）、私たちはEKSクラスタに加えてJumpboxにもインストールしました。つまり、ワークショップでこれまでに実行したすべての対話型コマンドは、そこでキャプチャされたことになる！もし誰かがEKSノードにSSHでアクセスしたり、エージェントがインストールされている場所でコマンドを実行した場合も、同じようにキャプチャされます。

これを見るには、左側の**Investigate**セクションに行き、次に**Activity Audit**に行きます。
![](指示画像/aa.png)
その後、**cmds**の1つをクリックして詳細を確認してください。

### この攻撃はなぜ成功したのでしょうか? 

この攻撃が成功するためには、多くのことが当てはまらなければならなかった：
1. 私たちのサービスがリモートでコードを実行される脆弱性があること。これは、私たち自身のコードが脆弱であること（今回のケースのように）、あるいは私たちのアプリが使用しているオープンソースパッケージ（pip、npm、maven、nugetなど）が脆弱であることのいずれかによる可能性があります。
1. 私たちが **curl** していたサービスは、**root** として実行されていました - そのため、コンテナのファイルシステム内のすべてを読み書きできるだけでなく、コンテナからホストにエスケープするときも root でした！
1. PodSpecは[**hostPID: true**](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/security-playground.yaml#L47)と[privrivileged **securityContext**](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/security-playground.yaml#L64)を持っていたため、コンテナ境界(実行中のLinuxネームスペース)からホストにエスケープすることができた。
1. 攻撃者は、実行時に**nmap**や暗号マイナーの**xmrig**のような新しい実行可能ファイルをコンテナに追加して実行することができた。
1. 攻撃者はインターネットからこれらのものをダウンロードすることができた（このPodはそのイグレスを介してインターネット上のあらゆる場所に到達することができたため）。
1. 私たちのサービスの ServiceAccount は過剰にプロビジョニングされており、K8s API を呼び出して他のワークロードを起動することができた（これは必要ない）。
    1. kubectl get rolebindings -o yaml -n security-playground && kubectl get roles -o yaml -n security-playground** を実行して、デフォルトの ServiceAccount に以下のルール/パーミッションでバインドされた Role があることを確認します：
        ```
        ルール
        - apiGroups：
            - '*'
            resources：
            - '*'
            動詞
            - '*'
        ```
    1. 少なくともそれはClusterRoleではなくRoleでした - つまりこのNamespaceでしかできないことがあるということです。しかし、Namespaceの中で完全な管理者としてできるダメージはたくさんあります！
1. 攻撃者はPod内から、EKSノードだけを対象としたEC2メタデータのエンドポイント（169.254.0.0/16）に到達できた。

これらはすべて修正できる：
* ワークロードの設定方法（Kubernetesが新しい[Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/)で強制できるようになりました。）
* Sysdig SecureのContainer Drift防止機能によるものもある。
* そして残りは、インターネットへのイグレス・ネットワーク・アクセスを制御する。

そして、この 3 つをすべて実行すれば、（単に攻撃を検知するだけでなく）攻撃全体を防ぐことができる！

### このワークロードを修正する方法（security-playground）

上記の各原因について、解決策を示します：
1. このケースで脆弱性を修正するには、静的アプリケーション・セキュリティ・テスト（SAST）製品を使って、安全でない コードを特定します。私たちのパートナーである[Snyk](https://snyk.io/product/snyk-code/)は、ここでは良い選択です。
    1. ![](instruction-images/Snyk-SAST.png)
    1. 代わりに、これがアプリ/コンテナ内の既知/公開の CVE（Log4Jなど）に基づくものであれば、Sysdigの脆弱性管理（今後のモジュールで取り上げます）がこれを検出し、コンテナのベースレイヤーまたはコードパッケージのいずれかに、脆弱性のない更新バージョンへのパッチを適用するよう知らせてくれるでしょう。
1. このコンテナをnon-rootで実行するには、実際には以下の方法でDockerfileを変更する必要がある。これが変更前の[Dockerfile](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/Dockerfile)で、これが変更後の[Dockerfile](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged)である。
    1. docker build の一部として、[使用するユーザとグループを追加する](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L3) 必要があります。
    1. Dockerfileに、デフォルトでそのユーザとして実行するよう指定する](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L8) 必要があります（これはあくまでデフォルトであり、実行時に上書きすることができます。
    1. ユーザー/グループが読み取りと実行（そしておそらく書き込みも）のパーミッションを持つフォルダにアプリを置く必要があります - [この場合、元の/appではなく、新しいユーザーのホームディレクトリを使用します](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/docker-build-security-playground/Dockerfile-unprivileged#L9)
    1. 最近のKubeCon Europeでは、最小特権コンテナの構築に関する素晴らしい講演があり、ここではさらに深く掘り下げています - https://youtu.be/uouH9fsWVIE
1. PodSpecから安全でないオプションを削除するだけです。しかし、理想的には、このようなオプションをPodSpecに入れられないようにする必要もあります。
    1. Kubernetesに組み込まれた機能(1.25でGAされた)で、そうしないように強制することができます - [Pod Security Admission](https://kubernetes.io/docs/concepts/security/pod-security-admission/).
        1. これは[各Namespaceにラベルを追加する](https://kubernetes.io/docs/tasks/configure-pod-container/enforce-standards-namespace-labels/)ことで機能する。ベースラインと制限の2つの基準について警告したり、強制したりすることができる。
            1. [ベースライン](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline) - HostPidやPrivilegedなど、PodSpecの最悪のパラメータは使用できませんが、コンテナをrootとして実行することはできます。
            1. [restricted](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) - これはさらに進んで、非 root での実行を含むすべての安全でないオプションをブロックする。
    1. また、Sysdig には Posture/Compliance 機能があり、デプロイ前に IaC を検知したり、実行時に問題を修正したりすることができる。
1. コンテナ・ドリフト（Container Drift）の実行時に追加される新しいスクリプト/バイナリの実行をブロックすることができる（このケースでは、ドリフトを防止するのではなく、検知するだけである）。
1. KubernetesのNetworkPolicy（今後のモジュールで取り上げます）か、各サービスが到達可能なものの許可リストを使用して、インターネットに到達するために明示的に認証されたプロキシを経由させることによって、インターネットへのPod（複数可）のイグレスアクセスを制限することができます。
1. Kubernetes APIへの不要なアクセスを許可するデフォルトのServiceAccountによって、Kubernetes APIへのRoleとRoleBindingを削除することができます。
1. 上記のようにNetworkPolicyでPodの169.254.0.0/16へのegressアクセスをブロックするか、AWSのドキュメントに記載されているようにIDMSv2で最大1ホップを確保するか、どちらかです - https://docs.aws.amazon.com/whitepapers/latest/security-practices-multi-tenant-saas-applications-eks/restrict-the-use-of-host-networking-and-block-access-to-instance-metadata-service.html

### 実際に修正する
私たちは、**なぜこの攻撃が機能したのか**の1-3が修正されたワークロードの例 - **security-playground-unprivileged**も実行しています。これは新しいnon-root Dockerfileで構築され、PSAが制限されたセキュリティ標準を強制するsecurity-playground-restricted Namespaceで実行されています（つまり、rootとして実行したり、コンテナのエスケープを許可するhostPIDや特権SecurityContextなどのオプションを持つことはできません）。kubectl describe namespace security-playground-restricted** - **pod-security**ラベルに注目してください。

オリジナルのKubernetes PodSpec [こちら](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/security-playground.yaml#L46) と、制限付きPSAをパスするために必要なすべての変更を加えたアップデート版 [こちら](https://github.com/jasonumiker-sysdig/example-scenarios/blob/main/security-playground-restricted.yaml#L26) を見ることができます。

1-3が修正された状態で、私たちの攻撃がどうなるかを見るには、**./example-curls-restricted.sh**を実行してください（security-playground-restrictedの異なるポート/サービスを指すだけで、前回のファイルと同じです）。注意してほしい：
* コンテナ内で root を必要とするもの（/etc/shadow の読み込み、/bin への書き込み、apt からのパッケージのインストールなど）は、Python アプリがそれを実行する権限を持っていないため、**500 Internal Server Error** で失敗します。
* root**と**hostPid**と**privileged**がないと、コンテナをエスケープできませんでした。
* 唯一うまくいったのは、ノードのEC2メタデータエンドポイントを叩くことと、xmrig crypto minerをユーザーのホームディレクトリにダウンロード/実行することだった。

また、SysdigでContainer Driftの防止（実行時に追加された新しい実行可能ファイルを実行できないようにする）を追加すると、EC2インスタンスのメタデータへのアクセス（将来のモジュールでNetworkPoliciesでブロックする）以外はすべてブロックされる。これを確認するには 
* security-playground-restricted-nodrift**を見る - (他のNamespaceのように)ドリフトを検出するのではなく、ワークロードが**security-playground-restricted-nodrift** Namespaceにある場合にブロックしていることに注意。
    * そして、別のHostPortでsecurity-playground-restrictedサービスの別のコピーを実行させています。
* *./example-curls-restricted-nodrift.sh** を実行し、同じカールを実行しますが、前回の例のように制限されているワークロードに対して実行します。
    1. Insights UI で結果のイベントを見ると、今回は Drift が検出されただけでなく、**防止**されたことがわかります。
    1. ![](instruction-images/driftprevented.png)

また、マルウェアを検出するだけでなく、ブロックすることもできるようになりました。
それを見るには 
* Policies**→**Runtime Policies**に移動し、**security-playground-restricted-nomalware**を見てください - （他のNamespaceのように）単にマルウェアを検出するのではなく、ワークロードが**security-playground-restricted-nomalware** Namespaceにある場合はブロックしていることに注意してください。
    security-playground-restricted-nomalware**Namespaceにワークロードがある場合にブロックしています。
* ./example-curls-restricted-nomalware.sh**を実行し、同じcurlsを実行しますが、Sysdigが（マルウェアを検知するだけでなく）マルウェアを防止しています（ただし、Container Driftはブロックしていません。）
    1. Insights UI で結果のイベントを見ると、マルウェアが今回検出されただけでなく、**実行を阻止**されたことがわかります。
    1. ![](instruction-images/malware.png)

このように、SysdigのContainer Driftとマルウェア検出だけでなく、ワークロードの姿勢を修正することを組み合わせることで、多くの一般的な攻撃を防ぐことができます！

最後に、security-playground-restricted を変更して、security-playground のようにセキュリティを弱体化させるテストをしてみましょう。以下のコマンドを実行して、安全でないコンテナイメージとPodSpecをそのネームスペースにデプロイしてみてください **kubectl apply -f security-playground-test.yaml**。security-playground-restricted**名前空間ではPSAが制限されているため許可されないと警告されていることに注意してください。DeploymentにPodを作成させても、そのDeployment（実際にはそのReplicaSet）は実際にPodを起動できないことに注意してください。
![](instruction-images/psa.png)

kubectl events security-playground -n security-playground-restricted** を実行すると、Pod作成の失敗を確認できます。

なぜPodが起動しないのかと頭を悩ませるよりも、パイプラインのもっと早い段階で、あるいはもっと遅い段階で、このようなことが起こる（そしてPodSpecを修正する必要がある）ことを知らせるべきです。

この表は、このワークロードを修正するための実験をまとめたものです：
|Exploit in the example-curl.sh|example-curl|security-playground|security-playground-restricted|security-playground-restricted + container drift enforcement|security-playground-restricted + malware enforcement|
|-|-|-|-|-|-|
|1|Reading the sensitive path /etc/shadow|allowed|blocked (by not running as root)|blocked (by not running as root)|blocked (by not running as root)|
|2|Writing a file to /bin then chmod +x'ing it and running it|allowed|blocked (by not running as root)|blocked (by not running as root)|blocked (by not running as root)|
|3|Installing nmap from apt and then running a network scan|allowed|blocked (by not running as root)|blocked (by not running as root)|blocked (by not running as root)|
|4|Running the nsenter command to 'break out' of our container Linux namespace to the host|allowed|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|
|5|Running the crictl command against the container runtime for the Node|allowed|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|
|6|Using the crictl command to grab a Kubernetes secret from another Pod on the same Node|allowed|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|
|7|Using the crictl command to run the Postgres CLI psql within another Pod on the same Node to exfiltrate some sensitive data|allowed|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|blocked (by not running as root and no hostPID and no privileged securityContext)|
|8|Using the Kubernetes CLI kubectl to launch another nefarious workload|allowed|blocked (by ServiceAccount not being overprovisioned)|blocked (by ServiceAccount not being overprovisioned and Container Drift Enforcement preventing kubectl being installed)|blocked (by ServiceAccount not being overprovisioned)|
|9*|Running a curl command against the AWS EC2 Instance Metadata endpoint for the Node from the security-playground Pod|allowed|allowed|allowed|allowed|
|10|Run the xmrig crypto miner|allowed|allowed|blocked (by Container Drift Enforcement blocking xmrig from being installed)|blocked (by Malware Enforcement)

*そして9は、NetworkPolicyやIDMSv2の1ホップへの制限によってブロックされる可能性がある。これは将来のNetworkPolicyモジュールで行う。

## モジュール 2 - ランタイム脅威の検知と防御（クラウド/AWS）

Sysdigのランタイム脅威検知は、LinuxカーネルのシスコールとKubernetesの監査証跡に限定されません - AWSのCloudTrail（同様にAzure、GCP、Okta、GitHubなど、常に追加されます）に対してエージェントレスでランタイム脅威を検知することもできます！エージェントレスというのは、CloudTrailを監視するFalcoがSysdigのSaaSバックエンドで実行されることを意味します。オプションとして[Cloud Connector](https://docs.sysdig.com/en/docs/installation/sysdig-secure/connect-cloud-accounts/aws/agent-based-with-ciem/)と呼ばれるお客様のアカウントでエージェントを実行することも可能ですが、ほとんどのお客様はSysdigがサービスとしてそれを行うことを好まれます。

AWSのCloudTrailの検出を簡単に見てみよう - EKSとAWS環境の両方をカバーすることがなぜ重要なのか。

### AWS IAM Roles for Service Accounts (IRSA)
AWS EKSには、[IAM Roles for Service Accounts (IRSA)](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html) と呼ばれる、PodにAWS APIへのアクセス権を与える仕組みがある。要するに、これはKubernetesの特定のサービスアカウントをAWSのIAMロールにバインドするもので、実行時にそのKubernetesサービスアカウントを利用するPodに、そのAWS IAMロールを利用するための認証情報を自動的にマウントします。

security-playground**Namespaceの**irsa** ServiceAccountは、**Action"： "s3:*"** ポリシーが適用されています。以下のコマンドを実行すると、その IAM Role の ARN を持つ ServiceAccount の Annotation が表示されます：
**kubectl get serviceaccount irsa -n security-playground -o yaml**

次のようなインラインポリシーがあります。よく見かける、s3サービス用の*です（実際には、バケット自体だけでなくコンテンツもカバーするために2つあります）。これは、単一のバケツResourceに適切にスコープされており、ないよりはましですが、なぜこのサービスのための*が悪い考えなのかがわかるでしょう。

```
{
    "バージョン"： "2012-10-17",
    "Statement"： [
        {
            "Action"： "s3:*",
            "リソース"： "arn:aws:s3:::attendeestack1-bucket83908e77-1d84qdfaymy9u",
            "効果"： "許可"
        },
        {
            "アクション"： "s3:*",
            "リソース"： "arn:aws:s3:::attendeestack1-bucket83908e77-1d84qdfaymy9u/*",
            "効果"： "許可"
        }
    ]
}
```

また、信頼関係を見ると、このロールは AWS IAM と統合するためにこの固有の OIDC プロバイダを割り当てられた EKS クラスタ内の **security-playground** Namespace 内の **irsa** ServiceAccount によってのみ引き受けられることがわかります。

```
{
    "Version"： "2012-10-17",
    "Statement"： [
        {
            "効果"： 許可
            "Principal"： {
                "Federated"： "arn:aws:iam::090334159717:oidc-provider/oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0"
            },
            "アクション"： "sts:AssumeRoleWithWebIdentity"、
            "条件"： {
                "StringEquals"： {
                    "oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0:aud": "sts.amazonaws.com"、
                    "oidc.eks.ap-southeast-2.amazonaws.com/id/25A0C359024FB4B509E838B84988ABB0:sub": "system:serviceaccount:security-playground:irsa"
                }
            }
        }
    ]
}
```

### Exploit
実行時にAWS CLIをコンテナにインストールし、いくつかのコマンドを実行すると、PodにIRSAロールが割り当てられているかどうかがわかります。rootに**example-curls-bucket-public.sh**ファイルがあるので、**cat example-curls-bucket-public.sh**で見て、**./example-curls-bucket-public.sh**で実行する。

AWS CLIのインストールは成功しましたが、S3の変更はアクセス権がないので失敗しました。security-playground Deploymentのマニフェストを更新し、これまで使用していた**default**のServiceAccountではなく、この**irsa** ServiceAccountを使用するようにしました。この変更を適用するには、**kubectl apply -f security-playground-irsa.yaml** を実行します。ここで、**./example-curls-bucket-public.sh** を再実行すると、今度はうまくいきます！

S3コンソールでこのバケットを見ると、バケット（とそのすべてのコンテンツ）がパブリックになっていることがわかるだろう（そして、攻撃者はS3のパブリックAPIからすぐにダウンロード/抽出することができる）！
![](instruction-images/bucketpublic.png)

### Sysdigの検出

ホスト側では、AWSに対して実行されているコマンドを含む多くの**Drift Detections**が表示されます。これはAWSのようなCLIをイメージに含めない良い理由です！![](インストラクション画像/s3drift.png)

しかし、AWS API側では、このバケットが公開されることに対する保護が削除されただけでなく、新しいBucket Policy(バケットを公開する)も適用されたことがわかります！

![](説明画像/s3cloudevents.png)
![](説明画像/s3cloudevents2.png)

> 注**： Sysdigのユーザー/チームがKubernetesクラスタとJumpboxだけを表示するようにフィルタリングされているため、残念ながら、これはまだ表示されません。コースのインストラクターは、皆さんが今日このAWSアカウントでこれらのコマンドを実行する際に、これらのイベントをお見せします - そして、それらは今後Team scopingに関するワークショップが改善された後に、ワークショップの参加者に見えるようになります。

### この攻撃を防ぐ方法/このワークロードを修正する方法

この IRSA の例は、以下の方法で防ぐことができました：
* IRSA のポリシーで s3* を使用せず、パブリックブロックの削除を許可したり、バケットポリシー（ファイルなどの読み書きのみ）を適用したりするなど、よりきめ細かく最小特権を設定すること。
    * 権限境界](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html)や[サービス制御ポリシー(SCPs)](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)のようなものも、このような過剰な権限を持つロールが作成されないようにするために役立ちます。
    * ![](https://docs.aws.amazon.com/images/IAM/latest/UserGuide/images/EffectivePermissions-scp-boundary-id.png)
* SysdigでContainer Driftを強制し、AWS CLIが実行時にダウンロード/実行できないようにする（イメージに含まれていないことも確認する）。

今回の例ではどちらを使っても防ぐことができたが、理想的には両方を行うことだ。

## モジュール 3 - ホストとコンテナの脆弱性管理

Linuxホストやコンテナイメージの脆弱性をスキャンするのに役立つツールやベンダーはたくさんあります。そして、ソリューションによっては、開発者のマシン、パイプライン、レジストリ、実行時など、さまざまな場所でスキャンしてくれます。Sysdigには、これらすべての場所で既知のCVEをスキャンできるものがあります。そして、実行時にスキャンする場合、私たちが追加したコンテキストは、ルーティングや優先順位付けに本当に役立ちます！

### ランタイム脆弱性スキャン
Sysdig のランタイム脆弱性スキャンを調べるには、以下の手順に従います：
1. Sysdig ブラウザのタブに移動し、左側の **Vulnerabilities** から **Runtime** に移動します。
    1. これは、過去15分以内にお客様の環境で実行されたすべてのコンテナと、当社のエージェントがインストールされているすべてのホスト/ノードのリストです。
    1. 自動的に重要度順にソートされるため、一番上のコンテナイメージは、（使用中の脆弱性の量と重要度に基づいて）修正すべき最も重要なものです。
    1. ![](instruction-images/vuln1.png)
1. 一番上のコンテナ/脆弱性をクリックし、一番上を調べます：
    1. 問題の画像とタグ - 現在実行中であることがわかります。
    1. 実行されているデプロイメント、ネームスペース、クラスタなどのランタイムコンテキスト。
    1. ![](instruction-images/vuln2.png)
1. 脆弱性**」タブをクリックする。
    1.![](instruction-images/vuln3.png)をクリックします。
1. CVEsの一つをクリックし、この脆弱性についてどこで知ったか、その脆弱性に関する修正または既知の悪用について、すべての詳細を記録する。
    1. ![](instruction-images/vuln4.png)
1. 脆弱性の詳細ペインを閉じる
1. 使用中**」フィルタボタンをクリックします。これは、実行されたのを見たことがない（したがって、悪用される可能性がかなり低い）すべての脆弱性を除外します。
1. Has fix**」ボタンをクリックします。これは、まだ新しいバージョンで修正プログラムが提供されていない脆弱性を除外します。
    1. これで残るのは、（イメージの中だけでなく）実際に稼働している脆弱性であり、**修正プログラムが存在する脆弱性です。これは、誰かに与えるべき、より合理的で優先順位の高いパッチ作業です！
    1. ![](instruction-images/vuln5.png)

### パイプライン脆弱性スキャン

コンテナイメージがレジストリに置かれる前に、ましてや実行環境に置かれる前に、脆弱性をスキャンするために、私たちはコマンドラインスキャンツールを用意しています。これは開発者のラップトップからパイプラインまで、どこでも実行できる。スキャンが失敗した場合（どのような条件でパスするか失敗するかは、きめ細かなポリシーで設定可能）、私たちのリターン・コードがゼロでないため、パイプラインは、修正されるまでそのステージを失敗させることができる。

以下は、脆弱性CLIスキャナー - https://docs.sysdig.com/en/docs/installation/sysdig-secure/install-vulnerability-cli-scanner/ - のインストールと実行方法の説明です。

私たちはすでにあなたのジャンプボックスにインストールしてあります。以下のコマンドを実行することで、Log4Jを含むイメージである**logstash:7.16.1**のスキャンを実行できます：

**./sysdig-cli-scanner -a app.au1.sysdig.com logstash:7.16.1**。

パイプライン・ステージのビルド・ログに出力されるだけでなく、Sysdig SaaSのUIで、出力に記載されているリンクをたどるか、UIの**脆弱性** -> **パイプライン**に進むことで、結果を調べることもできます。これはランタイムコンテキストが欠落していることに注意してください（パイプラインでスキャンされたため、ランタイムコンテキストをまだ知らないため）。

また、[レジストリ内のイメージをスキャンする機能](https://docs.sysdig.com/en/docs/installation/sysdig-secure/install-registry-scanner/) もありますが、このワークショップでは触れません。

## モジュール4 - Kubernetesの姿勢/コンプライアンス（設定ミスの修正など）

モジュール1で学んだように、Kubernetes/EKSクラスタとその上のワークロードが適切に設定されていることは非常に重要です。これは、お客様の姿勢（お客様のすべての設定を総合したもの）と、それらが様々な標準/ベンチマークに準拠しているかどうかに関するものであるため、「姿勢」または「コンプライアンス」と呼ばれます。

Sysdigは、CIS、NIST、SOC 2、PCI DSS、ISO 27001など、多くの一般的な規格に準拠していることを確認できます。現在の全リストをご覧になるには、左側の**ポリシー**をクリックし、**姿勢**の見出しの下にある**ポリシー**をクリックしてください。

Center for Internet Security (CIS)は、EKSを含む多くの一般的なリソースのセキュリティベンチマークを公開しています。詳しくはhttps://www.cisecurity.org/benchmark/kubernetes。このモジュールでは、クラスタとそのワークロードがこの標準に準拠しているかどうかを調べます。

1. ブラウザの Sysdig タブを開きます。
1. Posture**に移動し、次に**Compliance**に移動します。
1. Team and Zone-based authorization](https://docs.sysdig.com/en/docs/sysdig-secure/policies/zones/)を使用して、チームが自分のクラスタ/ゾーンのみを参照できるようにします。
1. あなたの見出しの下にある**CIS Amazon Elastic Kubernetes Service Benchmark**をクリックします（これはあなたのZoneに対して設定した唯一のコンプライアンス基準ですが、NIST、SOC2、PCIDSSなど他にも多くのコンプライアンス基準があります）。
    1. ![](指示画像/姿勢1.png)
1. ここには、私たちの攻撃を防ぐためのコントロールがいくつかあります。
1. それぞれの**Show Results**リンクをクリックすると、失敗したリソースのリストが表示されます。その後、**security-playground**リソースの隣にある**View Remediation**をクリックすると、改善手順を見ることができます：
    1. 4.2.6 ルートコンテナの入場を最小限にする
        1. RunAsUser rootが設定されているか、設定されていないコンテナ
        1. ルートを許可するコンテナ
    1. 4.2.1 特権コンテナの入室を最小限にする
        1. 特権として実行されるコンテナ
    1. 4.1.5 デフォルトのサービスアカウントが積極的に使用されないようにする。
        1. デフォルト」アカウントへの直接アクセス
    1. ![](指示画像/姿勢2.png)
    1.![](指示画像/姿勢3.png)

もし、**security-playground** のこれらの設定が CIS の EKS Benchmark をパスするように設定されていたら、 **security-playground-unprivileged** ワークロードと同じようになる。

また、このツールは、ワークロードやクラスタに関するセキュリティ上の問題を修正するのに役立つだけでなく、監査人に対して、遵守すべき標準に準拠していることを証明するのにも役立ちます。

## モジュール 5 - Kubernetes ネイティブファイアウォール (NetworkPolicies)

Kubernetesには組み込みのファイアウォールがあり、[NetworkPolices](https://kubernetes.io/docs/concepts/services-networking/network-policies/)と呼ばれるYAMLドキュメントを通して設定します。これらはIPやCIDRブロック/レンジだけでなく、Kubernetesの名前空間やラベルに基づいたルールを持つことができます。これはよりダイナミックで管理しやすくなります！

EKSを含む多くのKubernetesディストリビューション/オファリングでは、すぐに有効にはなりません。EKSの場合、NetworkPolicy Providerを選択する必要があります。一般的なものは[Calico](https://www.tigera.io/project-calico/)と[Cilium](https://cilium.io/)です。AWSは[Calicoをインストールするためのドキュメント](https://docs.aws.amazon.com/eks/latest/userguide/calico.html)を提供している。Calicoはクラスタにプリインストールされています。これらのプロバイダは基本的に、各Kubernetes Nodeにローカルファイアウォールを設定し、必要に応じてNetworkPoliciesを適用するためにすべてのNodeでそれらを更新します。

プロバイダをインストールした後でも、デフォルトではすべてのPodが他のすべてのPodと通信できます。そのため、そのトラフィックを制限するためのポリシーを実装する必要がある。最も安全なオプションは、デフォルトで拒否し、必要なものすべてを特別に許可することだ。既存の環境にポリシーを追加するのは少し大変かもしれません。そこでSysdigが役に立つ。

### Sysdig を使用してトラフィックを検出し、NetworkPolicies を生成します。

Sysdigはすべてのネットワークフローを追跡し、関係するすべてのKubernetesコンテキスト/ラベルを計算します。これにより、私たちが確認したトラフィックを表示し、そのトラフィックのみを許可するNetworkPoliciesを生成することができます。私たちのUIでは、あなたがそれらを許可したくない場合、私たちが見たもののチェックを外すことができます。

この機能を利用するには
1. ./example-curls-networkpolicy.sh**を実行して、security-playground Podがhello-server（別のNamespaceで実行中）にどのように到達できるかを確認してください。
1. ブラウザで Sysdig タブを開きます。
1. 左側の**Network**に移動します。
1. EKSクラスタ、名前空間**hello**、タイプ**Service**を選択します。
    1. ![](instruction-images/network1.png)
1. 右側のペインでは、hello Namespaceが以下のように構成されていることがわかります：
    1. バックエンドは、hello-server という名前のサービスと hello-server という名前のデプロイメントで構成されます。
    1. hello-clientとhello-client-blockedという2つのフロントエンドアプリが、hello-serverバックエンドサービスと通信します。
    1. また、security-playground サービスもバックエンドに接続していることがわかります（実行したエクスプロイトスクリプトには、接続するための **curl** がありました）。
    1. ![](instruction-images/network2.png)
1. Ingress**タブをクリックする。
1. ここで、hello-serverサービスと通信したくないもののチェックを外すことができます。
    1. hello-client**以外のすべてのチェックを外します。
    1. ![](instruction-images/network3.png)
1. Topology**に戻り、ブロックするものを赤い線で囲みます（hello-clientへのパスだけが許可されています）。
    1. ![](instruction-images/network4.png)
1. 生成されたポリシー**をクリックし、すべての内容をクリップボードにコピーします。
    1.![](指示画像/ネットワーク5.png)
1. ジャンプボックス端末のブラウザタブに戻る
1. vi policy.yaml**を実行します。
1. Iと入力して挿入モードに入る
1. PCの場合は**Shift-Ctrl-V**、Macの場合は**Shift-Command-V**でペーストする。
1. Escを押して挿入モードを終了し、**:wq**と入力して保存して終了する。
1. 適用するには、**kubectl apply -f policy.yaml**と入力します。
1. ./example-curls-networkpolicy.sh**を再度実行し、新しいNetworkPolicyによりhello-serverに到達できないことを確認します（タイムアウトします）。
1. kubectl logs deployment/hello-client-blocked -n hello**を実行し、hello-client-blockedサービスのログを確認します。
1. kubectl logs deployment/hello-client -n hello**を実行すると、hello-clientサービスからのログが表示されます。

#### ネットワークのイグレスの制御 - 特にインターネットへのイグレスの制御

これは、先ほどhello-serverでやったように、サービスへの入口を制御するだけでなく、 特にインターネットへの出口を制限するのにも便利です。

先ほどの安全でないセキュリティ・プレイグラウンドの例で、これがどのように役立つかを見てみましょう：
1. Sysdigブラウザのタブに戻る。
1. Network** セクションで、**security-playground** 名前空間と **Deployment** オブジェクトタイプを選択します。
    1. ![](instruction-images/network6.png)
1. ここでは、hello-server の相互作用の反対側(hello-server を呼び出すこのデプロイメント)だけでなく、apt パッケージをダウンロードしたり、クリプトマイナープールと話したりするために、curl を実行するときに呼び出されたすべてのインターネット IP も表示されます。
1. 上記で行ったように、すべてのインターネットegressを除外したNetworkPolicyを 生成できます。
    1. 実際には、クラスタ外のすべてのIP/CIDRを除外するようにデフォルト設定します（線が赤くなっているのはそのためです）：
        1. hello-serverのチェックを外し、security-playgroundがhello-serverにegressできないようにする。
        1. hello-serverのチェックを外して、security-playgroundがそのIPにイグジットできないようにする。
            1. もしこれらを許可したければ、それらの右側にあるプラスアイコンのチェックマークをクリックすることができる。
    1.![](インストラクション画像/ネットワーク7.png)
    1. これは、最初にカールでやったことの多くを防ぐもう一つのコントロールである！
1. Generated Policyタブを開く
    1. 生成されたポリシーをそのまま使うのではなく、policyTypesからIngressの行を削除して、最初にサービスに到達できるようにする。
        1. これをコピーしてテキストエディタに貼り付け、Ingressの行を削除してから、Ingressのないポリシーをクリップボードにコピーします。
    1. ![](instruction-images/network8.png)
1. ジャンプボックス端末のブラウザタブに戻る
1. vi policy2.yaml**を実行する。
1. Iと入力して挿入モードに入る
1. PCの場合は**Shift-Ctrl-V**、Macの場合は**Shift-Command-V**でペーストする。
1. Escを押して挿入モードを終了し、**:wq**と入力して保存して終了する。
1. 適用するには、**kubectl apply -f policy2.yaml**と入力します。
1. **example-curls.sh**を再実行し、このNetworkPolicyでブロックされた内容がどの程度起こったかをメモします。
    1. コンテナからホストに移動すると、NetworkPolicyは適用されなくなる（ただし、Nodeをカバーするファイアウォール/SecurityGroupは適用される）。
        1. これが、コンテナのエスケープを防ぐ必要があるもう1つの大きな理由です！

NetworkPoliciesの構文についてもっと知りたいなら、GitHubによく使われるパターンの例を集めた素晴らしいリソースがある - https://github.com/ahmetb/kubernetes-network-policy-recipes.

## 結論

以上、SysdigがAWS EKSを含むKubernetes環境のセキュリティ確保を支援するために顧客に提供している多くの機能の一部をas-a-serviceで簡単に紹介しました。

Sysdigがお客様のためにできることを、お客様の環境での無料トライアルでもっとご紹介したいと思います。詳細はファシリテーターまでお問い合わせください。

ご来場ありがとうございました！
