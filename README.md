<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tetris Synthwave Retro Arcade</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts for Retro Visuals -->
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        * {
            font-family: 'Press Start 2P', monospace;
            user-select: none;
            -webkit-user-select: none;
            touch-action: manipulation;
        }

        body {
            background-color: #0d0814;
            color: #fff;
            overflow-x: hidden;
        }

        /* Scanline Overlay Effect for CRT Aesthetics */
        .crt-overlay {
            position: fixed;
            top: 0; left: 0; width: 100vw; height: 100vh;
            background: linear-gradient(
                rgba(18, 16, 16, 0) 50%, 
                rgba(0, 0, 0, 0.25) 50%
            ), linear-gradient(
                90deg,
                rgba(255, 0, 0, 0.03),
                rgba(0, 255, 0, 0.01),
                rgba(0, 0, 255, 0.03)
            );
            background-size: 100% 4px, 6px 100%;
            pointer-events: none;
            z-index: 50;
        }

        /* Screen Flicker Animation */
        .crt-flicker {
            animation: flicker 0.15s infinite;
        }

        @keyframes flicker {
            0% { opacity: 0.98; }
            50% { opacity: 1; }
            100% { opacity: 0.97; }
        }

        /* Neon Text Glow Effects */
        .glow-cyan { text-shadow: 0 0 8px #00f0ff, 0 0 15px #00f0ff; }
        .glow-pink { text-shadow: 0 0 8px #ff007f, 0 0 15px #ff007f; }
        .glow-yellow { text-shadow: 0 0 8px #ffea00, 0 0 15px #ffea00; }
        .glow-green { text-shadow: 0 0 8px #00ff66, 0 0 15px #00ff66; }

        /* Canvas Borders */
        .arcade-border {
            border: 4px solid #3b2d54;
            box-shadow: 0 0 15px rgba(255, 0, 128, 0.3), inset 0 0 15px rgba(0, 240, 255, 0.2);
        }

        /* Retro Button Styles */
        .retro-btn {
            background: linear-gradient(180deg, #2a1f40 0%, #140d24 100%);
            border: 2px solid #5a4380;
            box-shadow: 0 4px 0 #0d0814, 0 0 10px rgba(147, 51, 234, 0.3);
            transition: all 0.08s ease;
            active: translateY(3px);
        }

        .retro-btn:active, .retro-btn.active {
            transform: translateY(3px);
            box-shadow: 0 1px 0 #0d0814;
            background: linear-gradient(180deg, #3d2d5c 0%, #201538 100%);
            border-color: #00f0ff;
        }

        /* Floating Level Up Toast Animation */
        @keyframes floatUp {
            0% { opacity: 0; transform: translate(-50%, 20px) scale(0.8); }
            20% { opacity: 1; transform: translate(-50%, 0) scale(1.1); }
            80% { opacity: 1; transform: translate(-50%, 0) scale(1); }
            100% { opacity: 0; transform: translate(-50%, -20px) scale(0.9); }
        }

        .toast-anim {
            animation: floatUp 2.5s ease-out forwards;
        }

        /* Victory Glowing Animation */
        @keyframes rainbowGlow {
            0% { text-shadow: 0 0 10px #ff007f, 0 0 20px #ff007f; color: #ff007f; }
            33% { text-shadow: 0 0 10px #00f0ff, 0 0 20px #00f0ff; color: #00f0ff; }
            66% { text-shadow: 0 0 10px #ffea00, 0 0 20px #ffea00; color: #ffea00; }
            100% { text-shadow: 0 0 10px #ff007f, 0 0 20px #ff007f; color: #ff007f; }
        }

        .victory-title {
            animation: rainbowGlow 2s infinite;
        }
    </style>
</head>
<body class="flex flex-col items-center justify-between min-h-screen p-2 crt-flicker select-none">
    
    <!-- Retro CRT Overlay -->
    <div class="crt-overlay"></div>

    <!-- Level Up Floating Banner -->
    <div id="level-toast" class="hidden fixed top-24 left-1/2 -translate-x-1/2 z-50 bg-black/90 border-2 border-yellow-400 px-6 py-3 rounded-lg text-center shadow-[0_0_20px_#ffea00] toast-anim pointer-events-none">
        <div class="text-xs text-yellow-300 mb-1">¡NIVEL SUBIDO!</div>
        <div id="toast-track-name" class="text-[10px] text-cyan-400">NUEVA MÚSICA ACTIVADA</div>
    </div>

    <!-- Header / Stats Bar -->
    <header class="w-full max-w-md flex justify-between items-center bg-black/60 p-3 rounded-lg border border-purple-900/60 shadow-lg mt-1">
        <div>
            <div class="text-[8px] text-purple-400">PUNTOS</div>
            <div id="score-val" class="text-sm glow-cyan font-bold">000000</div>
        </div>
        <div class="text-center">
            <div class="text-[8px] text-purple-400">MÁXIMA</div>
            <div id="highscore-val" class="text-xs glow-yellow">000000</div>
        </div>
        <div class="text-right">
            <div class="text-[8px] text-purple-400">LÍNEAS</div>
            <div id="lines-val" class="text-xs text-green-400 font-bold">0</div>
        </div>
    </header>

    <!-- Main Game Container -->
    <main class="w-full max-w-md my-2 flex flex-col items-center">
        <!-- Dashboard & Canvas Area -->
        <div class="flex flex-row justify-center items-start gap-2 w-full">
            
            <!-- Left Info Panel: Hold & Audio Toggle -->
            <div class="flex flex-col justify-between items-center gap-2 w-20 sm:w-24">
                <!-- Hold Piece Box -->
                <div class="bg-black/80 border-2 border-purple-800 p-2 rounded-lg text-center w-full">
                    <div class="text-[8px] text-purple-300 mb-1">GUARDAR</div>
                    <div class="flex justify-center items-center h-14">
                        <canvas id="hold-canvas" width="60" height="60" class="max-w-full max-h-full"></canvas>
                    </div>
                </div>

                <!-- Control Buttons for Sound/Music -->
                <div class="w-full space-y-2">
                    <button id="btn-music" class="retro-btn w-full py-2 text-[8px] text-cyan-300 rounded flex flex-col items-center justify-center">
                        <span id="music-icon">🎵</span>
                        <span id="music-text" class="mt-1">MÚS: ON</span>
                    </button>
                    <button id="btn-sfx" class="retro-btn w-full py-2 text-[8px] text-yellow-300 rounded flex flex-col items-center justify-center">
                        <span id="sfx-icon">🔊</span>
                        <span id="sfx-text" class="mt-1">SFX: ON</span>
                    </button>
                </div>
            </div>

            <!-- Canvas Grid Container -->
            <div class="relative arcade-border rounded-lg overflow-hidden bg-black/90">
                <canvas id="tetris-canvas" width="200" height="400" class="block"></canvas>
                
                <!-- Start / Game Over Overlay -->
                <div id="start-overlay" class="absolute inset-0 bg-black/90 flex flex-col items-center justify-center p-4 text-center z-20">
                    <h1 id="overlay-title" class="text-base font-bold glow-pink mb-3 leading-tight">SYNTHWAVE TETRIS</h1>
                    <p id="overlay-subtitle" class="text-[9px] text-cyan-300 mb-4 leading-relaxed">¡Cada 2 min o 10 líneas subes de nivel hasta el Nivel 20!</p>
                    
                    <div id="overlay-stats" class="hidden mb-4 space-y-1 text-[9px] text-gray-300">
                        <p>Puntuación: <span id="final-score" class="text-cyan-400">0</span></p>
                        <p>Nivel Alcanzado: <span id="final-level" class="text-yellow-400">1</span></p>
                        <p>Líneas: <span id="final-lines" class="text-green-400">0</span></p>
                    </div>

                    <button id="start-btn" class="retro-btn px-4 py-3 text-xs text-green-400 rounded-lg font-bold border-green-500 hover:bg-green-900/50 transition">
                        ¡JUGAR AHORA!
                    </button>
                </div>

                <!-- Pause Overlay -->
                <div id="pause-overlay" class="hidden absolute inset-0 bg-black/80 flex flex-col items-center justify-center z-20">
                    <h2 class="text-sm glow-yellow mb-2">PAUSADO</h2>
                    <p class="text-[8px] text-gray-400">Presiona P para Reanudar</p>
                </div>
            </div>

            <!-- Right Info Panel: Next Piece, Level & Timer -->
            <div class="flex flex-col justify-between items-center gap-2 w-20 sm:w-24">
                <!-- Next Piece Box -->
                <div class="bg-black/80 border-2 border-purple-800 p-2 rounded-lg text-center w-full">
                    <div class="text-[8px] text-purple-300 mb-1">SIGUIENTE</div>
                    <div class="flex justify-center items-center h-14">
                        <canvas id="next-canvas" width="60" height="60" class="max-w-full max-h-full"></canvas>
                    </div>
                </div>

                <!-- Level & Timer -->
                <div class="bg-black/80 border-2 border-purple-800 p-2 rounded-lg text-center w-full space-y-2">
                    <div>
                        <div class="text-[7px] text-gray-400">NIVEL</div>
                        <div id="level-val" class="text-xs glow-cyan font-bold mt-1">1</div>
                    </div>
                    <div>
                        <div class="text-[7px] text-gray-400">CAMBIO EN</div>
                        <div id="timer-val" class="text-[10px] text-yellow-400 font-bold mt-1">02:00</div>
                    </div>
                </div>
            </div>
        </div>

        <!-- Track Name Display -->
        <div class="w-full bg-purple-950/40 border border-purple-800/50 rounded p-1 text-center mt-2">
            <span class="text-[7px] text-purple-300">TEMA MUSICAL: </span>
            <span id="track-title" class="text-[8px] text-cyan-300 font-bold">1. Chiptune Chillwave</span>
        </div>

        <!-- Arcade Utility Controls -->
        <div class="flex justify-center gap-2 mt-2 w-full">
            <button id="btn-pause" class="retro-btn px-3 py-1.5 text-[8px] text-gray-300 rounded">PAUSA (P)</button>
            <button id="btn-restart" class="retro-btn px-3 py-1.5 text-[8px] text-gray-300 rounded">REINICIAR</button>
        </div>
    </main>

    <!-- Mobile Touch Controls Section -->
    <footer class="w-full max-w-md px-2 pb-2 flex flex-col gap-2">
        <!-- Primary Action Buttons Row -->
        <div class="grid grid-cols-3 gap-2">
            <button id="touch-hold" class="retro-btn py-3 text-yellow-400 rounded-lg flex flex-col items-center justify-center active:border-yellow-400">
                <span class="text-sm">📦</span>
                <span class="text-[8px] mt-0.5">GUARDAR</span>
            </button>
            <button id="touch-rotate" class="retro-btn py-3 text-cyan-400 rounded-lg flex flex-col items-center justify-center active:border-cyan-400">
                <span class="text-sm">🔄</span>
                <span class="text-[8px] mt-0.5">ROTAR</span>
            </button>
            <button id="touch-hard" class="retro-btn py-3 text-red-400 rounded-lg flex flex-col items-center justify-center active:border-red-400">
                <span class="text-sm">⚡</span>
                <span class="text-[8px] mt-0.5">CAÍDA</span>
            </button>
        </div>

        <!-- Directional D-Pad Controls Row -->
        <div class="grid grid-cols-3 gap-2">
            <button id="touch-left" class="retro-btn py-4 text-xl text-cyan-300 rounded-lg flex items-center justify-center active:border-cyan-300">◄</button>
            <button id="touch-down" class="retro-btn py-4 text-xl text-cyan-300 rounded-lg flex items-center justify-center active:border-cyan-300">▼</button>
            <button id="touch-right" class="retro-btn py-4 text-xl text-cyan-300 rounded-lg flex items-center justify-center active:border-cyan-300">►</button>
        </div>
    </footer>

    <!-- Game Engine Logic & Web Audio Synthesizer -->
    <script>
        // --- CONSTANTS AND CONFIGURATION ---
        const COLS = 10;
        const ROWS = 20;
        const BLOCK_SIZE = 20;
        const MAX_LEVEL = 20; // Final Victory Level

        // Tetromino Shapes Definition
        const TETROMINOES = {
            'I': { shape: [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]], colorIndex: 1 },
            'J': { shape: [[1,0,0],[1,1,1],[0,0,0]], colorIndex: 2 },
            'L': { shape: [[0,0,1],[1,1,1],[0,0,0]], colorIndex: 3 },
            'O': { shape: [[1,1],[1,1]], colorIndex: 4 },
            'S': { shape: [[0,1,1],[1,1,0],[0,0,0]], colorIndex: 5 },
            'T': { shape: [[0,1,0],[1,1,1],[0,0,0]], colorIndex: 6 },
            'Z': { shape: [[1,1,0],[0,1,1],[0,0,0]], colorIndex: 7 }
        };

        // Distinct Visual Theme Palettes per Level (Rotating modulo after 5 themes)
        const LEVEL_THEMES = [
            { name: "Chiptune Chillwave", bg: "#0d0814", grid: "#1f1533", colors: ['#000', '#00f0ff', '#0040ff', '#ffaa00', '#ffea00', '#00ff66', '#a855f7', '#ff0055'] },
            { name: "Upbeat Arps", bg: "#140a24", grid: "#291447", colors: ['#000', '#00ffff', '#3b82f6', '#f59e0b', '#facc15', '#10b981', '#c084fc', '#f43f5e'] },
            { name: "Dark Synthwave", bg: "#1a050d", grid: "#380d1e", colors: ['#000', '#38bdf8', '#6366f1', '#fb923c', '#fde047', '#4ade80', '#e879f9', '#ef4444'] },
            { name: "Hyper Drive", bg: "#051a0e", grid: "#0e381e", colors: ['#000', '#22d3ee', '#818cf8', '#f97316', '#fef08a', '#22c55e', '#f0abfc', '#fb7185'] },
            { name: "Cyberpunk Overdrive", bg: "#1f0019", grid: "#400034", colors: ['#000', '#67e8f9', '#a855f7', '#fbbf24', '#fef08a', '#34d399', '#f472b6', '#ff007f'] }
        ];

        // Music Tracks Config with dynamic tempo scaling
        const MUSIC_TRACKS = [
            {
                name: "Synthwave Chill",
                baseBpm: 110,
                bassLine: [130.81, 130.81, 155.56, 174.61, 130.81, 130.81, 116.54, 98.00],
                leadMelody: [261.63, 311.13, 349.23, 392.00, 466.16, 392.00, 349.23, 311.13]
            },
            {
                name: "Upbeat Arpeggiator",
                baseBpm: 125,
                bassLine: [146.83, 146.83, 174.61, 220.00, 146.83, 146.83, 130.81, 164.81],
                leadMelody: [293.66, 349.23, 440.00, 523.25, 440.00, 349.23, 293.66, 349.23]
            },
            {
                name: "Dark Minor Synth",
                baseBpm: 140,
                bassLine: [110.00, 110.00, 123.47, 130.81, 110.00, 110.00, 103.83, 98.00],
                leadMelody: [220.00, 233.08, 277.18, 293.66, 329.63, 293.66, 277.18, 233.08]
            },
            {
                name: "Hyper Chiptune",
                baseBpm: 155,
                bassLine: [164.81, 164.81, 196.00, 220.00, 246.94, 220.00, 196.00, 164.81],
                leadMelody: [329.63, 392.00, 440.00, 493.88, 587.33, 493.88, 440.00, 392.00]
            },
            {
                name: "Cyberpunk Finale",
                baseBpm: 170,
                bassLine: [123.47, 123.47, 146.83, 164.81, 174.61, 164.81, 146.83, 123.47],
                leadMelody: [246.94, 293.66, 329.63, 349.23, 392.00, 349.23, 329.63, 293.66]
            }
        ];

        // LOGICAL SPEED CURVE FUNCTION (Calculates ms per drop step for Levels 1-19)
        function getDropIntervalForLevel(lvl) {
            // Level 1: 800ms -> Level 19: ~60ms
            if (lvl >= MAX_LEVEL) return 50;
            const startMs = 800;
            const endMs = 60;
            const progress = (lvl - 1) / (MAX_LEVEL - 2); // 0 to 1 scaling from Lvl 1 to 19
            return Math.round(startMs * Math.pow(endMs / startMs, progress));
        }

        // --- GAME ENGINE STATE ---
        let canvas = document.getElementById('tetris-canvas');
        let ctx = canvas.getContext('2d');
        let holdCanvas = document.getElementById('hold-canvas');
        let holdCtx = holdCanvas.getContext('2d');
        let nextCanvas = document.getElementById('next-canvas');
        let nextCtx = nextCanvas.getContext('2d');

        let board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
        let score = 0;
        let highscore = localStorage.getItem('tetris_synthwave_highscore') || 0;
        let level = 1;
        let lines = 0;
        let gameOver = false;
        let gameWon = false;
        let isPaused = false;
        let gameRunning = false;

        const LEVEL_DURATION = 120; // 2 minutes
        let levelTimeRemaining = LEVEL_DURATION;
        let levelTimerInterval = null;

        let currentPiece = null;
        let nextPiece = null;
        let holdPiece = null;
        let canHold = true;

        let lastTime = 0;
        let dropCounter = 0;
        let dropInterval = getDropIntervalForLevel(1);

        let musicEnabled = true;
        let sfxEnabled = true;

        // --- WEB AUDIO SYNTHESIZER ---
        class SynthAudioEngine {
            constructor() {
                this.ctx = null;
                this.musicStep = 0;
                this.musicInterval = null;
                this.initialized = false;
            }

            init() {
                if (!this.ctx) {
                    const AudioContext = window.AudioContext || window.webkitAudioContext;
                    this.ctx = new AudioContext();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
                this.initialized = true;
            }

            playTone(freq, duration, type = 'square', gainVal = 0.1) {
                if (!sfxEnabled || !this.ctx) return;
                try {
                    const osc = this.ctx.createOscillator();
                    const gain = this.ctx.createGain();
                    osc.type = type;
                    osc.frequency.setValueAtTime(freq, this.ctx.currentTime);
                    gain.gain.setValueAtTime(gainVal, this.ctx.currentTime);
                    gain.gain.exponentialRampToValueAtTime(0.0001, this.ctx.currentTime + duration);

                    osc.connect(gain);
                    gain.connect(this.ctx.destination);

                    osc.start();
                    osc.stop(this.ctx.currentTime + duration);
                } catch(e){}
            }

            rotate() { this.playTone(400, 0.08, 'triangle', 0.1); }
            move() { this.playTone(180, 0.05, 'square', 0.05); }
            drop() { this.playTone(120, 0.1, 'sawtooth', 0.15); }
            hold() { this.playTone(320, 0.12, 'sine', 0.12); }
            
            clear() {
                if (!sfxEnabled || !this.ctx) return;
                [440, 554.37, 659.25, 880].forEach((f, i) => {
                    setTimeout(() => this.playTone(f, 0.15, 'square', 0.1), i * 60);
                });
            }

            levelUp() {
                if (!sfxEnabled || !this.ctx) return;
                [300, 400, 500, 700, 900].forEach((f, i) => {
                    setTimeout(() => this.playTone(f, 0.12, 'sawtooth', 0.15), i * 80);
                });
            }

            victoryFanfare() {
                if (!sfxEnabled || !this.ctx) return;
                const notes = [523.25, 659.25, 783.99, 1046.50, 880, 1046.50];
                notes.forEach((f, i) => {
                    setTimeout(() => this.playTone(f, 0.25, 'triangle', 0.2), i * 150);
                });
            }

            gameOver() {
                if (!sfxEnabled || !this.ctx) return;
                [300, 250, 200, 150].forEach((f, i) => {
                    setTimeout(() => this.playTone(f, 0.2, 'sawtooth', 0.2), i * 120);
                });
            }

            startMusicTrack(levelNum) {
                this.stopMusic();
                if (!musicEnabled || !this.ctx) return;

                const trackIndex = (levelNum - 1) % MUSIC_TRACKS.length;
                const track = MUSIC_TRACKS[trackIndex];

                // Dynamically speed up BPM slightly as level increases
                const currentBpm = track.baseBpm + Math.floor((levelNum - 1) * 1.5);
                const stepDuration = (60 / currentBpm) / 2;
                this.musicStep = 0;

                document.getElementById('track-title').textContent = `${levelNum}. ${track.name}`;

                this.musicInterval = setInterval(() => {
                    if (!gameRunning || isPaused || gameOver || gameWon) return;

                    const now = this.ctx.currentTime;

                    // Bass Line
                    const bassFreq = track.bassLine[this.musicStep % track.bassLine.length];
                    const bassOsc = this.ctx.createOscillator();
                    const bassGain = this.ctx.createGain();
                    bassOsc.type = 'sawtooth';
                    bassOsc.frequency.setValueAtTime(bassFreq, now);
                    bassGain.gain.setValueAtTime(0.04, now);
                    bassGain.gain.exponentialRampToValueAtTime(0.001, now + stepDuration * 0.9);
                    bassOsc.connect(bassGain);
                    bassGain.connect(this.ctx.destination);
                    bassOsc.start(now);
                    bassOsc.stop(now + stepDuration * 0.9);

                    // Lead Melody
                    if (this.musicStep % 2 === 0) {
                        const leadFreq = track.leadMelody[(this.musicStep / 2) % track.leadMelody.length];
                        const leadOsc = this.ctx.createOscillator();
                        const leadGain = this.ctx.createGain();
                        leadOsc.type = 'square';
                        leadOsc.frequency.setValueAtTime(leadFreq, now);
                        leadGain.gain.setValueAtTime(0.03, now);
                        leadGain.gain.exponentialRampToValueAtTime(0.0005, now + stepDuration * 1.5);
                        leadOsc.connect(leadGain);
                        leadGain.connect(this.ctx.destination);
                        leadOsc.start(now);
                        leadOsc.stop(now + stepDuration * 1.5);
                    }

                    // Percussion
                    if (this.musicStep % 4 === 2) {
                        this.playTone(800 + Math.random() * 400, 0.03, 'triangle', 0.02);
                    }

                    this.musicStep++;
                }, stepDuration * 1000);
            }

            stopMusic() {
                if (this.musicInterval) {
                    clearInterval(this.musicInterval);
                    this.musicInterval = null;
                }
            }
        }

        const audio = new SynthAudioEngine();
        document.getElementById('highscore-val').textContent = String(highscore).padStart(6, '0');

        function getRandomPiece() {
            const keys = Object.keys(TETROMINOES);
            const randKey = keys[Math.floor(Math.random() * keys.length)];
            const tetromino = TETROMINOES[randKey];
            return {
                shape: tetromino.shape.map(row => [...row]),
                colorIndex: tetromino.colorIndex,
                x: Math.floor(COLS / 2) - Math.ceil(tetromino.shape[0].length / 2),
                y: 0
            };
        }

        function applyLevelTheme(levelNum) {
            const themeIndex = (levelNum - 1) % LEVEL_THEMES.length;
            const theme = LEVEL_THEMES[themeIndex];
            document.body.style.backgroundColor = theme.bg;
            audio.startMusicTrack(levelNum);
        }

        function collide(piece, offset = { x: 0, y: 0 }) {
            const { shape, x, y } = piece;
            for (let r = 0; r < shape.length; r++) {
                for (let c = 0; c < shape[r].length; c++) {
                    if (shape[r][c] !== 0) {
                        let newX = x + c + offset.x;
                        let newY = y + r + offset.y;
                        if (newX < 0 || newX >= COLS || newY >= ROWS) return true;
                        if (newY >= 0 && board[newY][newX] !== 0) return true;
                    }
                }
            }
            return false;
        }

        function rotateMatrix(matrix) {
            const N = matrix.length;
            const result = Array.from({ length: N }, () => Array(N).fill(0));
            for (let r = 0; r < N; r++) {
                for (let c = 0; c < N; c++) {
                    result[c][N - 1 - r] = matrix[r][c];
                }
            }
            return result;
        }

        function playerRotate() {
            if (gameOver || gameWon || isPaused || !gameRunning) return;
            const originalShape = currentPiece.shape;
            currentPiece.shape = rotateMatrix(currentPiece.shape);

            let offset = 1;
            while (collide(currentPiece)) {
                currentPiece.x += offset;
                offset = -(offset + (offset > 0 ? 1 : -1));
                if (offset > currentPiece.shape[0].length) {
                    currentPiece.shape = originalShape;
                    return;
                }
            }
            audio.rotate();
        }

        function playerMove(dir) {
            if (gameOver || gameWon || isPaused || !gameRunning) return;
            if (!collide(currentPiece, { x: dir, y: 0 })) {
                currentPiece.x += dir;
                audio.move();
            }
        }

        function playerDrop() {
            if (gameOver || gameWon || isPaused || !gameRunning) return;
            if (!collide(currentPiece, { x: 0, y: 1 })) {
                currentPiece.y++;
                dropCounter = 0;
            } else {
                lockPiece();
            }
        }

        function playerHardDrop() {
            if (gameOver || gameWon || isPaused || !gameRunning) return;
            while (!collide(currentPiece, { x: 0, y: 1 })) {
                currentPiece.y++;
            }
            audio.drop();
            lockPiece();
        }

        function playerHold() {
            if (gameOver || gameWon || isPaused || !gameRunning || !canHold) return;
            
            audio.hold();
            if (!holdPiece) {
                holdPiece = { ...currentPiece, shape: TETROMINOES[getPieceKey(currentPiece)].shape };
                currentPiece = nextPiece;
                nextPiece = getRandomPiece();
                drawSidePreview(nextCtx, nextPiece);
            } else {
                const temp = { ...currentPiece, shape: TETROMINOES[getPieceKey(currentPiece)].shape };
                currentPiece = {
                    shape: TETROMINOES[getPieceKey(holdPiece)].shape,
                    colorIndex: holdPiece.colorIndex,
                    x: Math.floor(COLS / 2) - Math.ceil(holdPiece.shape[0].length / 2),
                    y: 0
                };
                holdPiece = temp;
            }
            canHold = false;
            drawSidePreview(holdCtx, holdPiece);
        }

        function getPieceKey(piece) {
            for (let key in TETROMINOES) {
                if (TETROMINOES[key].colorIndex === piece.colorIndex) return key;
            }
            return 'I';
        }

        function lockPiece() {
            currentPiece.shape.forEach((row, r) => {
                row.forEach((val, c) => {
                    if (val !== 0) {
                        const boardY = currentPiece.y + r;
                        const boardX = currentPiece.x + c;
                        if (boardY >= 0) {
                            board[boardY][boardX] = currentPiece.colorIndex;
                        }
                    }
                });
            });

            checkLineClears();

            currentPiece = nextPiece;
            nextPiece = getRandomPiece();
            canHold = true;

            drawSidePreview(nextCtx, nextPiece);

            if (collide(currentPiece)) {
                triggerGameOver();
            }
        }

        function checkLineClears() {
            let linesCleared = 0;

            outer: for (let r = ROWS - 1; r >= 0; r--) {
                for (let c = 0; c < COLS; c++) {
                    if (board[r][c] === 0) continue outer;
                }
                board.splice(r, 1);
                board.unshift(Array(COLS).fill(0));
                linesCleared++;
                r++;
            }

            if (linesCleared > 0) {
                const baseScores = [0, 100, 300, 500, 800];
                score += baseScores[linesCleared] * level;
                lines += linesCleared;
                audio.clear();
                updateScoreUI();

                if (lines >= level * 10) {
                    advanceLevel("¡10 LÍNEAS COMPLETADAS!");
                }
            }
        }

        function startLevelTimer() {
            clearInterval(levelTimerInterval);
            levelTimerInterval = setInterval(() => {
                if (gameRunning && !isPaused && !gameOver && !gameWon) {
                    levelTimeRemaining--;
                    updateTimerDisplay();

                    if (levelTimeRemaining <= 0) {
                        advanceLevel("¡TIEMPO TRANSCURRIDO!");
                    }
                }
            }, 1000);
        }

        function advanceLevel(reasonText = "") {
            if (level >= MAX_LEVEL) return;

            level++;

            // Trigger Final Game Victory at Level 20!
            if (level === MAX_LEVEL) {
                triggerVictory();
                return;
            }

            levelTimeRemaining = LEVEL_DURATION;
            audio.levelUp();
            applyLevelTheme(level);

            // Update fall speed using smooth exponential curve
            dropInterval = getDropIntervalForLevel(level);

            showLevelToast(reasonText);
            updateScoreUI();
            updateTimerDisplay();
        }

        function showLevelToast(reason) {
            const toast = document.getElementById('level-toast');
            const trackName = document.getElementById('toast-track-name');
            const currentTrack = MUSIC_TRACKS[(level - 1) % MUSIC_TRACKS.length];
            
            trackName.textContent = `${reason} 🎵 ${currentTrack.name}`;
            toast.classList.remove('hidden');
            
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 2500);
        }

        function updateTimerDisplay() {
            const minutes = Math.floor(levelTimeRemaining / 60);
            const seconds = levelTimeRemaining % 60;
            document.getElementById('timer-val').textContent = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
        }

        function updateScoreUI() {
            document.getElementById('score-val').textContent = String(score).padStart(6, '0');
            document.getElementById('lines-val').textContent = lines;
            document.getElementById('level-val').textContent = level;

            if (score > highscore) {
                highscore = score;
                localStorage.setItem('tetris_synthwave_highscore', highscore);
                document.getElementById('highscore-val').textContent = String(highscore).padStart(6, '0');
            }
        }

        function drawBlock(ctxTarget, x, y, colorIndex, isGhost = false) {
            const themeIndex = (level - 1) % LEVEL_THEMES.length;
            const color = LEVEL_THEMES[themeIndex].colors[colorIndex] || '#fff';

            ctxTarget.fillStyle = isGhost ? 'rgba(255, 255, 255, 0.15)' : color;
            ctxTarget.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, BLOCK_SIZE);

            if (!isGhost) {
                ctxTarget.fillStyle = 'rgba(255, 255, 255, 0.3)';
                ctxTarget.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, BLOCK_SIZE, 3);
                ctxTarget.fillRect(x * BLOCK_SIZE, y * BLOCK_SIZE, 3, BLOCK_SIZE);

                ctxTarget.fillStyle = 'rgba(0, 0, 0, 0.35)';
                ctxTarget.fillRect(x * BLOCK_SIZE, (y + 1) * BLOCK_SIZE - 3, BLOCK_SIZE, 3);
                ctxTarget.fillRect((x + 1) * BLOCK_SIZE - 3, y * BLOCK_SIZE, 3, BLOCK_SIZE);
            } else {
                ctxTarget.strokeStyle = color;
                ctxTarget.lineWidth = 1;
                ctxTarget.strokeRect(x * BLOCK_SIZE + 1, y * BLOCK_SIZE + 1, BLOCK_SIZE - 2, BLOCK_SIZE - 2);
            }
        }

        function drawSidePreview(ctxTarget, piece) {
            ctxTarget.clearRect(0, 0, ctxTarget.canvas.width, ctxTarget.canvas.height);
            if (!piece) return;

            const shape = piece.shape;
            const offsetX = (ctxTarget.canvas.width - shape[0].length * 15) / 2;
            const offsetY = (ctxTarget.canvas.height - shape.length * 15) / 2;

            const themeIndex = (level - 1) % LEVEL_THEMES.length;
            const color = LEVEL_THEMES[themeIndex].colors[piece.colorIndex];

            ctxTarget.fillStyle = color;
            shape.forEach((row, r) => {
                row.forEach((val, c) => {
                    if (val !== 0) {
                        ctxTarget.fillRect(offsetX + c * 15, offsetY + r * 15, 13, 13);
                    }
                });
            });
        }

        function getGhostY() {
            let ghostY = currentPiece.y;
            while (!collide(currentPiece, { x: 0, y: ghostY - currentPiece.y + 1 })) {
                ghostY++;
            }
            return ghostY;
        }

        function draw() {
            const themeIndex = (level - 1) % LEVEL_THEMES.length;
            const theme = LEVEL_THEMES[themeIndex];

            ctx.fillStyle = theme.bg;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            ctx.strokeStyle = theme.grid;
            ctx.lineWidth = 0.5;
            for (let r = 0; r < ROWS; r++) {
                ctx.beginPath();
                ctx.moveTo(0, r * BLOCK_SIZE);
                ctx.lineTo(canvas.width, r * BLOCK_SIZE);
                ctx.stroke();
            }
            for (let c = 0; c < COLS; c++) {
                ctx.beginPath();
                ctx.moveTo(c * BLOCK_SIZE, 0);
                ctx.lineTo(c * BLOCK_SIZE, canvas.height);
                ctx.stroke();
            }

            board.forEach((row, r) => {
                row.forEach((val, c) => {
                    if (val !== 0) drawBlock(ctx, c, r, val);
                });
            });

            if (currentPiece && !gameOver && !gameWon) {
                const ghostY = getGhostY();
                currentPiece.shape.forEach((row, r) => {
                    row.forEach((val, c) => {
                        if (val !== 0) {
                            drawBlock(ctx, currentPiece.x + c, ghostY + r, currentPiece.colorIndex, true);
                        }
                    });
                });

                currentPiece.shape.forEach((row, r) => {
                    row.forEach((val, c) => {
                        if (val !== 0) {
                            drawBlock(ctx, currentPiece.x + c, currentPiece.y + r, currentPiece.colorIndex);
                        }
                    });
                });
            }
        }

        function update(time = 0) {
            const deltaTime = time - lastTime;
            lastTime = time;

            if (gameRunning && !isPaused && !gameOver && !gameWon) {
                dropCounter += deltaTime;
                if (dropCounter > dropInterval) {
                    playerDrop();
                }
            }

            draw();
            requestAnimationFrame(update);
        }

        function startGame() {
            audio.init();
            board = Array.from({ length: ROWS }, () => Array(COLS).fill(0));
            score = 0;
            level = 1;
            lines = 0;
            dropInterval = getDropIntervalForLevel(1);
            levelTimeRemaining = LEVEL_DURATION;
            gameOver = false;
            gameWon = false;
            isPaused = false;
            gameRunning = true;
            holdPiece = null;
            canHold = true;

            applyLevelTheme(level);
            updateScoreUI();
            updateTimerDisplay();
            startLevelTimer();

            currentPiece = getRandomPiece();
            nextPiece = getRandomPiece();

            drawSidePreview(nextCtx, nextPiece);
            holdCtx.clearRect(0, 0, holdCanvas.width, holdCanvas.height);

            const title = document.getElementById('overlay-title');
            title.className = "text-base font-bold glow-pink mb-3 leading-tight";
            document.getElementById('overlay-subtitle').textContent = "¡Cada 2 min o 10 líneas subes de nivel hasta el Nivel 20!";
            document.getElementById('start-overlay').classList.add('hidden');
            document.getElementById('pause-overlay').classList.add('hidden');
        }

        function triggerVictory() {
            gameWon = true;
            gameRunning = false;
            clearInterval(levelTimerInterval);
            audio.stopMusic();
            audio.victoryFanfare();

            const title = document.getElementById('overlay-title');
            title.textContent = "🏆 ¡VICTORIA RETRO! 🏆";
            title.className = "text-sm font-bold victory-title mb-2 leading-tight";
            
            document.getElementById('overlay-subtitle').textContent = "¡HAS COMPLETADO EL NIVEL 20 Y LOGRADO LA MÁXIMA VICTORIA!";
            document.getElementById('final-score').textContent = score;
            document.getElementById('final-level').textContent = "20 (MÁXIMO)";
            document.getElementById('final-lines').textContent = lines;
            document.getElementById('overlay-stats').classList.remove('hidden');
            document.getElementById('start-btn').textContent = "JUGAR DE NUEVO";
            document.getElementById('start-overlay').classList.remove('hidden');
        }

        function triggerGameOver() {
            gameOver = true;
            gameRunning = false;
            clearInterval(levelTimerInterval);
            audio.stopMusic();
            audio.gameOver();

            const title = document.getElementById('overlay-title');
            title.textContent = "JUEGO TERMINADO";
            title.className = "text-base font-bold glow-pink mb-3 leading-tight";

            document.getElementById('overlay-subtitle').textContent = "¡Inténtalo otra vez para llegar al Nivel 20!";
            document.getElementById('final-score').textContent = score;
            document.getElementById('final-level').textContent = level;
            document.getElementById('final-lines').textContent = lines;
            document.getElementById('overlay-stats').classList.remove('hidden');
            document.getElementById('start-btn').textContent = "VOLVER A JUGAR";
            document.getElementById('start-overlay').classList.remove('hidden');
        }

        function togglePause() {
            if (!gameRunning || gameOver || gameWon) return;
            isPaused = !isPaused;
            document.getElementById('pause-overlay').classList.toggle('hidden', !isPaused);
            if (isPaused) {
                audio.stopMusic();
            } else {
                audio.startMusicTrack(level);
            }
        }

        // --- INPUT HANDLING ---
        document.addEventListener('keydown', (e) => {
            if (['ArrowUp', 'ArrowDown', 'ArrowLeft', 'ArrowRight', ' '].includes(e.key)) {
                e.preventDefault();
            }
            if (!gameRunning) return;

            switch (e.key) {
                case 'ArrowLeft': case 'a': case 'A': playerMove(-1); break;
                case 'ArrowRight': case 'd': case 'D': playerMove(1); break;
                case 'ArrowDown': case 's': case 'S': playerDrop(); break;
                case 'ArrowUp': case 'w': case 'W': playerRotate(); break;
                case ' ': playerHardDrop(); break;
                case 'c': case 'C': case 'Shift': playerHold(); break;
                case 'p': case 'P': togglePause(); break;
            }
        });

        function bindTouchControl(id, action, allowRepeat = false) {
            const btn = document.getElementById(id);
            let repeatTimer = null;

            const handleStart = (e) => {
                if (e.cancelable) e.preventDefault();
                audio.init();
                btn.classList.add('active');
                action();
                if (allowRepeat) {
                    repeatTimer = setInterval(action, 90);
                }
            };

            const handleEnd = (e) => {
                if (e.cancelable) e.preventDefault();
                btn.classList.remove('active');
                if (repeatTimer) {
                    clearInterval(repeatTimer);
                    repeatTimer = null;
                }
            };

            btn.addEventListener('touchstart', handleStart, { passive: false });
            btn.addEventListener('touchend', handleEnd, { passive: false });
            btn.addEventListener('touchcancel', handleEnd, { passive: false });
            btn.addEventListener('mousedown', handleStart);
            btn.addEventListener('mouseup', handleEnd);
            btn.addEventListener('mouseleave', handleEnd);
        }

        bindTouchControl('touch-left', () => playerMove(-1), true);
        bindTouchControl('touch-right', () => playerMove(1), true);
        bindTouchControl('touch-down', () => playerDrop(), true);
        bindTouchControl('touch-rotate', () => playerRotate(), false);
        bindTouchControl('touch-hard', () => playerHardDrop(), false);
        bindTouchControl('touch-hold', () => playerHold(), false);

        let touchStartX = 0;
        let touchStartY = 0;

        canvas.addEventListener('touchstart', (e) => {
            audio.init();
            touchStartX = e.touches[0].clientX;
            touchStartY = e.touches[0].clientY;
        }, { passive: true });

        canvas.addEventListener('touchend', (e) => {
            if (!gameRunning || isPaused) return;
            const deltaX = e.changedTouches[0].clientX - touchStartX;
            const deltaY = e.changedTouches[0].clientY - touchStartY;

            if (Math.abs(deltaX) < 15 && Math.abs(deltaY) < 15) {
                playerRotate();
            } else if (Math.abs(deltaX) > Math.abs(deltaY)) {
                if (deltaX > 30) playerMove(1);
                else if (deltaX < -30) playerMove(-1);
            } else {
                if (deltaY > 40) playerHardDrop();
            }
        }, { passive: true });

        document.getElementById('start-btn').addEventListener('click', startGame);
        document.getElementById('btn-pause').addEventListener('click', togglePause);
        document.getElementById('btn-restart').addEventListener('click', startGame);

        document.getElementById('btn-music').addEventListener('click', () => {
            musicEnabled = !musicEnabled;
            document.getElementById('music-text').textContent = musicEnabled ? "MÚS: ON" : "MÚS: OFF";
            document.getElementById('btn-music').classList.toggle('opacity-50', !musicEnabled);
            if (musicEnabled && gameRunning && !isPaused) {
                audio.startMusicTrack(level);
            } else {
                audio.stopMusic();
            }
        });

        document.getElementById('btn-sfx').addEventListener('click', () => {
            sfxEnabled = !sfxEnabled;
            document.getElementById('sfx-text').textContent = sfxEnabled ? "SFX: ON" : "SFX: OFF";
            document.getElementById('btn-sfx').classList.toggle('opacity-50', !sfxEnabled);
        });

        requestAnimationFrame(update);
    </script>
</body>
</html>