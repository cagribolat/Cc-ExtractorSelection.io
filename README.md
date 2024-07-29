<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cc Extractor</title>
    <style>
        body {
            height: 100vh;
            margin: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            background-image: url('https://images.unsplash.com/photo-1620658839921-293f998260d5?q=80&w=2070&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D');
            background-size: cover;
            background-position: center;
            font-family: Arial, sans-serif;
        }

        #container {
            background-color: rgba(255, 255, 255, 0.9);
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
            text-align: center;
        }

        .header-text {
            font-size: 16px;
            color: #000000; /* Black color */
            font-weight: bold; /* Bold text */
            margin-bottom: 10px;
        }

        #saveButton {
            background-color: green;
            color: white;
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 10px;
        }

        #srtContent {
            width: 300px;
            height: 200px;
            margin-top: 10px;
        }

        .color-buttons {
            margin: 20px 0;
        }

        .color-buttons button {
            margin-right: 10px;
            padding: 10px 20px;
            border: 2px solid #ffffff; /* Default border color */
            cursor: pointer;
            font-weight: bold;
            background-color: #ffffff; /* Default background color */
            color: #000000; /* Default text color */
            opacity: 0.5; /* Default opacity */
            transition: all 0.3s ease; /* Transition effect */
        }

        .color-buttons button.selected {
            opacity: 1; /* Fully opaque */
            border: 3px solid #ff0000; /* Red and thick border for selected button */
        }

        .color-buttons button:hover {
            opacity: 0.8; /* Slightly more opaque on hover */
        }
    </style>
</head>
<body>
    <div id="container">
        <div class="header-text">The human idea was built through the collaboration of Python and ChatGPT.</div>
        <h1>Cc-ExtractorSelection</h1>
        <div class="color-buttons">
            <button id="whiteBtn" onclick="setColor('#ffffff', this)" style="background-color: #ffffff; color: #000000;">White</button>
            <button id="yellowBtn" onclick="setColor('#ffff00', this)" style="background-color: #ffff00; color: #000000;">Yellow</button>
        </div>

        <label for="srtFile">Select SRT File:</label>
        <input type="file" id="srtFile" accept=".srt"><br><br>

        <button id="saveButton" onclick="processSrt()">Process and Save</button><br><br>

        <textarea id="srtContent" rows="10" cols="50" readonly></textarea>
    </div>

    <script>
        let selectedColor = null; // Default no color

        function setColor(color, button) {
            selectedColor = color;
            // Remove 'selected' class from all buttons
            document.querySelectorAll('.color-buttons button').forEach(btn => {
                btn.classList.remove('selected');
                btn.style.backgroundColor = '#ffffff';
                btn.style.color = '#000000';
                btn.style.border = '2px solid #ffffff'; // Default border
                btn.style.opacity = '0.5'; // Default opacity
            });
            // Update the clicked button's style
            button.classList.add('selected');
            button.style.backgroundColor = color;
            button.style.color = (color === '#ffffff') ? '#000000' : '#ffffff'; // Text color
            button.style.border = '3px solid #ff0000'; // Red and thick border for selected button
            button.style.opacity = '1'; // Fully opaque
        }

        function processSrt() {
            var fileInput = document.getElementById('srtFile');
            var srtContent = document.getElementById('srtContent');

            if (!fileInput.files || !fileInput.files[0]) {
                alert('Please select an SRT file.');
                return;
            }

            var file = fileInput.files[0];
            var reader = new FileReader();

            reader.onload = function(event) {
                var content = event.target.result;
                var decodedContent = new TextDecoder('UTF-8').decode(content);

                var processedContent = processAndSaveSrt(decodedContent, file.name);
                srtContent.value = processedContent;
            };

            reader.readAsArrayBuffer(file);
        }

        function processAndSaveSrt(srtContent, fileName) {
            var lines = srtContent.split('\n');
            var processedLines = [];
            var currentText = '';
            var currentTimecode = '';

            for (var i = 0; i < lines.length; i++) {
                var line = lines[i].trim();

                if (line === '') {
                    continue; // Skip empty lines
                }

                if (line.match(/^\d{2}:\d{2}:\d{2},\d{3} --> \d{2}:\d{2}:\d{2},\d{3}$/)) {
                    // If the line is a timecode, update the current timecode
                    currentTimecode = line;
                } else if (line === line.toUpperCase() && line !== line.toLowerCase()) {
                    // If the line contains only uppercase letters and is not all lowercase
                    // Add it to processed lines with the current timecode
                    if (selectedColor) {
                        processedLines.push(currentTimecode + '\n' + `<font color="${selectedColor}">${line}</font>`);
                    } else {
                        processedLines.push(currentTimecode + '\n' + line);
                    }
                }
            }

            // Join processed lines
            var processedContent = processedLines.join('\n\n');

            // Save processed content
            var blob = new Blob([processedContent], {type: 'text/plain'});
            var url = URL.createObjectURL(blob);
            var a = document.createElement('a');
            a.href = url;
            // If a color is selected, add color name to the file name, otherwise just append '_forced'
            a.download = fileName.replace('.srt', '') + (selectedColor ? '_' + getColorName(selectedColor) : '') + '_forced.srt'; 
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            URL.revokeObjectURL(url);

            return processedContent;
        }

        function getColorName(hexColor) {
            const colorNames = {
                '#ffff00': 'yellow',
                '#ffffff': 'white'
            };
            return colorNames[hexColor] || 'unknown';
        }
    </script>
</body>
</html>
