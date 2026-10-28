```html
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>3D Fen Bilimleri Oto-Pilot Labirent Oyunu</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@500;700;900&family=Rajdhani:wght@500;600;700&display=swap" rel="stylesheet">
    <!-- Three.js Library -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>

    <style>
        * {
            user-select: none;
            -webkit-user-select: none;
            box-sizing: border-box;
        }
        body, html {
            font-family: 'Rajdhani', sans-serif;
            background-color: #020617;
            color: #f3f4f6;
            overflow: hidden;
            margin: 0;
            padding: 0;
            width: 100vw;
            height: 100vh;
        }
        .font-orbitron {
            font-family: 'Orbitron', sans-serif;
        }
        #webgl-container {
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
            left: 0;
            z-index: 1;
        }
        .glass-panel {
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(20px);
            -webkit-backdrop-filter: blur(20px);
            border: 1px solid rgba(56, 189, 248, 0.3);
            box-shadow: 0 8px 32px 0 rgba(0, 0, 0, 0.7);
        }
        .glass-card {
            background: rgba(30, 41, 59, 0.7);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.15);
        }
        .glow-cyan {
            box-shadow: 0 0 25px rgba(6, 182, 212, 0.6);
        }
        .glow-amber {
            box-shadow: 0 0 30px rgba(245, 158, 11, 0.9);
        }
        #warpLines {
            pointer-events: none;
            background: radial-gradient(circle, transparent 40%, rgba(6, 182, 212, 0.35) 100%);
            opacity: 0;
            transition: opacity 0.2s ease-in-out;
        }
        @keyframes turbo-pop {
            0% { transform: scale(0.5); opacity: 0; }
            50% { transform: scale(1.2); opacity: 1; }
            100% { transform: scale(1); opacity: 0; }
        }
        .animate-turbo-pop {
            animation: turbo-pop 1.5s ease-out forwards;
        }
    </style>
</head>
<body class="w-full h-full relative">

    <!-- Speed Warp Effect Overlay -->
    <div id="warpLines" class="fixed inset-0 z-20 pointer-events-none"></div>

    <!-- Turbo Earned Banner Notification -->
    <div id="turboEarnedBanner" class="fixed top-1/3 left-1/2 -translate-x-1/2 -translate-y-1/2 z-50 pointer-events-none hidden">
        <div class="glass-panel px-8 py-4 rounded-3xl border-amber-400 glow-amber text-center animate-turbo-pop">
            <h2 class="text-3xl font-orbitron font-black text-amber-300 tracking-widest flex items-center gap-3">
                <i class="fa-solid fa-bolt text-amber-400 animate-bounce"></i> TURBO KAZANILDI!
            </h2>
            <p class="text-xs font-bold text-slate-200 mt-1 uppercase tracking-wider">Doğru Cevap Katlama Gücü Yüklendi</p>
        </div>
    </div>

    <!-- 3D Canvas -->
    <div id="webgl-container"></div>

    <!-- UI Overlay Layer -->
    <div id="ui-layer" class="relative z-10 w-full h-full pointer-events-none flex flex-col justify-between p-4 md:p-6">
        
        <!-- Top HUD Bar -->
        <div id="hudTop" class="hidden justify-between items-center w-full pointer-events-auto">
            <div class="glass-panel rounded-2xl px-4 py-3 flex items-center gap-4 md:gap-6 border-cyan-500/40">
                <!-- Grade -->
                <div class="flex items-center gap-2 border-r border-slate-700/80 pr-4">
                    <i class="fa-solid fa-atom text-cyan-400 text-xl animate-spin" style="animation-duration: 8s;"></i>
                    <div>
                        <span class="text-[9px] text-slate-400 block font-bold leading-none uppercase">Sınıf</span>
                        <span id="hudGradeText" class="font-orbitron font-bold text-cyan-300 text-sm md:text-base">5. Sınıf</span>
                    </div>
                </div>

                <!-- Shield/Lives -->
                <div class="flex items-center gap-2 border-r border-slate-700/80 pr-4">
                    <i class="fa-solid fa-shield-halved text-emerald-400 text-lg"></i>
                    <div>
                        <span class="text-[9px] text-slate-400 block font-bold leading-none uppercase">Kalkan</span>
                        <div id="hudLives" class="flex gap-1 mt-0.5 text-xs text-red-500">
                            <i class="fa-solid fa-heart"></i>
                            <i class="fa-solid fa-heart"></i>
                            <i class="fa-solid fa-heart"></i>
                        </div>
                    </div>
                </div>

                <!-- Score -->
                <div class="flex items-center gap-2 border-r border-slate-700/80 pr-4">
                    <i class="fa-solid fa-trophy text-amber-400 text-lg"></i>
                    <div>
                        <span class="text-[9px] text-slate-400 block font-bold leading-none uppercase">Puan</span>
                        <span id="hudScore" class="font-orbitron font-bold text-amber-400 text-sm md:text-base">0</span>
                    </div>
                </div>

                <!-- Timer -->
                <div class="flex items-center gap-2">
                    <i class="fa-solid fa-stopwatch text-indigo-400 text-lg"></i>
                    <div>
                        <span class="text-[9px] text-slate-400 block font-bold leading-none uppercase">Süre</span>
                        <span id="hudTimer" class="font-orbitron font-bold text-indigo-300 text-sm md:text-base">00:00</span>
                    </div>
                </div>
            </div>

            <!-- Path Progress Bar -->
            <div class="glass-panel rounded-2xl px-4 py-2 flex items-center gap-3 w-44 md:w-64">
                <i class="fa-solid fa-flag-checkered text-cyan-400 text-sm"></i>
                <div class="w-full bg-slate-800 h-2.5 rounded-full overflow-hidden border border-slate-700">
                    <div id="hudProgressBar" class="bg-gradient-to-r from-cyan-500 via-amber-400 to-indigo-500 h-full w-0 transition-all duration-200"></div>
                </div>
                <span id="hudProgressText" class="text-xs font-orbitron text-cyan-300 font-bold">%0</span>
            </div>
        </div>

        <!-- Speedometer & Turbo Control Overlay -->
        <div id="speedHUD" class="hidden pointer-events-auto self-end flex-col items-end gap-3 mb-2">
            <!-- Digital Speedometer -->
            <div class="glass-panel px-4 py-2 rounded-2xl border-cyan-500/50 flex items-center gap-3">
                <i class="fa-solid fa-gauge-high text-cyan-400 text-2xl animate-pulse"></i>
                <div>
                    <span class="text-[9px] text-slate-400 block font-bold leading-none uppercase">ANLIK HIZ</span>
                    <span id="speedValue" class="font-orbitron font-black text-cyan-300 text-2xl tracking-wider">140</span>
                    <span class="text-[10px] text-slate-400 font-bold">KM/S</span>
                </div>
            </div>

            <!-- Turbo Boost Charge Gauge & Trigger Button -->
            <div class="glass-panel p-3 rounded-2xl border-amber-500/60 flex flex-col gap-2 w-64">
                <div class="flex justify-between items-center text-xs font-orbitron font-bold">
                    <span class="text-amber-400 flex items-center gap-1">
                        <i class="fa-solid fa-bolt text-amber-400"></i> TURBO DEPO
                    </span>
                    <span id="turboChargesText" class="text-amber-300 font-black">0 HAK</span>
                </div>

                <div class="w-full bg-slate-900 h-3 rounded-full p-0.5 border border-slate-700 overflow-hidden">
                    <div id="turboGaugeBar" class="bg-gradient-to-r from-amber-500 to-yellow-300 h-full w-0 transition-all duration-300 rounded-full shadow-lg shadow-amber-500/50"></div>
                </div>

                <button id="boostBtn" onmousedown="activateBoost(true)" onmouseup="activateBoost(false)" ontouchstart="activateBoost(true)" ontouchend="activateBoost(false)" class="w-full py-2.5 bg-slate-800 border border-slate-700 text-slate-500 rounded-xl font-orbitron font-bold text-xs flex items-center justify-center gap-2 cursor-not-allowed transition-all">
                    <i class="fa-solid fa-rocket text-sm"></i>
                    <span id="boostBtnText">KİLİTLİ (SORU ÇÖZ)</span>
                </button>
            </div>
        </div>

        <div></div>
    </div>

    <!-- 1. START & GRADE SELECTION MODAL -->
    <div id="startModal" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-slate-950/90 backdrop-blur-lg">
        <div class="glass-panel rounded-3xl p-6 md:p-10 max-w-2xl w-full text-center border-cyan-500/40 relative overflow-hidden shadow-2xl">
            <div class="w-20 h-20 bg-gradient-to-tr from-cyan-500 via-teal-400 to-indigo-600 rounded-3xl mx-auto flex items-center justify-center text-4xl text-white shadow-xl shadow-cyan-500/30 mb-4 animate-bounce">
                🚀
            </div>

            <h1 class="text-3xl md:text-5xl font-orbitron font-black text-transparent bg-clip-text bg-gradient-to-r from-cyan-400 via-teal-200 to-indigo-400 mb-2 tracking-wider">
                FEN LABİRENTİ 3D
            </h1>
            <p class="text-slate-300 text-sm md:text-base max-w-lg mx-auto mb-6 font-medium">
                Otonom uzay dronu labirentte ışık hızında kayar! Soruları doğru yanıtladıkça kapıları aç ve <b class="text-amber-400">TURBO BOOST</b> kazan!
            </p>

            <h2 class="text-xs font-orbitron uppercase tracking-widest text-cyan-400 mb-4 font-bold">MEB Sınıf Seviyenizi Seçin:</h2>

            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-6">
                <button onclick="startGame(5)" class="glass-card hover:border-emerald-400 p-5 rounded-2xl flex flex-col items-center group transition-all duration-300 transform hover:-translate-y-1">
                    <span class="text-4xl font-orbitron font-black text-emerald-400 mb-1 group-hover:scale-110 transition-transform">5</span>
                    <span class="text-xs font-bold text-slate-300">SINIF</span>
                    <span class="text-[10px] text-slate-400 mt-1">7 Ünite</span>
                </button>
                <button onclick="startGame(6)" class="glass-card hover:border-blue-400 p-5 rounded-2xl flex flex-col items-center group transition-all duration-300 transform hover:-translate-y-1">
                    <span class="text-4xl font-orbitron font-black text-blue-400 mb-1 group-hover:scale-110 transition-transform">6</span>
                    <span class="text-xs font-bold text-slate-300">SINIF</span>
                    <span class="text-[10px] text-slate-400 mt-1">7 Ünite</span>
                </button>
                <button onclick="startGame(7)" class="glass-card hover:border-purple-400 p-5 rounded-2xl flex flex-col items-center group transition-all duration-300 transform hover:-translate-y-1">
                    <span class="text-4xl font-orbitron font-black text-purple-400 mb-1 group-hover:scale-110 transition-transform">7</span>
                    <span class="text-xs font-bold text-slate-300">SINIF</span>
                    <span class="text-[10px] text-slate-400 mt-1">7 Ünite</span>
                </button>
                <button onclick="startGame(8)" class="glass-card hover:border-amber-400 p-5 rounded-2xl flex flex-col items-center group transition-all duration-300 transform hover:-translate-y-1">
                    <span class="text-4xl font-orbitron font-black text-amber-400 mb-1 group-hover:scale-110 transition-transform">8</span>
                    <span class="text-xs font-bold text-slate-300">SINIF</span>
                    <span class="text-[10px] text-slate-400 mt-1">7 Ünite</span>
                </button>
            </div>
        </div>
    </div>

    <!-- 2. SCIENCE QUESTION MODAL -->
    <div id="questionModal" class="fixed inset-0 z-40 hidden items-center justify-center p-4 bg-slate-950/85 backdrop-blur-md">
        <div class="glass-panel rounded-3xl p-6 md:p-8 max-w-xl w-full border-cyan-500/50 relative shadow-2xl">
            <div class="flex items-center justify-between pb-3 border-b border-slate-700/80 mb-4">
                <span id="modalTopic" class="text-xs font-orbitron font-bold text-cyan-300 bg-cyan-950/80 px-3 py-1 rounded-full border border-cyan-500/40">
                    ÜNİTE KONUSU
                </span>
                <span class="text-xs font-bold text-amber-400 flex items-center gap-1.5 animate-pulse">
                    <i class="fa-solid fa-lock text-xs"></i> BİLİM KAPISI
                </span>
            </div>

            <h3 id="modalQuestionText" class="text-base md:text-lg font-semibold text-slate-100 mb-6 leading-relaxed">
                Soru yükleniyor...
            </h3>

            <div id="modalOptions" class="grid grid-cols-1 gap-3 mb-4"></div>

            <div id="modalFeedback" class="hidden p-3 rounded-xl text-center text-xs md:text-sm font-bold"></div>
        </div>
    </div>

    <!-- 3. RESULT MODAL -->
    <div id="resultModal" class="fixed inset-0 z-50 hidden items-center justify-center p-4 bg-slate-950/90 backdrop-blur-md">
        <div class="glass-panel rounded-3xl p-8 max-w-md w-full text-center border-slate-700 shadow-2xl">
            <div id="resultIcon" class="text-6xl mb-3">🏆</div>
            <h2 id="resultTitle" class="text-3xl font-orbitron font-black text-white mb-1">TEBRİKLER!</h2>
            <p id="resultSubtitle" class="text-slate-300 text-sm mb-6">3D Fen Macerasını başarıyla tamamladın!</p>

            <div class="bg-slate-900/90 rounded-2xl p-4 border border-slate-800 space-y-3 mb-6 text-sm">
                <div class="flex justify-between items-center">
                    <span class="text-slate-400">Toplam Puan:</span>
                    <span id="resScore" class="font-orbitron font-bold text-amber-400 text-lg">0</span>
                </div>
                <div class="flex justify-between items-center">
                    <span class="text-slate-400">Doğru / Yanlış:</span>
                    <span id="resAccuracy" class="font-bold text-emerald-400">0 / 0</span>
                </div>
                <div class="flex justify-between items-center">
                    <span class="text-slate-400">Tamamlama Süresi:</span>
                    <span id="resTime" class="font-orbitron font-bold text-indigo-300">00:00</span>
                </div>
            </div>

            <button onclick="resetToHome()" class="w-full py-4 bg-gradient-to-r from-cyan-500 to-blue-600 hover:from-cyan-400 hover:to-blue-500 font-orbitron font-bold text-white rounded-2xl shadow-lg shadow-cyan-500/25 transition-all">
                YENİDEN BAŞLA
            </button>
        </div>
    </div>

    <!-- JavaScript Engine & Game Logic -->
    <script>
        /* --------------------------------------------------------------------------
           1. MEB QUESTION BANK (GRADES 5, 6, 7, 8)
           -------------------------------------------------------------------------- */
        const QUESTION_BANK = {
            5: [
                { topic: "Güneş, Dünya ve Ay", q: "Güneş, Dünya ve Ay'ın hacimsel büyüklük sıralaması büyükten küçüğe nasıldır?", options: ["Güneş > Dünya > Ay", "Dünya > Güneş > Ay", "Ay > Dünya > Güneş", "Güneş > Ay > Dünya"], correct: 0 },
                { topic: "Canlılar Dünyası", q: "Kendi besinini fotosentez ile üretebilen üretici canlı hangisidir?", options: ["Yeşil Bitkiler", "Şapkalı Mantarlar", "Hayvanlar", "Maya Mantarları"], correct: 0 },
                { topic: "Kuvvetin Ölçülmesi", q: "Kuvvetin birimi ve ölçüm aleti hangi seçenekte doğru eşleştirilmiştir?", options: ["Newton - Dinamometre", "Gram - Dereceli Silindir", "Joule - Termometre", "Pascal - Terazi"], correct: 0 },
                { topic: "Madde ve Değişim", q: "Sıvı bir maddenin ısı vererek katı hale geçmesi olayına ne ad verilir?", options: ["Donma", "Erime", "Buharlaşma", "Süblimleşme"], correct: 0 },
                { topic: "Işığın Yayılması", q: "Işık saydam olmayan (opak) bir engel ile karşılaştığında arkasında ne oluşur?", options: ["Tam Gölge", "Yansıma", "Gökkuşağı", "Işık Kırılması"], correct: 0 }
            ],
            6: [
                { topic: "Güneş Sistemi", q: "Güneş sistemindeki en küçük ve Güneş'e en yakın gezegen hangisidir?", options: ["Merkür", "Mars", "Venüs", "Neptün"], correct: 0 },
                { topic: "Vücudumuzdaki Sistemler", q: "Sindirilmiş besinlerin kana emiliminin gerçekleştiği ana organ hangisidir?", options: ["İnce Bağırsak", "Mide", "Ağız", "Kalın Bağırsak"], correct: 0 },
                { topic: "Kuvvet ve Hareket", q: "Bir cisme etki eden bileşke kuvvet sıfır ise cisim için ne söylenebilir?", options: ["Dengelenmiş kuvvetler etkisindedir", "Hızlanan hareket yapar", "Yön değiştirir", "Yavaşlayan hareket yapar"], correct: 0 },
                { topic: "Madde ve Isı", q: "Binalarda ısı kaybını en aza indirmek için yapılan uygulamaya ne ad verilir?", options: ["Isı Yalıtımı", "Isı İletimi", "Konveksiyon", "Radyasyon"], correct: 0 },
                { topic: "Ses ve Özellikleri", q: "Ses aşağıdaki ortamlardan hangisinde YAYILMAZ?", options: ["Uzay Boşluğunda", "Deniz Suyunda", "Demir Çubukta", "Hava Ortamında"], correct: 0 }
            ],
            7: [
                { topic: "Güneş Sistemi ve Ötesi", q: "Gökyüzündeki takımyıldızlarını ve gök cisimlerini inceleyen gözlem merkezlerine ne denir?", options: ["Rasathane (Gözlemevi)", "Uzay İstasyonu", "Yapay Uydu", "Laboratuvar"], correct: 0 },
                { topic: "Hücre ve Bölünmeler", q: "Kromozom sayısının yarıya indiği ve üreme hücrelerinin oluştuğu bölünme hangisidir?", options: ["Mayoz Bölünme", "Mitoz Bölünme", "Döllenme", "Yenilenme"], correct: 0 },
                { topic: "Kuvvet ve Enerji", q: "Yüksekte duran bir cismin konumundan dolayı sahip olduğu enerji türü nedir?", options: ["Çekim Potansiyel Enerjisi", "Kinetik Enerji", "Esneklik Enerjisi", "Isı Enerjisi"], correct: 0 },
                { topic: "Saf Madde ve Karışımlar", q: "Sembollerle gösterilen ve aynı tür atomlardan oluşan saf maddelere ne ad verilir?", options: ["Element", "Bileşik", "Homojen Karışım", "Süspansiyon"], correct: 0 },
                { topic: "Işığın Etkileşimi", q: "Işığın az yoğun ortamdan çok yoğun ortama geçerken doğrultu değiştirmesine ne denir?", options: ["Işığın Kırılması", "Tam Yansıma", "Soğurulma", "Gölge Oluşumu"], correct: 0 }
            ],
            8: [
                { topic: "Mevsimler ve İklim", q: "Dünya'da mevsimlerin oluşmasının temel sebebi aşağıdakilerden hangisidir?", options: ["Eksen eğikliği ve Güneş etrafında dolanma", "Dünya'nın kendi etrafında dönmesi", "Güneş'in sıcaklığının değişmesi", "Ay'ın evreleri"], correct: 0 },
                { topic: "DNA ve Genetik Kod", q: "DNA'nın en küçük yapı birimi (nükleotid) içinde hangisi BULUNMAZ?", options: ["Protein Kılıf", "Deoksiriboz Şekeri", "Fosfat Grubu", "Organik Baz"], correct: 0 },
                { topic: "Basınç", q: "Sıvı basıncı aşağıdakilerden hangisine bağlı DEĞİLDİR?", options: ["Kabın şekline ve sıvı miktarına", "Sıvının derinliğine (h)", "Sıvının yoğunluğuna (d)", "Yerçekimi ivmesine"], correct: 0 },
                { topic: "Madde ve Endüstri", q: "pH değeri 2 olan bir kimyasal madde için hangisi söylenebilir?", options: ["Kuvvetli bir Asittir", "Kuvvetli bir Bazdır", "Nötr bir maddedir", "Tuzlu sudur"], correct: 0 },
                { topic: "Basit Makineler", q: "Hareketli makaralarda kuvvet kazancı ve yol kaybı oranı nedir?", options: ["Kuvvetten 2 kat kazanç, yoldan 2 kat kayıp", "Kuvvetten kayıp, yoldan kazanç", "Kazanç veya kayıp yoktur", "Kuvvetten 4 kat kazanç"], correct: 0 }
            ]
        };

        /* --------------------------------------------------------------------------
           2. WEB AUDIO SYNTHESIZER
           -------------------------------------------------------------------------- */
        const AudioFX = {
            ctx: null,
            engineOsc: null,
            engineGain: null,
            init() {
                if (!this.ctx) {
                    this.ctx = new (window.AudioContext || window.webkitAudioContext)();
                }
                if (this.ctx.state === 'suspended') {
                    this.ctx.resume();
                }
            },
            startEngine() {
                this.init();
                if (this.engineOsc) return;
                try {
                    this.engineOsc = this.ctx.createOscillator();
                    this.engineGain = this.ctx.createGain();
                    this.engineOsc.type = 'sawtooth';
                    this.engineOsc.frequency.setValueAtTime(65, this.ctx.currentTime);
                    this.engineGain.gain.setValueAtTime(0.04, this.ctx.currentTime);
                    this.engineOsc.connect(this.engineGain);
                    this.engineGain.connect(this.ctx.destination);
                    this.engineOsc.start();
                } catch(e) {}
            },
            setEnginePitch(mult) {
                if (this.engineOsc && this.ctx) {
                    this.engineOsc.frequency.linearRampToValueAtTime(60 + mult * 55, this.ctx.currentTime + 0.1);
                }
            },
            playCorrect() {
                this.init();
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(523.25, now);
                osc.frequency.exponentialRampToValueAtTime(1046.50, now + 0.4);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.4);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.4);
            },
            playWrong() {
                this.init();
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'sawtooth';
                osc.frequency.setValueAtTime(220, now);
                osc.frequency.linearRampToValueAtTime(110, now + 0.35);
                gain.gain.setValueAtTime(0.3, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.35);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.35);
            },
            playBoost() {
                this.init();
                const now = this.ctx.currentTime;
                const osc = this.ctx.createOscillator();
                const gain = this.ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(200, now);
                osc.frequency.exponentialRampToValueAtTime(800, now + 0.5);
                gain.gain.setValueAtTime(0.4, now);
                gain.gain.exponentialRampToValueAtTime(0.01, now + 0.5);
                osc.connect(gain);
                gain.connect(this.ctx.destination);
                osc.start(now);
                osc.stop(now + 0.5);
            }
        };

        /* --------------------------------------------------------------------------
           3. GAME STATE & VARIABLES
           -------------------------------------------------------------------------- */
        let scene, camera, renderer;
        let selectedGrade = 5;
        let score = 0;
        let lives = 3;
        let correctAnswers = 0;
        let wrongAnswers = 0;
        let gameActive = false;
        let animationFrameId = null;
        let timerSeconds = 0;
        let timerInterval = null;

        // Spline & Motion Controls
        let curvePath;
        let pathProgress = 0; // 0.0 to 1.0
        let baseSpeed = 0.12; // Fast cruise base speed
        let boostMultiplier = 1.0;
        let turboCharges = 0; // EARNED via correct answers
        let isBoosting = false;
        let currentBankingAngle = 0;

        let gateCheckpoints = [];
        let currentGateIndex = 0;
        let isWaitingAtGate = false;

        let activeParticleSystems = [];
        let dnaHelixes = [];
        let atomObjects = [];
        let droneMesh;
        let engineThrusterLight;
        let lastTime = performance.now();

        /* --------------------------------------------------------------------------
           4. THREE.JS BUILDERS
           -------------------------------------------------------------------------- */
        function createSciFiGridTexture() {
            const canvas = document.createElement('canvas');
            canvas.width = 256;
            canvas.height = 256;
            const ctx = canvas.getContext('2d');
            ctx.fillStyle = '#090d16';
            ctx.fillRect(0, 0, 256, 256);

            ctx.strokeStyle = '#0284c7';
            ctx.lineWidth = 3;
            ctx.strokeRect(5, 5, 246, 246);

            ctx.fillStyle = '#38bdf8';
            ctx.fillRect(10, 10, 10, 10);
            ctx.fillRect(236, 10, 10, 10);
            ctx.fillRect(10, 236, 10, 10);
            ctx.fillRect(236, 236, 10, 10);

            const texture = new THREE.CanvasTexture(canvas);
            texture.wrapS = THREE.RepeatWrapping;
            texture.wrapT = THREE.RepeatWrapping;
            return texture;
        }

        function initThreeScene() {
            const container = document.getElementById('webgl-container');
            container.innerHTML = '';

            scene = new THREE.Scene();
            scene.background = new THREE.Color(0x020617);
            scene.fog = new THREE.FogExp2(0x020617, 0.012);

            camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 1000);

            renderer = new THREE.WebGLRenderer({ antialias: true, powerPreference: "high-performance" });
            renderer.setSize(window.innerWidth, window.innerHeight);
            renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
            renderer.shadowMap.enabled = true;
            container.appendChild(renderer.domElement);

            const ambient = new THREE.AmbientLight(0x38bdf8, 0.7);
            scene.add(ambient);

            const sunLight = new THREE.DirectionalLight(0xffffff, 1.2);
            sunLight.position.set(40, 80, 40);
            scene.add(sunLight);

            buildSplineTrack();
            buildMazeEnvironment();
            createAutonomousDrone();
            createSpaceDust();

            window.addEventListener('resize', onWindowResize);
        }

        function buildSplineTrack() {
            const points = [
                new THREE.Vector3(0, 3, 0),
                new THREE.Vector3(0, 3, 50),
                new THREE.Vector3(40, 3, 80),
                new THREE.Vector3(80, 3, 80),
                new THREE.Vector3(100, 3, 30),
                new THREE.Vector3(60, 3, -10),
                new THREE.Vector3(60, 3, -60),
                new THREE.Vector3(-30, 3, -60),
                new THREE.Vector3(-70, 3, -10),
                new THREE.Vector3(-70, 3, 50),
                new THREE.Vector3(-20, 3, 100),
                new THREE.Vector3(30, 3, 140),
                new THREE.Vector3(30, 3, 180)
            ];

            curvePath = new THREE.CatmullRomCurve3(points);

            const tubeGeo = new THREE.TubeGeometry(curvePath, 160, 0.4, 8, false);
            const tubeMat = new THREE.MeshBasicMaterial({ color: 0x06b6d4, wireframe: true });
            const tubeMesh = new THREE.Mesh(tubeGeo, tubeMat);
            scene.add(tubeMesh);

            const questions = [...QUESTION_BANK[selectedGrade]].sort(() => Math.random() - 0.5);
            gateCheckpoints = [
                { t: 0.18, q: questions[0], cleared: false, meshGroup: null },
                { t: 0.38, q: questions[1], cleared: false, meshGroup: null },
                { t: 0.58, q: questions[2], cleared: false, meshGroup: null },
                { t: 0.78, q: questions[3], cleared: false, meshGroup: null },
                { t: 0.92, q: questions[4], cleared: false, meshGroup: null }
            ];

            gateCheckpoints.forEach((gate) => {
                const pos = curvePath.getPointAt(gate.t);
                const tangent = curvePath.getTangentAt(gate.t);
                gate.meshGroup = create3DHologramGate(pos, tangent);
            });
        }

        function create3DHologramGate(pos, tangent) {
            const group = new THREE.Group();
            group.position.copy(pos);

            const frameGeo = new THREE.TorusGeometry(5, 0.3, 16, 32);
            const frameMat = new THREE.MeshStandardMaterial({ color: 0x0284c7, metalness: 0.9 });
            const frame = new THREE.Mesh(frameGeo, frameMat);
            group.add(frame);

            const shieldGeo = new THREE.CircleGeometry(4.8, 32);
            const shieldMat = new THREE.MeshBasicMaterial({
                color: 0x38bdf8,
                transparent: true,
                opacity: 0.55,
                side: THREE.DoubleSide
            });
            const shield = new THREE.Mesh(shieldGeo, shieldMat);
            group.add(shield);

            const angle = Math.atan2(tangent.x, tangent.z);
            group.rotation.y = angle;

            scene.add(group);
            return { group, shield };
        }

        function buildMazeEnvironment() {
            const gridTex = createSciFiGridTexture();
            gridTex.repeat.set(40, 40);

            const floorGeo = new THREE.PlaneGeometry(600, 600);
            const floorMat = new THREE.MeshStandardMaterial({ map: gridTex, roughness: 0.3, metalness: 0.8 });
            const floor = new THREE.Mesh(floorGeo, floorMat);
            floor.rotation.x = -Math.PI / 2;
            scene.add(floor);

            for (let i = 0; i < 50; i++) {
                const t = Math.random();
                const pt = curvePath.getPointAt(t);
                const offset = (Math.random() > 0.5 ? 1 : -1) * (14 + Math.random() * 8);

                const colGeo = new THREE.BoxGeometry(5, 20, 5);
                const colMat = new THREE.MeshStandardMaterial({ color: 0x0f172a, metalness: 0.8, roughness: 0.2 });
                const col = new THREE.Mesh(colGeo, colMat);
                col.position.set(pt.x + offset, 10, pt.z + offset);
                scene.add(col);
            }

            for (let i = 0; i < 6; i++) {
                const t = (i + 1) / 7;
                const pt = curvePath.getPointAt(t);
                createFloatingDNA(pt.x + (i % 2 === 0 ? 12 : -12), 8, pt.z);
            }
        }

        function createFloatingDNA(x, y, z) {
            const group = new THREE.Group();
            group.position.set(x, y, z);

            const geom = new THREE.SphereGeometry(0.35, 8, 8);
            const mat1 = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            const mat2 = new THREE.MeshBasicMaterial({ color: 0xf59e0b });

            for (let i = 0; i < 16; i++) {
                const angle = i * 0.4;
                const s1 = new THREE.Mesh(geom, mat1);
                s1.position.set(Math.sin(angle) * 1.5, i * 0.4 - 3, Math.cos(angle) * 1.5);
                group.add(s1);

                const s2 = new THREE.Mesh(geom, mat2);
                s2.position.set(Math.sin(angle + Math.PI) * 1.5, i * 0.4 - 3, Math.cos(angle + Math.PI) * 1.5);
                group.add(s2);
            }

            dnaHelixes.push(group);
            scene.add(group);
        }

        function createAutonomousDrone() {
            const group = new THREE.Group();

            const bodyGeo = new THREE.SphereGeometry(1.2, 16, 16);
            const bodyMat = new THREE.MeshStandardMaterial({ color: 0x0ea5e9, emissive: 0x0284c7, emissiveIntensity: 0.6, metalness: 0.9 });
            const body = new THREE.Mesh(bodyGeo, bodyMat);
            group.add(body);

            const visorGeo = new THREE.CylinderGeometry(0.7, 0.7, 0.4, 16);
            const visorMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });
            const visor = new THREE.Mesh(visorGeo, visorMat);
            visor.rotation.x = Math.PI / 2;
            visor.position.set(0, 0.2, 0.7);
            group.add(visor);

            engineThrusterLight = new THREE.PointLight(0x38bdf8, 4, 25);
            engineThrusterLight.position.set(0, 0, -1.2);
            group.add(engineThrusterLight);

            droneMesh = group;
            scene.add(droneMesh);
        }

        function createSpaceDust() {
            const count = 300;
            const geo = new THREE.BufferGeometry();
            const pos = new Float32Array(count * 3);

            for (let i = 0; i < count * 3; i += 3) {
                pos[i] = (Math.random() - 0.5) * 300;
                pos[i + 1] = Math.random() * 40;
                pos[i + 2] = (Math.random() - 0.5) * 300;
            }

            geo.setAttribute('position', new THREE.BufferAttribute(pos, 3));
            const mat = new THREE.PointsMaterial({ color: 0x38bdf8, size: 0.6, transparent: true, opacity: 0.6 });
            const particles = new THREE.Points(geo, mat);
            scene.add(particles);
        }

        function createShatterParticles(pos) {
            const group = new THREE.Group();
            group.position.copy(pos);

            const pGeo = new THREE.BoxGeometry(0.3, 0.3, 0.3);
            const pMat = new THREE.MeshBasicMaterial({ color: 0x38bdf8 });

            for (let i = 0; i < 25; i++) {
                const p = new THREE.Mesh(pGeo, pMat);
                p.userData = {
                    vel: new THREE.Vector3(
                        (Math.random() - 0.5) * 15,
                        (Math.random() - 0.5) * 15,
                        (Math.random() - 0.5) * 15
                    )
                };
                group.add(p);
            }

            activeParticleSystems.push({ group, age: 0 });
            scene.add(group);
        }

        /* --------------------------------------------------------------------------
           5. TURBO BOOST SYSTEM
           -------------------------------------------------------------------------- */
        function updateTurboUI() {
            const gaugeBar = document.getElementById('turboGaugeBar');
            const chargesText = document.getElementById('turboChargesText');
            const boostBtn = document.getElementById('boostBtn');
            const boostBtnText = document.getElementById('boostBtnText');

            if (turboCharges > 0) {
                gaugeBar.style.width = '100%';
                chargesText.innerText = `${turboCharges} HAK`;
                boostBtn.className = "w-full py-2.5 bg-gradient-to-r from-amber-500 to-yellow-400 hover:from-amber-400 hover:to-yellow-300 text-slate-950 font-orbitron font-black text-xs flex items-center justify-center gap-2 rounded-xl cursor-pointer shadow-lg shadow-amber-500/40 active:scale-95 transition-all";
                boostBtnText.innerText = "TURBO BOOST (SPACE)";
            } else {
                gaugeBar.style.width = '0%';
                chargesText.innerText = `0 HAK`;
                boostBtn.className = "w-full py-2.5 bg-slate-800 border border-slate-700 text-slate-500 rounded-xl font-orbitron font-bold text-xs flex items-center justify-center gap-2 cursor-not-allowed transition-all";
                boostBtnText.innerText = "KİLİTLİ (DOĞRU CEVAP VER)";
            }
        }

        function grantTurboCharge() {
            turboCharges++;
            updateTurboUI();

            // Show Turbo Earned Screen Popup
            const banner = document.getElementById('turboEarnedBanner');
            banner.classList.remove('hidden');
            setTimeout(() => {
                banner.classList.add('hidden');
            }, 1400);
        }

        function activateBoost(enable) {
            const warpOverlay = document.getElementById('warpLines');

            if (enable && turboCharges > 0 && !isBoosting && gameActive && !isWaitingAtGate) {
                isBoosting = true;
                turboCharges--;
                boostMultiplier = 2.5; // Hyper speed!
                AudioFX.playBoost();
                AudioFX.setEnginePitch(2.0);

                if (warpOverlay) warpOverlay.style.opacity = '1';
                if (engineThrusterLight) engineThrusterLight.color.setHex(0xf59e0b);

                updateTurboUI();

                // Boost Duration: 3.5 Seconds
                setTimeout(() => {
                    isBoosting = false;
                    boostMultiplier = 1.0;
                    if (warpOverlay) warpOverlay.style.opacity = '0';
                    if (engineThrusterLight) engineThrusterLight.color.setHex(0x38bdf8);
                    AudioFX.setEnginePitch(1.0);
                }, 3500);
            }
        }

        // Keyboard Space Trigger
        window.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameActive) {
                e.preventDefault();
                activateBoost(true);
            }
        });

        /* --------------------------------------------------------------------------
           6. MAIN ANIMATION LOOP
           -------------------------------------------------------------------------- */
        function animate() {
            animationFrameId = requestAnimationFrame(animate);

            const now = performance.now();
            const delta = Math.min((now - lastTime) / 1000, 0.1);
            lastTime = now;

            if (gameActive && !isWaitingAtGate) {
                const effectiveSpeed = baseSpeed * boostMultiplier;
                pathProgress += (effectiveSpeed * delta) / 10;
                if (pathProgress > 1.0) pathProgress = 1.0;

                // Update Speedometer HUD
                const currentDisplaySpeed = Math.floor(effectiveSpeed * 1200);
                document.getElementById('speedValue').innerText = currentDisplaySpeed;

                // Dynamic FOV Warp Effect
                const targetFOV = isBoosting ? 85 : 65;
                camera.fov = THREE.MathUtils.lerp(camera.fov, targetFOV, 0.1);
                camera.updateProjectionMatrix();

                // Update HUD Progress Bar
                const pct = Math.floor(pathProgress * 100);
                document.getElementById('hudProgressBar').style.width = `${pct}%`;
                document.getElementById('hudProgressText').innerText = `%${pct}`;

                // Gate Approach Checkpoint
                if (currentGateIndex < gateCheckpoints.length) {
                    const nextGate = gateCheckpoints[currentGateIndex];
                    if (pathProgress >= nextGate.t && !nextGate.cleared) {
                        isWaitingAtGate = true;
                        isBoosting = false;
                        boostMultiplier = 1.0;
                        document.getElementById('warpLines').style.opacity = '0';
                        AudioFX.setEnginePitch(0.2);
                        triggerQuestionModal(nextGate);
                    }
                }

                if (pathProgress >= 1.0) {
                    winGame();
                }
            }

            // Smooth Drone & Camera Banking
            if (curvePath && droneMesh) {
                const currentPos = curvePath.getPointAt(pathProgress);
                const lookAheadPos = curvePath.getPointAt(Math.min(pathProgress + 0.02, 1.0));
                const tangent = curvePath.getTangentAt(pathProgress);

                droneMesh.position.copy(currentPos);
                droneMesh.lookAt(lookAheadPos);

                const nextTangent = curvePath.getTangentAt(Math.min(pathProgress + 0.03, 1.0));
                const turnAngle = tangent.x * nextTangent.z - tangent.z * nextTangent.x;
                currentBankingAngle = THREE.MathUtils.lerp(currentBankingAngle, turnAngle * 3.5, 0.1);
                droneMesh.rotation.z = -currentBankingAngle;

                const camOffset = new THREE.Vector3(0, 3.8, -8.5).applyQuaternion(droneMesh.quaternion);
                const targetCamPos = currentPos.clone().add(camOffset);

                camera.position.lerp(targetCamPos, 0.18);
                camera.lookAt(currentPos.clone().add(new THREE.Vector3(0, 1, 4).applyQuaternion(droneMesh.quaternion)));
            }

            // Rotate Decorative Objects
            dnaHelixes.forEach(dna => dna.rotation.y += 0.04);

            // Update Particle Systems
            activeParticleSystems.forEach((ps, idx) => {
                ps.age += delta;
                ps.group.children.forEach(p => p.position.addScaledVector(p.userData.vel, delta));
                if (ps.age > 0.8) {
                    scene.remove(ps.group);
                    activeParticleSystems.splice(idx, 1);
                }
            });

            renderer.render(scene, camera);
        }

        /* --------------------------------------------------------------------------
           7. QUESTION & QUIZ SYSTEM
           -------------------------------------------------------------------------- */
        function triggerQuestionModal(gate) {
            const modal = document.getElementById('questionModal');
            document.getElementById('modalTopic').innerText = gate.q.topic.toUpperCase();
            document.getElementById('modalQuestionText').innerText = gate.q.q;

            const optionsDiv = document.getElementById('modalOptions');
            optionsDiv.innerHTML = '';

            const feedback = document.getElementById('modalFeedback');
            feedback.classList.add('hidden');

            gate.q.options.forEach((opt, idx) => {
                const btn = document.createElement('button');
                btn.className = "w-full p-4 bg-slate-800/90 hover:bg-cyan-950/80 border border-slate-700 hover:border-cyan-400/80 rounded-xl text-left font-semibold text-sm text-slate-100 transition-all flex items-center gap-3";
                btn.innerHTML = `<span class="w-7 h-7 rounded-lg bg-slate-900 border border-slate-700 flex items-center justify-center font-orbitron font-bold text-xs text-cyan-400">${String.fromCharCode(65 + idx)}</span> <span>${opt}</span>`;
                btn.onclick = () => answerQuestion(idx, gate.q.correct, btn, gate);
                optionsDiv.appendChild(btn);
            });

            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        function answerQuestion(selectedIdx, correctIdx, btnEl, gateObj) {
            const allBtns = document.getElementById('modalOptions').children;
            for (let b of allBtns) b.disabled = true;

            const feedback = document.getElementById('modalFeedback');
            feedback.classList.remove('hidden');

            if (selectedIdx === correctIdx) {
                AudioFX.playCorrect();
                score += 250;
                correctAnswers++;
                
                // Grant Turbo Boost for Correct Answer!
                grantTurboCharge();

                btnEl.className = "w-full p-4 bg-emerald-600/80 border border-emerald-400 rounded-xl text-left font-bold text-sm text-white flex items-center gap-3";
                feedback.className = "p-3 rounded-xl text-center text-xs md:text-sm font-bold bg-emerald-500/20 text-emerald-300 border border-emerald-500/40";
                feedback.innerText = "⚡ HARİKA! Lazer kapı parçalandı ve TURBO HAKKI kazandın!";

                createShatterParticles(gateObj.meshGroup.group.position);
                gateObj.meshGroup.group.remove(gateObj.meshGroup.shield);
                gateObj.cleared = true;
                currentGateIndex++;

                setTimeout(() => {
                    closeModal('questionModal');
                    isWaitingAtGate = false;
                    AudioFX.setEnginePitch(1.0);
                    updateHUD();
                }, 700);

            } else {
                AudioFX.playWrong();
                lives--;
                wrongAnswers++;
                btnEl.className = "w-full p-4 bg-red-600/80 border border-red-400 rounded-xl text-left font-bold text-sm text-white flex items-center gap-3";
                allBtns[correctIdx].className = "w-full p-4 bg-emerald-600/80 border border-emerald-400 rounded-xl text-left font-bold text-sm text-white flex items-center gap-3";

                feedback.className = "p-3 rounded-xl text-center text-xs md:text-sm font-bold bg-red-500/20 text-red-300 border border-red-500/40";
                feedback.innerText = "❌ YANLIŞ CEVAP! Kalkan gücü düştü.";

                updateHUD();

                setTimeout(() => {
                    closeModal('questionModal');
                    if (lives <= 0) {
                        loseGame();
                    } else {
                        createShatterParticles(gateObj.meshGroup.group.position);
                        gateObj.meshGroup.group.remove(gateObj.meshGroup.shield);
                        gateObj.cleared = true;
                        currentGateIndex++;
                        isWaitingAtGate = false;
                        AudioFX.setEnginePitch(1.0);
                    }
                }, 1100);
            }
        }

        function closeModal(id) {
            const el = document.getElementById(id);
            if (el) {
                el.classList.add('hidden');
                el.classList.remove('flex');
            }
        }

        /* --------------------------------------------------------------------------
           8. HUD & GAME STATE MANAGEMENT
           -------------------------------------------------------------------------- */
        function updateHUD() {
            document.getElementById('hudGradeText').innerText = `${selectedGrade}. Sınıf`;
            document.getElementById('hudScore').innerText = score;

            const livesDiv = document.getElementById('hudLives');
            livesDiv.innerHTML = '';
            for (let i = 0; i < 3; i++) {
                livesDiv.innerHTML += (i < lives) 
                    ? '<i class="fa-solid fa-shield-halved text-emerald-400"></i>' 
                    : '<i class="fa-regular fa-shield text-slate-600"></i>';
            }
        }

        function startTimer() {
            timerSeconds = 0;
            clearInterval(timerInterval);
            timerInterval = setInterval(() => {
                if (gameActive && !isWaitingAtGate) {
                    timerSeconds++;
                    const mins = Math.floor(timerSeconds / 60).toString().padStart(2, '0');
                    const secs = (timerSeconds % 60).toString().padStart(2, '0');
                    document.getElementById('hudTimer').innerText = `${mins}:${secs}`;
                }
            }, 1000);
        }

        function startGame(grade) {
            selectedGrade = grade;
            score = 0;
            lives = 3;
            correctAnswers = 0;
            wrongAnswers = 0;
            pathProgress = 0;
            currentGateIndex = 0;
            isWaitingAtGate = false;
            turboCharges = 0;
            isBoosting = false;
            boostMultiplier = 1.0;
            gameActive = true;

            // Remove Start Modal instantly
            closeModal('startModal');

            // Display Gameplay HUD Elements
            document.getElementById('hudTop').classList.remove('hidden');
            document.getElementById('hudTop').classList.add('flex');
            document.getElementById('speedHUD').classList.remove('hidden');
            document.getElementById('speedHUD').classList.add('flex');

            updateHUD();
            updateTurboUI();
            startTimer();

            // Audio & 3D Initialization
            AudioFX.startEngine();
            initThreeScene();

            if (animationFrameId) cancelAnimationFrame(animationFrameId);
            lastTime = performance.now();
            animate();
        }

        function winGame() {
            gameActive = false;
            clearInterval(timerInterval);
            AudioFX.playCorrect();

            document.getElementById('resultIcon').innerText = "🏆";
            document.getElementById('resultTitle').innerText = "MÜKEMMEL!";
            document.getElementById('resultSubtitle').innerText = "Tüm Fen Bilimleri kapılarını başarıyla geçtin!";
            document.getElementById('resScore').innerText = score;
            document.getElementById('resAccuracy').innerText = `${correctAnswers} Doğru / ${wrongAnswers} Yanlış`;
            document.getElementById('resTime').innerText = document.getElementById('hudTimer').innerText;

            document.getElementById('resultModal').classList.remove('hidden');
            document.getElementById('resultModal').classList.add('flex');
        }

        function loseGame() {
            gameActive = false;
            clearInterval(timerInterval);

            document.getElementById('resultIcon').innerText = "💥";
            document.getElementById('resultTitle').innerText = "KALKAN ÇÖKTÜ!";
            document.getElementById('resultSubtitle').innerText = "Kalkan gücün bitti, tekrar dene!";
            document.getElementById('resScore').innerText = score;
            document.getElementById('resAccuracy').innerText = `${correctAnswers} Doğru / ${wrongAnswers} Yanlış`;
            document.getElementById('resTime').innerText = document.getElementById('hudTimer').innerText;

            document.getElementById('resultModal').classList.remove('hidden');
            document.getElementById('resultModal').classList.add('flex');
        }

        function resetToHome() {
            closeModal('resultModal');
            document.getElementById('hudTop').classList.add('hidden');
            document.getElementById('speedHUD').classList.add('hidden');
            
            const startModal = document.getElementById('startModal');
            startModal.classList.remove('hidden');
            startModal.classList.add('flex');
        }

        function onWindowResize() {
            if (camera && renderer) {
                camera.aspect = window.innerWidth / window.innerHeight;
                camera.updateProjectionMatrix();
                renderer.setSize(window.innerWidth, window.innerHeight);
            }
        }
    </script>
</body>
</html>
```
