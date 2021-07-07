---
title: "Cloud Composerで \"Can't connect to MySQL server on 'airflow-sqlproxy-service.default.svc.cluster.local'\" と出てしまいTaskが実行されないときの対処法"
emoji: "✨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["GCP", "CloudComposer", "Airflow", "GKE"]
published: true
---

### 状況

* Cloud Composerのジョブがエラーで失敗してしまう。
* 失敗したTaskは 'up_for_retry' になるが、try_numberが増えない。
* Taskのログを見ると、`Can't connect to MySQL server on 'airflow-sqlproxy-service.default.svc.cluster.local'` と出ている。

### 対応

```
# エラーになっているpodを探す
kubectl get pods --all-namespaces | grep Failed

# 該当のpodを削除する
# POD_NAMEは上で出てきた airflow-sqlproxy-XXXXXXXXXX-XXXXX を入れる
kubectl delete pod ${POD_NAME}
```

上記コマンドを入力して、

```
pod "${POD_NAME}" deleted
```

と表示されれば削除できている。後は自動で新しいsql-proxyが立ち上がるのを待てばOK。
