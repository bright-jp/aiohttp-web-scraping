# PythonでAIOHTTPを使用したWebスクレイピング

[![Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/) 

本ガイドでは、PythonでAIOHTTPを使用してWebスクレイピングを行うための基本を解説します。

- [AIOHTTPとは？](#what-is-aiohttp)
- [AIOHTTPでスクレイピング：ステップバイステップチュートリアル](#scraping-with-aiohttp-step-by-step-tutorial)
  - [ステップ #1：スクレイピングプロジェクトのセットアップ](#step-1-setting-up-a-scraping-project)
  - [ステップ #2：スクレイピング用ライブラリのセットアップ](#step-2-setting-up-the-scraping-libraries)
  - [ステップ #3：ターゲットページのHTMLを取得する](#step-3-getting-the-html-of-the-target-page)
  - [ステップ #4：HTMLを解析する](#step-4-parsing-the-html)
  - [ステップ #5：データ抽出ロジックを書く](#step-5-writing-the-data-extraction-logic)
  - [ステップ #6：スクレイピングしたデータをエクスポートする](#step-6-exporting-the-scraped-data)
  - [ステップ #7：すべてをまとめる](#step-7-putting-it-all-together)
- [Webスクレイピング向けAIOHTTP：高度な機能とテクニック](#aiohttp-for-web-scraping-advanced-features-and-techniques)
  - [カスタムヘッダーの設定](#setting-custom-headers)
  - [カスタムUser Agentの設定](#setting-a-custom-user-agent)
  - [Cookieの設定](#setting-cookies)
  - [プロキシ連携](#proxy-integration)
  - [エラーハンドリング](#error-handling)
  - [失敗したリクエストのリトライ](#retrying-failed-requests)
- [WebスクレイピングにおけるAIOHTTPとRequestsの比較](#aiohttp-vs-requests-for-web-scraping)
- [結論](#conclusion)

## What Is AIOHTTP?

[AIOHTTP](https://docs.aiohttp.org/en/stable/) は、Pythonの [`asyncio`](https://docs.python.org/3/library/asyncio.html) ライブラリ上に構築された、非同期のクライアント/サーバーHTTPフレームワークです。従来のHTTPクライアントとは異なり、AIOHTTPはクライアントセッションを使用して複数リクエストにわたる接続を管理するため、高い同時接続性を必要とするセッションベースのタスクにおいて非常に効率的な選択肢です。


**⚙️ Features**

- HTTPプロトコルのクライアント実装とサーバー実装の両方をサポートします。  
- クライアント/サーバーの両方でWebSocketをネイティブにサポートします。  
- Webサーバー構築のためのミドルウェアとプラガブルなルーティングを提供します。  
- 大容量データのストリーミングを効率的に管理します。  
- クライアントセッションの永続化が含まれており、接続を再利用して複数リクエスト時のオーバーヘッドを最小化します。  


## Scraping with AIOHTTP: Step-By-Step Tutorial

Webスクレイピングの文脈では、AIOHTTPはページの生HTMLコンテンツを取得するためのHTTPクライアントに過ぎません。そのHTMLからデータを解析・抽出するには、[BeautifulSoup](https://www.crummy.com/software/BeautifulSoup/bs4/doc/) のようなHTMLパーサーが必要です。

> **Warning**:\
> AIOHTTPは主にプロセスの初期段階で利用されますが、本ガイドではスクレイピングのワークフロー全体を順を追って説明します。より高度なAIOHTTPのWebスクレイピングテクニックを探している場合は、ステップ3を完了した後に次の章へ進んでください。

### Step #1: Setting Up a Scraping Project

Python3+ をインストールし、AIOHTTPスクレイピングプロジェクト用のディレクトリを作成します。

```bash
mkdir aiohttp-scraper
```

そのディレクトリに移動し、[virtual environment](https://docs.python.org/3/library/venv.html) をセットアップします。

```bash
cd aiohttp-scraper
python -m venv env
```

お好みのPython IDEでプロジェクトフォルダを開き、プロジェクトフォルダ内に `scraper.py` という名前のファイルを作成します。


IDEのターミナルで、仮想環境を有効化します。LinuxまたはmacOSでは次を使用します。

```bash
./env/bin/activate
```

Windowsでは次を実行します。

```powershell
env/Scripts/activate
```

### Step #2: Setting Up the Scraping Libraries

AIOHTTPとBeautifulSoupをインストールします。

```bash
pip install aiohttp beautifulsoup4
```

インストールした [`aiohttp`](https://docs.aiohttp.org/en/stable/) と [`beautifulsoup4`](https://pypi.org/project/beautifulsoup4/) の依存関係を `scraper.py` スクリプトへインポートします。

```python
import asyncio
import aiohttp 
from bs4 import BeautifulSoup
```

> **Note**:\
> `aiohttp` が動作するには `asyncio` が必要です。

次に、以下の `async` 関数ワークフローを `scrper.py` ファイルに追加します。

```python
async def scrape_quotes():
    # Scraping logic...

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

`scrape_quotes()` は、スクレイピングロジックがブロックせずに並行実行される非同期関数を定義します。最後に、`asyncio.run(scrape_quotes())` が非同期関数を開始して実行します。

### Step #3: Getting the HTML of the Target Page

この例では、[“Quotes to Scrape”](https://quotes.toscrape.com/) サイトからデータをスクレイピングする方法を説明します。

![The target site](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737465124750_image.png)

RequestsやAIOHTTPのようなライブラリでは、GETリクエストを行うだけでページのHTMLコンテンツを直接取得できます。しかし、AIOHTTPは [異なるリクエストライフサイクル](https://docs.aiohttp.org/en/stable/http_request_lifecycle.html) で動作します。  

AIOHTTPの主要コンポーネントは [`ClientSession`](https://docs.aiohttp.org/en/stable/client_reference.html) で、接続プールを管理し、デフォルトで [`Keep-Alive`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Keep-Alive) をサポートします。各リクエストで新規接続を開くのではなく既存接続を再利用することで、パフォーマンスが向上します。

リクエストの一般的な流れは、主に以下の3ステップです。

1. `ClientSession()` でセッションを開きます。
2. [`session.get()`](https://docs.aiohttp.org/en/stable/client_reference.html#aiohttp.ClientSession.get) でGETリクエストを非同期に送信します。
3. `await response.text()` のようなメソッドでレスポンスデータへアクセスします。

この設計により、イベントループは処理の合間に異なる [`with` contexts](https://docs.python.org/3/reference/datamodel.html#context-managers) をブロックせずに利用できるため、高い同時接続性を必要とするタスクに最適です。

これを踏まえると、AIOHTTPを使って次のアプローチでホームページのHTMLを取得できます。

```python
async with aiohttp.ClientSession() as session:
    async with session.get("http://quotes.toscrape.com") as response:
        # Access the HTML of the target page
        html = await response.text()
```

内部では、AIOHTTPがサーバーへのリクエスト送信を処理し、ページのHTMLコンテンツを含むサーバーのレスポンスを待機します。レスポンスを受け取った後、`await response.text()` メソッドがHTMLコンテンツを文字列として取得します。

`html` 変数を出力すると、次のようになります。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Quotes to Scrape</title>
    <link rel="stylesheet" href="/static/bootstrap.min.css">
    <link rel="stylesheet" href="/static/main.css">
</head>
<body>
    <!-- omitted for brevity... -->
</body>
</html>
```

### Step #4: Parsing the HTML

HTMLコンテンツをBeautifulSoupコンストラクタに渡して解析します。

```python
# Parse the HTML content using BeautifulSoup
soup = BeautifulSoup(html, "html.parser")
```

[`html.parser`](https://docs.python.org/3/library/html.parser.html) は、コンテンツを処理するために使用されるデフォルトのPython HTMLパーサーです。

`soup` オブジェクトには解析済みHTMLが含まれており、必要なデータを抽出するためのメソッドが提供されます。  

### Step #5: Writing the Data Extraction Logic

以下のコードを使用して、ページから引用データをスクレイピングできます。

```python
# Where to store the scraped data
quotes = []

# Extract all quotes from the page
quote_elements = soup.find_all("div", class_="quote")

# Loop through quotes and extract text, author, and tags
for quote_element in quote_elements:
    text = quote_element.find("span", class_="text").get_text().get_text().replace("“", "").replace("”", "")
    author = quote_element.find("small", class_="author")
    tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

    # Store the scraped data
    quotes.append({
        "text": text,
        "author": author,
        "tags": tags
    })
```

このコードスニペットは、スクレイピングしたデータを格納するために `quotes` というリストを初期化します。次に、引用のHTML要素をすべて見つけ、それらを反復処理して引用文、著者、タグなどの詳細を抽出します。抽出された各引用は辞書として `quotes` リストに格納され、データを簡単に参照またはエクスポートできるように整理されます。

### Step #6: Exporting the Scraped Data

以下のコードを使用して、スクレイピングしたデータをCSVファイルにエクスポートできます。

```python
# Open the file for export
with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
    writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])
    
    # Write the header row
    writer.writeheader()
    
    # Write the scraped quotes data
    writer.writerows(quotes)
```

上記スニペットは、`quotes.csv` という名前のファイルを書き込みモードで開きます。次に、列ヘッダー（`text`、`author`、`tags`）を設定し、ヘッダーを書き込み、その後 `quotes` リスト内の各辞書をCSVファイルへ書き込みます。

[`csv.DictWriter`](https://docs.python.org/3/library/csv.html#csv.DictWriter) はデータの整形を簡素化し、構造化データを保存しやすくします。これを動作させるには、Python標準ライブラリから `csv` をインポートします。

```python
import csv
```

### Step #7: Putting It All Together

AIOHTTPによるWebスクレイピングスクリプトの完全版はこちらです。

```python
import asyncio
import aiohttp
from bs4 import BeautifulSoup
import csv

# Define an asynchronous function to make the HTTP GET request
async def scrape_quotes():
    async with aiohttp.ClientSession() as session:
        async with session.get("http://quotes.toscrape.com") as response:
            # Access the HTML of the target page
            html = await response.text()

            # Parse the HTML content using BeautifulSoup
            soup = BeautifulSoup(html, "html.parser")

            # List to store the scraped data
            quotes = []

            # Extract all quotes from the page
            quote_elements = soup.find_all("div", class_="quote")

            # Loop through quotes and extract text, author, and tags
            for quote_element in quote_elements:
                text = quote_element.find("span", class_="text").get_text().replace("“", "").replace("”", "")
                author = quote_element.find("small", class_="author").get_text()
                tags = [tag.get_text() for tag in quote_element.find_all("a", class_="tag")]

                # Store the scraped data
                quotes.append({
                    "text": text,
                    "author": author,
                    "tags": tags
                })

            # Open the file name for export
            with open("quotes.csv", mode="w", newline="", encoding="utf-8") as file:
                writer = csv.DictWriter(file, fieldnames=["text", "author", "tags"])

                # Write the header row
                writer.writeheader()

                # Write the scraped quotes data
                writer.writerows(quotes)

# Run the asynchronous function
asyncio.run(scrape_quotes())
```

次で実行できます。

```bash
python scraper.py
```

または、Linux/macOSでは次を使用します。

```bash
python3 scraper.py
```

プロジェクトのルートフォルダに `quotes.csv` ファイルが作成されます。開くと、次のようになります。

![The final quotes file](https://github.com/luminati-io/aiohttp-web-scraping/blob/main/Images/s_C07E0B72CB9153F9B6E6EF6B76FDCD439C9910ACC1C4E94E70E103EE716CD2E2_1737466185816_image.png)

## AIOHTTP for Web Scraping: Advanced Features and Techniques

以下の例では、ターゲットサイトとして [HTTPBin.io `/anything` endpoint](https://httpbin.io/anything) を使用します。このAPIは、リクエスト送信者が送ったIPアドレス、ヘッダー、その他のデータを返します。

### Setting Custom Headers

AIOHTTPのリクエストでは、`headers` 引数で [カスタムヘッダーを指定](https://docs.aiohttp.org/en/stable/client_advanced.html#custom-request-headers) できます。

```python
import aiohttp
import asyncio

async def fetch_with_custom_headers():
    # Custom headers for the request
    headers = {
        "Accept": "application/json",
        "Accept-Language": "en-US,en;q=0.9,fr-FR;q=0.8,fr;q=0.7,es-US;q=0.6,es;q=0.5,it-IT;q=0.4,it;q=0.3"
    }

    async with aiohttp.ClientSession() as session:
        # Make a GET request with custom headers
        async with session.get("https://httpbin.io/anything", headers=headers) as response:
            data = await response.json()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_headers())
```

この方法で、AIOHTTPは [`Accept`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept) と [`Accept-Language`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Language) ヘッダーを設定したGET HTTPリクエストを実行します。

### Setting a Custom User Agent

[`User-Agent`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/User-Agent) は、[Webスクレイピングにおける最重要HTTPヘッダー](https://brightdata.jp/blog/web-data/http-headers-for-web-scraping) の1つです。デフォルトでは、AIOHTTPは次の `User-Agent` を使用します。

```
Python/<PYTHON_VERSION> aiohttp/<AIOHTTP_VERSION>
```

上記のデフォルト値では、自動化スクリプトからのリクエストであることが容易に識別され、ターゲットサイトにブロックされる可能性が高まります。

検知される可能性を下げるために、先ほどと同様に現実世界のカスタム `User-Agent` を設定できます。

```python
import aiohttp
import asyncio

async def fetch_with_custom_user_agent():
    # Define a Chrome-like custom User-Agent
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/132.0.0.0 Safari/537.36"
    }

    async with aiohttp.ClientSession(headers=headers) as session:
        # Make a GET request with the custom User-Agent
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_user_agent())
```

### Setting Cookies

HTTPヘッダーと同様に、`ClientSession()` 内の `cookies` を使用して [カスタムCookieを設定](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#custom-cookies) できます。

```python
import aiohttp
import asyncio

async def fetch_with_custom_cookies():
    # Define cookies as a dictionary
    cookies = {
        "session_id": "9412d7hdsa16hbda4347dagb",
        "user_preferences": "dark_mode=false"
    }

    async with aiohttp.ClientSession(cookies=cookies) as session:
        # Make a GET request with custom cookies
        async with session.get("https://httpbin.io/anything") as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_with_custom_cookies())
```

Cookieを使用すると、Webスクレイピングのリクエストに不可欠なセッションデータを含められます。

> **Note**:\
> `ClientSession` に設定したCookieは、そのセッションで行われるすべてのリクエスト間で共有されます。セッションCookieにアクセスするには、[`ClientSession.cookie_jar`](https://docs.aiohttp.org/en/v3.7.3/client_reference.html#aiohttp.ClientSession.cookie_jar) を参照してください。

### Proxy Integration

AIOHTTPでは、IP BANのリスクを減らすために、プロキシサーバー経由でリクエストをルーティングできます。これを行うには、`session` のHTTPメソッド関数で [`proxy` 引数](https://docs.aiohttp.org/en/v3.7.3/client_advanced.html#proxy-support) を使用します。

```python
import aiohttp
import asyncio

async def fetch_through_proxy():
    # Replace with the URL of your proxy server
    proxy_url = "<YOUR_PROXY_URL>"

    async with aiohttp.ClientSession() as session:
        # Make a GET request through the proxy server
        async with session.get("https://httpbin.io/anything", proxy=proxy_url) as response:
            data = await response.text()
            # Handle the response...
            print(data)

# Run the event loop
asyncio.run(fetch_through_proxy())
```

### Error Handling

デフォルトでは、AIOHTTPは接続またはネットワークの問題に対してのみエラーを送出します。[`4xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses) および [`5xx`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#server_error_responses) のステータスコードを受信した際にHTTPレスポンスに対して例外を送出するには、以下のいずれかのアプローチを使用できます。

1. **`ClientSession` 作成時に `raise_for_status=True` を設定**：レスポンスステータスが `4xx` または `5xx` の場合に、そのセッション経由のすべてのリクエストで例外を自動的に送出します。
2. **リクエストメソッドへ直接 `raise_for_status=True` を渡す**：他のメソッドへ影響させず、個別のリクエストメソッド（`session.get()` や `session.post()` など）に対してエラー送出を有効にします。
3. **`response.raise_for_status()` を手動で呼び出す**：例外を送出するタイミングを完全に制御でき、リクエスト単位で判断できます。

Option #1 example:

```python
import aiohttp
import asyncio

async def fetch_with_session_error_handling():
    async with aiohttp.ClientSession(raise_for_status=True) as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                # No need to call response.raise_for_status(), as it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_session_error_handling())
```

セッションレベルで `raise_for_status=True` を設定すると、そのセッションを通じて行われるすべてのリクエストは、`4xx` または `5xx` のレスポンスに対して `aiohttp.ClientResponseError` を送出します。

Option #2 example:

```python
import aiohttp
import asyncio

async def fetch_with_raise_for_status():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything", raise_for_status=True) as response:
                # No need to manually call response.raise_for_status(), it is automatic
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_raise_for_status())
```

この場合、`raise_for_status=True` 引数が `session.get()` 呼び出しへ直接渡されています。これにより、`4xx` または `5xx` のステータスコードに対して例外が自動的に送出されます。

Option #3 example:

```python
import aiohttp
import asyncio

async def fetch_with_manual_error_handling():
    async with aiohttp.ClientSession() as session:
        try:
            async with session.get("https://httpbin.io/anything") as response:
                response.raise_for_status()  # Manually raises error for 4xx/5xx
                data = await response.text()
                print(data)
        except aiohttp.ClientResponseError as e:
            print(f"HTTP error occurred: {e.status} - {e.message}")
        except aiohttp.ClientError as e:
            print(f"Request error occurred: {e}")

# Run the event loop
asyncio.run(fetch_with_manual_error_handling())
```

個々のリクエストをより細かく制御したい場合は、リクエスト後に `response.raise_for_status()` を手動で呼び出せます。このアプローチでは、エラーを処理する正確なタイミングを決定できます。  


### Retrying Failed Requests

AIOHTTPには、リクエストの自動リトライを行うための組み込みサポートはありません。これを実装するには、カスタムロジック、または [`aiohttp-retry`](https://github.com/inyutin/aiohttp_retry) のようなサードパーティライブラリを使用する必要があります。これにより、失敗したリクエストに対するリトライロジックを設定でき、一時的なネットワーク問題、タイムアウト、レート制限に対処しやすくなります。

[`aiohttp-retry`](https://pypi.org/project/aiohttp-retry/) をインストールします。

```bash
pip install aiohttp-retry
```

コードで使用します。

```python
import asyncio
from aiohttp_retry import RetryClient, ExponentialRetry

async def main():
    retry_options = ExponentialRetry(attempts=1)
    retry_client = RetryClient(raise_for_status=False, retry_options=retry_options)
    async with retry_client.get("https://httpbin.io/anything") as response:
        print(response.status)
        
    await retry_client.close()
```

これにより、指数バックオフ戦略によるリトライ動作が設定されます。詳細は [official docs](https://github.com/inyutin/aiohttp_retry?tab=readme-ov-file#documentation) を参照してください。

## AIOHTTP vs Requests for Web Scraping

以下は、AIOHTTPと [WebスクレイピングにおけるRequests](https://brightdata.jp/blog/web-data/python-requests-guide) を比較するためのサマリーテーブルです。

| **Feature** | **AIOHTTP** | **Requests** |
| --- | --- | --- |
| **GitHub stars** | 15.3k | 52.4k |
| **Client support** | ✔️  | ✔️  |
| **Sync support** | ❌   | ✔️  |
| **Async support** | ✔️  | ❌   |
| **Server support** | ✔️  | ❌   |
| **Connection pooling** | ✔️  | ✔️  |
| **HTTP/2 support** | ❌   | ❌   |
| **User-agent customization** | ✔️  | ✔️  |
| **Proxy support** | ✔️  | ✔️  |
| **Cookie handling** | ✔️  | ✔️  |
| **Retry mechanism** | サードパーティライブラリ経由でのみ利用可能 | `HTTPAdapter`s 経由で利用可能 |
| **Performance** | 高 | 中 |
| **Community support and popularity** | 中 | 大 |

完全な比較については、[Requests vs HTTPX vs AIOHTTP](https://brightdata.jp/blog/web-data/requests-vs-httpx-vs-aiohttp) に関するブログ記事をご覧ください。

## Conclusion

AIOHTTPは、オンラインデータを収集するためにHTTPリクエストを行う高速で信頼性の高いツールです。ただし、自動化されたHTTPリクエストは公開IPアドレスを露出させる可能性があります。プライバシーとセキュリティを保護するために、Bright Dataのプロキシサーバーを使用してIPアドレスをマスクすることをご検討ください。

- [Datacenter proxies](https://brightdata.jp/proxy-types/datacenter-proxies) – 770,000以上のデータセンターIP。
- [Residential proxies](https://brightdata.jp/proxy-types/residential-proxies) – 195か国以上で72M以上のレジデンシャルIP。
- [ISP proxies](https://brightdata.jp/proxy-types/isp-proxies) – 700,000以上のISP IP。
- [Mobile proxies](https://brightdata.jp/proxy-types/mobile-proxies) – 7M以上のモバイルIP。

今すぐ無料のBright Dataアカウントを作成して、当社のプロキシおよびスクレイピングソリューションをテストしてください！