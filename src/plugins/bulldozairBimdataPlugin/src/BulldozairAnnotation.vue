<template>
    <div
        class="bulldozair-annotation"
        @mousedown="onMouseDown"
        @keyup.delete="remove"
        ref="marker"
        :class="{ grabbing }"
    >
        <div
            class="bulldozair-annotation-label"
            v-if="index !== undefined"
        >{{ index }}</div>
        <svg
            xmlns="http://www.w3.org/2000/svg"
            height="35px"
            viewBox="0 0 24 24"
            width="35px"
            :style="{ fill: pinColor }"
        >
            <path
                d="M0 0h24v24H0z"
                fill="none"
            />
            <path
                d="M12 2C8.13 2 5 5.13 5 9c0 5.25 7 13 7 13s7-7.75 7-13c0-3.87-3.13-7-7-7zm0 9.5c-1.38 0-2.5-1.12-2.5-2.5s1.12-2.5 2.5-2.5 2.5 1.12 2.5 2.5-1.12 2.5-2.5 2.5z"
            />
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
        pinColor: String,
    },
    data() {
        return {
            grabbing: false,
            moved: false,
            initialPosition: null,
        };
    },
    mounted() {
        if (this.$refs.marker) {
            this.$refs.marker.addEventListener('mousedown', this.onMouseDown);
        }
    },
    beforeUnmount() {
        if (this.$refs.marker) {
            this.$refs.marker.removeEventListener('mousedown', this.onMouseDown);
        }
    },
    methods: {
        onAnnotationClick() {
            if (!this.moved) {
                this.$viewer.globalContext.hub.emit('bz-annotation-click', { noteId: this.noteId, positionId: this.positionId });
                console.log(this.$refs.marker.getBoundingClientRect())
            }
        },
        onMouseDown(event) {
            this.grabbing = true;
            this.moved = false;
            this.initialPosition = { x: event.clientX, y: event.clientY };
            document.addEventListener('mouseup', this.onMouseUp);
            document.addEventListener('mousemove', this.onMouseMove);
        },
        onMouseUp(event) {
            this.grabbing = false;
            document.removeEventListener('mousemove', this.onMouseMove);
            document.removeEventListener('mouseup', this.onMouseUp);

            if (!this.moved) {
                this.onAnnotationClick();
            } else {
                this.moved = false;
                this.moveDone(this.getCurrentPosition(event));
            }
        },
        onMouseMove(event) {
            const windowName = this.localContext.window.name;
            if(windowName === '2d') {
                return;
            } 
            const { clientX, clientY } = event;
            const deltaX = clientX - this.initialPosition.x;
            const deltaY = clientY - this.initialPosition.y;

            if (Math.abs(deltaX) > 5 || Math.abs(deltaY) > 5) {
                this.moved = true;
            }

            if (this.moved) {
                const position = this.getCurrentPosition(event);
                this.moveTo(position);
            }
        },
        getCurrentPosition(event) {
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
                position = { x: p0, y: p2, z: p1 };
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

            return position;
        }
    },
}
</script>
