<script setup>
import { ref, watch, computed } from 'vue';

const name = ref('');
const otr = ref(0);
const dp = ref(0);
watch(dp, (newVal) => {
})
const jangka_waktu = ref(0);
const pokok_hutang = computed(() => {
    if (String(dp.value).endsWith('%')) {
        let percentDp = parseInt(dp.value.replace('%', '')) / 100;
        return otr.value - (otr.value * percentDp);
    } else {
        return otr.value - dp.value;
    }
});
const bunga = ref(0);
watch(jangka_waktu, (newVal) => {
    const value = parseInt(newVal);

    if (value <= 12) {
        console.log('12');
        bunga.value = 12; // 12%
    } else if (value <= 24) {
        bunga.value = 14; // 14%
    } else if (value > 24) {
        bunga.value = 16.5; // 16.5%
    } else {
        bunga.value = 0;
    }
});
const monthly_angsuran = computed(() => {
    return (pokok_hutang.value + (pokok_hutang.value * bunga.value / 100)) / jangka_waktu.value ?? 0;
})

// Format number to Indonesian Rupiah
function formatRupiah(value) {
    if (!value) return "Rp 0";
    return new Intl.NumberFormat('id-ID', {
        style: 'currency',
        currency: 'IDR',
        minimumFractionDigits: 0,
        maximumFractionDigits: 0
    }).format(value);
}

// Parse string to number, removing non-numeric characters
function parseNumber(value) {
    if (typeof value === 'string') {
        return parseInt(value.replace(/[^\d%]/g, '')) || 0;
    }
    return value;
}

// Handle input for OTR field
function handleOtrInput(event) {
    const value = parseNumber(event.target.value);
    otr.value = value;
}

// Handle input for DP field (can be number or percentage)
function handleDpInput(event) {
    const value = event.target.value;
    if (value.includes('%')) {
        dp.value = value;
    } else {
        dp.value = parseNumber(value);
    }
}

// Handle input for jangka waktu field
function handleJangkaWaktuInput(event) {
    const value = parseNumber(event.target.value);
    jangka_waktu.value = value;
}

</script>

<template>
    <div class="w-full max-w-2xl p-4">
        <div class="grid grid-cols-2 gap-4">
            <div class="form-group flex flex-col gap-4">
                <div class="form-control">
                    <label for="name">Name</label>
                    <input type="text" id="name" class="input input-bordered w-full max-w-xs"
                        placeholder="Enter your name" v-model="name" />
                </div>
                <div class="form-control">
                    <label for="otr">OTR</label>
                    <div class="relative">
                        <input type="text" id="otr" class="input input-bordered w-full max-w-xs" v-model="otr" @input="handleOtrInput"  />
                    </div>
                </div>
                <div class="form-control">
                    <label for="dp">DP (Rupiah or %)</label>
                    <div class="relative">
                        <input type="text" id="dp" class="input input-bordered w-full max-w-xs" v-model="dp" @input="handleDpInput" />
                    </div>
                </div>
                <div class="form-control">
                    <label for="jangka_waktu">Jangka Waktu (bulan)</label>
                    <div class="flex items-center">
                        <input type="text" id="jangka_waktu" class="input input-bordered w-full max-w-xs"
                            placeholder="Enter your jangka waktu" v-model="jangka_waktu" 
                            @input="handleJangkaWaktuInput" />
                    </div>
                </div>

            </div>

            <div class="flex flex-col gap-4" >
                <p class="text-xl font-bold">Name : {{ name }}</p>
                <p class="text-xl font-bold">OTR : {{ formatRupiah(otr) }} </p>
                <p class="text-xl font-bold">DP : {{ dp.toString().endsWith('%') ? dp : formatRupiah(dp) }}</p>
                <p class="text-xl font-bold">Jangka Waktu : {{ jangka_waktu }} bulan</p>
                <p class="text-xl font-bold">Pokok Hutang : {{ formatRupiah(pokok_hutang) }}</p>
                <p class="text-xl font-bold">Bunga : {{ bunga ?? 0 }} %</p>
                <p class="text-xl font-bold">Bunga Rupiah: {{ formatRupiah(pokok_hutang * (bunga / 100)) }}</p>
                <p class="text-xl font-bold">Monthly Angsuran : {{ formatRupiah(monthly_angsuran) }}</p>
            </div>
        </div>
    </div>
</template>