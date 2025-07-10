<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';
  import AddPlaceForm from '$lib/components/AddPlaceForm.svelte';

  interface Place {
    id: string;
    created_at: string;
    name: string;
    location: string | null;
    description: string | null;
    visited: boolean;
  }

  let places: Place[] = [];
  let loading = true;
  let error: string | null = null;
  let updatingStatusFor: string | null = null; // To track which place is being updated
  let deletingPlaceId: string | null = null; // To track which place is being deleted

  async function fetchPlaces() {
    loading = true;
    error = null;
    try {
      const { data, error: supabaseError } = await supabase
        .from('places')
        .select('*')
        .order('created_at', { ascending: false });

      if (supabaseError) {
        throw supabaseError;
      }
      if (data) {
        places = data;
      }
    } catch (e: any) {
      error = e.message || 'Failed to fetch places.';
      console.error(e);
    } finally {
      loading = false;
    }
  }

  onMount(() => {
    fetchPlaces();
  });

  // When a new place is added by the form, prepend it to the list
  function handlePlaceAdded(event: CustomEvent<Place>) {
    const newPlace = event.detail;
    places = [newPlace, ...places];
    // Or, alternatively, refetch all places:
    // fetchPlaces();
    // Prepending is more optimistic and feels faster if the insert was successful.
  }

  async function toggleVisitedStatus(place: Place) {
    if (updatingStatusFor === place.id) return; // Prevent multiple clicks

    const originalStatus = place.visited;
    updatingStatusFor = place.id;

    // Optimistic UI update
    places = places.map(p => p.id === place.id ? { ...p, visited: !p.visited } : p);

    try {
      const { error: updateError } = await supabase
        .from('places')
        .update({ visited: !originalStatus })
        .eq('id', place.id);

      if (updateError) {
        throw updateError;
      }
      // Success, UI is already updated optimistically
    } catch (e: any) {
      console.error('Error updating visited status:', e);
      // Revert UI on error
      places = places.map(p => p.id === place.id ? { ...p, visited: originalStatus } : p);
      alert(`Failed to update status for ${place.name}: ${e.message}`);
    } finally {
      updatingStatusFor = null;
    }
  }

  async function deletePlace(placeToDelete: Place) {
    if (deletingPlaceId === placeToDelete.id) return; // Prevent multiple clicks

    // Optional: Add a confirmation dialog
    if (!confirm(`Are you sure you want to delete "${placeToDelete.name}"? This action cannot be undone.`)) {
      return;
    }

    deletingPlaceId = placeToDelete.id;
    const originalPlaces = [...places]; // Store for potential rollback

    // Optimistic UI update: remove the place from the list
    places = places.filter(p => p.id !== placeToDelete.id);

    try {
      const { error: deleteError } = await supabase
        .from('places')
        .delete()
        .eq('id', placeToDelete.id);

      if (deleteError) {
        throw deleteError;
      }
      // Success, UI is already updated
    } catch (e: any) {
      console.error('Error deleting place:', e);
      // Revert UI on error
      places = originalPlaces;
      alert(`Failed to delete ${placeToDelete.name}: ${e.message}`);
    } finally {
      deletingPlaceId = null;
    }
  }
</script>

<svelte:head>
  <title>Places to Visit</title>
  <meta name="description" content="A list of wonderful places to visit around the world." />
</svelte:head>

<div class="container">
  <h1>Places to Visit</h1>

  <AddPlaceForm on:placeAdded={handlePlaceAdded} />

  {#if loading && places.length === 0} <!-- Show loading only if places aren't already displayed -->
    <p>Loading places...</p>
  {:else if error}
    <p class="error-message">Error: {error}</p>
    <button on:click={fetchPlaces}>Try Again</button>
  {:else if places.length === 0}
    <p>No places added yet. Be the first to add one!</p>
  {:else}
    <ul class="places-list">
      {#each places as place (place.id)}
        <li class="place-item" class:visited={place.visited}>
          <h2>{place.name}</h2>
          {#if place.location}
            <p class="location">📍 {place.location}</p>
          {/if}
          {#if place.description}
            <p class="description">{place.description}</p>
          {/if}
          <div class="status">
            {place.visited ? 'Visited ✔' : 'Not Visited ⏳'}
          </div>
          <div class="actions">
            <button
              class="toggle-visited-btn"
              on:click={() => toggleVisitedStatus(place)}
              disabled={updatingStatusFor === place.id || !!deletingPlaceId}
            >
              {updatingStatusFor === place.id ? 'Updating...' : (place.visited ? 'Mark as Not Visited' : 'Mark as Visited')}
            </button>
            <button
              class="delete-btn"
              on:click={() => deletePlace(place)}
              disabled={deletingPlaceId === place.id || !!updatingStatusFor}
            >
              {deletingPlaceId === place.id ? 'Deleting...' : 'Delete'}
            </button>
          </div>
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style>
  .container {
    max-width: 800px;
    margin: 2rem auto;
    padding: 1rem;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
      Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    color: #333;
  }

  h1 {
    text-align: center;
    color: #2c3e50;
    margin-bottom: 2rem;
    font-weight: 300;
  }

  .error-message {
    color: #e74c3c;
    text-align: center;
    margin-bottom: 1rem;
  }

  .places-list {
    list-style: none;
    padding: 0;
  }

  .place-item {
    background-color: #fff;
    border: 1px solid #ecf0f1;
    border-radius: 8px;
    padding: 1.5rem;
    margin-bottom: 1rem;
    box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
    transition: transform 0.2s ease-in-out;
  }

  .place-item:hover {
    transform: translateY(-3px);
  }

  .place-item.visited {
    border-left: 5px solid #2ecc71; /* Green border for visited places */
  }

  .place-item h2 {
    margin-top: 0;
    margin-bottom: 0.5rem;
    color: #34495e;
    font-weight: 500;
  }

  .place-item .location {
    font-style: italic;
    color: #7f8c8d;
    margin-bottom: 0.5rem;
  }

  .place-item .description {
    color: #555;
    margin-bottom: 1rem;
    line-height: 1.6;
  }

  .place-item .status {
    font-size: 0.9em;
    color: #95a5a6;
    text-align: right;
  }

  .place-item.visited .status {
    color: #27ae60;
  }

  button {
    display: block;
    margin: 1rem auto;
    padding: 0.75rem 1.5rem;
    background-color: #3498db;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1rem;
    transition: background-color 0.2s ease;
  }

  button:hover {
    background-color: #2980b9;
  }

  .place-item .actions {
    margin-top: 1rem;
    display: flex;
    gap: 0.5rem; /* Add some space between buttons */
    justify-content: flex-end; /* Align buttons to the right */
  }

  .toggle-visited-btn, .delete-btn { /* Apply common styles */
    padding: 0.5rem 1rem;
    font-size: 0.9rem;
    background-color: var(--secondary-color, #2ecc71); /* Green for positive action */
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    transition: background-color 0.2s ease;
  }

  .place-item.visited .toggle-visited-btn {
    background-color: var(--dark-gray, #7f8c8d); /* Gray for "undo" action */
  }

  .toggle-visited-btn:hover:not(:disabled) {
    opacity: 0.85;
  }

  .toggle-visited-btn:disabled, .delete-btn:disabled {
    background-color: #bdc3c7;
    cursor: not-allowed;
  }

  .delete-btn {
    background-color: #e74c3c; /* Red for destructive action */
  }

  .delete-btn:hover:not(:disabled) {
    background-color: #c0392b; /* Darker red */
  }

</style>
