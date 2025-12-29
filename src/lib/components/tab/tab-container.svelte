<script lang="ts">
  import { Terminal } from "$lib";

  // contains info needed to render tabs
  const TAB_CONFIG = {
    "test": {
      component: null,
      props: {},
      title: "Test"
    },
    "terminal": {
      component: Terminal,
      props: {},
      title: "Terminal"
    }
  };

  const TABS = [
    { id: "test", type: "test", title: "Test" },
    { id: "terminal", type: "terminal", title: "Terminal"}
  ]
  
  type TabName = keyof typeof TAB_CONFIG;
  let activeTab: TabName = "test";
  
  function switchTab(tabName: TabName) {
    activeTab = tabName;
  }
</script>

<div class="h-full w-full flex flex-col">
  <!-- Tab Headers -->
  <div class="flex border-b border-b-blue-950">
    {#each TABS as tab}
          <button
        class="px-4 py-2 text-sm font-medium transition-colors duration-200 {activeTab === tab.type 
          ? 'bg-blue-600 text-white border-b-2 border-blue-400' 
          : 'bg-cyan-900 text-gray-300 hover:bg-indigo-800 hover:text-white'}"
        on:click={() => switchTab(tab.type as TabName)}
      >
        {tab.title}
      </button>
    {/each}
  </div>
  
  <!-- Tab Content -->
  <div class="flex-1 p-4 overflow-auto">
    {#if TAB_CONFIG[activeTab].component}
      <!-- Dynamic component rendering -->
      <svelte:component this={TAB_CONFIG[activeTab].component} {...TAB_CONFIG[activeTab].props} />
    {:else}
      <!-- Fallback content for tabs without components (like test tab) -->
      <div class="text-white">
        <h2 class="text-xl mb-4">{TAB_CONFIG[activeTab].title} Tab Content</h2>
        <p>This is the {activeTab} tab content. You can add any content here or assign a component.</p>
      </div>
    {/if}
  </div>
</div>