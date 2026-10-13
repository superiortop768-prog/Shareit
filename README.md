```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ShareMe Clone - Real P2P Transfer</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <!-- PeerJS for Real WebRTC P2P Connections -->
    <script src="https://unpkg.com/peerjs@1.5.1/dist/peerjs.min.js"></script>
    
    <!-- QR Code Libraries -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            -webkit-tap-highlight-color: transparent;
        }

        /* Custom Scrollbar for file list */
        .custom-scrollbar::-webkit-scrollbar {
            width: 6px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: transparent;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background-color: #cbd5e1;
            border-radius: 10px;
        }
        
        .pulse-btn {
            animation: pulse-soft 2s infinite;
        }
        
        @keyframes pulse-soft {
            0% { box-shadow: 0 0 0 0 rgba(59, 130, 246, 0.4); }
            70% { box-shadow: 0 0 0 10px rgba(59, 130, 246, 0); }
            100% { box-shadow: 0 0 0 0 rgba(59, 130, 246, 0); }
        }

        /* Shareit Specific Styles */
        .shareit-bg {
            background: linear-gradient(135deg, #1890ff 0%, #00d2ff 100%);
        }
        
        .radar-circle {
            position: absolute;
            border-radius: 50%;
            border: 1px solid rgba(255, 255, 255, 0.3);
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            animation: radar-pulse 3s infinite linear;
        }

        .radar-circle:nth-child(1) { width: 100px; height: 100px; animation-delay: 0s; }
        .radar-circle:nth-child(2) { width: 180px; height: 180px; animation-delay: 1s; }
        .radar-circle:nth-child(3) { width: 260px; height: 260px; animation-delay: 2s; }
        .radar-circle:nth-child(4) { width: 340px; height: 340px; animation-delay: 3s; }

        @keyframes radar-pulse {
            0% { transform: translate(-50%, -50%) scale(0.5); opacity: 1; }
            100% { transform: translate(-50%, -50%) scale(1.5); opacity: 0; }
        }

        .nav-tab.active {
            color: #1890ff;
            border-bottom: 2px solid #1890ff;
        }

        /* Hide sections by default for SPA feel */
        .screen {
            display: none;
        }
        .screen.active {
            display: flex;
        }
    </style>
</head>
<body class="h-screen w-full overflow-hidden text-gray-800">

    <!-- App Container (Mobile Sized) -->
    <div class="h-full w-full max-w-md mx-auto bg-gray-50 shadow-2xl relative flex flex-col overflow-hidden sm:rounded-3xl sm:h-[90vh] sm:mt-[5vh]">
        
        <!-- Top Navigation Bar -->
        <header class="shareit-bg text-white p-4 flex justify-between items-center z-20 shadow-md shrink-0 border-b border-white/10">
            <div class="flex items-center gap-3">
                <div class="w-8 h-8 bg-white/20 rounded-full flex items-center justify-center">
                    <i class="fas fa-user text-sm"></i>
                </div>
                <h1 class="text-xl font-bold tracking-wide">ShareMe</h1>
            </div>
            <div class="flex gap-4 items-center">
                <span id="connectionStatus" class="w-2 h-2 rounded-full bg-red-400" title="Disconnected"></span>
                <button class="hover:text-blue-200 transition"><i class="fas fa-search text-lg"></i></button>
                <button class="hover:text-blue-200 transition"><i class="fas fa-bars text-lg"></i></button>
            </div>
        </header>

        <!-- Main Content Area -->
        <main class="flex-1 relative overflow-hidden bg-gray-50 flex flex-col">
            
            <!-- 1. Home Screen (Send/Receive) -->
            <div id="homeScreen" class="screen active flex-col h-full w-full relative">
                <!-- Shareit Style Top Card -->
                <div class="bg-white rounded-b-3xl shadow-sm p-6 pt-8 pb-10 mb-2 relative z-10">
                    <div class="flex justify-between items-center mb-8">
                        <div>
                            <h2 class="text-gray-500 text-sm font-medium mb-1">Device ID</h2>
                            <p id="myDeviceIdDisplay" class="text-2xl font-bold text-gray-800 tracking-wide">...</p>
                            <p class="text-[10px] text-gray-400 mt-1"><i class="fas fa-wifi text-blue-400 mr-1"></i>WebRTC Ready</p>
                        </div>
                        <div class="w-16 h-16 bg-blue-50 rounded-full flex items-center justify-center border border-blue-100 shadow-inner">
                            <i class="fas fa-mobile-alt text-3xl text-blue-500"></i>
                        </div>
                    </div>

                    <!-- Main Action Buttons -->
                    <div class="flex w-full gap-4">
                        <button onclick="initiateSend()" class="flex-1 shareit-bg text-white py-4 rounded-2xl shadow-[0_8px_15px_rgba(24,144,255,0.3)] flex flex-col items-center gap-2 active:scale-95 transition-transform">
                            <div class="w-12 h-12 bg-white/20 rounded-full flex items-center justify-center mb-1">
                                <i class="fas fa-paper-plane text-2xl"></i>
                            </div>
                            <span class="font-bold text-lg">Send</span>
                        </button>
                        <button onclick="initiateReceive()" class="flex-1 bg-white text-blue-500 border border-blue-100 py-4 rounded-2xl shadow-[0_8px_15px_rgba(0,0,0,0.05)] flex flex-col items-center gap-2 active:scale-95 transition-transform">
                            <div class="w-12 h-12 bg-blue-50 rounded-full flex items-center justify-center mb-1">
                                <i class="fas fa-download text-2xl"></i>
                            </div>
                            <span class="font-bold text-lg">Receive</span>
                        </button>
                    </div>
                </div>

                <!-- Quick Categories -->
                <div class="flex-1 px-4 py-4 overflow-y-auto pb-20 custom-scrollbar">
                    <div class="flex justify-between items-center mb-4">
                        <h3 class="font-bold text-gray-700">Quick Share</h3>
                    </div>
                    <div class="grid grid-cols-4 gap-3 text-center">
                        <div onclick="initiateSend()" class="flex flex-col items-center gap-1 cursor-pointer">
                            <div class="w-12 h-12 bg-yellow-100 text-yellow-600 rounded-xl flex items-center justify-center text-xl"><i class="fas fa-images"></i></div>
                            <span class="text-xs font-medium text-gray-600">Photos</span>
                        </div>
                        <div onclick="initiateSend()" class="flex flex-col items-center gap-1 cursor-pointer">
                            <div class="w-12 h-12 bg-red-100 text-red-600 rounded-xl flex items-center justify-center text-xl"><i class="fas fa-play-circle"></i></div>
                            <span class="text-xs font-medium text-gray-600">Videos</span>
                        </div>
                        <div onclick="initiateSend()" class="flex flex-col items-center gap-1 cursor-pointer">
                            <div class="w-12 h-12 bg-green-100 text-green-600 rounded-xl flex items-center justify-center text-xl"><i class="fab fa-android"></i></div>
                            <span class="text-xs font-medium text-gray-600">Apps</span>
                        </div>
                        <div onclick="initiateSend()" class="flex flex-col items-center gap-1 cursor-pointer">
                            <div class="w-12 h-12 bg-blue-100 text-blue-600 rounded-xl flex items-center justify-center text-xl"><i class="fas fa-folder"></i></div>
                            <span class="text-xs font-medium text-gray-600">Files</span>
                        </div>
                    </div>
                    
                    <!-- Fake Banner -->
                    <div class="mt-6 bg-gradient-to-r from-purple-500 to-indigo-500 rounded-2xl p-4 text-white flex items-center justify-between shadow-md">
                        <div>
                            <h4 class="font-bold text-sm">Clean Phone Storage</h4>
                            <p class="text-xs text-purple-200 mt-1">Free up to 2.5 GB</p>
                        </div>
                        <button class="bg-white text-purple-600 px-4 py-1.5 rounded-full text-xs font-bold shadow-sm">Clean</button>
                    </div>
                </div>

                <!-- Bottom Navigation -->
                <div class="absolute bottom-0 w-full bg-white border-t border-gray-100 flex justify-around items-center py-2 pb-4 px-2 z-20 shadow-[0_-10px_20px_rgba(0,0,0,0.02)]">
                    <div class="flex flex-col items-center text-blue-500 gap-1 cursor-pointer">
                        <i class="fas fa-exchange-alt text-xl"></i>
                        <span class="text-[10px] font-bold">Transfer</span>
                    </div>
                    <div class="flex flex-col items-center text-gray-400 gap-1 cursor-pointer hover:text-blue-500">
                        <i class="fas fa-gamepad text-xl"></i>
                        <span class="text-[10px] font-medium">Games</span>
                    </div>
                    <div class="flex flex-col items-center text-gray-400 gap-1 cursor-pointer hover:text-blue-500">
                        <i class="fas fa-user-circle text-xl"></i>
                        <span class="text-[10px] font-medium">Me</span>
                    </div>
                </div>
            </div>

            <!-- 2. File Selection Screen (Send Mode) -->
            <div id="fileSelectScreen" class="screen flex-col h-full w-full bg-gray-50">
                <div class="bg-white shadow-sm shrink-0">
                    <div class="p-4 flex items-center gap-4">
                        <button onclick="goBackToHome()" class="text-gray-600 text-xl"><i class="fas fa-arrow-left"></i></button>
                        <h2 class="text-lg font-bold flex-1">Select Files</h2>
                        <button class="text-gray-600"><i class="fas fa-search"></i></button>
                    </div>
                    <!-- Tabs -->
                    <div class="flex overflow-x-auto custom-scrollbar px-2 border-b border-gray-100">
                        <button class="nav-tab active px-4 py-3 font-semibold text-sm whitespace-nowrap">Files</button>
                        <button class="nav-tab text-gray-500 px-4 py-3 font-semibold text-sm whitespace-nowrap">Videos</button>
                        <button class="nav-tab text-gray-500 px-4 py-3 font-semibold text-sm whitespace-nowrap">Apps</button>
                        <button class="nav-tab text-gray-500 px-4 py-3 font-semibold text-sm whitespace-nowrap">Photos</button>
                    </div>
                </div>

                <!-- Real File Input Trigger -->
                <div class="p-6 flex flex-col items-center justify-center shrink-0">
                    <div class="w-16 h-16 bg-blue-50 rounded-full flex items-center justify-center mb-3">
                        <i class="fas fa-folder-plus text-2xl text-blue-500"></i>
                    </div>
                    <input type="file" id="nativeFilePicker" multiple class="hidden" onchange="handleFileSelection(event)">
                    <button onclick="document.getElementById('nativeFilePicker').click()" class="bg-blue-500 text-white px-6 py-2.5 rounded-full font-bold shadow-md shadow-blue-200 active:scale-95 transition-transform">
                        Browse Device Storage
                    </button>
                </div>

                <!-- Selected Files List -->
                <div class="flex-1 overflow-y-auto px-3 pb-4 pt-2 custom-scrollbar" id="selectedFilesContainer">
                    <div class="flex flex-col items-center justify-center text-gray-400 mt-10 h-full">
                        <i class="fas fa-box-open text-4xl mb-3 opacity-50"></i>
                        <p class="text-sm">No items selected</p>
                    </div>
                </div>

                <!-- Bottom Action Bar -->
                <div class="w-full bg-white p-4 border-t shadow-[0_-4px_15px_rgba(0,0,0,0.05)] flex justify-between items-center z-20 shrink-0 relative mt-auto">
                    <div>
                        <p class="text-xs text-gray-500 mb-0.5">Selected</p>
                        <div class="flex items-center gap-2">
                            <span id="selectedCountBadge" class="bg-blue-500 text-white text-[10px] font-bold px-2 py-0.5 rounded-full hidden">0</span>
                            <span id="totalSizeTxt" class="font-bold text-gray-800 text-lg">0 B</span>
                        </div>
                    </div>
                    <button id="nextBtn" onclick="showConnectScreen()" class="bg-blue-500 text-white px-8 py-3 rounded-full font-bold shadow-lg shadow-blue-200 opacity-50 cursor-not-allowed transition-all flex items-center gap-2" disabled>
                        Send <i class="fas fa-paper-plane"></i>
                    </button>
                </div>
            </div>

            <!-- 3. Connect (Radar) Screen (SENDER) -->
            <div id="connectScreen" class="screen flex-col h-full w-full shareit-bg relative overflow-hidden text-white">
                <div class="p-4 flex items-center z-20 relative shrink-0">
                    <button onclick="showScreen('fileSelectScreen')" class="p-2 text-white hover:bg-white/20 rounded-full transition"><i class="fas fa-arrow-left"></i></button>
                    <h2 class="ml-4 text-lg font-bold">Select Receiver</h2>
                </div>

                <!-- Radar Animation -->
                <div class="flex-1 relative flex items-center justify-center overflow-hidden">
                    <div class="radar-circle"></div>
                    <div class="radar-circle"></div>
                    <div class="radar-circle"></div>
                    <div class="radar-circle"></div>
                    
                    <!-- Sender Avatar -->
                    <div class="z-10 w-20 h-20 bg-white rounded-full p-1 shadow-[0_0_20px_rgba(255,255,255,0.5)]">
                        <div class="w-full h-full bg-blue-100 rounded-full flex items-center justify-center border-2 border-blue-500">
                            <i class="fas fa-user-astronaut text-3xl text-blue-500"></i>
                        </div>
                    </div>
                </div>

                <!-- Search Input overlaying radar -->
                <div class="absolute bottom-8 w-full px-6 z-20">
                    <div class="bg-white/10 backdrop-blur-md border border-white/20 p-5 rounded-3xl text-center shadow-2xl">
                        <p class="text-sm font-medium mb-3">Enter receiver's ID or Scan QR</p>
                        
                        <!-- FIX: Input Box AND QR Scanner Button side-by-side -->
                        <div class="flex gap-2 mb-4">
                            <input type="text" id="receiverIdInput" placeholder="ID (e.g. A1B2C)" class="w-full text-center text-lg tracking-widest font-bold py-3.5 rounded-xl border-none outline-none uppercase bg-white text-blue-600 shadow-inner">
                            <button onclick="startQRScanner()" class="bg-blue-600 text-white w-16 rounded-xl flex items-center justify-center shadow-md active:scale-95 transition-transform" title="Scan QR Code">
                                <i class="fas fa-qrcode text-2xl"></i>
                            </button>
                        </div>
                        
                        <button onclick="connectToPeer()" id="connectBtn" class="bg-white text-blue-600 w-full py-3.5 rounded-xl font-bold shadow-lg active:scale-95 transition-transform text-lg">
                            Connect
                        </button>
                        <p id="connectError" class="text-red-200 text-xs mt-3 hidden font-medium"></p>
                    </div>
                </div>
            </div>

            <!-- 4. Receive Wait Screen (RECEIVER) -->
            <div id="receiveScreen" class="screen flex-col h-full w-full shareit-bg text-white relative overflow-hidden">
                <!-- Header properly separated -->
                <div class="p-4 flex items-center z-20 shrink-0 relative w-full">
                    <button onclick="goBackToHome()" class="p-2 text-white hover:bg-white/20 rounded-full transition"><i class="fas fa-arrow-left"></i></button>
                </div>
                
                <!-- Radar & QR Code Display -->
                <div class="flex-1 relative flex flex-col items-center justify-center z-10 text-center px-4 -mt-10">
                    <!-- Receiver Avatar -->
                    <div class="w-20 h-20 bg-white rounded-full p-1 shadow-[0_0_30px_rgba(255,255,255,0.6)] mx-auto mb-6 pulse-btn">
                        <div class="w-full h-full bg-blue-500 rounded-full flex items-center justify-center">
                            <i class="fas fa-satellite-dish text-3xl text-white"></i>
                        </div>
                    </div>
                    
                    <p class="text-blue-100 text-sm mb-2 font-medium">Your Device ID & QR Code</p>
                    
                    <div class="bg-white/20 backdrop-blur-md p-4 rounded-3xl border border-white/30 inline-block shadow-lg mb-6">
                        <div class="text-white text-4xl font-bold tracking-widest mb-4">
                            <span id="receiveDeviceId">...</span>
                        </div>
                        <!-- Safe QR Code Container -->
                        <div class="bg-white p-3 rounded-2xl mx-auto shadow-md w-fit h-fit" style="min-height: 150px; min-width: 150px; display:flex; align-items:center; justify-content:center;">
                            <div id="qrcode"></div>
                        </div>
                    </div>
                    
                    <p class="text-sm text-blue-100 animate-pulse font-medium">
                        <i class="fas fa-spinner fa-spin mr-2"></i> Waiting for sender to scan...
                    </p>
                </div>
            </div>

            <!-- 5. Transfer Progress Screen -->
            <div id="transferScreen" class="screen flex-col h-full w-full bg-gray-50 relative">
                <!-- Header -->
                <div class="shareit-bg p-4 flex items-center justify-between text-white shrink-0 shadow-md z-10">
                    <div class="flex items-center gap-3">
                        <button onclick="goBackToHome()" class="p-2 -ml-2 text-white hover:bg-white/20 rounded-full transition" title="Back to Home"><i class="fas fa-arrow-left"></i></button>
                        <h2 class="text-lg font-bold" id="transferTitle">Transferring</h2>
                    </div>
                    <div class="flex items-center gap-2 bg-white/20 px-3 py-1.5 rounded-full text-xs font-bold border border-white/30 backdrop-blur-sm">
                        <span class="w-2 h-2 rounded-ful
