<script>
    import { moveWindow, Position } from "@tauri-apps/plugin-positioner";
    import { getCurrentWindow, PhysicalSize } from "@tauri-apps/api/window";
    import { exit } from "@tauri-apps/plugin-process";
    import { TrayIcon } from "@tauri-apps/api/tray";
    import { Menu } from "@tauri-apps/api/menu";
    import { defaultWindowIcon } from "@tauri-apps/api/app";

    import bellSound from "./audio/bell.mp3";
    const bellAudio = new Audio(bellSound);

    moveWindow(Position.TopCenter);

    // window expansion (maximize);
    let isExpanded = $state(false);
    async function toggleExpansion() {
        if (isExpanded) {
            isExpanded = false;
            setTimeout(() => {
                getCurrentWindow().setSize(new PhysicalSize(200, 41));
            }, 300);
        } else {
            await getCurrentWindow().setSize(new PhysicalSize(200, 120));
            isExpanded = true;
        }
    }

    // mute
    let isMuted = $state(false);
    async function toggleMute() {
        if (isMuted) {
            isMuted = false;
        } else {
            isMuted = true;
        }
    }

    // pomodoro
    const presetTime = 1500;
    let currentTime = $state(presetTime);
    let isTimerRunning = $state(false);
    let timer;

    function startTimer() {
        if (!isTimerRunning) {
            isExpanded = false;
            isTimerRunning = true;
            // currentTime -= 60;
            currentTime--;

            timer = setInterval(() => {
                currentTime--;

                if (currentTime <= 0) {
                    clearInterval(timer);
                    currentTime = presetTime;
                    isTimerRunning = false;
                    bellAudio.play();
                }
            }, 1000);
        }
    }

    function pauseTimer() {
        if (isTimerRunning) {
            clearInterval(timer);
            isTimerRunning = false;
        }
    }

    function resetTimer() {
        if (isTimerRunning) {
            clearInterval(timer);
            currentTime = presetTime;
            isTimerRunning = false;
        }
    }

    function formatTime(time) {
        let mins = Math.floor(time / 60);
        let secs = time % 60;
        let formattedTime =
            String(mins).padStart(2, "0") + ":" + String(secs).padStart(2, "0");

        return formattedTime;
    }

    // time editing
    let timeSliderValue = $state(1500);
    $effect(() => {
        if (isExpanded) {
            currentTime = timeSliderValue;
        }
    });

    // exiting app
    async function closeApp() {
        await exit(1);
    }

    const menu = await Menu.new({
        items: [
            {
                id: "quit",
                text: "Quit",
                action: closeApp,
            },
            {
                id: "about",
                text: "About",
            },
        ],
    });

    const options = {
        icon: await defaultWindowIcon(),
        menu,
        menuOnLeftClick: true,
    };

    const tray = await TrayIcon.new(options);
</script>

<div class="mini-mode-text">{formatTime(currentTime)}</div>

<div class={isTimerRunning ? "container timer-running" : "container"}>
    <img class="decor" src="/Decor.svg" alt="decor" />

    <div class={isExpanded ? "pomoBX open" : "pomoBX"}>
        <div class="pomodoro-main-bx">
            {#if isTimerRunning}
                <button
                    class="pomoButton"
                    aria-label="settings"
                    onclick={() => {
                        resetTimer();
                    }}
                >
                    <img
                        src="/icons/ClockCounterClockwise.svg"
                        alt="reset timer"
                    />
                </button>
            {:else}
                <button
                    class="pomoButton"
                    aria-label="settings"
                    onclick={() => {
                        toggleExpansion();
                    }}
                >
                    <img src="/icons/FadersHorizontal.svg" alt="settings" />
                </button>
            {/if}

            <div class="pomo">{formatTime(currentTime)}</div>

            {#if isTimerRunning}
                <button
                    class="pomoButton"
                    aria-label="pause"
                    onclick={() => {
                        pauseTimer();
                    }}
                >
                    <img src="/icons/Pause.svg" alt="pause" />
                </button>
            {:else}
                <button
                    class="pomoButton"
                    aria-label="play"
                    onclick={() => {
                        startTimer();
                    }}
                >
                    <img src="/icons/Play.svg" alt="play" />
                </button>
            {/if}
        </div>

        <input
            type="range"
            name="range"
            min="300"
            max="3600"
            step="300"
            id="inputRange"
            class="inputRange"
            bind:value={timeSliderValue}
        />

        <div class="bottom-bx">
            <button
                class="exit"
                aria-label="exit"
                onclick={() => {
                    closeApp();
                }}
            >
                <img src="/icons/DoorOpen.svg" alt="exit" />
            </button>
        </div>
    </div>

    <img class="decor alt" src="/Decor.svg" alt="decor" />
</div>

<style>
    @import "./styles.css";
</style>
