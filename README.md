<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>超级接球大挑战</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdn.jsdelivr.net/npm/font-awesome@4.7.0/css/font-awesome.min.css" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        primary: '#4F46E5',
                        secondary: '#10B981',
                        accent: '#F59E0B',
                        danger: '#EF4444',
                        dark: '#1F2937',
                        light: '#F9FAFB'
                    },
                    fontFamily: {
                        sans: ['Inter', 'system-ui', 'sans-serif']
                    },
                    animation: {
                        'bounce-slow': 'bounce 2s infinite',
                        'pulse-fast': 'pulse 1s infinite',
                        'float': 'float 3s ease-in-out infinite',
                        'glow': 'glow 2s ease-in-out infinite alternate'
                    },
                    keyframes: {
                        float: {
                            '0%, 100%': { transform: 'translateY(0)' },
                            '50%': { transform: 'translateY(-10px)' }
                        },
                        glow: {
                            '0%': { filter: 'brightness(1) drop-shadow(0 0 5px rgba(79, 70, 229, 0.5))' },
                            '100%': { filter: 'brightness(1.1) drop-shadow(0 0 15px rgba(79, 70, 229, 0.8))' }
                        }
                    }
                }
            }
        }
    </script>
    <style type="text/tailwindcss">
        @layer utilities {
            .game-container {
                position: relative;
                overflow: hidden;
                touch-action: none;
                background: linear-gradient(135deg, #F9FAFB 0%, #EFF6FF 100%);
            }
            .game-overlay {
                background-color: rgba(0, 0, 0, 0.7);
                backdrop-filter: blur(6px);
            }
            .btn-3d {
                @apply relative px-6 py-3 rounded-xl font-bold transform transition-all duration-150 overflow-hidden;
            }
            .btn-3d-primary {
                @apply bg-primary text-white shadow-[0_6px_0_0_#4338CA] active:translate-y-2 active:shadow-[0_3px_0_0_#4338CA] hover:brightness-105;
            }
            .btn-3d-secondary {
                @apply bg-secondary text-white shadow-[0_6px_0_0_#059669] active:translate-y-2 active:shadow-[0_3px_0_0_#059669] hover:brightness-105;
            }
            .btn-3d-accent {
                @apply bg-accent text-white shadow-[0_6px_0_0_#D97706] active:translate-y-2 active:shadow-[0_3px_0_0_#D97706] hover:brightness-105;
            }
            .btn-3d-danger {
                @apply bg-danger text-white shadow-[0_6px_0_0_#DC2626] active:translate-y-2 active:shadow-[0_3px_0_0_#DC2626] hover:brightness-105;
            }
            .btn-3d-dark {
                @apply bg-dark text-white shadow-[0_6px_0_0_#111827] active:translate-y-2 active:shadow-[0_3px_0_0_#111827] hover:brightness-105;
            }
            .btn-reflection {
                @apply absolute top-0 left-0 right-0 h-1/3 bg-gradient-to-b from-white/30 to-transparent z-0;
            }
            .btn-text {
                @apply relative z-10;
            }
            .progress-bar {
                @apply w-full bg-gray-200 h-4 rounded-full overflow-hidden shadow-inner;
            }
            .progress-bar-fill {
                @apply h-full rounded-full transition-all duration-500 ease-out relative;
            }
            .progress-bar-reflection {
                @apply absolute top-0 left-0 right-0 h-1/2 bg-gradient-to-b from-white/40 to-transparent;
            }
            .icon-with-glow {
                @apply relative inline-block;
            }
            .icon-glow {
                @apply absolute -inset-1 rounded-full bg-primary/20 blur opacity-75 group-hover:opacity-100 transition duration-1000 group-hover:duration-200;
            }
            .game-card {
                @apply bg-white rounded-2xl shadow-lg border border-gray-100 overflow-hidden;
            }
            .level-node {
                @apply w-16 h-16 rounded-full flex items-center justify-center shadow-md transform transition-all duration-300;
            }
            .level-node-completed {
                @apply bg-green-100 border-4 border-green-500;
            }
            .level-node-current {
                @apply bg-primary border-4 border-primary/80 animate-pulse;
            }
            .level-node-locked {
                @apply bg-gray-100 border-4 border-gray-300 opacity-70;
            }
            .particle {
                @apply absolute rounded-full pointer-events-none;
            }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-light to-blue-50 min-h-screen flex flex-col items-center justify-center p-4 text-dark font-sans">
    <div class="max-w-md w-full mx-auto">
        <!-- 游戏标题 -->
        <div class="text-center mb-6 animate-float">
            <div class="relative inline-block">
                <div class="absolute -inset-1 bg-gradient-to-r from-primary to-secondary rounded-xl blur-lg opacity-30 animate-pulse-fast"></div>
                <h1 class="text-5xl font-bold text-primary relative z-10 mb-2 tracking-tight">超级接球大挑战</h1>
            </div>
            <p class="text-gray-600 text-lg">准备好测试你的反应速度了吗？</p>
        </div>

        <!-- 游戏容器 -->
        <div class="game-container rounded-2xl shadow-xl mb-6 border border-gray-200" style="height: 500px;">
            <canvas id="gameCanvas" class="w-full h-full"></canvas>
            
            <!-- 开始游戏界面 -->
            <div id="startScreen" class="absolute inset-0 flex flex-col items-center justify-center z-10">
                <div class="text-center mb-10">
                    <div class="relative inline-block mb-6">
                        <div class="absolute -inset-2 bg-gradient-to-r from-primary to-secondary rounded-full blur-xl opacity-30 animate-pulse-fast"></div>
                        <div class="w-24 h-24 rounded-full bg-white border-4 border-primary flex items-center justify-center relative z-10 shadow-lg">
                            <i class="fa fa-gamepad text-5xl text-primary"></i>
                            <div class="absolute top-3 left-3 w-6 h-6 bg-white/40 rounded-full"></div>
                        </div>
                    </div>
                    <h2 class="text-4xl font-bold mb-3 text-dark">超级接球大挑战</h2>
                    <p class="text-gray-600 text-lg mb-2">使用 ← → 方向键或触摸屏幕控制挡板</p>
                    <p class="text-gray-500">挑战你的反应极限！</p>
                </div>
                
                <div class="space-y-4 w-full max-w-xs">
                    <button id="startButton" class="btn-3d btn-3d-primary w-full group">
                        <span class="btn-text flex items-center justify-center">
                            <i class="fa fa-play-circle mr-2"></i>
                            开始游戏
                        </span>
                        <div class="btn-reflection"></div>
                    </button>
                    
                    <button id="levelsButton" class="btn-3d btn-3d-secondary w-full group">
                        <span class="btn-text flex items-center justify-center">
                            <i class="fa fa-trophy mr-2"></i>
                            游戏关卡
                        </span>
                        <div class="btn-reflection"></div>
                    </button>
                </div>
                
                <div class="mt-8 text-center">
                    <div class="inline-flex items-center px-4 py-2 bg-white/80 backdrop-blur-sm rounded-full shadow-md">
                        <div class="w-8 h-8 rounded-full bg-primary/10 flex items-center justify-center mr-2">
                            <i class="fa fa-star text-primary"></i>
                        </div>
                        <span class="font-bold">最高分: <span id="highScoreDisplay" class="text-primary">5</span></span>
                    </div>
                </div>
            </div>
            
            <!-- 游戏关卡选择界面 -->
            <div id="levelsScreen" class="absolute inset-0 game-overlay flex flex-col items-center justify-center z-20 hidden">
                <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-xs">
                    <div class="flex justify-between items-center mb-6">
                        <h2 class="text-2xl font-bold text-dark">游戏关卡</h2>
                        <button id="closeLevelsButton" class="w-8 h-8 rounded-full bg-gray-100 flex items-center justify-center hover:bg-gray-200 transition-colors">
                            <i class="fa fa-times"></i>
                        </button>
                    </div>
                    
                    <div class="levels-map flex flex-col gap-8 items-center relative mb-6">
                        <!-- 背景路径 -->
                        <div class="absolute z-0 w-2 bg-gray-200 h-full left-1/2 transform -translate-x-1/2"></div>
                      
                        <!-- 完成的关卡 -->
                        <div class="relative z-10">
                            <button class="level-node level-node-completed">
                                <svg class="w-8 h-8 text-green-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                                </svg>
                            </button>
                        </div>
                      
                        <!-- 当前关卡 -->
                        <div class="relative z-10 -ml-16">
                            <button class="level-node level-node-current">
                                <span class="text-white font-bold text-xl">2</span>
                            </button>
                        </div>
                      
                        <!-- 未解锁关卡 -->
                        <div class="relative z-10 ml-16">
                            <button class="level-node level-node-locked" disabled>
                                <svg class="w-8 h-8 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z" />
                                </svg>
                            </button>
                        </div>
                    </div>
                    
                    <button id="startLevelButton" class="btn-3d btn-3d-primary w-full group">
                        <span class="btn-text flex items-center justify-center">
                            <i class="fa fa-play-circle mr-2"></i>
                            开始关卡 2
                        </span>
                        <div class="btn-reflection"></div>
                    </button>
                </div>
            </div>
            
            <!-- 游戏界面 -->
            <div id="gameUI" class="absolute top-0 left-0 right-0 p-4 z-10 hidden">
                <div class="flex justify-between items-center">
                    <div class="flex items-center space-x-3">
                        <!-- 分数 -->
                        <div class="flex items-center px-3 py-1.5 bg-white/80 backdrop-blur-sm rounded-full shadow-md">
                            <div class="w-6 h-6 rounded-full bg-primary/10 flex items-center justify-center mr-2">
                                <i class="fa fa-star text-primary text-xs"></i>
                            </div>
                            <span class="font-bold text-sm">分数: <span id="currentScoreDisplay" class="text-primary">0</span></span>
                        </div>
                        
                        <!-- 等级 -->
                        <div class="flex items-center px-3 py-1.5 bg-white/80 backdrop-blur-sm rounded-full shadow-md">
                            <div class="w-6 h-6 rounded-full bg-accent/10 flex items-center justify-center mr-2">
                                <i class="fa fa-level-up text-accent text-xs"></i>
                            </div>
                            <span class="font-bold text-sm">等级: <span id="levelDisplay" class="text-accent">1</span></span>
                        </div>
                    </div>
                    
                    <div class="flex space-x-2">
                        <button id="pauseButton" class="w-8 h-8 rounded-full bg-white/80 backdrop-blur-sm flex items-center justify-center hover:bg-white shadow-md transition-colors">
                            <i class="fa fa-pause text-gray-700"></i>
                        </button>
                        <button id="soundButton" class="w-8 h-8 rounded-full bg-white/80 backdrop-blur-sm flex items-center justify-center hover:bg-white shadow-md transition-colors">
                            <i class="fa fa-volume-up text-gray-700"></i>
                        </button>
                    </div>
                </div>
                
                <!-- 进度条 -->
                <div class="mt-3">
                    <div class="progress-bar">
                        <div id="progressBar" class="progress-bar-fill bg-gradient-to-r from-primary to-secondary" style="width: 0%">
                            <div class="progress-bar-reflection"></div>
                        </div>
                    </div>
                    <div class="flex justify-between text-xs text-gray-500 mt-1">
                        <span>当前等级</span>
                        <span>下一级</span>
                    </div>
                </div>
            </div>
            
            <!-- 暂停菜单 -->
            <div id="pauseScreen" class="absolute inset-0 game-overlay flex flex-col items-center justify-center z-20 hidden">
                <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-xs text-center">
                    <h2 class="text-3xl font-bold mb-6 text-dark">游戏暂停</h2>
                    
                    <div class="space-y-4 mb-6">
                        <button id="resumeButton" class="btn-3d btn-3d-primary w-full group">
                            <span class="btn-text flex items-center justify-center">
                                <i class="fa fa-play mr-2"></i>
                                继续游戏
                            </span>
                            <div class="btn-reflection"></div>
                        </button>
                        
                        <button id="restartFromPauseButton" class="btn-3d btn-3d-accent w-full group">
                            <span class="btn-text flex items-center justify-center">
                                <i class="fa fa-refresh mr-2"></i>
                                重新开始
                            </span>
                            <div class="btn-reflection"></div>
                        </button>
                        
                        <button id="quitButton" class="btn-3d btn-3d-dark w-full group">
                            <span class="btn-text flex items-center justify-center">
                                <i class="fa fa-home mr-2"></i>
                                返回主页
                            </span>
                            <div class="btn-reflection"></div>
                        </button>
                    </div>
                </div>
            </div>
            
            <!-- 游戏结束界面 -->
            <div id="gameOverScreen" class="absolute inset-0 game-overlay flex flex-col items-center justify-center z-20 hidden">
                <div class="bg-white rounded-2xl shadow-2xl p-6 w-full max-w-xs text-center">
                    <div class="mb-6">
                        <div class="relative inline-block mb-4">
                            <div class="absolute -inset-2 bg-gradient-to-r from-danger to-accent rounded-full blur-xl opacity-30 animate-pulse-fast"></div>
                            <div class="w-20 h-20 rounded-full bg-white border-4 border-danger flex items-center justify-center relative z-10 shadow-lg">
                                <i class="fa fa-times-circle text-4xl text-danger"></i>
                                <div class="absolute top-2 left-2 w-5 h-5 bg-white/40 rounded-full"></div>
                            </div>
                        </div>
                        <h2 class="text-3xl font-bold mb-2 text-dark">游戏结束</h2>
                        <p class="text-gray-600 mb-4">别灰心，再试一次吧！</p>
                        
                        <div class="space-y-2 mb-6">
                            <div class="flex justify-between items-center bg-gray-50 p-3 rounded-xl">
                                <span class="font-medium text-gray-600">你的得分</span>
                                <span id="scoreDisplay" class="font-bold text-xl text-primary">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center bg-gray-50 p-3 rounded-xl">
                                <span class="font-medium text-gray-600">最高分</span>
                                <span id="finalHighScoreDisplay" class="font-bold text-xl text-accent">0</span>
                            </div>
                            
                            <div class="flex justify-between items-center bg-gray-50 p-3 rounded-xl">
                                <span class="font-medium text-gray-600">达到等级</span>
                                <span id="finalLevelDisplay" class="font-bold text-xl text-secondary">1</span>
                            </div>
                        </div>
                    </div>
                    
                    <div class="space-y-4">
                        <button id="restartButton" class="btn-3d btn-3d-primary w-full group">
                            <span class="btn-text flex items-center justify-center">
                                <i class="fa fa-refresh mr-2"></i>
                                再来一次
                            </span>
                            <div class="btn-reflection"></div>
                        </button>
                        
                        <button id="homeButton" class="btn-3d btn-3d-dark w-full group">
                            <span class="btn-text flex items-center justify-center">
                                <i class="fa fa-home mr-2"></i>
                                返回主页
                            </span>
                            <div class="btn-reflection"></div>
                        </button>
                    </div>
                </div>
            </div>
        </div>
        
        <!-- 游戏信息 -->
        <div class="game-card p-5 mb-6">
            <h3 class="font-bold text-xl mb-3 flex items-center text-dark">
                <div class="icon-with-glow mr-2">
                    <div class="icon-glow"></div>
                    <i class="fa fa-info-circle text-primary relative z-10"></i>
                </div>
                游戏说明
            </h3>
            <ul class="text-gray-700 space-y-2">
                <li class="flex items-start">
                    <i class="fa fa-check-circle text-secondary mt-1 mr-2"></i>
                    <span>使用键盘左右方向键或触摸屏幕移动挡板</span>
                </li>
                <li class="flex items-start">
                    <i class="fa fa-check-circle text-secondary mt-1 mr-2"></i>
                    <span>接住落下的球获得分数</span>
                </li>
                <li class="flex items-start">
                    <i class="fa fa-check-circle text-secondary mt-1 mr-2"></i>
                    <span>球碰到挡板会根据击中位置反弹，带有随机性</span>
                </li>
                <li class="flex items-start">
                    <i class="fa fa-check-circle text-secondary mt-1 mr-2"></i>
                    <span>球可以碰到墙壁反弹</span>
                </li>
                <li class="flex items-start">
                    <i class="fa fa-check-circle text-secondary mt-1 mr-2"></i>
                    <span>随着分数增加，球的速度会变快，挑战升级</span>
                </li>
            </ul>
        </div>
        
        <!-- 控制提示 -->
        <div class="game-card p-5">
            <h3 class="font-bold text-xl mb-3 flex items-center text-dark">
                <div class="icon-with-glow mr-2">
                    <div class="icon-glow"></div>
                    <i class="fa fa-gamepad text-primary relative z-10"></i>
                </div>
                操作控制
            </h3>
            <div class="grid grid-cols-2 gap-4">
                <div class="flex flex-col items-center bg-gray-50 p-3 rounded-xl">
                    <div class="w-12 h-12 rounded-full bg-primary/10 flex items-center justify-center mb-2">
                        <i class="fa fa-keyboard-o text-primary text-xl"></i>
                    </div>
                    <span class="text-sm text-center">键盘方向键<br>← →</span>
                </div>
                <div class="flex flex-col items-center bg-gray-50 p-3 rounded-xl">
                    <div class="w-12 h-12 rounded-full bg-secondary/10 flex items-center justify-center mb-2">
                        <i class="fa fa-hand-pointer-o text-secondary text-xl"></i>
                    </div>
                    <span class="text-sm text-center">触摸屏幕<br>左右滑动</span>
                </div>
                <div class="flex flex-col items-center bg-gray-50 p-3 rounded-xl">
                    <div class="w-12 h-12 rounded-full bg-accent/10 flex items-center justify-center mb-2">
                        <i class="fa fa-pause text-accent text-xl"></i>
                    </div>
                    <span class="text-sm text-center">暂停游戏<br>空格键或点击</span>
                </div>
                <div class="flex flex-col items-center bg-gray-50 p-3 rounded-xl">
                    <div class="w-12 h-12 rounded-full bg-danger/10 flex items-center justify-center mb-2">
                        <i class="fa fa-volume-up text-danger text-xl"></i>
                    </div>
                    <span class="text-sm text-center">音效开关<br>点击按钮</span>
                </div>
            </div>
        </div>
    </div>

    <script>
        // 游戏状态
        const gameState = {
            isPlaying: false,
            isPaused: false,
            score: 0,
            highScore: parseInt(localStorage.getItem('catchBallHighScore')) || 5,
            soundEnabled: true,
            ballSpeed: 5,
            ballHeight: 150,
            gameLevel: 1,
            levelProgress: 0,
            levelThreshold: 5 // 每5分升一级
        };

        // 游戏元素
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        let ball = {
            x: 0,
            y: 0,
            radius: 15,
            dx: 0,
            dy: 0,
            jumping: false,
            color: '#F59E0B', // accent color
            trail: [] // 用于球的轨迹效果
        };
        let paddle = {
            x: 0,
            y: 0,
            width: 100,
            height: 15,
            dx: 0,
            speed: 8,
            color: '#4F46E5', // primary color
            glowing: false
        };

        // 粒子系统
        let particles = [];

        // 调整Canvas大小
        function resizeCanvas() {
            const container = canvas.parentElement;
            canvas.width = container.clientWidth;
            canvas.height = container.clientHeight;
            
            // 初始化游戏元素位置
            if (!ball.jumping) {
                ball.x = canvas.width / 2;
                ball.y = canvas.height - paddle.height - ball.radius - 10;
            }
            paddle.x = (canvas.width - paddle.width) / 2;
            paddle.y = canvas.height - paddle.height - 10;
        }

        // 初始化游戏
        function initGame() {
            resizeCanvas();
            updateHighScoreDisplay();
            
            // 重置游戏状态
            gameState.score = 0;
            gameState.ballSpeed = 5;
            gameState.ballHeight = 150;
            gameState.gameLevel = 1;
            gameState.levelProgress = 0;
            
            // 重置球和挡板
            ball.x = canvas.width / 2;
            ball.y = canvas.height - paddle.height - ball.radius - 10;
            ball.dx = 0;
            ball.dy = 0;
            ball.jumping = false;
            ball.trail = [];
            ball.color = '#F59E0B'; // accent color
            
            paddle.x = (canvas.width - paddle.width) / 2;
            paddle.dx = 0;
            paddle.glowing = false;
            
            // 清空粒子
            particles = [];
            
            // 更新UI显示
            document.getElementById('currentScoreDisplay').textContent = gameState.score;
            document.getElementById('levelDisplay').textContent = gameState.gameLevel;
            updateProgressBar();
        }

        // 绘制游戏元素
        function draw() {
            // 清空画布
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            
            // 绘制背景网格
            drawGrid();
            
            // 绘制球的轨迹
            drawBallTrail();
            
            // 绘制粒子
            drawParticles();
            
            // 绘制挡板
            drawPaddle();
            
            // 绘制球
            drawBall();
        }

        // 绘制背景网格
        function drawGrid() {
            ctx.strokeStyle = 'rgba(79, 70, 229, 0.05)';
            ctx.lineWidth = 1;
            
            // 绘制垂直线
            for (let x = 0; x <= canvas.width; x += 30) {
                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, canvas.height);
                ctx.stroke();
            }
            
            // 绘制水平线
            for (let y = 0; y <= canvas.height; y += 30) {
                ctx.beginPath();
                ctx.moveTo(0, y);
                ctx.lineTo(canvas.width, y);
                ctx.stroke();
            }
        }

        // 绘制挡板
        function drawPaddle() {
            // 挡板发光效果
            if (paddle.glowing) {
                ctx.shadowColor = paddle.color;
                ctx.shadowBlur = 15;
            }
            
            // 挡板主体
            ctx.fillStyle = paddle.color;
            ctx.beginPath();
            ctx.roundRect(paddle.x, paddle.y, paddle.width, paddle.height, 8);
            ctx.fill();
            
            // 重置阴影
            if (paddle.glowing) {
                ctx.shadowBlur = 0;
            }
            
            // 挡板高光
            ctx.fillStyle = 'rgba(255, 255, 255, 0.4)';
            ctx.beginPath();
            ctx.roundRect(paddle.x + 2, paddle.y + 2, paddle.width - 4, 6, 4);
            ctx.fill();
            
            // 挡板反光
            ctx.fillStyle = 'rgba(255, 255, 255, 0.2)';
            ctx.beginPath();
            ctx.roundRect(paddle.x + 5, paddle.y + 1, paddle.width - 10, 3, 2);
            ctx.fill();
        }

        // 绘制球
        function drawBall() {
            // 球的发光效果
            ctx.shadowColor = ball.color;
            ctx.shadowBlur = 10;
            
            // 球主体
            ctx.fillStyle = ball.color;
            ctx.beginPath();
            ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
            ctx.fill();
            
            // 重置阴影
            ctx.shadowBlur = 0;
            
            // 球高光
            ctx.fillStyle = 'rgba(255, 255, 255, 0.5)';
            ctx.beginPath();
            ctx.arc(ball.x - ball.radius / 3, ball.y - ball.radius / 3, ball.radius / 3, 0, Math.PI * 2);
            ctx.fill();
            
            // 球反光
            ctx.fillStyle = 'rgba(255, 255, 255, 0.3)';
            ctx.beginPath();
            ctx.arc(ball.x + ball.radius / 4, ball.y + ball.radius / 4, ball.radius / 6, 0, Math.PI * 2);
            ctx.fill();
        }

        // 绘制球的轨迹
        function drawBallTrail() {
            if (ball.trail.length < 2) return;
            
            // 绘制轨迹线
            ctx.strokeStyle = ball.color + '40'; // 添加透明度
            ctx.lineWidth = ball.radius / 2;
            ctx.lineCap = 'round';
            ctx.beginPath();
            
            for (let i = 0; i < ball.trail.length - 1; i++) {
                const alpha = i / ball.trail.length;
                ctx.globalAlpha = alpha * 0.5;
                
                if (i === 0) {
                    ctx.moveTo(ball.trail[i].x, ball.trail[i].y);
                } else {
                    ctx.lineTo(ball.trail[i].x, ball.trail[i].y);
                }
            }
            
            ctx.stroke();
            ctx.globalAlpha = 1;
        }

        // 绘制粒子
        function drawParticles() {
            particles.forEach(particle => {
                ctx.fillStyle = particle.color;
                ctx.globalAlpha = particle.alpha;
                ctx.beginPath();
                ctx.arc(particle.x, particle.y, particle.size, 0, Math.PI * 2);
                ctx.fill();
            });
            
            ctx.globalAlpha = 1;
        }

        // 更新粒子
        function updateParticles() {
            particles = particles.filter(particle => {
                particle.x += particle.dx;
                particle.y += particle.dy;
                particle.alpha -= 0.02;
                particle.size *= 0.98;
                
                return particle.alpha > 0 && particle.size > 0.5;
            });
        }

        // 创建粒子效果
        function createParticles(x, y, color) {
            const particleCount = 10;
            
            for (let i = 0; i < particleCount; i++) {
                const angle = Math.random() * Math.PI * 2;
                const speed = Math.random() * 3 + 1;
                
                particles.push({
                    x: x,
                    y: y,
                    dx: Math.cos(angle) * speed,
                    dy: Math.sin(angle) * speed,
                    size: Math.random() * 5 + 2,
                    color: color,
                    alpha: Math.random() * 0.8 + 0.2
                });
            }
        }

        // 更新游戏状态
        function update() {
            if (!gameState.isPlaying || gameState.isPaused) return;
            
            // 更新挡板位置
            paddle.x += paddle.dx;
            
            // 挡板边界检查
            if (paddle.x < 0) {
                paddle.x = 0;
            } else if (paddle.x + paddle.width > canvas.width) {
                paddle.x = canvas.width - paddle.width;
            }
            
            // 更新球的位置
            if (ball.jumping) {
                // 记录球的位置用于轨迹
                ball.trail.push({ x: ball.x, y: ball.y });
                if (ball.trail.length > 10) {
                    ball.trail.shift();
                }
                
                ball.x += ball.dx;
                ball.y += ball.dy;
                
                // 添加一些空气阻力
                ball.dx *= 0.999;
                
                // 球碰到左右边界反弹
                if (ball.x - ball.radius <= 0 || ball.x + ball.radius >= canvas.width) {
                    ball.dx = -ball.dx * 0.9; // 反弹时损失一些能量
                    ball.x = ball.x - ball.radius <= 0 ? ball.radius : canvas.width - ball.radius;
                    
                    // 创建碰撞粒子效果
                    createParticles(ball.x, ball.y, ball.color);
                    playSound('wallHit');
                }
                
                // 球碰到顶部边界反弹
                if (ball.y - ball.radius <= 0) {
                    ball.dy = Math.abs(ball.dy) * 0.9; // 确保向下运动，损失一些能量
                    ball.y = ball.radius;
                    
                    // 创建碰撞粒子效果
                    createParticles(ball.x, ball.y, ball.color);
                    playSound('wallHit');
                }
                
                // 球碰到挡板
                if (
                    ball.y + ball.radius >= paddle.y &&
                    ball.y + ball.radius <= paddle.y + paddle.height &&
                    ball.x >= paddle.x &&
                    ball.x <= paddle.x + paddle.width
                ) {
                    playSound('bounce');
                    gameState.score++;
                    document.getElementById('currentScoreDisplay').textContent = gameState.score;
                    
                    // 创建碰撞粒子效果
                    createParticles(ball.x, ball.y, ball.color);
                    
                    // 短暂的挡板发光效果
                    paddle.glowing = true;
                    setTimeout(() => {
                        paddle.glowing = false;
                    }, 200);
                    
                    // 更新游戏难度和进度
                    updateGameDifficulty();
                    
                    // 计算球在挡板上的相对位置（-1到1，中心为0）
                    const paddleCenter = paddle.x + paddle.width / 2;
                    const hitPosition = (ball.x - paddleCenter) / (paddle.width / 2);
                    
                    // 根据击中位置设置反弹角度，增加一些随机性
                    const baseAngle = hitPosition * Math.PI / 3; // 最大60度角
                    const randomAngle = (Math.random() - 0.5) * Math.PI / 6; // 随机±15度
                    const bounceAngle = baseAngle + randomAngle;
                    
                    // 设置球的速度和方向
                    const speed = gameState.ballSpeed * 1.5;
                    ball.dx = Math.sin(bounceAngle) * speed;
                    ball.dy = -Math.cos(bounceAngle) * speed;
                    
                    // 确保球不会卡在挡板内
                    ball.y = paddle.y - ball.radius;
                }
                
                // 球落地，游戏结束
                if (ball.y + ball.radius > canvas.height) {
                    playSound('gameOver');
                    gameOver();
                }
            }
            
            // 更新粒子
            updateParticles();
        }

        // 更新游戏难度
        function updateGameDifficulty() {
            // 更新等级进度
            gameState.levelProgress++;
            updateProgressBar();
            
            // 每5分增加一个等级
            const newLevel = Math.floor(gameState.score / gameState.levelThreshold) + 1;
            if (newLevel > gameState.gameLevel) {
                gameState.gameLevel = newLevel;
                gameState.ballSpeed += 0.5;
                gameState.ballHeight += 20;
                
                // 更新等级显示
                document.getElementById('levelDisplay').textContent = gameState.gameLevel;
                
                // 调整球的颜色
                const hue = (30 + gameState.gameLevel * 15) % 360;
                ball.color = `hsl(${hue}, 90%, 55%)`;
                
                // 播放升级音效
                playSound('levelUp');
                
                // 显示升级提示
                showLevelUpNotification();
            }
        }

        // 更新进度条
        function updateProgressBar() {
            const progress = (gameState.levelProgress % gameState.levelThreshold) / gameState.levelThreshold * 100;
            document.getElementById('progressBar').style.width = `${progress}%`;
        }

        // 显示升级提示
        function showLevelUpNotification() {
            // 创建升级提示元素
            const notification = document.createElement('div');
            notification.className = 'absolute top-1/4 left-1/2 transform -translate-x-1/2 -translate-y-1/2 bg-gradient-to-r from-primary to-secondary text-white px-6 py-3 rounded-full shadow-lg z-30 animate-float';
            notification.innerHTML = `
                <div class="flex items-center">
                    <i class="fa fa-arrow-up-circle text-2xl mr-2"></i>
                    <span class="font-bold text-lg">升级到 ${gameState.gameLevel} 级！</span>
                </div>
            `;
            
            // 添加到游戏容器
            canvas.parentElement.appendChild(notification);
            
            // 3秒后移除
            setTimeout(() => {
                notification.classList.add('opacity-0', 'transition-opacity', 'duration-500');
                setTimeout(() => {
                    notification.remove();
                }, 500);
            }, 3000);
        }

        // 球跳跃
        function jumpBall() {
            if (!gameState.isPlaying || gameState.isPaused) return;
            
            playSound('jump');
            ball.jumping = true;
            
            // 随机初始角度，使每次跳跃方向略有不同
            const randomAngle = (Math.random() - 0.5) * Math.PI / 3; // 随机±30度
            const speed = gameState.ballSpeed * 2;
            ball.dx = Math.sin(randomAngle) * speed * 0.5;
            ball.dy = -Math.cos(randomAngle) * speed;
            ball.height = gameState.ballHeight;
        }

        // 游戏循环
        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        // 游戏结束
        function gameOver() {
            gameState.isPlaying = false;
            
            // 更新最高分
            if (gameState.score > gameState.highScore) {
                gameState.highScore = gameState.score;
                localStorage.setItem('catchBallHighScore', gameState.highScore);
                updateHighScoreDisplay();
            }
            
            // 显示游戏结束界面
            document.getElementById('scoreDisplay').textContent = gameState.score;
            document.getElementById('finalHighScoreDisplay').textContent = gameState.highScore;
            document.getElementById('finalLevelDisplay').textContent = gameState.gameLevel;
            document.getElementById('gameOverScreen').classList.remove('hidden');
        }

        // 更新最高分显示
        function updateHighScoreDisplay() {
            document.getElementById('highScoreDisplay').textContent = gameState.highScore;
        }

        // 播放音效
        function playSound(type) {
            if (!gameState.soundEnabled) return;
            
            // 使用Web Audio API创建简单音效
            const audioContext = new (window.AudioContext || window.webkitAudioContext)();
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);
            
            // 设置不同音效的参数
            switch (type) {
                case 'jump':
                    oscillator.frequency.setValueAtTime(440, audioContext.currentTime);
                    oscillator.frequency.exponentialRampToValueAtTime(880, audioContext.currentTime + 0.1);
                    gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
                    oscillator.start(audioContext.currentTime);
                    oscillator.stop(audioContext.currentTime + 0.1);
                    break;
                case 'bounce':
                    oscillator.frequency.setValueAtTime(660, audioContext.currentTime);
                    oscillator.frequency.exponentialRampToValueAtTime(330, audioContext.currentTime + 0.15);
                    gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.15);
                    oscillator.start(audioContext.currentTime);
                    oscillator.stop(audioContext.currentTime + 0.15);
                    break;
                case 'wallHit':
                    oscillator.frequency.setValueAtTime(550, audioContext.currentTime);
                    oscillator.frequency.exponentialRampToValueAtTime(440, audioContext.currentTime + 0.1);
                    gainNode.gain.setValueAtTime(0.08, audioContext.currentTime);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1);
                    oscillator.start(audioContext.currentTime);
                    oscillator.stop(audioContext.currentTime + 0.1);
                    break;
                case 'levelUp':
                    // 创建上升音调的音效
                    for (let i = 0; i < 3; i++) {
                        const osc = audioContext.createOscillator();
                        const gNode = audioContext.createGain();
                        
                        osc.connect(gNode);
                        gNode.connect(audioContext.destination);
                        
                        const baseFreq = 440 + (i * 220);
                        osc.frequency.setValueAtTime(baseFreq, audioContext.currentTime + (i * 0.1));
                        osc.frequency.exponentialRampToValueAtTime(baseFreq * 1.5, audioContext.currentTime + 0.1 + (i * 0.1));
                        
                        gNode.gain.setValueAtTime(0, audioContext.currentTime + (i * 0.1));
                        gNode.gain.linearRampToValueAtTime(0.1, audioContext.currentTime + 0.02 + (i * 0.1));
                        gNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.1 + (i * 0.1));
                        
                        osc.start(audioContext.currentTime + (i * 0.1));
                        osc.stop(audioContext.currentTime + 0.1 + (i * 0.1));
                    }
                    break;
                case 'gameOver':
                    oscillator.frequency.setValueAtTime(330, audioContext.currentTime);
                    oscillator.frequency.exponentialRampToValueAtTime(110, audioContext.currentTime + 0.5);
                    gainNode.gain.setValueAtTime(0.1, audioContext.currentTime);
                    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + 0.5);
                    oscillator.start(audioContext.currentTime);
                    oscillator.stop(audioContext.currentTime + 0.5);
                    break;
            }
        }

        // 事件监听
        function setupEventListeners() {
            // 键盘控制
            document.addEventListener('keydown', (e) => {
                if (!gameState.isPlaying) return;
                
                if (e.key === 'ArrowLeft') {
                    paddle.dx = -paddle.speed;
                } else if (e.key === 'ArrowRight') {
                    paddle.dx = paddle.speed;
                } else if (e.key === ' ' || e.key === 'Spacebar') {
                    // 空格键暂停/继续
                    togglePause();
                }
            });
            
            document.addEventListener('keyup', (e) => {
                if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
                    paddle.dx = 0;
                }
            });
            
            // 触摸控制
            let touchStartX = 0;
            let touchStartTime = 0;
            
            canvas.addEventListener('touchstart', (e) => {
                e.preventDefault();
                if (!gameState.isPlaying) return;
                
                touchStartX = e.touches[0].clientX;
                touchStartTime = Date.now();
            });
            
            canvas.addEventListener('touchmove', (e) => {
                e.preventDefault();
                if (!gameState.isPlaying) return;
                
                const touchX = e.touches[0].clientX;
                const touchDeltaX = touchX - touchStartX;
                
                // 根据触摸移动距离设置挡板速度
                const touchSpeed = Math.min(Math.max(touchDeltaX / 10, -5), 5);
                paddle.dx = touchSpeed;
                
                touchStartX = touchX;
            });
            
            canvas.addEventListener('touchend', (e) => {
                e.preventDefault();
                if (!gameState.isPlaying) return;
                
                const touchEndTime = Date.now();
                const touchDuration = touchEndTime - touchStartTime;
                
                // 如果触摸时间很短，可能是点击而不是滑动
                if (touchDuration < 100) {
                    // 点击屏幕暂停/继续
                    togglePause();
                } else {
                    paddle.dx = 0;
                }
            });
            
            // 按钮事件
            document.getElementById('startButton').addEventListener('click', startGame);
            document.getElementById('startLevelButton').addEventListener('click', startGame);
            document.getElementById('restartButton').addEventListener('click', restartGame);
            document.getElementById('restartFromPauseButton').addEventListener('click', restartGame);
            document.getElementById('homeButton').addEventListener('click', showStartScreen);
            document.getElementById('quitButton').addEventListener('click', showStartScreen);
            document.getElementById('pauseButton').addEventListener('click', togglePause);
            document.getElementById('resumeButton').addEventListener('click', togglePause);
            document.getElementById('soundButton').addEventListener('click', toggleSound);
            document.getElementById('levelsButton').addEventListener('click', showLevelsScreen);
            document.getElementById('closeLevelsButton').addEventListener('click', hideLevelsScreen);
            
            // 窗口大小变化时调整Canvas大小
            window.addEventListener('resize', resizeCanvas);
        }

        // 开始游戏
        function startGame() {
            gameState.isPlaying = true;
            gameState.isPaused = false;
            initGame();
            
            // 隐藏所有菜单
            document.getElementById('startScreen').classList.add('hidden');
            document.getElementById('levelsScreen').classList.add('hidden');
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('pauseScreen').classList.add('hidden');
            
            // 显示游戏UI
            document.getElementById('gameUI').classList.remove('hidden');
            
            // 延迟一秒后球开始跳跃
            setTimeout(() => {
                if (gameState.isPlaying) {
                    jumpBall();
                }
            }, 1000);
        }

        // 重新开始游戏
        function restartGame() {
            gameState.isPlaying = true;
            gameState.isPaused = false;
            initGame();
            
            // 隐藏所有菜单
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('pauseScreen').classList.add('hidden');
            
            // 显示游戏UI
            document.getElementById('gameUI').classList.remove('hidden');
            
            // 延迟一秒后球开始跳跃
            setTimeout(() => {
                if (gameState.isPlaying) {
                    jumpBall();
                }
            }, 1000);
        }

        // 显示开始界面
        function showStartScreen() {
            gameState.isPlaying = false;
            gameState.isPaused = false;
            
            // 隐藏所有菜单
            document.getElementById('gameOverScreen').classList.add('hidden');
            document.getElementById('pauseScreen').classList.add('hidden');
            document.getElementById('levelsScreen').classList.add('hidden');
            
            // 显示开始界面
            document.getElementById('startScreen').classList.remove('hidden');
            document.getElementById('gameUI').classList.add('hidden');
        }

        // 显示关卡选择界面
        function showLevelsScreen() {
            document.getElementById('levelsScreen').classList.remove('hidden');
        }

        // 隐藏关卡选择界面
        function hideLevelsScreen() {
            document.getElementById('levelsScreen').classList.add('hidden');
        }

        // 暂停/继续游戏
        function togglePause() {
            if (!gameState.isPlaying) return;
            
            gameState.isPaused = !gameState.isPaused;
            const pauseButton = document.getElementById('pauseButton');
            
            if (gameState.isPaused) {
                pauseButton.innerHTML = '<i class="fa fa-play text-gray-700"></i>';
                document.getElementById('pauseScreen').classList.remove('hidden');
            } else {
                pauseButton.innerHTML = '<i class="fa fa-pause text-gray-700"></i>';
                document.getElementById('pauseScreen').classList.add('hidden');
            }
        }

        // 切换音效
        function toggleSound() {
            gameState.soundEnabled = !gameState.soundEnabled;
            const soundButton = document.getElementById('soundButton');
            
            if (gameState.soundEnabled) {
                soundButton.innerHTML = '<i class="fa fa-volume-up text-gray-700"></i>';
            } else {
                soundButton.innerHTML = '<i class="fa fa-volume-off text-gray-700"></i>';
            }
        }

        // 初始化
        function init() {
            setupEventListeners();
            resizeCanvas();
            updateHighScoreDisplay();
            gameLoop();
        }

        // 启动游戏
        init();
    </script>
</body>
</html>
