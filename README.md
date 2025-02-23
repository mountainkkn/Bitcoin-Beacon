# Bitcoin-Beacon

bitcoin_dashboard/
├── static/ # CSS, JS 등 정적 파일
│ └── bootstrap.min.css # Bootstrap 파일 다운로드
├── templates/ # HTML 템플릿
│ └── index.html
├── app.py # 메인 Flask 앱
└── requirements.txt

2. 데이터 소스

거래소별 가격 비교 (김치프리미엄):
CoinGecko API: https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd,krw.
멤풀 모니터링, 수수료 추천:
Mempool.space API: https://mempool.space/api/v1/fees/recommended.
블록채굴 현황:
Blockchain.com API: https://blockchain.info/latestblock.

3. Flask 앱 설정 (app.py)

python
Wrap
Copy
from flask import Flask, render_template
import requests
import plotly.graph_objs as go
from plotly.subplots import make_subplots

app = Flask(**name**)

# 데이터 가져오기 함수

def get_price_data():
url = "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin&vs_currencies=usd,krw"
response = requests.get(url).json()
usd = response["bitcoin"]["usd"]
krw = response["bitcoin"]["krw"]
kimchi_premium = ((krw / usd - 1) \* 100)
return {"usd": usd, "krw": krw, "kimchi_premium": kimchi_premium}

def get_mempool_data():
url = "https://mempool.space/api/v1/fees/recommended"
return requests.get(url).json()

def get_block_data():
url = "https://blockchain.info/latestblock"
return requests.get(url).json()

# 메인 라우트

@app.route('/')
def dashboard():
prices = get_price_data()
mempool = get_mempool_data()
block = get_block_data()

    # Plotly 차트 (멤풀 수수료 예시)
    fig = go.Figure()
    fig.add_trace(go.Bar(
        x=["최소", "중간", "최대"],
        y=[mempool["economyFee"], mempool["hourFee"], mempool["fastestFee"]],
        name="수수료 (sat/vB)"
    ))
    graph_html = fig.to_html(full_html=False)

    return render_template('index.html', prices=prices, mempool=mempool, block=block, graph=graph_html)

if **name** == '**main**':
app.run(debug=True) 4. HTML 템플릿 (templates/index.html)

Bootstrap 먼저 다운로드해서 static/에 넣어 (https://getbootstrap.com/docs/5.3/getting-started/download/).
html
Wrap
Copy

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>비트코인 대시보드</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='bootstrap.min.css') }}">
</head>
<body>
    <div class="container mt-4">
        <h1 class="text-center mb-4">비트코인 대시보드</h1>
        
        <!-- 가격 비교 -->
        <div class="row">
            <div class="col-md-4">
                <div class="card mb-4">
                    <div class="card-body">
                        <h2 class="card-title">가격 비교</h2>
                        <p>USD: ${{ prices.usd }}</p>
                        <p>KRW: ₩{{ prices.krw }}</p>
                        <p>김치프리미엄: {{ "%.2f"|format(prices.kimchi_premium) }}%</p>
                    </div>
                </div>
            </div>

            <!-- 멤풀 모니터링 -->
            <div class="col-md-4">
                <div class="card mb-4">
                    <div class="card-body">
                        <h2 class="card-title">멤풀 수수료</h2>
                        <p>최소: {{ mempool.economyFee }} sat/vB</p>
                        <p>중간: {{ mempool.hourFee }} sat/vB</p>
                        <p>최대: {{ mempool.fastestFee }} sat/vB</p>
                        {{ graph|safe }}
                    </div>
                </div>
            </div>

            <!-- 블록 상태 -->
            <div class="col-md-4">
                <div class="card mb-4">
                    <div class="card-body">
                        <h2 class="card-title">최신 블록</h2>
                        <p>높이: {{ block.height }}</p>
                        <p>시간: {{ block.time }}</p>
                    </div>
                </div>
            </div>
        </div>
    </div>

</body>
</html>
5. 배포

Heroku로 배포:
requirements.txt 생성:
bash
Wrap
Copy
pip freeze > requirements.txt
Procfile 추가:
text
Wrap
Copy
web: gunicorn app:app
gunicorn 설치:
bash
Wrap
Copy
pip install gunicorn
Heroku CLI 설치 후:
bash
Wrap
Copy
heroku create
git init
git add .
git commit -m "Initial commit"
git push heroku main
학습 팁

Flask 기본: Flask 공식 문서 Quickstart (20분이면 충분).
Jinja2: HTML에 변수 넣는 법만 익히면 됨 ({{ 변수 }}).
Plotly: 공식 사이트 예제 보면서 차트 커스터마이징 연습.
Bootstrap: “Bootstrap 5 10분 강의” 유튜브 찾아봐.
