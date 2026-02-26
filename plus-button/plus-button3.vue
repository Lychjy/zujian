<template>
    <div class="cart-wrapper">

        <!-- 加入购物车：只在 1 → 0 时使用 transition -->
        <component :is="animateAdd ? 'transition' : 'div'" name="zoom-in-left">
            <button v-if="showAdd" class="add-btn" @click="onAdd">
                加入购物车
            </button>
        </component>

        <!-- 加减控制区 -->
        <div v-if="showControl" class="cart-control">
            <div ref="realMinus" class="btn minus" :class="{ hide: hideRealMinus }" @click="onMinus">−</div>

            <span class="count" :class="{ hide: hideCount }">{{ count }}</span>

            <div ref="plus" class="btn plus" @click="onPlus">+</div>
        </div>

        <!-- 影子减号（只负责滚动） -->
        <div v-if="showGhost" ref="ghostMinus" class="btn minus ghost">−</div>

    </div>
</template>

<script setup>
import { ref, nextTick } from 'vue'

/* ================= 状态 ================= */

const count = ref(0)

// 展示控制
const showAdd = ref(true)
const showControl = ref(false)

// 是否给「加入购物车」加 enter 动画
const animateAdd = ref(false)

// 加减内部显隐
const hideRealMinus = ref(false)
const hideCount = ref(false)

// 动画影子
const showGhost = ref(false)

/* ================= DOM ================= */

const realMinus = ref(null)
const plus = ref(null)
const ghostMinus = ref(null)

/* ================= 工具 ================= */

function centerDistance(from, to) {
    const f = from.getBoundingClientRect()
    const t = to.getBoundingClientRect()
    return (t.left + t.width / 2) - (f.left + f.width / 2)
}

/* ================= 事件 ================= */

// 加号
function onPlus() {
    count.value++
}

// 减号
async function onMinus() {
    if (count.value > 1) {
        count.value--
        return
    }

    // ===== 1 → 0 =====
    const distance = centerDistance(realMinus.value, plus.value)

    showGhost.value = true
    await nextTick()

    const ghost = ghostMinus.value
    const m = realMinus.value.getBoundingClientRect()

    ghost.style.left = `${m.left}px`
    ghost.style.top = `${m.top}px`
    ghost.style.transform = 'translateX(0)'
    ghost.style.opacity = '1'

    hideRealMinus.value = true
    hideCount.value = true

    requestAnimationFrame(() => {
        ghost.style.transition = 'transform 700ms ease, opacity 700ms'
        ghost.style.transform = `translateX(${distance}px) rotate(720deg)`
        ghost.style.opacity = '0'
    })

    setTimeout(() => {
        showGhost.value = false
        showControl.value = false
        count.value = 0

        // 👇 只在 1 → 0 时启用 zoom-in-left
        animateAdd.value = true
        showAdd.value = true

        hideRealMinus.value = false
        hideCount.value = false
    }, 500)
}

// 点击加入购物车
async function onAdd() {
    // ===== 0 → 1 =====
    // ❗不允许 zoom 影响 0 → 1
    animateAdd.value = false

    showAdd.value = false
    showControl.value = true
    count.value = 1

    await nextTick()

    const distance = centerDistance(realMinus.value, plus.value)

    showGhost.value = true
    await nextTick()

    const ghost = ghostMinus.value
    const p = plus.value.getBoundingClientRect()

    ghost.style.left = `${p.left}px`
    ghost.style.top = `${p.top}px`
    ghost.style.transform = 'translateX(0)'
    ghost.style.opacity = '1'

    hideRealMinus.value = true
    hideCount.value = true

    requestAnimationFrame(() => {
        ghost.style.transition = 'transform 700ms ease'
        ghost.style.transform = `translateX(${-distance}px) rotate(-720deg)`
    })

    setTimeout(() => {
        showGhost.value = false
        hideRealMinus.value = false
        hideCount.value = false
    }, 700)
}
</script>

<style scoped>
/* ================= 容器 ================= */

.cart-wrapper {
    display: inline-flex;
    align-items: center;
    height: 32px;
}

/* ================= 加入购物车 ================= */

.add-btn {
    height: 32px;
    padding: 0 16px;
    border-radius: 16px;
    border: none;
    background: #1890ff;
    color: #fff;
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    white-space: nowrap;
}

/* el-zoom-in-left（只用于 enter） */
.zoom-in-left-enter-active {
    transition:
        transform 600ms cubic-bezier(.55, 0, .1, 1),
        opacity 600ms cubic-bezier(.55, 0, .1, 1);
    transform-origin: left center;
}

.zoom-in-left-enter-from {
    opacity: 0;
    transform: scaleX(0) translateX(-100%);
}

.zoom-in-left-enter-to {
    opacity: 1;
    transform: scaleX(1) translateX(0);
}

/* ================= 加减区 ================= */

.cart-control {
    display: inline-flex;
    align-items: center;
}

.btn {
    width: 28px;
    height: 28px;
    border-radius: 50%;
    font-size: 18px;
    font-weight: bold;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;
}

.minus {
    background: #f0f0f0;
    color: #333;
}

.plus {
    background: #1890ff;
    color: #fff;
}

.count {
    min-width: 28px;
    margin: 0 6px;
    text-align: center;
    font-size: 15px;
    font-weight: 600;
    color: #1890ff;
    transition: opacity 300ms ease;
}

.hide {
    opacity: 0;
}

/* ================= 影子减号 ================= */

.ghost {
    position: fixed;
    z-index: 10;
    pointer-events: none;
}
</style>