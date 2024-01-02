<script>
  import {onMount} from "svelte"
  export let noteId;
  let noteContent = '';

  // Load note content from local storage on component mount
  onMount(() => {
    const savedNote = localStorage.getItem(`note_${noteId}`);
    if (savedNote) {
      noteContent = savedNote;
    }
  });

  // Save note content to local storage whenever it changes
  $: if (noteContent) {
    localStorage.setItem(`note_${noteId}`, noteContent);
  }
</script>
  
  <style>
    textarea {
      width: 97%;
      background-color: rgb(214, 251, 251);
      border-radius: 25px;
      font-family: ProductSans;
      padding: 10px;
      font-size: 1.4rem;
      min-height: 70vh;
    }

    p {
      font-size: 3vh;
      text-align: center;
    }
  </style>
  
  <div>
    <p>{noteId}</p>
    <textarea bind:value={noteContent} placeholder="Scrivi qui..."></textarea>
  </div>