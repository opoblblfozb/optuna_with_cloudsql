# optuna_remoteRDB使用法_with_GCP
## step1_SQLinstanceをたてる
- 色々あとで使うパラメタがある。
- DATABASE_USER, PASSWORD　⇒　初期設定時
- DBNAME　⇒　使用するデータベースを作成しておく。
- INSTANCE_CONNECTION_NAMEは、GCP上で確認可能

## step2_SQL接続のためのプロキシサーバーをたてる
### 2.1download
linux64
```
wget https://dl.google.com/cloudsql/cloud_sql_proxy.linux.amd64 -O cloud_sql_proxy
```
mac64
```
curl -o cloud_sql_proxy https://dl.google.com/cloudsql/cloud_sql_proxy.darwin.amd64
```

### 2.2権限
```
chmod +x cloud_sql_proxy
```

### 2.3起動
```
./cloud_sql_proxy -instances=INSTANCE_CONNECTION_NAME=tcp:5432
```


## step3_optunaを実行する
```
import optuna

def objective(trial):
    x = trial.suggest_uniform('x', -10, 10)
    score = (x - 2) ** 2
    print('x: %1.3f, score: %1.3f' % (x, score))
    return score
storage = optuna.storages.RDBStorage(
    url='postgresql+psycopg2://DATABASE_USER:PASSWORD@localhost:5432/DBNAME',
)
study = optuna.create_study(study_name="testes",storage=storage, load_if_exists=True)
study.optimize(objective, n_trials=100)
```
- study_nameとload_if_existsを設定しないと、リスタートできないので注意！


## 参考
- GCP公式
  - https://cloud.google.com/sql/docs/postgres/connect-external-app?hl=ja#sqlalchemy-tcp
  - https://cloud.google.com/sql/docs/postgres/quickstart-proxy-test?hl=ja#debianubuntu
- optuna公式
  - https://optuna.readthedocs.io/en/stable/tutorial/20_recipes/001_rdb.html#sphx-glr-tutorial-20-recipes-001-rdb-py
  - https://optuna.readthedocs.io/en/stable/reference/generated/optuna.storages.RDBStorage.html#optuna.storages.RDBStorage
- SQLAlcemyについて
  - https://qiita.com/ariku/items/75799665acd09520bed2
