<script>
    import {getContext} from 'svelte';
    import {addDuration, cloneDate, floor, rect} from '@event-calendar/common';

    let {_iEvents, _events, _viewDates, slotDuration, slotHeight, hiddenDays, _view, datesAboveResources,
        theme} = getContext('state');

    let y;
    let colDate;
    let colEl;
    let colRect;
    let resource;

    let _slotTimeLimits;

    let date;

    $: if ($_iEvents[0]) {
        removePointerEvent();
    }

    export function enterTimeGrid(date, el, jsEvent, slotTimeLimitsStore, resourceObj) {
        if (validEvent(jsEvent)) {
            colDate = date;
            colEl = el;
            colRect = rect(colEl);

            y = jsEvent.clientY;

            _slotTimeLimits = slotTimeLimitsStore;
            resource = resourceObj;
        }
    }

    export function enterDayGrid(date, jsEvent) {
        if (validEvent(jsEvent)) {
            colDate = date;
            colEl = null;

            y = _slotTimeLimits = resource = undefined;
        }
    }

    export function leave(jsEvent) {
        if (validEvent(jsEvent)) {
            removePointerEvent();
        }
    }

    function move() {
        if (!colDate) {
            return;
        }

        if (colEl) {
            // timeGrid
            let ry = y - colRect.top;
            date = addDuration(
                cloneDate(colDate),
                $slotDuration,
                floor(ry / $slotHeight + $_slotTimeLimits.min.seconds / $slotDuration.seconds)
            );
        } else {
            // dayGrid
            date = colDate;
        }

        if (!$_iEvents[1]) {
            createPointerEvent();
        }
        $_iEvents[1].start = date;
        $_iEvents[1].end = addDuration(cloneDate(date), $slotDuration);
        $_iEvents[1].resourceIds = resource ? [resource.id] : [];
    }

    export function handleScroll() {
        if (colEl) {
            colRect = rect(colEl);
            move();
        }
    }

    function handlePointerMove(jsEvent) {
        if (validEvent(jsEvent)) {
            y = jsEvent.clientY;
            move();
        }
    }

    function createPointerEvent() {
        $_iEvents[1] = {
            id: '{pointer}',
            title: '',
            display: 'pointer',
            extendedProps: {},
            backgroundColor: 'transparent'
        };
    }

    function removePointerEvent() {
        colDate = colEl = $_iEvents[1] = null;
    }

    function validEvent(jsEvent) {
        return jsEvent.isPrimary && jsEvent.pointerType === 'mouse';
    }
</script>

<svelte:window
    on:pointermove={handlePointerMove}
    on:scroll={handleScroll}
/>
