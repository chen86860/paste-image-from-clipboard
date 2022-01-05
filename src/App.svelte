<script lang="ts">
  export let value = "";
  export let images: string[] = [];

  const handlePaste = async (event: ClipboardEvent) => {
    const items: FileList = event.clipboardData.files;
    if (!items) return;

    images = await Promise.all(
      Array.from(items)
        .filter((item) => item.type.startsWith("image"))
        .map(async (item) => {
          const stream = item.stream();
          const response = new Response(stream);
          const blob = await response.blob();

          return URL.createObjectURL(blob);
        })
    );
    console.log({ images });
  };

  window.document.addEventListener("paste", handlePaste);

  const handleChange = (
    event: Event & {
      currentTarget: EventTarget & HTMLTextAreaElement;
    }
  ) => {
    value = event.currentTarget.value;
  };
</script>

<main>
  <h1>Past Image Here!</h1>
  <div>
    <textarea name="" on:change={(event) => handleChange(event)} {value} id="" cols="30" rows="10" />
    <ul class="list">
      {#each images as item}
        <li>
          <img src={item} alt="" />
        </li>
      {/each}
    </ul>
  </div>
</main>

<style>
  main {
    text-align: center;
    padding: 1em;
    max-width: 240px;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4em;
    font-weight: 100;
  }

  @media (min-width: 640px) {
    main {
      max-width: none;
    }
  }

  .list {
    display: flex;
  }

  .list img {
    width: 200px;
  }
</style>
