<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>贪吃蛇小游戏</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    
    <!-- Tailwind 配置 -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#4ade80',
                        secondary: '#166534',
                        accent: '#f59e0b',
                        dark: '#1e293b',
                        light: '#f8fafc'
                    },
                    fontFamily: {
                        game: ['"Press Start 2P"', 'cursive', 'sans-serif']
                    }
                }
            }
        }
    </script>
    
    <style type="text/tailwindcss">
        @layer utilities {
            .text-shadow {
                text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
            }
            .game-shadow {
                box-shadow: 0 0 20px rgba(74, 222, 128, 0.5);
            }
            .btn-hover {
                @apply transition-all duration-300 hover:scale-105 active:scale-95;
            }
            .pixel-corners {
                clip-path: polygon(
                    0% 5px, 5px 5px, 5px 0%, calc(100% - 5px) 0%, calc(100% - 5px) 5px, 
                    100% 5px, 100% calc(100% - 5px), calc(100% - 5px) calc(100% - 5px), 
                    calc(100% - 5px) 100%, 5px 100%, 5px calc(100% - 5px), 0% calc(100% - 5px)
                );
            }
        }
    </style>
    
    <!-- 导入游戏字体 -->
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
</head>
<body class="bg-gradient-to-br from-dark to-slate-800 min-h-screen flex flex-col items-center justify-center p-4 text-light font-sans">
    <div class="max-w-4xl w-full mx-auto flex flex-col items-center">
        <!-- 游戏标题 -->
        <h1 class="text-[clamp(1.5rem,5vw,2.5rem)] font-game text-primary text-center mb-6 text-shadow animate-pulse">
            贪吃蛇 <i class="fa fa-gamepad ml-2"></i>
        </h1>
        
        <!-- 游戏信息面板 -->
        <div class="w-full flex flex-col sm:flex-row justify-between items-center gap-4 mb-4 bg-slate-800/50 p-4 rounded-lg backdrop-blur-sm border border-primary/20">
            <div class="flex gap-6">
                <div class="text-center">
                    <p class="text-sm text-slate-300">当前得分</p>
                    <p id="score" class="text-2xl font-bold text-primary">0</p>
                </div>
                <div class="text-center">
                    <p class="text-sm text-slate-300">最高分</p>
                    <p id="highScore" class="text-2xl font-bold text-accent">0</p>
                </div>
            </div>
            
            <div class="flex gap-3">
                <button id="startBtn" class="bg-primary text-dark px-4 py-2 rounded-md font-bold btn-hover flex items-center">
                    <i class="fa fa-play mr-2"></i> 开始
                </button>
                <button id="pauseBtn" class="bg-slate-600 text-light px-4 py-2 rounded-md font-bold btn-hover flex items-center" disabled>
                    <i class="fa fa-pause mr-2"></i> 暂停
                </button>
                <button id="restartBtn" class="bg-accent text-dark px-4 py-2 rounded-md font-bold btn-hover flex items-center" disabled>
                    <i class="fa fa-refresh mr-2"></i> 重置
                </button>
            </div>
        </div>
        
        <!-- 游戏容器 -->
        <div class="relative w-full aspect-square max-w-lg mx-auto game-shadow rounded-lg overflow-hidden bg-slate-900 border-2 border-primary/50">
            <!-- 游戏画布 -->
            <canvas id="gameCanvas" class="w-full h-full"></canvas>
            
            <!-- 开始屏幕 -->
            <div id="startScreen" class="absolute inset-0 bg-dark/80 backdrop-blur-sm flex flex-col items-center justify-center gap-6 z-10">
                <h2 class="text-2xl font-game text-primary text-center">准备好了吗？</h2>
                <p class="text-slate-300 text-center max-w-xs">使用方向键控制蛇的移动，吃到食物得分，撞到墙壁或自己则游戏结束。</p>
                <button id="startScreenBtn" class="bg-primary text-dark px-6 py-3 rounded-md font-bold text-lg btn-hover pixel-corners">
                    开始游戏
                </button>
            </div>
            
            <!-- 游戏结束屏幕 -->
            <div id="gameOverScreen" class="absolute inset-0 bg-dark/80 backdrop-blur-sm flex flex-col items-center justify-center gap-6 z-10 hidden">
                <h2 class="text-2xl font-game text-red-500 text-center">游戏结束！</h2>
                <p class="text-xl text-slate-300">你的得分: <span id="finalScore" class="text-primary font-bold">0</span></p>
                <button id="restartScreenBtn" class="bg-accent text-dark px-6 py-3 rounded-md font-bold text-lg btn-hover pixel-corners">
                    再来一局
                </button>
            </div>
            
            <!-- 暂停屏幕 -->
            <div id="pauseScreen" class="absolute inset-0 bg-dark/70 backdrop-blur-sm flex flex-col items-center justify-center gap-6 z-10 hidden">
                <h2 class="text-2xl font-game text-yellow-400 text-center">游戏暂停</h2>
                <button id="resumeBtn" class="bg-primary text-dark px-6 py-3 rounded-md font-bold text-lg btn-hover pixel-corners">
                    继续游戏
                </button>
            </div>
        </div>
        
        <!-- 移动设备控制按钮 -->
        <div class="md:hidden grid grid-cols-3 gap-2 mt-6 w-full max-w-xs">
            <div class="col-start-2">
                <button id="upBtn" class="w-full bg-slate-700/70 hover:bg-primary/80 text-light p-4 rounded-md btn-hover">
                    <i class="fa fa-arrow-up text-xl"></i>
                </button>
            </div>
            <div class="col-start-1 row-start-2">
                <button id="leftBtn" class="w-full bg-slate-700/70 hover:bg-primary/80 text-light p-4 rounded-md btn-hover">
                    <i class="fa fa-arrow-left text-xl"></i>
                </button>
            </div>
            <div class="col-start-2 row-start-2">
                <button id="downBtn" class="w-full bg-slate-700/70 hover:bg-primary/80 text-light p-4 rounded-md btn-hover">
                    <i class="fa fa-arrow-down text-xl"></i>
                </button>
            </div>
            <div class="col-start-3 row-start-2">
                <button id="rightBtn" class="w-full bg-slate-700/70 hover:bg-primary/80 text-light p-4 rounded-md btn-hover">
                    <i class="fa fa-arrow-right text-xl"></i>
                </button>
            </div>
        </div>
        
        <!-- 游戏说明 -->
        <div class="mt-8 text-sm text-slate-400 max-w-md">
            <h3 class="text-slate-300 font-bold mb-2">游戏说明</h3>
            <ul class="list-disc list-inside space-y-1">
                <li>使用键盘方向键或屏幕按钮控制蛇的移动</li>
                <li>吃到食物后蛇会变长，同时得分增加</li>
                <li>撞到墙壁或蛇自身时游戏结束</li>
                <li>游戏速度会随着得分增加而逐渐加快</li>
            </ul>
        </div>
    </div>

    <script>
        // 获取DOM元素
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const highScoreElement = document.getElementById('highScore');
        const startBtn = document.getElementById('startBtn');
        const pauseBtn = document.getElementById('pauseBtn');
        const restartBtn = document.getElementById('restartBtn');
        const startScreen = document.getElementById('startScreen');
        const gameOverScreen = document.getElementById('gameOverScreen');
        const pauseScreen = document.getElementById('pauseScreen');
        const startScreenBtn = document.getElementById('startScreenBtn');
        const restartScreenBtn = document.getElementById('restartScreenBtn');
        const resumeBtn = document.getElementById('resumeBtn');
        const finalScoreElement = document.getElementById('finalScore');
        const upBtn = document.getElementById('upBtn');
        const downBtn = document.getElementById('downBtn');
        const leftBtn = document.getElementById('leftBtn');
        const rightBtn = document.getElementById('rightBtn');
        
        // 设置画布尺寸
        function resizeCanvas() {
            const container = canvas.parentElement;
            const size = Math.min(container.clientWidth, container.clientHeight);
            canvas.width = size;
            canvas.height = size;
            // 重新绘制游戏（如果游戏正在进行）
            if (gameState === 'running' || gameState === 'paused') {
                draw();
            }
        }
        
        // 初始化时调整一次画布大小
        resizeCanvas();
        // 窗口大小改变时调整画布
        window.addEventListener('resize', resizeCanvas);
        
        // 游戏常量
        const GRID_SIZE = 20; // 网格大小
        let CELL_SIZE; // 单元格大小，将在初始化时计算
        
        // 游戏变量
        let snake = [];
        let food = {};
        let direction = '';
        let nextDirection = '';
        let score = 0;
        let highScore = localStorage.getItem('snakeHighScore') || 0;
        let gameSpeed = 150; // 初始速度（毫秒）
        let gameLoop;
        let gameState = 'ready'; // ready, running, paused, gameOver
        
        // 更新最高分显示
        highScoreElement.textContent = highScore;
        
        // 初始化游戏
        function initGame() {
            // 计算单元格大小
            CELL_SIZE = canvas.width / GRID_SIZE;
            
            // 初始化蛇（从中心开始）
            const center = Math.floor(GRID_SIZE / 2);
            snake = [
                { x: center, y: center },
                { x: center - 1, y: center },
                { x: center - 2, y: center }
            ];
            
            // 设置初始方向
            direction = 'right';
            nextDirection = 'right';
            
            // 生成食物
            generateFood();
            
            // 重置分数
            score = 0;
            scoreElement.textContent = score;
            
            // 重置游戏速度
            gameSpeed = 150;
            
            // 更新游戏状态
            gameState = 'running';
            
            // 更新按钮状态
            updateButtonStates();
            
            // 开始游戏循环
            startGameLoop();
            
            // 隐藏所有屏幕
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            pauseScreen.classList.add('hidden');
        }
        
        // 生成食物
        function generateFood() {
            // 随机位置
            let newFood;
            do {
                newFood = {
                    x: Math.floor(Math.random() * GRID_SIZE),
                    y: Math.floor(Math.random() * GRID_SIZE)
                };
            } while (snake.some(segment => segment.x === newFood.x && segment.y === newFood.y));
            
            food = newFood;
        }
        
        // 开始游戏循环
        function startGameLoop() {
            // 清除之前的循环
            if (gameLoop) clearInterval(gameLoop);
            
            // 设置新的循环
            gameLoop = setInterval(() => {
                if (gameState === 'running') {
                    update();
                    draw();
                }
            }, gameSpeed);
        }
        
        // 更新游戏状态
        function update() {
            // 更新方向
            direction = nextDirection;
            
            // 获取蛇头
            const head = { x: snake[0].x, y: snake[0].y };
            
            // 根据方向移动蛇头
            switch (direction) {
                case 'up':
                    head.y -= 1;
                    break;
                case 'down':
                    head.y += 1;
                    break;
                case 'left':
                    head.x -= 1;
                    break;
                case 'right':
                    head.x += 1;
                    break;
            }
            
            // 检查碰撞（墙壁）
            if (head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE) {
                gameOver();
                return;
            }
            
            // 检查碰撞（自身）
            if (snake.some(segment => segment.x === head.x && segment.y === head.y)) {
                gameOver();
                return;
            }
            
            // 将新头部添加到蛇身
            snake.unshift(head);
            
            // 检查是否吃到食物
            if (head.x === food.x && head.y === food.y) {
                // 增加分数
                score += 10;
                scoreElement.textContent = score;
                
                // 更新最高分
                if (score > highScore) {
                    highScore = score;
                    highScoreElement.textContent = highScore;
                    localStorage.setItem('snakeHighScore', highScore);
                }
                
                // 生成新食物
                generateFood();
                
                // 加快游戏速度（每得50分加速一次）
                if (score % 50 === 0 && gameSpeed > 60) {
                    gameSpeed -= 10;
                    startGameLoop(); // 重新启动循环以应用新速度
                    
                    // 显示速度提升效果
                    showSpeedBoost();
                }
            } else {
                // 如果没吃到食物，移除尾部
                snake.pop();
            }
        }
        
        // 绘制游戏
        function draw() {
            // 清空画布
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 绘制网格背景（轻微的网格线）
            ctx.strokeStyle = 'rgba(74, 222, 128, 0.1)';
            ctx.lineWidth = 1;
            for (let i = 0; i < GRID_SIZE; i++) {
                // 水平线
                ctx.beginPath();
                ctx.moveTo(0, i * CELL_SIZE);
                ctx.lineTo(canvas.width, i * CELL_SIZE);
                ctx.stroke();
                
                // 垂直线
                ctx.beginPath();
                ctx.moveTo(i * CELL_SIZE, 0);
                ctx.lineTo(i * CELL_SIZE, canvas.height);
                ctx.stroke();
            }
            
            // 绘制蛇
            snake.forEach((segment, index) => {
                // 蛇头
                if (index === 0) {
                    ctx.fillStyle = '#166534'; // 深绿色
                    
                    // 绘制蛇头
                    ctx.beginPath();
                    ctx.roundRect(
                        segment.x * CELL_SIZE, 
                        segment.y * CELL_SIZE, 
                        CELL_SIZE, 
                        CELL_SIZE, 
                        8
                    );
                    ctx.fill();
                    
                    // 绘制眼睛
                    ctx.fillStyle = 'white';
                    const eyeSize = CELL_SIZE / 6;
                    const eyeOffset = CELL_SIZE / 4;
                    
                    switch (direction) {
                        case 'up':
                            ctx.beginPath();
                            ctx.arc(
                                segment.x * CELL_SIZE + eyeOffset,
                                segment.y * CELL_SIZE + eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.arc(
                                segment.x * CELL_SIZE + CELL_SIZE - eyeOffset,
                                segment.y * CELL_SIZE + eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.fill();
                            break;
                        case 'down':
                            ctx.beginPath();
                            ctx.arc(
                                segment.x * CELL_SIZE + eyeOffset,
                                segment.y * CELL_SIZE + CELL_SIZE - eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.arc(
                                segment.x * CELL_SIZE + CELL_SIZE - eyeOffset,
                                segment.y * CELL_SIZE + CELL_SIZE - eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.fill();
                            break;
                        case 'left':
                            ctx.beginPath();
                            ctx.arc(
                                segment.x * CELL_SIZE + eyeOffset,
                                segment.y * CELL_SIZE + eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.arc(
                                segment.x * CELL_SIZE + eyeOffset,
                                segment.y * CELL_SIZE + CELL_SIZE - eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.fill();
                            break;
                        case 'right':
                            ctx.beginPath();
                            ctx.arc(
                                segment.x * CELL_SIZE + CELL_SIZE - eyeOffset,
                                segment.y * CELL_SIZE + eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.arc(
                                segment.x * CELL_SIZE + CELL_SIZE - eyeOffset,
                                segment.y * CELL_SIZE + CELL_SIZE - eyeOffset,
                                eyeSize,
                                0,
                                Math.PI * 2
                            );
                            ctx.fill();
                            break;
                    }
                } else {
                    // 蛇身（使用渐变颜色）
                    const colorIndex = index / snake.length;
                    const greenValue = Math.floor(74 + (150 * colorIndex));
                    ctx.fillStyle = `rgb(${greenValue}, 222, 128)`;
                    
                    // 绘制蛇身
                    ctx.beginPath();
                    ctx.roundRect(
                        segment.x * CELL_SIZE, 
                        segment.y * CELL_SIZE, 
                        CELL_SIZE, 
                        CELL_SIZE, 
                        4
                    );
                    ctx.fill();
                }
            });
            
            // 绘制食物（带闪烁动画）
            const pulse = 0.8 + Math.sin(Date.now() / 200) * 0.2;
            ctx.fillStyle = `rgba(245, 158, 11, ${pulse})`; // 橙色食物
            
            // 绘制食物为圆形
            ctx.beginPath();
            ctx.arc(
                food.x * CELL_SIZE + CELL_SIZE / 2,
                food.y * CELL_SIZE + CELL_SIZE / 2,
                CELL_SIZE / 2 * 0.8,
                0,
                Math.PI * 2
            );
            ctx.fill();
            
            // 食物上的小点
            ctx.fillStyle = 'rgba(251, 191, 36, 0.8)';
            ctx.beginPath();
            ctx.arc(
                food.x * CELL_SIZE + CELL_SIZE / 2 - CELL_SIZE / 8,
                food.y * CELL_SIZE + CELL_SIZE / 2 - CELL_SIZE / 8,
                CELL_SIZE / 10,
                0,
                Math.PI * 2
            );
            ctx.fill();
        }
        
        // 显示速度提升效果
        function showSpeedBoost() {
            const speedBoost = document.createElement('div');
            speedBoost.textContent = '速度提升!';
            speedBoost.className = 'absolute top-4 left-1/2 transform -translate-x-1/2 bg-accent text-dark px-4 py-2 rounded-full font-bold z-20 animate-pulse';
            canvas.parentElement.appendChild(speedBoost);
            
            // 3秒后移除
            setTimeout(() => {
                speedBoost.classList.add('opacity-0', 'transition-opacity', 'duration-500');
                setTimeout(() => speedBoost.remove(), 500);
            }, 1500);
        }
        
        // 游戏结束
        function gameOver() {
            gameState = 'gameOver';
            clearInterval(gameLoop);
            
            // 更新最终分数
            finalScoreElement.textContent = score;
            
            // 显示游戏结束屏幕
            gameOverScreen.classList.remove('hidden');
            
            // 更新按钮状态
            updateButtonStates();
        }
        
        // 暂停游戏
        function pauseGame() {
            if (gameState === 'running') {
                gameState = 'paused';
                clearInterval(gameLoop);
                pauseScreen.classList.remove('hidden');
                updateButtonStates();
            }
        }
        
        // 恢复游戏
        function resumeGame() {
            if (gameState === 'paused') {
                gameState = 'running';
                startGameLoop();
                pauseScreen.classList.add('hidden');
                updateButtonStates();
            }
        }
        
        // 更新按钮状态
        function updateButtonStates() {
            switch (gameState) {
                case 'ready':
                    startBtn.disabled = false;
                    pauseBtn.disabled = true;
                    restartBtn.disabled = true;
                    break;
                case 'running':
                    startBtn.disabled = true;
                    pauseBtn.disabled = false;
                    restartBtn.disabled = false;
                    break;
                case 'paused':
                    startBtn.disabled = true;
                    pauseBtn.disabled = true;
                    restartBtn.disabled = false;
                    break;
                case 'gameOver':
                    startBtn.disabled = false;
                    pauseBtn.disabled = true;
                    restartBtn.disabled = false;
                    break;
            }
        }
        
        // 处理方向控制
        function handleDirection(newDirection) {
            // 防止180度转向
            if (
                (newDirection === 'up' && direction !== 'down') ||
                (newDirection === 'down' && direction !== 'up') ||
                (newDirection === 'left' && direction !== 'right') ||
                (newDirection === 'right' && direction !== 'left')
            ) {
                nextDirection = newDirection;
            }
        }
        
        // 事件监听器 - 键盘控制
        document.addEventListener('keydown', (e) => {
            // 如果游戏未开始或已结束，忽略方向键（除了开始游戏）
            if ((gameState === 'ready' || gameState === 'gameOver') && 
                !['Enter', ' '].includes(e.key)) {
                return;
            }
            
            switch (e.key) {
                case 'ArrowUp':
                    handleDirection('up');
                    break;
                case 'ArrowDown':
                    handleDirection('down');
                    break;
                case 'ArrowLeft':
                    handleDirection('left');
                    break;
                case 'ArrowRight':
                    handleDirection('right');
                    break;
                case ' ': // 空格暂停/继续
                    e.preventDefault();
                    if (gameState === 'running') {
                        pauseGame();
                    } else if (gameState === 'paused') {
                        resumeGame();
                    }
                    break;
                case 'Enter': // 回车开始/重新开始
                    if (gameState === 'ready' || gameState === 'gameOver') {
                        initGame();
                    }
                    break;
            }
        });
        
        // 事件监听器 - 按钮控制
        startBtn.addEventListener('click', initGame);
        pauseBtn.addEventListener('click', pauseGame);
        restartBtn.addEventListener('click', initGame);
        startScreenBtn.addEventListener('click', initGame);
        restartScreenBtn.addEventListener('click', initGame);
        resumeBtn.addEventListener('click', resumeGame);
        
        // 移动设备控制按钮
        upBtn.addEventListener('click', () => handleDirection('up'));
        downBtn.addEventListener('click', () => handleDirection('down'));
        leftBtn.addEventListener('click', () => handleDirection('left'));
        rightBtn.addEventListener('click', () => handleDirection('right'));
        
        // 初始绘制
        draw();
    </script>
</body>
</html>
