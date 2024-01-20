<script>
    import {ESPLoader, Transport} from "esptool-js";
    import JSZip from 'jszip';
    import CryptoJS from 'crypto-js'

    let device = null;
    let transport;
    let chip = null;
    let esploader;
    let fileContent, flashFiles = [], isConnected = false;


    if ("serial" in navigator) {
        console.log("Web Serial API is supported.");
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
            });
        } catch (e) {
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
    };

    let terminalData = [];

    // Terminal object for displaying data on the screen
    let espLoaderTerminal = {
        clean() {
            terminalData = [];
        },
        writeLine(data) {
            terminalData = [...terminalData, data];
        },
        write(data) {
            terminalData = [...terminalData, data];
        },
    };
    let progressBars = []
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
                    console.log((written / total) * 100)
                    progressBars[fileIndex] = (written / total) * 100;
                },
                calculateMD5Hash: (image) => CryptoJS.MD5(CryptoJS.enc.Latin1.parse(image)),
            };
            await esploader.write_flash(flashOptions).then((e)=>{
                console.log(e)
            });
        } catch (e) {
            console.error(e);
            terminalData = [`Error: ${e.message}`];
        } finally {
            // Hide progress bars and show erase buttons

        }
    }
</script>

<main>
    {#if isConnected}
        <h4>Connected to {chip}</h4>
    {/if}
    <button on:click={connect}>Connect</button>
    <button on:click={disconnect}>Disconnect</button>
    {#if isConnected}<button on:click={program}>program</button>{/if}

    {#if terminalData.length > 0}
        <div>
            <h3>Terminal Output:</h3>
            {#each terminalData as line }
                <p>{line}</p>
            {/each}
        </div>
    {/if}
</main>


<style>
    .logo {
        height: 6em;
        padding: 1.5em;
        will-change: filter;
        transition: filter 300ms;
    }

    .logo:hover {
        filter: drop-shadow(0 0 2em #646cffaa);
    }

    .logo.svelte:hover {
        filter: drop-shadow(0 0 2em #ff3e00aa);
    }

    .read-the-docs {
        color: #888;
    }
</style>