# login-website
AIT login website
<!DOCTYPE html>
<html lang="ml">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AIT Game Store - Login & Admin</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>
</head>
<body class="bg-slate-900 text-white min-h-screen p-4 font-sans">

    <div class="max-w-md mx-auto bg-slate-800 p-6 rounded-2xl shadow-xl mt-10 border border-slate-700">
        <h2 class="text-2xl font-bold text-center text-cyan-400 mb-6">AIT Game Store Portal</h2>

        <!-- Login / Register Form -->
        <div id="auth-section">
            <div class="flex border-b border-slate-700 mb-4">
                <button id="tab-login" onclick="switchTab('login')" class="w-1/2 py-2 font-bold border-b-2 border-cyan-400 text-cyan-400">ലോഗിൻ</button>
                <button id="tab-register" onclick="switchTab('register')" class="w-1/2 py-2 font-bold text-slate-400">രജിസ്റ്റർ</button>
            </div>

            <input type="text" id="username" placeholder="യൂസർനെയിം" class="w-full p-3 mb-3 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-cyan-400">
            <input type="password" id="password" placeholder="പാസ്‌വേഡ്" class="w-full p-3 mb-4 bg-slate-900 border border-slate-700 rounded-lg focus:outline-none focus:border-cyan-400">
            
            <button id="auth-btn" onclick="handleAuth()" class="w-full bg-cyan-500 hover:bg-cyan-600 font-bold py-3 rounded-lg transition">പ്രവേശിക്കുക</button>
        </div>

        <!-- Admin Dashboard (Log in ചെയ്ത ശേഷം കാണുന്നത്) -->
        <div id="dashboard-section" class="hidden">
            <div class="flex justify-between items-center mb-6">
                <p class="text-green-400 font-semibold">ഹലോ, <span id="user-display"></span>!</p>
                <button onclick="logout()" class="bg-red-500 text-xs px-3 py-1 rounded">ലോഗ് ഔട്ട്</button>
            </div>

            <h3 class="text-lg font-bold mb-3 text-cyan-300">പുതിയ ഗെയിം അപ്‌ലോഡ് ചെയ്യുക</h3>
            <input type="text" id="game-title" placeholder="ഗെയിം പേര്" class="w-full p-3 mb-3 bg-slate-900 border border-slate-700 rounded-lg">
            <input type="text" id="game-url" placeholder="ഗെയിം ലിങ്ക് (Embed/Iframe URL)" class="w-full p-3 mb-3 bg-slate-900 border border-slate-700 rounded-lg">
            <input type="text" id="game-thumb" placeholder="തമ്പ്നെയിൽ ഇമേജ് ലിങ്ക്" class="w-full p-3 mb-3 bg-slate-900 border border-slate-700 rounded-lg">
            <textarea id="game-desc" placeholder="ഗെയിം വിവരം" class="w-full p-3 mb-3 bg-slate-900 border border-slate-700 rounded-lg"></textarea>
            
            <button onclick="uploadGame()" class="w-full bg-green-500 hover:bg-green-600 font-bold py-3 rounded-lg transition">ഗെയിം പബ്ലിഷ് ചെയ്യുക 🚀</button>
        </div>
    </div>

    <script>
        // 🔴 ഫയർബേസ് സെർവർ കോഡ് (നിന്റെ പ്രോജക്റ്റ് ഡാറ്റ ചേർത്തിട്ടുണ്ട്)
        const firebaseConfig = {
            apiKey: "AIzaSyCXMAdXRsyubfJFZRUoXYoYjBWjBZd7VxE",
            authDomain: "ait-game-store.firebaseapp.com",
            databaseURL: "https://ait-game-store-default-rtdb.firebaseio.com",
            projectId: "ait-game-store",
            storageBucket: "ait-game-store.firebasestorage.app",
            messagingSenderId: "146438462272",
            appId: "1:146438462272:web:92a9cea7d1383f650d78a5"
        };

        firebase.initializeApp(firebaseConfig);
        const database = firebase.database();

        let currentMode = 'login';

        function switchTab(mode) {
            currentMode = mode;
            if(mode === 'login') {
                document.getElementById('tab-login').className = "w-1/2 py-2 font-bold border-b-2 border-cyan-400 text-cyan-400";
                document.getElementById('tab-register').className = "w-1/2 py-2 font-bold text-slate-400";
                document.getElementById('auth-btn').innerText = "പ്രവേശിക്കുക";
            } else {
                document.getElementById('tab-register').className = "w-1/2 py-2 font-bold border-b-2 border-cyan-400 text-cyan-400";
                document.getElementById('tab-login').className = "w-1/2 py-2 font-bold text-slate-400";
                document.getElementById('auth-btn').innerText = "അക്കൗണ്ട് ഉണ്ടാക്കുക";
            }
        }

        function handleAuth() {
            const user = document.getElementById('username').value.trim();
            const pass = document.getElementById('password').value.trim();
            if(!user || !pass) return alert("എല്ലാ വിവരങ്ങളും നൽകുക!");

            if(currentMode === 'register') {
                database.ref('users/' + user).set({ password: pass }, (err) => {
                    if(!err) {
                        alert("അക്കൗണ്ട് സക്സസ്ഫുൾ ആയി!");
                        loginUser(user);
                    }
                });
            } else {
                database.ref('users/' + user).once('value', snapshot => {
                    if(snapshot.exists() && snapshot.val().password === pass) {
                        loginUser(user);
                    } else {
                        alert("യൂസർനെയിം അല്ലെങ്കിൽ പാസ്‌വേഡ് തെറ്റാണ്!");
                    }
                });
            }
        }

        function loginUser(user) {
            localStorage.setItem('ait_user', user);
            document.getElementById('auth-section').classList.add('hidden');
            document.getElementById('dashboard-section').classList.remove('hidden');
            document.getElementById('user-display').innerText = user;
        }

        function logout() {
            localStorage.removeItem('ait_user');
            location.reload();
        }

        function uploadGame() {
            const title = document.getElementById('game-title').value.trim();
            const url = document.getElementById('game-url').value.trim();
            const thumb = document.getElementById('game-thumb').value.trim();
            const desc = document.getElementById('game-desc').value.trim();

            if(!title || !url) return alert("ഗെയിമിന്റെ പേരും ലിങ്കും നിർബന്ധമാണ്!");

            const newGameRef = database.ref('games').push();
            newGameRef.set({
                title: title,
                url: url,
                thumb: thumb || 'https://via.placeholder.com/300x180',
                desc: desc,
                likes: 0
            }, (err) => {
                if(!err) {
                    alert("ഗെയിം വിജയകരമായി പബ്ലിഷ് ചെയ്തു!");
                    location.href = "index.html"; // ഗെയിം സ്റ്റോറിലേക്ക് മടങ്ങുന്നു
                }
            });
        }
    </script>
</body>
</html>
