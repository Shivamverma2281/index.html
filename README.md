<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Voice Translator & A4 Printer</title>

    <style>
        /* --- CSS STYLING --- */
        body {
            font-family: 'Segoe UI', Arial, sans-serif;
            text-align: center;
            background: #f0f2f5;
            margin: 0;
            padding: 20px;
        }

        h1 {
            color: #333;
        }

        .controls {
            margin: 20px auto;
            background: white;
            padding: 15px;
            border-radius: 8px;
            box-shadow: 0 2px 5px rgba(0,0,0,0.1);
            max-width: 600px;
        }

        select, button {
            padding: 10px 15px;
            margin: 5px;
            font-size: 16px;
            border-radius: 5px;
            border: 1px solid #ccc;
            outline: none;
        }

        select {
            cursor: pointer;
            background: #fff;
        }

        button {
            cursor: pointer;
            color: white;
            border: none;
            transition: background 0.2s;
        }

        .btn-start { background-color: #28a745; }
        .btn-start:hover { background-color: #218838; }

        .btn-stop { background-color: #dc3545; }
        .btn-stop:hover { background-color: #c82333; }

        .btn-print { background-color: #007bff; }
        .btn-print:hover { background-color: #0069d9; }

        .status {
            font-weight: bold;
            margin: 10px 0;
            color: #666;
        }

        .text-areas {
            margin: 20px auto;
            max-width: 800px;
            text-align: left;
        }

        textarea {
            width: 100%;
            height: 100px;
            padding: 12px;
            box-sizing: border-box;
            border: 1px solid #ccc;
            border-radius: 6px;
            resize: vertical;
            font-size: 16px;
            margin-bottom: 20px;
        }

        /* A4 Sheet Preview Styling */
        .a4-container-label {
            font-weight: bold;
            text-align: center;
            margin-top: 30px;
            color: #444;
        }

        .a4 {
            width: 210mm;
            min-height: 297mm;
            background: white;
            margin: 20px auto;
            padding: 20mm;
            box-sizing: border-box;
            box-shadow: 0 0 15px rgba(0,0,0,0.15);
            text-align: left;
            border: 1px solid #ddd;
            word-wrap: break-word;
        }

        #translatedText {
            font-size: 18px;
            line-height: 1.6;
            color: #000;
            white-space: pre-wrap;
        }

        /* --- PRINT MEDIA STYLING (केवल A4 प्रिंट करने के लिए) --- */
        @media print {
            body * {
                visibility: hidden;
            }
            .a4, .a4 * {
                visibility: visible;
            }
            .a4 {
                position: absolute;
                left: 0;
                top: 0;
                width: 210mm;
                height: 297mm;
                box-shadow: none;
                border: none;
                margin: 0;
                padding: 0;
            }
        }
    </style>
</head>
<body>

<h1>Voice Translator</h1>

<!-- Controls Panel -->
<div class="controls">
    <select id="language">
        <option value="en-hi">English to Hindi (अंग्रेजी से हिंदी)</option>
        <option value="hi-en">Hindi to English (हिंदी से अंग्रेजी)</option>
    </select>

    <button class="btn-start" onclick="startListening()">Start</button>
    <button class="btn-stop" onclick="stopListening()">Stop</button>
    <button class="btn-print" onclick="printPage()">Print</button>

    <div id="statusText" class="status">Status: Idle</div>
</div>

<!-- Live Speech Text Output -->
<div class="text-areas">
    <h3>Your Voice (आपकी आवाज़ यहाँ दिखेगी):</h3>
    <textarea id="speechText" placeholder="Click 'Start' and speak..." readonly></textarea>
</div>

<!-- A4 Paper Preview Component -->
<div class="a4-container-label">A4 Document Print Preview</div>
<div class="a4">
    <div id="translatedText">Translated text will appear here inside the A4 sheet...</div>
</div>

<!-- --- JAVASCRIPT LOGIC --- -->
<script>
    let recognition;
    let isListening = false;

    // Check Browser Compatibility for Speech Recognition
    if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        recognition = new SpeechRecognition();
        recognition.continuous = true;
        recognition.interimResults = false;

        // Handle Speech results
        recognition.onresult = function(event) {
            let currentResultIndex = event.results.length - 1;
            let text = event.results[currentResultIndex][0].transcript;

            document.getElementById("speechText").value = text;
            updateStatus("Translating... (अनुवाद हो रहा है...)");
            translateText(text);
        };

        recognition.onerror = function(event) {
            console.error("Speech recognition error", event.error);
            updateStatus("Error occurred: " + event.error);
        };

        recognition.onend = function() {
            if(isListening) {
                // Automatically restart if stopped due to silence but 'Stop' wasn't clicked
                recognition.start();
            } else {
                updateStatus("Status: Stopped (बंद है)");
            }
        };

    } else {
        alert("Your browser does not support Speech Recognition. Please use Google Chrome.");
        document.getElementById("statusText").innerText = "Status: Browser Not Supported";
    }

    // Start Listening Function
    function startListening() {
        if (!recognition) return;

        let option = document.getElementById("language").value;
        if (option === "en-hi") {
            recognition.lang = "en-US"; // Listens to English
        } else {
            recognition.lang = "hi-IN"; // Listens to Hindi
        }

        isListening = true;
        recognition.start();
        updateStatus("Status: Listening... Speak now (आवाज़ सुनी जा रही है...)");
    }

    // Stop Listening Function
    function stopListening() {
        if (!recognition) return;
        isListening = false;
        recognition.stop();
        updateStatus("Status: Stopped (बंद है)");
    }

    // Translation API Call Function
    async function translateText(text) {
        let option = document.getElementById("language").value;
        let source, target;

        if (option === "en-hi") {
            source = "en";
            target = "hi";
        } else {
            source = "hi";
            target = "en";
        }

        // Using MyMemory Free Translation API
        let url = `https://api.mymemory.translated.net/get?q=${encodeURIComponent(text)}&langpair=${source}|${target}`;

        try {
            let response = await fetch(url);
            let data = await response.json();

            if(data.responseData && data.responseData.translatedText) {
                document.getElementById("translatedText").innerHTML = data.responseData.translatedText;
                updateStatus("Status: Listening... (अनुवाद पूरा हुआ)");
            } else {
                document.getElementById("translatedText").innerHTML = "Translation error.";
                updateStatus("Status: Listening... (अनुवाद में त्रुटि)");
            }
        } catch (error) {
            console.error("Translation Fetch Error:", error);
            document.getElementById("translatedText").innerHTML = "Failed to fetch translation.";
            updateStatus("Status: Error fetching translation");
        }
    }

    // Print Function (Triggers A4 Print Layout)
    function printPage() {
        window.print();
    }

    // Helper function to update status
    function updateStatus(msg) {
        document.getElementById("statusText").innerText = msg;
    }
</script>
</body>
</html>
