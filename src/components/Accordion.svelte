<script>
  export let title;
  export let open = false;
</script>

<div class="accordion">
  <button
    class="accordion-button"
    on:click={() => {
      open = !open;
    }}
    ><h3>{title}</h3>
    <span class={`caret${open ? " open" : ""}`} /></button
  >
  <hr />
  <div class={`accordion-text${open ? " open" : ""}`}><div><slot /></div></div>
</div>

<style>
  .accordion-button {
    background-color: transparent;
    border: none;
    border-radius: 0;
    color: black;
    padding: 1rem;
    position: relative;
    text-transform: none;
    text-align: left;
    width: 100%;
  }
  hr {
    background-color: black;
    border: none;
    height: 1px;
    margin: 0;
    width: 95%;
  }
  .accordion-text {
    max-height: 0;
    overflow: hidden;
    transition: max-height 0.3s ease-out;
  }

  .accordion-text div {
    padding: 1rem 1rem;
  }

  .accordion-text.open {
    max-height: 100vh;
  }

  .caret {
    top: 40%;
    right: 10%;
    display: block;
    border-top: solid 1px black;
    border-left: solid 1px black;
    position: absolute;
    width: 0.75rem;
    height: 0.75rem;
    transform: rotate(45deg);
    transform-origin: center;
    transition: transform 0.3s ease-out;
  }

  .caret.open {
    transform: rotate(225deg);
  }
</style>
