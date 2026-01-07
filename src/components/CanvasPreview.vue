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

        const userImg = new Image()
        userImg.crossOrigin = 'anonymous'
        userImg.src = props.userImage!

        userImg.onload = () => {
            // 1. Draw background first
            ctx.drawImage(bgImage, 0, 0, canvas.width, canvas.height)

            // 2. Draw user image inside circular mask
            const centerX = canvas.width / 2
            const centerY = 680 // Slightly above center
            const radius = 480 // Circle radius for profile photo

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

            // Create circular clipping path
            ctx.save()
            ctx.beginPath()
            ctx.arc(centerX, centerY, radius, 0, Math.PI * 2)
            ctx.closePath()
            ctx.clip()

            // Draw user image inside the clipped circle
            ctx.drawImage(userImg, drawX, drawY, drawWidth, drawHeight)
            ctx.restore()

            // 2.5. Add blur/feather effect at the edge of the circle
            // Create a radial gradient that fades from transparent to background color
            const blurWidth = 120 // Width of the blur effect
            const gradient = ctx.createRadialGradient(
                centerX,
                centerY,
                radius - blurWidth,
                centerX,
                centerY,
                radius + 20,
            )
            gradient.addColorStop(0, 'rgba(10, 40, 40, 0)')
            gradient.addColorStop(0.4, 'rgba(10, 40, 40, 0.3)')
            gradient.addColorStop(0.7, 'rgba(10, 40, 40, 0.6)')
            gradient.addColorStop(0.9, 'rgba(10, 40, 40, 0.85)')
            gradient.addColorStop(1, 'rgba(10, 40, 40, 1)')

            ctx.beginPath()
            ctx.arc(centerX, centerY, radius + 20, 0, Math.PI * 2)
            ctx.fillStyle = gradient
            ctx.fill()

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
