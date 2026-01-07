<script setup lang="ts">
import { ref, watch, onMounted } from 'vue'

const props = defineProps<{
    userImage: string | null
    scale: number
    offsetX: number
    offsetY: number
    bgImageUrl: string
    overlayImageUrl: string
}>()

const canvasRef = ref<HTMLCanvasElement | null>(null)

const drawCanvas = () => {
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

    const overlayImage = new Image()
    overlayImage.crossOrigin = 'anonymous'
    overlayImage.src = props.overlayImageUrl

    let imagesLoaded = 0
    const totalImages = 2

    const onImageLoad = () => {
        imagesLoaded++
        if (imagesLoaded < totalImages) return

        // Draw circular mask with user image
        // Profile circle is at (750, 720) with radius 485 to fit inside the gold ring
        const maskRadius = 485
        const centerX = canvas.width / 2
        const centerY = 720

        const userImg = new Image()
        userImg.crossOrigin = 'anonymous'
        userImg.src = props.userImage!

        userImg.onload = () => {
            // Create an offscreen canvas for the user image with effects
            const offscreenCanvas = document.createElement('canvas')
            offscreenCanvas.width = canvas.width
            offscreenCanvas.height = canvas.height
            const offCtx = offscreenCanvas.getContext('2d')
            if (!offCtx) return

            // Draw circular clipping region on offscreen canvas
            offCtx.save()
            offCtx.beginPath()
            offCtx.arc(centerX, centerY, maskRadius, 0, Math.PI * 2)
            offCtx.clip()

            // Calculate image dimensions to fit in circle
            const imgWidth = maskRadius * 2 * props.scale
            const imgHeight = (userImg.height / userImg.width) * imgWidth

            // Draw user image with offset
            const imgX = centerX - imgWidth / 2 + props.offsetX
            const imgY = centerY - imgHeight / 2 + props.offsetY

            offCtx.drawImage(userImg, imgX, imgY, imgWidth, imgHeight)

            // Create gradient mask for soft edges
            const maskCanvas = document.createElement('canvas')
            maskCanvas.width = canvas.width
            maskCanvas.height = canvas.height
            const maskCtx = maskCanvas.getContext('2d')
            if (!maskCtx) return

            // Draw solid circle as mask
            maskCtx.beginPath()
            maskCtx.arc(centerX, centerY, maskRadius, 0, Math.PI * 2)
            maskCtx.fillStyle = 'white'
            maskCtx.fill()

            // Apply mask to offscreen canvas
            offCtx.globalCompositeOperation = 'destination-in'
            offCtx.drawImage(maskCanvas, 0, 0)
            offCtx.restore()

            // Now draw everything to the main canvas
            // 1. Draw background
            ctx.drawImage(bgImage, 0, 0, canvas.width, canvas.height)

            // 2. Draw the masked user image
            ctx.drawImage(offscreenCanvas, 0, 0)

            // 3. Draw overlay on top of everything
            ctx.drawImage(overlayImage, 0, 0, canvas.width, canvas.height)
        }
    }

    bgImage.onload = onImageLoad
    overlayImage.onload = onImageLoad
}

watch(
    () => [
        props.userImage,
        props.scale,
        props.offsetX,
        props.offsetY,
        props.bgImageUrl,
        props.overlayImageUrl,
    ],
    () => {
        drawCanvas()
    },
    { immediate: true },
)

onMounted(() => {
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
