<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>アイヌ文化まとめテスト（顔認識アプリ）</title>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js" crossorigin="anonymous"></script>
    <script src="https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/face_mesh.js" crossorigin="anonymous"></script>
    <style>
        body { font-family: 'Hiragino Maru Gothic ProN', 'Comic Sans MS', sans-serif; background-color: #fdf6e3; margin: 0; padding: 20px; text-align: center; }
        h1 { color: #d33682; margin-bottom: 10px; }
        #game-board { display: none; max-width: 800px; margin: 0 auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        #video-container { position: relative; width: 320px; height: 240px; margin: 0 auto 20px; border-radius: 10px; overflow: hidden; transform: scaleX(-1); background-color: #eee; }
        video { width: 100%; height: 100%; object-fit: cover; }
        .question { font-size: 24px; font-weight: bold; margin-bottom: 20px; color: #073642; line-height: 1.4; }
        .options { display: flex; justify-content: center; gap: 20px; margin-bottom: 20px; }
        .option { flex: 1; padding: 15px; font-size: 20px; border-radius: 10px; border: 4px solid #eee; transition: all 0.2s; }
        .option-open { background-color: #ffe6e6; border-color: #ffb3b3; color: #cc0000; }
        .option-close { background-color: #e6f2ff; border-color: #b3d9ff; color: #0066cc; }
        .selected.option-open { border-color: #ff0000; box-shadow: 0 0 15px rgba(255,0,0,0.5); transform: scale(1.05); background-color: #ffcccc;}
        .selected.option-close { border-color: #0066cc; box-shadow: 0 0 15px rgba(0,102,204,0.5); transform: scale(1.05); background-color: #cce0ff;}
        #timer-container { font-size: 20px; font-weight: bold; color: #586e75; margin-bottom: 10px; }
        #timer { font-size: 36px; color: #dc322f; }
        #status { font-size: 16px; color: #586e75; margin-bottom: 15px; }
        #start-btn { font-size: 24px; padding: 15px 40px; background-color: #2aa198; color: white; border: none; border-radius: 30px; cursor: pointer; margin-top:20px; font-weight:bold;}
        #start-btn:hover { background-color: #258b82; }
        #result-overlay { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); color: white; font-size: 60px; font-weight: bold; align-items: center; justify-content: center; z-index: 100; flex-direction: column;}
        #result-sub { font-size: 30px; margin-top: 20px; }
        .emoji { font-size: 40px; margin-bottom: 10px; }

        /* 結果リストのデザイン */
        .result-list { text-align: left; font-size: 18px; line-height: 1.5; margin-top: 20px; max-height: 50vh; overflow-y: auto; padding-right: 10px; border-top: 2px dashed #ccc; padding-top: 20px;}
        .result-item { margin-bottom: 15px; padding-bottom: 10px; border-bottom: 1px solid #eee; page-break-inside: avoid; }
        .result-q { font-weight: bold; margin-bottom: 5px; color: #333; }
        .result-ans { color: #555; }
        .mark-correct { color: #dc322f; font-weight: bold; }
        .mark-incorrect { color: #268bd2; font-weight: bold; }

        /* 印刷用のスタイル設定 */
        @media print {
            body { background-color: white; padding: 0; margin: 0; }
            body * { visibility: hidden; }
            #print-area, #print-area * { visibility: visible; }
            #print-area { position: absolute; left: 0; top: 0; width: 100%; padding: 20px; border: none !important; box-shadow: none !important; margin: 0 !important; }
            .no-print { display: none !important; }
            .result-list { max-height: none; overflow: visible; border-top: 2px dashed #999; }
        }
    </style>
</head>
<body>
    <h1 class="no-print">アイヌ文化まとめテスト<br>〜お口でポン！〜</h1>
    <div id="start-screen" class="no-print">
        <p>元の3択問題を、<strong>2択問題</strong>にアレンジしました！<br>カメラを使って、顔の動きで答えるアプリです。</p>
        <div style="background: white; padding: 20px; border-radius: 10px; max-width: 500px; margin: 0 auto; text-align: left;">
            <strong>💡 あそびかた：</strong><br>
            問題と選択肢の読み上げが終わったあと、<strong>7秒以内</strong>に答えを選びます。<br><br>
            😮 <strong>左の答え</strong>だと思ったら：<br> 👉 <strong>「口を大きくあける」</strong><br><br>
            😐 <strong>右の答え</strong>だと思ったら：<br> 👉 <strong>「口をとじて待つ」</strong><br><br>
            🔊 <strong>問題の読み上げ機能つき！</strong><br>
            iPadの音量を出しておくと、問題文と選択肢を音声で読み上げます。
        </div>
        <button id="start-btn">スタート！</button>
    </div>

    <div id="game-board">
        <div id="status" class="no-print">カメラの準備中... 顔を画面にうつしてね！</div>
        <div id="video-container" class="no-print">
            <video id="video" autoplay playsinline></video>
        </div>
        <div id="timer-container" class="no-print">のこり <span id="timer">7.0</span> 秒</div>
        <div class="question no-print" id="question-text">ここに問題が出ます</div>
        <div class="options no-print">
            <div class="option option-open" id="opt-1">
                <div class="emoji">😮</div>
                口をあける<br><br><strong id="opt-1-text" style="font-size:24px;"></strong>
            </div>
            <div class="option option-close" id="opt-2">
                <div class="emoji">😐</div>
                口をとじる<br><br><strong id="opt-2-text" style="font-size:24px;"></strong>
            </div>
        </div>
    </div>

    <div id="result-overlay" class="no-print">
        <div id="result-main">⭕️ 正解！</div>
        <div id="result-sub"></div>
    </div>

    <script>
        const questions = [
            { q: "1. アイヌの言葉で「こんにちは」などのあいさつは？", o1: "イランカラㇷ゚テ", o2: "アロハ", ans: 1 },
            { q: "2. ヒグマのことを「山の神様」という意味で何と呼ぶ？", o1: "キムンカムイ", o2: "クマモン", ans: 1 },
            { q: "3. 木や草を使って建てられた伝統的な家は？", o1: "テント", o2: "チセ", ans: 2 },
            { q: "4. 口にくわえて、糸を引っ張って音を出す楽器は？", o1: "ムックリ", o2: "ギター", ans: 1 },
            { q: "5. 魔除けの意味があるデザインは？", o1: "水玉もよう", o2: "アイヌもよう", ans: 2 },
            { q: "6. 伝統的な服（アットゥㇱなど）は何の木の皮から作られる？", o1: "オヒョウ", o2: "サクラ", ans: 1 },
            { q: "7. 動物や自然など、大切なものを何と呼んで感謝した？", o1: "オバケ", o2: "カムイ", ans: 2 },
            { q: "8. サケや野菜を入れて塩味などで煮込んだ温かい汁物は？", o1: "オハウ", o2: "カレー", ans: 1 },
            { q: "9. みんなで集まって、歌ったり踊ったりするものを何という？", o1: "盆おどり", o2: "古式舞踊", ans: 2 },
            { q: "10. アイヌの人たちが古くからたくさん住んでいたのは？", o1: "北海道", o2: "沖縄", ans: 1 }
        ];

        let currentQIndex = 0;
        let isMouthOpen = false;
        let timer = 7.0;
        let timerInterval;
        let gameActive = false;
        let score = 0;
        let userResults = []; // 正解・不正解の記録用配列

        const videoElement = document.getElementById('video');
        const statusElement = document.getElementById('status');
        const opt1Element = document.getElementById('opt-1');
        const opt2Element = document.getElementById('opt-2');
        const timerElement = document.getElementById('timer');
        const resultOverlay = document.getElementById('result-overlay');
        const resultMain = document.getElementById('result-main');
        const resultSub = document.getElementById('result-sub');

        // 音声読み上げ用の関数
        function speakText(text, onEndCallback) {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel(); // 前の音声を止める
                
                // 「1. 」などの最初の数字を読み上げから省くと自然になります
                let speechText = text.replace(/^[0-9]+\.\s*/, '');
                
                const utterance = new SpeechSynthesisUtterance(speechText);
                utterance.lang = 'ja-JP';
                utterance.rate = 0.9; // 読み上げ速度をゆっくりに調整
                
                // 読み上げが終わった時、またはエラーになった時にタイマーを進める
                utterance.onend = function() {
                    if (onEndCallback) onEndCallback();
                };
                utterance.onerror = function() {
                    if (onEndCallback) onEndCallback();
                };
                
                window.speechSynthesis.speak(utterance);
            } else {
                // 音声読み上げが使えないブラウザの場合はすぐにコールバックを呼ぶ
                if (onEndCallback) onEndCallback();
            }
        }

        document.getElementById('start-btn').addEventListener('click', async () => {
            // iOS等で音声を出すための初期化処理（ダミー音声を再生）
            if ('speechSynthesis' in window) {
                const dummy = new SpeechSynthesisUtterance('');
                window.speechSynthesis.speak(dummy);
            }

            document.getElementById('start-screen').style.display = 'none';
            document.getElementById('game-board').style.display = 'block';
            await startCamera();
        });

        async function startCamera() {
            const faceMesh = new FaceMesh({locateFile: (file) => {
                return `https://cdn.jsdelivr.net/npm/@mediapipe/face_mesh/${file}`;
            }});
            faceMesh.setOptions({
                maxNumFaces: 1,
                refineLandmarks: true,
                minDetectionConfidence: 0.5,
                minTrackingConfidence: 0.5
            });
            faceMesh.onResults(onResults);

            const camera = new Camera(videoElement, {
                onFrame: async () => {
                    await faceMesh.send({image: videoElement});
                },
                width: 320,
                height: 240
            });
            camera.start();
        }

        function onResults(results) {
            if (results.multiFaceLandmarks && results.multiFaceLandmarks.length > 0) {
                statusElement.textContent = "🟢 顔を認識中！口を動かしてみてね。";
                const landmarks = results.multiFaceLandmarks[0];
                
                const upperLip = landmarks[13];
                const lowerLip = landmarks[14];
                const faceTop = landmarks[10];
                const faceBottom = landmarks[152];

                const lipDistance = Math.abs(lowerLip.y - upperLip.y);
                const faceHeight = Math.abs(faceBottom.y - faceTop.y);
                const ratio = lipDistance / faceHeight;

                isMouthOpen = ratio > 0.12; // 閾値を0.05から0.12に上げて、少し開いている状態を無視するように調整

                if (isMouthOpen) {
                    opt1Element.classList.add('selected');
                    opt2Element.classList.remove('selected');
                } else {
                    opt1Element.classList.remove('selected');
                    opt2Element.classList.add('selected');
                }

                if (!gameActive && currentQIndex < questions.length && !timerInterval) {
                    startGameLoop();
                }
            } else {
                statusElement.textContent = "⚠️ 顔が見つかりません。明るい場所でカメラを見てね。";
            }
        }

        function loadQuestion() {
            if (currentQIndex >= questions.length) {
                // 結果リストのHTMLを生成
                let resultListHTML = '<div class="result-list">';
                userResults.forEach(r => {
                    const markClass = r.isCorrect ? 'mark-correct' : 'mark-incorrect';
                    const markText = r.isCorrect ? '⭕️ 正解' : '❌ 不正解';
                    resultListHTML += `
                        <div class="result-item">
                            <div class="result-q">${r.qText}</div>
                            <div class="result-ans">
                                こたえ：<strong>${r.correctAnsText}</strong> 
                                （あなたの結果：<span class="${markClass}">${markText}</span>）
                            </div>
                        </div>
                    `;
                });
                resultListHTML += '</div>';

                // テスト終了画面の生成
                document.getElementById('game-board').innerHTML = `
                    <div id="print-area" style="padding: 40px; background: white; border: 2px solid #ccc; border-radius: 10px; margin-bottom: 20px;">
                        <h2 style="font-size:32px; color: #333; margin-top: 0;">アイヌ文化まとめテスト</h2>
                        <div style="text-align: left; font-size: 24px; margin: 20px 0; border-bottom: 2px dashed #999; padding-bottom: 10px;">
                            ＿＿年 ＿＿組 名前 ＿＿＿＿＿＿＿＿＿＿＿＿＿
                        </div>
                        <h3 style="font-size:28px; color: #333; margin-bottom: 10px;">結果発表</h3>
                        <p style="font-size:32px; margin-bottom: 0;">10問中 <span style="font-size:56px; font-weight:bold; color: #d33682;">${score}</span> 問 正解でした！🎉</p>
                        ${resultListHTML}
                    </div>
                    <div class="no-print">
                        <button onclick="window.print()" style="font-size: 20px; padding: 15px 30px; border-radius: 30px; cursor:pointer; background-color:#2aa198; color:white; border:none; margin: 10px; font-weight:bold; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">🖨️ 記録を印刷する</button>
                        <button onclick="location.reload()" style="font-size: 20px; padding: 15px 30px; border-radius: 30px; cursor:pointer; background-color:#cb4b16; color:white; border:none; margin: 10px; font-weight:bold; box-shadow: 0 4px 6px rgba(0,0,0,0.1);">もう一度あそぶ</button>
                    </div>
                `;
                return;
            }
            
            const q = questions[currentQIndex];
            document.getElementById('question-text').textContent = q.q;
            document.getElementById('opt-1-text').textContent = q.o1;
            document.getElementById('opt-2-text').textContent = q.o2;
            timer = 7.0;
            timerElement.textContent = timer.toFixed(1);
            
            // 読み上げ中はタイマーをストップしておく
            gameActive = false;
            document.getElementById('timer-container').style.color = "#93a1a1"; // 読み上げ中はグレーに
            
            // 問題文と選択肢を読み上げ、終わったらタイマースタート
            const textToSpeak = q.q + "。口をあける、" + q.o1 + "。口をとじる、" + q.o2;
            speakText(textToSpeak, () => {
                gameActive = true;
                document.getElementById('timer-container').style.color = "#586e75"; // 色を元に戻す
            });
        }

        function startGameLoop() {
            loadQuestion();
            timerInterval = setInterval(() => {
                if(!gameActive) return;
                timer -= 0.1;
                timerElement.textContent = Math.max(0, timer).toFixed(1);
                
                if (timer <= 0) {
                    gameActive = false;
                    checkAnswer();
                }
            }, 100);
        }

        function checkAnswer() {
            // 時間切れになったら音声を止める
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
            }

            const selectedAns = isMouthOpen ? 1 : 2;
            const q = questions[currentQIndex];
            const correctAns = q.ans;
            const isCorrect = (selectedAns === correctAns);
            const correctText = correctAns === 1 ? q.o1 : q.o2;

            // 結果を保存する
            userResults.push({
                qText: q.q,
                correctAnsText: correctText,
                isCorrect: isCorrect
            });
            
            resultOverlay.style.display = 'flex';
            if (isCorrect) {
                resultMain.textContent = "⭕️ 大正解！";
                resultOverlay.style.background = "rgba(42, 161, 152, 0.9)";
                score++;
            } else {
                resultMain.textContent = "❌ ざんねん！";
                resultOverlay.style.background = "rgba(220, 50, 47, 0.9)";
            }
            
            resultSub.textContent = `正解は「${correctText}」だよ！`;

            setTimeout(() => {
                resultOverlay.style.display = 'none';
                currentQIndex++;
                loadQuestion();
            }, 3000);
        }
    </script>
</body>
</html>
