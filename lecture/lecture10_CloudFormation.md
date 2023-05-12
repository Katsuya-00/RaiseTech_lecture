# CloudFormation
 AWSのインフラストラクチャを自動化するためのサービスです。  
 クラウドフォーメーションを使用すると、AWSリソース（例：EC2インスタンス、VPC、RDS、ロードバランサーなど）をテンプレートと呼ばれるJSONまたはYAML形式のファイルで記述し、AWS環境を一元的に管理できます。  
 #### メリット　　
 - 環境の一元管理：テンプレートを使用することで、AWS環境を一元的に管理できます。
- 自動化：AWSリソースの作成、更新、および削除を自動化できます。
- コスト削減：AWSリソースを必要なときに作成し、必要がなくなったときに削除することにより、コストを削減できます。
- リソースのトレース：テンプレートを使用することで、AWSリソースの変更や作成に関する変更ログを取得することができます。

### CloudFormationの全体像
テンプレートをクラウドフォーメーションにアップロードし、読み込ませる事でスタックが作成されます。  
全体像としては下記の通りになります。  
- テンプレート  
　↓　AWS CloudFormationの心臓部  
　↓　Json,Yaml形式のファイル  
　↓　リソースの定義、パラメータの定義  
- AWS　CloudFormation　  
　↓　フレームワーク  
　↓　スタックの作製/変更/削除/エラー検知とロールバック  
- スタック  
　   AWSリソースの集合

### 運用のベストプラクティス
テンプレートを分割していきます。
規模が大きくなると一つのテンプレートだと、管理するのに時間と手間がかかります。
変更時の影響範囲も大きくなってしまう為、
ベストプラクティスに沿った形で
NetworkLayer , SecurityLayer , Application の３つのテンプレートに分けていきます。

![テンプレート分割](https://user-images.githubusercontent.com/128438140/236090278-1f326e07-4765-4695-bd72-1c26807eecc8.jpeg)
[※参照 AWS Black Belt Online Seminar AWS CloudFormation アップデート](https://www.slideshare.net/AmazonWebServicesJapan/aws-black-belt-online-seminar-aws-cloudformation)

## 現在までに作った環境のコード化
### テンプレートの作成
#### ●NetworkLayer 

[VPC](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-vpc.html)、[Subnet](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-subnet.html)、[InternetGateway](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-internetgateway.html)、[Routetable](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-resource-ec2-routetable.html)を作成していく。  

[NetworkLayerテンプレート](RaiseTechCFn_templates/RaiseTech-Network_layer.yaml)  
ここでの警告は2つ有りました。  
１．倫理IDでは、記号が使えないということ  
２．AvailabilityZoneを直接書くとテンプレートの汎用性が悪くなるのでNG  
　　組み込み関数の!Select.を使用する  
　　GetAZsで指定されたリージョンのアベイラビリティーゾーンをアルファベット順にリストする配列を返すというもので前段で指定した数字とAZがリンクしている

```
一部抜粋してます
AWSTemplateFormatVersion: 2010-09-09　# AWS テンプレートバージョン
Description: RaiseTech-Network_Layer　# テンプレートの説明
Resources: # スタックを構成するリソースとプロパティ
  RaiseTechVPC: # 倫理ID　※※エラー　記号使えないです
    Type: AWS::EC2::VPC
    Properties: # リソース毎のプロパティ
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      Tags:
        - Key: Name
          Value: RaiseTechVPC　# 物理ID
  PublicSubnetRaiseTech1a:
    Type: AWS::EC2::Subnet
    Properties:
      AvailabilityZone: !Select. # 直接availabilityZoneを書くのはNG
        - 0
        - Fn::GetAZs: !Ref AWS::Region
      VpcId: !Ref RaiseTechVPC　# !Ref 倫理IDで物理IDを取得
      CidrBlock: 10.0.10.0/24
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: PublicSubnetRaiseTech1a
```

Security_LayerやApplication_LayerへリソースIDを渡す必要があるものをOutputsで作成します。
```
一部抜粋してます
OutPuts: 
 SecurityGroupRDSRaiseTech:
    Value: !Ref SecurityGroupRDSRaiseTech
    Export:
      Name: SecurityLayer-SecurityGroupRDSRaiseTech　
```

#### ●SecurityLayer  
各種セキュリティグループを作成します。  
今回は、[EC2](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html),[RDS](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-properties-rds-security-group.html),[ALB](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html)のセキュリティグループを設定しました。

[SecurityLayerテンプレート](RaiseTechCFn_templates/RaiseTech-Security_layer.yaml)  

同じテンプレート内でリソースの値を参照する際は、組み込み関数Refを使用しますが、別テンプレートでエクスポートされた値を参照する場合は組み込み関数ImportValueを使用します。

```
一部抜粋してます
SecurityGroupRDSRaiseTech:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SecurityGroupRDSRaiseTech
      GroupName: SecurityGroupRDSRaiseTech
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 3306
          ToPort: 3306
          SourceSecurityGroupId: !Ref SecurityGroupEC2RaiseTech
      Tags:
        - Key: Name
          Value: SecurityGroupRDSRaiseTech
      VpcId: !ImportValue NetworkLayer-RaiseTechVPC # !ImportValueで別テンプレートでエクスポートされた値を参照する
```
#### ●ApplicationLayer
[EC2](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html),[RDS](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/AWS_RDS.html),[ALB](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/AWS_ElasticLoadBalancingV2.html),[S3](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/AWS_S3.html)を作成します。

[ApplicationLayerテンプレート](RaiseTechCFn_templates/RaiseTech-Application_layer.yaml)  

Parametersを利用して、スタック作成時に設定できるようにします。  
⇛最初はParametersを記載する必要性が理解出来ず。。
Propertiesへ直接入力してました。最後に気づいたのですが、Parametersを指定することで、「違う環境で同じテンプレートを使用することが出来る」というのが狙いであると理解しました。  
メリットをまとめておきます。  
` 1.インスタンスタイプやデータベースのサイズなど、AWSリソースの設定に関するパラメータを動的に変更する場合。`

`2.テンプレートを再利用するために、テンプレートのパラメータを設定する場合。`  

`3.テンプレートのパラメータを外部から取得する場合、例えばCLIコマンドやAPI呼び出しでパラメータを渡す場合など。`  

`4.複数のスタックを作成する場合、スタック毎に異なるパラメータを指定することができます。`


PropertiesではRef関数を使います。
ここでの警告は、当初RDSのMasterPasswordを直接Propertiesへ入力していた為、怒られたのでパラメーターに記載した。



```
一部抜粋してます
Parameters:
  InstanceType:　# InstanceTypeを選択出来る
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - m1.small
      - m1.large
  DBInstanceIdentifier:　# DB名を決める
    Type: String
  DBInstanceType:　#　DBInstanceTypeを選択出来る
    Type: String
    Default: db.t2.micro
    AllowedValues:
      - db.t2.micro
      - db.t2.small
      - db.t2.medium
  DBPassword:　# MasterPasswordを直接入力するのはNG
    Type: String
    NoEcho: true　# パラメータの値がCloudFormationのコンソールやCLIから表示されなくなります。
  DBSubnetGroupDescription: # DBサブネットグループの説明
    Type: String
  S3BucketName:　# S3のバケットの名前
    Type: String
```

キーペアの作成も下記で行う事ができます

```
  CFnKeyPair: # キーペアの新規作成
    Type: 'AWS::EC2::KeyPair'
    Properties:
      KeyName: CFnKeyPair
```

### スタックの作成
AWSコンソールのCloudFormationからスタックの作成をしていきます。
先程作ったテンプレートをアップロードします。

スタック作成完了
![スタック作成完了](https://user-images.githubusercontent.com/128438140/236090315-54c6ecc0-562f-4bd0-917c-0a835e700c45.jpeg)

NetworkLayer Resources
![NetworkLayer_Resources](https://user-images.githubusercontent.com/128438140/236090362-42a8ec1d-075b-4e49-96d8-17147408ef0b.jpeg)

SecurityLayer Resources
![SecurityLayer_Resources](https://user-images.githubusercontent.com/128438140/236090386-1942d946-afab-440b-a462-4cc8ec84d4a2.jpeg)

ApplicationLayer Resources
![ApplicationLayer_Resources](https://user-images.githubusercontent.com/128438140/236090392-9a02d2c7-fdb0-4b1d-913c-8c19ac10b89a.jpeg)

# 感想

CloudFormationを利用する前は、インフラのコード化をする事で、「難しそうだし、何が良いのか？コンソールから選んで作る方が簡単だし、楽じゃないの？」という気持ちはありました。  
実際学んで実行してみると、メリットを理解する事が出来ました。  
インフラ環境を簡単に再現出来るのと、AWSリソースの作成、更新、削除の手間が大幅に削減されるなと実行してみて感じました。
得に更新、削除を行う場合、コンソール画面では、一つ一つ設定や、削除を行っていく必要があるのに対し、更新であれば、テンプレートの更新、削除であればスタッツの削除を行うだけで済むので、すごく楽になるという印象でした。
今回作成したテンプレートの書き方以外にも、様々な記述の仕方も有るという事がわかりましたので、引き続き学んでいきます。


