## サービス概要

学校現場では、保護者との個人面談の日程調整を教員が手作業で行っています。実際に現場で感じていた課題は次の通りです。

- **手作業での割り振りは大きな負担**：全家庭分の面談を、様々な条件を考慮しながら組み立てる必要がある
- **1つの家庭が複数の担任と面談するケースがある**：兄弟が別クラスの場合や、通常学級と特別支援学級の両方に籍を持つ児童の場合、保護者1人が複数の担任と面談するため、担任同士で連続した時間に調整する必要がある
- **既存サービスは「先着順」が基本**：ITに不慣れな家庭や忙しい家庭が不利になりやすく、公平性に欠ける。また上記のような兄弟・特別支援学級を考慮した調整機能を持つサービスが存在しない

そこで本アプリでは、**「全家庭を必ず割り当てる」「学校特有の制約条件を尊重する」**という前提を置き、これらを同時に満たすスケジューリングアルゴリズムを軸とした機能・データ設計を行っています。

### 必須条件

- 限られた面談枠の中で、全家庭を必ず割り当てる
- 保護者が提出した面談不可日時には割り当てない
- 兄弟は連続した枠に配置する（例：兄が5年2組の枠 → 弟が3年2組の枠）
- 特別支援学級の児童は、通常学級の面談と連続した枠に配置する（例：3年1組の枠 → ひまわり学級の枠）

### 割り当ての優先度制御

兄弟や特別支援学級の児童がいる家庭は、「連続した枠」が必要なため、選べる枠が限られます。こうした制約の強い家庭を後回しにすると、空き枠が埋まってしまうため、**制約の強さをスコア化し、スコアの高い家庭から枠を確保**する方式を採用しています。

| 条件                   | スコア | 理由                                                                  |
| ---------------------- | :----: | --------------------------------------------------------------------- |
| 兄弟がいる             |   +4   | 連続2枠が必要。担任2人とも通常学級（30人前後）で空き枠が重なりにくい  |
| 特別支援学級を含む     |   +2   | 同じく連続2枠が必要だが、少人数学級のため兄弟のケースより選択肢は多い |
| 面談不可日時を提出済み |   +1   | 連続枠は不要だが、保護者側の都合で候補となる枠が減る                  |

合計スコアの降順で処理し、条件を満たす枠が見つからなかった家庭は割り当てをスキップして、教員が手動で最終調整します。

## 主な機能

## 技術スタック

| 分類           | 技術                                                            |
| -------------- | --------------------------------------------------------------- |
| フロントエンド | Next.js 16 / React 19 / TypeScript 5                            |
| UI             | MUI v9 / MUI X Charts                                           |
| データ取得     | fetch API                                                       |
| バックエンド   | Ruby 3.2 / Ruby on Rails 8.1（APIモード）                       |
| 認証           | JWT（自前実装、有効期限30分）                                   |
| 外部連携       | oauth2 gem（Google OAuth 2.0）/ Gmail API / Google Calendar API |
| データベース   | MySQL 8.0（本番：Amazon RDS）                                   |
| PDF出力        | Grover（Puppeteer + Chromium）                                  |
| インフラ       | AWS ECS Fargate / ECR / S3 / CloudFront / RDS / Route53 / ACM   |
| 開発環境       | Docker / Docker Compose                                         |
| テスト         | RSpec                                                           |
| 静的解析       | RuboCop / Brakeman / bundler-audit / ESLint                     |
| CI/CD          | GitHub Actions                                                  |

# Tsunagu

保護者面談の日程調整を自動化する、学校向けスケジューリングシステムです。

現役小学校教員としての実務経験をもとに、「兄弟の連続配置」「特別支援学級との調整」といった学校現場特有の複雑な制約を同時に満たすスケジューリングアルゴリズムを実装しています。

アプリケーションや実装内容は以下から確認できます。

- アプリケーション: <準備中（デプロイ後にURLを記載）>
- API: <準備中（デプロイ後にURLを記載）>
- GitHub (フロントエンド): <https://github.com/yasuhiro-dev/tsunagu-frontend>
- GitHub (バックエンド): <https://github.com/yasuhiro-dev/tsunagu-backend>

## 目次

- [機能](#機能)
- [開発環境 (フロントエンド)](#開発環境-フロントエンド)
- [開発環境 (バックエンド)](#開発環境-バックエンド)
- [本番環境](#本番環境)
  - [インフラ構成図](#インフラ構成図)
- [ER図](#er図)
- [使用技術 (フロントエンド)](#使用技術-フロントエンド)
- [使用技術 (バックエンド)](#使用技術-バックエンド)
- [使用技術 (インフラ・その他)](#使用技術-インフラその他)
- [画面](#画面)
- [工夫した点](#工夫した点)
- [テスト・静的解析](#テスト静的解析)
- [各種リンク](#各種リンク)

## 機能

- 認証
  - メール / パスワードログイン
  - パスワードリセット
  - ログイン / ログアウト
  - ユーザー登録
  - ゲストユーザーログイン (認証済み)

- 保護者
  - 児童の複数登録（兄弟・特別支援学級との併籍に対応）
  - 面談不可日時の提出
  - 確定した面談日程の確認
  - Googleカレンダーへの登録
- 教員
  - 面談期間・時間枠の設定
  - 面談枠の自動割り当て（制約充足アルゴリズム）
  - 担当児童一覧の確認（割り当て状況の確認）
  - 面談スケジュールのPDF出力
  - リマインドメール送信（Google連携）
- 管理者
  - 保護者の回答締切日の設定
  - 教員・保護者アカウントの管理（検索・一括削除・ページネーション）
  - 割当状況の可視化

## 開発環境 (フロントエンド)

フロントエンドは Next.js（TypeScript）で構築しています。面談枠の表示・割り当て結果の確認・保護者の面談不可日入力などの画面を担当します。

```
docker compose up -d
docker compose exec next npm install
docker compose exec next npm run dev
```

- URL: <http://localhost:3001>
- 開発言語: TypeScript
- フレームワーク: Next.js / React
- UIライブラリ: MUI

## 開発環境 (バックエンド)

バックエンドは Rails API mode で構築しています。認証、面談枠の自動割り当てロジック、PDF出力、Google連携（Gmail / カレンダー）などのAPIを提供します。

```
docker compose up -d
docker compose exec rails bundle install
docker compose exec rails bin/rails db:create db:migrate db:seed
```

- Rails API: <http://localhost:(3000)>
- DB: MySQL

## 本番環境

本番環境では、Next.js と Rails API をそれぞれ ECS Fargate サービスとして分けて運用しています。

- Route 53 - DNS
- ACM - HTTPS証明書
- ECS Fargate - Next.js / Rails API
- Amazon RDS for MySQL - データベース
- ECR - Dockerイメージ管理
- GitHub Actions - CI/CD

他にも使うものがあれば追加

### インフラ構成図

<!-- 構成図の画像を docs/images/readme/infrastructure.png のように配置してリンクしてください -->

![Tsunagu インフラ構成図](docs/images/readme/infrastructure.png)

#### リクエストの流れ

1. ユーザーは独自ドメインにHTTPSでアクセスします。
2. (CloudFront経由でNext.jsのコンテンツを配信します。)
3. Next.js のサーバー側処理が、必要に応じて Rails API へリクエストを送ります。
4. ALB が Rails API の ECS Fargate タスクへリクエストを転送します。
<!-- 実際の構成（ALB構成、Next.jsのホスティング方法など）に合わせて修正してください -->

#### 外部連携

- RDS: ユーザー、児童、クラス、面談枠、割り当て結果などの永続化・取得に利用します。
- Gmail API: リマインドメール送信に利用します。
- Google Calendar API: 保護者の面談日程をGoogleカレンダーへ登録する際に利用します。
- OAuth 2.0（Google）: 教員アカウントとGoogleアカウントの連携に利用します。

#### デプロイの流れ

`main` ブランチへの push を GitHub Actions が検知し、Docker イメージを ECR へ push したうえで、ECS サービスへデプロイします。

## ER図

<!-- ER図の画像を docs/images/readme/er-diagram.png のように配置してリンクしてください -->

![Tsunagu ER図](docs/images/readme/er-diagram.png)

| テーブル                  | 役割                                                                                                |
| ------------------------- | --------------------------------------------------------------------------------------------------- |
| `users`                   | 認証情報。Googleカレンダー連携用のトークンも保持                                                    |
| `teachers`                | 教員のプロフィール情報                                                                              |
| `families`                | 保護者のプロフィール情報                                                                            |
| `children`                | 児童情報                                                                                            |
| `class_rooms`             | クラス情報。担任の教員に紐づく                                                                      |
| `child_class_rooms`       | 児童の所属クラス。1人の児童が複数クラスに所属できる（通常学級と支援学級を掛け持ちするケースに対応） |
| `schedules`               | 面談を実施する期間の設定                                                                            |
| `meeting_slots`           | 面談枠。日時ごとに区切られた1コマ                                                                   |
| `assignments`             | 面談枠に割り当てられた児童の記録                                                                    |
| `family_unavailabilities` | 保護者が提出した面談不可の日時                                                                      |
| `sessions`                | ログインセッションの管理                                                                            |

## 使用技術 (フロントエンド)

| 技術                                            | バージョン / 補足      |
| ----------------------------------------------- | ---------------------- |
| [Next.js](https://nextjs.org/docs)              | 16.x                   |
| [React](https://react.dev/)                     | 19.x                   |
| [TypeScript](https://www.typescriptlang.org/)   | 5.x                    |
| [MUI](https://mui.com/)                         | v9                     |
| [MUI X Charts](https://mui.com/x/react-charts/) | 割当状況の可視化に使用 |

## 使用技術 (バックエンド)

| 技術                                                                             | バージョン / 補足               |
| -------------------------------------------------------------------------------- | ------------------------------- |
| [Ruby](https://www.ruby-lang.org/)                                               | 3.2.x                           |
| [Rails](https://rubyonrails.org/)                                                | 8.1.x / API mode                |
| [MySQL](https://www.mysql.com/) / [mysql2](https://github.com/brianmario/mysql2) | 8.0（本番：Amazon RDS）         |
| [jwt](https://github.com/jwt/ruby-jwt)                                           | 認証（自前実装、有効期限30分）  |
| [bcrypt](https://github.com/bcrypt-ruby/bcrypt-ruby)                             | パスワードのハッシュ化          |
| [oauth2](https://github.com/oauth-xx/oauth2)                                     | Google OAuth 2.0連携            |
| [Grover](https://github.com/Studiosity/grover)                                   | PDF生成（Puppeteer + Chromium） |
| [RSpec Rails](https://github.com/rspec/rspec-rails)                              | テスト                          |
| [RuboCop](https://rubocop.org/)                                                  | 静的解析                        |
| [Brakeman](https://brakemanscanner.org/)                                         | セキュリティ静的解析            |
| [bundler-audit](https://github.com/rubysec/bundler-audit)                        | 依存gemの脆弱性チェック         |

## 使用技術 (インフラ・その他)

| 技術                                                                                            | 用途                               |
| ----------------------------------------------------------------------------------------------- | ---------------------------------- |
| [AWS ECS Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html) | Next.js / Rails API のコンテナ実行 |
| [Amazon RDS for MySQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)  | 本番DB                             |
| [Route 53](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/Welcome.html)              | DNS                                |
| [ACM](https://docs.aws.amazon.com/acm/latest/userguide/acm-overview.html)                       | HTTPS証明書                        |
| [ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/what-is-ecr.html)                  | Dockerイメージ管理                 |
| [GitHub Actions](https://docs.github.com/en/actions)                                            | CI/CD                              |

## 画面

### トップページ

<!-- スクリーンショットを挿入 -->

![トップページ](docs/images/readme/top-page.png)

トップページでは、アプリの概要と主要導線が分かるようにしています。

### 認証

<!-- 2枚並べて挿入 -->

| ![ログイン画面](docs/images/readme/login-page.png) | ![新規登録画面](docs/images/readme/signup-page.png) |
| -------------------------------------------------- | --------------------------------------------------- |
| ログイン                                           | 新規登録（児童の複数登録に対応）                    |

### 面談枠の自動割り当て

<!-- デモGIF・スクリーンショットを挿入 -->

![割り当てデモ](docs/images/readme/assignment-demo.gif)

兄弟姉妹の連続配置・特別支援学級との調整など、制約条件を踏まえて面談枠を自動で割り当てます。

### 面談期間・時間枠の設定

<!-- スクリーンショットを挿入 -->

![面談期間設定](docs/images/readme/schedule-setting.png)

教員が面談を実施する日程と、1日あたりの時間帯を設定します。ここで作られた1コマ1コマが面談枠となります。

### 保護者の面談不可日提出

<!-- スクリーンショットを挿入 -->

![面談不可日提出](docs/images/readme/unavailability.png)

提示された面談枠のうち、都合の悪い枠をWeb上で選択して提出します。

### 確定した面談日程の確認

<!-- スクリーンショットを挿入 -->

![面談日程確認](docs/images/readme/confirmed-schedule.png)

保護者が自分の子どもの面談日時を確認し、ワンクリックでGoogleカレンダーに登録できます。

### 面談表PDF出力

<!-- スクリーンショットを挿入 -->

![PDF出力](docs/images/readme/pdf-export.png)

割り当て結果を、先生が確認・共有しやすいPDF形式で出力できます。

### 管理画面

<!-- 2枚並べて挿入 -->

| ![ユーザー管理](docs/images/readme/admin-users.png) | ![割当状況の可視化](docs/images/readme/admin-chart.png) |
| --------------------------------------------------- | ------------------------------------------------------- |
| 教員・保護者アカウントの検索・一括削除              | クラスごとの割り当て状況をグラフで確認                  |

### パスワードリセット

<!-- 2枚並べて挿入 -->

| ![リセット申請画面](docs/images/readme/password-reset-request.png) | ![リセット用メール](docs/images/readme/password-reset-email.png) |
| ------------------------------------------------------------------ | ---------------------------------------------------------------- |
| リセット申請                                                       | 届いたメールから再設定画面へ遷移                                 |

## 工夫した点

### 1. 割り当てロジックの責務分割

面談枠の自動割り当てロジックは、`GroupChildren` → `PrioritySort` → `AvailableSlots` → `TimeFilter` → `SiblingsFilter` → `SupportFilter` → `Assigner` という一連のクラス群に分割して実装しています。1クラス1責務に分けることで、それぞれを独立してテストでき、条件の追加・変更にも対応しやすい設計にしています。

### 2. 制約の強さをスコア化した優先度制御

兄弟・特別支援学級のように「連続した枠」を必要とする家庭は選べる枠が限られるため、後回しにすると割り当て不能になりやすい問題がありました。そこで制約の強さをスコア化し（兄弟+4、特別支援学級+2、面談不可日提出済み+1）、スコアの高い家庭から順に枠を確保する方式にすることで、限られた枠の中でも全家庭の割り当て成功率を高めています。

### 3. 多対多関係によるクラス設計

児童が複数のクラス（特別支援学級との併籍など）に所属しうるという学校現場特有の事情を踏まえ、`Child`と`ClassRoom`は中間テーブル`child_class_rooms`を介した多対多の関係で設計しました。`children`テーブル自体にクラス関連のカラムを持たせることなく、`child.class_rooms`という関連付けメソッド経由で柔軟にクラス情報を取得・更新できる構成にしています。

## テスト・静的解析

```
# Rails
docker compose exec rails env RAILS_ENV=test bundle exec rspec
docker compose exec rails bundle exec rubocop
docker compose exec rails bundle exec brakeman

# Next.js
docker compose exec next npm run lint
```

CIではRSpec、RuboCop、Brakeman、ESLintを実行しています。

<!-- Vitestは主要コンポーネントの確認用として導入しています。 -->

## 各種リンク

- アプリケーション: <準備中（デプロイ後にURLを記載）>
- GitHub (フロントエンド): <https://github.com/yasuhiro-dev/tsunagu-frontend>
- GitHub (バックエンド): <https://github.com/yasuhiro-dev/tsunagu-backend>
