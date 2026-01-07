<script setup lang="ts">
import { ref, watch, nextTick, onMounted } from 'vue'

const props = defineProps<{
    userImage: string | null
    scale: number
    offsetX: number
    offsetY: number
    bgImageUrl: string
    titleColor?: string
    dateColor?: string
}>()

const canvasRef = ref<HTMLCanvasElement | null>(null)
let fontsLoaded = false
let isMounted = false

// Load fonts
const loadFonts = async () => {
    if (fontsLoaded) return
    try {
        const cinzel = new FontFace(
            'Cinzel',
            'url(https://fonts.gstatic.com/s/cinzel/v23/8vIU7ww63mVu7gtR-kwKxNvkNOjw-tbnfY3lC45r.woff2)',
        )
        const inter = new FontFace(
            'Inter',
            'url(https://fonts.gstatic.com/s/inter/v13/UcCO3FwrK3iLTeHuS_fvQtMwCp50KnMw2boKoduKmMEVuLyfAZ9hiJ-Ek-_EeA.woff2)',
        )
        const [loadedCinzel, loadedInter] = await Promise.all([cinzel.load(), inter.load()])
        document.fonts.add(loadedCinzel)
        document.fonts.add(loadedInter)
        fontsLoaded = true
    } catch (e) {
        console.warn('Font loading failed, using fallback fonts:', e)
        fontsLoaded = true // Continue anyway with fallback fonts
    }
}

const drawCanvas = async () => {
    // Wait for next tick to ensure canvas ref is available
    await nextTick()

    const canvas = canvasRef.value
    if (!canvas || !props.userImage) return

    const ctx = canvas.getContext('2d')
    if (!ctx) return

    // Set canvas size
    canvas.width = 1500
    canvas.height = 1500

    const bgImage = new Image()
    bgImage.crossOrigin = 'anonymous'
    bgImage.src = props.bgImageUrl

    bgImage.onerror = () => {
        console.error('Failed to load background image:', props.bgImageUrl)
        // Draw without background
        loadUserImage(ctx, canvas)
    }

    bgImage.onload = () => {
        // 1. Draw background first
        ctx.drawImage(bgImage, 0, 0, canvas.width, canvas.height)
        loadUserImage(ctx, canvas)
    }
}

const loadUserImage = (ctx: CanvasRenderingContext2D, canvas: HTMLCanvasElement) => {
    const userImg = new Image()
    userImg.crossOrigin = 'anonymous'
    userImg.src = props.userImage!

    userImg.onerror = () => {
        console.error('Failed to load user image:', props.userImage)
    }

    userImg.onload = async () => {
        // Load fonts
        await loadFonts()

        // 2. Draw user image with soft fading edge
        const centerX = canvas.width / 2
        const centerY = canvas.height / 2 // True center
        const radius = 550 // Profile photo radius
        const fadeWidth = 350 // Width of the fade zone

        // Calculate image dimensions to cover circle while maintaining aspect ratio
        const imgAspect = userImg.width / userImg.height
        let drawWidth, drawHeight

        if (imgAspect > 1) {
            // Image is wider - fit to height
            drawHeight = radius * 2 * props.scale
            drawWidth = drawHeight * imgAspect
        } else {
            // Image is taller - fit to width
            drawWidth = radius * 2 * props.scale
            drawHeight = drawWidth / imgAspect
        }

        // Center the image within the circle with offset
        const drawX = centerX - drawWidth / 2 + props.offsetX
        const drawY = centerY - drawHeight / 2 + props.offsetY

        // Create an offscreen canvas to draw the user image with soft edges
        const offCanvas = document.createElement('canvas')
        offCanvas.width = canvas.width
        offCanvas.height = canvas.height
        const offCtx = offCanvas.getContext('2d')!

        // Draw user image on offscreen canvas (clipped to larger area)
        offCtx.save()
        offCtx.beginPath()
        offCtx.arc(centerX, centerY, radius + 20, 0, Math.PI * 2)
        offCtx.closePath()
        offCtx.clip()
        offCtx.drawImage(userImg, drawX, drawY, drawWidth, drawHeight)
        offCtx.restore()

        // Apply radial gradient mask to fade edges - smoother curve
        offCtx.globalCompositeOperation = 'destination-in'
        const maskGradient = offCtx.createRadialGradient(
            centerX,
            centerY,
            radius - fadeWidth,
            centerX,
            centerY,
            radius + 30,
        )
        // More gradient stops for smoother transition
        maskGradient.addColorStop(0, 'rgba(0, 0, 0, 1)')
        maskGradient.addColorStop(0.2, 'rgba(0, 0, 0, 1)')
        maskGradient.addColorStop(0.35, 'rgba(0, 0, 0, 0.95)')
        maskGradient.addColorStop(0.5, 'rgba(0, 0, 0, 0.85)')
        maskGradient.addColorStop(0.6, 'rgba(0, 0, 0, 0.7)')
        maskGradient.addColorStop(0.7, 'rgba(0, 0, 0, 0.5)')
        maskGradient.addColorStop(0.8, 'rgba(0, 0, 0, 0.3)')
        maskGradient.addColorStop(0.88, 'rgba(0, 0, 0, 0.15)')
        maskGradient.addColorStop(0.94, 'rgba(0, 0, 0, 0.05)')
        maskGradient.addColorStop(1, 'rgba(0, 0, 0, 0)')

        offCtx.beginPath()
        offCtx.arc(centerX, centerY, radius + 30, 0, Math.PI * 2)
        offCtx.fillStyle = maskGradient
        offCtx.fill()

        // Draw the masked image onto main canvas
        ctx.drawImage(offCanvas, 0, 0)

        // 3. Draw text overlay programmatically
        const titleColor = props.titleColor || '#d4a853'
        const dateColor = props.dateColor || '#ffffff'

        // Text shadow
        ctx.shadowColor = 'rgba(0, 0, 0, 0.5)'
        ctx.shadowBlur = 6
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 2

        // Main title line 1
        ctx.font = '600 100px Cinzel, serif'
        ctx.fillStyle = titleColor
        ctx.textAlign = 'center'
        ctx.textBaseline = 'middle'
        ctx.fillText('MBSTU Islamic Seminar', canvas.width / 2, 1160)

        // Main title line 2
        ctx.font = '500 68px Cinzel, serif'
        ctx.fillText('& Book Fair 2026', canvas.width / 2, 1270)

        // Date
        ctx.font = '500 70px Inter, sans-serif'
        ctx.fillStyle = dateColor
        ctx.fillText('10 January', canvas.width / 2, 1380)

        // Reset shadow
        ctx.shadowColor = 'transparent'
        ctx.shadowBlur = 0
        ctx.shadowOffsetX = 0
        ctx.shadowOffsetY = 0
    }
}

watch(
    () => [
        props.userImage,
        props.scale,
        props.offsetX,
        props.offsetY,
        props.bgImageUrl,
        props.titleColor,
        props.dateColor,
    ],
    () => {
        if (isMounted) {
            drawCanvas()
        }
    },
)

onMounted(() => {
    isMounted = true
    drawCanvas()
})

defineExpose({
    canvasRef,
})
</script>

<template>
    <div class="relative group">
        <canvas ref="canvasRef" class="w-full rounded-xl" aria-label="Profile photo preview" />
    </div>
</template>
