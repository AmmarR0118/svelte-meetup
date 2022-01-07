<script>
  import { slide } from 'svelte/transition';
</script>

<span transition:slide>
  <slot />
</span>

<style>
  span {
    display: inline-block;
    margin: 0 0.25rem;
    padding: 0 0.5rem;
    background-color: #cf0056;
    border: 1px solid #cf0056;
    border-radius: 3px;
    color: white;
    font-size: 0.8rem;
    font-family: 'Lato', sans-serif;
  }
</style>
