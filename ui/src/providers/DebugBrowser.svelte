<script>
    import { debugData } from '../utils/debugData';
	import { fly } from 'svelte/transition';

	let show = false;
    //EXAMPLE

	
	// Debug data for development
	const debugLocales = {
		dispatch_detach: "Detach",
		dispatch_attach: "Attach",
		unit: "Unit",
		units: "Units",
		additionals: "More Units",
	};

	const debugPlayer = {
		citizenid: "DEV123",
		job: { type: "police", name: "LSPD" },
		charinfo: { firstname: "Dev", lastname: "Officer" },
		metadata: { callsign: "1-Adam-1" }
	};

	const debugDispatchMenu = [
		{
			id: 1,
			message: "Armed Robbery in Progress",
			code: "10-31",
			icon: "fas fa-gun",
			time: Date.now() - 300000, // 5 minutes ago
			priority: 1,
			street: "Grove Street",
			coords: [-1234.56, 7890.12],
			gender: "Male",
			automaticGunFire: false,
			weapon: "Pistol",
			units: [],
			name: "John Doe",
			number: "555-0123",
			information: "Suspect wearing black hoodie",
			vehicle: "Sedan",
			color: "Black",
			plate: "ABC123",
			class: "Compact",
			doors: "4",
			heading: "North",
			jobs: ["police", "sheriff"]
		},
		{
			id: 2,
			message: "Traffic Stop",
			code: "10-54",
			icon: "fas fa-car",
			time: Date.now() - 600000, // 10 minutes ago
			priority: 3,
			street: "Main Street",
			coords: [-1100.25, 7500.75],
			gender: "Female",
			automaticGunFire: false,
			weapon: "",
			units: [debugPlayer],
			name: "Jane Smith",
			number: "555-0456",
			information: "Speeding violation",
			vehicle: "SUV",
			color: "Red",
			plate: "XYZ789",
			class: "SUV",
			doors: "4",
			heading: "South",
			jobs: ["police"]
		}
	];

	let options = [
		{
			component: 'Show',
			actions : [
				{
					name: "Show Dispatch",
					custom: true,
					customFunction: () => {
						// Initialize all debug data in sequence
						debugData([
							{
								action: "setBrowserMode",
								data: true
							},
							{
								action: "setupUI",
								data: {
									player: debugPlayer,
									locales: debugLocales,
									keybind: "G",
									maxCallList: 10,
									shortCalls: true
								}
							},
							{
								action: "setDispatchs",
								data: debugDispatchMenu
							},
							{
								action: "setVisible",
								data: true
							}
						], 100); // Small delay to ensure proper initialization
						console.log('Debug: All systems initialized');
					}
				},
				{
					name: "Add New Call",
					custom: true,
					customFunction: () => {
						const newCall = {
							id: Math.floor(Math.random() * 1000) + 100,
							message: ["Armed Robbery", "Traffic Accident", "Domestic Violence", "Burglary", "Drug Deal"][Math.floor(Math.random() * 5)],
							code: "10-" + (Math.floor(Math.random() * 99) + 10),
							icon: "fas fa-exclamation-triangle",
							time: Date.now(),
							priority: Math.floor(Math.random() * 3) + 1,
							street: "Test Street " + Math.floor(Math.random() * 50),
							coords: [Math.random() * 2000 - 1000, Math.random() * 2000 - 1000],
							gender: ["Male", "Female"][Math.floor(Math.random() * 2)],
							automaticGunFire: Math.random() > 0.8,
							weapon: ["Pistol", "Knife", "Rifle", "None"][Math.floor(Math.random() * 4)],
							units: [],
							name: "Test Person " + Math.floor(Math.random() * 100),
							number: "555-" + Math.floor(Math.random() * 9999).toString().padStart(4, '0'),
							information: "Random test incident",
							vehicle: ["Sedan", "SUV", "Truck", "Motorcycle"][Math.floor(Math.random() * 4)],
							color: ["Black", "White", "Red", "Blue", "Silver"][Math.floor(Math.random() * 5)],
							plate: Math.random().toString(36).substr(2, 7).toUpperCase(),
							class: "Standard",
							doors: "4",
							heading: ["North", "South", "East", "West"][Math.floor(Math.random() * 4)],
							jobs: ["police"]
						};
						const currentMenu = [...debugDispatchMenu, newCall];
						debugDispatchMenu.push(newCall);
						
						debugData([{
							action: "setDispatchs",
							data: currentMenu
						}]);
						console.log('Debug: New call added', newCall);
						console.log('Debug: Total calls now:', currentMenu.length);
					}
				}
			]
		},

		// {
		// 	component: 'Set Locales',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "updateLocale",
		// 			data: debugLocales,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Warrants',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setWarrantsData",
		// 			data: debugWarrants,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Bulletin',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setBulletinData",
		// 			data: debugBulletin,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Active Units',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setActiveUnits",
		// 			data: debugActiveUnits,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Camera',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setCameraData",
		// 			data: debugCamera,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Profile',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setProfileData",
		// 			data: debugProfiles,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Incidents',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setIncidentsData",
		// 			data: convertedIncidents,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Reports',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setReportsData",
		// 			data: debugReports,
		// 		},
		// 	]
		// },
		// {
		// 	component: 'Logs',
		// 	actions : [
		// 		{
		// 			name: "setData",
		// 			action: "setLogs",
		// 			data: debugLogs,
		// 		},
		// 	]
		// },
	]

</script>


<div class="absolute top-0 z-[1000]">
	<button class="bg-[#232B33] text-white p-2 font-medium"
		on:click={() => {
			show = !show;
		}}
	>
	Show
	</button>
	{#if show}
	<div class="w-fit h-fit bg-[#25303B] p-2">
		{#each options as option}
		<div class="flex flex-row gap-2 items-center">
			<p class="text-white mr-4">{option.component}</p>
			{#each option.actions as action}
			<button class="bg-[#0098A3] text-white p-2"
				on:click={() => {

					if (action.custom == true) {
						action.customFunction();
						return
					}
					debugData([
						{
							action: action.action,
							data: action.data,
						},
					])
				}}
			>
			{action.name}
			</button>
			{/each}
		</div>
		{/each}
	</div>
	{/if}
</div>

