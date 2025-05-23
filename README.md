<!--
author: André Dietrich; Sebastian Zug

language: de

narrator: Deutsch Female

import: https://raw.githubusercontent.com/liaTemplates/ABCjs/main/README.md
        https://raw.githubusercontent.com/LiaTemplates/GGBScript/refs/heads/main/README.md
        https://raw.githubusercontent.com/LiaTemplates/Communica/0.0.2/README.md

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



# LiaScript & Kollaboration: OER Hackathon 2025

https://LiaScript.github.io
---------------------------

[qr-code](https://github.com/LiaPlayground/OER-Hackathon-2025-Lightning-Talk)

> __GitHub:__ https://github.com/LiaPlayground/OER-Hackathon-2025-Lightning-Talk
>
> __LiaScript:__ https://LiaScript.github.io/course/?https://raw.githubusercontent.com/LiaPlayground/OER-Hackathon-2025-Lightning-Talk/refs/heads/main/README.md
>
> __LiveEditor:__ https://liascript.github.io/LiveEditor/?/show/file/https://raw.githubusercontent.com/LiaPlayground/OER-Hackathon-2025-Lightning-Talk/refs/heads/main/README.md

## Kollaboration

https://github.com/TUBAF-IfI-LiaScript/
---------------------------------------

    {{1}}
!?[Kollaborative OER Erstellung auf GitHUb](media/liascript-github.mp4)<!-- autoplay -->

## Ein Dokument verschiedene Darstellungen

                  --{{0}}--
Hallo, ich bin ein Markdown-Dokument, das im Lehrbuchmodus, als Folien oder als interaktive Präsentation dargestellt werden kann. Ändere den Präsentationsmodus und schau, was passiert.

                  --{{1}}--
Jeder Markdown-Block kann mit zwei geschweiften Klammern und einer Zahl versehen werden, um festzulegen, wann dieser Block in der Präsentation erscheinen soll.

                    {{1}}
* Diese Liste wird bei Schritt 1 sichtbar
* und wird nicht entfernt

                  --{{2}}--
Im Gegensatz dazu wird die folgende Tabelle nur bei Schritt 2 sichtbar.

                   {{2-3}}
| Tier            |	Gewicht in kg | Lebensdauer Jahre |
|-----------------|--------------:|------------------:|
| Maus            |	        0.028 |                02 |
| Schaf           |	           90 |                12 |
| Mensch          |	           68 |                70 |

                 --{{3}}--
Wie du vielleicht erwartest, bin ich ein Kommentar für Animationsschritt 3. Ich werde im Lehrbuchmodus sichtbar sein oder in den anderen Modi laut vorgelesen werden. Versuche, das Dokument in eine andere Sprache zu übersetzen und schau, was passiert.

                   {{3}}
```
Kommentare in LiaScript werden mit zwei Klammern gekennzeichnet,
die von zwei Strichen umgeben sind. Sie sollen nützliche
Informationen über den Animationsschritt enthalten.
```
                   {{3}}
> __Um die Google-Übersetzung zu deaktivieren, musst du diese Seite neu laden.__

### Quizze

Wann wurde Göttingen gegründet?

- [( )] 953 n. Chr.
- [( )] zwischen 1150 und 1200 n. Chr.
- [(X)] 1230 n. Chr.

#### Mehr

                    {{UK English Male |>}}
The film that I saw [[(that)|those|these|then]] night wasn’t very good.
It was all [[ about ]] a man [[ who ]] built a
time machine so he [[ could ]] travel back in time.
It took him ages and ages [[ to ]] build the machine.

    {{2}}
<!-- data-title="LMS market share 2022 (North America)" -->
| Brightspace    | Canvas          | Classroom      | Moodle          | Others         | Schoology      |
|:--------------:|:---------------:|:--------------:|:---------------:|:--------------:|:--------------:|
| [[(3 %)|11 %]] | [[13 %|(28 %)]] | [[8 %|(24 %)]] | [[(11 %)|31 %]] | [[2 %|(12 %)]] | [[7 %|(22 %)]] |

---

    {{3}}
| LMS | Average cost per month   |
|-----|--------------------------|
| Brightspace | [[13 $|(80 $)]]  |
| Canvas      | [[(10 $)|99 $]]  |
| Classroom   | [[(0 free)|3 $]] |
| Moodle      | [[11 $ |(15 $)]] |
| Schoology   | [[(4 $)|12 $]]   |

### oEmbed ??

??[SketchFab Göttingen](https://sketchfab.com/3d-models/goettingen-gauss-weber-denkmal-monument-a8c83c264df849c9abf2c50dfac99802 "Das Denkmal steht für die Öffentlichkeit zur Betrachtung auf dem Göttinger Stadtwall. Es handelt sich hierbei um einen High End 3D Photogrammetrie-Scan. Auszug aus Wikipedia: Johann Carl Friedrich Gauß (latinisiert Carolus Fridericus Gauss; * 30. April 1777 in Braunschweig; † 23. Februar 1855 in Göttingen) war ein deutscher Mathematiker, Astronom, Geodät und Physiker. Wegen seiner überragenden wissenschaftlichen Leistungen galt er bereits zu seinen Lebzeiten als Princeps Mathematicorum („Fürst der Mathematiker; Erster unter den Mathematikern“).")

??[Falstad Circuit Sim](https://www.falstad.com/circuit/circuitjs.html)

## Erweiterungen

https://github.com/topics/liascript-template
--------------------------------------------

    {{1}}
``` abc  @ABCJS.render
% channel: 0
X:353
T: GLUECK AUF DER STEIGER KOEMMT
O: Europa, Mitteleuropa, Deutschland
R: Staende -, Bergmanns - Lied
M: 4/4
L: 1/16
K: G
 | G8F4A4 | G8z8 |
B8A4c4 | B8z4
G2A2 | B4B4B4A2B2 | c4A3AA4
A2B2 | c4c4c4B2c2 | d4B3BB4
A4 | G8F8 | G4e4d4
c2A2 | B8A8 | G8z8
```

### Programmierung

#### SparQL

https://github.com/LiaTemplates/Communica

``` sql
# source: https://fragments.dbpedia.org/2015/en

SELECT ?s ?p ?o WHERE {
  ?s ?p <http://dbpedia.org/resource/Ukraine>.
  ?s ?p ?o
} LIMIT 10
```
@Communica.SPARQL


#### WebSerial & MicroPython
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

![Simpsons Schule](https://media0.giphy.com/media/v1.Y2lkPTc5MGI3NjExMDZjYW1vYmgxd3Y4Ynl5emxrOXVnbmQ2MHRoNzl5MGJpYmt4dTE4byZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/UGXZWnlUlv1AI/giphy.gif)<!--
style="width: 100%"
-->

## Remote Labore: Edrys-Lite

https://edrys-labs.github.io

!?[YouTube 1](https://www.youtube.com/watch?v=6ZjGHorc2ds "link: https://cross-lab.org/deploying-remote-labs-in-seconds-with-edrys-lite/")
!?[YouTube 2](https://www.youtube.com/watch?v=Uv79Y8EhBVw "link: https://cross-lab.org/crosslab-at-university-future-festival/")
!?[YouTube 3](https://www.youtube.com/watch?v=Lri9IQBPJLU "link: https://cross-lab.org/crosslab-at/")