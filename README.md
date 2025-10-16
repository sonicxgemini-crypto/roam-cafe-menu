<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RoamCafe (Create by Admin73)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800;900&display=swap" rel="stylesheet">
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <!-- NOTE: Replace YOUR_GOOGLE_MAPS_API_KEY with your actual key for map functionality -->
    <!-- The map will not display correctly without a valid key. For local testing, mock data is used. -->
    <script async defer src="https://maps.googleapis.com/maps/api/js?key=YOUR_GOOGLE_MAPS_API_KEY&callback=initMap"></script>
    <script>
        // Set dark mode based on local storage or system preference
        if (localStorage.getItem('theme') === 'dark' || (!('theme' in localStorage) && window.matchMedia('(prefers-color-scheme: dark)').matches)) {
            document.documentElement.classList.add('dark');
        } else {
            document.documentElement.classList.remove('dark');
        }
    </script>
    <style>
        body { font-family: 'Inter', sans-serif; }
        .backdrop-blur-panel { backdrop-filter: blur(16px); -webkit-backdrop-filter: blur(16px); }
        .loader {
            border: 5px solid #f3f3f3;
            border-top: 5px solid #f59e0b;
            border-radius: 50%;
            width: 40px;
            height: 40px;
            animation: spin 1s linear infinite;
        }
        html.dark .loader { border: 5px solid #4b5563; border-top: 5px solid #f59e0b; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        /* Toggle Switch Styles */
        .switch { position: relative; display: inline-block; width: 50px; height: 28px; }
        .switch input { opacity: 0; width: 0; height: 0; }
        .slider { position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0; background-color: #ccc; transition: .4s; border-radius: 28px; }
        .slider:before { position: absolute; content: ""; height: 20px; width: 20px; left: 4px; bottom: 4px; background-color: white; transition: .4s; border-radius: 50%; }
        input:checked + .slider { background-color: #f59e0b; }
        input:checked + .slider:before { transform: translateX(22px); }
        /* Custom scrollbar */
        #order-items-list, #options-modal, #item-editor-modal, #admin-dashboard-container {
            scrollbar-width: thin;
            scrollbar-color: #f59e0b transparent;
        }
        #order-items-list::-webkit-scrollbar, #options-modal::-webkit-scrollbar, #item-editor-modal::-webkit-scrollbar, #admin-dashboard-container::-webkit-scrollbar { width: 8px; }
        #order-items-list::-webkit-scrollbar-track, #options-modal::-webkit-scrollbar-track, #item-editor-modal::-webkit-scrollbar-track, #admin-dashboard-container::-webkit-scrollbar-track { background: transparent; }
        #order-items-list::-webkit-scrollbar-thumb, #options-modal::-webkit-scrollbar-thumb, #item-editor-modal::-webkit-scrollbar-thumb, #admin-dashboard-container::-webkit-scrollbar-thumb { background-color: #f59e0b; border-radius: 20px; border: 3px solid transparent; background-clip: content-box; }
        
        .option-btn { border-color: #e5e7eb; }
        html.dark .option-btn { border-color: #4b5563; }
        .option-btn.selected { border-color: #f59e0b; background-color: #f59e0b; color: #111827; font-weight: 600; }

        /* Admin Dashboard Tab Styling */
        .admin-tab {
            transition: all 0.2s ease-in-out;
            border-bottom: 3px solid transparent;
        }
        .admin-tab.active {
            color: #f59e0b;
            border-bottom-color: #f59e0b;
        }
        /* Map style for tracker */
        #map { height: 400px; width: 100%; border-radius: 1rem; }
    </style>
</head>
<body class="bg-slate-50 dark:bg-gray-900 text-gray-900 dark:text-gray-200 antialiased transition-colors duration-300">

    <!-- Auth Modal -->
    <div id="auth-container" class="fixed inset-0 min-h-screen flex items-center justify-center p-4 hidden bg-gray-900/50 backdrop-blur-sm z-[80]">
        <div class="w-full max-w-md bg-gray-800/90 rounded-xl shadow-2xl p-8 space-y-6">
            <div id="auth-header" class="text-center">
                <h1 class="text-3xl font-extrabold text-yellow-400">Staff & Admin Login</h1>
                <p id="auth-subtitle" class="text-gray-400 mt-1">Access the internal system or administrative tools.</p>
            </div>
            <div id="login-form-wrapper" class="space-y-6">
                <form id="login-form" class="space-y-4">
                    <div>
                        <label for="login-email" class="block text-sm font-medium text-gray-300">Email Address</label>
                        <input type="email" id="login-email" name="LoginEmail" required class="mt-1 block w-full px-4 py-2 bg-gray-700 border border-gray-600 rounded-lg text-white placeholder-gray-400 focus:ring-yellow-500 focus:border-yellow-500">
                    </div>
                    <div>
                        <label for="login-password" class="block text-sm font-medium text-gray-300">Password</label>
                        <input type="password" id="login-password" name="LoginPassword" required class="mt-1 block w-full px-4 py-2 bg-gray-700 border border-gray-600 rounded-lg text-white placeholder-gray-400 focus:ring-yellow-500 focus:border-yellow-500">
                    </div>
                    <button type="submit" id="login-btn" class="w-full py-3 px-4 bg-yellow-500 hover:bg-yellow-600 rounded-lg text-gray-900 font-bold transition duration-150 ease-in-out disabled:bg-gray-600 disabled:cursor-not-allowed flex items-center justify-center">
                        <span class="btn-text">Log In</span>
                    </button>
                </form>
                <button id="back-to-guest-order-btn" class="w-full py-2 px-4 bg-gray-600 hover:bg-gray-700 rounded-lg text-white font-bold transition duration-150 ease-in-out flex items-center justify-center mt-6">← Back to Order</button>
            </div>
        </div>
    </div>

    <!-- Main Ordering App -->
    <div id="app-container" class="hidden">
        <div class="container mx-auto px-4 py-8 relative">
            <!-- Control Buttons (Top Right) -->
            <div id="control-buttons" class="absolute top-4 right-4 flex items-center space-x-4 z-20">
                <div class="flex items-center space-x-2" title="Toggle Dark Mode">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" /></svg>
                    <label for="dark-mode-toggle-checkbox" class="switch">
                        <input type="checkbox" id="dark-mode-toggle-checkbox">
                        <span class="slider"></span>
                    </label>
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20.354 15.354A9 9 0 018.646 3.646 9.003 9.003 0 0012 21a9.003 9.003 0 008.354-5.646z" /></svg>
                </div>
                <div class="h-6 w-px bg-gray-300 dark:bg-gray-600"></div>
                <button id="dashboard-btn" title="Admin Dashboard" type="button" class="hidden p-2 rounded-lg text-gray-900 bg-yellow-400 hover:bg-yellow-500 focus:outline-none focus:ring-4 focus:ring-yellow-300"><svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z" /></svg></button>
                <button id="open-tracker-btn" title="Track Your Order" type="button" class="p-2 rounded-lg text-gray-900 bg-yellow-400 hover:bg-yellow-500 focus:outline-none focus:ring-4 focus:ring-yellow-300">
                    <svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <path d="M12 22s-8-4.5-8-11.8A8 8 0 0 1 12 2a8 8 0 0 1 8 8.2c0 7.3-8 11.8-8 11.8z" />
                        <circle cx="12" cy="10" r="3" />
                    </svg>
                </button>
                <button id="staff-login-btn" title="Staff/Admin Login" type="button" class="p-2 rounded-lg text-gray-900 bg-yellow-400 hover:bg-yellow-500 focus:outline-none focus:ring-4 focus:ring-yellow-300"><svg class="w-6 h-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M15.75 6a3.75 3.75 0 11-7.5 0 3.75 3.75 0 017.5 0zM4.501 20.118a7.5 7.5 0 0114.998 0A17.933 17.933 0 0112 21.75c-2.676 0-5.216-.584-7.499-1.632z" /></svg></button>
                <button id="logout-btn" title="Logout" type="button" class="hidden p-2 rounded-lg text-gray-900 bg-white dark:bg-gray-800 hover:bg-gray-100 dark:hover:bg-gray-700"><svg class="w-6 h-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" d="M15.75 9V5.25A2.25 2.25 0 0013.5 3h-6a2.25 2.25 0 00-2.25 2.25v13.5A2.25 2.25 0 007.5 21h6a2.25 2.25 0 002.25-2.25V15m3 0l3-3m0 0l-3-3m3 3H9" /></svg></button>
            </div>
            
            <!-- Header -->
            <header class="text-center mb-10 pt-12 flex flex-col items-center">
                <img src="https://media.licdn.com/dms/image/v2/C4E0BAQGZl3iCXl50TA/company-logo_200_200/company-logo_200_200/0/1655697183236?e=2147483647&v=beta&t=uU1m3DkGHwtarQUQq8jfwiSb9kbT6OQlH7-xyirp1P0" alt="Tube Coffee Logo" class="w-24 h-24 mb-4 rounded-full shadow-lg">
                <h1 class="text-4xl md:text-5xl font-black tracking-tight text-yellow-500 drop-shadow-lg">Tube Coffee</h1>
                <p id="app-user-status" class="text-xl italic font-semibold mt-2 tracking-wider drop-shadow-lg">Order System</p>
            </header>
            
            <!-- Main Content Grid -->
            <main class="grid grid-cols-1 lg:grid-cols-3 gap-8 lg:gap-12">
                
                <!-- Menu Section (Left 2/3) -->
                <div id="menu-section" class="lg:col-span-2">
                    <div id="menu-content" class="mt-8"></div>
                </div>
                
                <!-- Order Sidebar (Right 1/3) -->
                <aside id="order-sidebar-wrapper" class="hidden lg:col-span-1 lg:block">
                    <div id="order-content-panel" class="lg:sticky lg:top-8 bg-white/90 dark:bg-gray-800/85 rounded-2xl shadow-2xl p-6 backdrop-blur-panel">
                        <button id="order-close-btn" title="Close Cart" class="lg:hidden absolute top-4 right-4 text-gray-500 hover:text-gray-900 dark:text-gray-400 dark:hover:text-white"><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" /></svg></button>
                        <div class="flex justify-between items-center border-b border-gray-200 dark:border-gray-600 pb-3 mb-4">
                            <h2 class="text-2xl font-extrabold">Your Order</h2>
                            <button id="clear-cart-btn" title="Clear Cart" class="text-sm font-semibold text-red-500 hover:text-red-400 hidden">Clear All</button>
                        </div>
                        <div id="order-items-list" class="space-y-4 max-h-[40vh] overflow-y-auto pr-2">
                            <p id="empty-order-message" class="text-gray-500 dark:text-gray-400 text-center py-10">Your cart is empty.</p>
                        </div>
                        <div id="totals-section" class="mt-6 border-t border-gray-200 dark:border-gray-600 pt-4 space-y-3" style="display: none;">
                            <div class="flex justify-between items-center text-lg">
                                <span>Total (USD):</span>
                                <span id="total-usd" class="font-bold text-xl text-green-600 dark:text-green-400">$0.00</span>
                            </div>
                            <div class="flex justify-between items-center text-lg">
                                <span>Total (KHR):</span>
                                <span id="total-khr" class="font-bold text-xl text-cyan-600 dark:text-cyan-400">0៛</span>
                            </div>
                        </div>
                        <button id="place-order-btn" class="mt-6 w-full bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-3 rounded-lg text-lg disabled:bg-gray-400 dark:disabled:bg-gray-600 disabled:cursor-not-allowed" disabled>Place Order</button>
                    </div>
                </aside>
            </main>
        </div>
    </div>
    
    <!-- Floating Cart Button (Mobile) -->
    <button id="floating-cart-btn" class="fixed bottom-6 right-6 z-40 bg-white dark:bg-gray-800 text-gray-900 dark:text-white w-16 h-16 rounded-2xl shadow-2xl transition duration-300 transform hover:scale-105 flex items-center justify-center lg:hidden">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.28 2.28c-.48.48-.48 1.28 0 1.76l2.28 2.28" /></svg>
        <span id="cart-item-count-badge" class="absolute -top-2 -right-2 w-8 h-8 text-base font-bold text-white transform bg-green-600 rounded-full flex items-center justify-center border-2 border-white dark:border-gray-800 hidden">+0</span>
    </button>

    <!-- Admin Dashboard Modal -->
    <div id="admin-dashboard-container" class="hidden fixed inset-0 bg-slate-50 dark:bg-gray-900 z-50 overflow-y-auto">
        <div class="container mx-auto px-4 sm:px-6 lg:px-8 py-8">
            <header class="flex flex-wrap justify-between items-center gap-4 mb-8 pt-12">
                <h1 class="text-4xl md:text-5xl font-black tracking-tight text-yellow-500 drop-shadow-lg">Admin Dashboard</h1>
                <button id="back-to-app-btn" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-2 px-6 rounded-lg flex items-center"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>Back to POS</button>
            </header>

            <div class="border-b border-gray-200 dark:border-gray-700 mb-8">
                <nav class="-mb-px flex space-x-6" id="admin-tabs">
                    <button data-tab="dashboard-content" class="admin-tab active py-4 px-1 text-lg font-semibold">Dashboard</button>
                    <button data-tab="menu-management" class="admin-tab py-4 px-1 text-lg font-semibold">Menu Management</button>
                    <button data-tab="user-management" class="admin-tab py-4 px-1 text-lg font-semibold">User Management</button>
                </nav>
            </div>

            <div id="admin-tab-content">
                <!-- Dashboard Panel -->
                <div id="dashboard-content-panel">
                    <div id="dashboard-stats" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
                    </div>
                </div>
                <!-- Menu Management Panel -->
                <div id="menu-management-panel" class="hidden">
                    <div class="flex justify-end mb-6">
                        <button id="add-new-item-btn" class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-6 rounded-lg flex items-center">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 6v6m0 0v6m0-6h6m-6 0H6" /></svg>
                            Add New Item
                        </button>
                    </div>
                    <div id="menu-management-list" class="space-y-8">
                    </div>
                </div>
                <!-- User Management Panel -->
                <div id="user-management-panel" class="hidden space-y-8">
                    <div class="bg-white/80 dark:bg-gray-800/80 backdrop-blur-panel p-6 rounded-xl shadow-lg">
                        <h2 class="text-2xl font-bold mb-4">Staff & User Accounts</h2>
                        <div id="user-management-table" class="overflow-x-auto"></div>
                    </div>
                    <div class="bg-white/80 dark:bg-gray-800/80 backdrop-blur-panel p-6 rounded-xl shadow-lg">
                        <h2 class="text-2xl font-bold mb-4">Administrator Accounts</h2>
                        <div id="admin-management-content">
                            <div class="mb-4">
                                <label for="new-admin-email" class="block font-medium mb-1">Promote User to Admin</label>
                                <div class="flex gap-2">
                                    <input type="email" id="new-admin-email" placeholder="Enter user's email" class="flex-grow px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500">
                                    <button id="add-admin-btn" class="bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-lg">Add Admin</button>
                                </div>
                            </div>
                            <div>
                                <h3 class="font-semibold mb-2">Current Admins</h3>
                                <ul id="admin-list" class="space-y-2"></ul>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
    
    <!-- Alert/Confirm Modal -->
    <div id="alert-modal" class="fixed inset-0 bg-black bg-opacity-60 z-[90] items-center justify-center p-4 hidden">
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-2xl w-full max-w-sm text-center p-6">
            <p id="alert-message" class="mb-6 text-lg"></p>
            <div id="alert-actions">
                <button id="alert-ok-btn" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-2 px-6 rounded-lg">OK</button>
                <button id="alert-confirm-btn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-6 rounded-lg hidden">Confirm</button>
                <button id="alert-cancel-btn" class="bg-gray-300 hover:bg-gray-400 dark:bg-gray-600 dark:hover:bg-gray-500 font-bold py-2 px-6 rounded-lg hidden ml-2">Cancel</button>
            </div>
        </div>
    </div>

    <!-- Receipt Modal -->
    <div id="receipt-modal" class="fixed inset-0 bg-black bg-opacity-60 z-[70] items-center justify-center p-4 hidden">
        <div id="receipt-content" class="bg-white dark:bg-gray-800 rounded-lg shadow-2xl w-full max-w-sm p-6 space-y-4 text-gray-900 dark:text-gray-200">
            <div class="text-center">
                <img src="https://media.licdn.com/dms/image/v2/C4E0BAQGZl3iCXl50TA/company-logo_200_200/company-logo_200_200/0/1655697183236?e=2147483647&v=beta&t=uU1m3DkGHwtarQUQq8jfwiSb9kbT6OQlH7-xyirp1P0" alt="Tube Coffee Logo" class="w-12 h-12 mx-auto mb-4 rounded-full">
                <h2 class="text-2xl font-extrabold">Order Receipt</h2>
                <div class="text-sm text-gray-500 dark:text-gray-400 space-y-1 mt-2" id="receipt-header-info"></div>
            </div>
            <div id="receipt-items-list" class="space-y-3 pt-4 border-t border-gray-200 dark:border-gray-700"></div>
            <div id="receipt-totals" class="pt-3 border-t border-gray-200 dark:border-gray-700 space-y-2"></div>
            <div class="flex flex-col items-center justify-center space-y-2 pt-4 border-t border-gray-200 dark:border-gray-700">
                <div class="qr-upload-box flex items-center justify-center h-36 w-36 bg-slate-100 dark:bg-gray-700 rounded-lg cursor-pointer text-center relative overflow-hidden" onclick="handleQrUploadClick()">
                    <span id="qr-placeholder-text" class="text-sm font-semibold text-gray-500 dark:text-gray-400">Upload QR</span>
                    <input type="file" id="qr-input" accept="image/*" class="absolute inset-0 opacity-0 cursor-pointer" onchange="previewQr(event)">
                    <img id="qr-preview" class="absolute inset-0 w-full h-full object-contain hidden" alt="Uploaded QR Code">
                </div>
                <p class="text-xs text-gray-400">Click to upload payment QR code</p>
            </div>
            <p class="text-sm text-center text-gray-500 dark:text-gray-400 pt-2">Thank you for your order!</p>
            <div class="flex justify-center space-x-3 pt-4">
                <button id="download-receipt-btn" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-2 px-4 rounded-lg">Download Receipt</button>
                <button id="close-receipt-btn" class="bg-gray-300 hover:bg-gray-400 text-gray-900 dark:bg-gray-700 dark:hover:bg-gray-600 dark:text-white font-bold py-2 px-4 rounded-lg">Close</button>
            </div>
        </div>
    </div>

    <!-- Options Modal -->
    <div id="options-modal" class="fixed inset-0 bg-black bg-opacity-60 z-[70] items-center justify-center p-4 hidden">
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-2xl w-full max-w-lg text-left p-6 space-y-4 relative max-h-[90vh] overflow-y-auto">
            <button id="close-options-modal-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600 dark:hover:text-white z-10">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
            </button>
            <div class="flex items-start space-x-4">
                <img id="options-modal-img" src="" alt="Product Image" class="w-24 h-24 object-cover rounded-lg">
                <div>
                    <h2 id="options-modal-title" class="text-2xl font-extrabold">Item Name</h2>
                    <p id="options-modal-base-price" class="text-lg text-gray-500 dark:text-gray-400"></p>
                </div>
            </div>
            <div id="options-modal-content" class="space-y-5"></div>
            <div class="pt-4 border-t dark:border-gray-700 flex items-center justify-between">
                <div class="flex items-center space-x-3">
                    <button id="options-quantity-decrease" class="text-xl font-bold w-10 h-10 bg-gray-200 dark:bg-gray-700 rounded-full">-</button>
                    <span id="options-quantity" class="font-bold text-xl w-8 text-center">1</span>
                    <button id="options-quantity-increase" class="text-xl font-bold w-10 h-10 bg-gray-200 dark:bg-gray-700 rounded-full">+</button>
                </div>
                <button id="add-configured-item-btn" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-3 px-8 rounded-lg text-lg">
                    Add to Cart - <span id="options-modal-total-price">$0.00</span>
                </button>
            </div>
        </div>
    </div>

    <!-- Item Editor Modal (Admin) -->
    <div id="item-editor-modal" class="fixed inset-0 bg-black bg-opacity-60 z-[90] items-center justify-center p-4 hidden">
        <div class="bg-white dark:bg-gray-800 rounded-lg shadow-2xl w-full max-w-2xl text-left p-6 space-y-4 relative max-h-[90vh] overflow-y-auto">
            <button id="close-item-editor-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600 dark:hover:text-white z-10"><svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg></button>
            <h2 id="item-editor-title" class="text-2xl font-extrabold mb-4">Add New Item</h2>
            <form id="item-editor-form" class="space-y-4">
                <input type="hidden" id="item-id-input">
                <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <div>
                        <label for="item-name" class="block text-sm font-medium mb-1">Item Name</label>
                        <input type="text" id="item-name" required class="w-full px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500">
                    </div>
                    <div>
                        <label for="item-category" class="block text-sm font-medium mb-1">Category</label>
                        <input type="text" id="item-category" required class="w-full px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500" placeholder="e.g., ☕ Coffee">
                    </div>
                </div>
                <div>
                    <label for="item-price" class="block text-sm font-medium mb-1">Base Price (USD)</label>
                    <input type="number" id="item-price" step="0.01" required class="w-full px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500">
                    <p class="text-xs text-gray-500 mt-1">Set to 0 if price is determined by options (e.g., Size).</p>
                </div>
                <div>
                    <label for="item-img" class="block text-sm font-medium mb-1">Image URL</label>
                    <input type="url" id="item-img" class="w-full px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500">
                </div>
                <div>
                    <label for="item-options" class="block text-sm font-medium mb-1">Options (JSON format)</label>
                    <textarea id="item-options" rows="6" class="w-full px-3 py-2 bg-gray-100 dark:bg-gray-700 border border-gray-300 dark:border-gray-600 rounded-lg focus:ring-yellow-500 focus:border-yellow-500 font-mono text-sm"></textarea>
                    <p class="text-xs text-gray-500 mt-1">Leave blank for no options. Use valid JSON. Check console for formatting errors on save.</p>
                </div>
                <div class="flex justify-end space-x-3 pt-4 border-t dark:border-gray-700">
                    <button type="button" id="save-and-add-new-btn" class="bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-6 rounded-lg">Save & Add New</button>
                    <button type="submit" id="save-item-btn" class="bg-yellow-500 hover:bg-yellow-600 text-black font-bold py-2 px-6 rounded-lg">Save Item</button>
                </div>
            </form>
        </div>
    </div>
    
    <!-- Tracker Modal -->
    <div id="tracker-modal" class="hidden fixed inset-0 bg-black bg-opacity-60 z-[70] items-center justify-center p-4">
        <div class="w-full max-w-4xl bg-white dark:bg-gray-800 shadow-2xl rounded-xl p-8 space-y-8 relative">
            <button id="close-tracker-modal-btn" class="absolute top-4 right-4 text-gray-400 hover:text-gray-600 dark:hover:text-white">
                <svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
            </button>

            <header class="text-center space-y-2">
                <h1 class="text-3xl font-extrabold text-gray-900 dark:text-white tracking-tight">☕ Tube Coffee Tracker</h1>
                <p class="text-gray-500 dark:text-gray-400 text-lg">Enter your order ID to view the live status and map location.</p>
            </header>

            <div class="flex flex-col sm:flex-row gap-4">
                <input type="text" id="order-id-input" placeholder="e.g., TUBE-1002"
                        class="flex-grow p-4 border border-gray-300 dark:border-gray-600 rounded-lg focus:outline-none focus:ring-4 focus:ring-yellow-300 text-lg text-gray-900 dark:text-gray-200 bg-white dark:bg-gray-700">
                <button id="track-order-btn"
                        class="bg-yellow-500 hover:bg-yellow-600 text-gray-900 font-bold py-4 px-8 rounded-lg transition duration-150 ease-in-out disabled:bg-gray-400 flex items-center justify-center text-lg">
                    <span class="btn-text">Track Order</span>
                </button>
            </div>

            <div id="order-status-panel" class="hidden border-t border-gray-200 dark:border-gray-700 pt-8 space-y-6">

                <div id="status-card" class="p-4 rounded-lg shadow-md" role="alert">
                    <p class="font-bold text-xl mb-1">Status: <span id="display-status"></span></p>
                    <p id="display-details"></p>
                    <p id="display-meta-info" class="mt-2 text-sm">Item: <span id="display-item-name"></span> for <span id="display-customer-name"></span></p>
                </div>

                <div>
                    <h3 class="text-2xl font-bold mb-4 text-gray-800 dark:text-gray-200">Live Location</h3>
                    <div id="map" class="h-96 w-full rounded-xl bg-gray-200 dark:bg-gray-700"></div>
                    <p id="map-message" class="mt-4 text-center text-red-500 hidden font-medium">Map not available for this status.</p>
                </div>
            </div>
        </div>
    </div>
    
    <script>
    // --- MOCK BACKEND FOR LOCAL PREVIEW ---
    const MOCK_DELIVERY_DATA = [
        { id: 'TUBE-1001', customerName: 'Alice', itemName: 'Coffee Bundle', status: 'READY', details: 'Order finalized. Awaiting pickup by driver.', lat: 37.7888, lng: -122.4019 },
        { id: 'TUBE-1002', customerName: 'Bob', itemName: 'Matcha Kit', status: 'IN_TRANSIT', details: 'Driver John is 5 minutes away from your location.', lat: 37.7766, lng: -122.4206 },
        { id: 'TUBE-1003', customerName: 'Charlie', itemName: 'Pastry Box', status: 'DELIVERED', details: 'Successfully delivered on 2024-05-10 at 14:30.', lat: 37.7951, lng: -122.3957 },
        { id: 'TUBE-1004', customerName: 'Diana', itemName: 'Latte', status: 'CANCELED', details: 'Order canceled by customer.', lat: 0, lng: 0 }
    ];
    
    const MOCK_SALARY_DATA = [
        { name: 'Admin User', email: 'admin@tube.com', daily_rate_usd: 20.00 },
        { name: 'Staff Member', email: 'staff@tube.com', daily_rate_usd: 15.00 }
    ];

    if (typeof google === 'undefined' || typeof google.script === 'undefined') {
        const MOCK_DELAY = 300;
        let MOCK_MENU_DATA = { 
            "✨ NEW PRODUCTS": [ { id: "NP1", name: "Egg Tart", price: 3.00, img: "https://images.unsplash.com/photo-1558553535-97d441136b9a?q=80&w=800", category: "✨ NEW PRODUCTS", options: { "Size": [{ name: "2个", price: 3.00, isBasePrice: true }, { name: "4个", price: 5.00, isBasePrice: true }] }}],
            "☕ Coffee": [ { id: "C1", name: "Americano", price: 2.50, img: "https://images.unsplash.com/photo-1511920183276-5941c3e34254?q=80&w=800", category: "☕ Coffee", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Add": [{ name: "Nothing", price: 0 }, { name: "1 Portion Coffee", price: 0.5 }, { name: "Double Portion Coffee", price: 1.0 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "C2", name: "Latte", price: 3.00, img: "https://images.unsplash.com/photo-1561651823-2395383521a2?q=80&w=800", category: "☕ Coffee", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Add": [{ name: "Nothing", price: 0 }, { name: "Honey", price: 0.5 }, { name: "Vanilla Syrup", price: 0.5 }, { name: "Caramel Syrup", price: 0.5 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "C3", name: "Cappuccino", price: 3.00, img: "https://images.unsplash.com/photo-1557006021-b4ab79a552d8?q=80&w=800", category: "☕ Coffee", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Add": [{ name: "Nothing", price: 0 }, { name: "Cinnamon", price: 0.25 }, { name: "Chocolate Powder", price: 0.25 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "C4", name: "Mocha", price: 3.50, img: "https://images.unsplash.com/photo-1597036523927-4340d99eda6d?q=80&w=800", category: "☕ Coffee", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Add": [{ name: "Nothing", price: 0 }, { name: "Whipped Cream", price: 0.75 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "C5", name: "Espresso", price: 2.00, img: "https://images.unsplash.com/photo-1610889556528-95f3b1451935?q=80&w=800", category: "☕ Coffee" } ],
            "🍵 Tea & Matcha": [ { id: "T1", name: "Matcha Latte", price: 3.75, img: "https://images.unsplash.com/photo-1541533267755-d1836c53e414?q=80&w=800", category: "🍵 Tea & Matcha", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Add": [{ name: "Nothing", price: 0 }, { name: "Pearl", price: 0.5 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "T2", name: "Iced Passion Tea", price: 2.75, img: "https://images.unsplash.com/photo-1629891892103-6c8410298920?q=80&w=800", category: "🍵 Tea & Matcha", options: { "Add": [{ name: "Nothing", price: 0 }, { name: "Aloe Vera", price: 0.5 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "T3", name: "Peach Oolong Tea", price: 3.25, img: "https://images.unsplash.com/photo-1627435641363-f257b1c3e1e9?q=80&w=800", category: "🍵 Tea & Matcha", options: { "Type": ["🧊 Ice", "🔥 Hot"], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }} ],
            "🥤 Frappe & Ice Blended": [ { id: "F1", name: "Caramel Frappe", price: 4.50, img: "https://images.unsplash.com/photo-1572490122747-3968b75cc699?q=80&w=800", category: "🥤 Frappe & Ice Blended", options: { "Add": [{ name: "Nothing", price: 0 }, { name: "Whipped Cream", price: 0.75 }, { name: "Extra Caramel Drizzle", price: 0.5 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "F2", name: "Java Chip Frappe", price: 4.75, img: "https://images.unsplash.com/photo-1542849187-5ec6ea5e6a27?q=80&w=800", category: "🥤 Frappe & Ice Blended", options: { "Add": [{ name: "Nothing", price: 0 }, { name: "Whipped Cream", price: 0.75 }, { name: "Extra Java Chips", price: 0.5 }], "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }}, { id: "F3", name: "Strawberry Smoothie", price: 4.00, img: "https://images.unsplash.com/photo-1624823129929-b9d979a09325?q=80&w=800", category: "🥤 Frappe & Ice Blended", options: { "Sugar Level": ["0%", "25%", "50%", "75%", "100%", "125%"] }} ],
            "🍰 Desserts": [ { id: "D1", name: "Egg Tart", price: 3.00, img: "https://images.unsplash.com/photo-1558553535-97d441136b9a?q=80&w=800", category: "🍰 Desserts", options: { "Size": [{ name: "2个", price: 3.00, isBasePrice: true }, { name: "4个", price: 5.00, isBasePrice: true }] }}, { id: "P1", name: "Butter Croissant", price: 2.25, img: "https://images.unsplash.com/photo-1530983103984-22415099178a?q=80&w=800", category: "🍰 Desserts" }, { id: "P2", name: "Chocolate Danish", price: 2.75, img: "https://images.unsplash.com/photo-1617443129654-20b57117e36c?q=80&w=800", category: "🍰 Desserts" } ]
        };
        let MOCK_USERS = [ { name: 'Admin User', email: 'admin@tube.com', status: 'allowed', role: 'admin', registered: '2023-10-26' }, { name: 'Staff Member', email: 'staff@tube.com', status: 'allowed', role: 'user', registered: '2023-10-25' }, { name: 'Blocked User', email: 'blocked@tube.com', status: 'blocked', role: 'user', registered: '2023-10-24' }];
        const MOCK_DASHBOARD_STATS = { totalRevenue: 12530.50, totalOrders: 830, topSeller: { name: 'Latte', count: 450 }};

        let successHandler = (r) => console.log("Mock Success:", r), failureHandler = (e) => console.error("Mock Failure:", e);
        const mockRunner = {
            withSuccessHandler: (h) => (successHandler = h, mockRunner), withFailureHandler: (h) => (failureHandler = h, mockRunner),
            getMenuItems: () => setTimeout(() => successHandler(MOCK_MENU_DATA), MOCK_DELAY),
            validateUser: (c) => setTimeout(() => { const u = MOCK_USERS.find(i=>i.email.toLowerCase()===c.email.toLowerCase()); if(u&&u.status!=='blocked'){successHandler({status:'success',role:u.role,name:u.name,email:u.email})}else{successHandler({status:'error',message:'Invalid credentials or user blocked'})}}, MOCK_DELAY),
            placeOrder: (d) => setTimeout(() => successHandler({status:'success',orderId:`MOCK-${Date.now()}`,orderData:d}), MOCK_DELAY),
            getAppUsers: () => setTimeout(() => successHandler(MOCK_USERS), MOCK_DELAY),
            getAdminUsers: () => setTimeout(() => successHandler(MOCK_USERS.filter(u=>u.role==='admin').map(u=>u.email)), MOCK_DELAY),
            updateUserStatus:(e,s)=>setTimeout(()=>(MOCK_USERS.find(u=>u.email===e).status=s,successHandler({success:true,message:`Status updated for ${e}.`})),MOCK_DELAY),
            deleteUser:(e)=>setTimeout(()=>(MOCK_USERS=MOCK_USERS.filter(u=>u.email!==e),successHandler({success:true,message:`${e} deleted.`})),MOCK_DELAY),
            addAdmin:(e)=>setTimeout(()=>{const u=MOCK_USERS.find(u=>u.email===e);if(u){u.role='admin';successHandler({success:true,message:`${e} promoted.`})} else {successHandler({success:false,message:`User ${e} not found.`})}}, MOCK_DELAY),
            removeAdmin:(e)=>setTimeout(()=>{const u=MOCK_USERS.find(u=>u.email===e);if(u){u.role='user';successHandler({success:true,message:`${e} demoted.`})} else {successHandler({success:false,message:`User ${e} not found.`})}}, MOCK_DELAY),
            getDashboardStats: () => setTimeout(() => successHandler(MOCK_DASHBOARD_STATS), MOCK_DELAY),
            saveMenuItem: (itemData) => setTimeout(() => {
                // Remove existing item across all categories
                Object.keys(MOCK_MENU_DATA).forEach(cat => { MOCK_MENU_DATA[cat] = MOCK_MENU_DATA[cat].filter(i => i.id !== itemData.id); });
                // Add item to its category (create category if new)
                if (!MOCK_MENU_DATA[itemData.category]) { MOCK_MENU_DATA[itemData.category] = []; }
                const existingIndex = MOCK_MENU_DATA[itemData.category].findIndex(i => i.id === itemData.id);
                if (existingIndex > -1) { MOCK_MENU_DATA[itemData.category][existingIndex] = itemData; }
                else { MOCK_MENU_DATA[itemData.category].push(itemData); }
                successHandler({success: true, message: 'Item saved successfully!'});
            }, MOCK_DELAY),
            deleteMenuItem: (itemId) => setTimeout(() => {
                Object.keys(MOCK_MENU_DATA).forEach(cat => { MOCK_MENU_DATA[cat] = MOCK_MENU_DATA[cat].filter(i => i.id !== itemId); });
                successHandler({success: true, message: 'Item deleted successfully!'});
            }, MOCK_DELAY),
            calculateSalaries: () => setTimeout(() => {
                const dailyRate = (u) => parseFloat(u.daily_rate_usd) || 0;
                const totalDays = 31;
                const mockResults = MOCK_SALARY_DATA.map(staff => ({
                    name: staff.name, email: staff.email, dailyRate: dailyRate(staff),
                    totalDays: totalDays, dailySalary: dailyRate(staff), monthlySalary: dailyRate(staff) * totalDays
                }));
                successHandler({ status: 'success', data: mockResults, month: 'Mock October 2025' });
            }, MOCK_DELAY),
            getDeliveryStatus: (orderId) => {
                setTimeout(() => {
                    const order = MOCK_DELIVERY_DATA.find(item => item.id === orderId.toUpperCase().trim());
                    if (order) { successHandler({ status: 'success', data: order }); } 
                    else if (orderId.startsWith('MOCK-')) {
                        const mockStatus = parseInt(orderId.split('-')[1]) % 3 === 0 ? 'READY' : 'IN_TRANSIT';
                        successHandler({ status: 'success', data: { id: orderId, customerName: 'Recent Customer', itemName: 'Mixed Order', status: mockStatus, details: `Tracing live mock order: ${orderId}`, lat: 37.7766, lng: -122.4206 } });
                    } else { successHandler({ status: 'error', message: 'Order ID not found. Please check and try again.' }); }
                }, MOCK_DELAY);
            }
        };
        window.google = { script: { run: mockRunner } };
    }

    document.addEventListener('DOMContentLoaded', () => {
        // --- GLOBAL STATE ---
        let currentUser = null, order = [], menuData = {}, confirmCallback = null;
        let currentModalItem = null, currentModalSelections = {};
        const KHR_CONVERSION_RATE = 4100;
        const el = (id) => document.getElementById(id);
        
        let lastPlacedOrderId = localStorage.getItem('lastPlacedOrderId') || 'TUBE-1002';

        // --- TRACKER GLOBALS ---
        let map;
        let marker;
        let mapInitialized = false;

        const STATUS_COLORS = {
            'READY': { border: 'border-yellow-400', background: 'bg-yellow-50 dark:bg-yellow-900/50', text: 'text-yellow-800 dark:text-yellow-300' },
            'IN_TRANSIT': { border: 'border-green-400', background: 'bg-green-50 dark:bg-green-900/50', text: 'text-green-800 dark:text-green-300' },
            'DELIVERED': { border: 'border-blue-400', background: 'bg-blue-50 dark:bg-blue-900/50', text: 'text-blue-800 dark:text-blue-300' },
            'CANCELED': { border: 'border-red-400', background: 'bg-red-50 dark:bg-red-900/50', text: 'text-red-800 dark:text-red-300' }
        };

        // --- EXPONENTIAL BACKOFF WRAPPER ---
        /**
         * Runs a Google Apps Script function with exponential backoff on failure.
         * Note: This implementation is for client-side resilience and does not use fetch/API.
         */
        async function googleScriptRun(functionName, args = [], button = null, maxRetries = 3) {
            if (button) {
                // Store original HTML for restoration
                button.dataset.originalHtml = button.innerHTML; 
                setLoadingState(button, true);
            }

            for (let i = 0; i < maxRetries; i++) {
                try {
                    // Use a Promise wrapper to handle the asynchronous nature of google.script.run
                    const result = await new Promise((resolve, reject) => {
                        const runner = google.script.run
                            .withSuccessHandler(resolve)
                            .withFailureHandler(reject);
                        
                        // Dynamically call the function with arguments
                        runner[functionName].apply(runner, args);
                    });

                    // Success: exit loop and return result
                    if (button) setLoadingState(button, false, button.dataset.originalHtml);
                    return result;

                } catch (error) {
                    if (i === maxRetries - 1) {
                        // Final failure: show alert
                        if (button) setLoadingState(button, false, button.dataset.originalHtml);
                        showAlert(`Server error calling ${functionName} after ${maxRetries} attempts: ${error.message}`);
                        throw error; 
                    }
                    // Calculate delay: 2^retry_count * 1000 ms (1s, 2s, 4s)
                    const delay = Math.pow(2, i) * 1000;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }


        // --- INITIALIZATION ---
        function initializeApp() {
            const savedUser = sessionStorage.getItem('currentUser');
            try { currentUser = savedUser ? JSON.parse(savedUser) : null; } catch (e) { currentUser = null; }
            
            const savedOrder = localStorage.getItem('tubeCoffeeOrder');
            try { order = savedOrder ? JSON.parse(savedOrder) : []; } catch(e) { order = []; }

            updateUIVisibility();
            loadMenu();
            renderOrder();
            setupEventListeners();
        }

        // --- UI & RENDER FUNCTIONS ---
        function updateUIVisibility() {
            el('app-container').classList.remove('hidden');
            el('auth-container').classList.add('hidden');
            el('admin-dashboard-container').classList.add('hidden');
            const isGuest = !currentUser || currentUser.role === 'guest';
            el('staff-login-btn').classList.toggle('hidden', !isGuest);
            el('logout-btn').classList.toggle('hidden', isGuest);
            el('dashboard-btn').classList.toggle('hidden', !currentUser || currentUser.role !== 'admin');
            
            if (isGuest) {
                currentUser = { name: 'Guest User', email: 'guest@ordering.system', role: 'guest' };
                el('app-user-status').textContent = "Ordering as Guest";
            } else {
                el('app-user-status').textContent = currentUser.role === 'admin' ? `Welcome, Admin ${currentUser.name}` : `Welcome, Staff ${currentUser.name}`;
            }
        }

        async function loadMenu() {
            el('menu-content').innerHTML = '<div class="flex justify-center p-10"><div class="loader"></div></div>';
            try {
                const data = await googleScriptRun('getMenuItems');
                if (data.error) {
                    el('menu-content').innerHTML = `<p class="text-center text-red-500">${data.error}</p>`;
                    return;
                }
                menuData = data;
                renderMenu();
            } catch (e) {
                // Error handled by googleScriptRun
                el('menu-content').innerHTML = `<p class="text-center text-red-500">Failed to load menu data.</p>`;
            }
        }
        
        function renderMenu() {
            let html = '';
            const sortedCategories = Object.keys(menuData).sort();
            sortedCategories.forEach(cat => {
                if (!menuData[cat] || menuData[cat].length === 0) return;
                html += `<div class="text-center my-8 py-4 border-t border-b border-gray-200 dark:border-gray-700"><h2 class="text-3xl font-extrabold text-gray-800 dark:text-white">${cat}</h2></div><p class="text-sm text-gray-500 dark:text-gray-400 mb-6 text-center">Showing all ${menuData[cat].length} results</p><div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-3 gap-8">`;
                menuData[cat].forEach(item => {
                    const defaultImg = 'https://placehold.co/600x400/EEE/333?text=Tube+Coffee';
                    const imgSrc = item.img && item.img.startsWith('http') ? item.img : defaultImg;
                    const buttonText = item.options ? 'Select options' : 'Add to Order';
                    let priceDisplay = '';
                    if (item.options && item.options.Size && item.options.Size.every(s => typeof s === 'object' && s.isBasePrice)) {
                        const prices = item.options.Size.map(s => s.price);
                        priceDisplay = `$${Math.min(...prices).toFixed(2)} - $${Math.max(...prices).toFixed(2)}`;
                    } else {
                        priceDisplay = item.price ? '$' + item.price.toFixed(2) : 'N/A';
                    }
                    html += `<div class="bg-white dark:bg-gray-800 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 flex flex-col text-center"><img src="${imgSrc}" alt="${item.name}" onerror="this.onerror=null;this.src='${defaultImg}';" class="w-full h-64 object-cover rounded-t-lg"><div class="p-5 flex flex-col flex-grow"><h3 class="text-xl font-bold text-gray-800 dark:text-gray-200">${item.name}</h3><p class="text-md font-semibold text-green-600 dark:text-green-400 mt-2 flex-grow">${priceDisplay}</p><div class="mt-4"><button class="add-to-order-btn bg-gray-800 dark:bg-gray-700 hover:bg-yellow-500 hover:text-black text-white dark:text-gray-200 dark:hover:text-black font-bold px-6 py-3 rounded-lg transition-colors duration-200 ease-in-out w-full" data-id="${item.id}">${buttonText}</button></div></div></div>`;
                });
                html += '</div>';
            });
            el('menu-content').innerHTML = html;
        }

        function renderOrder() {
            const orderItemsList = el('order-items-list');
            if (order.length === 0) {
                orderItemsList.innerHTML = '<p id="empty-order-message" class="text-gray-500 dark:text-gray-400 text-center py-10">Your cart is empty.</p>';
                el('totals-section').style.display = 'none';
                el('clear-cart-btn').classList.add('hidden');
            } else {
                orderItemsList.innerHTML = '';
                order.forEach(cartItem => {
                    const itemData = findMenuItem(cartItem.itemId);
                    if (itemData) {
                        let optionsHtml = '';
                        if(cartItem.options) {
                            optionsHtml = `<div class="text-xs text-gray-500 dark:text-gray-400 mt-1">
                                                ${Object.values(cartItem.options).map(opt => opt.name).join(', ')}
                                                </div>`;
                        }
                        const itemHtml = `
                            <div class="flex items-start space-x-3" data-id="${cartItem.cartItemId}">
                                <img src="${itemData.img || 'https://placehold.co/100x100/EEE/333?text=N/A'}" alt="${itemData.name}" class="w-16 h-16 object-cover rounded-md">
                                <div class="flex-grow">
                                    <p class="font-bold">${itemData.name}</p>
                                    ${optionsHtml}
                                    <p class="text-sm font-semibold text-green-500 dark:text-green-400">$${cartItem.unitPrice.toFixed(2)} ea.</p>
                                </div>
                                <div class="flex items-center space-x-2">
                                    <button class="update-quantity-btn text-lg font-bold w-7 h-7 bg-gray-200 dark:bg-gray-700 rounded-full" data-action="decrease">-</button>
                                    <span class="font-bold w-6 text-center">${cartItem.quantity}</span>
                                    <button class="update-quantity-btn text-lg font-bold w-7 h-7 bg-gray-200 dark:bg-gray-700 rounded-full" data-action="increase">+</button>
                                </div>
                                <button class="remove-item-btn text-red-500 hover:text-red-700 pt-1">
                                    <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
                                </button>
                            </div>`;
                        orderItemsList.insertAdjacentHTML('beforeend', itemHtml);
                    }
                });
                el('totals-section').style.display = 'block';
                el('clear-cart-btn').classList.remove('hidden');
            }
            calculateTotals();
            localStorage.setItem('tubeCoffeeOrder', JSON.stringify(order));
        }

        function calculateTotals() {
            let totalUSD = 0;
            let totalItems = 0;
            order.forEach(cartItem => {
                totalUSD += cartItem.unitPrice * cartItem.quantity;
                totalItems += cartItem.quantity;
            });
            const totalKHR = totalUSD * KHR_CONVERSION_RATE;

            el('total-usd').textContent = `$${totalUSD.toFixed(2)}`;
            el('total-khr').textContent = `${totalKHR.toLocaleString('en-US')}៛`;
            el('place-order-btn').disabled = totalItems === 0;
            el('cart-item-count-badge').textContent = `+${totalItems}`;
            el('cart-item-count-badge').classList.toggle('hidden', totalItems === 0);
        }

        // --- EVENT LISTENERS ---
        function setupEventListeners() {
            const darkModeToggle = el('dark-mode-toggle-checkbox');
            darkModeToggle.checked = document.documentElement.classList.contains('dark');
            darkModeToggle.addEventListener('change', () => {
                document.documentElement.classList.toggle('dark', darkModeToggle.checked);
                localStorage.setItem('theme', darkModeToggle.checked ? 'dark' : 'light');
            });

            el('menu-content').addEventListener('click', handleMenuClick);
            el('order-items-list').addEventListener('click', handleOrderListClick);
            el('clear-cart-btn').addEventListener('click', handleClearCart);
            el('place-order-btn').addEventListener('click', handlePlaceOrder);
            el('floating-cart-btn').addEventListener('click', () => { 
                el('order-sidebar-wrapper').classList.remove('hidden'); 
                el('order-sidebar-wrapper').classList.add('fixed', 'inset-0', 'bg-black/50', 'z-40'); 
            });
            el('order-close-btn').addEventListener('click', () => { 
                el('order-sidebar-wrapper').classList.add('hidden'); 
                el('order-sidebar-wrapper').classList.remove('fixed', 'inset-0', 'bg-black/50', 'z-40'); 
            });
            
            el('staff-login-btn').addEventListener('click', () => el('auth-container').classList.remove('hidden'));
            el('back-to-guest-order-btn').addEventListener('click', () => el('auth-container').classList.add('hidden'));
            el('login-form').addEventListener('submit', handleLogin);
            el('logout-btn').addEventListener('click', handleLogout);

            el('alert-ok-btn').addEventListener('click', () => closeAlert());
            el('alert-confirm-btn').addEventListener('click', () => { 
                closeAlert(); 
                if (confirmCallback) confirmCallback(); 
                confirmCallback = null; 
            });
            el('alert-cancel-btn').addEventListener('click', () => closeAlert());

            el('close-receipt-btn').addEventListener('click', () => { el('receipt-modal').classList.add('hidden'); el('receipt-modal').classList.remove('flex'); });
            el('download-receipt-btn').addEventListener('click', downloadReceipt);

            el('options-modal').addEventListener('click', handleOptionsModalClick);

            el('dashboard-btn').addEventListener('click', showAdminDashboard);
            el('back-to-app-btn').addEventListener('click', updateUIVisibility);
            el('admin-tabs').addEventListener('click', handleAdminTabClick);
            el('admin-management-content').addEventListener('click', handleAdminManagementAction);
            el('menu-management-list').addEventListener('click', handleMenuManagementAction);
            el('add-new-item-btn').addEventListener('click', () => showItemEditor());
            el('close-item-editor-btn').addEventListener('click', () => { el('item-editor-modal').classList.add('hidden'); el('item-editor-modal').classList.remove('flex'); });
            el('item-editor-form').addEventListener('submit', handleSaveItem);
            el('save-and-add-new-btn').addEventListener('click', handleSaveAndAddNew);
            
            el('open-tracker-btn').addEventListener('click', () => {
                el('tracker-modal').classList.add('flex');
                el('tracker-modal').classList.remove('hidden');
                el('order-id-input').value = lastPlacedOrderId;
                // Trigger map resize if map is visible/initialized when modal opens
                if (mapInitialized) {
                    trackOrder(); 
                    setTimeout(() => google.maps.event.trigger(map, 'resize'), 100);
                }
            });
            el('close-tracker-modal-btn').addEventListener('click', () => {
                el('tracker-modal').classList.add('hidden');
                el('tracker-modal').classList.remove('flex');
            });
            el('track-order-btn').addEventListener('click', trackOrder);
            el('order-id-input').addEventListener('keypress', (e) => {
                if (e.key === 'Enter') trackOrder();
            });
        }

        // --- HANDLER FUNCTIONS ---
        function handleMenuClick(e) {
            const btn = e.target.closest('.add-to-order-btn');
            if (!btn) return;
            const id = btn.dataset.id;
            const item = findMenuItem(id);
            if (item.options) { showOptionsMenu(item); } 
            else {
                const existingCartItem = order.find(ci => ci.itemId === id && !ci.options);
                if (existingCartItem) { existingCartItem.quantity++; } 
                else { order.push({ cartItemId: `cart-${Date.now()}`, itemId: id, quantity: 1, unitPrice: item.price }); }
                renderOrder();
            }
        }

        function handleOrderListClick(e) {
            const target = e.target.closest('button');
            if (!target) return;
            const cartItemId = target.closest('[data-id]').dataset.id;
            const cartItemIndex = order.findIndex(item => item.cartItemId === cartItemId);
            if (cartItemIndex === -1) return;

            if (target.classList.contains('update-quantity-btn')) {
                const action = target.dataset.action;
                if (action === 'increase') { order[cartItemIndex].quantity++; } 
                else if (action === 'decrease' && order[cartItemIndex].quantity > 1) { order[cartItemIndex].quantity--; }
                else if (action === 'decrease' && order[cartItemIndex].quantity <= 1) { order.splice(cartItemIndex, 1); }
            } else if (target.classList.contains('remove-item-btn')) {
                order.splice(cartItemIndex, 1);
            }
            renderOrder();
        }

        function handleClearCart() { showConfirm('Are you sure you want to clear the entire cart?', () => { order = []; renderOrder(); }); }

        async function handlePlaceOrder() {
            const orderItems = order.map(cartItem => {
                const itemData = findMenuItem(cartItem.itemId);
                let optionsText = 'N/A';
                if (cartItem.options) { optionsText = Object.values(cartItem.options).map(opt => opt.name).join(', '); }
                return { name: itemData.name, quantity: cartItem.quantity, price: cartItem.unitPrice, options: optionsText };
            });

            const orderPayload = { user: currentUser, items: orderItems, timestamp: new Date().toISOString() };
            const btn = el('place-order-btn');

            try {
                const response = await googleScriptRun('placeOrder', [orderPayload], btn);
                if (response.status === 'success') {
                    lastPlacedOrderId = response.orderId;
                    localStorage.setItem('lastPlacedOrderId', lastPlacedOrderId);
                    showReceipt(response.orderId, response.orderData);
                    order = [];
                    renderOrder();
                } else { showAlert('There was an error placing your order.'); }
            } catch (e) {
                // Already handled by googleScriptRun
            }
        }

        async function handleLogin(e) {
            e.preventDefault();
            const btn = el('login-btn');
            const credentials = { email: el('login-email').value, password: el('login-password').value };
            try {
                const response = await googleScriptRun('validateUser', [credentials], btn);
                if (response.status === 'success') {
                    currentUser = { name: response.name, email: response.email, role: response.role };
                    sessionStorage.setItem('currentUser', JSON.stringify(currentUser));
                    el('auth-container').classList.add('hidden');
                    updateUIVisibility();
                    showAlert(`Welcome, ${currentUser.name}!`);
                } else { showAlert(response.message || 'Invalid credentials.'); }
            } catch (e) {
                // Already handled by googleScriptRun
            }
        }

        function handleLogout() {
            sessionStorage.removeItem('currentUser');
            currentUser = null;
            updateUIVisibility();
            showAlert('You have been logged out.');
        }

        function handleOptionsModalClick(e) {
            const btn = e.target.closest('button');
            if (!btn) return;

            if (btn.id === 'close-options-modal-btn') { el('options-modal').classList.add('hidden'); el('options-modal').classList.remove('flex'); }
            if (btn.classList.contains('option-btn')) {
                const groupName = btn.closest('.option-group').dataset.groupName;
                currentModalSelections[groupName] = { name: btn.dataset.name, price: parseFloat(btn.dataset.price), isBasePrice: btn.dataset.isBasePrice === 'true' };
                highlightSelectedOptions();
                updateOptionsModalPrice();
            }
            if(btn.id === 'options-quantity-increase') { let qty = parseInt(el('options-quantity').textContent); el('options-quantity').textContent = qty + 1; updateOptionsModalPrice(); }
            if(btn.id === 'options-quantity-decrease') { let qty = parseInt(el('options-quantity').textContent); if (qty > 1) { el('options-quantity').textContent = qty - 1; updateOptionsModalPrice(); } }
            if(btn.id === 'add-configured-item-btn') {
                const cartItem = {
                    cartItemId: `cart-${Date.now()}`, itemId: currentModalItem.id,
                    quantity: parseInt(el('options-quantity').textContent),
                    options: JSON.parse(JSON.stringify(currentModalSelections)),
                    unitPrice: calculateUnitPriceWithOptions()
                };
                order.push(cartItem);
                renderOrder();
                el('options-modal').classList.add('hidden');
                el('options-modal').classList.remove('flex');
            }
        }

        // --- UTILITY & HELPER FUNCTIONS ---
        function findMenuItem(id) { for(const cat in menuData) { const item = menuData[cat].find(i=>i.id===id); if(item) return item;} return null; }
        
        function setLoadingState(button, isLoading, originalText = 'Submit') {
            const loaderSize = button.id === 'track-order-btn' ? '!w-5 !h-5 !border-2 !border-t-black' : '!w-5 !h-5 !border-2 !border-t-black';
            button.disabled = isLoading;
            button.innerHTML = isLoading ? `<span class="loader ${loaderSize}"></span>` : `<span class="btn-text">${originalText}</span>`;
        }

        function showAlert(message, onOk) {
            el('alert-message').textContent = message;
            el('alert-confirm-btn').classList.add('hidden');
            el('alert-cancel-btn').classList.add('hidden');
            el('alert-ok-btn').classList.remove('hidden');
            el('alert-modal').classList.add('flex');
            el('alert-modal').classList.remove('hidden');
            if(onOk) { el('alert-ok-btn').onclick = () => { closeAlert(); onOk(); }; }
        }

        function showConfirm(message, onConfirm) {
            confirmCallback = onConfirm;
            el('alert-message').textContent = message;
            el('alert-ok-btn').classList.add('hidden');
            el('alert-confirm-btn').classList.remove('hidden');
            el('alert-cancel-btn').classList.remove('hidden');
            el('alert-modal').classList.add('flex');
            el('alert-modal').classList.remove('hidden');
        }

        function closeAlert() {
            el('alert-modal').classList.remove('flex');
            el('alert-modal').classList.add('hidden');
            el('alert-ok-btn').onclick = () => closeAlert();
        }

        // --- RECEIPT FUNCTIONS ---
        function showReceipt(orderId, orderData) {
            el('receipt-header-info').innerHTML = `<p><strong>Order ID:</strong> ${orderId}</p><p><strong>Date:</strong> ${new Date().toLocaleString()}</p><p><strong>Cashier:</strong> ${currentUser.name}</p>`;
            let itemsHtml = '', total = 0;
            for(const item of orderData.items) {
                itemsHtml += `<div class="flex justify-between items-start text-sm"><div class="flex-grow pr-2"><p>${item.quantity}x ${item.name}</p>${item.options !== 'N/A' ? `<p class="text-xs text-gray-400">${item.options}</p>` : ''}</div><span class="font-semibold whitespace-nowrap">$${(item.quantity * item.price).toFixed(2)}</span></div>`;
                total += item.quantity * item.price;
            }
            el('receipt-items-list').innerHTML = itemsHtml;
            el('receipt-totals').innerHTML = `<div class="flex justify-between font-bold text-lg"><span>Total (USD):</span><span>$${total.toFixed(2)}</span></div><div class="flex justify-between font-bold text-lg"><span>Total (KHR):</span><span>${(total * KHR_CONVERSION_RATE).toLocaleString('en-US')}៛</span></div>`;
            
            el('qr-preview').classList.add('hidden');
            el('qr-placeholder-text').classList.remove('hidden');
            el('qr-input').value = '';

            el('receipt-modal').classList.remove('hidden');
            el('receipt-modal').classList.add('flex');
        }

        function downloadReceipt() {
            const { jsPDF } = window.jspdf;
            html2canvas(el('receipt-content'), { scale: 2 }).then(canvas => {
                const imgData = canvas.toDataURL('image/png');
                const pdf = new jsPDF({ orientation: 'portrait', unit: 'pt', format: [canvas.width, canvas.height] });
                pdf.addImage(imgData, 'PNG', 0, 0, canvas.width, canvas.height);
                pdf.save(`tube-coffee-receipt-${Date.now()}.pdf`);
            });
        }
        
        // --- ITEM OPTIONS MODAL FUNCTIONS ---
        function showOptionsMenu(item) {
            currentModalItem = item;
            currentModalSelections = {};
            el('options-modal-title').textContent = item.name;
            el('options-modal-img').src = item.img;
            const contentDiv = el('options-modal-content');
            contentDiv.innerHTML = '';

            for (const groupName in item.options) {
                let groupHtml = `<div class="option-group" data-group-name="${groupName}"><h3 class="text-lg font-bold mb-2">${groupName}</h3><div class="flex flex-wrap gap-2">`;
                const options = item.options[groupName];
                options.forEach((option, index) => {
                    const isAddOn = typeof option === 'object';
                    const name = isAddOn ? option.name : option;
                    const price = isAddOn ? option.price : 0;
                    const isBasePrice = isAddOn ? (option.isBasePrice || false) : false;
                    
                    // Set default selection (the first option)
                    if (index === 0) { currentModalSelections[groupName] = { name, price, isBasePrice }; }
                    
                    let priceText = (isBasePrice ? `($${price.toFixed(2)})` : (price > 0 ? `<span class="text-xs font-semibold">+$${price.toFixed(2)}</span>` : ''));
                    groupHtml += `<button class="option-btn border-2 rounded-lg px-4 py-2 transition" data-name="${name}" data-price="${price}" data-is-base-price="${isBasePrice}">${name} ${priceText}</button>`;
                });
                groupHtml += `</div></div>`;
                contentDiv.innerHTML += groupHtml;
            }
            el('options-quantity').textContent = '1';
            updateOptionsModalPrice();
            el('options-modal').classList.add('flex');
            el('options-modal').classList.remove('hidden');
            highlightSelectedOptions();
        }

        function highlightSelectedOptions() {
            document.querySelectorAll('#options-modal .option-group').forEach(group => {
                const groupName = group.dataset.groupName;
                const selected = currentModalSelections[groupName];
                if(selected) { group.querySelectorAll('.option-btn').forEach(btn => { btn.classList.toggle('selected', btn.dataset.name === selected.name); }); }
            });
        }

        function calculateUnitPriceWithOptions() {
            if (!currentModalItem) return 0;
            let basePrice = currentModalItem.price;
            let addonsPrice = 0;
            
            // Check for option that overrides base price (like "Size")
            for (const group in currentModalSelections) { 
                if (currentModalSelections[group].isBasePrice) { basePrice = currentModalSelections[group].price; break; }
            }
            
            // Calculate add-on price for non-base price options
            for (const group in currentModalSelections) { 
                if (!currentModalSelections[group].isBasePrice) { addonsPrice += currentModalSelections[group].price || 0; }
            }
            return basePrice + addonsPrice;
        }

        function updateOptionsModalPrice() {
            if (!currentModalItem) return;
            const quantity = parseInt(el('options-quantity').textContent);
            const unitPrice = calculateUnitPriceWithOptions();
            el('options-modal-base-price').textContent = `Unit Price: $${unitPrice.toFixed(2)}`;
            el('options-modal-total-price').textContent = `$${(unitPrice * quantity).toFixed(2)}`;
        }

        // --- ADMIN DASHBOARD FUNCTIONS ---
        function showAdminDashboard() {
            if (currentUser && currentUser.role === 'admin') {
                el('app-container').classList.add('hidden');
                el('admin-dashboard-container').classList.remove('hidden');
                // Auto-select and load the first tab
                el('admin-tabs').querySelector('.admin-tab[data-tab="dashboard-content"]').click();
            } else {
                 showAlert("Access Denied. Only administrators can view the dashboard.");
            }
        }

        function handleAdminTabClick(e) {
            const btn = e.target.closest('.admin-tab');
            if (!btn) return;
            const tabName = btn.dataset.tab;

            el('admin-tabs').querySelectorAll('.admin-tab').forEach(t => t.classList.remove('active'));
            btn.classList.add('active');

            el('admin-tab-content').querySelectorAll('div[id$="-panel"]').forEach(p => p.classList.add('hidden'));
            el(`${tabName}-panel`).classList.remove('hidden');
            
            if (tabName === 'dashboard-content') loadDashboardStats();
            if (tabName === 'menu-management') loadMenuForManagement();
            if (tabName === 'user-management') loadAdminUsers();
        }

        async function loadDashboardStats() {
            const statsContainer = el('dashboard-stats');
            statsContainer.innerHTML = '<div class="col-span-full flex justify-center items-center h-48"><div class="loader mx-auto"></div></div>';

            try {
                const coreStats = await googleScriptRun('getDashboardStats');
                // Using a separate call for salary data in mock to demonstrate functionality
                const salaryRes = await googleScriptRun('calculateSalaries');

                let salaryHtml = '';
                if (salaryRes.status === 'success') {
                    salaryHtml = `<h3 class="col-span-full text-2xl font-bold pt-4 pb-2 border-b dark:border-gray-700">Staff Salary Overview for ${salaryRes.month}</h3>
                                    <div class="col-span-full grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">`;
                    salaryRes.data.forEach(staff => {
                        salaryHtml += `<div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg border-l-4 border-yellow-500">
                                            <p class="text-sm font-medium text-gray-500 dark:text-gray-400">${staff.name} - Daily</p>
                                            <p class="text-3xl font-bold text-green-600 dark:text-green-400">$${staff.dailySalary.toFixed(2)}</p>
                                            <p class="text-xs text-gray-500 dark:text-gray-400 mt-2">Est. Monthly: <span class="font-semibold">$${staff.monthlySalary.toFixed(2)}</span></p>
                                           </div>`;
                    });
                    salaryHtml += `</div>`;
                } else {
                    salaryHtml = `<h3 class="col-span-full text-2xl font-bold pt-4 pb-2 border-b dark:border-gray-700">Salary Overview</h3><p class="text-red-500 col-span-full">${salaryRes.error}</p>`;
                }

                statsContainer.innerHTML = `
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
                        <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Total Revenue</p>
                        <p class="text-3xl font-bold text-yellow-500">$${(coreStats.totalRevenue || 0).toLocaleString('en-US', {minimumFractionDigits: 2, maximumFractionDigits: 2})}</p>
                    </div>
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
                        <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Total Orders</p>
                        <p class="text-3xl font-bold">${(coreStats.totalOrders || 0).toLocaleString()}</p>
                    </div>
                    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg">
                        <p class="text-sm font-medium text-gray-500 dark:text-gray-400">Top Seller</p>
                        <p class="text-3xl font-bold">${coreStats.topSeller.name} <span class="text-lg font-normal">(${(coreStats.topSeller.count || 0)})</span></p>
                    </div>
                    <div class="hidden lg:block"></div> 
                    ${salaryHtml}`;
            } catch (e) {
                // Already handled by googleScriptRun
                statsContainer.innerHTML = `<p class="text-red-500 col-span-full">Failed to load dashboard data.</p>`;
            }
        }

        function loadMenuForManagement() {
            const listContainer = el('menu-management-list');
            // Menu data is already cached from the main screen loadMenu(), just render it
            if (Object.keys(menuData).length === 0) {
                 listContainer.innerHTML = '<p class="text-center text-gray-500 py-10">Menu data is not loaded. Please try refreshing the app page.</p>';
                 return;
            }

            let html = '';
            const sortedCategories = Object.keys(menuData).sort();
            sortedCategories.forEach(cat => {
                if (!menuData[cat] || menuData[cat].length === 0) return;
                html += `<div class="p-6 bg-white dark:bg-gray-800 rounded-xl shadow-lg"><h3 class="text-2xl font-bold mb-4 border-b pb-2 dark:border-gray-600">${cat}</h3><div class="space-y-3">`;
                menuData[cat].forEach(item => {
                    html += `
                        <div class="flex items-center justify-between p-3 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700/50">
                            <div class="flex items-center gap-4">
                                <img src="${item.img || 'https://placehold.co/100x100/EEE/333?text=N/A'}" class="w-12 h-12 rounded-md object-cover">
                                <div><p class="font-bold">${item.name}</p><p class="text-sm text-green-500">$${(item.price || 0).toFixed(2)}</p></div>
                            </div>
                            <div class="space-x-2">
                                <button class="admin-menu-btn p-2 rounded bg-blue-500 text-white" data-action="edit" data-id="${item.id}"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path d="M17.414 2.586a2 2 0 00-2.828 0L7 10.172V13h2.828l7.586-7.586a2 2 0 000-2.828z" /><path fill-rule="evenodd" d="M2 6a2 2 0 012-2h4a1 1 0 010 2H4v10h10v-4a1 1 0 112 0v4a2 2 0 01-2 2H4a2 2 0 01-2-2V6z" clip-rule="evenodd" /></svg></button>
                                <button class="admin-menu-btn p-2 rounded bg-red-500 text-white" data-action="delete" data-id="${item.id}"><svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" /></svg></button>
                            </div>
                        </div>`;
                });
                html += `</div></div>`;
            });
            listContainer.innerHTML = html;
        }

        async function loadAdminUsers() {
            el('user-management-table').innerHTML = '<div class="loader mx-auto"></div>';
            el('admin-list').innerHTML = '<div class="loader !w-6 !h-6"></div>';
            
            try {
                const users = await googleScriptRun('getAppUsers');
                renderUserManagementTable(users);
            } catch (e) {
                // Already handled by googleScriptRun
                el('user-management-table').innerHTML = '<p class="text-red-500">Failed to load user data.</p>';
            }

            try {
                const admins = await googleScriptRun('getAdminUsers');
                renderAdminList(admins);
            } catch (e) {
                // Already handled by googleScriptRun
                el('admin-list').innerHTML = '<p class="text-red-500">Failed to load admin data.</p>';
            }
        }

        function renderUserManagementTable(users) {
            let tableHtml = `<table class="w-full text-left"><thead><tr class="border-b dark:border-gray-600"><th class="p-2">Name</th><th class="p-2">Email</th><th class="p-2">Status</th><th class="p-2">Role</th><th class="p-2">Registered</th><th class="p-2">Actions</th></tr></thead><tbody>`;
            (users || []).forEach(user => {
                const registeredDate = user.registered ? new Date(user.registered).toLocaleDateString() : 'N/A';
                tableHtml += `<tr class="border-b dark:border-gray-700 hover:bg-gray-100 dark:hover:bg-gray-700/50"><td class="p-2">${user.name}</td><td class="p-2">${user.email}</td><td class="p-2"><span class="px-2 py-1 text-xs font-semibold rounded-full ${user.status === 'allowed' ? 'bg-green-200 text-green-800' : 'bg-red-200 text-red-800'}">${user.status}</span></td><td class="p-2">${user.role}</td><td class="p-2">${registeredDate}</td><td class="p-2 space-x-1"><button class="admin-user-btn p-1 rounded ${user.status === 'allowed' ? 'bg-yellow-500 text-white' : 'bg-green-500 text-white'}" data-action="toggle-status" data-email="${user.email}" data-new-status="${user.status === 'allowed' ? 'blocked' : 'allowed'}">${user.status === 'allowed' ? 'Block' : 'Unblock'}</button><button class="admin-user-btn p-1 rounded bg-red-500 text-white" data-action="delete" data-email="${user.email}">Delete</button></td></tr>`;
            });
            tableHtml += `</tbody></table>`;
            el('user-management-table').innerHTML = tableHtml;
            // Re-attach event listener to the parent element for delegation
            el('user-management-panel').addEventListener('click', handleUserManagementAction, { once: true });
        }
        
        function renderAdminList(admins) {
            let listHtml = '';
            (admins || []).forEach(email => {
                listHtml += `<li class="flex justify-between items-center p-2 bg-gray-100 dark:bg-gray-700 rounded-md"><span>${email}</span><button class="admin-admin-btn text-red-500 hover:text-red-700" data-action="remove-admin" data-email="${email}">Remove</button></li>`;
            });
            el('admin-list').innerHTML = listHtml || '<p>No admins found.</p>';
        }

        function handleUserManagementAction(e) {
            const btn = e.target.closest('.admin-user-btn');
            if (!btn) return;
            const action = btn.dataset.action;
            const email = btn.dataset.email;
            if (action === 'toggle-status') { 
                googleScriptRun('updateUserStatus', [email, btn.dataset.newStatus], btn)
                    .then(res => { if(res.success) loadAdminUsers(); showAlert(res.message); })
                    .catch(() => {/* Handled by wrapper */});
            } 
            else if (action === 'delete') { 
                showConfirm(`Are you sure you want to delete user ${email}?`, () => { 
                    googleScriptRun('deleteUser', [email], btn)
                        .then(res => { if(res.success) loadAdminUsers(); showAlert(res.message); })
                        .catch(() => {/* Handled by wrapper */});
                }); 
            }
        }

        function handleAdminManagementAction(e) {
            const btn = e.target.closest('button');
            if (!btn) return;
            
            if (btn.id === 'add-admin-btn') {
                const emailInput = el('new-admin-email');
                if (emailInput.value) { 
                    googleScriptRun('addAdmin', [emailInput.value], btn)
                        .then(res => { 
                            showAlert(res.message); 
                            if (res.success) { 
                                loadAdminUsers(); 
                                emailInput.value = ''; 
                            } 
                        })
                        .catch(() => {/* Handled by wrapper */}); 
                }
            } else if (btn.classList.contains('admin-admin-btn') && btn.dataset.action === 'remove-admin') {
                showConfirm(`Are you sure you want to demote ${btn.dataset.email} from admin?`, () => { 
                    googleScriptRun('removeAdmin', [btn.dataset.email], btn)
                        .then(res => { if (res.success) loadAdminUsers(); showAlert(res.message); })
                        .catch(() => {/* Handled by wrapper */});
                });
            }
        }

        function handleMenuManagementAction(e) {
            const btn = e.target.closest('.admin-menu-btn');
            if (!btn) return;
            const action = btn.dataset.action;
            const id = btn.dataset.id;
            const item = findMenuItem(id);

            if (action === 'edit') { showItemEditor(item); }
            else if (action === 'delete') {
                showConfirm(`Are you sure you want to delete "${item.name}"?`, () => {
                    googleScriptRun('deleteMenuItem', [id], btn)
                        .then(res => {
                            showAlert(res.message);
                            if(res.success) {
                                // Update local cache
                                Object.keys(menuData).forEach(cat => { menuData[cat] = menuData[cat].filter(i => i.id !== id); });
                                renderMenu(); // Update POS view
                                loadMenuForManagement(); // Update management view
                            }
                        })
                        .catch(() => {/* Handled by wrapper */});
                });
            }
        }

        function showItemEditor(item = null) {
            const form = el('item-editor-form');
            form.reset();
            if (item) {
                el('item-editor-title').textContent = 'Edit Item';
                el('item-id-input').value = item.id;
                el('item-name').value = item.name;
                el('item-category').value = item.category;
                el('item-price').value = item.price;
                el('item-img').value = item.img || '';
                el('item-options').value = item.options ? JSON.stringify(item.options, null, 2) : '';
            } else {
                el('item-editor-title').textContent = 'Add New Item';
                el('item-id-input').value = `ITEM-${Date.now()}`;
            }
            el('item-editor-modal').classList.remove('hidden');
            el('item-editor-modal').classList.add('flex');
        }

        async function saveItem(onSuccessCallback) {
            const saveBtn = el('save-item-btn');
            
            let optionsJSON = null;
            const optionsString = el('item-options').value.trim();
            if (optionsString) {
                try { optionsJSON = JSON.parse(optionsString); } 
                catch (err) { showAlert('Error: Options field contains invalid JSON.'); return; }
            }
            const itemData = {
                id: el('item-id-input').value, name: el('item-name').value,
                category: el('item-category').value, price: parseFloat(el('item-price').value),
                img: el('item-img').value, options: optionsJSON
            };
            if (!itemData.name || !itemData.category) { showAlert('Item Name and Category are required.'); return; }
            
            try {
                const res = await googleScriptRun('saveMenuItem', [itemData], saveBtn);
                showAlert(res.message);
                if (res.success) {
                    // Update client-side cache immediately
                    Object.keys(menuData).forEach(cat => { menuData[cat] = menuData[cat].filter(i => i.id !== itemData.id); });
                    if (!menuData[itemData.category]) menuData[itemData.category] = [];
                    menuData[itemData.category].push(itemData);
                    renderMenu();
                    loadMenuForManagement();
                    if (onSuccessCallback) onSuccessCallback();
                }
            } catch (e) {
                // Already handled by googleScriptRun
            }
        }
        
        function handleSaveItem(e) {
            e.preventDefault();
            saveItem(() => { el('item-editor-modal').classList.add('hidden'); el('item-editor-modal').classList.remove('flex'); });
        }
        
        function handleSaveAndAddNew() {
            saveItem(() => { showItemEditor(); });
        }
        
        // --- TRACKER FUNCTIONS ---
        function initMap() {
            if (typeof google === 'undefined' || typeof google.maps === 'undefined') {
                console.error("Google Maps API failed to load. This is often due to a missing or invalid API key.");
                mapInitialized = false;
                const mapMsg = el('map-message');
                if (mapMsg) {
                    mapMsg.classList.remove('hidden');
                    mapMsg.textContent = "Live map tracking is unavailable. Please provide a valid API key.";
                }
                return;
            }
            
            map = new google.maps.Map(el('map'), {
                center: { lat: 37.785, lng: -122.405 }, // Default center: San Francisco
                zoom: 12,
                mapId: 'DEMO_MAP_ID' // Required for some styles, can be customized
            });

            marker = new google.maps.Marker({ map: map, title: "Delivery Location", visible: false });
            mapInitialized = true;
        }
        window.initMap = initMap;

        async function trackOrder() {
            const orderId = el('order-id-input').value.trim();
            const trackBtn = el('track-order-btn');
            if (!orderId) { showAlert("Please enter a valid Order ID."); return; }

            el('order-status-panel').classList.add('hidden');
            el('map-message').classList.add('hidden');

            try {
                const response = await googleScriptRun('getDeliveryStatus', [orderId], trackBtn, 1); // Track status usually shouldn't require multiple retries
                if (response.status === 'success') { renderStatus(response.data); } 
                else {
                    showAlert(response.message);
                    el('order-status-panel').classList.add('hidden');
                }
            } catch (e) {
                // Already handled by googleScriptRun
            }
        }
        
        function renderStatus(data) {
            const panel = el('order-status-panel');
            const statusCard = el('status-card');
            const statusKey = data.status.toUpperCase().replace(/\s/g, '_');
            const colors = STATUS_COLORS[statusKey] || { border: 'border-gray-400', background: 'bg-gray-50 dark:bg-gray-700', text: 'text-gray-800 dark:text-gray-300' };

            statusCard.className = `p-4 rounded-lg shadow-md border-l-4 ${colors.border} ${colors.background} ${colors.text}`;
            el('display-status').textContent = data.status.toUpperCase().replace(/_/g, ' ');
            el('display-status').className = `font-extrabold ${colors.text.replace('-800', '-600')}`;
            el('display-details').textContent = data.details;
            el('display-details').className = `text-gray-700 dark:text-gray-300`;
            el('display-item-name').textContent = data.itemName;
            el('display-customer-name').textContent = data.customerName;
            el('display-meta-info').className = `mt-2 text-sm text-gray-500 dark:text-gray-400`;

            panel.classList.remove('hidden');
            
            const canShowMap = mapInitialized && data.lat !== 0 && data.lng !== 0 && statusKey !== 'DELIVERED' && statusKey !== 'CANCELED';
            el('map').classList.toggle('hidden', !canShowMap);
            el('map-message').classList.toggle('hidden', canShowMap);

            if (canShowMap) {
                const position = { lat: data.lat, lng: data.lng };
                marker.setPosition(position);
                marker.setMap(map);
                marker.setVisible(true);
                map.setCenter(position);
                map.setZoom(15);
            } else {
                if (marker) marker.setMap(null);
                if (statusKey === 'DELIVERED') el('map-message').textContent = "Your order has been delivered.";
                else if (statusKey === 'CANCELED') el('map-message').textContent = "Map tracking is unavailable for canceled orders.";
                else if (!mapInitialized) el('map-message').textContent = "Live map is unavailable due to a missing API key.";
                else el('map-message').textContent = "Awaiting location data for this order.";
            }
        }
        
        // --- Standalone functions for inline HTML calls (QR Code) ---
        function handleQrUploadClick() { document.getElementById('qr-input').click(); }
        function previewQr(event) {
            if(event.target.files && event.target.files[0]) {
                const reader = new FileReader();
                reader.onload = e => {
                    const qrPreview = document.getElementById('qr-preview');
                    qrPreview.src = e.target.result;
                    qrPreview.classList.remove('hidden');
                    document.getElementById('qr-placeholder-text').classList.add('hidden');
                };
                reader.readAsDataURL(event.target.files[0]);
            }
        }
        window.handleQrUploadClick = handleQrUploadClick;
        window.previewQr = previewQr;

        initializeApp();
    });
    </script>
</body>
</html>
