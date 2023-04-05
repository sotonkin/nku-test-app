<script lang="ts">
    import { spring } from 'svelte/motion';		
	import { onMount, onDestroy } from 'svelte';    
	let power:number = 50;
	let uniqID:number;
	export let ID ='POWER_123';
	const displayed_power = spring();
	let client:any;
	let resived:boolean = false;
	const modulo=(n: number, m: number) =>((n % m) + m) % m;
	
	const sendEvent = (value:number)=>{
		client && !resived && client.publish(ID,JSON.stringify({value, sender:uniqID}));	
		resived = false;			
	};
	
	const setPower = (value:number)=>{
		resived = false;
		power = value;
	};

	$: displayed_power.set(power);
	$: offset = modulo($displayed_power, 1);
	$: sendEvent(power);	

	onMount(() => {
		uniqID = Math.random();
		client = mqtt.connect("wss://test.mosquitto.org:8081");
		client.on('connect', (connack: any)=>{		
			console.log('connected')
			connack && connack.returnCode === 0 && client.subscribe(ID,()=>{
				client.publish(ID,JSON.stringify({init:true, sender:uniqID}));
			});			
			client.on('message', (topic: any, message: any) =>{				
				let msg =  JSON.parse(String(message));								
				if(msg.sender !== uniqID){
					if(msg.init){
						resived = false;
						sendEvent(power);
					}else{
						resived = true;
						power = msg.value || 0;
					}		
				}
			});
		})
	});
	onDestroy(()=>{
		client && client.end();
	});
</script>
<h1>мощность</h1>
<div class="power">
	<button on:click={() => setPower(power>0?power -= 1:0)}>-</button>
	<div class="power-viewport">
		<div class="power-digits" style="transform: translate(0, {100 * offset}%)">
			<strong class="hidden" aria-hidden="true">{Math.floor($displayed_power + 1)+'%'}</strong>
			<strong>{Math.floor($displayed_power)+'%'}</strong>
		</div>
	</div>
	<button on:click={() => setPower(power<100?power += 1:100)}>+</button>
</div>
<input type="range" bind:value={power} min="0" max="100" step="5" class="range" on:change={()=>resived=false}>
<style>
	.power {
		display: flex;
		border-top: 1px solid rgba(0, 0, 0, 0.1);
		border-bottom: 1px solid rgba(0, 0, 0, 0.1);
		margin: 1rem 0;
	}
	.power button {
		width: 2em;
		padding: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		border: 0;
		background-color: transparent;
		touch-action: manipulation;
		font-size: 2rem;
	}
	.power button:hover {
		background-color: var(--color-bg-1);
	}
	.power-viewport {
		width: 8em;
		height: 4em;
		overflow: hidden;
		text-align: center;
		position: relative;
	}
	.range {
		width: 16em;
	}
	.power-viewport strong {
		position: absolute;
		display: flex;
		width: 100%;
		height: 100%;
		font-weight: 400;
		color: var(--color-theme-1);
		font-size: 3rem;
		align-items: center;
		justify-content: center;
	}
	.power-digits {
		position: absolute;
		width: 100%;
		height: 100%;
	}
	.hidden {
		top: -100%;
		user-select: none;
	}
</style>
