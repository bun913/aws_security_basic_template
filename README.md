# AWSで設定するべき・意識するべきセキュリティ対策について

本リポジトリの内容はクラスメソッド様が出している以下Yotubue・ブログの内容を見て、まとめ・アウトプットしたものです。

Youtube

https://www.youtube.com/watch?v=Ci_4dhUCZm8&t=1513s

ブログ版はこちら

https://dev.classmethod.jp/articles/aws-security-all-in-one-2021/


## 前説

### セキュリティの基本

- セキュリティの基本・正しく意味のあるセキュリティを。
- 一人で頑張るのではなく全員で取り組み
- 人力で頑張るのではなく、精度の高い機械に任せる
- オンプレミスの代わりではなくクラウドの恩恵を受ける

- 守るものを把握する
  - どこになるか
  - 誰が責任者か
  - 誰が管理しちえるのか
  - 誰がアクセスできるのか
  - 今すぐにう状態がわかるか
- 全て把握できていないと守れない

- 脅威と脆弱性
  - 資産・脅威・脆弱性が合わさって初めてリスクが生まれる
    - この辺はIPAの試験にも頻出

- セキュリティは誰の責任か
  - 全員
  - それぞれが対応すべき範囲は違うけど


## 初級編

- 踏み台EC2にはセキュアにSSMで接続する
  - sshで接続しない
  - IMDSv2によりSSRF対策を設定する
- コンピューティング
  - Fargateなど自分で管理するべき範囲が狭いものを使う
  - lambdaのアクセス制御
  - 必ずAPIGateway経由で公開する
- データベース
  - バックアップによる復旧可能性
  - TLSによる通信経路の暗号化
  - IAMによる管理アクセスの制御
- ストレージ
  - S3
    - パブリックアクセスブロック
    - バージョニング
    - オブジェクトロック
    - ユーザーからアップロードさせる場合はセキュリティスキャンをする
      - https://dev.classmethod.jp/articles/cloud-one-file-storage-security/
- OS・ミドル
  - 脆弱性管理
  - AWSならSSMインベントリ + Inspector + SSMパッチマネージャー
  - より快適なのは FutureVulsなどの1つの基盤で一連の管理ができる仕組み

## 中級編

- AWSのセキュリティを維持する
- いまセキュアな状態かわからん
  - ConfigRulesでアラートを出して、自動復旧させる
  - SecurityHubはConfigRulesを利用したセキュリティチェックが可能
  - IAM AccessAnalyzerやPermissionsBoundaryを利用する
- IAM発行の権限を渡していく
  - IAM AccessAnalyzer
  - 外部に公開するリソースを不特定多数に公開してよいかチェックしてくれる
  - 開発者の権限昇格を防止
    - PermmisonsBoundaryに書かれていることしかできなくする
    - 委任した人が新しく作ったロールにも制約をつなげることができる
- 脅威検知(AWSは簡単)
  - GuardDuty
    - コイン毎に具
    - IAM不正利用
    - S3への不振なアクセス
- AmazonDetective
  - インシデント調査のつらみを吸収してくれる
  - VPCフローログとか色々ログをもっていて、グラフィカルに見せてくれる

## 上級編

- 組織全体でガバナンスを聞かせる
  - セキュリティチェック
  - CloudFormationによる自動化
- Oranizations
  - SCPでアクセス制御
  - 特定リージョン使用禁止
  - CloudTrail無効化禁止
  - GuardDutyの無効化禁止
- ControlTower
  - Organizationより色々やってくれる

## これらを踏まえてすること

以下をTerraformで簡単に作成できるようにする

初級編

- [x] 踏み台の代わりになるFargateタスク
  - セッションマネージャーでアクセスさせる
  - Terraformでレシピを作っておく
  - https://zenn.dev/bun913/articles/aws-bastion-fargate-and-ec2
    - やってみた記事
- [x] SSMパッチマネージャーをEC2に適用させるセットを作っておく
  - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/ssm_patch_baseline
  - https://www.zead.co.jp/column/ssm_patch/
  - https://zenn.dev/bun913/articles/aws-bastion-fargate-and-ec2
    - やってみた記事
- [x] AWS WAFをTerraformで設定
  - [x] https://github.com/bun913/aws_cloudfront_and_waf
  - https://zenn.dev/bun913/articles/waf-cloudfront-by-terraform
    - やってみた記事

中級編

- [x] 各種検出系サービスの有効化・通知(Terraform)
  - https://github.com/bun913/aws_security_hub
  - ConfiguRulesの設定
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/config_config_rule
  - セキュリティハブの設定
    - https://dev.classmethod.jp/articles/2020-strongest-aws-securitycheck-practice/
    - https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/securityhub_account
  - GuardDurtyで脅威検知
    - https://docs.aws.amazon.com/ja_jp/guardduty/latest/ug/what-is-guardduty.html
    - これからインシデント対応の計画を立てる
    - ポチッと有効化するだけで良い
    - 全リージョン・全アカウントで有効化する
    - EC2タイプの何か起こったら、SecuiryGroup変更で隔離。その後の調査はプロに依頼する。
  - インシデント対応の計画を立てる
    - どんなインシデントが発生するか

上級編


Terraformではないけど、利用を推奨したいサービス

- S3のセキュリティチェック
  - https://dev.classmethod.jp/articles/cloud-one-file-storage-security/
- 
