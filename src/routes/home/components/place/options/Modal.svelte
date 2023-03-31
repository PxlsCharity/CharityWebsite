<script>
   import { onMount } from 'svelte';

    onMount(getCanvas);

    let canvas;
    let canvasContainer;
    let canvasProperties = {
        width: 0,
        height: 0,
        translateX: 0,
        translateY: 0,
        zoom: 1
    };
    let image;
    let imageProperties = {
        width: 0,
        height: 0
    };

    async function getCanvas() {
        const response = await fetch('https://charity.pxls.space/api/place', {
            method: 'GET'
        });
        const blob = await response.blob();
        const ctx = canvas.getContext('2d');
        const img = new Image();
        img.src = URL.createObjectURL(blob);
        img.onload = function(){
            canvas.width = img.width;
            canvas.height = img.height;
            canvasProperties.width = img.width;
            canvasProperties.height = img.height;
            ctx.drawImage(img, 0, 0);
        }
    }

    function translateCanvas(e) {
        if ((e.target.id !== 'template' && e.buttons === 1) || e.buttons === 2) {
            canvasProperties.translateX = canvasProperties.translateX + e.movementX;
            canvasProperties.translateY = canvasProperties.translateY + e.movementY;
            if (canvasProperties.translateX > 100) canvasProperties.translateX = 100;
            if (canvasProperties.translateY > 100) canvasProperties.translateY = 100;
            if (canvasProperties.translateX < -(canvasProperties.width * canvasProperties.zoom - canvasContainer.clientWidth) - 100) canvasProperties.translateX = -(canvasProperties.width * canvasProperties.zoom - canvasContainer.clientWidth) - 100;
            if (canvasProperties.translateY < -(canvasProperties.height * canvasProperties.zoom - canvasContainer.clientHeight) - 100) canvasProperties.translateY = -(canvasProperties.height * canvasProperties.zoom - canvasContainer.clientHeight) - 100;
            canvas.style.transform = `translate(${canvasProperties.translateX}px, ${canvasProperties.translateY}px)`;
            image.style.transform = `translate(${canvasProperties.translateX + subTemplate.x * canvasProperties.zoom}px, ${canvasProperties.translateY + subTemplate.y * canvasProperties.zoom}px)`;
        }
        if (e.target.id === 'template' && e.buttons === 1) {
            const x = Math.floor((-canvasProperties.translateX + e.layerX) / canvasProperties.zoom);
            const y = Math.floor((-canvasProperties.translateY + e.layerY) / canvasProperties.zoom);
            const newX = Math.floor((-canvasProperties.translateX + e.layerX + e.movementX) / canvasProperties.zoom);
            const newY = Math.floor((-canvasProperties.translateY + e.layerY + e.movementY) / canvasProperties.zoom);
            const deltaX = newX - x;
            const deltaY = newY - y;
            subTemplate.x = subTemplate.x + deltaX;
            subTemplate.y = subTemplate.y + deltaY;
            if (subTemplate.x < 0) subTemplate.x = 0;
            if (subTemplate.y < 0) subTemplate.y = 0;
            if (subTemplate.x > canvasProperties.width - 1) subTemplate.x = canvasProperties.width - 1;
            if (subTemplate.y > canvasProperties.height - 1) subTemplate.y = canvasProperties.height - 1;
            image.style.transform = `translate(${canvasProperties.translateX + subTemplate.x * canvasProperties.zoom}px, ${canvasProperties.translateY + subTemplate.y * canvasProperties.zoom}px)`;
        }
        // https://pride.place/template.png
    }

    function zoomCanvas(e) {
        const x = -canvasProperties.translateX + e.layerX;
        const y = -canvasProperties.translateY + e.layerY;
        canvasProperties.zoom = e.deltaY < 0 ? canvasProperties.zoom * 2 : canvasProperties.zoom / 2;
        if (canvasProperties.zoom >= 1 && canvasProperties.zoom <= 64) {
            canvas.style.width = `${canvasProperties.width * canvasProperties.zoom}px`;
            canvas.style.height = `${canvasProperties.height * canvasProperties.zoom}px`;
            image.style.width = `${imageProperties.width * canvasProperties.zoom}px`;
            image.style.height = `${imageProperties.height * canvasProperties.zoom}px`;

            canvasProperties.translateX = e.deltaY < 0 ? canvasProperties.translateX - x : canvasProperties.translateX + x / 2;
            canvasProperties.translateY = e.deltaY < 0 ? canvasProperties.translateY - y : canvasProperties.translateY + y / 2;
            
            if (canvasProperties.translateX > 100) canvasProperties.translateX = 100;
            if (canvasProperties.translateY > 100) canvasProperties.translateY = 100;
            if (canvasProperties.translateX < -(canvasProperties.width * canvasProperties.zoom - canvasContainer.clientWidth) - 100) canvasProperties.translateX = -(canvasProperties.width * canvasProperties.zoom - canvasContainer.clientWidth) - 100;
            if (canvasProperties.translateY < -(canvasProperties.height * canvasProperties.zoom - canvasContainer.clientHeight) - 100) canvasProperties.translateY = -(canvasProperties.height * canvasProperties.zoom - canvasContainer.clientHeight) - 100;
            
            canvas.style.transform = `translate(${canvasProperties.translateX}px, ${canvasProperties.translateY}px)`;
            image.style.transform = `translate(${canvasProperties.translateX + subTemplate.x * canvasProperties.zoom}px, ${canvasProperties.translateY + subTemplate.y * canvasProperties.zoom}px)`;
        }
        if (canvasProperties.zoom > 64) canvasProperties.zoom = 64;
        if (canvasProperties.zoom < 1) canvasProperties.zoom = 1;
    }

    async function loadURL(url) {
        const response = await fetch(`https://pxls.space/cors/${url}`, {
            method: 'GET'
        });
        const blob = await response.blob();
        if (['image/png', 'image/jpeg', 'image/webp', 'image/gif', 'image/avif', 'image/tiff'].includes(blob.type)) {
            const ctx = image.getContext('2d');
            const img = new Image();
            img.src = URL.createObjectURL(blob);
            img.onload = function(){
                image.width = img.width;
                image.height = img.height;
                imageProperties.width = img.width;
                imageProperties.height = img.height;
                ctx.drawImage(img, 0, 0);
            }
            image.style.transform = `translate(${canvasProperties.translateX + subTemplate.x * canvasProperties.zoom}px, ${canvasProperties.translateY + subTemplate.y * canvasProperties.zoom}px)`;
        }
    }

    function closeModal() {
        image.width = 0;
        image.height = 0;
        image.style.transform = canvas.style.transform;
        modalOpen = false;
        template = template;
    }

    export let template;
    export let modalOpen;
    export let subTemplate;
</script>

<div class="{modalOpen ? '' : 'hidden'} fixed inset-0 z-20 w-full h-full flex" on:wheel|preventDefault>
    <div class="bg-base-100 w-1/2 z-30 p-4">
        {#if subTemplate !== null && subTemplate !== undefined}
            <label for="templateName" class="label">
                <span class="label-text">Template Name</span>
            </label>
            <input type="text" placeholder="Type here..." class="input input-bordered w-full mb-2" bind:value={subTemplate.name}/>
            <div class="collapse collapse-arrow bg-base-200 rounded-md mb-2">
                <input type="checkbox" checked />
                <div class="collapse-title text-xl font-medium">
                    Template Sources
                </div>
                <div class="collapse-content">
                    {#each subTemplate.sources as source}
                    <div class="flex">
                        <div class="w-full mr-2">
                            <label for="templateSources" class="label">
                                <span class="label-text">Template Image URL</span>
                            </label>
                            <input type="text" placeholder="Type here..." class="input input-bordered w-full mb-2" bind:value={source}/>
                        </div>
                        <div>
                            <label for="templateSources" class="label">
                                <span class="label-text">&nbsp;</span>
                            </label>
                            <button class="btn" on:click={loadURL(source)}>Load</button>
                        </div>
                    </div>
                    {/each}
                    <button class="btn" on:click={() => {subTemplate.sources.push(""); subTemplate = subTemplate}}>Add Source</button>
                    <button class="btn {subTemplate && subTemplate.sources.filter(s => s !== "").length > 0 ? '' : 'btn-disabled'}" on:click={() => {subTemplate.sources.pop(); subTemplate = subTemplate}}>Remove Source</button>
                </div>
            </div>
            <div class="flex flex-row">
                <div class="mr-2 w-full">
                    <label for="templateSources" class="label flex">
                        <span class="label-text">X</span>
                    </label>
                    <input type="number" placeholder="" class="input input-bordered w-full mb-2" bind:value={subTemplate.x}/>
                </div>
                <div class="w-full">
                    <label for="templateSources" class="label">
                        <span class="label-text">Y</span>
                    </label>
                    <input type="number" placeholder="" class="input input-bordered w-full mb-2" bind:value={subTemplate.y}/>
                </div>
            </div>
        {/if}
        <button class="btn" on:click={() => {template.templates.pop(); closeModal()}}>Cancel</button>
        <button class="btn {subTemplate && subTemplate.name && subTemplate.name !== '' && subTemplate.sources.filter(s => s !== "").length > 0 ? '' : 'btn-disabled'}" on:click={closeModal}>Done</button>
    </div>
    <div bind:this={canvasContainer} class="bg-base-200 w-1/2" on:mousemove|preventDefault={translateCanvas} on:wheel|preventDefault={zoomCanvas} on:contextmenu|preventDefault>
        <div class="absolute w-max">
            <canvas class="[image-rendering:pixelated]" bind:this={canvas}/>
        </div>
        <div class="absolute w-max">
            <canvas id="template" class="[image-rendering:pixelated] z-40" bind:this={image}/>
        </div>
    </div>
</div>