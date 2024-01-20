<script>
    import {ESPLoader, Transport} from "esptool-js";
    import JSZip from 'jszip';
    import CryptoJS from 'crypto-js'
    import {Alert, Button, Progressbar, Spinner, Toast} from 'flowbite-svelte';
    import {sineOut} from 'svelte/easing';
    import {ArrowRightOutline, CheckCircleSolid, LinkSolid, UploadOutline} from 'flowbite-svelte-icons';

    let device = null;
    let transport;
    let chip = null;
    let esploader;
    let  flashFiles = [], isConnected = false;
    let state = "", support = false, error = ""
    if ("serial" in navigator) {
        console.log("Web Serial API is supported.");
        support = true
    }


    const downloadAndUnzip = async () => {
        try {
            const response = await fetch('/main.zip');
            const zipData = await response.arrayBuffer();
            const zipper = new JSZip();
            const unzipResult = await zipper.loadAsync(zipData);

            for (const [filePath, data] of Object.entries(unzipResult.files)) {
                if (filePath.startsWith("__MACOSX")) {
                } else if (filePath.endsWith('.ino.bootloader.bin')) {
                    flashFiles.push({data: await zipper.file(data.name)?.async("binarystring"), address: 0x1000});
                } else if (filePath.endsWith('.ino.partitions.bin')) {
                    flashFiles.push({data: await zipper.file(data.name)?.async("binarystring"), address: 0x8000});
                } else if (filePath.endsWith('.ino.bin')) {
                    flashFiles.push({data: await zipper.file(data.name)?.async("binarystring"), address: 0x10000});
                }
            }

            console.log(flashFiles);

        } catch (error) {
            console.error('Error downloading and unzipping:', error);
        }
    };


    // Trigger the download and unzip function on component mount
    $: downloadAndUnzip();

    const connect = async () => {
        state = "connecting"
        if (device === null) {
            device = await navigator.serial.requestPort();
            transport = new Transport(device);
        }

        try {
            const flashOptions = {
                transport,
                baudrate: 115200,
                terminal: espLoaderTerminal,
            };
            esploader = new ESPLoader(flashOptions);
            console.log(esploader)
            await esploader.main_fn().then((e) => {
                console.log(e)
                isConnected = true
                chip = e
                error = ""
                state = "connected"
            });
        } catch (e) {
            error = e.message
            state = ""
            isConnected = false
            console.error(e);
            terminalData = [e.message]
        }
    };
    const disconnect = () => {
        if (device) {
            device.close();
            device = null;
        }
        espLoaderTerminal.clean();
        chip = null;
        isConnected = false
        error = ""
        // state = ""
    };

    let terminalData = [];

    // Terminal object for displaying data on the screen
    let espLoaderTerminal = {
        clean() {
            terminalData = [];
        },
        writeLine(data) {
            terminalData = [...terminalData, data];
            console.log(data)
        },
        write(data) {
            terminalData = [...terminalData, data];
            console.log(data)
        },
    };
    let progress = 0
    const program = async () => {
        try {
            console.log(flashFiles)
            const flashOptions = {
                fileArray: flashFiles,
                // flashSize: "4MB",
                // eraseAll: false,
                // compress: true,
                // flashFreq: "80MHz",
                // flashMode: "QIO",
                flashSize: "keep",
                eraseAll: false,
                compress: true,
                reportProgress: (fileIndex, written, total) => {
                    state = "uploading"
                    progress = Math.ceil((written / total) * 100);
                    console.log(progress)
                },
                calculateMD5Hash: (image) => CryptoJS.MD5(CryptoJS.enc.Latin1.parse(image)),
            };
            await esploader.write_flash(flashOptions).then((e) => {
                console.log(e)
                state = "success"
                error = ""
            });
        } catch (e) {
            console.error(e);
            terminalData = [`Error: ${e.message}`];
            state = "failed"
            error = e.message
        } finally {
            // Hide progress bars and show erase buttons
            disconnect()

        }
    }
    console.log(state)
</script>

<main>

    <p class="text-4xl dark:text-white mb-8">Flow Firmware Updater</p>
    {#if !support}
        <p class="text-2xl dark:text-white bg-red-600 mb-8 p-2 rounded">Browser doesn't support this system</p>

    {/if}

    {#if isConnected}
        <Alert color="dark" class="mb-4">
            <h4>Connected to {chip}</h4>
        </Alert>
    {/if}

    {#if error}
        <Alert color="dark" class="mb-4">
            <h4>Error: {error}</h4>
        </Alert>
    {/if}

    {#if state === "success"}
        <Toast class="m-4" color="green">
            <svelte:fragment slot="icon">
                <CheckCircleSolid class="w-5 h-5"/>
                <span class="sr-only">Check icon</span>
            </svelte:fragment>
            Firmware updated successfully
        </Toast>
    {:else if (state === "failed")}

        <Toast class="m-4" color="red">
            <svelte:fragment slot="icon">
                <CheckCircleSolid class="w-5 h-5"/>
                <span class="sr-only">Check icon</span>
            </svelte:fragment>
            Error in Firmware update
        </Toast>
    {/if}

    <Button disabled={!support} on:click={connect}>
        <LinkSolid class="w-4 h-3.5 me-2 mr-2"/>
        {#if state === "connecting"}
            <Spinner class="me-3" size="4" color="white"/>
            Connecting ...
        {:else if (state === "connected")}
            Connected
        {:else }
            Connect to device
        {/if}
    </Button>

    <Button disabled={!support} color="red" on:click={disconnect}>
        <ArrowRightOutline class="w-4 h-3.5 me-2 ml-2"/>
              Disconnect device
    </Button>
    <br/>

    {#if isConnected}
        <Button class="m-4" color="green" disabled={state==="uploading"} on:click={program}>
            <UploadOutline class="w-3.5 h-3.5 me-2"/>
            Update firmware
        </Button>
    {/if}

    {#if state === "uploading"}
        <Progressbar
                {progress}
                animate
                precision={2}
                labelOutside="Uploading"
                labelInside
                tweenDuration={1500}
                easing={sineOut}
                size="h-6"
                labelInsideClass="bg-blue-600 text-blue-100 text-base font-medium text-center p-1 leading-none rounded-full"
                class="mb-8"
        />
    {/if}


    <!--{#if terminalData.length > 0}-->
    <!--    <div>-->
    <!--        <h3>Terminal Output:</h3>-->
    <!--        {#each terminalData as line }-->
    <!--            <p>{line}</p>-->
    <!--        {/each}-->
    <!--    </div>-->
    <!--{/if}-->
</main>


<style>

</style>