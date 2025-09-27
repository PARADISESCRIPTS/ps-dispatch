<script>
  import { PLAYER, Locale, DISPATCH_MENU, DISPATCH_MUTED, DISPATCH_DISABLED, IS_RIGHT_MARGIN, processedDispatchMenu } from '@store/stores';
  import { fly, slide } from 'svelte/transition';
	import { timeAgo } from '@utils/timeAgo'
	import { SendNUI } from '@utils/SendNUI'
	import { onDestroy, onMount } from 'svelte'
  
  let activeCallId = null;
  let additionalUnitsVisible = {};
  let unsubscribe;

  $: menuRight = false;

  onMount(() => {
    unsubscribe = IS_RIGHT_MARGIN.subscribe((value) => {
      menuRight = value;
    })
    
    // Debug: Log store values to console
    DISPATCH_MENU.subscribe(value => {
      console.log('DISPATCH_MENU updated:', value);
    });
    
    PLAYER.subscribe(value => {
      console.log('PLAYER updated:', value);
    });
    
    Locale.subscribe(value => {
      console.log('Locale updated:', value);
    });
    
    processedDispatchMenu.subscribe(value => {
      console.log('processedDispatchMenu updated:', value);
    });
  })

  onDestroy(() => {
    unsubscribe();
  })
  
  function toggleDispatch(id) {
    if (activeCallId === id) {
      activeCallId = null;
    } else {
      activeCallId = id;
    }
  }

  function CheckIfAttached(units, player) {
    for (let i = 0; i < units.length; i++) {
      if (units[i].citizenid === player) {
        return true;
      }
    }
    return false;
  }

  function toggleAdditionalUnits(callId) {
    additionalUnitsVisible[callId] = !additionalUnitsVisible[callId];
  }

  function getAdditionalUnitsCount(dispatch) {
    const maxVisibleUnits = 3;
    const additionalUnits = dispatch.units.length - maxVisibleUnits;
    return Math.max(0, additionalUnits);
  }

  function toggleMargin() {
    menuRight = !menuRight;

    IS_RIGHT_MARGIN.set(menuRight);
  }

  function toggleMute() {
    DISPATCH_MUTED.update(value => !value);
    SendNUI("toggleMute", { boolean: $DISPATCH_MUTED });
  }

  function toggleAlerts() {
    DISPATCH_DISABLED.update(value => !value);
    SendNUI("toggleAlerts", { boolean: $DISPATCH_DISABLED });
  }

  function getDispatchData(dispatch) {
    return [
      { icon: 'fas fa-clock', label: 'Time', value: timeAgo(dispatch.time) },
      { icon: 'fas fa-user', label: 'Name', value: dispatch.name },
      { icon: 'fas fa-phone', label: 'Number', value: dispatch.number },
      { icon: 'fas fa-comment', label: 'Information', value: dispatch.information },
      { icon: 'fas fa-map-location-dot', label: 'Street', value: dispatch.street },
      { icon: 'fas fa-user', label: 'Gender', value: dispatch.gender },
      { icon: 'fas fa-gun', label: 'Automatic Gun Fire', value: dispatch.automaticGunFire },
      { icon: 'fas fa-gun', label: 'Weapon', value: dispatch.weapon },
      { icon: 'fas fa-car', label: 'Vehicle', value: dispatch.vehicle },
      { icon: 'fas fa-rectangle-list', label: 'Plate', value: dispatch.plate },
      { icon: 'fas fa-droplet', label: 'Color', value: dispatch.color },
      { icon: 'fas fa-car', label: 'Class', value: dispatch.class },
      { icon: 'fas fa-door-open', label: 'Doors', value: dispatch.doors },
      { icon: 'fas fa-compass', label: 'Heading', value: dispatch.heading },
      { icon: 'fas fa-user-group', label: 'Units', value: dispatch.units.length },
    ];
  }
</script>

<style>
  @keyframes emergency-lights {
    0% {
      background: linear-gradient(90deg, 
        rgba(220, 38, 38, 0.5) 0%, 
        rgba(220, 38, 38, 0.2) 25%, 
        transparent 50%, 
        rgba(30, 64, 175, 0.1) 75%, 
        rgba(30, 64, 175, 0.3) 100%);
      background-size: 200% 100%;
      background-position: 0% 0%;
    }
    50% {
      background: linear-gradient(90deg, 
        rgba(30, 64, 175, 0.3) 0%, 
        rgba(30, 64, 175, 0.1) 25%, 
        transparent 50%, 
        rgba(220, 38, 38, 0.2) 75%, 
        rgba(220, 38, 38, 0.5) 100%);
      background-size: 200% 100%;
      background-position: 100% 0%;
    }
    100% {
      background: linear-gradient(90deg, 
        rgba(220, 38, 38, 0.5) 0%, 
        rgba(220, 38, 38, 0.2) 25%, 
        transparent 50%, 
        rgba(30, 64, 175, 0.1) 75%, 
        rgba(30, 64, 175, 0.3) 100%);
      background-size: 200% 100%;
      background-position: 0% 0%;
    }
  }

  .emergency-overlay {
    position: relative;
    background-color: rgba(7, 17, 51, 0.98);
  }

  .emergency-overlay::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    animation: emergency-lights 1.5s infinite;
    pointer-events: none;
    z-index: 1;
  }

  .emergency-overlay > * {
    position: relative;
    z-index: 2;
  }

  :global(::-webkit-scrollbar) {
    width: 4px;
  }
  
  :global(::-webkit-scrollbar-track) {
    background: rgb(30, 41, 59);
  }
  
  :global(::-webkit-scrollbar-thumb) {
    background: rgb(59, 130, 246);
    border-radius: 2px;
  }
  
  :global(::-webkit-scrollbar-thumb:hover) {
    background: rgb(37, 99, 235);
  }
</style>

<div class="w-screen h-screen flex items-center justify-end { menuRight ? 'flex-row' : 'flex-row-reverse' } " transition:fly="{{ x: menuRight ? 400 : -400 }}">
  <!-- CONTROL PANEL -->
  <div class="w-[4vh] h-[85%] flex flex-col gap-[1vh]" class:ml-[1vh]={!menuRight} class:mr-[1vh]={menuRight}>

    <!-- REFRESH ALERTS -->
    <button class="w-full h-[4vh] flex items-center justify-center bg-slate-800 hover:bg-blue-600 text-slate-300 hover:text-white rounded-lg transition-all duration-200 border border-slate-700 hover:border-blue-500"
      title="Refresh Alerts"
      on:click={() => {
        SendNUI("refreshAlerts");
      }}
    >
      <i class="fas fa-arrows-rotate text-[1.6vh]"></i>
    </button>
    
    <!-- TOGGLE MUTE -->
    <button class="w-full h-[4vh] flex items-center justify-center {$DISPATCH_MUTED ? 'bg-red-900 hover:bg-red-800 border-red-800 hover:border-red-700' : 'bg-slate-800 hover:bg-blue-600 border-slate-700 hover:border-blue-500'} text-slate-300 hover:text-white rounded-lg transition-all duration-200 border"
      title={$DISPATCH_MUTED ? 'Unmute Dispatch' : 'Mute Dispatch'}
      on:click={toggleMute}
    >
      <i class="fas fa-volume-{$DISPATCH_MUTED ? "xmark" : "high"} text-[1.6vh]"></i>
    </button>
    
    <!-- TOGGLE ALERTS -->
    <button class="w-full h-[4vh] flex items-center justify-center {$DISPATCH_DISABLED ? 'bg-amber-900 hover:bg-amber-800 border-amber-800 hover:border-amber-700' : 'bg-slate-800 hover:bg-blue-600 border-slate-700 hover:border-blue-500'} text-slate-300 hover:text-white rounded-lg transition-all duration-200 border"
      title={$DISPATCH_DISABLED ? 'Enable Alerts' : 'Disable Alerts'}
      on:click={toggleAlerts}
    >
      <i class="fas fa-{$DISPATCH_DISABLED ? "bell-slash" : "bell"} text-[1.6vh]"></i>
    </button>
    
    <!-- CLEAR BLIPS -->
    <button class="w-full h-[4vh] flex items-center justify-center bg-slate-800 hover:bg-blue-600 text-slate-300 hover:text-white rounded-lg transition-all duration-200 border border-slate-700 hover:border-blue-500"
      title="Clear Map Blips"
      on:click={() => {
        SendNUI("clearBlips");
      }}
    >
    <i class="fas fa-ban text-[1.6vh]"></i>
    </button>
    
    <!-- Toggle Margin -->
    <button class="w-full h-[4vh] flex items-center justify-center bg-slate-800 hover:bg-blue-600 text-slate-300 hover:text-white rounded-lg transition-all duration-200 border border-slate-700 hover:border-blue-500"
      title="Toggle Position"
      on:click={toggleMargin}
    >
    <i class="fas fa-{menuRight ? "hand-point-left" : "hand-point-right"} text-[1.6vh]"></i>
    </button>
  </div>
  <!-- DISPATCH MENU -->
  <div class="w-[28%] h-[97%] overflow-hidden" class:ml-[2vh]={!menuRight} class:mr-[2vh]={menuRight}>
    <!-- HEADER -->
    <div class="bg-slate-900 border-l-4 border-blue-500 p-[1.5vh] mb-[1vh]">
      <div class="flex items-center justify-between">
        <h1 class="text-white font-semibold text-[1.8vh] flex items-center gap-[1vh]">
          <i class="fas fa-radio text-blue-400 text-[1.6vh]"></i>
          ACTIVE DISPATCHES
        </h1>
        <div class="text-blue-400 text-[1.2vh] font-mono">
          {$processedDispatchMenu?.length || 0}
        </div>
      </div>
    </div>
    
    <!-- CONTENT AREA -->
    <div class="h-[calc(100%-6vh)] overflow-y-auto pr-[0.5vh]">
      {#if $DISPATCH_MENU && $processedDispatchMenu.length > 0}
      {#each $processedDispatchMenu as dispatch, index}
      <div class="mb-[1vh] {dispatch.priority == 1 ? 'border-l-4 border-red-500' : 'border-l-4 border-blue-500'}">
        <!-- DISPATCH CARD -->
        <div class="{dispatch.priority == 1 ? 'emergency-overlay' : 'bg-slate-900 hover:bg-slate-800'} transition-colors duration-200">
          <!-- MAIN CONTENT -->
          <button class="w-full p-[1.5vh] text-left" on:click={() => toggleDispatch(dispatch.id)}>
            <!-- HEADER ROW -->
            <div class="flex items-center gap-[1vh] mb-[1vh]">
              <!-- CALL ID -->
              <div class="px-[1vh] py-[0.3vh] bg-blue-600 text-white font-bold text-[1.2vh] font-mono rounded-lg">
                #{dispatch.id}
              </div>
              <!-- CODE -->
              <div class="px-[1vh] py-[0.3vh] {dispatch.priority == 1 ? 'bg-red-600' : 'bg-slate-700'} text-white font-semibold text-[1.2vh] rounded-lg">
                {dispatch.code}
              </div>
              <!-- STATUS ICON -->
              <div class="ml-auto flex items-center gap-[1vh]">
                <div class="{dispatch.priority == 1 ? 'text-red-400' : 'text-blue-400'} text-[1.4vh]">
                  <i class="{dispatch.icon}"></i>
                </div>
                <i class="fas fa-chevron-{activeCallId === dispatch.id ? 'up' : 'down'} text-slate-500 text-[1vh]"></i>
              </div>
            </div>
            
            <!-- MESSAGE -->
            <div class="mb-[1vh]">
              <h3 class="text-white text-[1.4vh] font-medium">{dispatch.message}</h3>
              <p class="text-slate-400 text-[1.1vh] mt-[0.3vh]">
                <i class="fas fa-clock mr-[0.5vh]"></i>
                {timeAgo(dispatch.time)}
              </p>
            </div>
            
            <!-- QUICK INFO -->
            <div class="grid grid-cols-2 gap-[0.5vh] text-[1vh] text-slate-400">
              {#if dispatch.street}
                <div class="flex items-center gap-[0.3vh]">
                  <i class="fas fa-map-marker-alt text-[0.9vh] w-[1vh]"></i>
                  <span class="truncate">{dispatch.street}</span>
                </div>
              {/if}
              {#if dispatch.name}
                <div class="flex items-center gap-[0.3vh]">
                  <i class="fas fa-user text-[0.9vh] w-[1vh]"></i>
                  <span class="truncate">{dispatch.name}</span>
                </div>
              {/if}
              {#if dispatch.vehicle}
                <div class="flex items-center gap-[0.3vh]">
                  <i class="fas fa-car text-[0.9vh] w-[1vh]"></i>
                  <span class="truncate">{dispatch.vehicle}</span>
                </div>
              {/if}
              <div class="flex items-center gap-[0.3vh]">
                <i class="fas fa-users text-[0.9vh] w-[1vh]"></i>
                <span>{dispatch.units.length} Units</span>
              </div>
            </div>
          </button>
          <!-- EXPANDED DETAILS -->
          {#if activeCallId === dispatch.id}
          <div class="border-t border-slate-700 bg-slate-800 px-[1.5vh] py-[1vh]" transition:slide={{ duration: 300 }}>
            <!-- DETAILED INFO -->
            <div class="grid grid-cols-1 gap-[0.5vh] mb-[1vh] text-[1.1vh]">
              {#each getDispatchData(dispatch) as field}
                {#if field.value && !['Time', 'Units'].includes(field.label)}
                  <div class="flex items-center gap-[1vh] py-[0.3vh]">
                    <i class="{field.icon} text-blue-400 text-[1vh] w-[1.2vh]"></i>
                    <span class="text-slate-500 min-w-[8vh]">{field.label}:</span>
                    <span class="text-slate-300">{field.value}</span>
                  </div>
                {/if}
              {/each}
            </div>
            
            <!-- ASSIGNED UNITS -->
            {#if dispatch.units.length > 0}
              <div class="mb-[1vh]">
                <div class="border-t border-slate-700 pt-[1vh]">
                  <h4 class="text-slate-400 text-[1.1vh] font-medium mb-[0.5vh] flex items-center gap-[0.5vh]">
                    <i class="fas fa-users text-[1vh]"></i>
                    ASSIGNED UNITS ({dispatch.units.length})
                  </h4>
                  <div class="space-y-[0.3vh] max-h-[15vh] overflow-y-auto">
                    {#each dispatch.units.slice(0, additionalUnitsVisible[dispatch.id] ? dispatch.units.length : 3) as unit}
                      <div class="bg-slate-700 p-[0.8vh] flex items-center gap-[1vh] text-[1vh]">
                        <span class="px-[0.8vh] py-[0.2vh] bg-blue-600 text-white font-bold font-mono">
                          {unit.metadata.callsign}
                        </span>
                        <span class="px-[0.8vh] py-[0.2vh] uppercase text-white font-medium {unit.job.type == "leo" ? "bg-blue-700" : unit.job.type == "ems" ? "bg-red-700" : "bg-slate-600" }">
                          {unit.job.name}
                        </span>
                        <span class="text-slate-300 flex-1">
                          {unit.charinfo.firstname} {unit.charinfo.lastname}
                        </span>
                      </div>
                    {/each}
                    
                    {#if dispatch.units.length > 3 && !additionalUnitsVisible[dispatch.id]}
                      <button class="w-full bg-slate-600 hover:bg-slate-500 p-[0.5vh] transition-colors duration-200 text-slate-300 text-[1vh]" on:click={() => toggleAdditionalUnits(dispatch.id)}>
                        <i class="fas fa-chevron-down mr-[0.5vh]"></i>
                        Show {getAdditionalUnitsCount(dispatch)} more units
                      </button>
                    {/if}
                  </div>
                </div>
              </div>
            {/if}
            
            <!-- ACTION BUTTON -->
            <button class="w-full {dispatch.priority == 1 ? (CheckIfAttached(dispatch.units, $PLAYER.citizenid) ? 'bg-red-800 hover:bg-red-700' : 'bg-red-700 hover:bg-red-600') : (CheckIfAttached(dispatch.units, $PLAYER.citizenid) ? 'bg-amber-700 hover:bg-amber-600' : 'bg-blue-700 hover:bg-blue-600')} p-[1vh] transition-colors duration-200 text-white font-medium text-[1.2vh] flex items-center justify-between"
              on:click={() => {
                if (CheckIfAttached(dispatch.units, $PLAYER.citizenid)) {
                  SendNUI("detachUnit", dispatch );
                  SendNUI("refreshAlerts");
                } else {
                  SendNUI("attachUnit", dispatch );
                  SendNUI("refreshAlerts");
                }
              }}>
              <div class="flex items-center gap-[1vh]">
                <i class="fas fa-{CheckIfAttached(dispatch.units, $PLAYER.citizenid) ? 'unlink' : 'link'} text-[1.1vh]"></i>
                <span>
                  {#if CheckIfAttached(dispatch.units, $PLAYER.citizenid)}
                    {$Locale.dispatch_detach}
                  {:else}
                    {$Locale.dispatch_attach}
                  {/if}
                </span>
                <span class="text-[1vh] opacity-75">({dispatch.units.length} {$Locale.units})</span>
              </div>
              <i class="fas fa-chevron-right text-[1vh]"></i>
            </button>
          </div>
          {/if}
        </div>
      </div>
      {/each}
      {:else}
        <!-- EMPTY STATE -->
        <div class="flex flex-col items-center justify-center h-full text-center py-[4vh]">
          <div class="bg-slate-900 border-l-4 border-blue-500 p-[3vh] text-center">
            <i class="fas fa-radio text-[4vh] text-slate-600 mb-[1vh]"></i>
            <h3 class="text-slate-400 font-medium text-[1.6vh] mb-[0.5vh]">No Active Dispatches</h3>
            <p class="text-slate-500 text-[1.2vh]">All units on standby</p>
          </div>
        </div>
      {/if}
    </div>
  </div>
</div>