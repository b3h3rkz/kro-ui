<template>
    <div>
        <slot :open="open" :close="close" :toggle="toggle" name="activator"></slot>
        <Teleport to="#kro-portal">
            <div :tabindex="isOpen ? 1 : -1" :class="{'kro-dialog': true, 'kro-dialog--is-open': isOpen}">
                <div @click="() => { if (!persistent) { close(); } }" class="kro-dialog__scrim"></div>
                <kro-surface raised class="kro-dialog__content" @transitionend="onTransitionEnded">
                    <template v-if="shouldMountContent">
                        <div v-if="!!$slots.title" class="kro-dialog__title">
                            <slot :open="open" :close="close" :toggle="toggle" name="title"></slot>
                        </div>
                        <slot :open="open" :close="close" :toggle="toggle"></slot>
                        <div v-if="!!$slots.controls" class="kro-dialog__controls">
                            <slot :open="open" :close="close" :toggle="toggle" name="controls"></slot>
                        </div>
                    </template>
                </kro-surface>
            </div>
        </Teleport>
    </div>
</template>

<script lang='ts'>
    import { ref, onMounted, nextTick, onUnmounted } from 'vue';
    import { KroSurface } from '../Surface';

    export default {
        components: { KroSurface },
        props: {
            open: Boolean,
            persistent: Boolean,
        },
        setup(props, { emit }) {
            const isOpen = ref(false);
            const shouldMountContent = ref(false);

            const onTransitionEnded = (e) => {
                if (e.propertyName === 'transform') {
                    if (!isOpen.value) {
                        shouldMountContent.value = false;
                        emit('close');
                    }
                }
            }

            const close = (e) => { 
                if (e?.type === 'keydown') {
                    if (e.key === 'Escape') {
                        if (!props.persistent) {
                            isOpen.value = false;
                            window.removeEventListener('keydown', close);
                        }
                    }
                } else {
                    isOpen.value = false;
                }

            };

            const open = () => { 
                isOpen.value = true;
                shouldMountContent.value = true;
                window.addEventListener('keydown', close);

                emit('open');
            };

            const toggle = () => { isOpen ? close(null) : open(); };

            onMounted(async () => {
                if (props.open)
                    window.setTimeout(() => open(), 0);
            });

            onUnmounted(() => { window.removeEventListener('keydown', close); });

            return {
                isOpen,
                shouldMountContent,

                open,
                close,
                toggle,

                onTransitionEnded,
            }
        }
    }
</script>

<style lang="scss">

    @import '../../styles/general/layers';

    .kro-dialog {
        @include useLayer(dialog);

        position: fixed;
        top: 0; left: 0; right: 0; bottom: 0;
        display: grid;
        place-content: center;

        &:not(.kro-dialog--is-open) {
            pointer-events: none;

            .kro-dialog__scrim { opacity: 0; }
            .kro-dialog__content { opacity: 0; transform: scale(0); }
        }
    }

    .kro-dialog__scrim {
        position: absolute;
        top: 0; left: 0; right: 0; bottom: 0;
        background: rgba(0, 0, 0, .25);

        transition: opacity 150ms cubic-bezier(0.4, 0.0, 0.2, 1);
    }

    .kro-dialog__content {
        max-width: 600px;

        display: grid;
        gap: 1rem;
        grid-auto-flow: row;

        position: relative;
        transition: opacity 150ms cubic-bezier(0.4, 0.0, 0.2, 1),
                    transform 150ms cubic-bezier(0.4, 0.0, 0.2, 1);
    }

    .kro-dialog__title {
        font-size: 1.5rem;
        font-weight: 500;
        padding-top: 0.5rem;

        display: grid;
        grid-auto-flow: column;
        grid-auto-columns: min-content;
        white-space: nowrap;
        align-items: center;
        gap: 0.5rem;
    }

    .kro-dialog__controls {
        display: grid;
        grid-auto-flow: column;
        grid-auto-columns: min-content;
        gap: 1rem;
        justify-content: flex-end;
    }

</style>