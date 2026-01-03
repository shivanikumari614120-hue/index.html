<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GuruAI Vision Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap');
        body { font-family: 'Poppins', sans-serif; background: #020617; color: white; overflow: hidden; }
        .glass { background: rgba(255, 255, 255, 0.03); backdrop-filter: blur(12px); border: 1px solid rgba(255, 255, 255, 0.1); }
        .chat-box { height: calc(100vh - 240px); scroll-behavior: smooth; }
        .mic-active { animation: pulse-red 1.5s infinite; background: #ef4444 !important; }
        #preview-container img { max-height: 100px; border-radius: 10px; margin-top: 10px; border: 2px solid #4f46e5; }
    </style>
</head>
<body class="flex flex-col h-screen">

    <header class="p-4 glass flex justify-between items-center border-b border-indigo-500/20">
        <h1 class="text-xl font-bold text-indigo-400">GuruAI Vision</h1>
        <div class="flex items-center gap-2">
            <select id="voice-select" class="bg-slate-800 text-[10px] p-1 rounded border border-white/20 outline-none w-24"></select>
            <input type="checkbox" id="voice-toggle" class="hidden peer" checked>
            <div onclick="document.getElementById('voice-toggle').click()" class="w-8 h-4 bg-gray-600 rounded-full cursor-pointer relative peer-checked:bg-green-600">
                <div class="absolute bg-white w-3 h-3 rounded-full top-0.5 left-0.5 transition-all peer-checked:left-4"></div>
            </div>
        </div>
    </header>

    <div id="chat-container" class="chat-box overflow-y-auto p-4 flex flex-col gap-4">
        <div class="glass p-3 rounded-2xl max-w-[85%] self-start text-sm border-l-4 border-indigo-500">
            Aap photo bhej kar bhi sawal puch sakte hain! Camera icon par click karein.
        </div>
    </div>

    <div id="preview-container" class="px-4 hidden flex items-center gap-2">
        <span class="text-xs text-gray-400">Preview:</span>
        <div id="image-preview"></div>
        <button onclick="clearImage()" class="text-red-500 text-xs font-bold">X</button>
    </div>

    <div id="loader" class="hidden px-6 py-1 text-[10px] text-cyan-400 animate-pulse font-mono">ANALYZING_IMAGE...</div>

    <div class="p-4 bg-slate-900/80 backdrop-blur-2xl border-t border-white/10">
        <div class="flex gap-2 items-center max-w-4xl mx-auto">
            <input type="file" id="file-input" accept="image/*" class="hidden" onchange="previewFile()">
            
            <button onclick="document.getElementById('file-input').click()" class="bg-slate-700 p-3 rounded-full hover:bg-slate-600">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 9a2 2 0 012-2h.93a2 2 0 001.664-.89l.812-1.22A2 2 0 0110.07 4h3.86a2 2 0 011.664.89l.812 1.22A2 2 0 0018.07 7H19a2 2 0 012 2v9a2 2 0 01-2 2H5a2 2 0 01-2-2V9z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 13a3 3 0 11-6 0 3 3 0 016 0z" />
                </svg>
            </button>

            <button id="mic-btn" onclick="startVoice()" class="bg-indigo-600 p-3 rounded-full shadow-lg">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11a7 7 0 01-7 7m0 0a7 7 0 01-7-7m7 7v4m0 0H8m4 0h4m-4-8a3 3 0 01-3-3V5a3 3 0 116 0v6a3 3 0 01-3 3z" />
                </svg>
            </button>
            
            <input type="text" id="user-input" class="flex-1 bg-white/5 border border-white/10 rounded-xl px-4 py-3 outline-none text-sm" placeholder="Sawal likhein ya photo dein...">
            
            <button onclick="askAI()" class="bg-indigo-600 p-3 rounded-xl hover:bg-indigo-500">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7m0 0l-7 7m7-7H3" />
                </svg>
            </button>
        </div>
    </div>

    <script type="module">
        const API_KEY = "YOUR_GEMINI_API_KEY_HERE";
        const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key=${API_KEY}`;

        let base64Image = "";

        // --- File Handling ---
        window.previewFile = () => {
            const file = document.getElementById('file-input').files[0];
            const reader = new FileReader();
            reader.onloadend = () => {
                base64Image = reader.result.split(',')[1];
                document.getElementById('image-preview').innerHTML = `<img src="${reader.result}">`;
                document.getElementById('preview-container').classList.remove('hidden');
            };
            if (file) reader.readAsDataURL(file);
        };

        window.clearImage = () => {
            base64Image = "";
            document.getElementById('preview-container').classList.add('hidden');
            document.getElementById('file-input').value = "";
        };

        // --- AI Logic (Updated for Image) ---
        window.askAI = async () => {
            const promptText = document.getElementById('user-input').value.trim();
            if (!promptText && !base64Image) return;

            appendMessage(promptText || "Analyzing Image...", 'user');
            document.getElementById('user-input').value = "";
            document.getElementById('loader').classList.remove('hidden');

            const payload = {
                contents: [{
                    parts: [
                        { text: `You are an expert tutor. Solve this in Hindi. ${promptText}` }
                    ]
                }]
            };

            if (base64Image) {
                payload.contents[0].parts.push({
                    inline_data: { mime_type: "image/jpeg", data: base64Image }
                });
            }

            try {
                const response = await fetch(API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                const data = await response.json();
                const aiResponse = data.candidates[0].content.parts[0].text;
                appendMessage(aiResponse, 'ai');
                if (document.getElementById('voice-toggle').checked) speak(aiResponse);
                clearImage();
            } catch (e) {
                appendMessage("Error processing request.", 'ai');
            } finally {
                document.getElementById('loader').classList.add('hidden');
            }
        };

        // --- Baki functions (speak, startVoice, appendMessage) pichle code jaise hi rahenge ---
        // (Wahi code continue karein jo pichli chat mein voices ke liye tha)
        
        function appendMessage(text, sender) {
            const div = document.createElement('div');
            div.className = sender === 'user' ? "glass bg-indigo-500/20 p-3 rounded-2xl max-w-[85%] self-end text-sm border border-indigo-500/20 shadow-lg" : "glass p-3 rounded-2xl max-w-[85%] self-start text-sm border border-white/10 shadow-lg";
            div.innerHTML = text.replace(/\n/g, '<br>');
            document.getElementById('chat-container').appendChild(div);
            document.getElementById('chat-container').scrollTop = document.getElementById('chat-container').scrollHeight;
        }

        function speak(text) {
            window.speechSynthesis.cancel();
            const utterance = new SpeechSynthesisUtterance(text.replace(/[*#_]/g, ''));
            utterance.lang = 'hi-IN';
            window.speechSynthesis.speak(utterance);
        }

        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        if (SpeechRecognition) {
            const rec = new SpeechRecognition();
            rec.lang = 'hi-IN';
            window.startVoice = () => { rec.start(); document.getElementById('mic-btn').classList.add('mic-active'); };
            rec.onresult = (e) => { document.getElementById('user-input').value = e.results[0][0].transcript; askAI(); };
            rec.onend = () => document.getElementById('mic-btn').classList.remove('mic-active');
        }
    </script>
</body>
</html>
