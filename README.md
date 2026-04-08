<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Tracy's Travel Map</title>
    
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <script src="https://cdn.jsdelivr.net/npm/sortablejs@1.15.0/Sortable.min.js"></script>
    <script src="https://vjs.zencdn.net/7.20.3/video.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'bts-purple': '#A689CA', 
                        'soft-purple': '#F3EFFF',
                        'deep-purple': '#7C59D9'
                    }
                }
            }
        }
    </script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Noto+Sans+TC:wght@400;500;700&display=swap');
        body { font-family: 'Noto Sans TC', sans-serif; background-color: #A689CA; overflow-x: hidden; }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        .glass-header { background: rgba(255, 255, 255, 0.2); backdrop-filter: blur(8px); }
        .tab-active { background: white; color: #7C59D9; border-radius: 999px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .drag-ghost { opacity: 0.5; background: #f3efff; }
    </style>
</head>
<body class="antialiased text-slate-800">

    <div id="app" class="max-w-md mx-auto min-h-screen flex flex-col relative shadow-2xl">
        
        <header class="relative w-full h-[40vh] overflow-hidden">
            <img src="header.png" class="w-full h-full object-cover" alt="BTS Header">
            
            <div class="absolute bottom-16 right-4 left-4 glass-header rounded-full p-1 flex justify-between items-center z-20">
                <button v-for="tab in tabs" :key="tab.id" @click="currentTab = tab.id"
                        :class="['flex-1 py-2 flex flex-col items-center transition-all', currentTab === tab.id ? 'tab-active' : 'text-white']">
                    <i :data-lucide="tab.icon" class="w-5 h-5"></i>
                </button>
            </div>

            <div class="absolute bottom-0 w-full h-12 bg-soft-purple rounded-t-[40px] z-10"></div>
        </header>

        <main class="flex-1 bg-soft-purple px-5 -mt-1 pb-24 z-10 min-h-[60vh]">
            
            <div v-if="currentTab === 'itinerary'">
                <div class="flex gap-4 overflow-x-auto no-scrollbar py-2 mb-6">
                    <div v-for="day in dates" :key="day.date" 
                         @click="selectedDate = day.date"
                         :class="['flex flex-col items-center min-w-[60px] p-3 rounded-2xl transition-all', selectedDate === day.date ? 'bg-bts-purple text-white shadow-lg' : 'bg-white text-slate-400']">
                        <span class="text-xs font-bold">{{ day.weekday }}</span>
                        <span class="text-xl font-bold">{{ day.dayNum }}</span>
                    </div>
                </div>

                <div class="bg-white/60 rounded-2xl p-4 mb-6 flex items-center justify-between border border-white">
                    <div class="flex items-center gap-3">
                        <i data-lucide="cloud-sun" class="text-orange-400 w-8 h-8"></i>
                        <div>
                            <p class="text-lg font-bold">22°C</p>
                            <p class="text-xs text-slate-500">體感 20°C · 首爾</p>
                        </div>
                    </div>
                    <span class="text-xs text-purple-400 font-medium italic">Sunny Day</span>
                </div>

                <div id="itinerary-list" class="space-y-4">
                    <div v-for="(item, index) in currentDayItems" :key="item.id" :data-id="item.id">
                        
                        <div v-if="index > 0" class="flex items-center gap-2 my-2 ml-10 text-slate-400 text-xs italic">
                            <i :data-lucide="item.transportIcon" class="w-3 h-3"></i>
                            <span>{{ item.travelTime }} ({{ item.transportType }})</span>
                        </div>

                        <div class="bg-white rounded-3xl p-5 shadow-sm border border-purple-50 flex items-start gap-4 active:scale-95 transition-transform" @click="openEdit(item)">
                            <div class="bg-soft-purple p-3 rounded-2xl text-bts-purple">
                                <i :data-lucide="getCategoryIcon(item.category)"></i>
                            </div>
                            <div class="flex-1">
                                <h3 class="font-bold text-slate-800 text-lg">{{ item.name }}</h3>
                                <div class="grid grid-cols-2 gap-1 mt-2">
                                    <p v-if="item.time" class="text-xs text-slate-500 flex items-center gap-1"><i data-lucide="clock" class="w-3 h-3"></i> {{ item.time }}</p>
                                    <p v-if="item.price" class="text-xs text-slate-500 flex items-center gap-1"><i data-lucide="ticket" class="w-3 h-3"></i> {{ item.price }}</p>
                                </div>
                                <p class="text-xs text-purple-400 mt-2 line-clamp-1 border-t pt-2 border-slate-50">{{ item.note }}</p>
                            </div>
                            <button @click.stop="openNav(item.address)" class="text-bts-purple p-2 hover:bg-soft-purple rounded-full">
                                <i data-lucide="navigation"></i>
                            </button>
                        </div>
                    </div>
                </div>

                <button @click="showAddModal = true" class="fixed bottom-28 right-6 bg-bts-purple text-white p-5 rounded-full shadow-2xl z-50 animate-bounce">
                    <i data-lucide="plus" class="w-6 h-6"></i>
                </button>
            </div>

            <div v-if="currentTab === 'info'" class="py-4 space-y-6">
                <div class="bg-white rounded-3xl p-6 shadow-sm border border-purple-100">
                    <h2 class="text-lg font-bold mb-4 flex items-center gap-2"><i data-lucide="refresh-cw" class="text-bts-purple"></i> 匯率換算</h2>
                    <div class="flex items-center gap-4">
                        <input v-model="currencyAmount" type="number" class="w-full bg-soft-purple p-3 rounded-xl outline-none" placeholder="輸入日幣 (JPY)">
                        <i data-lucide="arrow-right" class="text-slate-300"></i>
                        <div class="w-full bg-bts-purple text-white p-3 rounded-xl font-bold">
                            {{ (currencyAmount * 0.052).toFixed(2) }} HKD
                        </div>
                    </div>
                </div>
                
                <div class="bg-indigo-900 text-white rounded-3xl p-6 relative overflow-hidden">
                    <div class="flex justify-between items-center mb-6">
                        <span class="text-xs opacity-60">CX 520 · 2026.04.28</span>
                        <i data-lucide="plane-takeoff" class="w-5 h-5"></i>
                    </div>
                    <div class="flex justify-between items-center">
                        <div>
                            <h4 class="text-3xl font-bold">HKG</h4>
                            <p class="text-xs opacity-60">15:30</p>
                        </div>
                        <div class="flex-1 flex flex-col items-center px-4">
                            <span class="text-[10px] mb-1">3h 40m</span>
                            <div class="w-full h-[1px] bg-white/30 relative">
                                <div class="absolute -top-1 left-1/2 -translate-x-1/2 bg-indigo-900 px-1"><i data-lucide="plane" class="w-3 h-3"></i></div>
                            </div>
                        </div>
                        <div class="text-right">
                            <h4 class="text-3xl font-bold">ICN</h4>
                            <p class="text-xs opacity-60">20:10</p>
                        </div>
                    </div>
                </div>
            </div>

            <div v-if="currentTab === 'spend'" class="py-4">
                <div class="bg-bts-purple text-white rounded-3xl p-8 mb-6 text-center shadow-lg">
                    <p class="text-sm opacity-80">總花費估算</p>
                    <h2 class="text-4xl font-bold mt-2">¥ 128,400</h2>
                    <p class="text-xs mt-2">≈ HKD 6,676</p>
                </div>
            </div>

        </main>

        <div v-if="showAddModal" class="fixed inset-0 bg-black/60 z-[100] flex items-end">
            <div class="bg-white w-full rounded-t-[40px] p-8 animate-slide-up">
                <div class="flex justify-between items-center mb-6">
                    <h2 class="text-xl font-bold">新增行程</h2>
                    <button @click="showAddModal = false"><i data-lucide="x"></i></button>
                </div>
                <div class="space-y-4">
                    <input v-model="newItem.name" type="text" placeholder="行程名稱" class="w-full bg-soft-purple p-4 rounded-2xl outline-none">
                    <select v-model="newItem.category" class="w-full bg-soft-purple p-4 rounded-2xl outline-none">
                        <option value="spot">景點</option>
                        <option value="food">美食</option>
                        <option value="hotel">住宿</option>
                    </select>
                    <textarea v-model="newItem.note" placeholder="備註..." class="w-full bg-soft-purple p-4 rounded-2xl h-24 outline-none"></textarea>
                    
                    <div class="flex gap-4 pt-4">
                        <button @click="showAddModal = false" class="flex-1 py-4 text-slate-400 font-bold">取消</button>
                        <button @click="addItem" class="flex-1 py-4 bg-bts-purple text-white rounded-2xl font-bold shadow-lg shadow-purple-200">儲存行程</button>
                    </div>
                </div>
            </div>
        </div>

    </div>

    <script>
        const { createApp, ref, computed, onMounted, nextTick } = Vue;

        createApp({
            setup() {
                const currentTab = ref('itinerary');
                const selectedDate = ref('2026-04-30');
                const currencyAmount = ref(1000);
                const showAddModal = ref(false);
                
                const tabs = [
                    { id: 'itinerary', icon: 'map-pin' },
                    { id: 'info', icon: 'info' },
                    { id: 'shop', icon: 'shopping-basket' },
                    { id: 'spend', icon: 'wallet' }
                ];

                const dates = [
                    { weekday: 'Wed', dayNum: '28', date: '2026-04-28' },
                    { weekday: 'Thu', dayNum: '29', date: '2026-04-29' },
                    { weekday: 'Fri', dayNum: '30', date: '2026-04-30' },
                    { weekday: 'Sat', dayNum: '01', date: '2026-05-01' }
                ];

                const itineraryItems = ref([
                    { id: 1, date: '2026-04-30', name: 'DK 演唱會場地', time: '18:00', category: 'spot', price: '¥15,000', note: '記得帶手燈和應援物', address: 'Seoul Olympic Hall', transportType: '地鐵', transportIcon: 'train', travelTime: '25min' },
                    { id: 2, date: '2026-04-30', name: '聖水洞 咖啡廳', time: '14:00', category: 'food', price: '¥2,000', note: '這家店 DK 去過！', address: 'Seongsu-dong Cafe', transportType: '步行', transportIcon: 'footprints', travelTime: '10min' }
                ]);

                const currentDayItems = computed(() => {
                    return itineraryItems.value.filter(item => item.date === selectedDate.value);
                });

                const newItem = ref({ name: '', category: 'spot', note: '' });

                const addItem = () => {
                    itineraryItems.value.push({
                        id: Date.now(),
                        date: selectedDate.value,
                        ...newItem.value,
                        time: '12:00',
                        transportType: '開車',
                        transportIcon: 'car',
                        travelTime: '15min'
                    });
                    showAddModal.value = false;
                    newItem.value = { name: '', category: 'spot', note: '' };
                    nextTick(() => lucide.createIcons());
                };

                const getCategoryIcon = (cat) => {
                    const map = { spot: 'palmtree', food: 'utensils', hotel: 'hotel', flight: 'plane' };
                    return map[cat] || 'star';
                };

                const openNav = (addr) => {
                    window.open(`http://googleusercontent.com/maps.google.com/7{encodeURIComponent(addr)}`);
                };

                onMounted(() => {
                    lucide.createIcons();
                    // 初始化拖曳排序
                    const el = document.getElementById('itinerary-list');
                    if (el) {
                        Sortable.create(el, {
                            animation: 150,
                            ghostClass: 'drag-ghost',
                            onEnd: (evt) => {
                                // 這裡實作陣列重新排序邏輯
                                console.log('已重新排序');
                            }
                        });
                    }
                });

                return { 
                    currentTab, tabs, dates, selectedDate, 
                    itineraryItems, currentDayItems, getCategoryIcon, 
                    currencyAmount, openNav, showAddModal, newItem, addItem 
                };
            }
        }).mount('#app');
    </script>

    <style>
        @keyframes slide-up {
            from { transform: translateY(100%); }
            to { transform: translateY(0); }
        }
        .animate-slide-up { animation: slide-up 0.3s ease-out; }
    </style>
</body>
</html>
