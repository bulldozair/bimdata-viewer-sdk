<template>
    <div
        class="bulldozair-annotation"
        @mousedown="onMouseDown"
        @mousemove="onMouseMove"
        @touchmove="onMouseMove"
        @mouseup="onMouseUp"
        @touchend="onMouseUp"
        @touchcancel="onMouseUp"
    >
        <div 
            class="bulldozair-annotation-label"
            :class="{ 'strike-through': strikeThrough }"
            v-if="index !== undefined">
            {{ index }}
        </div>
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
        annotation: Object,
        index: Number,
        noteId: String,
        positionId: String,
        pinColor: String,
        strikeThrough: Boolean,
    },
    data() {
        return {
            grabbing: false,
            moved: false,
            initialPosition: {},
        };
    },
    methods: {
        onMouseDown(event) {
            this.grabbing = true;
            this.moved = false;
            this.initialPosition = { x: event.clientX, y: event.clientY };
        },
        onMouseMove(event) {
            const { clientX, clientY } = event;
            const deltaX = clientX - this.initialPosition?.x ?? 0;
            const deltaY = clientY - this.initialPosition?.y ?? 0;

            this.moved = Math.abs(deltaX) > 5 || Math.abs(deltaY) > 5;
        },
        onMouseUp() {
            this.grabbing = false;
            if (!this.moved) {
                this.$viewer.globalContext.hub.emit('bz-annotation-click', {
                    noteId: this.noteId,
                    positionId: this.positionId
                });
            } else {
                this.$viewer.globalContext.hub.emit('bz-annotation-move', {
                    noteId: this.noteId,
                    positionId: this.positionId,
                    position: { x: this.annotation.x, y: this.annotation.y, z: this.annotation.z },
                });
            }
        },
    },
}
</script>
