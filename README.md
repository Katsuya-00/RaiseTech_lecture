# AWS（RaiseTech）での学習記録
- [学習記録アウトプット一覧](lecture)
# ①AWS上に環境構築
- データベースの基本的なCRUD処理が出来るサンプルアプリケーションのデプロイを実行しました。　　
- ACMでSSL証明書を発行、Route53で独自ドメインを取得しALBのリスナールールを利用して接続をHTTPS化
- [デプロイ手順書](lecture/lecture05_deproy.md)

![testcrud net](https://github.com/Katsuya-00/RaiseTech_lecture/assets/128438140/8a79c1c5-0010-42b7-84fb-25bd8a6b69d6)



## AWS構成図

<img width="649" alt="スクリーンショット 2023-05-13 11 36 27" src="https://github.com/Katsuya-00/RaiseTech_lecture/assets/128438140/4eefdeaf-4ad9-454f-83be-7956916e2c65">


## 使用技術
- AWS
  - VPC
  - EC2
  - RDS (Amazon RDS for MySQL)
  - Route53
  - ACM
  - ALB
  - S3
- Ruby 3.1.2 (サンプルアプリケーション）
- Ruby on Rails 7.0.4 (サンプルアプリケーション）
- MySQL 8.0
- Nginx
- Unicorn

# ②CloudFormationでの環境構築
CloudFormationでの環境構築を実行
- [CloudFormation手順書](lecture/lecture10_CloudFormation.md)
# AWS構成図

<img width="642" alt="スクリーンショット 2023-05-13 11 56 07" src="https://github.com/Katsuya-00/RaiseTech_lecture/assets/128438140/a50e6782-815d-4d9d-b02b-77db7f86c26d">




