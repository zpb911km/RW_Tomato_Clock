<template>
    <div ref="main" class="rw-clock" @mousedown="dragStart($event)">
        <div ref="outer" class="rw-clock-outer">
            <div
                ref="inner"
                class="rw-clock-inner"
                @click="handleInnerClick($event)"
                @contextmenu="handleInnerRightClick($event)"
                @wheel="handleInnerWheel($event)"
            ></div>
            <div
                class="rw-clock-ring"
                v-for="i in ringCount"
                :key="i"
                :style="getRingPosition(i)"
            >
                <img :src="ringStates[i - 1]" alt="" class="ring-dot" />
            </div>
        </div>
    </div>
</template>
<script setup lang="ts">
import { ref, computed, onMounted, watch } from "vue";
import { getCurrentWebviewWindow } from "@tauri-apps/api/webviewWindow";
import { PhysicalSize, PhysicalPosition } from "@tauri-apps/api/dpi";

// 导入所有需要的图片资源
import img1 from "../assets/1_transparent.png";
import img2 from "../assets/2_transparent.png";
import img3 from "../assets/3_transparent.png";
import img4 from "../assets/4_transparent.png";
import img5 from "../assets/5_transparent.png";
import img6 from "../assets/6_transparent.png";
import img7 from "../assets/7_transparent.png";
import img8 from "../assets/8_transparent.png";
import img9 from "../assets/9_transparent.png";
import img10 from "../assets/10_transparent.png";
import ring_full_img from "../assets/full_20.png";
import ring_half_img from "../assets/half_20.png";
import ring_none_img from "../assets/transparent_20.png";

// UI组件的状态变量
const main = ref<HTMLDivElement | null>(null);
const outer = ref<HTMLDivElement | null>(null);
const inner = ref<HTMLDivElement | null>(null);
const width = 300;
const height = width;
const count = ref(0);
let dragOffset: { x: number; y: number } = { x: 0, y: 0 };
const ringCount = 20;
const ringRadius = 140;
const isFlashing = ref(false);
let setted_time = 0;

// 番茄钟应用变量
enum AppStatus {
    Running,
    Paused,
    Setting,
}
const inner_n = 60;
const status = ref(AppStatus.Running);
const timeLeft = ref(0);
let countDown: any;
const start = async () => {
    countDown = setInterval(async () => {
        if (status.value === AppStatus.Running) {
            timeLeft.value--;
            if (timeLeft.value <= 0) {
                status.value = AppStatus.Setting;
                clearInterval(countDown);
            }
            await resize_webview_add();
            console.log(timeLeft.value);
        }
    }, 1000);
};
const ringPercent = computed(() => {
    return ((timeLeft.value % 60) / 60) * 100;
});
const centerLevel = computed(() => {
    return Math.floor((timeLeft.value / inner_n) % 10) + 1;
});
watch(centerLevel, (newVal) => {
    drawInner(newVal);
});

watch(timeLeft, async (newVal) => {
    if (newVal === 0) {
        isFlashing.value = true;
        if (main.value) {
            while (isFlashing.value) {
                main.value.classList.toggle('flashing');
                await resize_webview_add();
                await new Promise((resolve) => setTimeout(resolve, 500));
            }
        }
    } else {
        isFlashing.value = false;
        if (main.value) {
            main.value.classList.remove('flashing');
        }
    }
});
const start_pause_continue = async () => {
    if (status.value === AppStatus.Running) {
        status.value = AppStatus.Paused;
        clearInterval(countDown);
    } else if (status.value === AppStatus.Paused) {
        status.value = AppStatus.Running;
        start();
    } else if (status.value === AppStatus.Setting) {
        status.value = AppStatus.Running;
        start();
    }
};

const stop = async () => {
    status.value = AppStatus.Setting;
    timeLeft.value = setted_time;
    await resize_webview_add();
    clearInterval(countDown);
};

const add_minute = async (delta: number) => {
    if (status.value === AppStatus.Setting) {
        timeLeft.value += delta;
        setted_time = timeLeft.value;
        await resize_webview_add();
    }
};

const minus_minute = async (delta: number) => {
    if (status.value === AppStatus.Setting) {
        timeLeft.value -= delta;
        if (timeLeft.value < 0) {
            timeLeft.value = 0;
        }
        setted_time = timeLeft.value;
        await resize_webview_add();
    }
};

const handleInnerClick = async (event: MouseEvent) => {
    if (event.clientX === dragOffset.x && event.clientY === dragOffset.y) {
        start_pause_continue();
    }
};

const handleInnerRightClick = async (event: MouseEvent) => {
    event.preventDefault();
    console.log("inner right click");
    stop();
};

const handleInnerWheel = async (event: WheelEvent) => {
    event.preventDefault();
    const delta = event.deltaY;
    if (status.value === AppStatus.Running) {
        return;
    }
    clearInterval(countDown);
    console.log(delta);
    if (delta < 0) {
        add_minute(30);
    } else if (delta > 0) {
        minus_minute(30);
    }
};

// 创建图片映射
const imageMap = {
    1: img1,
    2: img2,
    3: img3,
    4: img4,
    5: img5,
    6: img6,
    7: img7,
    8: img8,
    9: img9,
    10: img10,
};

// 创建环形点状态数组的计算属性
const ringStates = computed(() => {
    const states = [];
    for (let i = 1; i <= ringCount; i++) {
        const sp = ringCount - Math.ceil((ringPercent.value * ringCount) / 100);
        const hf =
            Math.ceil((ringPercent.value * ringCount) / 100) -
            (ringPercent.value * ringCount) / 100;
        if (i <= sp) {
            states.push(ring_none_img);
        } else {
            if (hf > 0.5 && i <= sp + 1) {
                states.push(ring_half_img);
            } else {
                states.push(ring_full_img);
            }
        }
    }
    return states;
});

const drawInner = async (level: number) => {
    if (level < 1 || level > 10) {
        return;
    }
    if (inner.value) {
        inner.value.style.backgroundImage = `url(${
            imageMap[level as keyof typeof imageMap]
        })`;
        resize_webview_add();
    }
};

// 计算环形点的位置
const getRingPosition = (index: number) => {
    const angle = (index * 2 * Math.PI) / ringCount - Math.PI / 2;
    const x = 50 + ((ringRadius * Math.cos(angle)) / width) * 100;
    const y = 50 + ((ringRadius * Math.sin(angle)) / height) * 100;
    return {
        position: "absolute" as const,
        left: `${x}%`,
        top: `${y}%`,
        transform: "translate(-50%, -50%)",
    };
};

const resize_webview_add = async () => {
    const webviewWindow = getCurrentWebviewWindow();
    const ratio = 1;
    let size = new PhysicalSize(
        width + ratio * (count.value % 2),
        height + ratio * (count.value % 2)
    );
    webviewWindow.setSize(size);
    count.value++;
    if (count.value > 100) {
        count.value = 0;
    }
};

const dragStart = async (event: MouseEvent) => {
    event.preventDefault();
    if (event.button !== 0) return; // 只处理左键

    const appWindow = getCurrentWebviewWindow();
    // const position = await appWindow.innerPosition();
    dragOffset = {
        x: event.clientX,
        y: event.clientY,
    };

    const onMouseMove = async (moveEvent: MouseEvent) => {
        moveEvent.preventDefault();
        const newX = moveEvent.screenX - dragOffset.x;
        const newY = moveEvent.screenY - dragOffset.y;
        await appWindow.setPosition(new PhysicalPosition(newX, newY));
    };

    const onMouseUp = () => {
        document.removeEventListener("mousemove", onMouseMove);
        document.removeEventListener("mouseup", onMouseUp);
    };
    document.addEventListener("mousemove", onMouseMove);
    document.addEventListener("mouseup", onMouseUp);
};

onMounted(() => {
    start();
    drawInner(centerLevel.value);
    // 初始化闪烁状态
    if (timeLeft.value === 0 && main.value) {
        main.value.classList.add('flashing');
    }
    // clearInterval(countDown);
});
</script>
<style scoped>
.rw-clock {
    width: 100%;
    height: 100%;
    background: transparent;
    transition: background-color 0.3s ease;
}

.rw-clock.flashing {
    animation: flash 0.5s infinite alternate;
}

@keyframes flash {
    from {
        background-color: rgba(255, 0, 0, 0.2);
    }
    to {
        background-color: rgba(255, 0, 0, 0.1);
    }
}

.rw-clock-outer {
    width: 100%;
    height: 100%;
    background-size: cover;
    position: relative;
}

.rw-clock-inner {
    width: 80%;
    height: 80%;
    background-size: cover;
    cursor: pointer;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}

.rw-clock-ring {
    position: absolute;
    width: 12px;
    height: 12px;
    pointer-events: none;
}

.ring-dot {
    width: 100%;
    height: 100%;
    display: block;
}
</style>
