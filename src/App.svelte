<script lang="ts">
  import { onMount } from "svelte";

  export let content = "";
  export let images: string[] = [];
  export let fileUpload: HTMLInputElement;
  export let textarea: HTMLTextAreaElement;

  const handleImageRemove = (removeIndex: number) => {
    const newImages = images.filter((_, index) => index !== removeIndex);
    images = newImages;
  };

  const handleFileProcessing = async (files: FileList) => {
    if (images.length >= 6) {
      alert("Max number of images to process is 6!");
      return;
    }
    if (!files) return;

    images = images.concat(
      await Promise.all(
        Array.from(files)
          .filter((item) => item.type.startsWith("image"))
          .map(async (item) => {
            const stream = item.stream();
            const response = new Response(stream);
            const blob = await response.blob();

            return URL.createObjectURL(blob);
          })
      )
    );
  };

  const handlePaste = async (event: ClipboardEvent) => {
    console.log({ event: event });
    if (!(document.activeElement === textarea && textarea.contains(event.target))) {
      console.log("exit");
      return;
    }
    if (images.length > 9) return;
    const items: FileList = event.clipboardData.files;

    if (!items) return;
    handleFileProcessing(items);
  };

  const handleFileUpload = (event) => {
    const files: FileList = event.target.files;
    handleFileProcessing(files);
  };

  const handleChange = (
    event: Event & {
      currentTarget: EventTarget & HTMLTextAreaElement;
    }
  ) => {
    content = event.currentTarget.value;
  };

  onMount(() => {
    window.document.addEventListener("paste", handlePaste);

    return () => window.document.removeEventListener("paste", handlePaste);
  });
</script>

<main>
  <h1>Share You Ideas</h1>
  <div class="container">
    <textarea
      placeholder="Share your ideas here..."
      value={content}
      cols="60"
      rows="10"
      on:change={(event) => handleChange(event)}
      bind:this={textarea}
    />

    <ul class="list">
      {#each images as item, index}
        <li class="item">
          <img src={item} alt="" />
          <button class="item-close" on:click={() => handleImageRemove(index)}>
            <svg viewBox="0 0 17 17" fill="currentColor"
              ><path
                d="M9.565 8.595l5.829 5.829a.686.686 0 01-.97.97l-5.83-5.83-5.828 5.83a.686.686 0 01-.97-.97l5.829-5.83-5.83-5.828a.686.686 0 11.97-.97l5.83 5.829 5.829-5.83a.686.686 0 01.97.97l-5.83 5.83z"
              /></svg
            >
          </button>
        </li>
      {/each}
    </ul>

    <div class="function">
      <button class="function-image-upload" on:click={() => fileUpload.click()}>
        <svg viewBox="0 0 22 20" fill="currentColor"
          ><path
            d="M3 5.5a1 1 0 00-1 1v10a1 1 0 001 1h13a1 1 0 001-1v-10a1 1 0 00-1-1H3zm0-2h13a3 3 0 013 3v10a3 3 0 01-3 3H3a3 3 0 01-3-3v-10a3 3 0 013-3zm1.5 6a1.5 1.5 0 100-3 1.5 1.5 0 000 3zm.488 6.155c.203-1.296.656-1.976 1.337-2.21.276-.094.609.009 1.55.495 1.57.811 2.416 1.025 3.567.457 1.115-.55 1.568-1.39 1.837-2.754l.064-.339c.13-.703.216-.954.35-1.083.341-.328.583-.359.989-.127a1 1 0 10.991-1.737 2.665 2.665 0 00-3.366.422c-.549.528-.723 1.039-.93 2.16l-.06.317c-.16.812-.336 1.138-.76 1.347-.368.182-.743.087-1.765-.44-1.446-.747-2.113-.953-3.117-.609-1.474.506-2.355 1.827-2.663 3.79a1 1 0 101.976.311zM6 2.5a1 1 0 110-2h12a4 4 0 014 4v9a1 1 0 01-2 0v-9a2 2 0 00-2-2H6z"
          /></svg
        >
        Image
      </button>
      <input class="file-upload" on:change={handleFileUpload} type="file" accept="image/*" bind:this={fileUpload} />
    </div>
    <p class="tips">You can paste or upload to images in textarea ✌️</p>
  </div>
</main>

<style lang="less">
  main {
    padding: 1em;
    margin: 0 auto;
    max-width: 600px;

    h1 {
      color: #000;
      text-transform: uppercase;
      font-size: 4em;
      background: #ff9800;
      font-style: oblique;
      text-align: center;
    }

    .container {
      width: 100%;
      height: 300px;
      position: relative;
      textarea {
        width: 100%;
        height: 300px;
        resize: none;
        border-radius: 10px;
        padding: 1em;
      }

      .list {
        left: 0;
        bottom: 0;
        right: 0;
        position: absolute;
        display: flex;
        list-style: none;
        padding: 0;
        margin: 10px;

        .item {
          position: relative;
          .item-close {
            position: absolute;
            right: 12px;
            top: 12px;
            width: 16px;
            height: 16px;
            background: rgba(255, 255, 255, 0.842);
            border: none;
            margin: 0;
            cursor: pointer;
            border-radius: 50%;
            transition: all 0.3s ease;
            &:hover {
              background: rgba(255, 255, 255, 0.96);
            }
            svg {
              width: 10px;
              height: 10px;
              fill: rgb(63, 63, 63);
              position: absolute;
              top: 3px;
              left: 3px;
              right: 3px;
            }
          }

          img {
            width: 60px;
            height: 60px;
            object-fit: cover;
            margin: 10px;
            box-shadow: 0 0 2px rgba(0, 0, 0, 0.5);
            border-radius: 4px;
          }
        }
      }

      .function {
        .function-image-upload {
          cursor: pointer;
          background: none;
          display: inline-flex;
          align-items: center;
          border: none;
          transition: all 0.3s ease;
          svg {
            height: 30px;
            fill: #ff9800;
            margin-right: 10px;
            transition: all 0.3s ease;
          }

          &:hover {
            svg {
              fill: rgb(247, 158, 26);
            }
          }
        }

        .file-upload {
          display: none;
        }
      }

      .tips {
        color: #dddddd;
        font-style: oblique;
      }
    }
  }
</style>
