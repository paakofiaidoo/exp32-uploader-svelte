# ESP32 Firmware Updater with Browser Serial API

This Svelte app allows you to update the firmware of an ESP32 device using the Browser Serial API. It is optimized for the latest Chromium-based browsers.

## Installation

Follow these steps to set up and run the ESP32 Firmware Updater:

1. **Clone the Repository:**
   ```bash
   git clone https://github.com/your-username/esp32-firmware-updater.git
   cd esp32-firmware-updater
   ```

2. **Install Dependencies:**
   ```bash
   npm install

   ```
   
3. **Export Binary from Arduino:**
    - Export the firmware binary for your ESP32 device from the Arduino IDE.

4. **Create "main.zip":**
    - Combine and zip the binary files into a single ZIP archive named "main.zip."
    - Place the "main.zip" file in the `public` folder of the Svelte app.


5. **Run the App:**
   ```bash
   npm run dev
   ```

6. **Access the App:**
    - Open your browser and go to `http://localhost:5174` to access the ESP32 Firmware Updater.

## Usage

1. Open the app in a Chromium-based browser.
2. Connect your ESP32 device to the computer.
3. Follow the on-screen instructions to initiate the firmware update.
4. The app will use the Browser Serial API to communicate with the connected ESP32 device.
5. Monitor the update progress and wait for the process to complete.

## Best Use Case

This app is designed for updating remote firmware on ESP32-powered devices with serial access. It streamlines the process using the Browser Serial API for a seamless and efficient firmware update.

## Important Notes

- Ensure that your ESP32 device is connected and powered before initiating the firmware update.
- Verify that your browser supports the Browser Serial API, as this app relies on this feature.

## Troubleshooting

If you encounter any issues or have questions, please check the following:

- Verify that the ESP32 device is correctly connected.
- Ensure the "main.zip" file is placed in the `public` folder.
- Confirm that your browser supports the Browser Serial API.

For additional assistance, refer to the [Svelte documentation](https://svelte.dev/docs) or open an issue on the GitHub repository.

Feel free to contribute, report bugs, or suggest improvements. Happy updating!