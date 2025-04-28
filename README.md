<!--
author: André Dietrich; Sebastian Zug

language: de

narrator: Deutsch Female

import: https://raw.githubusercontent.com/liaTemplates/ABCjs/main/README.md
        https://raw.githubusercontent.com/LiaTemplates/GGBScript/refs/heads/main/README.md

@WebSerial
<script>
(async function() {
  // Check if the Web Serial API is supported.
  if (!("serial" in navigator)) {
    console.error("Web Serial API is not supported in this browser.");
    return;
  }

  // Declare connection-related variables for later cleanup.
  let port = null;
  let reader = null;

  try {
    // Request and open the serial port.
    port = await navigator.serial.requestPort();
    await port.open({ baudRate: 115200 });

    // Create a TextEncoder instance.
    const encoder = new TextEncoder();
    // Function to stop any currently running code by sending Ctrl-C.
    async function stopCurrentProgram() {
      try {
        const writer = port.writable.getWriter();
        // Send Ctrl-C (ASCII 0x03) to interrupt any running code.
        await writer.write(encoder.encode("\x03"));
        // Wait briefly to allow the interrupt to be processed.
        await new Promise(resolve => setTimeout(resolve, 100));
        // Send a second Ctrl-C in case the first one was missed.
        await writer.write(encoder.encode("\x03"));
        writer.releaseLock();
      } catch (e) {
        console.error("Error sending Ctrl-C:", e);
      }
    }

    // Stop any running code before sending new code.
    await stopCurrentProgram();

    // Retrieve the entire Python code from the liascript input.
    const pythonCode = `@input(0)`;

    // Function to send code using MicroPython's paste mode.
    // In paste mode, the REPL buffers all lines until Ctrl‑D is received,
    // then it compiles and executes the entire code block at once.
    async function sendCodeInPasteMode(code) {
      const writer = port.writable.getWriter();
      // Enter paste mode (Ctrl‑E, ASCII 0x05).
      await writer.write(encoder.encode("\x05"));
      // Wait briefly for paste mode to be activated.
      await new Promise(resolve => setTimeout(resolve, 100));

      // Split the code into lines, preserving all indentation.
      const codeLines = code.split(/\r?\n/);
      for (const line of codeLines) {
        // Send each line exactly as-is, with CR+LF.
        await writer.write(encoder.encode(line + "\r\n"));
      }
      // Exit paste mode by sending Ctrl‑D (ASCII 0x04).
      await writer.write(encoder.encode("\x04"));
      writer.releaseLock();
      send.lia("LIA: terminal");
    }

    // Function that sends the code and reads output until the REPL prompt (">>>") is detected.
    // This ensures the entire block is executed before further input is allowed.
    async function sendCodeAndWaitForPrompt(code) {
      await sendCodeInPasteMode(code);
      let outputBuffer = "";
      const tempReader = port.readable.getReader();
      const decoder = new TextDecoder();
      let promptFound = false;

      while (!promptFound) {
        const { value, done } = await tempReader.read();
        if (done) break;
        if (value) {
          const text = decoder.decode(value);
          outputBuffer += text;
          console.stream(text);
          // Look for the REPL prompt (adjust if your prompt differs).
          if (outputBuffer.includes(">>>")) {
            promptFound = true;
          }
        }
      }
      await tempReader.releaseLock();
      return outputBuffer;
    }

    // Send the Python code and wait until the prompt is detected.
    await sendCodeAndWaitForPrompt(pythonCode);
    console.log("Python code executed and prompt detected.");

    // Now that execution is complete, enable terminal input.
    send.lia("LIA: terminal");

    // Start a global read loop to capture and display subsequent output.
    reader = port.readable.getReader();
    const globalDecoder = new TextDecoder();
    (async function readLoop() {
      try {
        while (true) {
          const { value, done } = await reader.read();
          if (done) {
            console.debug("Stream closed");
            send.lia("LIA: stop");
            break;
          }
          if (value) {
            console.stream(globalDecoder.decode(value));
          }
        }
      } catch (error) {
        console.error("Read error:", error);
      } finally {
        try { reader.releaseLock(); } catch (e) { /* ignore */ }
      }
    })();

    // Handler to send terminal input lines to MicroPython.
    send.handle("input", input => {
      (async function() {
        try {
          const writer = port.writable.getWriter();
          // Send the terminal input (preserving any whitespace) with CR+LF.
          await writer.write(encoder.encode(input + "\r\n"));
          writer.releaseLock();
        } catch (e) {
          console.error("Error sending input to MicroPython:", e);
        }
      })();
    });

    // Handler to clean up all connections and variables when a "stop" command is received.
    send.handle("stop", async () => {
      console.log("Cleaning up connections and stopping execution.");

      // Cancel the reader if it exists.
      if (reader) {
        try {
          await reader.cancel();
        } catch (e) {
          console.error("Error canceling reader:", e);
        }
        try { reader.releaseLock(); } catch (e) { /* ignore */ }
      }

      // Close the serial port if it's open.
      if (port) {
        try {
          await port.close();
        } catch (e) {
          console.error("Error closing port:", e);
        }
      }

      // Reset connection variables.
      port = null;
      reader = null;
      console.log("Cleanup complete.");
    });

  } catch (error) {
    console.error("Error connecting to the MicroPython device:", error);
    send.lia("LIA: stop");
  }
})();

"LIA: wait"
</script>
@end

-->



# LiaScript: OER Hackathon 2025


!?[Kollaborative OER Erstellung auf GitHUb](https://private-user-images.githubusercontent.com/10922356/293702414-00a24602-dc63-4b9a-894b-80967b914513.mp4?jwt=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NDU4NzM0MDIsIm5iZiI6MTc0NTg3MzEwMiwicGF0aCI6Ii8xMDkyMjM1Ni8yOTM3MDI0MTQtMDBhMjQ2MDItZGM2My00YjlhLTg5NGItODA5NjdiOTE0NTEzLm1wND9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTA0MjglMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUwNDI4VDIwNDUwMlomWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPThkZjlmYTE5ZTc5MDJiZjQ2NGJhNTQxMGZkMjNlOTVkNWNjZjA2ZDFmZjQzMzY1YmRlZmQ5YjE1OTc5ZTY1Y2QmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.0e7TkA2Xp11TV3-dkOxnb_HDv_JD-X4Qdd6U1keIWK4)


## Demo




## Programmierung

### Musik mit ABCjs

```abc
X: 1
M: 4/4
L: 1/8
K: Emin
|:D2|"Em"EBBA B2 EB|~B2 AB dBAG|"D"FDAD BDAD|FDAD dAFD|
"Em"EBBA B2 EB|B2 AB defg|"D"afe^c dBAF|"Em"DEFD E2:|
```
@ABCJS.eval


### WebSerial & MicroPython
<!--
persistent: true
-->



``` python
from microbit import *

# Display a scrolling message
display.scroll("Hello edrys!")

# Read the temperature
temp = temperature()
print("Temperature:", temp)

# Display a heart on the LED matrix
display.show(Image.HEART)
```
@WebSerial


<video autoplay="false" id="videoElement" style="display: none; width: 100%; padding: 5px"></video>

<script input="submit" default="Open Camera">
const video = document.querySelector("#videoElement")

if (video.srcObject === null) {
    if (navigator.mediaDevices.getUserMedia) {
        navigator.mediaDevices.getUserMedia({ video: true })
            .then(function (stream) {
                video.srcObject = stream
                video.style.display = "block"
                send.lia("Close Camera")
            })
            .catch(function (error) {
                console.log("Something went wrong!")
                send.lia("Camera Problem")
            });

        send.output("Waiting for Camera")
        "LIA: wait"
    } else {
        "No Camera connected"
    }
} else {
    const tracks = video.srcObject.getTracks()
    // Stop all tracks
    tracks.forEach(track => track.stop())
    video.style.display = "none"
    video.srcObject = null
    "Open Camera"
}
</script>


### Interactive Geometry


> Source: https://github.com/LiaTemplates/GGBScript
>
> A GGBScript JavaScript interpreter based on JavaScript.
>
> `import: https://raw.githubusercontent.com/LiaTemplates/GGBScript/refs/heads/main/README.md`

    {{1}}
``` js @GGBScript
Titel("Punkt A & B");

// Definiere einen Punkt
const A = Punkt(1, 2, "A");
const B = Punkt([4, 6], "B");
```

    {{2}}
<section>

A = (<script input="range" min="0" max="100" value="50" step="1" default="50" output="A0">
@input
</script>,
<script input="range" min="-100" max="100" value="50" step="1" default="50" output="A1">
@input
</script>
)

B = (<script input="range" min="0" max="100" value="96" step="1" default="96" output="B0">
@input
</script>,
<script input="range" min="-100" max="100" value="27" step="1" default="27" output="B1">
@input
</script>
)


C = (<script input="range" min="0" max="100" value="20" step="1" default="20" output="C0">
@input
</script>,
<script input="range" min="-100" max="100" value="20" step="1" default="20" output="C1">
@input
</script>
)

Rotation: 
<script input="range" min="0" max="360" value="0" step="1" default="0" output="rotation">
@input
</script>°


``` js @GGBScript
UserAxisLimits(0,150,0,60);

const A = Punkt(@input(`A0`), @input(`A1`), "A");
const B = Punkt(@input(`B0`), @input(`B1`), "B");
const C = Punkt(@input(`C0`), @input(`C1`), "C");

const P = Polygon("A", "B", "C");

Farbe(P, "red");

const M = Mittelpunkt(P);

const P2 = Rotation(P, M, @input(`rotation`));

Farbe(P2, "blue");

Kreis(M, 16, "Kreis");
```

</section>

## Klassenräume


## Remote Labore

