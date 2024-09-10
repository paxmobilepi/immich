<script lang="ts">
  import { quintOut } from 'svelte/easing';
  import { slide } from 'svelte/transition';
  import { clickOutside } from '$lib/actions/click-outside';

  export let isVisible: boolean = false;
  export let direction: 'left' | 'right' = 'right';
  export let x = 0;
  export let y = 0;
  export let id: string | undefined = undefined;
  export let ariaLabel: string | undefined = undefined;
  export let ariaLabelledBy: string | undefined = undefined;
  export let ariaActiveDescendant: string | undefined = undefined;

  export let menuElement: HTMLUListElement | undefined = undefined;
  export let onClose: (() => void) | undefined = undefined;

  let left: number;
  let top: number;

  // We need to bind clientHeight since the bounding box may return a height
  // of zero when starting the 'slide' animation.
  let height: number;

  $: {
    if (menuElement) {
      const rect = menuElement.getBoundingClientRect();
      const directionWidth = direction === 'left' ? rect.width : 0;
      const menuHeight = Math.min(menuElement.clientHeight, height) || 0;

      const calcLeft = Math.min(window.innerWidth - rect.width, x - directionWidth);
      left = Math.max(0, calcLeft);
      top = Math.min(window.innerHeight - menuHeight, y);
    }
  }
</script>

<div
  bind:clientHeight={height}
  class="fixed z-10 overflow-hidden rounded-lg duration-[250ms] ease-in {isVisible
    ? 'shadow-lg transition-shadow'
    : 'shadow-none transition-none'}"
  class:shadow-none={!isVisible}
  class:shadow-lg={isVisible}
  class:transition-none={!isVisible}
  style:left="{left}px"
  style:top="{top}px"
  transition:slide={{ duration: 250, easing: quintOut }}
  use:clickOutside={{ onOutclick: onClose }}
>
  <ul
    {id}
    aria-activedescendant={ariaActiveDescendant ?? ''}
    aria-label={ariaLabel}
    aria-labelledby={ariaLabelledBy}
    bind:this={menuElement}
    class="flex flex-col transition-all duration-[250ms] ease-in-out outline-none overflow-y-auto immich-scrollbar bg-slate-100 relative min-w-[200px] max-w-[200px] sm:max-w-[256px] rounded-lg {isVisible
      ? `${direction === 'left' ? 'right-0' : 'left-0'} max-h-dvh`
      : `${direction === 'left' ? 'right-[-120px]' : 'left-[-120px]'} max-h-0`}"
    role="menu"
    tabindex="-1"
  >
    <slot />
  </ul>
</div>
