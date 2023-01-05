<script>
    import { createEventDispatcher, tick  } from 'svelte';

    export let date;
    export let options = {};
    export let open = false;

    const dispatch = createEventDispatcher();
    let root;
    let mode = 0;
    let dates = [];
    let settings = {};
    let translations = {
        pt: {
            clear: 'Apagar', today: 'Hoje', back: 'Voltar', initial: 'Inicial',
            months: ['Janeiro', 'Fevereiro', 'Março', 'Abril', 'Maio', 'Junho', 'Julho', 'Agosto', 'Setembro', 'Outubro', 'Novembro', 'Dezembro'], 
            weekdays: ['D', 'S', 'T', 'Q', 'Q', 'S', 'S'] 
        },
        en: { 
            clear: 'Clear', today: 'Today', back: 'Back', initial: 'Initial',
            months: ['January', 'February', 'Mars', 'April', 'May', 'June', 'July', 'August', 'September', 'October', 'November', 'December'], 
            weekdays: ['Su', 'Mo', 'Tu', 'We', 'Th', 'Fr', 'Sa'] 
        },
        es: { 
            clear: 'Borrar', today: 'Hoy', back: 'Volver', initial: 'Inicial',
            months: ['Eneiro', 'Febrero', 'Marzo', 'Abril', 'Mayo', 'Junio', 'Julio', 'Agosto', 'Septiembre', 'Octubre', 'Noviembre', 'Diciembre'], 
            weekdays: ['Do', 'Lu', 'Ma', 'Mi', 'Ju', 'Vi', 'Sa'] 
        }            
    }

    let defaults = {
        min: '',                                    // Minimum date allowed to be picked. Empty = no limits
        max: '',                                    // Maximum date allowed to be picked. Empty = no limits
        align: 'center',                            // Picker alignment relative to it's container
        lang: 'en',                                 // Can be a supported ISO code or an object with the translations
        format: ['D', 'M', 'Y'],                    // How date appears at the top bar and also how it is returned if string
        output: 'date',                             // Can be date=As date object, utc=As date.UTC object, a date format to return as string (eg. DD-MM-YYYY)
        showButtons: true,                          // Show the bottom command bar
        keyboard: false,                            // If true keyboard shortcuts are available
        sundayFirst: true,                          // If true sunday is the first day of the week, otherwise it's last
        yearRange: {                                // Range of years to browse in
            start: new Date().getFullYear() - 80, 
            end: new Date().getFullYear() + 20
        }
    }
    let currTrans = null;                           // The translation applied
    let workDate;                                   // Date to work on. It's the pre selected date
    let initialDate;                                // The date initially passed to the picker                                      
    let years = [];                                 // Years array to show the years pane
    let keysEventFlag = false;                      // Controls setting and unsetting keyboard events
    let weekDaysArr = [];                           // Array with the week days abbreviations
    let validYearMin = 0;                           // Control the valid years functionality
    let validYearMax = Number.MAX_VALUE;            // Control the valid years functionality

    $: if (open) { 
        if (settings.keyboard && !keysEventFlag) { 
            keysEventFlag = true;
            document.addEventListener('keyup', keysEvent);
        }
        if (!workDate) {
            let numYears = settings.yearRange.end - settings.yearRange.start + 1;

            years = Array(numYears).fill(0).map((e, i) => { return { y: settings.yearRange.start + i, inrange: 0 }});
            console.log(currTrans)
            workDate = setWorkingDate(date || new Date()); 
            initialDate = new Date(workDate.date);
            renderMonth(workDate);
        }            
    } else {
        if (settings.keyboard && keysEventFlag) { 
            document.removeEventListener('keyup', keysEvent); 
            keysEventFlag = false;
        }
        workDate = null;
    }

    $: updateSettings(options);

    function keysEvent(evt) {
        switch(evt.code) {
            case 'Enter': finish(workDate.day); break;
            case 'Escape': cmds('back'); break;
            case 'ArrowLeft': NavigateTo(-1); break;
            case 'ArrowRight': NavigateTo(1); break;
        }
    }

    function updateSettings() {
        settings = {...defaults, ...options};

        if (settings.min && typeof settings.min == 'string') { settings.min = toDate(settings.min); }
        if (settings.max && typeof settings.max == 'string') { settings.max = toDate(settings.max); }
        if (settings.min) { validYearMin = settings.min.getFullYear(); }
        if (settings.max) { validYearMax = settings.max.getFullYear(); }

        // If language is blank, set the default
        if (!settings.lang) { settings.lang = 'en'; }

        // If language is a string, set the appropriate language translation structure
        if (typeof settings.lang == 'string') { 
            currTrans = translations[Object.keys(translations).indexOf(settings.lang) == -1 ? 'en' : settings.lang];
        } else { 
            // Otherwise it must be a custom translation, so let's set it!
            currTrans = settings.lang;
        }
        let wdArr = JSON.parse(JSON.stringify(currTrans.weekdays));
        if (settings.sundayFirst) {
            weekDaysArr = wdArr;
        } else {
            wdArr.push(wdArr.shift());
            weekDaysArr = wdArr;            
        }
    }

    function clickOutside(node) { 
        window.addEventListener('click', handleClick);
        
        function handleClick(e) {
            if (!node.contains(e.target)) {
                e.preventDefault();
                node.dispatchEvent(new CustomEvent('outclick'));
            }
        }
        return {
            destroy() {
                // the node has been removed from the DOM
                window.removeEventListener('click', handleClick)
            }
        };
    } 

    // Sets a date to the working date object
    function setWorkingDate(dt) { 
        if (!dt) { return; }
        if (typeof dt == 'string') { dt = toDate(dt); }

        return {
            date: dt,
            year: dt.getFullYear(),
            month: dt.getMonth() + 1,
            indexMonth: dt.getMonth(),
            monthstr: currTrans.months[dt.getMonth()],
            day: dt.getDate()
        }
    }
    
    function isValidDate(date) {
        return date && Object.prototype.toString.call(date) === "[object Date]" && !isNaN(date);
    }

    function toDate(isostr) { 
        if (!isostr) { return null; }
        if (isValidDate(isostr)) { return isostr; }
        let tmpdt = new Date(isostr.split(/[T ]/).shift() + ' 12:00:00'); 
        return isValidDate(tmpdt) ? tmpdt : null; 
    }

    // Compare 2 dates for equality. Can be string or date. Ignores time.
    function equalDates(d1, d2) {
        let dx1 = typeof d1 == 'string' ? new Date(d1) : d1;
        let dx2 = typeof d2 == 'string' ? new Date(d2) : d2;
        return dx1.getFullYear() == dx2.getFullYear() && dx1.getMonth() == dx2.getMonth() && dx1.getDate() == dx2.getDate();
    }

    function addDays(date, days) { return new Date(date.setDate(date.getDate() + days)); }

    function isToday(dt) {
        return equalDates(new Date(), dt);
    }

    // Returns whether the passed argument is a valid date object
    function isDate(dt) {
        return dt instanceof Date && !isNaN(dt.valueOf());
    }

    // Clones a date
    function clone(dt) {
        return isDate(dt) ? toDate(dt.toISOString().replace('T', ' ')) : null;
    }

    // Return a date corresponding to the first day of the month
    function firstDayOfMonth(dt) { 
        return new Date(dt.getFullYear(), dt.getMonth(), 1, 12, 0, 0);
    }

    // Returns whether the year passed is leap year
    function isLeapYear(year) {
        return ((year % 4 == 0) && (year % 100 != 0)) || (year % 400 == 0);
    }

    // Returns number of days in a passed month
    function getDaysInMonth(year, month) {
        return [31, (isLeapYear(year) ? 29 : 28), 31, 30, 31, 30, 31, 31, 30, 31, 30, 31][month];
    }

    // Responds for the arrows
    function NavigateTo(direction) { //console.log(workDate)
        let dt = addTo(workDate.date, direction, 'm'); //console.log(dt)
        workDate = setWorkingDate(dt);console.log(workDate); console.log('--------------------------------')
        renderMonth(workDate);
        //console.log(dates)
        mode = 0;
    }


    // Adds or subtracts days, months or years from passed date
    function addTo(date2add, amount, unit) { 
        let resp = new Date(date2add);

        switch (unit) {
            case 'd':
                resp = new Date(resp.setDate(resp.getDate() + amount));
                break;

            case 'm':
                let mday = date2add.getDate();
                resp.setDate(1);
                resp.setMonth(resp.getMonth() + amount);
                resp.setDate(Math.min(mday, getDaysInMonth(resp.getFullYear(), resp.getMonth())));      
                break;  

            case 'y':
                resp = new Date(resp.setFullYear(amount));
                break;                      
        }
        return resp;
    }

    // Renders a month of a year passed in dt.
    // min and max determines which cells can be selected
    // inidate is the date at the box when it was clicked
    // inpFmt is the input format being used
    function renderMonth(wkdate) { 
        let chosen = wkdate.date; //console.log('chosen', chosen)
        if (!chosen) { return []; }

        let counter = firstDayOfMonth(chosen);                      // Position passed date at first of month 
        let gday = counter.getDay();
        let repos = settings.sundayFirst ? gday :  gday == 0 ? 6 : gday - 1;        console.log(counter.getDay(), repos)
        counter = addTo(counter, -repos, 'd');    // Position first of month date at last (previous) first day of week
        let m = chosen.getMonth();
        let lines = [];

        for (let l = 0; l < 6; l++) {
            let line = [];
            for (let c = 0; c < 7; c++) {
                let disabled = (settings.min && counter < settings.min) || (settings.max && counter > settings.max);
                let obj = { 
                    date: clone(counter),                               // Full date for this cell
                    day: counter.getDate(),                             // Day of the month
                    outofmonth: (counter.getMonth() != m),              // If this date belongs to the current month
                    disabled: disabled,                                 // If this date is disabled
                    today: isToday(counter),                            // Indicates today's date
                    selected: equalDates(workDate.date, counter)          // Indicates the initial date (passed to this component when it opened)
                };
                line.push(obj);
                counter = addDays(counter, 1);
            }
            lines.push(line);
        }
        dates = lines; 
    }

    // Choose new month or year from the respective panes
    function choose(amount, type) {
        type == 'y' ? workDate.date.setFullYear(amount) : workDate.date.setMonth(amount);
        workDate = setWorkingDate(workDate.date);
        renderMonth(workDate);        
        mode = 0;  console.log(this)
    }

    // Completes a date selection and send it back to whoever called
    function finish(day) {
        workDate.date.setDate(day);
        let fmt = settings.output;

        switch(fmt) {
            case 'date': date = workDate.date; break;
            case 'iso': date = workDate.date.toISOString().substr(0, 10); break;
            default:
                let tmpfmt = fmt;
                fmt.split(/[T /:\-.]/).forEach((d) => {
                    switch(d) {
                        case 'DD':   tmpfmt = tmpfmt.replace('DD', ('0' + workDate.date.getDate()).slice(-2)); break;
                        case 'D':    tmpfmt = tmpfmt.replace('D', workDate.date.getDate()); break;
                        case 'MM':   tmpfmt = tmpfmt.replace('MM', ('0' + (workDate.date.getMonth() + 1)).slice(-2)); break;
                        case 'M':    tmpfmt = tmpfmt.replace('M', workDate.date.getMonth() + 1); break;
                        case 'YYYY': tmpfmt = tmpfmt.replace('YYYY', workDate.date.getFullYear()); break;
                        case 'YY':   tmpfmt = tmpfmt.replace('YY', workDate.date.getFullYear().slice(-2)); break;
                    }
                });
                date = tmpfmt;
                break;
        }
        dispatch('date', date);
        open = false;
    }

    // Change panes
    async function switchMode(newmode) {
        if (newmode == mode) { mode = 0; } else { mode = newmode; }
        if (mode == 2) { 
            await tick();
            let el = root.querySelector('div.years > ol > li.selected');
            el.scrollIntoView(); 
        }
    }

    function cmds(arg) {
        switch(arg) {
            case 'today': alert('ok')
                workDate = setWorkingDate(new Date());
                renderMonth(workDate);
                mode = 0;
                break;

            case 'inital':
                workDate = setWorkingDate(initialDate);
                renderMonth(workDate);
                mode = 0;
                break;

            case 'clear':
                date = null;
                initialDate = null;
                workDate = null;
                dates = [];
                open = false;
                break;

            case 'back':
                initialDate = null;
                workDate = null;
                dates = [];                
                open = false;                
        }
    }
</script>

{#if open && workDate}
    <section bind:this={root} class="{settings.align}" use:clickOutside on:outclick={cmds.bind(this, 'back')}>
        <div>
            <b class="chevron left" on:click={NavigateTo.bind(this, -1)}></b>
            <span>
                <span on:click={switchMode.bind(this, 0)}>{workDate.day}</span>
                <span on:click={switchMode.bind(this, 1)}>{workDate.monthstr}</span>
                <span on:click={switchMode.bind(this, 2)}>{workDate.year}</span>
            </span>

            <b class="chevron right" on:click={NavigateTo.bind(this, 1)}></b>
        </div>
        <aside>
            <div>
                <ul>
                    {#each weekDaysArr as day}
                        <li class="weekday">{day}</li>
                    {/each}
                    <br>
                    {#each dates as line}
                        {#each line as cell}
                            <!-- svelte-ignore a11y-click-events-have-key-events -->
                            <li class:disabled={cell.disabled} class:outofmonth={cell.outofmonth && !cell.disabled} class:today={cell.today}
                                class:selected={cell.selected && !cell.disabled} on:click={finish.bind(this, cell.day)}>{cell.day}</li>
                        {/each}
                        <br>
                    {/each}
                </ul>
            </div>
            {#if mode == 1}
                <div class="months">
                    <ol>
                        {#each currTrans.months as month, ind}
                            <!-- svelte-ignore a11y-click-events-have-key-events -->
                            <li on:click={choose.bind(this, ind, 'm')} class:selected={workDate.indexMonth == ind}>{month}</li>
                        {/each}
                    </ol>
                </div>
            {:else if mode == 2}
                <div class="years">
                    <ol>
                        {#each years as year}
                            <!-- svelte-ignore a11y-click-events-have-key-events -->
                            <li class:selected={workDate.year == year.y} on:click={choose.bind(this, year.y, 'y')} 
                                class:valid={validYearMin > 0 && validYearMax < Number.MAX_VALUE && year.y >= validYearMin && year.y <= validYearMax}>
                                {year.y}
                            </li>
                        {/each}
                    </ol>
                </div>
            {/if}            
        </aside>
        {#if settings.showButtons}
            <footer>
                <!-- svelte-ignore a11y-click-events-have-key-events -->
<!--                 <span on:click={cmds.bind(this, 'initial')}>Initial</span> -->
                <span on:click={cmds.bind(this, 'today')}>{currTrans.today}</span>
                <span on:click={cmds.bind(this, 'clear')}>{currTrans.clear}</span>
                <span on:click={cmds.bind(this, 'back')}>{currTrans.back}</span>
            </footer>
        {/if}
    </section>
{/if}

<style>
    section.center  { margin: 0 auto; }
    section.left    { margin: 0 0 auto; }
    section.right   { margin: 0 0 0 auto; }

    footer {
        display: flex;
        justify-content: space-between;
        padding: 0;
        border-top: 1px solid #eee;
        text-transform: uppercase;
        font-size: 10px;
        letter-spacing: 0.5px;
        font-weight: 600;
        user-select: none;
        color: dodgerblue;
    }

    footer > span {
        cursor: pointer;
        padding: 3px 8px;
        transition: all 0.4s ease;
    }

    footer > span:hover {
        background-color: dodgerblue;
        color: #fff;
    }    

    aside { position: relative; }

    aside > div.months,
    aside > div.years {
        position: absolute;
        left: 0;
        top: 0;
        right: 0;
        bottom: 0;
        background-color: #fff;
        overflow: hidden;
    }

    aside ol {
        padding: 10px;
        margin: 0;
        list-style: none;
        display: flex;
        flex-wrap: wrap;
        align-items: stretch;
    }

    aside ol > li {
        width: 50%;
        padding: 7px 0;
        text-transform: uppercase;
        transition: all 0.4s ease;
        cursor: pointer;
        border: 1px solid transparent;
    }

    aside div.years {
        overflow-y: scroll;
        scrollbar-width: none;   
    }

    aside div.years::-webkit-scrollbar {
        width: 0;
        height: 0;
    }

    aside div.years ol > li {
        width: calc(25% - 4px);
        margin: 2px;
    }

    aside div.years ol > li.valid { border-color: var(--int-barbkg); }

    .weekday {
        font-weight: 700;
        border: none;
        pointer-events: none;
    }

    .disabled {
        pointer-events: none;
        color: var(--color-off, #eee);
        border-color: transparent;
    }

    .outofmonth { color: var(--color-off, #eee); }

    .today {
        border: 1px solid dodgerblue;
        box-shadow: inset 0 0 1px 1px dodgerblue;
    }

    .selected {
        background-color: crimson;
        color: #fff;
    }

    section * { box-sizing: border-box; }

    section {
        display: flex;
        background-color: var(--int-background);
        font-size: var(--int-fontsize);
        border: 1px solid var(--int-bordercolor);
        width: fit-content;
        flex-direction: column;
        user-select: none;

        --int-fontsize: var(--fontsize, 16px);
        --int-background: var(--background, #fff);
        --int-color: var(--color, #888);
        --int-disabled: var(--disabled, #eee);
        --int-barbkg: var(--barbkg, dodgerblue);
        --int-barcolor: var(--barcolor, #fff);
        --int-bordercolor: var(--bordercolor, #eee);
        --int-selectedbkg: var(--selectedbkg, crimson);
        --int-selectedcolor: var(--selectedcolor, #fff);
    }

    section > div:first-child {
        background-color: var(--int-barbkg);
        color: var(--int-barcolor);
        display: flex;
        justify-content: space-between;
        align-items: center;
        flex-direction: row;
        font-weight: 700;
        padding: 10px;
    }

    section > div:first-child * { cursor: pointer; }

    section > div:last-child {
        background-color: #fff;
        color: #777;
        flex-grow: 1;
        padding: 5px;
    }    

/*     section > div:first-child > div {
        font-weight: 700;
        padding: 10px 12px;
    }

    section > div:first-child > div > div:first-child {
        font-size: 30px;
    } */

    section > div:first-child                { text-transform: uppercase; }
    section > div:first-child > b            { margin-bottom: -5px; }
    section > div:first-child > b:last-child { margin-right: 2px; }

/*     section > div:first-child > div > div:last-child {
        font-size: 110%;
    }    */ 

    ul {
        padding: 0;
        margin: 0;
        list-style: none;
    }

    ul > li {
        width: calc(14.2% - 1.8px);
        padding: 5px 10px;
        display: inline-block;
        cursor: pointer;
        border: 1px solid #efefef;
        margin: 1px;
        transition: all 0.4s ease;
    }

    ul > li:not(.disabled):hover,
    ol > li:hover {
        background-color: crimson;
        color: #fff;
        border-color: rgb(66, 8, 19);
    }

    .chevron::before {
        border-style: solid;
        border-width: 0.25em 0.25em 0 0;
        content: '';
        display: inline-block;
        height: 0.45em;
        left: 0.15em;
        position: relative;
        top: 0.15em;
        transform: rotate(-45deg);
        vertical-align: top;
        width: 0.45em;
    }

    .chevron.right:before {
        left: 0;
        transform: rotate(45deg);
    }

    .chevron.bottom:before {
        top: 0;
        transform: rotate(135deg);
    }

    .chevron.left:before {
        left: 0.25em;
        transform: rotate(-135deg);
    }    
</style>