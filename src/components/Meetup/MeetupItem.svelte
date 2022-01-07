<script>
  import meetups from '../../meetups-store';
  import Button from '../UI/Button.svelte';
  import Badge from '../UI/Badge.svelte';
  import Error from '../UI/Error.svelte';
  import { createEventDispatcher } from 'svelte';

  export let id;
  export let title;
  export let subtitle;
  export let description;
  export let imageURL;
  export let address;
  export let isFav;

  let error;

  let isLoading = false;

  let dispatch = createEventDispatcher();

  function toggleFavorite() {
    isLoading = true;
    fetch(
      `https://meetups-e2e4b-default-rtdb.firebaseio.com/meetups/${id}.json`,
      {
        method: 'PATCH',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ isFavorite: !isFav }),
      }
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error('Failed!');
        }

        meetups.toggleFavorite(id);
        isLoading = false;
      })
      .catch((err) => {
        error = err;
        isLoading = false;
        console.log(err);
      });
  }

  function clearError() {
    error = null;
  }
</script>

{#if error}
  <Error message={error.message} on:cancel={clearError} />
{/if}

<article>
  <header>
    <h1>
      {title}
      {#if isFav} <Badge>FAVORITE</Badge> {/if}
    </h1>
    <h2>{subtitle}</h2>
    <p>{address}</p>
  </header>
  <div class="image">
    <img src={imageURL} alt="" />
  </div>
  <div class="content">
    <p>{description}</p>
  </div>
  <footer>
    <Button type="button" mode="outline" on:click={() => dispatch('edit', id)}
      >Edit</Button
    >
    {#if isLoading}
      <span>Changing...</span>
    {:else}
      <Button
        mode="outline"
        type="button"
        color={isFav ? undefined : 'success'}
        on:click={toggleFavorite}>{isFav ? 'Unfavorite' : 'Favorite'}</Button
      >
    {/if}
    <Button type="button" href="/{id}">Show Details</Button>
  </footer>
</article>

<style>
  article {
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
    border-radius: 5px;
    background: white;
    margin: 1rem;
  }

  header,
  .content,
  footer {
    padding: 1rem;
  }

  .image {
    width: 100%;
    height: 14rem;
  }

  .image img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  h1 {
    font-size: 1.25rem;
    margin: 0.5rem 0;
    font-family: 'Roboto Slab', sans-serif;
  }

  h1.is-favorite {
    background: #01a129;
    color: white;
    padding: 0 0.5rem;
    border-radius: 5px;
  }

  h2 {
    font-size: 1rem;
    color: #808080;
    margin: 0.5rem 0;
  }

  p {
    font-size: 1.25rem;
    margin: 0;
  }

  div {
    text-align: right;
  }

  .content {
    height: 4rem;
  }
</style>
