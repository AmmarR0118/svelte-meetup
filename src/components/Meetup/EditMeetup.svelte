<script>
  import meetups from '../../meetups-store';
  import { createEventDispatcher } from 'svelte';
  import { isEmpty, isValidEmail } from '../../helpers/validation';
  import Modal from '../UI/Modal.svelte';
  import Button from '../UI/Button.svelte';
  import TextInput from '../UI/TextInput.svelte';
  import Error from '../UI/Error.svelte';

  export let id;

  let title = '';
  let subtitle = '';
  let address = '';
  let imageURL = '';
  let email = '';
  let description = '';
  let error;

  if (id) {
    const unsuscribe = meetups.subscribe((items) => {
      const selectedMeetup = items.find((i) => i.id === id);

      title = selectedMeetup.title;
      subtitle = selectedMeetup.subtitle;
      address = selectedMeetup.address;
      imageURL = selectedMeetup.imageURL;
      email = selectedMeetup.contactEmail;
      description = selectedMeetup.description;
    });

    unsuscribe();
  }

  let dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: addressValid = !isEmpty(address);
  $: imageURLValid = !isEmpty(imageURL);
  $: descriptionValid = !isEmpty(description);
  $: emailValid = isValidEmail(email);
  $: isFormValid =
    titleValid &&
    subtitleValid &&
    addressValid &&
    imageURLValid &&
    descriptionValid &&
    emailValid;

  function submitForm() {
    const meetupData = {
      title: title,
      subtitle: subtitle,
      description: description,
      imageURL: imageURL,
      address: address,
      contactEmail: email,
    };

    if (id) {
      fetch(
        `https://meetups-e2e4b-default-rtdb.firebaseio.com/meetups/${id}.json`,
        {
          method: 'PATCH',
          headers: {
            'Content-Type': 'application/json',
          },
          body: JSON.stringify(meetupData),
        }
      )
        .then((res) => {
          if (!res.ok) {
            throw new Error('Failed!');
          }

          meetups.updateMeetup(id, meetupData);
        })
        .catch((err) => {
          error = err;
          console.log(err);
        });
    } else {
      fetch('https://meetups-e2e4b-default-rtdb.firebaseio.com/meetups.json', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ ...meetupData, isFavorite: false }),
      })
        .then((res) => {
          if (!res.ok) {
            throw new Error('Failed!');
          }

          return res.json();
        })
        .then((data) => {
          meetups.addMeetup({
            ...meetupData,
            isFavorite: false,
            id: data.name,
          });
        })
        .catch((err) => {
          error = err;
          console.log(err);
        });
    }
    dispatch('save');
  }

  function cancel() {
    dispatch('cancel');
  }

  function deleteMeetup() {
    fetch(
      `https://meetups-e2e4b-default-rtdb.firebaseio.com/meetups/${id}.json`,
      { method: 'DELETE' }
    )
      .then((res) => {
        if (!res.ok) {
          throw new Error('Failed!');
        }

        meetups.deleteMeetup(id);
      })
      .catch((err) => {
        error = err;
        console.log(err);
      });

    dispatch('save');
  }

  function clearError() {
    error = null;
  }
</script>

{#if error}
  <Error message={error.message} />
{/if}

<Modal title="Edit Meetup Data" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Title"
      value={title}
      valid={titleValid}
      validityMessage="Please enter a valid title."
      on:input={(e) => (title = e.target.value)}
    />
    <TextInput
      id="subtitle"
      label="Sub Title"
      value={subtitle}
      valid={subtitleValid}
      validityMessage="Please enter a valid subtitle."
      on:input={(e) => (subtitle = e.target.value)}
    />
    <TextInput
      id="address"
      label="Address"
      value={address}
      valid={addressValid}
      validityMessage="Please enter a valid address."
      on:input={(e) => (address = e.target.value)}
    />
    <TextInput
      id="imageURL"
      label="Image URL"
      value={imageURL}
      valid={imageURLValid}
      validityMessage="Please enter a valid image url."
      on:input={(e) => (imageURL = e.target.value)}
    />
    <TextInput
      id="email"
      label="E-Mail"
      type="email"
      value={email}
      valid={emailValid}
      validityMessage="Please enter a valid email address."
      on:input={(e) => (email = e.target.value)}
    />
    <TextInput
      id="description"
      label="Description"
      controlType="textarea"
      bind:value={description}
      valid={descriptionValid}
      validityMessage="Please enter a valid description."
    />
  </form>

  <div slot="footer">
    <Button mode="outline" type="button" on:click={cancel}>Cancel</Button>
    <Button type="button" on:click={submitForm} disabled={!isFormValid}
      >Save</Button
    >
    {#if id}
      <Button type="button" on:click={deleteMeetup}>Delete</Button>
    {/if}
  </div>
</Modal>

<style>
  form {
    width: 100%;
  }
</style>
