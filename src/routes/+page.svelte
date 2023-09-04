<script lang="ts">
	import Monaco from 'svelte-monaco';
    import Node from "svelte-mosaic";
	import type { ChangeEventHandler } from 'svelte/elements';

    let fileUpload: HTMLInputElement;
	let value = `<!-- orange star -->
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
    <polygon points="100,25 120,75 170,75 130,115 150,165 100,135 50,165 70,115 30,75 80,75" fill="orange" />
</svg>
`

    const handleFileSelect: ChangeEventHandler<HTMLInputElement> = (event) => {
        const file = event.currentTarget?.files![0];
        const reader = new FileReader();
        reader.onload = (event) => {
            value = event.target!.result as string;
        };

        reader.readAsText(file);
        fileUpload.value = '';
    }
</script>

<header>
    <h1>svg-editor</h1>
    <button on:click={() => fileUpload.click()}>Upload</button>
    <button>Download</button>
</header>

<input bind:this={fileUpload} on:change|preventDefault={handleFileSelect} type="file" id="file" name="file" accept=".svg" hidden />

<Node direction="horizontal">
    <div slot="alpha" id="editor">
        <Monaco
            options={{ 
                language: 'html',
                automaticLayout: true,
            }}
            theme="monokai"
            bind:value
        />
    </div>
    
    <div slot="beta" id="display">
        {@html value}
    </div>
</Node>

<style>
    div#editor {
        height: 100%;
        width: 100%;
    }

    header {
        display: flex;
        justify-content: space-around;
        align-items: center;
        background-color: darkgray;
    }

    #display {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100%;
    }
</style>
