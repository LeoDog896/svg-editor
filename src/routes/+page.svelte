<script lang="ts">
	import Monaco from 'svelte-monaco';
	import Node from 'svelte-mosaic';
	import type { ChangeEventHandler } from 'svelte/elements';
	import { localStore } from 'svelte-persistent';
	import download from 'downloadjs';

	const storagePrefix = 'leodog896-svg-editor-';
	let fileName = localStore(`${storagePrefix}filename`, 'untitled');

	let fileUpload: HTMLInputElement;
	let canvas: HTMLCanvasElement;

	let value = localStore(
		`${storagePrefix}filecontent`,
		`<!-- orange star -->
<svg width="200" height="200" xmlns="http://www.w3.org/2000/svg">
    <polygon points="100,25 120,75 170,75 130,115 150,165 100,135 50,165 70,115 30,75 80,75" fill="#DE8F6E" />
</svg>
`
	);

	const handleFileSelect: ChangeEventHandler<HTMLInputElement> = (event) => {
		const file = event.currentTarget?.files![0];
		const reader = new FileReader();
		reader.onload = (event) => {
			$fileName = file.name.replace('.svg', '');
			$value = event.target!.result as string;
		};

		reader.readAsText(file);
		fileUpload.value = '';
	};

	const downloadSVG = () => {
		const svg = new Blob([$value], { type: 'image/svg+xml;charset=utf-8' });
		const url = URL.createObjectURL(svg);
		const img = new Image();
		img.onload = () => {
			download(
				$value,
				$fileName
					.replaceAll('{width}', img.width.toString())
					.replaceAll('{height}', img.height.toString()) + '.svg',
				'image/svg+xml'
			);
		};

		img.src = url;
	};

	const downloadPNG = () => {
		const svg = new Blob([$value], { type: 'image/svg+xml;charset=utf-8' });
		const url = URL.createObjectURL(svg);
		const img = new Image();
		img.onload = () => {
			canvas.width = img.width;
			canvas.height = img.height;
			const ctx = canvas.getContext('2d');
			ctx?.drawImage(img, 0, 0);
			const png = canvas.toDataURL('image/png');
			download(
				png,
				$fileName
					.replaceAll('{width}', img.width.toString())
					.replaceAll('{height}', img.height.toString()) + '.png',
				'image/png'
			);
		};
		img.src = url;
	};
</script>

<canvas bind:this={canvas} hidden />

<header>
	<div class="title">
		<h1>svg-editor</h1>
		<input type="text" placeholder="file name" bind:value={$fileName} />
	</div>
	<div id="buttons">
		<button on:click={() => fileUpload.click()}>Upload</button>
		<button on:click={downloadSVG}>Download as SVG</button>
		<button on:click={downloadPNG}>Download as PNG</button>
		<button>Format Code</button>
	</div>
</header>

<input
	bind:this={fileUpload}
	on:change|preventDefault={handleFileSelect}
	type="file"
	id="file"
	name="file"
	accept=".svg"
	hidden
/>

<Node direction="horizontal">
	<div slot="alpha" id="editor">
		<Monaco
			options={{
				language: 'html',
				automaticLayout: true
			}}
			theme="monokai"
			bind:value={$value}
		/>
	</div>

	<div slot="beta" id="display">
		{@html $value}
	</div>
</Node>

<style>
	div#editor {
		height: 100%;
		width: 100%;
	}

	.title {
		display: flex;
		align-items: center;
	}

	.title input {
		margin-left: 1rem;
	}

	header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		background-color: #88ab75;
		color: #f0f4ef;
	}

	#buttons button {
		padding: 0.5rem;
		margin: 0.5rem;
		border-radius: 0.5rem;
		border: none;
		background-color: #f0f4ef;
		color: #2d93ad;
		font-size: 1rem;
		font-family: 'Manrope Variable', sans-serif;
	}

	#buttons button:hover {
		background-color: #2d93ad;
		color: #f0f4ef;
		cursor: pointer;
	}

	header h1 {
		margin-left: 1rem;
		font-weight: 800;
	}

	header #buttons {
		margin-right: 1rem;
	}

	#display {
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100%;
        z-index: -1;
	}

    :global(svg) {
        user-select: none;
        pointer-events: none;
        z-index: -1;
    }
</style>
