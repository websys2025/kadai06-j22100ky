[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/J1qyflp_)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=19632760&assignment_repo_type=AssignmentRepo)
## 第6講目の演習課題
### 課題6-1．eStat-APIを使って、人口推計以外のデータを取得するプログラム kadai6-1.py を作成せよ。
* リファレンスを参照して、任意のデータを自分で選ぶこと。
* 作成したファイルは自分で追加すること。
* 取得したデータの種類、エンドポイントと機能、使い方などを、コード内にコメントで記述すること。
import requests
import pandas as pd

# ▼ アプリケーションIDを入力してください（e-Statのユーザー登録で取得）
APP_ID = "YOUR_APP_ID"

# ▼ e-Stat APIのエンドポイント（統計データ取得）
API_URL = "https://api.e-stat.go.jp/rest/3.0/app/json/getStatsData"

# ▼ 使用する統計表ID（労働力調査 - 完全失業率）
# 統計表ID「0003107567」は、総務省が公表する「労働力調査（基本集計）」の年次データ
params = {
    "appId": APP_ID,
    "statsDataId": "0003107567",   # 労働力調査（年次）
    "cdCat01": "000",              # 総数
    "metaGetFlg": "Y",
    "cntGetFlg": "N",
    "explanationGetFlg": "Y",
    "annotationGetFlg": "Y",
    "sectionHeaderFlg": "1",
    "lang": "J"
}

response = requests.get(API_URL, params=params)
data = response.json()

# ▼ 統計データ部分を抽出
values = data['GET_STATS_DATA']['STATISTICAL_DATA']['DATA_INF']['VALUE']
df = pd.DataFrame(values)

# ▼ メタ情報取得
meta_info = data['GET_STATS_DATA']['STATISTICAL_DATA']['CLASS_INF']['CLASS_OBJ']

# ▼ コードを日本語に変換
for class_obj in meta_info:
    col = '@' + class_obj['@id']
    id_to_name = {}
    class_list = class_obj['CLASS'] if isinstance(class_obj['CLASS'], list) else [class_obj['CLASS']]
    for c in class_list:
        id_to_name[c['@code']] = c['@name']
    df[col] = df[col].replace(id_to_name)

# ▼ 列名変換
col_replace = {'@unit': '単位', '$': '値'}
for class_obj in meta_info:
    col_replace['@' + class_obj['@id']] = class_obj['@name']
df.rename(columns=col_replace, inplace=True)

print(df.head())

# ---------------------------------------------
# 【データ概要】
# 統計名：労働力調査（総務省）
# 内容：全国の完全失業率や労働力人口など
# statsDataId：0003107567
# データ形式：JSON（e-Stat API）
# エンドポイント：https://api.e-stat.go.jp/rest/3.0/app/json/getStatsData
# ---------------------------------------------

### 課題6-2．世の中のオープンデータを調査し、そこから一つ選んで、データを取得するプログラム kadai6-2.py を作成せよ。
* 作成したファイルは自分で追加すること。。
* 参照するオープンデータの名前と概要、エンドポイントと機能、使い方などを、コード内にコメントで記述すること。
import requests
import pandas as pd

APP_ID = "ここに自分のアプリケーションIDを書く"
API_URL  = "https://api.e-stat.go.jp/rest/3.0/app/json/getStatsData"

params = {
    "appId": APP_ID,
    "statsDataId":"0000020201",
    "cdArea":"12101,12102,12103,12104,12105,12106",
    "cdCat01": "A1101",
    "metaGetFlg":"Y",
    "cntGetFlg":"N",
    "explanationGetFlg":"Y",
    "annotationGetFlg":"Y",
    "sectionHeaderFlg":"1",
    "replaceSpChars":"0",
    "lang": "J"  # 日本語を指定
}

response = requests.get(API_URL, params=params)
# Process the response
data = response.json()

# 統計データからデータ部取得
values = data['GET_STATS_DATA']['STATISTICAL_DATA']['DATA_INF']['VALUE']

# JSONからDataFrameを作成
df = pd.DataFrame(values)

# メタ情報取得
meta_info = data['GET_STATS_DATA']['STATISTICAL_DATA']['CLASS_INF']['CLASS_OBJ']

# 統計データのカテゴリ要素をID(数字の羅列)から、意味のある名称に変更する
for class_obj in meta_info:

    # メタ情報の「@id」の先頭に'@'を付与した文字列が、統計データの列名と対応している
    column_name = '@' + class_obj['@id']

    # 統計データの列名を「@code」から「@name」に置換するディクショナリを作成
    id_to_name_dict = {}
    if isinstance(class_obj['CLASS'], list):
        for obj in class_obj['CLASS']:
            id_to_name_dict[obj['@code']] = obj['@name']
    else:
        id_to_name_dict[class_obj['CLASS']['@code']] = class_obj['CLASS']['@name']

    # ディクショナリを用いて、指定した列の要素を置換
    df[column_name] = df[column_name].replace(id_to_name_dict)

# 統計データの列名を変換するためのディクショナリを作成
col_replace_dict = {'@unit': '単位', '$': '値'}
for class_obj in meta_info:
    org_col = '@' + class_obj['@id']
    new_col = class_obj['@name']
    col_replace_dict[org_col] = new_col

# ディクショナリに従って、列名を置換する
new_columns = []
for col in df.columns:
    if col in col_replace_dict:
        new_columns.append(col_replace_dict[col])
    else:
        new_columns.append(col)

df.columns = new_columns
print(df)
### 課題提出方法（期限：6/15）
* 随時、作業内容をコミットして、自分の課題リポジトリに履歴を残すこと。
* 上記の課題が完成したら、プルリクエストを使って提出すること。
* 教員、TA/SAが確認したらフィードバックする。
