---
title: "AWS 請求通知 BOT を AWS Lambda + AWS SDK + Go で作る~ SDK調査編 ~"
emoji: "🕌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Go","AWS"]
published: true
---

# AWS 請求通知 BOT を AWS Lambda + AWS SDK + Go で作る ~ SDK調査編 ~

## あらまし

AWS　請求額が設定ミスで跳ね上がる事例が多々発生しがちです、その対策として、現在の請求額を通知する BOT を作るのも一つです。
今回 Cost Explorer の SDK に関して個人的に調査を行った。

## Cost Explorer とは

AWSのコストを可視化できます。

https://aws.amazon.com/jp/aws-cost-management/aws-cost-explorer

#### 料金について

Cost Explorer のユーザーインターフェイスを使用したコストと使用状況の表示は無料。 
Cost Explorer API を使用して、プログラムでデータにアクセスすることもできます。 

API リクエストごとに $0.01$ USD の料金が発生する。

今日(2022/04/07)のレート
$$1 USD = 123.69 YEN$$

$$0.01 \times 123.69 = 1.2369$$

API リクエストごとに $1.2369$ 円 の料金が発生する。

なので、毎日リクエストを一回行ったとして、

$$1.2369 \times 30 = 37.107$$

一ヶ月(30日と仮定)約 $37.107$ 円程度の料金が発生する。

## Cost Explorer の SDK

https://docs.aws.amazon.com/ja_jp/aws-cost-management/latest/APIReference/API_Operations_AWS_Cost_Explorer_Service.html

Go で実装する際のドキュメント

https://docs.amazonaws.cn/sdk-for-go/api/service/costexplorer/#CostExplorer

## USD しか取れない問題

USD ベースの rate がとれる API 「OpenExchangeRates」 を利用してUSDをJPYに変換する。

https://docs.openexchangerates.org/docs

フリープランでは 1000 リクエスト/月 まで利用できる。

取得した様子

![スクリーンショット 2022-04-05 13.56.57.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1009976/7530dbd4-9e04-c001-f490-8b2ff97b47ec.png)

使い方は非常にシンプルで、app_id をクエリパラメータに与えて Get リクエストを投げるだけ。

Go言語で実装した例
app_id は ssm パラメータストア等の SecureString を利用。

```Go
func GetOpenexchangeratesJpy(sess *session.Session) float64 {
	// Base URL: https://docs.openexchangerates.org
	base := "https://openexchangerates.org/api/latest.json?app_id=%s"

	// Application id: https://docs.openexchangerates.org/docs/authentication
	// Using ssm parametor store: https://ap-northeast-1.console.aws.amazon.com/systems-manager/parameters
	svc := ssm.New(sess)
	app_id, err := svc.GetParameter(&ssm.GetParameterInput{
		Name:           aws.String("<YOUR_OPENEXCHANGERATES_APP_ID_PARAMSNAME>"),
		WithDecryption: aws.Bool(true),
	})
	if err != nil {
		log.Println(err.Error())
	}

	// Create Request url
	url := fmt.Sprintf(base, *app_id.Parameter.Value)

	// Start Request
	resp, err := http.Get(url)
	if err != nil {
		log.Println(err.Error())
	}
	defer resp.Body.Close()

	source, err := ioutil.ReadAll(resp.Body)
	if err != nil {
		log.Println(err.Error())
	}

	// Parse json
	desc := Schema{}
	json.Unmarshal(source, &desc)

	return desc.Rates.JPY
}

```

## Cost Explorer SDK のパラメータまとめ

今回は `GetCostAndUsage` を利用する。

https://docs.aws.amazon.com/ja_jp/aws-cost-management/latest/APIReference/API_GetCostAndUsage.html

#### Filter

さまざまなディメンションで AWS コストをフィルタリングできる。
たとえば、SERVICE と LINKED_ACCOUNT を指定して、そのアカウントのサービスの使用に関連付けられているコストを取得できる。

このパラメータは必須ではない。

#### Granularity

AWS コストの粒度を MONTHLY、DAILY、または HOURLY で設定できる。

有効な値:DAILY|MONTHLY|HOURLY

このパラメータは必須。

#### GroupBy

AWSコストは、最大2つの異なるグループ (ディメンション、タグキー、コストカテゴリ、またはタイプ別の2つのグループ) を使用してグループ化できる。

DIMENSIONタイプの有効な値は、

AZ、
INSTANCE_TYPE、
LEGAL_ENTITY_NAME、
INVOICING_ENTITY、
LINKED_ACCOUNT、
OPERATION、
PLATFORM、
PURCHASE_TYPE、
SERVICE、
TENANCY、
RECORD_TYPE、
USAGE_TYPE

また、タグの種類でグループ化し、有効なタグキーを含めると、空の文字列を含むすべてのタグ値が取得される。

もう少し詳しく書くと、

AZ に関しては アベイラビリティーゾーン毎の情報
INSTANCE_TYPE に関しては インスタンスタイプ(例えば、 t2.micro など)
LEGAL_ENTITY_NAME, INVOICING_ENTITY に関しては 例えば、Amazon Web Services Japan G.K. など
LINKED_ACCOUNT に関しては アカウントに紐づく情報
OPERATION に関しては オペレーション毎の情報（例えば RunInstances など）
PLATFORM に関しては プラットフォーム毎の情報
PURCHASE_TYPE に関しては オンデマンド、リザーブド、saving 等のプラン毎の情報
SERVICE に関しては サービス毎の情報
TENANCY に関しては (例えば、Sharedなど)
RECORD_TYPE に関しては (例えば、DiscountedUsage,Taxなど)
USAGE_TYPE に関しては (例えば、APS1-EUN1-AWS-Out-Bytesなど)

等のキーを指定可能で1回の呼び出しで2つまで指定可能。

このパラメータは必須ではない。

#### Metrics

有効な値は AmortizedCost、BlendedCost、NetAmortizedCost、NetUnblendedCost、NormalizedUsageAmount、UnblendedCost、UsageQuantity

こちらの記事が参考になる。

https://qiita.com/tamura_CD/items/4a9a412faf379b334986

今回は `UnblendedCost` を 利用する。

ちなみにこのパラメータは必須。

#### NextPageToken

次の結果セットを取得するトークン。

前回の呼び出しからの応答の結果が最大ページサイズを超える場合にトークンが提供される。

今回の調査の実装例に関しては簡単のためにページングは考慮しない。
結果オブジェクトに NextPageToken が含まれていたら後続を読み込む処理が必要になる。

#### TimePeriod

AWS コストを取得するための開始日と終了日を設定できる。

開始日は含みますが、終了日は含まない。

たとえば、start が 2017-01-01 で end が 2017-05-01 の場合、コストと使用状況のデータは 2017-01-01 から 2017-04-30 まで取得されるが、2017-05-01 は含まれない。

このパラメータももちろん必須。

# Goでの実装

通知予定の部分

```Go
package main

import (
	"aws-billing-notify/pkg/aws/profile"
	"aws-billing-notify/pkg/aws/sdk/ce/costandusage/calc"
	"aws-billing-notify/pkg/aws/sdk/ce/costandusage/granularity"
	"aws-billing-notify/pkg/aws/sdk/ce/costandusage/group"
	"aws-billing-notify/pkg/aws/sdk/ce/costandusage/metric"
	"aws-billing-notify/pkg/aws/sdk/ce/costandusage/term"
	"fmt"
	"log"
	"strconv"
	"time"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
	"github.com/aws/aws-sdk-go/service/costexplorer"
	"github.com/aws/aws-sdk-go/service/costexplorer/costexploreriface"
	"github.com/aws/aws-sdk-go/service/ssm"
)

type costParams struct {
	Granularity *string
	Term        *costexplorer.DateInterval
	Metrics     []*string
	Groups      []*costexplorer.GroupDefinition
}

func (c costParams) getCost(svc costexploreriface.CostExplorerAPI, start *string, end *string) (result *costexplorer.GetCostAndUsageOutput) {
	c.Granularity = granularity.Monthly.String()

	c.Metrics = []*string{
		metric.UnblendedCost.String(),
	}

	c.Term = &costexplorer.DateInterval{
		Start: start,
		End:   end,
	}

	service := costexplorer.GroupDefinition{
		Key:  group.Service.Key(),
		Type: group.Dimention.Type(),
	}
	c.Groups = append(c.Groups, &service)

	input := costexplorer.GetCostAndUsageInput{
		Granularity: c.Granularity,
		TimePeriod:  c.Term,
		Metrics:     c.Metrics,
		GroupBy:     c.Groups,
	}

	result, err := svc.GetCostAndUsage(&input)
	if err != nil {
		log.Println(err.Error())
	}
	return result
}

func main() {
	sess, err := session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
		Profile:           profile.Name,
	}) // todo: change lambda
	if err != nil {
		log.Println(err.Error())
	}

	c := costParams{}
	svce := costexplorer.New(sess)
	jst, err := time.LoadLocation("Asia/Tokyo")
	if err != nil {
		log.Println(err.Error())
	}

	start, end := term.CreateThisMonthRange(jst)

	cost := c.getCost(svce, start, end)

	svc := ssm.New(sess)
	rawjpy, err := svc.GetParameter(&ssm.GetParameterInput{
		Name:           aws.String("<YOUR_OPENEXCHANGERATES_JPY_RATE_PARAMSNAME>"),
		WithDecryption: aws.Bool(false),
	})
	if err != nil {
		log.Println(err.Error())
	}

	jpy, err := strconv.ParseFloat(*rawjpy.Parameter.Value, 64)
	if err != nil {
		log.Println(err.Error())
	}

	for _, group := range cost.ResultsByTime[0].Groups {
		amount, err := strconv.ParseFloat(*group.Metrics[*metric.UnblendedCost.String()].Amount, 64)
		if err != nil {
			log.Println(err.Error())
		}
		// Set openexchangerates jpy
		val := fmt.Sprintf("- %s: %f 円", *group.Keys[0], amount*jpy)
		fmt.Println(val)
	}
	// Set openexchangerates jpy
	fmt.Println(calc.Sum(cost, jpy))
}
```

レート更新部

```Go
package main

import (
	"aws-billing-notify/pkg/aws/profile"
	"aws-billing-notify/pkg/openexchangerates"
	"log"

	"github.com/aws/aws-sdk-go/aws"
	"github.com/aws/aws-sdk-go/aws/session"
)

func main() {
	sess, err := session.NewSessionWithOptions(session.Options{
		SharedConfigState: session.SharedConfigEnable,
		Profile:           profile.Name,
	})
	if err != nil {
		log.Println(err.Error())
	}
	openexchangerates.PutOpenexchangeratesJpy(sess)
}
```

# まとめ

今回は SDK の検証まででした。
AWS を利用したサービス設計時、ある程度正しい見積もりができないと大きな損害を生む可能性があるので、信用を失うことになりかねないです。
AWS の料金体系に関して、今後も継続して勉強を続けていこうと思いました。

次回は、Lambda を立てて実際に通知まで行きたいと思います。

