# AWS（RaiseTech）での学習記録
- [学習記録アウトプット一覧](lecture)
# AWS上に環境構築
データベースの基本的なCRUD処理が出来るサンプルアプリケーションのデプロイを実行しました。　　
- [デプロイ手順書](lecture/lecture05_deproy.md)

![サンプルアプリ１](https://github.com/Katsuya-00/RaiseTech_lecture/assets/128438140/c4cd22ac-e5d7-4e84-83d1-90853d66a2c6)

- ACMでSSL証明書を発行、Route53で独自ドメインを取得しALBのリスナールールを利用して接続をHTTPS化

## AWS構成図

<img width="543" alt="スクリーンショット 2023-05-13 5 46 56" src="https://github.com/Katsuya-00/RaiseTech_lecture/assets/128438140/e59070cd-6c1b-4b79-89c6-399f0cdb4ab7">

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
