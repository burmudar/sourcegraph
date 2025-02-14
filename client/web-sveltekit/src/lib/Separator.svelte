<script lang="ts" context="module">
    import { derived, type Writable } from 'svelte/store'

    import { createLocalWritable } from '$lib/stores'

    const dividerStore = createLocalWritable<Record<string, number>>('dividers', {})

    export function getSeparatorPosition(name: string, defaultValue: number): Writable<number> {
        const { subscribe } = derived(dividerStore, dividers => dividers[name] ?? defaultValue)

        return {
            subscribe,
            set(value) {
                dividerStore.update(dividers => ({ ...dividers, [name]: value }))
            },
            update(updater) {
                dividerStore.update(dividers => ({ ...dividers, [name]: updater(dividers[name]) }))
            },
        }
    }
</script>

<script lang="ts">
    /**
     * Store to write current position (0-1) to.
     */
    export let currentPosition: Writable<number>

    let divider: HTMLElement | null = null
    let offset = 0
    let dragging = false

    function onMouseMove(event: MouseEvent) {
        event.preventDefault()
        if (divider?.parentElement) {
            let width = (event.x - offset) / divider.parentElement.clientWidth
            if (width < 0) {
                width = 0
            } else if (width > 1) {
                width = 1
            }
            $currentPosition = width
        }
    }

    function endResize() {
        dragging = false
        window.removeEventListener('mousemove', onMouseMove)
        window.removeEventListener('mouseup', endResize)
    }

    function startResize(event: MouseEvent) {
        event.preventDefault()
        if (divider?.parentElement) {
            dragging = true
            offset = divider.parentElement.getBoundingClientRect().x + divider.clientWidth
            window.addEventListener('mousemove', onMouseMove)
            window.addEventListener('mouseup', endResize)
        }
    }
</script>

<!-- TODO: implement keyboard handlers. See https://www.w3.org/WAI/ARIA/apg/patterns/windowsplitter/ -->
<div
    bind:this={divider}
    role="separator"
    tabindex="0"
    aria-valuemin={0}
    aria-valuemax={100}
    aria-valuenow={$currentPosition}
    class:dragging
    on:mousedown={startResize}
>
    <!-- spacer is used to increase the interactable surface-->
    <div class="spacer" />
</div>

<style lang="scss">
    div[role='separator'] {
        flex-shrink: 0;
        position: relative;
        width: 1px;
        margin: 0 5px;
        background-color: var(--border-color);
        cursor: col-resize;

        .spacer {
            position: absolute;
            top: 0;
            bottom: 0;
            left: -5px;
            margin-left: -50%;
            width: 10px;
        }

        &.dragging {
            background-color: var(--oc-blue-3);
            margin: 0 4px;
            width: 3px;
        }
    }
</style>
