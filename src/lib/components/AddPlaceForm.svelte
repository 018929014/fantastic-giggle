<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { createEventDispatcher } from 'svelte';

  const dispatch = createEventDispatcher();

  let name = '';
  let location = '';
  let description = '';
  let adding = false;
  let errorMessage: string | null = null;
  let successMessage: string | null = null;

  async function handleSubmit() {
    if (!name.trim()) {
      errorMessage = 'Name is required.';
      return;
    }

    adding = true;
    errorMessage = null;
    successMessage = null;

    try {
      const { data, error } = await supabase
        .from('places')
        .insert([{ name, location, description }])
        .select(); // .select() will return the inserted rows

      if (error) {
        throw error;
      }

      if (data) {
        successMessage = `Successfully added "${data[0].name}"!`;
        // Reset form
        name = '';
        location = '';
        description = '';
        // Notify parent component that a place was added
        dispatch('placeAdded', data[0]);
      }
    } catch (e: any) {
      console.error('Error adding place:', e);
      errorMessage = e.message || 'Failed to add place. Please try again.';
    } finally {
      adding = false;
      // Clear messages after a few seconds
      setTimeout(() => {
        successMessage = null;
        errorMessage = null;
      }, 4000);
    }
  }
</script>

<form on:submit|preventDefault={handleSubmit} class="add-place-form">
  <h3>Add a New Place</h3>

  {#if errorMessage}
    <p class="message error">{errorMessage}</p>
  {/if}
  {#if successMessage}
    <p class="message success">{successMessage}</p>
  {/if}

  <div class="form-group">
    <label for="name">Name*</label>
    <input type="text" id="name" bind:value={name} required disabled={adding} />
  </div>

  <div class="form-group">
    <label for="location">Location</label>
    <input type="text" id="location" bind:value={location} disabled={adding} />
  </div>

  <div class="form-group">
    <label for="description">Description</label>
    <textarea id="description" bind:value={description} rows="3" disabled={adding}></textarea>
  </div>

  <button type="submit" disabled={adding}>
    {adding ? 'Adding...' : 'Add Place'}
  </button>
</form>

<style>
  .add-place-form {
    background-color: #ffffff;
    padding: 1.5rem 2rem;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
    margin-top: 2rem;
    margin-bottom: 2rem;
  }

  .add-place-form h3 {
    text-align: center;
    color: var(--text-color, #333);
    margin-top: 0;
    margin-bottom: 1.5rem;
    font-weight: 500;
  }

  .form-group {
    margin-bottom: 1rem;
  }

  .form-group label {
    display: block;
    margin-bottom: 0.5rem;
    color: #555;
    font-size: 0.9rem;
    font-weight: 500;
  }

  .form-group input[type="text"],
  .form-group textarea {
    width: 100%;
    padding: 0.75rem;
    border: 1px solid var(--light-gray, #ecf0f1);
    border-radius: 4px;
    box-sizing: border-box;
    font-size: 1rem;
    color: var(--text-color, #333);
    background-color: #fdfdfd;
  }

  .form-group input[type="text"]:focus,
  .form-group textarea:focus {
    outline: none;
    border-color: var(--primary-color, #3498db);
    box-shadow: 0 0 0 2px rgba(52, 152, 219, 0.2);
  }

  .form-group textarea {
    resize: vertical;
    min-height: 80px;
  }

  button {
    display: block;
    width: 100%;
    padding: 0.8rem 1.5rem;
    background-color: var(--primary-color, #3498db);
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-size: 1.1rem;
    font-weight: 500;
    transition: background-color 0.2s ease;
  }

  button:disabled {
    background-color: #bdc3c7;
    cursor: not-allowed;
  }

  button:not(:disabled):hover {
    background-color: #2980b9; /* Darken primary color */
  }

  .message {
    padding: 0.75rem;
    margin-bottom: 1rem;
    border-radius: 4px;
    font-size: 0.95rem;
    text-align: center;
  }

  .message.error {
    background-color: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
  }

  .message.success {
    background-color: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
  }
</style>
