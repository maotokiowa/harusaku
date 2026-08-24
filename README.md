<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>みんなの せんきょ・とうひょうクイズ（タッチ版）</title>
    <style>
        /* 📱 iPad・タブレット用スタイル設定 */
        * { box-sizing: border-box; touch-action: manipulation; }
        body { font-family: "Hiragino Sans", "Meiryo", sans-serif; text-align: center; background-color: #f0f4f8; margin: 0; padding: 20px; color: #333; user-select: none; -webkit-user-select: none; }
        
        .container { max-width: 900px; margin: 0 auto; }
        
        /* ❓ 問題文表示エリア */
        #question { font-size: 34px; font-weight: bold; margin: 20px 0; line-height: 1.5; padding: 20px; background: white; border-radius: 20px; box-shadow: 0 4px 10px rgba(0,0,0,0.08); min-height: 140px; display: flex; align-items: center; justify-content: center; }
        
        /* 🔘 タッチボタンのコンテナ */
        .choices-container { display: flex; justify-content: center; gap: 20px; margin-top: 30px; }
        
        /* 👆 タッチしやすい巨大ボタン */
        .touch-btn { 
            flex: 1;
            display: flex; 
            flex-direction: column;
            align-items: center; 
            justify-content: center;
            padding: 30px 15px; 
            font-size: 32px; 
            font-weight: bold; 
            border: 6px solid #1976d2; 
            border-radius: 25px; 
            background-color: #ffffff; 
            color: #1976d2;
            cursor: pointer;
            box-shadow: 0 8px 15px rgba(0,0,0,0.1);
            transition: transform 0.1s, background-color 0.2s;
            min-height: 180px;
            line-height: 1.3;
        }

        /* タッチした瞬間の凹むアニメーション */
        .touch-btn:active {
            transform: scale(0.96);
            background-color: #e3f2fd;
        }

        /* 正解・不正解時のボタンカラー */
        .touch-btn.correct-btn { background-color: #4caf50 !important; border-color: #2e7d32 !important; color: white !important; }
        .touch-btn.incorrect-btn { background-color: #f44336 !important; border-color: #c62828 !important; color: white !important; }
        .touch-btn:disabled { opacity: 0.7; cursor: default; }

        /* 💬 ナビゲーション・状態メッセージ */
        #message { font-size: 32px; margin-top: 25px; font-weight: bold; height: 45px; color: #ff5722; }

        /* 🚀 スタート画面ボタン */
        #start-btn { font-size: 38px; font-weight: bold; padding: 25px 60px; background-color: #ff5722; color: white; border: none; border-radius: 30px; cursor: pointer; box-shadow: 0 8px 20px rgba(255,87,34,0.3); margin-top: 50px; }
        #start-btn:active { transform: scale(0.95); }

        /* 🏆 結果発表・印刷用エリア */
        #result-section { display: none; background: white; padding: 30px; border-radius: 20px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); text-align: left; }
        .result-title { font-size: 40px; color: #ff5722; font-weight: bold; margin-bottom: 20px; text-align: center; }
        .result-score { font-size: 50px; color: #4caf50; font-weight: bold; margin-bottom: 10px; text-align: center; }
        .result-details { font-size: 26px; color: #555; margin-bottom: 30px; text-align: center; }
        
        .result-table { width: 100%; border-collapse: collapse; margin-top: 25px; font-size: 18px; }
        .result-table th, .result-table td { border: 2px solid #ccc; padding: 10px; text-align: center; }
        .result-table th { background-color: #f2f2f2; }
        .is-correct { color: #4caf50; font-weight: bold; }
        .is-incorrect { color: #f44336; font-weight: bold; background-color: #ffebee; }

        .print-btn-container { text-align: center; margin-top: 30px; }
        .print-btn { font-size: 26px; font-weight: bold; padding: 15px 40px; background-color: #00bcd4; color: white; border: none; border-radius: 15px; cursor: pointer; }

        @media print {
            body { background: white; color: black; padding: 0; }
            #question, .choices-container, #message, #start-btn, .print-btn-container { display: none !important; }
            #result-section { display: block !important; border: none; box-shadow: none; padding: 0; }
        }
    </style>
</head>
<body>

    <div class="container">
        <div id="question">👆 下のボタンを押してクイズをはじめてね！</div>
        
        <button id="start-btn" onclick="startQuiz()">タッチしてスタート！</button>

        <div class="choices-container" id="choicesSection" style="display: none;">
            <button class="touch-btn" id="btn0" onclick="checkAnswer(0)">-</button>
            <button class="touch-btn" id="btn1" onclick="checkAnswer(1)">-</button>
        </div>

        <div id="message"></div>

        <!-- 結果発表画面 -->
        <div id="result-section">
            <div class="result-title">🎉 クイズ お疲れ様でした！</div>
            <hr style="border: 1px solid #ccc; margin-bottom: 20px;">
            <div id="print-date" style="font-size: 20px; text-align: right; color: #666; margin-bottom: 20px;"></div>
            <div class="result-score" id="resRate">正答率：0％</div>
            <div class="result-details" id="resCount">0問 中 0問 せいかい！</div>
            
            <table class="result-table">
                <thead>
                    <tr>
                        <th style="width: 10%;">問題</th>
                        <th style="width: 40%;">問題の内容</th>
                        <th style="width: 20%;">選んだ答え</th>
                        <th style="width: 20%;">正しい答え</th>
                        <th style="width: 10%;">結果</th>
                    </tr>
                </thead>
                <tbody id="result-table-body">
                </tbody>
            </table>

            <div style="font-size: 22px; margin-top: 30px; text-align: left; line-height: 1.8;">
                【 実施記録 】<br>
                ・ お名前：___________________________<br>
                ・ 先生やスタッフからのコメント：<br>
                <div style="border-bottom: 1px dashed #999; margin-top: 35px;"></div>
                <div style="border-bottom: 1px dashed #999; margin-top: 35px;"></div>
            </div>
            
            <div class="print-btn-container">
                <button class="print-btn" onclick="window.print()">🖨️ 印刷・PDF保存する</button>
            </div>
        </div>
    </div>

    <script>
        // 全20問データ（かっこ表記なしの綺麗で読みやすいテキスト）
        const quizData = [
            { q: "おうちの人ではなく、投票所にいる2名のスタッフさんが手助けしてくれる特別なルールは？", c: ["代理投票", "おうち投票"], ans: 0 },
            { q: "代理投票のとき、受付でスタッフさんに何と言えばいいでしょう？", c: ["「かわりに書いてください」", "「代理投票おねがいします」"], ans: 1 },
            { q: "だれを応援したいかをスタッフさんに伝えるとき、どうすればよいでしょう？", c: ["指をさしたりお話をして伝える", "心のなかで強く念じる"], ans: 0 },
            { q: "スタッフさんが代わりに名前を書くとき、もうひとりのスタッフさんは何をしているでしょう？", c: ["おやつを食べている", "まちがいがないか見ている"], ans: 1 },
            { q: "スタッフさんに書いてもらったあと、さいごに投票箱へ紙を入れるのはだれでしょう？", c: ["じぶん", "スタッフさん"], ans: 0 },
            { q: "投票をするとき、守らなければいけない大切なお約束はどちらでしょう？", c: ["だれの紙が一番きれいか見せ合う", "ほかの人の紙を見たり邪魔をしない"], ans: 1 },
            { q: "大好きなアイドルやキャラクターを選ぶときは、どんな理由で選んでいいでしょう？", c: ["じぶんの「大すき！」で選ぶ", "みんなのための「やくそく」で選ぶ"], ans: 0 },
            { q: "「市長さん」などの、まちのリーダーを選ぶときは、何で選ぶのが正しいでしょう？", c: ["ダンスが上手かどうか", "みんなのための「やくそく」"], ans: 1 },
            { q: "まちのリーダーを選ぶということは、どういうことでしょう？", c: ["代わりに「こうなったらいいな」をやってくれる人を探す", "一番歌がうまい人を選ぶ"], ans: 0 },
            { q: "選挙で投票する人を選ぶとき、まず最初にすることは何でしょう？", c: ["直感で決める", "情報を集める"], ans: 1 },
            { q: "選挙で「えらぶ人」になれるのは、何歳以上のおとなの人たちでしょう？", c: ["18歳以上", "20歳以上"], ans: 0 },
            { q: "投票所入場券は、どこに届くでしょう？", c: ["市役所", "家"], ans: 1 },
            { q: "代理投票が必要な人は、どんな人でしょう？", c: ["手が痛かったり名前を書くのが難しい人", "ただ文字を書くのがめんどくさい人"], ans: 0 },
            { q: "選挙で投票数が同じだった場合、何で決めるでしょう？", c: ["くじ引き", "じゃんけん"], ans: 0 },
            { q: "「アイドル」を選ぶときは、何で選ぶでしょう？", c: ["じぶんの「好き」", "みんなのための「約束」"], ans: 0 },
            { q: "「市長さん」を選ぶときは、何で選ぶでしょう？", c: ["じぶんの「好き」", "みんなのための「約束」"], ans: 1 },
            { q: "みんなで何かを決めるとき、数が多いほうの意見にするきめ方を何というでしょう？", c: ["たすうけつ", "ジャンケン"], ans: 0 },
            { q: "代理投票のとき、投票所でお手伝いをしてくれるスタッフさんは何人でしょう？", c: ["10人", "2人"], ans: 1 },
            { q: "おうちの人に代わりに名前を書いてもらって投票することはできるでしょうか？", c: ["できない", "できる"], ans: 0 },
            { q: "自分の必要な支援を書いて、投票所に持って行くことができる紙を何というでしょう？", c: ["図書カード", "選挙支援カード"], ans: 1 }
        ];

        let currentId = 0;
        let isAnswering = false;
        let correctCount = 0;
        let userRecords = [];

        const qChange = document.getElementById("question");
        const btn0 = document.getElementById("btn0");
        const btn1 = document.getElementById("btn1");
        const msg = document.getElementById("message");
        const startBtn = document.getElementById("start-btn");
        const choicesSection = document.getElementById("choicesSection");

        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

        function cleanTextForSpeech(text) {
            if (!text) return "";
            return text.replace(/上手/g, 'じょうず');
        }

        function speakText(text, callback) {
            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                
                const cleanText = cleanTextForSpeech(text);
                const uttr = new SpeechSynthesisUtterance(cleanText);
                uttr.lang = 'ja-JP';
                uttr.rate = 0.8; // 🐢 ゆっくりめ

                uttr.onend = () => { if (callback) callback(); };
                uttr.onerror = () => { if (callback) callback(); };
                window.speechSynthesis.speak(uttr);
            } else {
                if (callback) callback();
            }
        }

        function playSound(type) {
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);

            if (type === 'correct') {
                osc.frequency.setValueAtTime(523.25, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.15);
                setTimeout(() => {
                    const osc2 = audioCtx.createOscillator();
                    const gain2 = audioCtx.createGain();
                    osc2.connect(gain2);
                    gain2.connect(audioCtx.destination);
                    osc2.frequency.setValueAtTime(659.25, audioCtx.currentTime);
                    gain2.gain.setValueAtTime(0.15, audioCtx.currentTime);
                    osc2.start();
                    osc2.stop(audioCtx.currentTime + 0.3);
                }, 150);
            } else if (type === 'incorrect') {
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(130, audioCtx.currentTime);
                gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.4);
            }
        }

        function startQuiz() {
            if (audioCtx.state === 'suspended') { audioCtx.resume(); }
            startBtn.style.display = "none";
            choicesSection.style.display = "flex";
            loadQuestion(true);
        }

        function loadQuestion(isFirst = false) {
            if (currentId >= quizData.length) {
                showResults();
                return;
            }

            // ボタンリセット
            btn0.className = "touch-btn";
            btn1.className = "touch-btn";
            btn0.disabled = false;
            btn1.disabled = false;

            qChange.innerText = `第${currentId + 1}問：` + quizData[currentId].q;
            btn0.innerText = quizData[currentId].c[0];
            btn1.innerText = quizData[currentId].c[1];
            
            msg.innerText = "問題を聞いてね ⏳";
            msg.style.color = "#888";
            isAnswering = true;

            const speechQuestion = `だい ${currentId + 1} もん。 ${quizData[currentId].q}`;
            const speechChoices = `${quizData[currentId].c[0]}。 または、 ${quizData[currentId].c[1]}。`;
            
            // 1. 問題文を読み上げ
            speakText(speechQuestion, () => {
                // 2. 1秒の間（ポーズ）
                setTimeout(() => {
                    // 3. 選択肢を読み上げ
                    speakText(speechChoices, () => {
                        msg.innerText = "ボタンをタッチしてね！";
                        msg.style.color = "#ff5722";
                        isAnswering = false;
                    });
                }, 1000);
            });
        }

        function checkAnswer(selected) {
            if (isAnswering) return; 
            isAnswering = true;
            window.speechSynthesis.cancel(); // 読み上げ中断

            btn0.disabled = true;
            btn1.disabled = true;

            const currentQ = quizData[currentId];
            const isCorrect = (selected === currentQ.ans);

            userRecords.push({
                id: currentId + 1,
                question: currentQ.q, 
                userAns: currentQ.c[selected],
                correctAns: currentQ.c[currentQ.ans],
                isCorrect: isCorrect
            });

            if (selected === 0) {
                btn0.classList.add(isCorrect ? "correct-btn" : "incorrect-btn");
            } else {
                btn1.classList.add(isCorrect ? "correct-btn" : "incorrect-btn");
            }

            if (isCorrect) {
                msg.innerText = "せいかい！ ⭕️";
                msg.style.color = "#4caf50";
                correctCount++;
                playSound('correct');
                speakText("せいかい！");
            } else {
                msg.innerText = "ざんねん！ ❌";
                msg.style.color = "#f44336";
                playSound('incorrect');
                speakText("ざんねん！");
            }

            setTimeout(() => {
                currentId++;
                loadQuestion();
            }, 2200);
        }

        function showResults() {
            qChange.style.display = "none";
            choicesSection.style.display = "none";
            msg.style.display = "none";

            let rate = Math.round((correctCount / quizData.length) * 100);
            
            const now = new Date();
            document.getElementById("print-date").innerText = `実施日：${now.getFullYear()}年${now.getMonth()+1}月${now.getDate()}日`;
            document.getElementById("resRate").innerText = `正答率：${rate}％`;
            document.getElementById("resCount").innerText = `${quizData.length}問 中 ${correctCount}問 せいかい！`;
            
            const tableBody = document.getElementById("result-table-body");
            tableBody.innerHTML = ""; 
            userRecords.forEach(rec => {
                const row = document.createElement("tr");
                row.innerHTML = `
                    <td>${rec.id}</td>
                    <td style="font-size:16px; font-weight:bold; text-align:left;">${rec.question}</td>
                    <td>${rec.userAns}</td>
                    <td>${rec.correctAns}</td>
                    <td class="${rec.isCorrect ? 'is-correct' : 'is-incorrect'}">${rec.isCorrect ? '〇' : '×'}</td>
                `;
                tableBody.appendChild(row);
            });
            
            document.getElementById("result-section").style.display = "block";
            speakText(`クイズおわり！よくがんばったね。正答率は、${rate}パーセントでした！`);
        }
    </script>
</body>
</html>
