<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sút Phạt Chọn Người Ngẫu Nhiên</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome CDN -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Canvas Confetti Library -->
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;600;700;800;900&display=swap');
        
        body {
            font-family: 'Montserrat', sans-serif;
            background-color: #0f172a;
            color: #f8fafc;
            overflow-x: hidden;
            touch-action: manipulation;
        }

        /* Sân bóng 3D View */
        .stadium-container {
            perspective: 1000px;
            overflow: hidden;
            position: relative;
        }

        .pitch {
            background: linear-gradient(to bottom, #15803d 0%, #166534 100%);
            background-image: repeating-linear-gradient(
                0deg,
                #16a34a,
                #16a34a 30px,
                #15803d 30px,
                #15803d 60px
            );
            transform-style: preserve-3d;
            box-shadow: inset 0 0 100px rgba(0, 0, 0, 0.6);
        }

        /* Khung thành 3D */
        .goal-post {
            border: 8px solid #ffffff;
            border-bottom: none;
            box-shadow: 0 0 15px rgba(255, 255, 255, 0.8), inset 0 0 10px rgba(0,0,0,0.5);
            position: relative;
            background: rgba(255, 255, 255, 0.05);
            transition: all 0.3s ease;
        }

        /* Lưới khung thành */
        .goal-net {
            background-image: 
                linear-gradient(to right, rgba(255, 255, 255, 0.25) 1px, transparent 1px),
                linear-gradient(to bottom, rgba(255, 255, 255, 0.25) 1px, transparent 1px);
            background-size: 15px 15px;
        }

        /* Lưới ô mục tiêu linh hoạt */
        #targetGrid {
            display: grid;
            gap: 4px;
            width: 100%;
            height: 100%;
            align-content: center;
            justify-content: center;
        }

        /* Ô mục tiêu tùy biến */
        .target-grid-item {
            transition: all 0.25s ease;
            backdrop-filter: blur(2px);
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            overflow: hidden;
            word-break: break-word;
        }

        .target-grid-item:hover {
            background-color: rgba(234, 179, 8, 0.4);
            border-color: #eab308;
            transform: scale(1.05);
            z-index: 20;
        }

        .target-selected {
            animation: pulseTarget 0.5s infinite alternate;
            background-color: rgba(34, 197, 94, 0.8) !important;
            border-color: #22c55e !important;
            box-shadow: 0 0 25px #22c55e;
            z-index: 30;
        }

        @keyframes pulseTarget {
            0% { transform: scale(1); }
            100% { transform: scale(1.06); }
        }

        /* Quả bóng & Thủ môn */
        .soccer-ball {
            will-change: transform;
        }

        .ball-shadow {
            will-change: transform, opacity;
        }

        .goalkeeper {
            transition: left 0.8s ease-out, transform 0.8s ease-out;
            will-change: transform, left;
        }

        /* Ronaldo Player Styles */
        .ronaldo-container {
            position: absolute;
            bottom: 2.2rem;
            left: calc(50% - 65px);
            z-index: 25;
            transition: transform 0.3s ease, left 0.3s ease;
            transform-origin: bottom center;
        }

        @media (min-width: 640px) {
            .ronaldo-container {
                bottom: 2.8rem;
                left: calc(50% - 75px);
            }
        }

        /* Animation khi CR7 chạy đà sút bóng */
        .ronaldo-run {
            animation: ronaldoRunUp 0.4s forwards ease-in-out;
        }

        @keyframes ronaldoRunUp {
            0% {
                transform: translate(0, 0) rotate(0deg);
            }
            50% {
                transform: translate(35px, -8px) rotate(8deg);
            }
            80% {
                transform: translate(50px, -2px) rotate(-12deg) scale(1.05);
            }
            100% {
                transform: translate(52px, 0px) rotate(5deg);
            }
        }

        /* Custom scrollbar */
        .custom-scroll::-webkit-scrollbar {
            width: 5px;
        }
        .custom-scroll::-webkit-scrollbar-track {
            background: rgba(15, 23, 42, 0.6);
        }
        .custom-scroll::-webkit-scrollbar-thumb {
            background: #334155;
            border-radius: 4px;
        }
        .custom-scroll::-webkit-scrollbar-thumb:hover {
            background: #10b981;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between select-none">

    <!-- Header -->
    <header class="bg-slate-900/90 backdrop-blur border-b border-slate-800 sticky top-0 z-50 px-4 py-3 shadow-md">
        <div class="max-w-7xl mx-auto flex items-center justify-between">
            <div class="flex items-center gap-3">
                <div class="bg-gradient-to-tr from-emerald-500 to-green-400 p-2.5 rounded-xl shadow-lg shadow-green-500/20 text-slate-950 font-black text-xl flex items-center justify-center">
                    <i class="fa-solid fa-futbol"></i>
                </div>
                <div>
                    <h1 class="font-extrabold text-lg sm:text-xl bg-gradient-to-r from-emerald-400 via-teal-300 to-yellow-400 bg-clip-text text-transparent">
                        SÚT PHẠT RANDOM
                    </h1>
                    <p class="text-xs text-slate-400 hidden sm:block">Trò chơi chọn người may mắn ngẫu nhiên</p>
                </div>
            </div>

            <!-- Header buttons -->
            <div class="flex items-center gap-2 sm:gap-4">
                <button id="soundToggleBtn" onclick="toggleSound()" class="px-3 py-2 bg-slate-800 hover:bg-slate-700 text-slate-200 rounded-lg text-sm font-semibold flex items-center gap-2 border border-slate-700 transition">
                    <i id="soundIcon" class="fa-solid fa-volume-high text-emerald-400"></i>
                    <span class="hidden sm:inline" id="soundText">Âm thanh: Bật</span>
                </button>
                <button onclick="toggleSidebar()" class="lg:hidden px-3 py-2 bg-emerald-600 hover:bg-emerald-500 text-white rounded-lg text-sm font-semibold flex items-center gap-2 shadow-lg shadow-emerald-600/30">
                    <i class="fa-solid fa-users"></i>
                    <span id="mobileCountBadge" class="bg-emerald-900 text-emerald-200 px-1.5 py-0.5 rounded-full text-xs">0</span>
                </button>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-7xl w-full mx-auto p-2 sm:p-4 lg:p-6 flex-1 grid grid-cols-1 lg:grid-cols-12 gap-6 items-start">

        <!-- SÂN BÓNG (STADIUM DISPLAY - 8 Columns) -->
        <section class="lg:col-span-8 flex flex-col items-center justify-center bg-slate-900 border border-slate-800 rounded-2xl p-3 sm:p-6 shadow-2xl overflow-hidden relative min-h-[550px] sm:min-h-[650px]">
            
            <!-- Đèn chiếu sáng -->
            <div class="absolute top-0 left-0 right-0 flex justify-between px-8 pt-2 pointer-events-none opacity-40">
                <div class="w-24 h-6 bg-cyan-400 blur-xl rounded-full"></div>
                <div class="w-32 h-6 bg-white blur-xl rounded-full"></div>
                <div class="w-24 h-6 bg-cyan-400 blur-xl rounded-full"></div>
            </div>

            <!-- Bảng trạng thái -->
            <div class="z-10 mb-3 bg-slate-950/80 border border-slate-700/60 px-5 py-1.5 rounded-full flex items-center gap-4 text-xs sm:text-sm shadow-xl">
                <div class="flex items-center gap-2">
                    <span class="w-2.5 h-2.5 rounded-full bg-emerald-500 animate-ping"></span>
                    <span class="text-slate-300 font-medium">Sẵn sàng sút</span>
                </div>
                <div class="h-4 w-px bg-slate-700"></div>
                <div class="text-amber-400 font-bold">
                    Số ô còn lại: <span id="totalTargetsText" class="text-white text-base font-extrabold">0</span>
                </div>
            </div>

            <!-- Khung cảnh Sân Bóng & Khung Thành -->
            <div class="stadium-container w-full max-w-3xl flex flex-col items-center justify-end relative my-auto">
                
                <!-- KHUNG THÀNH -->
                <div id="goalElement" class="goal-post w-full h-64 sm:h-80 rounded-t-md relative flex flex-col justify-between overflow-hidden shadow-2xl">
                    <div class="goal-net absolute inset-0 z-0"></div>

                    <!-- LƯỚI Ô MỤC TIÊU -->
                    <div id="targetGrid" class="absolute inset-0 z-10 p-2 overflow-y-auto custom-scroll">
                        <!-- GENERATED JS TARGETS HERE -->
                    </div>

                    <!-- THỦ MÔN LEO MESSI -->
                    <div id="goalkeeper" class="goalkeeper absolute bottom-0 left-1/2 -translate-x-1/2 w-14 h-20 sm:w-20 sm:h-26 z-20 flex flex-col items-center justify-end pointer-events-none">
                        <div class="relative w-full h-full flex flex-col items-center justify-end drop-shadow-lg">
                            <!-- SVG THỦ MÔN LEO MESSI (GK #10) -->
                            <svg viewBox="0 0 100 130" class="w-full h-full">
                                <!-- Bóng chân -->
                                <ellipse cx="50" cy="125" rx="18" ry="4" fill="rgba(0,0,0,0.3)"/>
                                
                                <!-- Giày & Tất thủ môn -->
                                <path d="M 32 110 L 26 123 L 36 123 Z" fill="#3b82f6"/>
                                <path d="M 60 110 L 64 123 L 74 123 Z" fill="#3b82f6"/>
                                <rect x="30" y="94" width="8" height="18" rx="2" fill="#10b981"/>
                                <rect x="58" y="94" width="8" height="18" rx="2" fill="#10b981"/>
                                
                                <!-- Quần đùi Thủ môn -->
                                <path d="M 28 72 L 48 72 L 46 95 L 26 93 Z" fill="#064e3b"/>
                                <path d="M 52 72 L 72 72 L 74 93 L 54 95 Z" fill="#064e3b"/>
                                <text x="34" y="87" fill="#ffffff" font-size="7" font-weight="900">10</text>

                                <!-- Găng tay thủ môn dang rộng - BÊN TRÁI -->
                                <path d="M 26 42 L 10 32 L 6 38 L 22 52 Z" fill="#10b981"/>
                                <path d="M 10 32 L 2 28 L 0 35 L 8 42 Z" fill="#f97316"/>

                                <!-- Găng tay thủ môn dang rộng - BÊN PHẢI -->
                                <path d="M 74 42 L 90 32 L 94 38 L 78 52 Z" fill="#10b981"/>
                                <path d="M 90 32 L 98 28 L 100 35 L 92 42 Z" fill="#f97316"/>

                                <!-- Áo thủ môn (Xanh lá Neon GK) -->
                                <path d="M 26 42 L 74 42 L 72 73 L 28 73 Z" fill="#10b981"/>
                                <path d="M 35 42 L 50 73 L 42 73 Z" fill="#059669" opacity="0.6"/>
                                <path d="M 65 42 L 50 73 L 58 73 Z" fill="#059669" opacity="0.6"/>
                                
                                <!-- Số 10 trên áo thủ môn Messi -->
                                <text x="50" y="63" text-anchor="middle" fill="#ffffff" font-size="18" font-weight="900" font-family="sans-serif">10</text>
                                
                                <!-- Cổ áo & Mặt Messi -->
                                <rect x="44" y="34" width="12" height="10" fill="#fca5a5"/>
                                <path d="M 40 22 C 40 12 60 12 60 22 L 58 35 L 42 35 Z" fill="#fca5a5"/>
                                
                                <!-- Râu quai nón đặc trưng Leo Messi -->
                                <path d="M 41 27 C 41 37 59 37 59 27 L 57 34 C 52 38 48 38 43 34 Z" fill="#b45309"/>
                                
                                <!-- Tóc kiểu Messi -->
                                <path d="M 39 20 C 42 8 58 8 61 20 C 58 13 42 13 39 20 Z" fill="#451a03"/>
                                <path d="M 41 14 C 47 7 57 9 60 15 C 54 10 46 10 41 14 Z" fill="#78350f"/>
                                
                                <!-- Tai & Mắt -->
                                <circle cx="39" cy="25" r="2.5" fill="#fca5a5"/>
                                <circle cx="61" cy="25" r="2.5" fill="#fca5a5"/>
                                <circle cx="46" cy="24" r="1" fill="#1e1b4b"/>
                                <circle cx="54" cy="24" r="1" fill="#1e1b4b"/>
                            </svg>
                            <!-- Nhãn tên Messi -->
                            <div class="bg-amber-400 text-slate-950 text-[8px] sm:text-[9px] font-black px-1.5 py-0.5 rounded uppercase tracking-tighter -mt-1 shadow border border-amber-300">LEO MESSI (GK #10)</div>
                        </div>
                    </div>
                </div>

                <!-- CỎ SÂN BÓNG -->
                <div class="pitch w-full h-40 sm:h-52 rounded-b-2xl relative flex flex-col items-center justify-end border-t-2 border-white/40 overflow-hidden shadow-inner">
                    <!-- Vòng cấm -->
                    <div class="absolute top-0 w-48 sm:w-64 h-20 sm:h-28 border-2 border-white/40 border-t-0 rounded-b-full"></div>
                    
                    <!-- Chấm Penalty -->
                    <div class="w-3 h-3 bg-white rounded-full mb-8 sm:mb-10 shadow-md relative">
                        <div class="absolute -inset-1 border border-white/50 rounded-full animate-pulse"></div>
                    </div>

                    <!-- RONALDO MẶC ÁO ARGENTINA SỐ 7 -->
                    <div id="ronaldo" class="ronaldo-container w-16 h-24 sm:w-20 sm:h-28 flex flex-col items-center justify-end">
                        <div class="relative w-full h-full flex flex-col items-center justify-end drop-shadow-xl">
                            <!-- SVG CẦU THỦ CR7 ÁO ARGENTINA #7 -->
                            <svg viewBox="0 0 100 130" class="w-full h-full">
                                <defs>
                                    <!-- Sọc áo Argentina -->
                                    <pattern id="argStripes" width="16" height="40" patternUnits="userSpaceOnUse">
                                        <rect width="8" height="40" fill="#7dd3fc"/>
                                        <rect x="8" width="8" height="40" fill="#ffffff"/>
                                    </pattern>
                                </defs>
                                
                                <!-- Bóng chân -->
                                <ellipse cx="50" cy="125" rx="20" ry="4" fill="rgba(0,0,0,0.3)"/>
                                
                                <!-- Giày & Tất -->
                                <path d="M 32 110 L 28 123 L 38 123 Z" fill="#e11d48"/>
                                <path d="M 60 110 L 62 123 L 72 123 Z" fill="#e11d48"/>
                                <rect x="30" y="95" width="8" height="18" rx="2" fill="#38bdf8"/>
                                <rect x="58" y="95" width="8" height="18" rx="2" fill="#38bdf8"/>
                                
                                <!-- Quần đùi Xanh đen -->
                                <path d="M 30 72 L 48 72 L 46 96 L 28 94 Z" fill="#0f172a"/>
                                <path d="M 52 72 L 70 72 L 72 94 L 54 96 Z" fill="#0f172a"/>
                                <!-- Số 7 trên quần -->
                                <text x="35" y="88" fill="#ffffff" font-size="8" font-weight="900">7</text>
                                
                                <!-- Áo Argentina sọc xanh trắng -->
                                <path d="M 28 42 L 72 42 L 70 73 L 30 73 Z" fill="url(#argStripes)"/>
                                <!-- Tay áo -->
                                <path d="M 28 42 L 18 52 L 24 58 L 30 48 Z" fill="#7dd3fc"/>
                                <path d="M 72 42 L 82 52 L 76 58 L 70 48 Z" fill="#ffffff"/>
                                <!-- Cánh tay dang rộng dáng CR7 -->
                                <path d="M 18 52 L 12 70 L 17 72 L 24 58 Z" fill="#fca5a5"/>
                                <path d="M 82 52 L 88 70 L 83 72 L 76 58 Z" fill="#fca5a5"/>
                                
                                <!-- SỐ 7 TO TRÊN ÁO -->
                                <text x="50" y="65" text-anchor="middle" fill="#0f172a" font-size="20" font-weight="900" font-family="sans-serif">7</text>
                                
                                <!-- Cổ & Mặt Ronaldo -->
                                <rect x="44" y="34" width="12" height="10" fill="#fca5a5"/>
                                <!-- Khuôn mặt & Kiểu tóc CR7 vuốt ngược -->
                                <path d="M 40 22 C 40 12 60 12 60 22 L 58 35 L 42 35 Z" fill="#fca5a5"/>
                                <!-- Tóc CR7 -->
                                <path d="M 39 20 C 42 10 58 10 61 20 C 58 14 42 14 39 20 Z" fill="#1e1b4b"/>
                                <path d="M 41 15 C 48 8 55 10 59 16 C 53 11 45 11 41 15 Z" fill="#312e81"/>
                                <!-- Tai & Mắt -->
                                <circle cx="39" cy="26" r="2.5" fill="#fca5a5"/>
                                <circle cx="61" cy="26" r="2.5" fill="#fca5a5"/>
                            </svg>
                            <!-- Nhãn tên CR7 -->
                            <div class="bg-sky-500 text-slate-950 text-[8px] font-black px-1 rounded uppercase tracking-tighter -mt-1 shadow">CR7 (ARG #7)</div>
                        </div>
                    </div>

                    <!-- QUẢ BÓNG -->
                    <div id="ballShadow" class="ball-shadow absolute bottom-6 sm:bottom-8 w-10 h-3 bg-black/50 rounded-full blur-xs z-20"></div>
                    <div id="ball" class="soccer-ball absolute bottom-6 sm:bottom-8 w-10 h-10 sm:w-12 sm:h-12 z-30 cursor-pointer hover:scale-110 active:scale-95 flex items-center justify-center text-white text-3xl sm:text-4xl drop-shadow-2xl" onclick="shootBall()">
                        <i class="fa-solid fa-circle-futbol text-white bg-slate-900 rounded-full"></i>
                    </div>
                </div>
            </div>

            <!-- NÚT SÚT BÓNG TO BẢN -->
            <div class="mt-6 w-full max-w-xs z-10">
                <button id="shootBtn" onclick="shootBall()" class="w-full group relative inline-flex items-center justify-center p-0.5 overflow-hidden text-base font-bold text-slate-900 rounded-xl bg-gradient-to-br from-yellow-400 via-amber-500 to-emerald-500 hover:text-white shadow-xl shadow-amber-500/20 active:scale-95 transition-all duration-200">
                    <span class="w-full relative px-6 py-3.5 transition-all ease-in duration-75 bg-amber-400 group-hover:bg-opacity-0 rounded-xl flex items-center justify-center gap-3 text-slate-950 font-extrabold uppercase tracking-wider text-base sm:text-lg">
                        <i class="fa-solid fa-shoe-prints text-slate-900 text-xl group-hover:rotate-12 transition-transform"></i>
                        SÚT BÓNG NGAY!
                    </span>
                </button>
            </div>
        </section>

        <!-- SIDEBAR DANH SÁCH (4 Columns) -->
        <aside id="sidebar" class="lg:col-span-4 bg-slate-900 border border-slate-800 rounded-2xl p-4 sm:p-5 shadow-2xl flex flex-col gap-4">
            <div class="flex items-center justify-between border-b border-slate-800 pb-3">
                <h2 class="font-bold text-lg text-slate-100 flex items-center gap-2">
                    <i class="fa-solid fa-list-ol text-emerald-400"></i>
                    Danh Sách Tham Gia
                </h2>
                <span id="playerCountBadge" class="bg-emerald-500/20 text-emerald-400 text-xs font-bold px-2.5 py-1 rounded-full border border-emerald-500/30">
                    0 người
                </span>
            </div>

            <!-- Nhập danh sách nhanh -->
            <div class="flex flex-col gap-2">
                <label class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Thêm danh sách (Không giới hạn)</label>
                <textarea id="nameInput" rows="3" placeholder="Dán danh sách tên tại đây (xuống dòng hoặc dấu phẩy)...&#10;Nguyễn Văn A&#10;Trần Thị B&#10;Lê Văn C" class="w-full bg-slate-950 border border-slate-800 rounded-xl p-3 text-sm text-slate-200 placeholder-slate-600 focus:outline-none focus:border-emerald-500 focus:ring-1 focus:ring-emerald-500 transition resize-none custom-scroll"></textarea>
                <div class="flex gap-2">
                    <button onclick="addNamesFromInput()" class="flex-1 bg-emerald-600 hover:bg-emerald-500 active:bg-emerald-700 text-white font-semibold py-2 px-3 rounded-lg text-sm transition flex items-center justify-center gap-2 shadow-lg shadow-emerald-600/20">
                        <i class="fa-solid fa-plus"></i> Thêm Vào Danh Sách
                    </button>
                    <button onclick="clearAllNames()" class="bg-slate-800 hover:bg-rose-900/40 hover:text-rose-400 text-slate-400 border border-slate-700 px-3 py-2 rounded-lg text-sm transition" title="Xóa tất cả">
                        <i class="fa-solid fa-trash-can"></i>
                    </button>
                </div>
            </div>

            <!-- Danh sách người chơi -->
            <div class="flex-1 flex flex-col min-h-[200px] max-h-[280px]">
                <div class="flex items-center justify-between text-xs text-slate-400 mb-2">
                    <span>Danh sách hiện tại</span>
                    <button onclick="resetExcluded()" class="text-emerald-400 hover:underline flex items-center gap-1">
                        <i class="fa-solid fa-rotate-left"></i> Khôi phục người đã chọn
                    </button>
                </div>

                <div id="playerListContainer" class="flex-1 overflow-y-auto space-y-2 pr-1 custom-scroll">
                    <!-- ITEMS RENDERED BY JS -->
                </div>
            </div>

            <!-- Lịch sử trúng thưởng -->
            <div class="border-t border-slate-800 pt-3">
                <div class="flex items-center justify-between text-xs text-slate-400 mb-2">
                    <span class="font-bold text-amber-400 flex items-center gap-1"><i class="fa-solid fa-trophy"></i> Lịch sử trúng phạt</span>
                    <button onclick="clearHistory()" class="hover:text-rose-400 underline">Xóa</button>
                </div>
                <div id="historyList" class="max-h-24 overflow-y-auto text-xs space-y-1 text-slate-300 custom-scroll">
                    <div class="text-slate-600 italic text-center py-1">Chưa có lượt sút nào</div>
                </div>
            </div>

            <!-- Options -->
            <div class="border-t border-slate-800 pt-3 flex items-center justify-between text-xs text-slate-400">
                <label class="flex items-center gap-2 cursor-pointer select-none">
                    <input type="checkbox" id="autoExcludeCheck" checked class="rounded bg-slate-950 border-slate-700 text-emerald-500 focus:ring-emerald-500 w-4 h-4">
                    <span>Tự động ẩn người đã trúng</span>
                </label>
            </div>
        </aside>
    </main>

    <!-- MODAL HIỂN THỊ NGƯỜI CHIẾN THẮNG -->
    <div id="winnerModal" class="fixed inset-0 z-50 bg-slate-950/80 backdrop-blur-md flex items-center justify-center p-4 hidden opacity-0 transition-opacity duration-300">
        <div class="bg-gradient-to-b from-slate-900 to-slate-950 border-2 border-amber-500/50 rounded-3xl p-6 sm:p-8 max-w-md w-full text-center shadow-2xl shadow-amber-500/20 transform scale-90 transition-transform duration-300 relative overflow-hidden">
            
            <div class="absolute -top-24 -left-24 w-48 h-48 bg-amber-500/20 rounded-full blur-3xl pointer-events-none"></div>
            <div class="absolute -bottom-24 -right-24 w-48 h-48 bg-emerald-500/20 rounded-full blur-3xl pointer-events-none"></div>

            <div class="w-20 h-20 bg-gradient-to-tr from-amber-400 to-yellow-300 rounded-full flex items-center justify-center mx-auto mb-4 text-slate-950 text-4xl shadow-xl shadow-amber-500/30 animate-bounce">
                <i class="fa-solid fa-trophy"></i>
            </div>

            <h3 class="text-sm font-bold text-amber-400 uppercase tracking-widest mb-1">XIN CHÚC MỪNG!</h3>
            <h4 class="text-xs text-slate-400 mb-4">QUẢ BÓNG ĐÃ LỌT LƯỚI TẠI VỊ TRÍ CỦA</h4>

            <div id="winnerName" class="text-2xl sm:text-3xl font-black text-white bg-slate-800/80 border border-slate-700 py-4 px-6 rounded-2xl mb-6 shadow-inner text-emerald-400 break-words">
                NGUYỄN VĂN A
            </div>

            <div class="flex flex-col sm:flex-row gap-3">
                <button onclick="closeWinnerModal()" class="flex-1 bg-gradient-to-r from-emerald-500 to-teal-600 hover:from-emerald-400 hover:to-teal-500 text-slate-950 font-extrabold py-3 px-4 rounded-xl shadow-lg transition active:scale-95">
                    <i class="fa-solid fa-play mr-1"></i> Tiếp Tục Sút
                </button>
            </div>
        </div>
    </div>

    <script>
        /* ===================================================================
           STATE MANAGEMENT
           =================================================================== */
        let players = Array.from({ length: 24 }, (_, i) => ({
            id: i + 1,
            name: `Học sinh ${i + 1}`,
            excluded: false
        }));

        let winnersHistory = [];
        let isShooting = false;
        let soundEnabled = true;
        let goalkeeperInterval = null;
        let audioCtx = null;

        /* ===================================================================
           WEB AUDIO API SYNTHESIZER (NO EXTERNAL AUDIO FILES NEEDED)
           =================================================================== */
        function getAudioContext() {
            if (!audioCtx) {
                audioCtx = new (window.AudioContext || window.webkitAudioContext)();
            }
            if (audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
            return audioCtx;
        }

        function playKickSound() {
            if (!soundEnabled) return;
            try {
                const ctx = getAudioContext();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = 'triangle';
                osc.frequency.setValueAtTime(160, ctx.currentTime);
                osc.frequency.exponentialRampToValueAtTime(30, ctx.currentTime + 0.15);
                gain.gain.setValueAtTime(1, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.15);
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + 0.15);
            } catch(e) { console.error(e); }
        }

        function playWhistleSound() {
            if (!soundEnabled) return;
            try {
                const ctx = getAudioContext();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(2200, ctx.currentTime);
                osc.frequency.setValueAtTime(2800, ctx.currentTime + 0.08);
                osc.frequency.setValueAtTime(2200, ctx.currentTime + 0.16);
                gain.gain.setValueAtTime(0.25, ctx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.4);
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + 0.4);
            } catch(e) { console.error(e); }
        }

        function playGoalCheer() {
            if (!soundEnabled) return;
            playWhistleSound();
            try {
                const ctx = getAudioContext();
                const bufferSize = ctx.sampleRate * 1.5;
                const buffer = ctx.createBuffer(1, bufferSize, ctx.sampleRate);
                const data = buffer.getChannelData(0);
                for (let i = 0; i < bufferSize; i++) {
                    data[i] = Math.random() * 2 - 1;
                }
                const noise = ctx.createBufferSource();
                noise.buffer = buffer;
                const filter = ctx.createBiquadFilter();
                filter.type = 'bandpass';
                filter.frequency.value = 1000;
                filter.Q.value = 1.2;

                const gain = ctx.createGain();
                gain.gain.setValueAtTime(0.01, ctx.currentTime);
                gain.gain.linearRampToValueAtTime(0.3, ctx.currentTime + 0.2);
                gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 1.5);

                noise.connect(filter);
                filter.connect(gain);
                gain.connect(ctx.destination);
                noise.start();
            } catch(e) { console.error(e); }
        }

        function toggleSound() {
            soundEnabled = !soundEnabled;
            const icon = document.getElementById('soundIcon');
            const text = document.getElementById('soundText');
            if (soundEnabled) {
                icon.className = 'fa-solid fa-volume-high text-emerald-400';
                text.innerText = 'Âm thanh: Bật';
            } else {
                icon.className = 'fa-solid fa-volume-xmark text-slate-500';
                text.innerText = 'Âm thanh: Tắt';
            }
        }

        /* ===================================================================
           DYNAMIC GOAL & GRID RENDERER
           =================================================================== */
        function renderAll() {
            renderPlayerList();
            renderGoalTargets();
            renderHistory();
        }

        function renderGoalTargets() {
            const grid = document.getElementById('targetGrid');
            const totalText = document.getElementById('totalTargetsText');
            const activePlayers = players.filter(p => !p.excluded);

            totalText.innerText = activePlayers.length;

            if (activePlayers.length === 0) {
                grid.style.display = 'flex';
                grid.className = 'absolute inset-0 z-10 flex items-center justify-center p-4';
                grid.innerHTML = `
                    <div class="text-center bg-slate-950/80 px-4 py-3 rounded-xl border border-dashed border-slate-700 text-amber-400 text-xs sm:text-sm font-medium">
                        <i class="fa-solid fa-circle-exclamation mr-1"></i> Tất cả người chơi đã được chọn hoặc danh sách trống!
                    </div>
                `;
                return;
            }

            grid.style.display = 'grid';

            const count = activePlayers.length;
            let cols = 4;
            let fontSize = 'text-xs sm:text-sm';
            let numSize = 'text-[9px] sm:text-[10px]';
            let pyPadding = 'py-1 sm:py-2';

            if (count <= 6) {
                cols = count <= 3 ? count : 3;
                fontSize = 'text-sm sm:text-base font-extrabold';
            } else if (count <= 12) {
                cols = 4;
                fontSize = 'text-xs sm:text-sm';
            } else if (count <= 24) {
                cols = 6;
                fontSize = 'text-[11px] sm:text-xs';
                pyPadding = 'py-1';
            } else if (count <= 48) {
                cols = 8;
                fontSize = 'text-[9px] sm:text-[11px]';
                numSize = 'text-[8px]';
                pyPadding = 'py-0.5';
            } else if (count <= 80) {
                cols = 10;
                fontSize = 'text-[8px] sm:text-[10px]';
                numSize = 'hidden';
                pyPadding = 'py-0.5';
            } else {
                cols = 12;
                fontSize = 'text-[7px] sm:text-[9px]';
                numSize = 'hidden';
                pyPadding = 'py-0.5';
            }

            grid.style.gridTemplateColumns = `repeat(${cols}, minmax(0, 1fr))`;

            grid.innerHTML = activePlayers.map((player, idx) => `
                <div id="target-player-${player.id}" class="target-grid-item bg-slate-950/75 border border-emerald-500/30 rounded sm:rounded-lg ${pyPadding} px-0.5 text-center cursor-pointer hover:border-amber-400 transition">
                    <span class="${numSize} font-black text-amber-400/80 uppercase leading-none mb-0.5">${numSize !== 'hidden' ? '#' + (idx + 1) : ''}</span>
                    <span class="${fontSize} font-bold text-slate-100 truncate w-full px-0.5 leading-tight">${escapeHtml(player.name)}</span>
                </div>
            `).join('');
        }

        /* ===================================================================
           SHOOTING & PHYSICS ANIMATION
           =================================================================== */
        function shootBall() {
            if (isShooting) return;

            const activePlayers = players.filter(p => !p.excluded);
            if (activePlayers.length === 0) {
                alert("Vui lòng thêm người chơi hoặc khôi phục danh sách!");
                return;
            }

            isShooting = true;
            document.getElementById('shootBtn').disabled = true;

            const ronaldoEl = document.getElementById('ronaldo');
            if (ronaldoEl) {
                ronaldoEl.classList.add('ronaldo-run');
            }

            // Pick random target winner
            const winnerIndex = Math.floor(Math.random() * activePlayers.length);
            const winner = activePlayers[winnerIndex];

            const targetEl = document.getElementById(`target-player-${winner.id}`);
            const goalEl = document.getElementById('goalElement');
            const ballEl = document.getElementById('ball');
            const ballShadow = document.getElementById('ballShadow');
            const keeperEl = document.getElementById('goalkeeper');

            if (!targetEl || !goalEl || !ballEl) {
                isShooting = false;
                document.getElementById('shootBtn').disabled = false;
                return;
            }

            // Trì hoãn sút bóng một chút để Ronaldo chạy đà
            setTimeout(() => {
                playKickSound();

                // Calculate precise 3D trajectory
                const targetRect = targetEl.getBoundingClientRect();
                const ballRect = ballEl.getBoundingClientRect();
                const goalRect = goalEl.getBoundingClientRect();

                const targetCenterX = targetRect.left + targetRect.width / 2;
                const targetCenterY = targetRect.top + targetRect.height / 2;

                const ballCenterX = ballRect.left + ballRect.width / 2;
                const ballCenterY = ballRect.top + ballRect.height / 2;

                const deltaX = targetCenterX - ballCenterX;
                const deltaY = targetCenterY - ballCenterY;

                // Move Goalkeeper out of way or dive
                const goalWidth = goalRect.width;
                const targetRelativeLeft = targetRect.left - goalRect.left + targetRect.width / 2;
                let keeperTargetX = targetRelativeLeft < goalWidth / 2 ? goalWidth * 0.75 : goalWidth * 0.25;
                
                keeperEl.style.left = `${keeperTargetX}px`;
                keeperEl.style.transform = `translateX(-50%) rotate(${targetRelativeLeft < goalWidth / 2 ? 30 : -30}deg)`;

                // Animate ball
                ballEl.style.transition = "transform 1.1s cubic-bezier(0.15, 0.85, 0.35, 1.2)";
                ballEl.style.transform = `translate(${deltaX}px, ${deltaY}px) scale(0.5) rotate(1080deg)`;
                
                if (ballShadow) {
                    ballShadow.style.transition = "transform 1.1s cubic-bezier(0.15, 0.85, 0.35, 1.2), opacity 1.1s";
                    ballShadow.style.transform = `translate(${deltaX * 0.3}px, 0px) scale(0.2)`;
                    ballShadow.style.opacity = '0.2';
                }

                // Rapid random grid glow effect during flight
                let highlightInterval = setInterval(() => {
                    const randItem = activePlayers[Math.floor(Math.random() * activePlayers.length)];
                    const randEl = document.getElementById(`target-player-${randItem.id}`);
                    if (randEl) {
                        randEl.classList.add('bg-amber-500/40');
                        setTimeout(() => randEl.classList.remove('bg-amber-500/40'), 150);
                    }
                }, 80);

                // Impact with goal target
                setTimeout(() => {
                    clearInterval(highlightInterval);
                    targetEl.classList.add('target-selected');

                    playGoalCheer();
                    triggerConfetti();

                    setTimeout(() => {
                        showWinnerModal(winner);
                    }, 400);

                }, 1100);
            }, 300); // 300ms chạy đà của Ronaldo
        }

        function resetBallPosition() {
            const ballEl = document.getElementById('ball');
            const ballShadow = document.getElementById('ballShadow');
            const keeperEl = document.getElementById('goalkeeper');
            const ronaldoEl = document.getElementById('ronaldo');

            if (ronaldoEl) {
                ronaldoEl.classList.remove('ronaldo-run');
            }

            if (ballEl) {
                ballEl.style.transition = 'none';
                ballEl.style.transform = 'translate(0px, 0px) scale(1) rotate(0deg)';
            }
            if (ballShadow) {
                ballShadow.style.transition = 'none';
                ballShadow.style.transform = 'translate(0px, 0px) scale(1)';
                ballShadow.style.opacity = '1';
            }
            if (keeperEl) {
                keeperEl.style.left = '50%';
                keeperEl.style.transform = 'translateX(-50%) rotate(0deg)';
            }

            document.querySelectorAll('.target-selected').forEach(el => {
                el.classList.remove('target-selected');
            });
        }

        function triggerConfetti() {
            if (typeof confetti === 'function') {
                confetti({
                    particleCount: 120,
                    spread: 80,
                    origin: { y: 0.6 }
                });
            }
        }

        /* ===================================================================
           WINNER MODAL & PLAYER LIST MANAGEMENT
           =================================================================== */
        function showWinnerModal(winner) {
            const modal = document.getElementById('winnerModal');
            const nameEl = document.getElementById('winnerName');

            nameEl.innerText = winner.name;
            modal.classList.remove('hidden');

            const now = new Date();
            const timeStr = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;
            winnersHistory.unshift({ name: winner.name, time: timeStr });

            setTimeout(() => {
                modal.classList.remove('opacity-0');
                modal.children[0].classList.remove('scale-90');
                modal.children[0].classList.add('scale-100');
            }, 10);

            if (document.getElementById('autoExcludeCheck').checked) {
                winner.excluded = true;
            }
        }

        function closeWinnerModal() {
            const modal = document.getElementById('winnerModal');
            modal.classList.add('opacity-0');
            modal.children[0].classList.remove('scale-100');
            modal.children[0].classList.add('scale-90');

            setTimeout(() => {
                modal.classList.add('hidden');
                resetBallPosition();
                renderAll();
                isShooting = false;
                document.getElementById('shootBtn').disabled = false;
            }, 300);
        }

        function renderPlayerList() {
            const container = document.getElementById('playerListContainer');
            const badge = document.getElementById('playerCountBadge');
            const mobileBadge = document.getElementById('mobileCountBadge');

            if (badge) badge.innerText = `${players.length} người`;
            if (mobileBadge) mobileBadge.innerText = `${players.length}`;

            if (players.length === 0) {
                container.innerHTML = `
                    <div class="text-center py-8 text-slate-500 text-xs italic">
                        Chưa có người chơi nào.<br>Hãy nhập danh sách ở trên!
                    </div>
                `;
                return;
            }

            container.innerHTML = players.map((p, idx) => `
                <div class="flex items-center justify-between p-2 rounded-lg ${p.excluded ? 'bg-slate-950/40 opacity-50 border border-slate-800/50' : 'bg-slate-950/90 border border-slate-800 hover:border-slate-700'} transition text-xs">
                    <div class="flex items-center gap-2 overflow-hidden mr-2">
                        <span class="text-slate-500 font-mono text-[10px] w-5">${idx + 1}.</span>
                        <span class="font-medium text-slate-200 truncate ${p.excluded ? 'line-through text-slate-500' : ''}">${escapeHtml(p.name)}</span>
                    </div>
                    <div class="flex items-center gap-1 shrink-0">
                        <button onclick="toggleExcludePlayer(${p.id})" title="${p.excluded ? 'Khôi phục' : 'Tạm ẩn'}" class="p-1 rounded ${p.excluded ? 'text-emerald-400 hover:bg-emerald-950/50' : 'text-slate-400 hover:text-amber-400 hover:bg-slate-800'} transition">
                            <i class="fa-solid ${p.excluded ? 'fa-eye' : 'fa-eye-slash'}"></i>
                        </button>
                        <button onclick="removePlayer(${p.id})" title="Xóa" class="p-1 text-slate-500 hover:text-rose-400 hover:bg-rose-950/30 rounded transition">
                            <i class="fa-solid fa-xmark"></i>
                        </button>
                    </div>
                </div>
            `).join('');
        }

        function renderHistory() {
            const container = document.getElementById('historyList');
            if (winnersHistory.length === 0) {
                container.innerHTML = `<div class="text-slate-600 italic text-center py-1">Chưa có lượt sút nào</div>`;
                return;
            }
            container.innerHTML = winnersHistory.map((item, index) => `
                <div class="flex items-center justify-between bg-slate-950/60 px-2.5 py-1 rounded border border-slate-800">
                    <span class="text-amber-400 font-bold">#${winnersHistory.length - index}</span>
                    <span class="text-slate-200 font-semibold truncate max-w-[150px]">${escapeHtml(item.name)}</span>
                    <span class="text-[10px] text-slate-500">${item.time}</span>
                </div>
            `).join('');
        }

        function addNamesFromInput() {
            const input = document.getElementById('nameInput');
            const text = input.value.trim();

            if (!text) return;

            const rawNames = text.split(/[\n,]+/);
            rawNames.forEach(raw => {
                const name = raw.trim();
                if (name.length > 0) {
                    players.push({
                        id: Date.now() + Math.random(),
                        name: name,
                        excluded: false
                    });
                }
            });

            input.value = '';
            renderAll();
        }

        function removePlayer(id) {
            players = players.filter(p => p.id !== id);
            renderAll();
        }

        function toggleExcludePlayer(id) {
            const p = players.find(player => player.id === id);
            if (p) {
                p.excluded = !p.excluded;
                renderAll();
            }
        }

        function resetExcluded() {
            players.forEach(p => p.excluded = false);
            renderAll();
        }

        function clearAllNames() {
            if (players.length === 0) return;
            players = [];
            renderAll();
        }

        function clearHistory() {
            winnersHistory = [];
            renderHistory();
        }

        function toggleSidebar() {
            const sidebar = document.getElementById('sidebar');
            sidebar.classList.toggle('hidden');
        }

        function escapeHtml(str) {
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }

        function startGoalkeeperIdle() {
            const keeper = document.getElementById('goalkeeper');
            let direction = 1;
            let currentPos = 50;

            goalkeeperInterval = setInterval(() => {
                if (isShooting || !keeper) return;
                currentPos += direction * 3;
                if (currentPos > 75) direction = -1;
                if (currentPos < 25) direction = 1;
                keeper.style.left = `${currentPos}%`;
            }, 800);
        }

        window.onload = () => {
            renderAll();
            startGoalkeeperIdle();
        };
    </script>
</body>
</html>
