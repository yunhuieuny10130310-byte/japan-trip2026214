# japan-trip2026214
<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>櫻桃家日本之旅 2026</title>
    
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap" rel="stylesheet">

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'bg-main': '#f9f7f2',      /* 淺米色背景 */
                        'card-white': '#ffffff',   /* 卡片白 */
                        'primary': '#c0a080',      /* 拿鐵主色 */
                        'accent': '#a67c52',       /* 焦糖強調色 */
                        'text-dark': '#594a42',    /* 深咖文字 */
                        'text-light': '#9c8a7e',   /* 淺咖文字 */
                    },
                    boxShadow: {
                        'card': '0 2px 10px rgba(166, 124, 82, 0.08)',
                        'nav': '0 -4px 20px rgba(0,0,0,0.05)',
                    }
                }
            }
        }
    </script>

    <style>
        body { 
            font-family: 'Noto Sans TC', sans-serif; 
            background-color: #f9f7f2; 
            color: #594a42; 
            -webkit-tap-highlight-color: transparent; 
            /* 預留超大底部空間，絕對不會被擋住 */
            padding-bottom: 120px; 
            overscroll-behavior-y: none;
        }

        /* 隱藏捲軸但可滑動 */
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        /* 固定頂部標題 */
        .header-fixed {
            position: sticky;
            top: 0;
            z-index: 50;
            background-color: #f9f7f2;
            padding-top: max(15px, env(safe-area-inset-top));
            padding-bottom: 10px;
            border-bottom: 1px solid rgba(0,0,0,0.05);
        }

        /* 日期卡片選中狀態 */
        .date-card.active { 
            background-color: #a67c52; 
            color: white; 
            transform: scale(1.02);
            box-shadow: 0 4px 12px rgba(166, 124, 82, 0.3);
            border-color: #a67c52;
        }

        /* 底部導覽列 - 強制置頂層級 */
        .bottom-nav {
            position: fixed;
            bottom: 0;
            left: 0;
            width: 100%;
            background: rgba(255, 255, 255, 0.98);
            backdrop-filter: blur(10px);
            border-top: 1px solid #e5e5e5;
            padding-bottom: max(15px, env(safe-area-inset-bottom));
            padding-top: 12px;
            z-index: 9999; /* 最高層級 */
        }
    </style>
</head>
<body>

<div id="app" class="min-h-screen">
    
    <div class="header-fixed px-5 shadow-sm">
        <h1 class="text-2xl font-bold text-accent tracking-wide">櫻桃家日本之旅 🌸</h1>
        <p class="text-sm text-text-light mt-1">2026.02.14 - 02.21 | 名古屋 & 東京</p>
    </div>

    <div class="px-4 pt-2">
        
        <div v-if="currentTab === 'itinerary'">
            
            <div class="sticky top-[85px] z-40 bg-bg-main/95 pt-2 pb-2 backdrop-blur-sm">
                <div class="flex overflow-x-auto hide-scrollbar gap-3 pb-2">
                    <div v-for="(day, index) in tripDays" :key="index" 
                         @click="selectedDayIndex = index"
                         class="date-card min-w-[70px] bg-white rounded-xl p-3 text-center border border-gray-200 cursor-pointer transition-all duration-200 flex-shrink-0"
                         :class="{ 'active': selectedDayIndex === index }">
                        <div class="text-xs opacity-80">{{ day.date }}</div>
                        <div class="text-lg font-bold">{{ day.week }}</div>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl p-4 mb-4 flex items-center justify-between shadow-card border-l-4 border-blue-300 mt-2">
                <div class="flex items-center gap-4">
                    <div class="text-3xl">{{ getWeatherIcon(currentDayData.weather) }}</div>
                    <div>
                        <div class="font-bold text-gray-700 text-lg">{{ currentDayData.temp }}°C</div>
                        <div class="text-xs text-gray-500">{{ currentDayData.weatherDesc }}</div>
                    </div>
                </div>
                <div class="text-right">
                    <div class="text-sm font-bold text-gray-600">{{ currentDayData.location }}</div>
                    <div class="text-xs text-gray-400">體感 {{ currentDayData.feels_like }}°C</div>
                </div>
            </div>

            <div class="flex flex-col gap-4 pb-24">
                <div v-for="(item, index) in currentDayItems" :key="item.id"
                     class="bg-white rounded-xl shadow-card relative overflow-hidden transition-transform active:scale-[0.99] border border-gray-50">
                    
                    <div v-if="index > 0" class="flex justify-center items-center py-2 text-xs text-gray-400 gap-2 bg-gray-50 border-b border-gray-100">
                        <i :class="getTransportIcon(item.transportType)"></i>
                        <span @click="openGoogleMapRoute(currentDayItems[index-1].name, item.name)" class="underline cursor-pointer decoration-dotted text-accent">
                            {{ item.transportTime || '點擊導航' }}
                        </span>
                    </div>
                    <div v-else-if="item.type !== 'flight'" class="flex justify-center items-center py-2 text-xs text-gray-400 gap-2 bg-gray-50 border-b border-gray-100">
                        <i class="fa-solid fa-hotel"></i>
                        <span class="text-gray-400">飯店出發</span>
                        <i :class="getTransportIcon(item.transportType)"></i>
                        <span @click="openGoogleMapRoute(currentDayData.hotel, item.name)" class="underline cursor-pointer decoration-dotted text-accent">
                            {{ item.transportTime || '點擊導航' }}
                        </span>
                    </div>

                    <div class="p-4" @click="openGoogleMap(item.name)">
                        <div class="flex justify-between items-start mb-2">
                            <span class="px-2 py-0.5 rounded text-xs font-medium text-white shadow-sm" :class="getTypeColor(item.type)">
                                {{ getTypeName(item.type) }}
                            </span>
                            <div class="flex gap-2 items-center">
                                <button class="text-xs bg-gray-100 px-2 py-1 rounded text-accent flex items-center">
                                    <i class="fa-solid fa-location-dot mr-1"></i>導航
                                </button>
                            </div>
                        </div>
                        
                        <h3 class="text-lg font-bold text-gray-800 mb-1 leading-tight">{{ item.name }}</h3>
                        
                        <div class="flex flex-wrap gap-y-1 gap-x-3 text-sm text-gray-500 mb-2">
                            <span v-if="item.time" class="text-accent font-bold"><i class="fa-regular fa-clock mr-1"></i>{{ item.time }}</span>
                            <span v-if="item.ticket"><i class="fa-solid fa-ticket mr-1"></i>{{ item.ticket }}</span>
                        </div>
                        
                        <div v-if="item.note" class="text-sm text-gray-600 bg-bg-main p-3 rounded-lg border border-gray-100 mt-2">
                            {{ item.note }}
                        </div>
                    </div>
                </div>

                <div v-if="currentDayItems.length === 0" class="text-center py-10 text-gray-400 bg-white rounded-xl border border-dashed border-gray-200">
                    <i class="fa-regular fa-calendar-xmark text-4xl mb-2 opacity-30"></i>
                    <p>本日無行程</p>
                    <button @click="showAddModal = true" class="mt-4 text-accent text-sm underline">點擊右下角新增</button>
                </div>
            </div>
            
            <button @click="showAddModal = true" class="fixed bottom-24 right-5 w-14 h-14 bg-accent text-white rounded-full shadow-lg flex items-center justify-center text-2xl z-40 active:bg-primary transition-transform active:scale-90">
                <i class="fa-solid fa-plus"></i>
            </button>
        </div>

        <div v-if="currentTab === 'info'" class="flex flex-col gap-4 pb-24">
            <div class="bg-white rounded-xl shadow-card p-5 border-l-4 border-accent">
                <h3 class="font-bold text-lg mb-3 text-text-dark"><i class="fa-solid fa-calculator mr-2"></i>即時匯率</h3>
                <div class="flex items-center gap-2 mb-2">
                    <div class="relative w-full">
                        <span class="absolute left-3 top-2 text-gray-400 text-sm">JPY</span>
                        <input type="number" v-model="calcJpy" class="w-full p-2 pl-10 border rounded bg-gray-50 focus:outline-none focus:ring-2 focus:ring-accent" placeholder="輸入日幣">
                    </div>
                    <i class="fa-solid fa-arrow-right text-gray-300"></i>
                    <div class="w-full p-2 bg-gray-100 rounded text-gray-700 font-bold text-center">
                        NT$ {{ (calcJpy * exchangeRate).toFixed(0) }}
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-card p-5 border-l-4 border-green-500">
                <h3 class="font-bold text-lg mb-3 text-green-700"><i class="fa-solid fa-bed mr-2"></i>住宿憑證</h3>
                
                <div class="mb-4 border-b pb-4">
                    <h4 class="font-bold text-lg text-gray-800">名古屋 JR 門樓飯店</h4>
                    <p class="text-xs text-gray-400 mb-2">2/14 - 2/18 (4晚)</p>
                    <div class="bg-green-50 p-3 rounded text-sm space-y-2 border border-green-100">
                        <div class="flex justify-between"><span>確認編號:</span> <span class="font-mono font-bold text-green-800">1616326757791746</span></div>
                        <div class="flex justify-between"><span>PIN 碼:</span> <span class="font-mono font-bold text-red-500 bg-white px-2 rounded border border-red-200">7070</span></div>
                    </div>
                    <div class="mt-2 flex gap-2">
                        <button @click="openGoogleMap('Nagoya JR Gate Tower Hotel')" class="flex-1 py-2 bg-gray-100 rounded text-xs text-gray-600 font-bold"><i class="fa-solid fa-location-arrow mr-1"></i>導航</button>
                    </div>
                </div>

                <div>
                    <h4 class="font-bold text-lg text-gray-800">三井花園飯店大手町</h4>
                    <p class="text-xs text-gray-400 mb-2">2/18 - 2/21 (3晚)</p>
                    <div class="bg-green-50 p-3 rounded text-sm space-y-2 border border-green-100">
                        <div class="flex justify-between"><span>確認編號:</span> <span class="font-mono font-bold text-green-800">1616327582739562</span></div>
                        <div class="flex justify-between"><span>PIN 碼:</span> <span class="font-mono font-bold text-red-500 bg-white px-2 rounded border border-red-200">8835</span></div>
                    </div>
                    <div class="mt-2 flex gap-2">
                        <button @click="openGoogleMap('Mitsui Garden Hotel Otemachi')" class="flex-1 py-2 bg-gray-100 rounded text-xs text-gray-600 font-bold"><i class="fa-solid fa-location-arrow mr-1"></i>導航</button>
                    </div>
                </div>
            </div>

            <div class="bg-white rounded-xl shadow-card p-5 border-l-4 border-blue-500">
                <h3 class="font-bold text-lg mb-3 text-blue-700"><i class="fa-solid fa-plane mr-2"></i>航班 & 座位</h3>
                <div class="mb-4">
                    <div class="flex justify-between items-center mb-1">
                        <span class="bg-blue-100 text-blue-800 text-xs px-2 py-1 rounded font-bold">去程 2/14</span>
                        <span class="font-bold text-blue-800 text-lg">CI150</span>
                    </div>
                    <div class="flex justify-between text-sm items-center my-2">
                        <div class="text-center"><span class="block font-bold text-xl">TPE</span><span class="text-xs text-gray-400">17:15</span></div>
                        <i class="fa-solid fa-arrow-right text-gray-300"></i>
                        <div class="text-center"><span class="block font-bold text-xl">NGO</span><span class="text-xs text-gray-400">20:50</span></div>
                    </div>
                    <div class="mt-2 text-sm bg-gray-50 p-2 rounded text-center text-gray-600">座位: 35A, 36A, 36B, 35B</div>
                </div>
                <div class="border-t pt-3">
                    <div class="flex justify-between items-center mb-1">
                        <span class="bg-blue-100 text-blue-800 text-xs px-2 py-1 rounded font-bold">回程 2/21</span>
                        <span class="font-bold text-blue-800 text-lg">CI221</span>
                    </div>
                    <div class="flex justify-between text-sm items-center my-2">
                        <div class="text-center"><span class="block font-bold text-xl">NRT</span><span class="text-xs text-gray-400">14:15</span></div>
                        <i class="fa-solid fa-arrow-right text-gray-300"></i>
                        <div class="text-center"><span class="block font-bold text-xl">TPE</span><span class="text-xs text-gray-400">17:15</span></div>
                    </div>
                </div>
            </div>
        </div>

        <div v-if="currentTab === 'shopping'" class="pb-24">
            <div class="grid grid-cols-2 gap-4">
                <div v-for="(item, index) in shoppingList" :key="index" class="bg-white rounded-xl shadow-card overflow-hidden relative border border-gray-100">
                    <button @click="removeShoppingItem(index)" class="absolute top-2 right-2 bg-black/40 text-white w-7 h-7 rounded-full text-xs z-10 backdrop-blur-md flex items-center justify-center"><i class="fa-solid fa-xmark"></i></button>
                    <div class="h-32 bg-gray-50 flex items-center justify-center overflow-hidden">
                        <img v-if="item.image" :src="item.image" class="w-full h-full object-cover">
                        <i v-else class="fa-regular fa-image text-3xl text-gray-300"></i>
                    </div>
                    <div class="p-3">
                        <div class="font-bold text-gray-800 truncate">{{ item.name }}</div>
                        <div class="text-xs text-gray-400 mt-1">預計購買</div>
                    </div>
                </div>
            </div>
            
            <div v-if="shoppingList.length === 0" class="text-center py-10 text-gray-400">
                <i class="fa-solid fa-bag-shopping text-4xl mb-2 opacity-30"></i>
                <p>清單空空的</p>
                <p class="text-xs">快把想買的東西加進來！</p>
            </div>

             <button @click="showShoppingModal = true" class="fixed bottom-24 right-5 w-14 h-14 bg-accent text-white rounded-full shadow-lg flex items-center justify-center text-2xl z-40 active:bg-primary transition-transform active:scale-90">
                <i class="fa-solid fa-plus"></i>
            </button>
        </div>

        <div v-if="currentTab === 'expense'" class="pb-24">
            <div class="bg-gradient-to-br from-accent to-primary text-white rounded-xl p-6 mb-4 shadow-lg text-center relative overflow-hidden">
                <div class="relative z-10">
                    <div class="text-sm opacity-90 mb-1 font-medium">總花費 (JPY)</div>
                    <div class="text-4xl font-bold tracking-tight">¥{{ totalExpense.toLocaleString() }}</div>
                    <div class="text-xs opacity-90 mt-3 border-t border-white/20 pt-2 inline-block px-4">約台幣 NT$ {{ (totalExpense * exchangeRate).toFixed(0).toLocaleString() }}</div>
                </div>
                <div class="absolute -right-6 -bottom-8 text-9xl opacity-10 rotate-12"><i class="fa-solid fa-coins"></i></div>
            </div>

            <div class="flex flex-col gap-3">
                <div v-for="(exp, index) in expenses" :key="index" class="bg-white p-4 rounded-xl shadow-card flex justify-between items-center relative pl-3 border border-gray-50">
                    <div class="absolute left-0 top-0 bottom-0 w-1.5 rounded-l-xl" :class="getCategoryColor(exp.category)"></div>
                    <button @click="removeExpense(index)" class="absolute top-2 right-2 text-gray-300 hover:text-red-500 p-2"><i class="fa-solid fa-trash-can text-sm"></i></button>
                    
                    <div class="flex items-center gap-3">
                        <div class="w-10 h-10 rounded-full flex items-center justify-center bg-gray-50 text-gray-500 shadow-sm border border-gray-100">
                            <i :class="getCategoryIcon(exp.category)"></i>
                        </div>
                        <div>
                            <div class="font-bold text-gray-800 text-sm">{{ exp.name }}</div>
                            <div class="text-xs text-gray-400 mt-0.5">{{ exp.date }} · {{ exp.method }}</div>
                        </div>
                    </div>
                    <div class="font-bold text-gray-700 pr-6">¥{{ parseInt(exp.amount).toLocaleString() }}</div>
                </div>
                <div v-if="expenses.length === 0" class="text-center py-10 text-gray-400">尚無記帳資料</div>
            </div>
             <button @click="showExpenseModal = true" class="fixed bottom-24 right-5 w-14 h-14 bg-accent text-white rounded-full shadow-lg flex items-center justify-center text-2xl z-40 active:bg-primary transition-transform active:scale-90">
                <i class="fa-solid fa-plus"></i>
            </button>
        </div>

    </div>

    <div class="bottom-nav flex justify-around items-center px-2 shadow-nav">
        <button @click="currentTab = 'itinerary'" class="flex-1 flex flex-col items-center gap-1 p-2 transition-colors duration-200" :class="currentTab === 'itinerary' ? 'text-accent' : 'text-gray-300'">
            <i class="fa-solid fa-map-location-dot text-xl mb-0.5"></i>
            <span class="text-[10px] font-bold">行程</span>
        </button>
        <button @click="currentTab = 'info'" class="flex-1 flex flex-col items-center gap-1 p-2 transition-colors duration-200" :class="currentTab === 'info' ? 'text-accent' : 'text-gray-300'">
            <i class="fa-solid fa-circle-info text-xl mb-0.5"></i>
            <span class="text-[10px] font-bold">資訊</span>
        </button>
        <button @click="currentTab = 'shopping'" class="flex-1 flex flex-col items-center gap-1 p-2 transition-colors duration-200" :class="currentTab === 'shopping' ? 'text-accent' : 'text-gray-300'">
            <i class="fa-solid fa-bag-shopping text-xl mb-0.5"></i>
            <span class="text-[10px] font-bold">購物</span>
        </button>
        <button @click="currentTab = 'expense'" class="flex-1 flex flex-col items-center gap-1 p-2 transition-colors duration-200" :class="currentTab === 'expense' ? 'text-accent' : 'text-gray-300'">
            <i class="fa-solid fa-coins text-xl mb-0.5"></i>
            <span class="text-[10px] font-bold">花費</span>
        </button>
    </div>

    <div v-if="showAddModal" class="fixed inset-0 bg-black/60 z-[999] flex items-center justify-center p-4 backdrop-blur-sm animate-fade-in">
        <div class="bg-white rounded-2xl w-full max-w-sm p-5 shadow-2xl">
            <h3 class="font-bold text-xl mb-4 text-center text-text-dark">新增行程</h3>
            <div class="space-y-3">
                <div class="grid grid-cols-2 gap-2">
                    <select v-model="formItem.type" class="w-full p-3 border border-gray-200 rounded-lg bg-gray-50 text-sm">
                        <option value="spot">📍 景點</option><option value="food">🍴 美食</option><option value="shopping">🛍️ 購物</option>
                        <option value="hotel">🏨 住宿</option><option value="flight">✈️ 航班</option><option value="other">🔹 其他</option>
                    </select>
                    <input v-model="formItem.time" placeholder="時間 (如 10:00)" class="w-full p-3 border border-gray-200 rounded-lg text-sm">
                </div>
                <input v-model="formItem.name" placeholder="行程名稱" class="w-full p-3 border border-gray-200 rounded-lg">
                <textarea v-model="formItem.note" placeholder="備註 (票券號碼、注意事項)" class="w-full p-3 border border-gray-200 rounded-lg h-24 text-sm"></textarea>
                
                <div class="bg-bg-main p-3 rounded-lg border border-gray-100">
                    <label class="text-xs text-gray-500 mb-2 block font-bold">交通方式 (前往此處)</label>
                    <div class="flex gap-2">
                        <select v-model="formItem.transportType" class="p-2 border rounded text-sm bg-white"><option value="walk">步行</option><option value="train">電車/巴士</option><option value="car">計程車/包車</option></select>
                        <input v-model="formItem.transportTime" placeholder="時間 (如 15分)" class="w-full p-2 border rounded text-sm">
                    </div>
                </div>
            </div>
            <div class="flex gap-3 mt-6">
                <button @click="closeModal" class="flex-1 py-3 bg-gray-100 rounded-xl text-gray-600 font-bold">取消</button>
                <button @click="saveItem" class="flex-1 py-3 bg-accent text-white rounded-xl font-bold shadow-lg">儲存</button>
            </div>
        </div>
    </div>
    
    <div v-if="showShoppingModal" class="fixed inset-0 bg-black/60 z-[999] flex items-center justify-center p-4 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-sm p-5 shadow-2xl">
            <h3 class="font-bold text-xl mb-4 text-center">新增購物清單</h3>
            <div class="space-y-3">
                <input v-model="shoppingForm.name" placeholder="商品名稱" class="w-full p-3 border rounded-lg">
                <label class="block w-full p-4 border-2 border-dashed border-gray-300 rounded-lg text-center text-gray-400 cursor-pointer hover:bg-gray-50 transition-colors">
                    <i class="fa-solid fa-camera mr-2"></i>上傳圖片
                    <input type="file" @change="handleImageUpload" class="hidden" accept="image/*">
                </label>
                <img v-if="shoppingForm.image" :src="shoppingForm.image" class="h-24 w-full object-cover rounded-lg border border-gray-200">
            </div>
            <div class="flex gap-3 mt-6">
                <button @click="showShoppingModal = false" class="flex-1 py-3 bg-gray-100 rounded-xl">取消</button>
                <button @click="saveShoppingItem" class="flex-1 py-3 bg-accent text-white rounded-xl font-bold">新增</button>
            </div>
        </div>
    </div>

    <div v-if="showExpenseModal" class="fixed inset-0 bg-black/60 z-[999] flex items-center justify-center p-4 backdrop-blur-sm">
        <div class="bg-white rounded-2xl w-full max-w-sm p-5 shadow-2xl">
            <h3 class="font-bold text-xl mb-4 text-center">新增花費</h3>
            <div class="space-y-3">
                <div class="grid grid-cols-2 gap-2">
                    <select v-model="expenseForm.category" class="w-full p-3 border rounded-lg bg-gray-50 text-sm">
                        <option value="food">🍴 飲食</option><option value="shopping">🛍️ 購物</option><option value="traffic">🚄 交通</option><option value="ticket">🎫 門票</option><option value="other">🔹 其他</option>
                    </select>
                    <select v-model="expenseForm.method" class="w-full p-3 border rounded-lg bg-gray-50 text-sm">
                        <option value="現金">💵 現金</option><option value="信用卡">💳 信用卡</option><option value="Suica">🐧 IC卡</option>
                    </select>
                </div>
                <input v-model="expenseForm.name" placeholder="項目名稱" class="w-full p-3 border rounded-lg">
                <input type="number" v-model="expenseForm.amount" placeholder="金額 (日幣)" class="w-full p-3 border rounded-lg">
            </div>
            <div class="flex gap-3 mt-6">
                <button @click="showExpenseModal = false" class="flex-1 py-3 bg-gray-100 rounded-xl">取消</button>
                <button @click="saveExpense" class="flex-1 py-3 bg-accent text-white rounded-xl font-bold">記帳</button>
            </div>
        </div>
    </div>

</div>

<script>
    const { createApp, ref, computed, onMounted } = Vue;

    createApp({
        setup() {
            const currentTab = ref('itinerary');
            const selectedDayIndex = ref(0);
            
            // 完整行程表資料 (已將原本PDF和討論內容寫入)
            const tripDays = [
                { date: '2/14', week: 'SAT', fullDate: '2026-02-14', weather: 'cloud', temp: 8, feels_like: 5, location: '名古屋', weatherDesc: '多雲微冷', hotel: '名古屋JR門樓飯店' },
                { date: '2/15', week: 'SUN', fullDate: '2026-02-15', weather: 'sun', temp: 10, feels_like: 8, location: '名古屋', weatherDesc: '晴朗舒適', hotel: '名古屋JR門樓飯店' },
                { date: '2/16', week: 'MON', fullDate: '2026-02-16', weather: 'cloud', temp: 6, feels_like: 3, location: '吉卜力', weatherDesc: '陰天保暖', hotel: '名古屋JR門樓飯店' },
                { date: '2/17', week: 'TUE', fullDate: '2026-02-17', weather: 'snow', temp: -3, feels_like: -8, location: '合掌村', weatherDesc: '大雪寒冷', hotel: '名古屋JR門樓飯店' },
                { date: '2/18', week: 'WED', fullDate: '2026-02-18', weather: 'sun', temp: 9, feels_like: 6, location: '東京', weatherDesc: '移動日晴', hotel: '三井花園飯店大手町' },
                { date: '2/19', week: 'THU', fullDate: '2026-02-19', weather: 'sun', temp: 5, feels_like: 2, location: '河口湖', weatherDesc: '晴朗看山', hotel: '三井花園飯店大手町' },
                { date: '2/20', week: 'FRI', fullDate: '2026-02-20', weather: 'sun', temp: 10, feels_like: 8, location: '東京', weatherDesc: '逛街好日', hotel: '三井花園飯店大手町' },
                { date: '2/21', week: 'SAT', fullDate: '2026-02-21', weather: 'cloud', temp: 11, feels_like: 9, location: '東京', weatherDesc: '返程多雲', hotel: '溫暖的家' }
            ];

            const itineraryData = ref({});
            const shoppingList = ref([]);
            const expenses = ref([]);
            
            // 初始化資料：使用新 Key (_final) 強制重置，確保資料正確顯示
            const initData = () => {
                const storageKey = 'itineraryData_final_v1'; // 更改 Key 確保不會讀到舊的壞資料
                
                if(!localStorage.getItem(storageKey)) {
                    // 預設資料寫入
                    itineraryData.value = {
                        '2026-02-14': [
                            { id: 1, type: 'flight', name: 'CI150 抵達名古屋 (20:50)', time: '20:50', note: '入境預留 2.5 小時，準備搭車', transportType: 'walk', transportTime: '入境' },
                            { id: 2, type: 'other', name: '搭乘 μ-SKY 特急 (1號車)', time: '22:07', ticket: '座位: 8A-9B', note: '購票號: 966-401-04，22:35 抵達名古屋站', transportType: 'train', transportTime: '28分' },
                            { id: 3, type: 'hotel', name: 'Check-in JR門樓飯店', time: '22:45', note: '車站直結 15F，Check-in', transportType: 'walk', transportTime: '3分' }
                        ],
                        '2026-02-15': [
                            { id: 4, type: 'spot', name: '名古屋城', time: '10:00', note: '搭地鐵名城線，7號出口有電梯', transportType: 'train', transportTime: '20分' },
                            { id: 5, type: 'shopping', name: '大塚屋 (車道本店)', time: '13:00', note: '媽媽的手作天堂，地鐵車道站', transportType: 'train', transportTime: '15分' },
                            { id: 6, type: 'shopping', name: 'Animate 名古屋', time: '15:00', note: '《戀與深空》聯名最後一天！', transportType: 'train', transportTime: '10分' },
                            { id: 7, type: 'shopping', name: '榮區 (Loft/3COINS)', time: '16:30', note: '百貨內有電梯，方便逛街', transportType: 'walk', transportTime: '5分' }
                        ],
                        '2026-02-16': [
                            { id: 8, type: 'spot', name: '吉卜力公園', time: '10:00', note: '搭 Linimo 至愛·地球博記念公園站', transportType: 'train', transportTime: '50分' },
                            { id: 9, type: 'food', name: '除夕大餐：敘敘苑', time: '19:00', note: '名站 M3 大樓 9F，看夜景', transportType: 'train', transportTime: '回程' }
                        ],
                        '2026-02-17': [
                            { id: 10, type: 'spot', name: '合掌村一日遊', time: '08:30', note: '名鐵巴士中心出發', transportType: 'car', transportTime: '3.5hr' },
                            { id: 11, type: 'food', name: '落人咖啡廳', time: '12:00', note: '喝紅豆湯避寒，有圍爐裏', transportType: 'walk', transportTime: '村內' }
                        ],
                        '2026-02-18': [
                            { id: 12, type: 'other', name: '新幹線 Nozomi', time: '10:30', note: '往東京，選 E 側座位看富士山', transportType: 'train', transportTime: '1.5hr' },
                            { id: 13, type: 'shopping', name: '新宿購物 (Lumine 2)', time: '14:00', note: '甜酷風服飾 + Okadaya 手作', transportType: 'train', transportTime: '15分' },
                            { id: 14, type: 'hotel', name: '三井花園飯店大手町', time: '18:00', note: '大手町站 A2 出口電梯', transportType: 'train', transportTime: '20分' }
                        ],
                        '2026-02-19': [
                            { id: 15, type: 'other', name: '懶人集合法', time: '08:45', note: '請飯店叫計程車直達集合點', transportType: 'walk', transportTime: '0分' },
                            { id: 16, type: 'spot', name: '河口湖一日遊', time: '09:00', note: '集合點：東京站丸之內北口', transportType: 'car', transportTime: '計程車' }
                        ],
                        '2026-02-20': [
                            { id: 17, type: 'spot', name: 'Shibuya SKY', time: '10:00', note: '無死角美景，電梯直達', transportType: 'train', transportTime: '20分' },
                            { id: 18, type: 'shopping', name: '最後補貨 (原宿/銀座)', time: '14:00', note: '原宿 @cosme, 銀座 Yuzawaya', transportType: 'train', transportTime: '15分' }
                        ],
                        '2026-02-21': [
                            { id: 19, type: 'other', name: '成田特急 N\'EX', time: '10:30', note: '前往成田機場', transportType: 'train', transportTime: '1hr' },
                            { id: 20, type: 'flight', name: 'CI221 返台', time: '14:15', note: '帶著滿滿戰利品回家', transportType: 'walk', transportTime: '登機' }
                        ]
                    };
                    saveToLocal(); 
                } else {
                    itineraryData.value = JSON.parse(localStorage.getItem(storageKey));
                    shoppingList.value = JSON.parse(localStorage.getItem('shoppingList_final') || '[]');
                    expenses.value = JSON.parse(localStorage.getItem('expenses_final') || '[]');
                }
            };

            const showAddModal = ref(false);
            const formItem = ref({ type: 'spot', name: '', time: '', note: '', transportType: 'walk', transportTime: '' });
            const showShoppingModal = ref(false);
            const shoppingForm = ref({ name: '', image: null });
            const showExpenseModal = ref(false);
            const expenseForm = ref({ category: 'food', name: '', amount: '', date: '', method: '現金' });
            
            const calcJpy = ref('');
            const exchangeRate = 0.22;

            const currentDayData = computed(() => tripDays[selectedDayIndex.value]);
            const currentDayItems = computed(() => itineraryData.value[currentDayData.value.fullDate] || []);
            const totalExpense = computed(() => expenses.value.reduce((sum, item) => sum + Number(item.amount), 0));

            const saveToLocal = () => {
                localStorage.setItem('itineraryData_final_v1', JSON.stringify(itineraryData.value));
                localStorage.setItem('shoppingList_final', JSON.stringify(shoppingList.value));
                localStorage.setItem('expenses_final', JSON.stringify(expenses.value));
            };

            const saveItem = () => {
                const dateKey = currentDayData.value.fullDate;
                if (!itineraryData.value[dateKey]) itineraryData.value[dateKey] = [];
                itineraryData.value[dateKey].push({ ...formItem.value, id: Date.now() });
                saveToLocal();
                showAddModal.value = false;
                formItem.value = { type: 'spot', name: '', time: '', note: '', transportType: 'walk', transportTime: '' };
            };

            const saveShoppingItem = () => {
                shoppingList.value.push({ ...shoppingForm.value });
                saveToLocal();
                showShoppingModal.value = false;
                shoppingForm.value = { name: '', image: null };
            };
            
            const handleImageUpload = (e) => {
                const file = e.target.files[0];
                if (file) {
                    const reader = new FileReader();
                    reader.onload = (e) => shoppingForm.value.image = e.target.result;
                    reader.readAsDataURL(file);
                }
            };

            const removeShoppingItem = (index) => {
                if(confirm('刪除?')) { shoppingList.value.splice(index, 1); saveToLocal(); }
            };

            const saveExpense = () => {
                expenses.value.push({ ...expenseForm.value, date: new Date().toLocaleDateString() });
                saveToLocal();
                showExpenseModal.value = false;
                expenseForm.value = { category: 'food', name: '', amount: '', date: '', method: '現金' };
            };
            
            const removeExpense = (index) => {
                if(confirm('刪除?')) { expenses.value.splice(index, 1); saveToLocal(); }
            };

            const openGoogleMap = (keyword) => window.open(`https://www.google.com/maps/search/?api=1&query=${encodeURIComponent(keyword)}`, '_blank');
            const openGoogleMapRoute = (origin, dest) => window.open(`https://www.google.com/maps/dir/?api=1&origin=${encodeURIComponent(origin)}&destination=${encodeURIComponent(dest)}&travelmode=transit`, '_blank');
            const closeModal = () => { showAddModal.value = false; };

            const getWeatherIcon = (type) => ({ 'sun': '☀️', 'cloud': '☁️', 'rain': '🌧️', 'snow': '❄️' }[type] || '🌤️');
            const getTypeColor = (type) => ({ 'spot': 'bg-pink-400', 'food': 'bg-orange-400', 'shopping': 'bg-purple-400', 'hotel': 'bg-green-500', 'flight': 'bg-blue-500', 'other': 'bg-gray-400' }[type]);
            const getTypeName = (type) => ({ 'spot': '景點', 'food': '美食', 'shopping': '購物', 'hotel': '住宿', 'flight': '航班', 'other': '其他' }[type]);
            const getTransportIcon = (type) => (type === 'walk' ? 'fa-person-walking' : type === 'car' ? 'fa-car' : 'fa-train-subway');
            const getCategoryIcon = (cat) => ({ 'traffic': 'fa-train', 'food': 'fa-utensils', 'ticket': 'fa-ticket', 'shopping': 'fa-bag-shopping', 'other': 'fa-ellipsis' }[cat] || 'fa-circle');
            const getCategoryColor = (cat) => ({ 'traffic': 'bg-blue-400', 'food': 'bg-orange-400', 'ticket': 'bg-red-400', 'shopping': 'bg-purple-400', 'other': 'bg-gray-400' }[cat]);

            onMounted(() => { initData(); });

            return {
                currentTab, tripDays, selectedDayIndex, currentDayItems, currentDayData,
                showAddModal, formItem, saveItem, closeModal,
                shoppingList, showShoppingModal, shoppingForm, saveShoppingItem, handleImageUpload, removeShoppingItem,
                expenses, showExpenseModal, expenseForm, saveExpense, removeExpense, totalExpense,
                calcJpy, exchangeRate,
                getWeatherIcon, getTypeColor, getTypeName, getTransportIcon, getCategoryIcon, getCategoryColor,
                openGoogleMap, openGoogleMapRoute
            };
        }
    }).mount('#app');
</script>
</body>
</html>
