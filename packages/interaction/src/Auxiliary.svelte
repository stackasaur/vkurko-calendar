<script>
    import {getContext} from 'svelte';
    import {listen} from 'svelte/internal';
    import Action from './Action.svelte';
    import Pointer from './Pointer.svelte';
    import Resizer from './Resizer.svelte';

    let {theme, editable, eventStartEditable, eventDurationEditable, pointer, _bodyEl,
        _interaction, _classes, _draggable, _resizable, _scroll} = getContext('state');

    $_interaction.resizer = Resizer;

    $: $_classes = (className, event) => {
        switch (event.display) {
            case 'ghost': return `${$theme.event} ${$theme.ghost}`;
            case 'preview': return `${$theme.event} ${$theme.preview}`;
            case 'pointer': return `${$theme.event} ${$theme.pointer}`;
            default: return className + ($_draggable(event) ? ' ' + $theme.draggable : '');
        }
    };

    $: $_draggable = event => (event.startEditable ?? $eventStartEditable) || (event.editable ?? $editable);
    $: $_resizable = event => (event.durationEditable ?? $eventDurationEditable) || (event.editable ?? $editable);

    $: if ($_bodyEl) {
        listen($_bodyEl, 'scroll', bodyScrollHandler);
    }

    function bodyScrollHandler() {
        for (let component of Object.values($_interaction)) {
            component?.handleScroll?.();
        }
    }
</script>

<Action bind:this={$_interaction.action}/>
{#if $pointer}
    <Pointer bind:this={$_interaction.pointer}/>
{/if}
