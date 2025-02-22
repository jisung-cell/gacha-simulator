# gacha-simulator
Flask-based recruitment simulation web app

from flask import Flask, request, jsonify
import random

app = Flask(__name__)

appearance_probabilities = {
    "일반": 30.8642, "희귀": 18.5185, "영웅": 12.3457, "전설": 9.2593, "신화": 1.8519,
    "골드 +200": 6.1728, "골드 +400": 3.0864, "골드 +600": 1.8519,
    "신화석 +1": 3.0864, "신화석 +2": 1.8519,
    "보석 +10": 6.1728, "보석 +20": 3.0864, "보석 +30": 1.8519
}

acquisition_probabilities = {
    "일반": 50, "희귀": 40, "영웅": 30, "전설": 20, "신화": 5,
    "골드 +200": 70, "골드 +400": 60, "골드 +600": 50,
    "신화석 +1": 40, "신화석 +2": 30,
    "보석 +10": 80, "보석 +20": 70, "보석 +30": 60
}

def draw_unit():
    items = list(appearance_probabilities.keys())
    weights = list(appearance_probabilities.values())
    return random.choices(items, weights)[0]

def recruit_multiple(times, multiplier=1):
    results = {}
    acquired_count = 0
    for _ in range(times):
        unit = draw_unit()
        if random.uniform(0, 100) <= acquisition_probabilities.get(unit, 100):
            results[unit] = results.get(unit, 0) + multiplier
            acquired_count += 1
    return results, acquired_count

def calculate_free_recruit(total_acquired_count):
    return min(max(total_acquired_count - 4, 0), 4)

@app.route('/')
def index():
    return """
    <h1>모집 시뮬레이터</h1>
    <p>초대장 개수를 입력하고 모집 버튼을 누르세요.</p>
    <form action="/recruit" method="post">
        초대장 개수: <input type="number" name="invitations" min="30" step="30" value="30"><br>
        <button type="submit" name="mode" value="1">일반 모집</button>
        <button type="submit" name="mode" value="2">10배 모집</button>
    </form>
    """

@app.route('/recruit', methods=['POST'])
def recruit():
    invitations = int(request.form.get('invitations', 0) or 0)
    mode = request.form.get('mode')

    required_invitations = 30 if mode == "1" else 300
    if invitations < required_invitations:
        return f"<p style='color:red;'>초대장이 {required_invitations}장 이상 필요합니다.</p>"

    times = (invitations // required_invitations) * 10
    multiplier = 10 if mode == "2" else 1

    results, acquired_count = recruit_multiple(times, multiplier)
    free_recruits = calculate_free_recruit(acquired_count)

    result_html = "<h2>모집 결과</h2><ul>"
    for item, count in results.items():
        result_html += f"<li>{item}: {count}</li>"
    result_html += f"</ul><p>총 획득 개수: {acquired_count}</p>"
    result_html += f"<p>무료 모집 횟수: {free_recruits}</p>"

    return result_html

if __name__ == "__main__":
    app.run(debug=True)