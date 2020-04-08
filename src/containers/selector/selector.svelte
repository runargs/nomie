<script>
  // components
  import NText from "../../components/text/text.svelte";
  import NModal from "../../components/modal/modal.svelte";
  import NItem from "../../components/list-item/list-item.svelte";
  import NIcon from "../../components/icon/icon.svelte";
  import NToolbarGrid from "../../components/toolbar/toolbar-grid.svelte";
  import Dymoji from "../../components/dymoji/dymoji.svelte";

  //Utils
  import { createEventDispatcher } from "svelte";
  import { fade } from "svelte/transition";

  // Stores
  import { TrackerStore } from "../../store/trackers";
  import { PeopleStore } from "../../store/people-store";
  import { Interact } from "../../store/interact";
  import { ContextStore } from "../../store/context-store";
  import { Lang } from "../../store/lang";

  // Props
  export let show = false;
  export let multiple = false;
  // export let multiple = false;

  // Consts
  const dispatch = createEventDispatcher();

  // State
  let state = {
    selected: [],
    items: [],
    multiple
  };

  // Holder of the alphabet for the list
  let alphaGroup = {};

  // When tracker store loads. Turn trackers into array sorted by label
  $: type = $Interact.selector.type;

  $: if ($Interact.selector.show) {
    state.selected = [];
    alphaGroup = {};
    console.log("Show the selector! (me)");
    console.log("set type to ", $Interact.selector.type);
    switch ($Interact.selector.type) {
      case "tracker":
        state.title = "Select a Tracker";
        state.items = Object.keys($TrackerStore || {})
          .map(tag => {
            return $TrackerStore[tag];
          })
          .sort((a, b) => {
            return a.label > b.label ? 1 : -1;
          });
        break;

      case "person":
        state.title = "Select a Person";
        console.log("$PeopleStore.people", $PeopleStore.people);
        state.items = Object.keys($PeopleStore.people || {})
          .map(username => {
            return $PeopleStore.people[username];
          })
          .sort((a, b) => {
            return a.username > b.username ? 1 : -1;
          });
        break;

      case "context":
        state.title = "Select Context";
        console.log("$PeopleStore.people", $ContextStore);
        state.items = Object.keys($ContextStore || {})
          .map(tag => {
            return $ContextStore[tag];
          })
          .sort((a, b) => {
            return a > b ? 1 : -1;
          });
        break;
    }
  }

  //   $: state.items = Object.keys($TrackerStore || {})
  //     .map(tag => {
  //       return $TrackerStore[tag];
  //     })
  //     .sort((a, b) => {
  //       return a.label.substr(0, 1).toLowerCase() >=
  //         b.label.substr(0, 1).toLowerCase()
  //         ? 1
  //         : -1;
  //     });

  // When selected, auto create an array of selected trackers
  $: state.selectedArray = Object.keys(state.selected).map(tag => {
    alphaGroup = {};
    return state.selected[tag];
  });

  // If show changes, set selected to notihng

  // Methods
  const methods = {
    toggle(item) {
      if (multiple) {
        let index = state.selected.indexOf(item);
        if (index > -1) {
          // unselect
          state.selected.splice(index, 1);
        } else {
          state.selected.push(item);
        }
      } else {
        state.selected = [item];
      }
    },
    close() {
      dispatch("cancel");
    },
    // Check if a letter has been shown
    alphaGroupExists(item) {
      //   if (state.items.length > 10) {
      //     // get first letter
      //     let alpha = tracker.label.substr(0, 1).toLowerCase();
      //     // If it has value - return true...
      //     if (alphaGroup.hasOwnProperty(alpha)) {
      //       return true;
      //     } else {
      //       // Else - populate the alphaGroup, then return false
      //       alphaGroup[alpha] = true;
      //       return false;
      //     }
      //   } else {
      //     // if it's less than 10 trackers - just show them without the letters
      //     return true;
      //   }
    }
  };
</script>

<style lang="scss">
  :global(.tracker-selector-modal .sticky-top) {
    position: sticky;
    top: 0px;
  }
</style>

{#if show}
  <NModal
    title={state.title}
    type="fullscreen"
    className="tracker-selector-modal"
    allowClose={true}
    on:close={methods.close}>

    {#if state.items.length == 0}
      <NItem class="text-inverse-2">Nothing found</NItem>
    {/if}

    {#if type == 'tracker'}
      <div class="list">
        {#each state.items as item}
          <NItem
            className="bottom-line {state.selected.hasOwnProperty(item) ? 'bg-selected' : ''}"
            title={item.label}
            on:click={() => {
              methods.toggle(item);
            }}>
            <span slot="left">
              <NText size="lg">{item.emoji}</NText>
            </span>
            <span slot="right">
              {#if state.selected.indexOf(item) > -1}
                <NIcon name="checkmarkFilled" className="fill-primary-bright" />
              {/if}
            </span>
          </NItem>
        {/each}
      </div>
    {:else if type == 'person'}
      <!-- It's a person list -->
      <div class="list">
        {#each state.items as person}
          <NItem
            className="bottom-line {state.selected.indexOf(person) > -1 ? 'bg-selected' : ''}"
            title={person.displayName}
            on:click={() => {
              methods.toggle(person);
            }}>
            <span slot="left">
              <Dymoji username={person.username} size="24" />
            </span>
            <span slot="right">
              {#if state.selected.indexOf(person) > -1}
                <NIcon name="checkmarkFilled" className="fill-primary-bright" />
              {/if}
            </span>
          </NItem>
        {/each}
      </div>
    {:else if type == 'context'}
      <div class="list">
        {#each state.items as context}
          <NItem
            className="bottom-line {state.selected.indexOf(context) > -1 ? 'bg-selected' : ''}"
            title={'+' + context}
            on:click={() => {
              methods.toggle(context);
            }}>
            <span slot="right">
              {#if state.selected.indexOf(context) > -1}
                <NIcon name="checkmarkFilled" className="fill-primary-bright" />
              {/if}
            </span>
          </NItem>
        {/each}
      </div>
    {/if}
    <div slot="footer" class="n-row">
      <button class="btn btn-light btn-lg w-100 mr-2" on:click={methods.close}>
        {Lang.t('general.close')}
      </button>
      {#if state.selectedArray.length > 0}
        <button
          transition:fade
          class="btn btn-primary btn-lg w-100"
          on:click={() => {
            dispatch('select', state.selectedArray);
          }}>
          Select
        </button>
      {/if}
    </div>
  </NModal>
{/if}