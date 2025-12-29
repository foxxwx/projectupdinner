# projectupdinner
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>宴会座位查询</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            background: linear-gradient(135deg, #6a11cb 0%, #2575fc 100%);
            min-height: 100vh;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
        }
        
        .container {
            background: white;
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            max-width: 500px;
            width: 100%;
        }
        
        .header {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .header h1 {
            color: #2c3e50;
            font-size: 28px;
            margin-bottom: 10px;
        }
        
        .header p {
            color: #7f8c8d;
            font-size: 16px;
        }
        
        .search-box {
            position: relative;
            margin-bottom: 20px;
        }
        
        .search-box input {
            width: 100%;
            padding: 15px 20px;
            border: 2px solid #e0e0e0;
            border-radius: 12px;
            font-size: 16px;
            transition: all 0.3s;
        }
        
        .search-box input:focus {
            outline: none;
            border-color: #3498db;
            box-shadow: 0 0 0 3px rgba(52, 152, 219, 0.1);
        }
        
        .result {
            background: #f8f9fa;
            border-radius: 12px;
            padding: 25px;
            text-align: center;
            min-height: 200px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }
        
        .table-number {
            font-size: 72px;
            font-weight: bold;
            color: #e74c3c;
            margin: 10px 0;
        }
        
        .hint {
            color: #7f8c8d;
            font-size: 14px;
            margin-top: 10px;
        }
        
        .table-layout {
            display: grid;
            grid-template-columns: repeat(5, 1fr);
            gap: 10px;
            margin-top: 20px;
        }
        
        .table-item {
            background: #f1f8ff;
            border-radius: 8px;
            padding: 10px;
            text-align: center;
            border: 2px solid #e3f2fd;
        }
        
        .table-item.active {
            background: #e3f2fd;
            border-color: #2196f3;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🎉 欢迎参加宴会</h1>
            <p>请输入您的姓名查询座位</p>
        </div>
        
        <div class="search-box">
            <input 
                type="text" 
                id="nameInput" 
                placeholder="请输入您的姓名（支持模糊搜索）"
                autocomplete="off"
            >
        </div>
        
        <div class="result" id="result">
            <p style="color: #7f8c8d;">等待输入姓名...</p>
        </div>
        
        <div class="hint" id="hint">
            已录入 100 位宾客信息，共 10 桌
        </div>
    </div>

    <script>
        // 模拟数据库 - 实际使用时可替换为真实数据
        const seatData = {
            "张三": 1, "李四": 1, "王五": 1, "赵六": 1, "钱七": 1, "孙八": 1, "周九": 1, "吴十": 1, "郑十一": 1, "王十二": 1,
            "陈明": 2, "刘芳": 2, "杨光": 2, "黄蓉": 2, "赵云": 2, "孙权": 2, "刘备": 2, "曹操": 2, "诸葛亮": 2, "司马懿": 2,
            "林平": 3, "周涛": 3, "吴刚": 3, "郑爽": 3, "王菲": 3, "刘烨": 4, "张艺": 4, "陈坤": 4, "杨幂": 4, "黄晓明": 4,
            "赵薇": 5, "范冰冰": 5, "李晨": 5, "邓超": 5, "孙俪": 5, "周迅": 6, "吴京": 6, "徐峥": 6, "沈腾": 6, "马丽": 6,
            "贾玲": 7, "岳云鹏": 7, "郭德纲": 7, "于谦": 7, "张云雷": 7, "杨九郎": 8, "孟鹤堂": 8, "周九良": 8, "秦霄贤": 8, "何九华": 8,
            "刘德华": 9, "张学友": 9, "郭富城": 9, "黎明": 9, "梁朝伟": 9, "周润发": 10, "成龙": 10, "李连杰": 10, "甄子丹": 10, "吴彦祖": 10
        };

        // 错别字容错映射
        const typoMap = {
            "张山": "张三", "李思": "李四", "王无": "王五", "赵陆": "赵六",
            "张3": "张三", "李4": "李四", "王5": "王五"
        };

        // 拼音支持
        const pinyinMap = {
            "zhangsan": "张三", "lisi": "李四", "wangwu": "王五",
            "zhaoliu": "赵六", "qianqi": "钱七"
        };

        function searchName() {
            const input = document.getElementById('nameInput');
            const resultDiv = document.getElementById('result');
            const hintDiv = document.getElementById('hint');
            let name = input.value.trim();
            
            if (!name) {
                resultDiv.innerHTML = '<p style="color: #7f8c8d;">请输入姓名查询</p>';
                return;
            }
            
            // 1. 精确匹配
            if (seatData[name]) {
                showResult(name, seatData[name]);
                return;
            }
            
            // 2. 错别字容错
            if (typoMap[name]) {
                const correctName = typoMap[name];
                showResult(correctName, seatData[correctName], true);
                return;
            }
            
            // 3. 拼音匹配
            if (pinyinMap[name.toLowerCase()]) {
                const correctName = pinyinMap[name.toLowerCase()];
                showResult(correctName, seatData[correctName], true);
                return;
            }
            
            // 4. 模糊匹配
            const matchedNames = Object.keys(seatData).filter(fullName => 
                fullName.includes(name) || 
                name.includes(fullName) ||
                getSimilarity(fullName, name) > 0.6
            );
            
            if (matchedNames.length === 1) {
                showResult(matchedNames[0], seatData[matchedNames[0]], true);
            } else if (matchedNames.length > 1) {
                resultDiv.innerHTML = `
                    <p style="color: #e74c3c; margin-bottom: 10px;">找到多个匹配：</p>
                    ${matchedNames.map(n => 
                        `<p style="margin: 5px 0; cursor: pointer; color: #3498db;" 
                          onclick="selectName('${n}')">${n} - 第 ${seatData[n]} 桌</p>`
                    ).join('')}
                `;
            } else {
                resultDiv.innerHTML = `
                    <p style="color: #e74c3c;">未找到 "${name}"</p>
                    <p style="color: #7f8c8d; font-size: 14px; margin-top: 10px;">
                        请检查姓名是否正确，或联系工作人员协助
                    </p>
                `;
            }
        }

        function showResult(name, table, isFuzzy = false) {
            const resultDiv = document.getElementById('result');
            const hint = isFuzzy ? `（已为您匹配到 ${name}）` : '';
            
            resultDiv.innerHTML = `
                <p style="color: #2c3e50; margin-bottom: 10px;">${name} 先生/女士${hint}</p>
                <div class="table-number">${table}</div>
                <p style="color: #27ae60; font-size: 18px; margin-top: 10px;">第 ${table} 桌</p>
                <p style="color: #7f8c8d; font-size: 14px; margin-top: 10px;">
                    请前往对应桌号就坐，祝您用餐愉快！
                </p>
            `;
        }

        function selectName(name) {
            document.getElementById('nameInput').value = name;
            searchName();
        }

        // 简单的字符串相似度计算
        function getSimilarity(s1, s2) {
            let longer = s1;
            let shorter = s2;
            if (s1.length < s2.length) {
                longer = s2;
                shorter = s1;
            }
            const longerLength = longer.length;
            if (longerLength === 0) return 1.0;
            return (longerLength - editDistance(longer, shorter)) / parseFloat(longerLength);
        }

        function editDistance(s1, s2) {
            s1 = s1.toLowerCase();
            s2 = s2.toLowerCase();
            const costs = [];
            for (let i = 0; i <= s1.length; i++) {
                let lastValue = i;
                for (let j = 0; j <= s2.length; j++) {
                    if (i === 0) {
                        costs[j] = j;
                    } else if (j > 0) {
                        let newValue = costs[j - 1];
                        if (s1.charAt(i - 1) !== s2.charAt(j - 1)) {
                            newValue = Math.min(Math.min(newValue, lastValue), costs[j]) + 1;
                        }
                        costs[j - 1] = lastValue;
                        lastValue = newValue;
                    }
                }
                if (i > 0) costs[s2.length] = lastValue;
            }
            return costs[s2.length];
        }

        // 实时搜索
        document.getElementById('nameInput').addEventListener('input', debounce(searchName, 300));
        
        function debounce(func, wait) {
            let timeout;
            return function executedFunction(...args) {
                const later = () => {
                    clearTimeout(timeout);
                    func(...args);
                };
                clearTimeout(timeout);
                timeout = setTimeout(later, wait);
            };
        }
    </script>
</body>
</html>
