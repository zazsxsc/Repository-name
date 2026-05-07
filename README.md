<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能推演助手 - 8大算法</title>
    <script src="https://kit.fontawesome.com/a076d05399.js" crossorigin="anonymous"></script>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { font-family: 'Segoe UI', sans-serif; background: #f5f7fa; color: #333; }
        
        .container { max-width: 600px; margin: 0 auto; padding: 20px; }
        
        /* 头部 */
        .header { 
            background: linear-gradient(135deg, #2c3e50, #4a6491); 
            color: white; padding: 20px; border-radius: 15px; 
            margin-bottom: 20px; text-align: center; 
        }
        .header h1 { font-size: 1.5rem; margin-bottom: 5px; }
        .header p { opacity: 0.9; font-size: 0.9rem; }
        
        /* 算法选择 */
        .algo-selector { margin-bottom: 20px; }
        .algo-buttons { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 10px; }
        .algo-btn { 
            padding: 8px 12px; border: 2px solid #4a6491; border-radius: 8px; 
            background: white; color: #4a6491; font-size: 0.9rem; cursor: pointer; 
        }
        .algo-btn.active { background: #4a6491; color: white; }
        
        /* 卡片 */
        .card { 
            background: white; border-radius: 12px; padding: 20px; 
            margin-bottom: 20px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); 
        }
        .card-title { font-size: 1.1rem; color: #2c3e50; margin-bottom: 15px; }
        
        /* 输入框 */
        .textarea { 
            width: 100%; min-height: 120px; padding: 12px; border: 2px solid #e0e0e0; 
            border-radius: 8px; font-family: monospace; font-size: 0.9rem; 
            margin-bottom: 15px; resize: vertical; 
        }
        
        /* 按钮 */
        .btn { 
            width: 100%; padding: 15px; background: #4a6491; color: white; 
            border: none; border-radius: 8px; font-size: 1rem; font-weight: 600; 
            cursor: pointer; margin-top: 10px; 
        }
        .btn:hover { background: #2c3e50; }
        
        /* 结果 */
        .result { display: none; background: #f8f9fa; padding: 20px; border-radius: 10px; }
        .combo { margin: 20px 0; }
        .numbers { display: flex; justify-content: center; gap: 10px; margin: 15px 0; }
        .number { 
            width: 50px; height: 50px; background: #4a6491; color: white; 
            border-radius: 50%; display: flex; align-items: center; 
            justify-content: center; font-size: 1.5rem; font-weight: bold; 
        }
        .prop { 
            display: inline-block; background: #e9ecef; padding: 6px 12px; 
            border-radius: 20px; font-size: 0.9rem; margin: 0 5px; 
        }
        
        /* 热码 */
        .hot-cold { display: flex; justify-content: space-between; margin: 20px 0; }
        .digit { 
            display: inline-block; width: 30px; height: 30px; background: #FF6B6B; 
            color: white; border-radius: 50%; text-align: center; line-height: 30px; 
            margin: 0 2px; 
        }
        .cold-digit { background: #4A90E2; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>智能推演助手</h1>
            <p>8大算法 · 实时分析</p>
        </div>
        
        <div class="algo-selector">
            <div class="algo-buttons" id="algoButtons"></div>
        </div>
        
        <div class="card">
            <div class="card-title">历史数据</div>
            <textarea id="historyData" class="textarea" placeholder="输入历史开奖数据，格式：数字-数字-数字=和值/属性/形态
示例：
7-3-3=13/小单/对子
6-2-2=10/小双/对子
9-0-1=10/小双/杂六">7-3-3=13/小单/对子
6-2-2=10/小双/对子
9-0-1=10/小双/杂六
4-8-6=18/大双/杂六
7-7-3=17/大单/对子
2-0-8=10/小双/杂六
9-2-3=14/大双/杂六</textarea>
            <button class="btn" onclick="analyze()">开始智能推演</button>
        </div>
        
        <div class="hot-cold">
            <div>热码: <span id="hotDigits">7 3 8</span></div>
            <div>冷码: <span id="coldDigits">0 1 5</span></div>
        </div>
        
        <div class="result" id="result">
            <div class="card-title">推演结果</div>
            
            <div class="combo">
                <div style="color: #FF6B6B; font-weight: 600; margin-bottom: 10px;">
                    组合A - 对子延续策略
                </div>
                <div class="numbers" id="comboA"></div>
                <div id="propsA"></div>
            </div>
            
            <div class="combo">
                <div style="color: #4CD964; font-weight: 600; margin-bottom: 10px;">
                    组合B - 大单过渡策略
                </div>
                <div class="numbers" id="comboB"></div>
                <div id="propsB"></div>
            </div>
            
            <button class="btn" style="background: #4CD964;" onclick="copyResult()">复制结果</button>
        </div>
    </div>

    <script>
        // 算法定义
        const algorithms = [
            { id: 1, name: '热码跟踪', color: '#FF6B6B' },
            { id: 2, name: '冷热转换', color: '#4A90E2' },
            { id: 3, name: '形态延续', color: '#FF9800' },
            { id: 4, name: '区间跳变', color: '#9C27B0' },
            { id: 5, name: '奇偶平衡', color: '#2196F3' },
            { id: 6, name: '龙虎合', color: '#4CAF50' },
            { id: 7, name: '边缘中心', color: '#795548' },
            { id: 8, name: '智能混合', color: '#607D8B' }
        ];

        let currentAlgo = algorithms[0];

        // 初始化算法按钮
        function initAlgoButtons() {
            const container = document.getElementById('algoButtons');
            container.innerHTML = algorithms.map(algo => `
                <button class="algo-btn ${algo.id === 1 ? 'active' : ''}" 
                        onclick="selectAlgo(${algo.id})"
                        style="border-color: ${algo.color}; color: ${algo.color}; 
                               ${algo.id === 1 ? `background: ${algo.color}; color: white` : ''}">
                    ${algo.name}
                </button>
            `).join('');
        }

        function selectAlgo(id) {
            currentAlgo = algorithms.find(a => a.id === id) || algorithms[0];
            initAlgoButtons();
            alert('已切换到: ' + currentAlgo.name);
        }

        // 核心算法
        function analyze() {
            const input = document.getElementById('historyData').value;
            const data = parseData(input);
            
            if (data.length < 3) {
                alert('需要至少3期数据');
                return;
            }
            
            // 分析热码冷码
            const { hot, cold } = analyzeDigits(data);
            updateHotCold(hot, cold);
            
            // 根据算法生成组合
            let comboA, comboB;
            switch(currentAlgo.id) {
                case 1: // 热码跟踪
                    comboA = [hot[0], hot[0], hot[1] || hot[0]];
                    comboB = [hot[0], hot[1] || hot[0], hot[2] || hot[1] || hot[0]];
                    break;
                case 2: // 冷热转换
                    comboA = [cold[0], cold[1], cold[2]];
                    comboB = [cold[0], hot[0], hot[1] || hot[0]];
                    break;
                case 3: // 形态延续
                    const lastPattern = data[data.length-1].pattern;
                    if (lastPattern === '对子') {
                        const d = Math.floor(Math.random()*10);
                        comboA = [d, d, (d+1)%10];
                        comboB = [(d+2)%10, (d+3)%10, (d+4)%10];
                    } else {
                        comboA = [1,2,3];
                        comboB = [4,5,6];
                    }
                    break;
                case 4: // 区间跳变
                    const sums = data.map(d => d.sum);
                    const lastSum = sums[sums.length-1];
                    if (lastSum <= 13) {
                        comboA = [7,7,3]; // 大单
                        comboB = [8,8,2]; // 大双
                    } else {
                        comboA = [1,2,3]; // 小单
                        comboB = [0,2,4]; // 小双
                    }
                    break;
                case 5: // 奇偶平衡
                    comboA = [1,3,5]; // 全奇
                    comboB = [0,2,4]; // 全偶
                    break;
                case 6: // 龙虎合
                    comboA = [9,1,2]; // 龙
                    comboB = [2,1,9]; // 虎
                    break;
                case 7: // 边缘中心
                    comboA = [0,0,0]; // 极小
                    comboB = [9,9,9]; // 极大
                    break;
                case 8: // 智能混合
                    comboA = [hot[0], hot[1] || hot[0], hot[2] || hot[1] || hot[0]];
                    comboB = [cold[0], hot[0], Math.floor(Math.random()*10)];
                    break;
                default:
                    comboA = [7,7,4];
                    comboB = [9,6,2];
            }
            
            // 显示结果
            displayResult(comboA, comboB);
        }

        function parseData(input) {
            return input.trim().split('\n')
                .map(line => {
                    const parts = line.split('=')[0].split('-').map(Number);
                    return {
                        numbers: parts,
                        sum: parts.reduce((a,b) => a+b, 0)
                    };
                });
        }

        function analyzeDigits(data) {
            const digitCount = Array(10).fill(0);
            data.forEach(d => d.numbers.forEach(n => digitCount[n]++));
            
            const hot = digitCount.map((c,d) => ({d,c}))
                .sort((a,b) => b.c - a.c)
                .slice(0,3)
                .map(x => x.d);
            
            const cold = digitCount.map((c,d) => ({d,c}))
                .sort((a,b) => a.c - b.c)
                .slice(0,3)
                .map(x => x.d);
                
            return { hot, cold };
        }

        function updateHotCold(hot, cold) {
            document.getElementById('hotDigits').innerHTML = 
                hot.map(d => `<span class="digit">${d}</span>`).join('');
            document.getElementById('coldDigits').innerHTML = 
                cold.map(d => `<span class="digit cold-digit">${d}</span>`).join('');
        }

        function displayResult(comboA, comboB) {
            // 显示组合A
            document.getElementById('comboA').innerHTML = comboA.map(n => 
                `<div class="number" style="background: ${currentAlgo.color}">${n}</div>`
            ).join(' + ');
            
            const propsA = getProperties(comboA);
            document.getElementById('propsA').innerHTML = `
                和值: <span class="prop">${propsA.sum}</span>
                属性: <span class="prop">${propsA.sizeSingle}</span>
                形态: <span class="prop">${propsA.pattern}</span>
            `;
            
            // 显示组合B
            document.getElementById('comboB').innerHTML = comboB.map(n => 
                `<div class="number" style="background: #4CD964">${n}</div>`
            ).join(' + ');
            
            const propsB = getProperties(comboB);
            document.getElementById('propsB').innerHTML = `
                和值: <span class="prop">${propsB.sum}</span>
                属性: <span class="prop">${propsB.sizeSingle}</span>
                形态: <span class="prop">${propsB.pattern}</span>
            `;
            
            // 显示结果区域
            document.getElementById('result').style.display = 'block';
        }

        function getProperties(combo) {
            const sum = combo.reduce((a,b) => a+b, 0);
            const isSmall = sum <= 13;
            const isEven = sum % 2 === 0;
            
            const [a,b,c] = [...combo].sort((x,y) => x-y);
            let pattern = '杂六';
            if (a === b && b === c) pattern = '豹子';
            else if (a === b || b === c || a === c) pattern = '对子';
            else if (c-a === 2 && b-a === 1) pattern = '顺子';
            
            return {
                sum,
                sizeSingle: (isSmall ? '小' : '大') + (isEven ? '双' : '单'),
                pattern
            };
        }

        function copyResult() {
            const comboA = document.getElementById('comboA').textContent;
            const comboB = document.getElementById('comboB').textContent;
            const text = `智能推演结果\n组合A: ${comboA}\n组合B: ${comboB}`;
            
            navigator.clipboard.writeText(text)
                .then(() => alert('结果已复制!'))
                .catch(() => alert('复制失败'));
        }

        // 初始化
        document.addEventListener('DOMContentLoaded', function() {
            initAlgoButtons();
            // 默认推演一次
            setTimeout(() => {
                analyze();
                document.getElementById('result').style.display = 'block';
            }, 100);
        });
    </script>
</body>
</html>
