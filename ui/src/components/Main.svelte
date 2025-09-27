<script>
  import { onMount, afterUpdate } from 'svelte';
  import { DISPATCH, removeDispatch, RESPOND_KEYBIND, IS_RIGHT_MARGIN, shortCalls } from '@store/stores';
  import { fly } from 'svelte/transition';
  import { timeAgo } from '@utils/timeAgo';

  let notifications = [];
  
  DISPATCH.subscribe(value => {
    notifications = value || [];
  });

  function removeNotification(id) {
    removeDispatch(id);
  }

  onMount(() => {
    notifications.forEach(notification => {
      const { data, timer } = notification;
      setTimeout(() => {
        removeNotification(data.id);
      }, timer);
    });
  });

  afterUpdate(() => {
    notifications.forEach(notification => {
      const { data, timer } = notification;
      setTimeout(() => {
        removeNotification(data.id);
      }, timer);
    });
  });

  function getDispatchData(dispatch) {
    if ($shortCalls) {
      return [
        { label: 'Call', value: dispatch.data.message },
        { icon: 'fas fa-comment', label: 'Information', value: dispatch.data.information },
      ];
    } else {
      return [
      { icon: 'fas fa-clock', label: 'Time', value: timeAgo(dispatch.data.time) },
      { icon: 'fas fa-user', label: 'Name', value: dispatch.data.name },
      { icon: 'fas fa-phone', label: 'Number', value: dispatch.data.number },
      { icon: 'fas fa-comment', label: 'Information', value: dispatch.data.information },
      { icon: 'fas fa-map-location-dot', label: 'Street', value: dispatch.data.street },
      { icon: 'fas fa-user', label: 'Gender', value: dispatch.data.gender },
      { icon: 'fas fa-gun', label: 'Automatic Gun Fire', value: dispatch.data.automaticGunFire },
      { icon: 'fas fa-gun', label: 'Weapon', value: dispatch.data.weapon },
      { icon: 'fas fa-car', label: 'Vehicle', value: dispatch.data.vehicle },
      { icon: 'fas fa-rectangle-list', label: 'Plate', value: dispatch.data.plate },
      { icon: 'fas fa-droplet', label: 'Color', value: dispatch.data.color },
      { icon: 'fas fa-car', label: 'Class', value: dispatch.data.class },
      { icon: 'fas fa-door-open', label: 'Doors', value: dispatch.data.doors },
      { icon: 'fas fa-compass', label: 'Heading', value: dispatch.data.heading },
      ];
    }
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

<div class="w-screen h-screen flex justify-end { $IS_RIGHT_MARGIN ? 'flex-row' : 'flex-row-reverse' } items-end">
  <div class="w-[28%] h-[97%]"
       class:ml-[2vh]={!$IS_RIGHT_MARGIN}
       class:mr-[2vh]={$IS_RIGHT_MARGIN}
      >
    {#each notifications.slice().reverse() as dispatch, index (dispatch.data.id)}
      <div class="w-full h-fit my-[0.5vh] {dispatch.data.priority == 1 ? 'border-l-4 emergency-overlay' : 'border-l-4 border-blue-500'} {dispatch.data.priority == 1 ? 'emergency-overlay' : 'bg-slate-900 hover:bg-slate-800'} transition-colors duration-200" transition:fly="{{ x: $IS_RIGHT_MARGIN ? 400 : -400 }}">
        <!-- HEADER -->
        <div class="{dispatch.data.priority == 1 ? 'emergency-overlay' : 'bg-slate-800'} p-[1.5vh] border-b {dispatch.data.priority == 1 ? 'emergency-overlay' : 'border-slate-700'}">
          <div class="flex items-center gap-[1vh]">
            <!-- CALL ID -->
            <div class="px-[1vh] py-[0.3vh] bg-blue-600 text-white font-bold text-[1.2vh] font-mono rounded-lg">
              #{dispatch.data.id}
            </div>
            <!-- CODE -->
            <div class="px-[1vh] py-[0.3vh] {dispatch.data.priority == 1 ? 'bg-red-600' : 'bg-slate-700'} text-white font-semibold text-[1.2vh] rounded-lg">
              {dispatch.data.code}
            </div>
            <!-- MESSAGE -->
            <div class="flex-1">
              <h3 class="text-white text-[1.4vh] font-medium">{dispatch.data.message}</h3>
            </div>
            <!-- STATUS ICON -->
            <div class="{dispatch.data.priority == 1 ? 'text-red-400' : 'text-blue-400'} text-[1.4vh]">
              <i class="{dispatch.data.icon}"></i>
            </div>
          </div>
        </div>
        
        <!-- CONTENT -->
        <div class="flex">
          <div class="flex flex-col p-[1.5vh] gap-y-[0.5vh] text-[1.1vh] w-[70%]">
              {#if dispatch.data}
                {#each getDispatchData(dispatch) as field}
                  {#if field.value}
                    <div class="flex items-center gap-[0.5vh] text-slate-400">
                      <i class="{field.icon} text-blue-400 text-[1vh] w-[1.2vh]"></i>
                      <span class="text-slate-500 min-w-[6vh]">{field.label}:</span>
                      <span class="text-slate-300">{field.value}</span>
                    </div>
                  {/if}
                {/each}
              {/if}
          </div>
          <div class="w-[30%] flex items-end justify-center p-[1.5vh]">
            {#if index === 0}
              <div class="bg-blue-700 hover:bg-blue-600 text-white px-[1.5vh] py-[0.6vh] text-[1.2vh] font-medium transition-colors duration-200">
                [{$RESPOND_KEYBIND}] Respond
              </div>
            {/if}
          </div>
        </div>
      </div>
    {/each}
  </div>
</div>