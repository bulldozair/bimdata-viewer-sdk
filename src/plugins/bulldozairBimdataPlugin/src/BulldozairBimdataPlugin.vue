<template>
    <!-- https://vuejs.org/guide/essentials/template-syntax.html -->
    <div></div>
</template>

<script>
import BulldozairAnnotation from './bulldozairAnnotation.vue';

export default {
    data() {
        return {
            index: 0,
        };
    },
    render() {
        return null;
    },
    methods: {
        createAnnotation: ({ x, y, z, number, noteId, positionId }) => {
            const { state } = this.$viewer;
            const annotation = state.addAnnotation({
                component: BulldozairAnnotation,
                props: {
                    index: number,
                    noteId,
                    positionId,
                    moveDone() {
                    },
                    moveTo(position) {
                        return Object.assign(annotation, position)
                    },
                    remove() {
                        this.$viewer.globalContext.hub.emit('bz-annotation-remove', { annotation });
                        return state.removeAnnotation(annotation)
                    },
                },
                x,
                y,
                z,
            });
        },
    },
    onOpen() {
        const { state } = this.$viewer;
        const context = this.$viewer.localContext;
        context.startAnnotationMode(({ x, y, z, object }) => {
            this.$viewer.globalContext.hub.emit('bz-annotation-create', { x, y, z, object });
            context.stopAnnotationMode();
            this.$close();
        });
    },
};
</script>

<style>
.bulldozair-annotation {
    user-select: none;
    cursor: grab;
    margin-top: -62.5px;
    margin-left: -17.5px;
}

.bulldozair-annotation .bulldozair-annotation-label {
    background: white;
    padding: 2px;
    border-radius: 4px;
    width: 35px;
    text-align: center
}

.bulldozair-annotation.grabbing {
    cursor: grabbing;
}

.bulldozair-annotation:focus {
    border: 2px solid var(--color-high);
    background-color: var(--color-warning);
}
</style>
