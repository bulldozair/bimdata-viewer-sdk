<template>
    <div
        class="bulldozair-annotation"
        ref="marker"
        :class="{ grabbing }"
    >
        <div class="bulldozair-annotation-label">{{ index }}</div>
        <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            version="1.1"
            x="0px"
            y="0px"
            viewBox="0 0 512 512"
            style='fill:lightblue;'
            xml:space="preserve"
            @keyup.delete="remove"
        >
            <g>
                <g>
                    <path
                        style="stroke: lightblue;stroke-width: 15px;"
                        d="M256,0C153.755,0,70.573,83.182,70.573,185.426c0,126.888,165.939,313.167,173.004,321.035    c6.636,7.391,18.222,7.378,24.846,0c7.065-7.868,173.004-194.147,173.004-321.035C441.425,83.182,358.244,0,256,0z M256,278.719    c-51.442,0-93.292-41.851-93.292-93.293S204.559,92.134,256,92.134s93.291,41.851,93.291,93.293S307.441,278.719,256,278.719z"
                    />
                </g>
            </g>
        </svg>
    </div>
</template>
<script>
export default {
    props: {
        localContext: Object,
        index: Number,
        noteId: String,
        positionId: String,
        moveTo: Function,
        moveDone: Function,
        remove: Function,
    },
    data() {
        return {
            grabbing: false,
        };
    },
    mounted() {
        this.$refs.marker.addEventListener('mousedown', this.onMouseDown);
    },
    beforeUnmount() {
        this.$refs.marker.removeEventListener('mousedown', this.onMouseDown);
    },
    methods: {
        onAnnotationClick() {
            this.$viewer.globalContext.hub.emit('bz-annotation-click', { noteId: this.noteId, positionId: this.positionId });
        },
        onMouseDown() {
            this.grabbing = true;
            document.addEventListener('mouseup', this.onMouseUp);
            document.addEventListener('mousemove', this.onMouseMove);
        },
        onMouseUp() {
            this.grabbing = false;
            this.moveDone();
            document.removeEventListener('mousemove', this.onMouseMove);
        },
        onMouseMove(event) {
            let position;

            const windowName = this.localContext.window.name;

            if (windowName === '3d') {
                const { clientX, clientY } = event;

                const { xeokit } = this.localContext.viewer;
                const { x, y } = xeokit.scene.canvas.canvas.getBoundingClientRect();

                const pickResult = xeokit.scene.pick({
                    pickSurface: true,
                    canvasPos: [clientX - x, clientY - y],
                });

                const [p0, p1, p2] = pickResult ? pickResult.worldPos : [0, 0, 0];
                position = { x: p0, y: p2, z: p1 }; // xeokit is y-up
            } else {
                const { movementX, movementY } = event;

                const engine2d = this.localContext.viewer.viewer;
                const { x: cx, y: cy } = engine2d.canvas.getBoundingClientRect();
                const { x, y } = this.$refs.marker.getBoundingClientRect();

                position = engine2d.camera.getPosition({
                    x: x - cx + movementX,
                    y: y - cy + movementY,
                });
            }

            this.moveTo(position);
        },
    },
}
</script>