<script>
    import Simpledatepicker from './simpledatepicker.svelte';
    import { kineticscroll } from './kineticscroll.js';
	export let open = false;

    let options = {
/*         lang: {
            clear: 'Effacer', today: 'Aujourd\'hui', back: 'Revenir', initial: 'Initiale',
            months: ['Janvier', 'Février', 'Mars', 'Avril', 'Mai', 'Juin', 'Juillet', 'Août', 'Septembre', 'Octobre', 'Novembre', 'Décembre'],
            weekdays: ['Di', 'Lu', 'Ma', 'Me', 'Je', 'Ve', 'Sa'] 
        } */
        lang: 'pt',
        output: 'DD-MM-YYYY'
    }

    let exdt = ''; //new Date(2023, 0, 31);
    console.log(exdt)

    function toDate(isostr) { 
        let tmpdt = new Date(isostr.split(' ').shift() + ' 12:00:00');
        return  tmpdt; //new Date(tmpdt.getTime() + tmpdt.getTimezoneOffset() * 60000);
    }

    //console.log(toDate('2022-12-30 09:58:22'));
</script>

<section>
<main>
<!-- 	<h1>Simple Date Picker!</h1>
	<p></p> -->
    <br>

    <div class="date">
        <div><input type="text" bind:value={exdt} /></div>
        <div><button on:click={() => open = !open}>Picker</button>
        <br>
        <Simpledatepicker bind:date={exdt} bind:options bind:open></Simpledatepicker>
    </div>

</main>

<div id="content">
    <div id="indicator"></div>
    <div class="pure-u-1" id="view" use:kineticscroll={{indicator: 'indicator'}}>
        <table class="pure-u-1 pure-table pure-table-striped pure-table-horizontal">
          <tbody>
            <tr class="blue"><td>#2: Kinetic scrolling with momentum</td></tr>
            <tr class="blue"><td>No edge bouncing effect</td></tr>
            <tr><td>&nbsp;</td></tr>
            <tr><td>Abby <b>Riley</b></td></tr>
            <tr><td>Adela <b>Lewis</b></td></tr>
            <tr><td>Adriana <b>Bailey</b></td></tr>
            <tr><td>Aileen <b>Munoz</b></td></tr>
            <tr><td>Alba <b>Wiggins</b></td></tr>
            <tr><td>Alexandria <b>Morales</b></td></tr>
            <tr><td>Alicia <b>Richardson</b></td></tr>
            <tr><td>Alison <b>Wells</b></td></tr>
            <tr><td>Allyson <b>Kennedy</b></td></tr>
            <tr><td>Brandie <b>Vincent</b></td></tr>
            <tr><td>Brianna <b>Nichols</b></td></tr>
            <tr><td>Brigitte <b>George</b></td></tr>
            <tr><td>Brooke <b>Gould</b></td></tr>
            <tr><td>Candace <b>Hughes</b></td></tr>
            <tr><td>Carey <b>Whitney</b></td></tr>
            <tr><td>Carly <b>Stevens</b></td></tr>
            <tr><td>Carol <b>Olson</b></td></tr>
            <tr><td>Carolyn <b>Ellis</b></td></tr>
            <tr><td>Cassandra <b>Rodgers</b></td></tr>
            <tr><td>Cathleen <b>Petersen</b></td></tr>
            <tr><td>Cecile <b>Mueller</b></td></tr>
            <tr><td>Celina <b>Koch</b></td></tr>
            <tr><td>Charlotte <b>Oneil</b></td></tr>
            <tr><td>Cheri <b>Mcgowan</b></td></tr>
            <tr><td>Chris <b>Buckner</b></td></tr>
            <tr><td>Christie <b>Faulkner</b></td></tr>
            <tr><td>Chrystal <b>Farmer</b></td></tr>
            <tr><td>Clare <b>Edwards</b></td></tr>
            <tr><td>Claudia <b>Clark</b></td></tr>
            <tr><td>Colette <b>Pennington</b></td></tr>
            <tr><td>Connie <b>Gill</b></td></tr>
            <tr><td>Corina <b>Yang</b></td></tr>
            <tr><td>Corrine <b>Black</b></td></tr>
            <tr><td>Cynthia <b>Gilmore</b></td></tr>
            <tr><td>Danielle <b>Copeland</b></td></tr>
            <tr><td>Darlene <b>Armstrong</b></td></tr>
            <tr><td>Deanna <b>Pena</b></td></tr>
            <tr><td>Deborah <b>Schmidt</b></td></tr>
            <tr><td>Deidre <b>Sharp</b></td></tr>
            <tr><td>Delores <b>Mcgee</b></td></tr>
            <tr><td>Desiree <b>Berg</b></td></tr>
            <tr><td>Dianna <b>Cohen</b></td></tr>
            <tr><td>Dixie <b>Dunlap</b></td></tr>
            <tr><td>Dominique <b>Flynn</b></td></tr>
            <tr><td>Doreen <b>York</b></td></tr>
            <tr><td>Dorthy <b>Maynard</b></td></tr>
            <tr><td>Ebony <b>Farrell</b></td></tr>
            <tr><td>Edwina <b>Burnett</b></td></tr>
            <tr><td>Elba <b>Mccormick</b></td></tr>
            <tr><td>Elisa <b>Brown</b></td></tr>

            <tr><td>Shari <b>Stanton</b></td></tr>
            <tr><td>Shauna <b>Adkins</b></td></tr>
            <tr><td>Sheila <b>Lindsay</b></td></tr>
            <tr><td>Shelly <b>Soto</b></td></tr>
            <tr><td>Sherrie <b>Neal</b></td></tr>
            <tr><td>Silvia <b>Pope</b></td></tr>
            <tr><td>Sondra <b>Jackson</b></td></tr>
            <tr><td>Sophia <b>Dillard</b></td></tr>
            <tr><td>Stacie <b>Barry</b></td></tr>
            <tr><td>Stephanie <b>Hull</b></td></tr>
            <tr><td>Susana <b>Wynn</b></td></tr>
            <tr><td>Suzanne <b>Levine</b></td></tr>
            <tr><td>Tabatha <b>Ratliff</b></td></tr>
            <tr><td>Tamera <b>Phillips</b></td></tr>
            <tr><td>Tammie <b>Murray</b></td></tr>
            <tr><td>Tanisha <b>Brewer</b></td></tr>
            <tr><td>Taylor <b>Patrick</b></td></tr>
            <tr><td>Terri <b>Kramer</b></td></tr>
            <tr><td>Thelma <b>Reilly</b></td></tr>
            <tr><td>Tiffany <b>Farley</b></td></tr>
            <tr><td>Toni <b>Oneill</b></td></tr>
            <tr><td>Traci <b>Bates</b></td></tr>
            <tr><td>Trina <b>Bradley</b></td></tr>
            <tr><td>Ursula <b>Erickson</b></td></tr>
            <tr><td>Vanessa <b>Gross</b></td></tr>
            <tr><td>Veronica <b>Wilder</b></td></tr>
            <tr><td>Victoria <b>Snider</b></td></tr>
            <tr><td>Virgie <b>Rivas</b></td></tr>
            <tr><td>Wanda <b>Barr</b></td></tr>
            <tr><td>Wilda <b>Ferguson</b></td></tr>
            <tr><td>Winifred <b>Dejesus</b></td></tr>
            <tr><td>Young <b>Frank</b></td></tr>
          </tbody>
         </table>
     </div>
<br><br><br><br>
</div>
</section>
<style>

#indicator {
    position: absolute;
    z-index: 99;
    top: 0px;
    left: 3px;
    width: 4px;
    height: 30px;
    background-color: #888;
    border: none;
    margin: 2px 0;
}

section {
    display: flex;
}

table td {
    line-height: 320%;
    padding-left: 2em;
}

#content {
    overflow: hidden;
    width: 300px;
    height: 500px;
    margin: auto;
    background-color: #efefef;
}






    div.date {
        width: 400px;
        margin: auto;
    }



	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>