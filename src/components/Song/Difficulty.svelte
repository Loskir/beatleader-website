<script>
  import {getHumanDiffInfo, getIconNameForDiff} from '../../utils/scoresaber/format'
  import Value from '../Common/Value.svelte'

  export let diff;
  export let useShortName = false;
  export let reverseColors = false;
  export let stars = null;
  export let starsSuffix = "*"
  export let enabled = true;
  export let pointer = false;
  export let showDiffIcons = false;

  $: diffColor = enabled ? diffInfo.color : "gray";
  $: diffInfo = diff ? getHumanDiffInfo(diff) : null;
  $: title = useShortName && diffInfo.type !== 'Standard' ? diffInfo.name: diffInfo.fullName;
</script>

{#if diffInfo}
  <span class="{'diff ' + (reverseColors ? 'reversed' : '')}"
        style="color: {reverseColors ? 'white' : diffColor}; background-color: {reverseColors ? diffColor : 'transparent'}; {pointer ? "cursor: pointer !important": ""}"
        {title} on:click>
    {#if showDiffIcons}
    <span class="icon">
      <div class="{getIconNameForDiff(diffInfo)}" title="{diffInfo.type}"></div>
    </span>
      
    {/if}

    {#if stars}
      <Value value={stars} suffix={starsSuffix} zero="" {title}/>
    {:else}
      {useShortName ? diffInfo.shortName : diffInfo.fullName}
    {/if}
  </span>
{/if}

<style>
    .diff {
        display: inline-block;
    }

    .reversed {
        font-weight: 600;
        padding: 0 2px;
        min-width: 1.5em;
        max-height: 1.5em;
        border-radius: 2px;
    }

    .icon {
      height: 1rem !important;
      width: 1rem !important;
    }
</style>