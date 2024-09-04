<script>
import BulldozairAnnotation from './BulldozairAnnotation.vue';

export default {
    render() {
        return null;
    },
    methods: {
        createAnnotation({ x, y, z, number, noteId, positionId, pinColor, strikeThrough }) {
            this.$viewer.state.addAnnotation({
                component: BulldozairAnnotation,
                props: {
                    index: number,
                    noteId,
                    positionId,
                    pinColor,
                    strikeThrough,
                },
                x,
                y,
                z,
            });
        },
    },
    onOpen() {
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
    cursor: pointer;
    margin-top: -52px;
    margin-left: -17.5px;
    width: 35px;
    height: 52px;
}

.bulldozair-annotation .bulldozair-annotation-label {
    background: white;
    padding: 2px;
    border-radius: 4px;
    width: 35px;
    text-align: center
}
.bulldozair-annotation .bulldozair-annotation-label.strike-through {
    text-decoration: line-through;
}

.bulldozair-annotation.grabbing {
    cursor: grabbing;
}

.bulldozair-annotation:focus {
    border: 2px solid var(--color-high);
    background-color: var(--color-warning);
}
</style>
