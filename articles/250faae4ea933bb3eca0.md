---
title: "Cloud Composerから管理用のMySQLに接続できないときの対処法"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GCP", "CloudComposer", "Airflow", "GKE"]
published: true
---

### 事象

* Cloud Composerのジョブがエラーで失敗してしまう。
* 失敗したTaskは 'up_for_retry' になるが、try_numberが増えない。
* Taskのログを見ると、`Can't connect to MySQL server on 'airflow-sqlproxy-service.default.svc.cluster.local'` と出ている。

### 注意

Cloud Composer環境を複数起動している場合は以下のコマンドそのままだと動かない可能性があるため、該当するclusterやnamespace, pod等の取得は各自の環境に合わせて行うこと。

### 状況確認

* まず、ブラウザからCloud Composer環境のモニタリングを見てみる。
  * データベースのヘルスが Unhealthy になっていることが確認できたので、MySQLに正常にアクセスできていないことがわかる。
* 次に airflow-scheduler からMySQLに入れるか確認する。
  * airflow-scheduler に入る。
    ```
    # clusterの名前と場所を取得
    $ cluster_info=$(gcloud container clusters list | grep ${Composerの環境名} | grep Running
    $ cluster=$(echo $cluster_info | cut -d' ' -f1)
    $ location=$(echo $cluster_info | cut -d' ' -f2)

    # credentialを取得
    $ gcloud container clusters get-credentials $cluster --zone=$location

    # podのnamespaceと名前を取得
    $ pod_info=$(kubectl get pods --all-namespaces | grep airflow-scheduler | grep Running)
    $ namespace=$(echo $pod_info | cut -d' ' -f1)
    $ pod=$(echo $pod_info | cut -d' ' -f2)

    # airflow-scheduler に入る
    $ kubectl exec --namespace=$namespace $pod -c airflow-scheduler -it -- bash
    ```
  * MySQLに入る。
    ```
    # MySQLのログイン情報を取得
    # mysql+mysqldb://${USER}:${PASSWORD}@${HOST}/${DATABASE}?charset=utf8 の形式で取得できる
    $ echo $AIRFLOW__CORE__SQL_ALCHEMY_CONN

    # MySQLにログイン
    $ mysql -h ${HOST} -u ${USER} -p${PASSWORD} ${DATABASE}

    # 適当にクエリを発行してみる
    mysql> select * from task_instance limit 1\G
    ```
    * クエリの結果がきちんと返ってきたので、MySQLは正常に動作している。
* airflow-sqlproxy が正常に動いているか確認する。
  * MySQLは正常なので、MySQL (Cloud SQL) にアクセスするためのSQL Proxyに問題がありそう。
    ```
    # エラーになっているpodを探す
    $ kubectl get pods --all-namespaces | grep airflow-sqlproxy | grep Failed
    ```
    * Failedになっている airflow-sqlproxy-XXXXXXXXXX-XXXXX が返ってきた。
### 対応

上記の状況確認でSQL Proxyが動いているpodに問題がありそうなので、これを再起動してみる。
```
# 再起動するためにはpodを削除すればよい
# POD_NAMEは上で出てきた airflow-sqlproxy-XXXXXXXXXX-XXXXX を入れる
$ kubectl delete pod ${POD_NAME}
pod "${POD_NAME}" deleted
```

後は自動で新しいsqlproxyが立ち上がるのを待てばOK。

### References

1. [Cloud ComposerでAirflow Workerに入る
](https://qiita.com/notrogue/items/272aa76fe335cd47476f)
