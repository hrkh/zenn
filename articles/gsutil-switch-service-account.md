---
title: "gsutilコマンド使用時のサービスアカウントを環境変数で切り替える方法"
emoji: "📘"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gcp", "gsutil"]
published: true
---

GCPのCloud SDK (gsutil) で使われるサービスアカウントを環境変数で切り替えて利用しようと思ったところ、なかなか上手くいかなかったので備忘録として残します。

###  やりたかったこと

`gsutil` コマンドでGCSを操作するにあたって、実行するスクリプトごとにサービスアカウントを切り替える必要があった。
`gcloud auth activate-service-account` コマンドでサービスアカウントを切り替えることはもちろん可能だが、これは他のスクリプトにも影響を与えてしまうため避けたかった。

###  解決策

gcloud named configurationを複数作成しておき、環境変数 `CLOUDSDK_ACTIVE_CONFIG_NAME` でconfigurationを指定すればよかった。

### 詳細

1. configurationを作成する

    ```bash
    $ gcloud config configuration create <configuration_name>
    ```

1. サービスアカウントをactivateする

    ```bash
    $ gcloud auth activate-service-account <account> --key_file=<path/to/key_file>
    ```

1. 元々activeになっていたconfigurationをactivate (今回はdefaultと想定)

    ```bash
    $ gcloud config configuration activate default
    ```

1. 設定を確認する

    ```bash
    # 元々のアカウントの設定になっていればOK
    $ gcloud config list

    # 今回設定したサービスアカウントになっていればOK
    $ CLOUDSDK_ACTIVE_CONFIG_NAME=<configuration_name> gcloud config list
    ```

1. サービスアカウントを指定してgsutilコマンドを実行する

    ```bash
    # gsutilコマンドが指定のサービスアカウントで実行できているはず
    $ CLOUDSDK_ACTIVE_CONFIG_NAME=<configuration_name> gsutil ls
    ```

### 試したが上手くいかなかったこと

* 環境変数 `GOOGLE_APPLICATION_CREDENTIALS` でサービスアカウントを指定する
    *  これはクライアントライブラリ（Python APIなど）のサービスアカウントを指定するものらしく、Cloud SDKでは無視されてしまう
* 環境変数 `CLOUDSDK_AUTH_CREDENTIAL_FILE_OVERRIDE` でサービスアカウントを指定する
    * `gcloud` コマンドや `bq` コマンドでは動くようだが、`gsutil` コマンドでは無視されてしまう

### References

- [Activating a configuration - Managing gcloud CLI configurations](https://cloud.google.com/sdk/docs/configurations#activating_a_configuration)
- [Authorizing with a service account - Authorizing the gcloud CLI](https://cloud.google.com/sdk/docs/authorizing#authorizing_with_a_service_account)
