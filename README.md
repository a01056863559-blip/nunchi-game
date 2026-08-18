<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>3초 눈치 게임 - 업그레이드 버전</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        body {
            background-color: #121212;
            color: #ffffff;
            font-family: 'Apple SD Gothic Neo', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            overflow: hidden;
        }
        .game-container {
            width: 100%;
            max-width: 400px;
            height: 100vh;
            max-height: 800px;
            background: linear-gradient(135deg, #1e1e2f, #121212);
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            padding: 20px;
            position: relative;
            overflow: hidden;
        }
        /* 화면 흔들림 효과 애니메이션 */
        @keyframes shake {
            0% { transform: translate(0, 0); }
            20% { transform: translate(-10px, 5px); }
            40% { transform: translate(10px, -5px); }
            60% { transform: translate(-5px, -5px); }
            80% { transform: translate(5px, 5px); }
            100% { transform: translate(0, 0); }
        }
        .shake { animation: shake 0.4s ease; }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            font-size: 18px;
            font-weight: bold;
        }
        .stage-badge {
            background: #ff4757;
            padding: 5px 12px;
            border-radius: 15px;
        }
        .content-box {
            text-align: center;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 15px;
            padding: 30px 20px;
            margin: 20px 0;
        }
        .situation-title {
            font-size: 20px;
            margin-bottom: 15px;
            font-weight: bold;
            color: #ffa502;
        }
        /* 2D 캐릭터 연출 박스 */
        .character-art {
            font-size: 50px;
            margin-bottom: 10px;
            transition: transform 0.2s;
        }
        .gauge-wrapper {
            background: #333;
            height: 25px;
            border-radius: 15px;
            position: relative;
            overflow: hidden;
            margin-top: 20px;
        }
        .target-zone {
            position: absolute;
            height: 100%;
            background: rgba(46, 213, 115, 0.4);
            border-left: 2px dashed #2ed573;
            border-right: 2px dashed #2ed573;
        }
        .needle {
            position: absolute;
            width: 6px;
            height: 100%;
            background: #ff4757;
            top: 0;
            border-radius: 3px;
        }
        .action-btn {
            background: linear-gradient(135deg, #ff4757, #ff6b81);
            color: white;
            border: none;
            padding: 20px;
            font-size: 20px;
            font-weight: bold;
            border-radius: 15px;
            cursor: pointer;
            box-shadow: 0 5px 15px rgba(255, 71, 87, 0.4);
            transition: transform 0.1s;
        }
        .action-btn:active { transform: scale(0.95); }
        .feedback-text {
            font-size: 24px;
            font-weight: bold;
            text-align: center;
            height: 35px;
            margin-top: 10px;
        }
    </style>
</head>
<body>

<div class="game-container" id="gameContainer">
    <div class="header">
        <div class="stage-badge" id="levelText">LEVEL 1</div>
        <div id="scoreText">🪙 골드: 0</div>
    </div>

    <div class="content-box">
        <div class="character-art" id="charArt">👔</div>
        <div class="situation-title" id="situationText">"아, 이번 자료 언제 완성돼?"</div>
        
        <div class="gauge-wrapper">
            <div class="target-zone" id="targetZone"></div>
            <div class="needle" id="needle"></div>
        </div>
    </div>

    <div class="feedback-text" id="feedbackText">눈치를 빠르게 채세요!</div>

    <button class="action-btn" onclick="checkNunchi()">눈치 템포 잡기!</button>
</div>

<script>
    let level = 1;
    let gold = 0;
    let speed = 2;
    let needlePosition = 0;
    let direction = 1;
    let isPlaying = true;
    let targetLeft = 40;
    let targetWidth = 20;

    const situations = [
        "\"아, 이번 자료 언제 완성돼?\"",
        "\"퇴근하기 전에 이것 좀 보고 가~\"",
        "\"다들 회식 안 가고 뭐 하지?\"",
        "\"김 대리, 지금 나랑 눈 마주쳤지?\"",
        "\"이 프로젝트 방향이 좀 잘못된 것 같은데...\""
    ];

    const chars = ["👔", "💻", "☕", "😡", "📋"];

    function initStage() {
        targetWidth = Math.max(10, 25 - (level * 2));
        targetLeft = Math.floor(Math.random() * (70 - targetWidth)) + 10;
        
        let zone = document.getElementById('targetZone');
        zone.style.left = targetLeft + '%';
        zone.style.width = targetWidth + '%';

        document.getElementById('levelText').innerText = `LEVEL ${level}`;
        document.getElementById('situationText').innerText = situations[Math.floor(Math.random() * situations.length)];
        document.getElementById('charArt').innerText = chars[Math.floor(Math.random() * chars.length)];
        document.getElementById('feedbackText').innerText = "";
    }

    let animationId;
    function animateNeedle() {
        if (!isPlaying) return;
        
        needlePosition += speed * direction;
        if (needlePosition >= 98 || needlePosition <= 0) {
            direction *= -1;
        }
        
        document.getElementById('needle').style.left = needlePosition + '%';
        animationId = requestAnimationFrame(animateNeedle);
    }

    function checkNunchi() {
        if (!isPlaying) return;

        isPlaying = false;
        cancelAnimationFrame(animationId);

        let feedback = document.getElementById('feedbackText');
        let container = document.getElementById('gameContainer');

        if (needlePosition >= targetLeft && needlePosition <= (targetLeft + targetWidth)) {
            // 성공!
            feedback.style.color = "#2ed573";
            feedback.innerText = "🎉 눈치 100단 성공! 완벽하다!";
            gold += 100;
            level++;
            speed += 0.8;
            document.getElementById('scoreText').innerText = `🪙 골드: ${gold}`;
            
            container.classList.add('shake');
            setTimeout(() => container.classList.remove('shake'), 400);

            setTimeout(() => {
                isPlaying = true;
                initStage();
                animateNeedle();
            }, 1200);
        } else {
            // 실패!
            feedback.style.color = "#ff4757";
            feedback.innerText = "💥 눈치 상실! 대참사 발생!";
            
            container.classList.add('shake');
            setTimeout(() => container.classList.remove('shake'), 400);

            setTimeout(() => {
                level = 1;
                speed = 2;
                isPlaying = true;
                initStage();
                animateNeedle();
            }, 1500);
        }
    }

    // 게임 시작
    initStage();
    animateNeedle();
</script>

</body>
</html>
