<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Stopwatch</title>
    <style>
        body {
            font-family: 'Arial', sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
            transition: background-color 0.3s, color 0.3s;
        }

        body.dark-mode {
            background-color: #121212;
            color: #ffffff;
        }

        .stopwatch {
            background-color: white;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
            text-align: center;
            width: 350px;
            transition: background-color 0.3s, box-shadow 0.3s;
        }

        body.dark-mode .stopwatch {
            background-color: #1e1e1e;
            box-shadow: 0 10px 30px rgba(255, 255, 255, 0.1);
        }

        .display {
            font-size: 3em;
            margin: 20px 0;
            font-family: 'Courier New', monospace;
            color: #333;
            transition: color 0.3s;
        }

        body.dark-mode .display {
            color: #ffffff;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 10px;
            margin-top: 20px;
        }

        .controls button {
            padding: 12px 24px;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 1em;
            font-weight: bold;
            transition: background-color 0.3s, transform 0.2s;
        }

        .controls button:hover {
            transform: scale(1.05);
        }

        #startPause {
            background-color: #4CAF50;
            color: white;
        }

        #startPause.running {
            background-color: #ff9800;
        }

        #reset {
            background-color: #f44336;
            color: white;
        }

        #lap {
            background-color: #2196F3;
            color: white;
        }

        .laps {
            margin-top: 20px;
            max-height: 200px;
            overflow-y: auto;
            padding: 10px;
            border-radius: 8px;
            background-color: #f9f9f9;
            transition: background-color 0.3s;
        }

        body.dark-mode .laps {
            background-color: #2c2c2c;
        }

        .lap-time {
            padding: 8px;
            border-bottom: 1px solid #eee;
            font-size: 0.9em;
            color: #555;
            transition: color 0.3s;
        }

        body.dark-mode .lap-time {
            color: #ccc;
        }

        .lap-time:last-child {
            border-bottom: none;
        }

        .dark-mode-toggle {
            margin-top: 20px;
            padding: 10px 20px;
            background-color: #333;
            color: white;
            border: none;
            border-radius: 8px;
            cursor: pointer;
            font-size: 0.9em;
            transition: background-color 0.3s;
        }

        body.dark-mode .dark-mode-toggle {
            background-color: #555;
        }
    </style>
</head>
<body>
    <div class="stopwatch">
        <div class="display" id="display">00:00.00</div>
        <div class="controls">
            <button id="startPause">Start</button>
            <button id="lap">Lap</button>
            <button id="reset">Reset</button>
        </div>
        <div class="laps" id="laps"></div>
    </div>
    <button class="dark-mode-toggle" id="darkModeToggle">Dark Mode</button>

    <script>
        class Stopwatch {
            constructor(displayElement, lapsContainer) {
                this.display = displayElement;
                this.laps = lapsContainer;
                this.running = false;
                this.startTime = null;
                this.elapsedTime = 0;
                this.timer = null;
                this.lapTimes = [];

                this.startPauseBtn = document.getElementById('startPause');
                this.lapBtn = document.getElementById('lap');
                this.resetBtn = document.getElementById('reset');

                this.bindEvents();
            }

            bindEvents() {
                this.startPauseBtn.addEventListener('click', () => this.toggle());
                this.lapBtn.addEventListener('click', () => this.recordLap());
                this.resetBtn.addEventListener('click', () => this.reset());
            }

            toggle() {
                if (this.running) {
                    this.pause();
                } else {
                    this.start();
                }
            }

            start() {
                if (!this.running) {
                    this.running = true;
                    this.startTime = Date.now() - this.elapsedTime;
                    this.timer = setInterval(() => this.update(), 10);
                    this.startPauseBtn.textContent = 'Pause';
                    this.startPauseBtn.classList.add('running');
                }
            }

            pause() {
                if (this.running) {
                    this.running = false;
                    clearInterval(this.timer);
                    this.elapsedTime = Date.now() - this.startTime;
                    this.startPauseBtn.textContent = 'Start';
                    this.startPauseBtn.classList.remove('running');
                }
            }

            reset() {
                this.running = false;
                clearInterval(this.timer);
                this.elapsedTime = 0;
                this.startTime = null;
                this.lapTimes = [];
                this.updateDisplay();
                this.clearLaps();
                this.startPauseBtn.textContent = 'Start';
                this.startPauseBtn.classList.remove('running');
            }

            recordLap() {
                if (this.running) {
                    const currentTime = Date.now() - this.startTime;
                    this.lapTimes.push(currentTime);
                    this.updateLaps();
                }
            }

            update() {
                this.elapsedTime = Date.now() - this.startTime;
                this.updateDisplay();
            }

            updateDisplay() {
                const time = this.formatTime(this.elapsedTime);
                this.display.textContent = time;
            }

            updateLaps() {
                this.laps.innerHTML = '';
                this.lapTimes.forEach((time, index) => {
                    const lapElement = document.createElement('div');
                    lapElement.className = 'lap-time';
                    lapElement.textContent = `Lap ${index + 1}: ${this.formatTime(time)}`;
                    this.laps.appendChild(lapElement);
                });
            }

            clearLaps() {
                this.laps.innerHTML = '';
            }

            formatTime(milliseconds) {
                const totalSeconds = Math.floor(milliseconds / 1000);
                const minutes = Math.floor(totalSeconds / 60);
                const seconds = totalSeconds % 60;
                const cents = Math.floor((milliseconds % 1000) / 10);
                
                return `${this.pad(minutes)}:${this.pad(seconds)}.${this.pad(cents)}`;
            }

            pad(number) {
                return number.toString().padStart(2, '0');
            }
        }

        // Initialize the stopwatch
        const display = document.getElementById('display');
        const laps = document.getElementById('laps');
        const stopwatch = new Stopwatch(display, laps);

        // Dark mode toggle
        const darkModeToggle = document.getElementById('darkModeToggle');
        darkModeToggle.addEventListener('click', () => {
            document.body.classList.toggle('dark-mode');
            darkModeToggle.textContent = document.body.classList.contains('dark-mode') ? 'Light Mode' : 'Dark Mode';
        });
    </script>
</body>
</html>
