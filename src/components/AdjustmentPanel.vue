<script setup lang="ts">
import { ZoomIn, ArrowLeftRight, ArrowUpDown } from 'lucide-vue-next'

defineProps<{
    scale: number
    offsetX: number
    offsetY: number
}>()

const emit = defineEmits<{
    'update:scale': [value: number]
    'update:offsetX': [value: number]
    'update:offsetY': [value: number]
}>()
</script>

<template>
    <div class="space-y-5">
        <!-- Scale Control -->
        <div class="group">
            <label class="flex items-center justify-between text-sm font-medium text-white/80 mb-3">
                <span class="flex items-center gap-2">
                    <ZoomIn class="w-4 h-4 text-emerald-400" />
                    Zoom Level
                </span>
                <span class="px-2 py-0.5 bg-white/10 rounded-md text-emerald-300 font-mono text-xs">
                    {{ (scale * 100).toFixed(0) }}%
                </span>
            </label>
            <input
                type="range"
                :value="scale"
                @input="emit('update:scale', parseFloat(($event.target as HTMLInputElement).value))"
                min="0.5"
                max="3"
                step="0.05"
                class="w-full h-2 bg-white/10 rounded-lg appearance-none cursor-pointer accent-emerald-500 hover:accent-emerald-400 transition-all"
                aria-label="Adjust photo scale"
            />
        </div>

        <!-- Horizontal Offset Control -->
        <div class="group">
            <label class="flex items-center justify-between text-sm font-medium text-white/80 mb-3">
                <span class="flex items-center gap-2">
                    <ArrowLeftRight class="w-4 h-4 text-cyan-400" />
                    Horizontal Position
                </span>
                <span class="px-2 py-0.5 bg-white/10 rounded-md text-cyan-300 font-mono text-xs">
                    {{ offsetX > 0 ? '+' : '' }}{{ offsetX }}px
                </span>
            </label>
            <input
                type="range"
                :value="offsetX"
                @input="
                    emit('update:offsetX', parseFloat(($event.target as HTMLInputElement).value))
                "
                min="-500"
                max="500"
                step="5"
                class="w-full h-2 bg-white/10 rounded-lg appearance-none cursor-pointer accent-cyan-500 hover:accent-cyan-400 transition-all"
                aria-label="Adjust horizontal position"
            />
        </div>

        <!-- Vertical Offset Control -->
        <div class="group">
            <label class="flex items-center justify-between text-sm font-medium text-white/80 mb-3">
                <span class="flex items-center gap-2">
                    <ArrowUpDown class="w-4 h-4 text-violet-400" />
                    Vertical Position
                </span>
                <span class="px-2 py-0.5 bg-white/10 rounded-md text-violet-300 font-mono text-xs">
                    {{ offsetY > 0 ? '+' : '' }}{{ offsetY }}px
                </span>
            </label>
            <input
                type="range"
                :value="offsetY"
                @input="
                    emit('update:offsetY', parseFloat(($event.target as HTMLInputElement).value))
                "
                min="-500"
                max="500"
                step="5"
                class="w-full h-2 bg-white/10 rounded-lg appearance-none cursor-pointer accent-violet-500 hover:accent-violet-400 transition-all"
                aria-label="Adjust vertical position"
            />
        </div>
    </div>
</template>
