<script>
    import {getContext, onMount, createEventDispatcher} from 'svelte';
    import {is_function} from 'svelte/internal';
    import {
        createEventContent,
        height,
        toEventWithLocalDates,
        toViewWithLocalDates,
        setContent,
        repositionEvent,
        helperEvent,
        previewEvent
    } from '@event-calendar/common';

    export let chunk;
    export let longChunks = {};

    let {displayEventEnd, eventBackgroundColor, eventTextColor, eventClick, eventColor, eventContent, eventDidMount,
        eventMouseEnter, eventMouseLeave, theme, _view, _intlEventTime, _interaction, _classes, _draggable} = getContext('state');

    const dispatch = createEventDispatcher();

    let el;
    let event;
    let classes;
    let style;
    let content;
    let timeText;
    let margin = 1;
    let hidden = false;
    let display;

    $: event = chunk.event;

    $: {
        display = event.display;

        // Class & Style
        let bgColor = event.backgroundColor || $eventBackgroundColor || $eventColor;
        let txtColor = event.textColor || $eventTextColor;
        style =
            `width:calc(${chunk.days * 100}% + ${(chunk.days - 1) * 7}px);` +
            `margin-top:${margin}px;`
        ;
        if (bgColor) {
            style += `background-color:${bgColor};`;
        }
        if (txtColor) {
            style += `color:${txtColor};`;
        }

        classes = $_classes($theme.event, event);
    }

    // Content
    $: [timeText, content] = createEventContent(chunk, $displayEventEnd, $eventContent, $theme, $_intlEventTime, $_view);

    onMount(() => {
        if (is_function($eventDidMount)) {
            $eventDidMount({
                event: toEventWithLocalDates(event),
                timeText,
                el,
                view: toViewWithLocalDates($_view)
            });
        }
    });

    function createHandler(fn, display) {
        return !helperEvent(display) && is_function(fn)
            ? jsEvent => fn({event: toEventWithLocalDates(event), el, jsEvent, view: toViewWithLocalDates($_view)})
            : undefined;
    }

    function createDragHandler(resize) {
        return jsEvent => $_interaction.action.drag(event, jsEvent, resize);
    }

    export function reposition() {
        if (!el || previewEvent(display)) {
            return;
        }
        margin = repositionEvent(chunk, longChunks, height(el));
    }
</script>

<div
    bind:this={el}
    class="{classes}"
    {style}
    on:click={createHandler($eventClick, display)}
    on:mouseenter={createHandler($eventMouseEnter, display)}
    on:mouseleave={createHandler($eventMouseLeave, display)}
    on:pointerdown={!helperEvent(display) && $_draggable(event) && createDragHandler()}
>
    <div class="{$theme.eventBody}" use:setContent={content}></div>
    <svelte:component
        this={$_interaction.resizer}
        {event}
        on:pointerdown={createDragHandler(true)}
    />
</div>
