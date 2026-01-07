<script setup lang="ts">
import { ref } from 'vue'
import { ImageIcon, Upload, CheckCircle2, Download, Sparkles, Camera, Move } from 'lucide-vue-next'
import CanvasPreview from './CanvasPreview.vue'
import AdjustmentPanel from './AdjustmentPanel.vue'

const BG_IMAGE_URL = '/images/bg.svg'
const OVERLAY_IMAGE_URL = '/images/overlay.svg'

const PREDEFINED_PHOTOS = [
    { id: 'avatar1', src: '/images/avatars/avatar1.jpg', label: 'Avatar 1' },
    { id: 'avatar2', src: '/images/avatars/avatar2.jpg', label: 'Avatar 2' },
    { id: 'avatar3', src: '/images/avatars/avatar3.jpg', label: 'Avatar 3' },
    { id: 'avatar4', src: '/images/avatars/avatar4.jpg', label: 'Avatar 4' },
    { id: 'avatar5', src: '/images/avatars/avatar5.jpg', label: 'Avatar 5' },
    { id: 'avatar6', src: '/images/avatars/avatar6.png', label: 'Avatar 6' },
]

const userImage = ref<string | null>(null)
const selectedPreset = ref<string | null>(null)
const scale = ref(1)
const offsetX = ref(0)
const offsetY = ref(0)
const fileInputRef = ref<HTMLInputElement | null>(null)
const canvasPreviewRef = ref<InstanceType<typeof CanvasPreview> | null>(null)

const handleImageUpload = (e: Event) => {
    const target = e.target as HTMLInputElement
    const file = target.files?.[0]
    if (file) {
        const reader = new FileReader()
        reader.onload = (event) => {
            userImage.value = event.target?.result as string
            selectedPreset.value = null
            resetTransforms()
        }
        reader.readAsDataURL(file)
    }
}

const handlePresetSelect = (preset: (typeof PREDEFINED_PHOTOS)[0]) => {
    userImage.value = preset.src
    selectedPreset.value = preset.id
    resetTransforms()
}

const resetTransforms = () => {
    scale.value = 1
    offsetX.value = 0
    offsetY.value = 0
}

const handleDownload = () => {
    if (!userImage.value || !canvasPreviewRef.value?.canvasRef) return

    const canvas = canvasPreviewRef.value.canvasRef
    const link = document.createElement('a')
    link.href = canvas.toDataURL('image/png')
    link.download = 'mbstu-islamic-seminar-profile.png'
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)
}

const triggerFileInput = () => {
    fileInputRef.value?.click()
}
</script>

<template>
    <div
        class="min-h-screen bg-linear-to-br from-teal-900 via-emerald-900 to-cyan-900 relative overflow-hidden"
    >
        <!-- Animated background pattern -->
        <div class="absolute inset-0 opacity-10">
            <div
                class="absolute inset-0 bg-[url('data:image/svg+xml;base64,PHN2ZyB3aWR0aD0iNjAiIGhlaWdodD0iNjAiIHZpZXdCb3g9IjAgMCA2MCA2MCIgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIj48ZyBmaWxsPSJub25lIiBmaWxsLXJ1bGU9ImV2ZW5vZGQiPjxnIGZpbGw9IiNmZmZmZmYiIGZpbGwtb3BhY2l0eT0iMC40Ij48cGF0aCBkPSJNMzYgMzRoLTJ2LTRoLTR2LTJoNHYtNGgydjRoNHYyaC00djR6bTAtMzBoLTJ2NGgtNHYyaDR2NGgydi00aDR2LTJoLTR2LTR6TTYgMzRINHYtNEgwdi0yaDR2LTRoMnY0aDR2Mkg2djR6bTAtMzBINHY0SDB2Mmg0djRoMlY2aDRWNEg2VjB6Ii8+PC9nPjwvZz48L3N2Zz4=')] animate-pulse"
            ></div>
        </div>

        <!-- Decorative blobs -->
        <div
            class="absolute top-0 left-0 w-96 h-96 bg-emerald-500/20 rounded-full blur-3xl -translate-x-1/2 -translate-y-1/2"
        ></div>
        <div
            class="absolute bottom-0 right-0 w-96 h-96 bg-cyan-500/20 rounded-full blur-3xl translate-x-1/2 translate-y-1/2"
        ></div>
        <div
            class="absolute top-1/2 left-1/2 w-64 h-64 bg-teal-400/10 rounded-full blur-2xl -translate-x-1/2 -translate-y-1/2"
        ></div>

        <div class="container mx-auto py-8 px-4 max-w-6xl relative z-10">
            <!-- Header -->
            <header class="mb-8 md:mb-12 text-center">
                <div
                    class="inline-flex items-center gap-2 bg-white/10 backdrop-blur-sm px-3 py-1.5 md:px-4 md:py-2 rounded-full text-emerald-300 text-xs md:text-sm font-medium mb-3 md:mb-4"
                >
                    <Sparkles class="w-3 h-3 md:w-4 md:h-4" />
                    <span>10 January 2026 • Islamic Seminar & Book Fair</span>
                </div>
                <h1
                    class="text-3xl sm:text-4xl md:text-5xl font-bold text-white mb-3 md:mb-4 tracking-tight"
                >
                    Profile Photo
                    <span
                        class="bg-linear-to-r from-emerald-400 to-cyan-400 bg-clip-text text-transparent"
                    >
                        Generator
                    </span>
                </h1>
                <p class="text-base md:text-lg text-emerald-100/70 max-w-md mx-auto px-4">
                    Create your personalized MBSTU Islamic Seminar profile picture in seconds
                </p>
            </header>

            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <!-- Left Column - Photo Selection -->
                <div class="space-y-6">
                    <!-- Upload Photo Section -->
                    <section
                        class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 p-6 shadow-2xl"
                    >
                        <h2 class="text-xl font-semibold text-white mb-5 flex items-center gap-3">
                            <div class="p-2 bg-emerald-500/20 rounded-xl">
                                <Camera class="w-5 h-5 text-emerald-400" />
                            </div>
                            Choose Your Photo
                        </h2>

                        <!-- Predefined Avatars Grid -->
                        <div class="grid grid-cols-3 gap-3 mb-5">
                            <button
                                v-for="preset in PREDEFINED_PHOTOS"
                                :key="preset.id"
                                @click="handlePresetSelect(preset)"
                                :class="[
                                    'group relative aspect-square rounded-2xl overflow-hidden border-2 transition-all duration-300 hover:scale-[1.02] hover:shadow-lg hover:shadow-emerald-500/20',
                                    selectedPreset === preset.id
                                        ? 'border-emerald-400 ring-4 ring-emerald-400/30'
                                        : 'border-white/20 hover:border-white/40',
                                ]"
                            >
                                <img
                                    :src="preset.src"
                                    :alt="preset.label"
                                    class="w-full h-full object-cover transition-transform duration-300 group-hover:scale-110"
                                />
                                <div
                                    v-if="selectedPreset === preset.id"
                                    class="absolute inset-0 bg-linear-to-t from-emerald-600/80 to-emerald-500/40 flex items-center justify-center"
                                >
                                    <CheckCircle2 class="w-8 h-8 text-white drop-shadow-lg" />
                                </div>
                                <div
                                    v-else
                                    class="absolute inset-0 bg-linear-to-t from-black/50 to-transparent opacity-0 group-hover:opacity-100 transition-opacity"
                                ></div>
                            </button>
                        </div>

                        <!-- Divider -->
                        <div class="relative my-6 flex items-center gap-4">
                            <div class="flex-1 h-px bg-white/20"></div>
                            <span class="text-sm text-emerald-200/70 font-medium whitespace-nowrap">
                                or upload your own
                            </span>
                            <div class="flex-1 h-px bg-white/20"></div>
                        </div>

                        <button
                            @click="triggerFileInput"
                            :class="[
                                'group w-full border-2 border-dashed rounded-2xl p-6 md:p-8 text-center transition-all duration-300',
                                userImage && !selectedPreset
                                    ? 'border-emerald-400 bg-emerald-500/20'
                                    : 'border-white/30 hover:border-emerald-400/60 hover:bg-white/5',
                            ]"
                        >
                            <div
                                :class="[
                                    'w-16 h-16 mx-auto mb-4 rounded-2xl flex items-center justify-center transition-all duration-300',
                                    userImage && !selectedPreset
                                        ? 'bg-emerald-500/30'
                                        : 'bg-white/10 group-hover:bg-emerald-500/20',
                                ]"
                            >
                                <Upload
                                    :class="[
                                        'w-8 h-8 transition-colors',
                                        userImage && !selectedPreset
                                            ? 'text-emerald-300'
                                            : 'text-white/60 group-hover:text-emerald-400',
                                    ]"
                                />
                            </div>
                            <p
                                :class="[
                                    'font-semibold text-lg mb-1',
                                    userImage && !selectedPreset
                                        ? 'text-emerald-300'
                                        : 'text-white',
                                ]"
                            >
                                {{
                                    userImage && !selectedPreset
                                        ? 'Photo uploaded!'
                                        : 'Drop image here or click to upload'
                                }}
                            </p>
                            <p class="text-sm text-white/50">Supports PNG, JPG up to 10MB</p>
                        </button>

                        <input
                            ref="fileInputRef"
                            type="file"
                            accept="image/*"
                            @change="handleImageUpload"
                            class="hidden"
                            aria-label="Upload profile photo"
                        />
                    </section>
                </div>

                <!-- Right Column - Preview & Download -->
                <div class="space-y-6">
                    <section
                        class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 p-6 shadow-2xl"
                    >
                        <h2 class="text-xl font-semibold text-white mb-5 flex items-center gap-3">
                            <div class="p-2 bg-cyan-500/20 rounded-xl">
                                <ImageIcon class="w-5 h-5 text-cyan-400" />
                            </div>
                            Live Preview
                        </h2>

                        <template v-if="userImage">
                            <div class="rounded-2xl overflow-hidden ring-1 ring-white/20">
                                <CanvasPreview
                                    ref="canvasPreviewRef"
                                    :user-image="userImage"
                                    :scale="scale"
                                    :offset-x="offsetX"
                                    :offset-y="offsetY"
                                    :bg-image-url="BG_IMAGE_URL"
                                    :overlay-image-url="OVERLAY_IMAGE_URL"
                                />
                            </div>
                        </template>
                        <template v-else>
                            <div
                                class="aspect-square rounded-2xl bg-white/5 border border-white/10 flex items-center justify-center"
                            >
                                <div class="text-center">
                                    <div
                                        class="w-20 h-20 mx-auto mb-4 rounded-full bg-white/10 flex items-center justify-center"
                                    >
                                        <ImageIcon class="w-10 h-10 text-white/30" />
                                    </div>
                                    <p class="text-white/40 font-medium">
                                        Select or upload a photo
                                    </p>
                                    <p class="text-white/25 text-sm mt-1">
                                        Your preview will appear here
                                    </p>
                                </div>
                            </div>
                        </template>
                    </section>

                    <!-- Controls Section -->
                    <template v-if="userImage">
                        <section
                            class="bg-white/10 backdrop-blur-md rounded-3xl border border-white/20 p-6 shadow-2xl"
                        >
                            <h2
                                class="text-xl font-semibold text-white mb-5 flex items-center gap-3"
                            >
                                <div class="p-2 bg-violet-500/20 rounded-xl">
                                    <Move class="w-5 h-5 text-violet-400" />
                                </div>
                                Adjust Your Photo
                            </h2>
                            <AdjustmentPanel
                                :scale="scale"
                                :offset-x="offsetX"
                                :offset-y="offsetY"
                                @update:scale="scale = $event"
                                @update:offset-x="offsetX = $event"
                                @update:offset-y="offsetY = $event"
                            />
                        </section>

                        <!-- Download Button -->
                        <button
                            @click="handleDownload"
                            class="group w-full bg-linear-to-r from-emerald-500 to-cyan-500 hover:from-emerald-400 hover:to-cyan-400 text-white text-base md:text-lg py-4 md:py-5 rounded-2xl shadow-xl shadow-emerald-500/25 font-semibold flex items-center justify-center gap-2 md:gap-3 transition-all duration-300 hover:shadow-emerald-500/40 hover:scale-[1.02]"
                        >
                            <Download
                                class="w-5 h-5 md:w-6 md:h-6 transition-transform group-hover:-translate-y-0.5"
                            />
                            <span class="hidden sm:inline">Download Your Profile Photo</span>
                            <span class="sm:hidden">Download Photo</span>
                        </button>
                    </template>
                </div>
            </div>

            <!-- Footer -->
            <footer class="mt-10 md:mt-16 text-center space-y-3 md:space-y-4 pb-6">
                <div
                    class="inline-flex items-center gap-2 text-white/40 text-xs md:text-sm flex-wrap justify-center"
                >
                    <span>Made with</span>
                    <span class="text-red-400">❤️</span>
                    <span>for</span>
                    <span class="text-emerald-400 font-medium">MBSTU Islamic Seminar</span>
                </div>
                <div class="flex items-center justify-center gap-4 md:gap-6 text-xs md:text-sm">
                    <a
                        href="https://github.com/frost23z/mbstu-islamic-seminar-profile-pic-generator"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="inline-flex items-center gap-2 text-white/50 hover:text-white transition-colors"
                    >
                        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                            <path
                                d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"
                            />
                        </svg>
                        GitHub
                    </a>
                    <span class="text-white/20">•</span>
                    <a
                        href="https://github.com/frost23z/mbstu-islamic-seminar-profile-pic-generator/issues"
                        target="_blank"
                        rel="noopener noreferrer"
                        class="inline-flex items-center gap-2 text-white/50 hover:text-emerald-400 transition-colors"
                    >
                        <svg
                            class="w-4 h-4"
                            fill="none"
                            stroke="currentColor"
                            viewBox="0 0 24 24"
                            stroke-width="2"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197m13.5-9a2.5 2.5 0 11-5 0 2.5 2.5 0 015 0z"
                            />
                        </svg>
                        Contribute
                    </a>
                </div>
            </footer>
        </div>
    </div>
</template>
