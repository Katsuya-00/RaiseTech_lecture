# 第６回　課題

## 目次

 [1. CloudTrail](#anchor1)  
 [2. CloudWatch](#anchor2)  
 [3. AWS 見積](#anchor3)  
 [4. AWS 利用料金の確認](#anchor4)  
 [5. 今回の学び](#anchor5)

<a id="anchor1"></a>

## CloudTrailのイベント確認
### CloudTrailとは
ユーザーアクティビティとAPI使用状況のログを収集するためのサービスです。

ユーザーが操作した内容を保存して、AWSアカウントの異常なアクティビティを検出・確認することでセキュリティが向上します。
（アプリケーションに関しては、アプリケーションのログを見ないと見れない）

デフォルトではCloudTrailは過去90日間のイベントを表示、検索、及びダウンロードができます。
90日以上のイベントを保存する場合、Amazon S3に保存する必要があります。（証跡を作成ボタンにて作成可能）

### CloudTrailにて、イベント確認しました
試しに証跡を作成してみた時のログ（後で消しましたが）

![履歴_証跡の作成](https://user-images.githubusercontent.com/128438140/232274779-fbf077ae-2468-4d19-b61d-8af18ceca224.jpeg)
### イベントに含まれていた情報
- イベントネーム：CreateTrail
- 時間
- ユーザー情報（画像途切れてますが）
- ログの保存に使用するS3バケット名
　
等の情報が含まれていました。

<a id="anchor2"></a>

## CroudWatchでALBのアラーム通知をする

### CroudWatchとは
EC2やRDSなどのAWSサービスやオンプレミスのサーバーなどを監視出来るサービスです。 各AWSリソースの変更をトリガーに何かアクションを起こしたり（CloudWatchイベント）、EC2のCPU使用率やディスクの読み取り量などを記録・監視する（CloudWatchメトリクス）ことが可能です。

### ALBのアラーム通知を設定しました
#### unhealthyになった際のアラーム通知

まずは、CloudWatchの画面より、アラームの作成で指定のtargetgroupがunhealthyになった際にEメールにて通知するように設定を行いました。

次に、EC2からアプリケーションのサーバーを止めました。そうすることでELBが異常を検知し、unhealthyになります。

![targetgroupがunhealthy](https://user-images.githubusercontent.com/128438140/232274792-874c07b3-68e0-4dfa-921b-9ca367eaa90d.jpeg)

アラームが作動し
![アラーム状態確認](https://user-images.githubusercontent.com/128438140/232274798-7f8531ca-208b-472d-87e4-4f6c148c8897.jpeg)

メールで通知されました。
![アラームEメール確認](https://user-images.githubusercontent.com/128438140/232274804-b7b999a3-84cc-4dc1-93e5-73e54d806bb0.jpeg)

OKアクションの追加をすることで、状態がOKになった際（healthy)も通知してくれる様にしました。

![OKアクションの追加](https://user-images.githubusercontent.com/128438140/232278371-bbe7efe2-796b-423a-83b1-98cc96be8943.jpeg)

EC2からサーバーを再稼働。
状態がOKになったメール通知がきました。

![OKメール通知](https://user-images.githubusercontent.com/128438140/232280383-94aab2ac-9dac-48d5-803e-adfa19832453.jpeg)

<a id="anchor3"></a>

## AWS利用料の見積作成
AWS Pricing Calculatorにて、現状のインスタンスに基づいて、見積もりを作成しました。

https://calculator.aws/#/estimate?id=a62f580a19e282c3e1f54ee962b96fc2955fceb8

<a id="anchor4"></a>

## AWSマネジメントコンソールから、請求情報確認

### 現在の利用料
RDSが誤ってEC2と別リージョンに構築していたので、一時的にマルチAZ構成にして、リージョン変更した為、その分の利用料のみ掛かっています。($0.06+税)

![今月の費用](https://user-images.githubusercontent.com/128438140/232274812-5db55392-38ca-4ac5-a4f5-5370cccbf476.jpeg)

無料利用枠に収まっているか確認
現状、無料利用枠で収まっているが、EBSが予測では超える事に。  
スナップショットは記録していないので、cloud9のEC2を残したままだった分を、一度削除してみて、様子見してみる。

![無料利用枠の確認](https://user-images.githubusercontent.com/128438140/232279642-73b9e33b-4aa5-4012-b228-fd4bf44de06f.jpeg)

ついでにBudgetsで予算登録しておきました。

![budgetsで予算登録](https://user-images.githubusercontent.com/128438140/232274815-048d4bf3-5baa-423a-8af6-2b997f4f8604.jpeg)


<a id="anchor5"></a>

# 今回の学び
ログや、アラーム通知は、運用していく中で、かなり重要な部分だと感じました。

恐らく、実際のシステムを想定すると、異常が起きると、復旧まで１分１秒を争う世界だと思うので、手かがりや異常への気付きという部分は、いかに早く対応出来るかという部分が勝負になってくると想定されます。
今の段階から、そういう部分に触れて、慣れておく必要もあると感じました。  
コスト面に関しても、掛かっている認識は無かったのと、無料利用枠を超えそうな物も有るので、今後注意して見ておこうと思いました。


