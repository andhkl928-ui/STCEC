import React, { useState, useEffect, useRef } from 'react';
import { 
  MapPin, Calendar, DollarSign, Cloud, Sun, CloudRain, 
  Camera, Edit2, Trash2, Plus, Send, User, Users, 
  Navigation, Share2, Info, Check, X, Smartphone,
  Cpu, Droplets, Coffee, Music, Search, Bus, Utensils,
  Menu, Mic, Layers, Locate, ArrowRightCircle,
  Mountain, Waves, Palette, Zap, Box, Copy, CheckCircle,
  ExternalLink
} from 'lucide-react';

// --- Firebase Imports ---
import { initializeApp } from "firebase/app";
import { getAuth, signInWithCustomToken, signInAnonymously, onAuthStateChanged } from "firebase/auth";
import { getFirestore, doc, onSnapshot, setDoc, updateDoc, arrayUnion, getDoc } from "firebase/firestore";

// --- Mock Data (Fallback & Initial) ---
const INITIAL_ITINERARY = [
  {
    day: 1,
    date: 'Day 1',
    title: '穿越古今：烏鎮',
    theme: 'water',
    focus: '科技與人工智慧優勢、烏鎮歷史文化',
    events: [
      { id: 101, time: '12:00', location: '烏鎮民國時代', type: 'food', desc: '感受民國風情，午餐體驗。', weather: 'cloudy', temp: 24, learning: '民國歷史與建築特色', x: 200, y: 150 },
      { id: 102, time: '13:30', location: '烏鎮元宇宙體驗館', type: 'tech', desc: '虛擬數位人AI對話，沉浸式體感虛擬實境。', weather: 'cloudy', temp: 25, learning: '電腦網絡、AI應用與國家數據安全', x: 350, y: 220 },
      { id: 103, time: '14:00', location: '烏鎮西栅景區深度遊', type: 'culture', desc: '大劇院、木心美術館、昭明書院、白蓮塔。', weather: 'sunny', temp: 26, learning: '傳統建築美學與現代設計的融合', x: 280, y: 300 },
      { id: 104, time: '16:00', location: '水上集市', type: 'culture', desc: '藍草學堂、皮影戲、花鼓戲體驗。', weather: 'sunny', temp: 25, learning: '非物質文化遺產傳承', x: 150, y: 400 },
      { id: 105, time: '18:00', location: '烏鎮錦岸私房菜', type: 'food', desc: '羊肉、河蝦、白水魚特色晚餐。', weather: 'cloudy', temp: 23, learning: '地方飲食文化', x: 400, y: 350 },
    ],
    journal: []
  },
  {
    day: 2,
    date: 'Day 2',
    title: '工匠精神：烏鎮至杭州',
    theme: 'craft',
    focus: '非遺工藝、智慧城市體驗',
    events: [
      { id: 201, time: '10:00', location: '早茶客', type: 'food', desc: '水上戲台旁品嚐江南特色早茶。', weather: 'rain', temp: 22, learning: '傳統習俗與生活美學', x: 220, y: 180 },
      { id: 202, time: '12:00', location: '宏源泰染坊 / 湖筆工坊', type: 'culture', desc: '分2組各自體驗：藍印花布扎染(前店後坊) 或 湖筆製作工藝(1.5小時)。', weather: 'cloudy', temp: 24, learning: '工匠精神與文化自信', x: 300, y: 250 },
      { id: 205, time: '14:00', location: '乘車前往杭州', type: 'travel', desc: '專車接送，車程約1小時。', weather: 'cloudy', temp: 25, learning: '', x: 500, y: 500 },
      { id: 203, time: '15:30', location: '湖濱路行人徒步區', type: 'tech', desc: '杭州城市心臟，5G全覆蓋，AR掃描虛擬景點。', weather: 'sunny', temp: 27, learning: '智慧城市與生活科技化', x: 180, y: 600 },
      { id: 204, time: '18:00', location: '知味觀湖濱總店', type: 'food', desc: '品嚐西湖醋魚、東坡肉等杭幫菜。', weather: 'cloudy', temp: 25, learning: '', x: 250, y: 620 },
    ],
    journal: []
  },
  {
    day: 3,
    date: 'Day 3',
    title: '數字人文：西湖',
    theme: 'nature',
    focus: '茶文化、AR遊西湖、數字文保',
    events: [
      { id: 301, time: '09:00', location: '龍井村', type: 'culture', desc: '採茶、炒茶、品茶深度體驗。', weather: 'sunny', temp: 22, learning: '勞動價值、茶文化與生態安全', x: 100, y: 150 },
      { id: 305, time: '12:00', location: '杭州酒家 (午餐)', type: 'food', desc: '體驗當地特色菜，包括東坡肉、龍井蝦仁。', weather: 'sunny', temp: 24, learning: '杭幫菜飲食文化', x: 250, y: 300 },
      { id: 302, time: '14:00', location: '西湖岳湖景區 (AR遊)', type: 'tech', desc: '運用「掌上西湖」App體驗虛實融合景點。', weather: 'sunny', temp: 26, learning: 'AR技術在旅遊與文化的應用', x: 350, y: 320 },
      { id: 303, time: '15:30', location: '靈隱飛來峰景區', type: 'culture', desc: '觀賞飛來峰石刻，了解3D掃描文物保存技術。', weather: 'cloudy', temp: 25, learning: '科技賦能文物保護', x: 150, y: 450 },
      { id: 306, time: '18:00', location: '樓外樓 (晚餐)', type: 'food', desc: '百年老店，能隔窗直望西湖夜景。', weather: 'rain', temp: 23, learning: '老字號歷史傳承', x: 380, y: 340 },
      { id: 304, time: '19:40', location: '《最憶是杭州》表演', type: 'tech', desc: '水上全息投影表演，梁祝與高山流水。', weather: 'rain', temp: 20, learning: '科技與藝術的跨界融合', x: 400, y: 360 },
    ],
    journal: []
  },
  {
    day: 4,
    date: 'Day 4',
    title: '未來科技：阿里巴巴',
    theme: 'tech',
    focus: '數字經濟、電競產業',
    events: [
      { id: 401, time: '09:00', location: '阿里巴巴西溪園區', type: 'tech', desc: '參觀訪客中心、未來科技展廳(生成式AI)。', weather: 'sunny', temp: 28, learning: '數字經濟發展與企業創新', x: 450, y: 150 },
      { id: 404, time: '12:00', location: '阿里巴巴員工食堂 (午餐)', type: 'food', desc: '能體驗真實工作環境及企業文化。', weather: 'sunny', temp: 28, learning: '企業文化體驗', x: 470, y: 180 },
      { id: 402, time: '14:00', location: '中國杭州電競中心', type: 'tech', desc: '與電競選手交流，了解科學化訓練。', weather: 'cloudy', temp: 27, learning: '新興產業機遇與職業規劃', x: 250, y: 500 },
      { id: 405, time: '17:30', location: '皇飯兒 (晚餐)', type: 'food', desc: '位於河坊街的百年老字號杭幫菜，特色菜有豆腐魚頭﹑蝦油菠菜等。', weather: 'cloudy', temp: 25, learning: '傳統飲食文化', x: 300, y: 600 },
      { id: 403, time: '19:00', location: '酒店全體分享會', type: 'tech', desc: '試用AI科技作分享(製作影片/360拍攝/動畫配音)，總結所得。', weather: 'cloudy', temp: 24, learning: '資訊素養與總結反思', x: 320, y: 650 },
    ],
    journal: []
  },
  {
    day: 5,
    date: 'Day 5',
    title: '智能生活：回程',
    theme: 'future',
    focus: 'AI新成果、元宇宙生活',
    events: [
      { id: 501, time: '09:00', location: '文三數字生活街區', type: 'tech', desc: '體驗DeepSeek人臉裝置、宇樹機器人。', weather: 'sunny', temp: 26, learning: '人工智能的最新發展', x: 200, y: 200 },
      { id: 502, time: '11:00', location: '滋味淵 (機器人餐廳)', type: 'tech', desc: '體驗全機器人上菜服務。', weather: 'sunny', temp: 27, learning: '科技改變服務業', x: 250, y: 250 },
      { id: 503, time: '13:00', location: '杭州蕭山國際機場', type: 'travel', desc: '參觀絲綢文化建築元素，準備回港。', weather: 'sunny', temp: 28, learning: '', x: 500, y: 700 },
    ],
    journal: []
  }
];

const BUDGET_ITEMS = [
  { item: '機票 (國泰來回)', price: 2400 },
  { item: '南柵客棧 (每人每晚)', price: 219 },
  { item: '桔子水晶酒店 (每人每晚)', price: 225 },
  { item: '旅遊巴分攤 (約)', price: 341 },
  { item: '印象西湖表演', price: 400 },
  { item: '電競體驗費用', price: 300 }, 
  { item: '餐飲總預算 (約)', price: 1000 },
];

// --- Water Town Background Component ---
const WaterTownBackground = () => (
  <div className="absolute inset-0 pointer-events-none overflow-hidden z-0">
    {/* Base Gradient - Deep Ink & Teal */}
    <div className="absolute inset-0 bg-gradient-to-b from-slate-900 via-teal-950 to-slate-900" />
    
    {/* Traditional Pattern - Stylized Waves/Tiles */}
    <svg className="absolute w-full h-full opacity-5" width="100%" height="100%">
      <pattern id="pattern-water" x="0" y="0" width="40" height="40" patternUnits="userSpaceOnUse">
        <path d="M0 20 Q10 10 20 20 T40 20" fill="none" stroke="#2dd4bf" strokeWidth="1"/>
        <path d="M0 30 Q10 20 20 30 T40 30" fill="none" stroke="#2dd4bf" strokeWidth="1" opacity="0.5"/>
      </pattern>
      <rect x="0" y="0" width="100%" height="100%" fill="url(#pattern-water)"></rect>
    </svg>

    {/* Ink Wash Elements (Abstract Mountains) */}
    <svg className="absolute bottom-0 w-full h-96 opacity-30" viewBox="0 0 100 100" preserveAspectRatio="none">
       <path d="M0 100 L30 60 Q50 40 70 60 L100 100 Z" fill="url(#ink-grad-1)" />
       <path d="M40 100 L60 50 Q75 30 90 50 L120 100 Z" fill="url(#ink-grad-2)" />
       <defs>
         <linearGradient id="ink-grad-1" x1="0%" y1="0%" x2="0%" y2="100%">
           <stop offset="0%" stopColor="#0f766e" stopOpacity="0.4" />
           <stop offset="100%" stopColor="#0f766e" stopOpacity="0" />
         </linearGradient>
         <linearGradient id="ink-grad-2" x1="0%" y1="0%" x2="0%" y2="100%">
           <stop offset="0%" stopColor="#115e59" stopOpacity="0.6" />
           <stop offset="100%" stopColor="#115e59" stopOpacity="0" />
         </linearGradient>
       </defs>
    </svg>
    
    {/* Floating Particles (Fireflies/Lanterns) */}
    <div className="absolute top-20 left-10 w-1 h-1 bg-teal-200 rounded-full blur-[1px] animate-pulse"></div>
    <div className="absolute top-40 right-20 w-1.5 h-1.5 bg-cyan-300 rounded-full blur-[2px] animate-pulse delay-700"></div>
    <div className="absolute bottom-60 left-1/3 w-1 h-1 bg-emerald-200 rounded-full blur-[1px] animate-pulse delay-300"></div>
  </div>
);

// --- Theme Art Component (Header) ---
const ThemeHeader = ({ theme }) => {
  const gradients = {
    water: "from-cyan-900/80 to-slate-900/80",
    craft: "from-indigo-900/80 to-slate-900/80",
    nature: "from-emerald-900/80 to-slate-900/80",
    tech: "from-blue-900/80 to-slate-900/80",
    future: "from-violet-900/80 to-slate-900/80",
  };

  return (
    <div className={`relative h-40 w-full overflow-hidden rounded-2xl bg-gradient-to-br ${gradients[theme] || gradients.water} mb-6 border border-teal-500/20 shadow-lg group backdrop-blur-sm`}>
      {/* Animated Abstract Backgrounds */}
      <div className="absolute inset-0 opacity-40">
        {theme === 'water' && (
           <svg className="w-full h-full" viewBox="0 0 100 100" preserveAspectRatio="none">
             <path d="M0,50 Q25,30 50,50 T100,50 L100,100 L0,100 Z" fill="url(#grad1)" className="animate-pulse" style={{animationDuration: '3s'}}/>
             <path d="M0,70 Q25,50 50,70 T100,70 L100,100 L0,100 Z" fill="url(#grad2)" className="animate-pulse" style={{animationDuration: '4s'}}/>
             <defs>
               <linearGradient id="grad1" x1="0%" y1="0%" x2="100%" y2="0%"><stop offset="0%" stopColor="#22d3ee" stopOpacity="0.4"/><stop offset="100%" stopColor="#0891b2" stopOpacity="0.2"/></linearGradient>
               <linearGradient id="grad2" x1="0%" y1="0%" x2="100%" y2="0%"><stop offset="0%" stopColor="#22d3ee" stopOpacity="0.6"/><stop offset="100%" stopColor="#0891b2" stopOpacity="0.3"/></linearGradient>
             </defs>
           </svg>
        )}
        {theme === 'craft' && (
           <svg className="w-full h-full" width="100" height="100" viewBox="0 0 100 100">
             <pattern id="pattern-circles" x="0" y="0" width="20" height="20" patternUnits="userSpaceOnUse">
               <circle cx="10" cy="10" r="2" fill="#818cf8" fillOpacity="0.5"/>
               <path d="M0,10 L20,10 M10,0 L10,20" stroke="#818cf8" strokeWidth="0.5" strokeOpacity="0.3"/>
             </pattern>
             <rect width="100%" height="100%" fill="url(#pattern-circles)" />
             <path d="M10,10 Q50,90 90,10" stroke="#a5b4fc" strokeWidth="1" fill="none" className="opacity-50" />
           </svg>
        )}
        {theme === 'nature' && (
           <svg className="w-full h-full" viewBox="0 0 100 100" preserveAspectRatio="none">
              <path d="M0,100 L20,60 L40,100 Z" fill="#34d399" fillOpacity="0.2"/>
              <path d="M30,100 L50,40 L70,100 Z" fill="#10b981" fillOpacity="0.2"/>
              <path d="M60,100 L80,50 L100,100 Z" fill="#059669" fillOpacity="0.2"/>
              <circle cx="80" cy="20" r="10" fill="#fcd34d" fillOpacity="0.4" className="animate-pulse"/>
           </svg>
        )}
        {theme === 'tech' && (
           <svg className="w-full h-full">
              <defs>
                 <pattern id="grid" width="20" height="20" patternUnits="userSpaceOnUse">
                    <path d="M 20 0 L 0 0 0 20" fill="none" stroke="#3b82f6" strokeWidth="0.5" strokeOpacity="0.3"/>
                 </pattern>
              </defs>
              <rect width="100%" height="100%" fill="url(#grid)" />
              <circle cx="50%" cy="50%" r="30" stroke="#60a5fa" strokeWidth="1" fill="none" strokeDasharray="4 2" className="animate-spin-slow" style={{transformOrigin: 'center'}}/>
           </svg>
        )}
        {theme === 'future' && (
           <svg className="w-full h-full">
              <rect x="10" y="10" width="20" height="20" fill="none" stroke="#a78bfa" strokeWidth="2" />
              <rect x="40" y="40" width="30" height="30" fill="none" stroke="#c084fc" strokeWidth="2" className="animate-bounce" style={{animationDuration: '3s'}}/>
              <line x1="0" y1="100" x2="100" y2="0" stroke="#d8b4fe" strokeWidth="1" strokeDasharray="5 5" />
           </svg>
        )}
      </div>

      {/* Foreground Icon & Label */}
      <div className="absolute bottom-4 left-4 flex items-center gap-3">
         <div className="p-3 bg-white/10 backdrop-blur-md rounded-xl border border-white/20 shadow-lg">
            {theme === 'water' && <Waves className="text-cyan-300" size={24} />}
            {theme === 'craft' && <Palette className="text-indigo-300" size={24} />}
            {theme === 'nature' && <Mountain className="text-emerald-300" size={24} />}
            {theme === 'tech' && <Zap className="text-blue-300" size={24} />}
            {theme === 'future' && <Box className="text-violet-300" size={24} />}
         </div>
         <div>
            <h3 className="text-white font-bold text-lg tracking-wide drop-shadow-md">
              {theme === 'water' ? '水鄉文化' : 
               theme === 'craft' ? '傳統工藝' : 
               theme === 'nature' ? '自然生態' : 
               theme === 'tech' ? '數字科技' : '未來生活'}
            </h3>
            <div className="h-1 w-12 bg-white/50 rounded-full mt-1"></div>
         </div>
      </div>
      
      {/* Decorative Badge */}
      <div className="absolute top-3 right-3">
         <span className="text-[10px] font-mono text-white/60 border border-white/20 px-2 py-1 rounded-full uppercase tracking-wider bg-black/20">
           Theme: {theme ? theme.toUpperCase() : 'DEFAULT'}
         </span>
      </div>
    </div>
  );
};

const WeatherBadge = ({ type, temp }) => {
  let icon = <Sun size={16} className="text-yellow-400" />;
  if (type === 'cloudy') icon = <Cloud size={16} className="text-gray-200" />;
  if (type === 'rain') icon = <CloudRain size={16} className="text-blue-300" />;

  return (
    <div className="flex items-center gap-1 bg-white/10 backdrop-blur-md px-2 py-1 rounded-full text-xs font-mono text-cyan-100 border border-white/10">
      {icon}
      <span>{temp}°C</span>
    </div>
  );
};

// --- Firebase Initialization Outside Component ---

let app, auth, db;
const firebaseConfigStr = typeof __firebase_config !== 'undefined' ? __firebase_config : null;
const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

if (firebaseConfigStr) {
  try {
    const firebaseConfig = JSON.parse(firebaseConfigStr);
    app = initializeApp(firebaseConfig);
    auth = getAuth(app);
    db = getFirestore(app);
  } catch (e) {
    console.error("Firebase init failed:", e);
  }
}

export default function HangzhouApp() {
  const [activeTab, setActiveTab] = useState('itinerary');
  const [itinerary, setItinerary] = useState(INITIAL_ITINERARY);
  const [selectedDay, setSelectedDay] = useState(1);
  const [showEditModal, setShowEditModal] = useState(null); 
  const [splitBill, setSplitBill] = useState({ amount: '', payer: '', sharers: 2 });
  const [weatherLoading, setWeatherLoading] = useState(false);
  const [user, setUser] = useState(null);
  const [dbConnected, setDbConnected] = useState(false);

  // --- Firebase Auth & Sync ---

  useEffect(() => {
    if (!auth) return;

    const initAuth = async () => {
      if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
        await signInWithCustomToken(auth, __initial_auth_token);
      } else {
        await signInAnonymously(auth);
      }
    };

    initAuth();
    const unsubscribe = onAuthStateChanged(auth, setUser);
    return () => unsubscribe();
  }, []);

  useEffect(() => {
    if (!user || !db) return;

    const tripDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'trip_data', 'main');

    const unsubscribe = onSnapshot(tripDocRef, (docSnap) => {
      if (docSnap.exists()) {
        const data = docSnap.data();
        if (data.itinerary) setItinerary(data.itinerary);
        if (data.splitBill) setSplitBill(data.splitBill);
        setDbConnected(true);
      } else {
        setDoc(tripDocRef, {
          itinerary: INITIAL_ITINERARY,
          splitBill: { amount: '', payer: '', sharers: 2 }
        });
      }
    }, (error) => {
      console.error("Sync error:", error);
    });

    return () => unsubscribe();
  }, [user]);

  // --- Actions ---

  const syncToCloud = async (newItinerary, newBill) => {
    if (!db || !user) {
      if (newItinerary) setItinerary(newItinerary);
      if (newBill) setSplitBill(newBill);
      return;
    }
    
    const tripDocRef = doc(db, 'artifacts', appId, 'public', 'data', 'trip_data', 'main');
    try {
      await updateDoc(tripDocRef, {
        ...(newItinerary && { itinerary: newItinerary }),
        ...(newBill && { splitBill: newBill })
      });
    } catch (e) {
      console.error("Save failed:", e);
    }
  };

  const handleDaySelect = (day) => {
    setSelectedDay(day);
    document.getElementById('day-content')?.scrollTo(0, 0);
  };

  const deleteEvent = (dayIndex, eventId) => {
    const newItinerary = [...itinerary];
    newItinerary[dayIndex].events = newItinerary[dayIndex].events.filter(e => e.id !== eventId);
    syncToCloud(newItinerary, null);
  };

  const addEvent = (dayIndex) => {
    const newItinerary = [...itinerary];
    const newEvent = {
      id: Date.now(),
      time: '00:00',
      location: '新地點',
      type: 'culture',
      desc: '點擊編輯詳情',
      weather: 'sunny',
      temp: 25,
      learning: '',
      x: 300, y: 300
    };
    newItinerary[dayIndex].events.push(newEvent);
    syncToCloud(newItinerary, null);
    setShowEditModal(newEvent.id);
  };

  const updateEvent = (dayIndex, updatedEvent) => {
    const newItinerary = [...itinerary];
    const eventIndex = newItinerary[dayIndex].events.findIndex(e => e.id === updatedEvent.id);
    newItinerary[dayIndex].events[eventIndex] = updatedEvent;
    syncToCloud(newItinerary, null);
    setShowEditModal(null);
  };

  const addJournalEntry = (dayIndex, text) => {
    if (!text.trim()) return;
    const newItinerary = [...itinerary];
    newItinerary[dayIndex].journal.push({
      id: Date.now(),
      text,
      time: new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}),
      sender: user ? user.uid.slice(0, 4) : 'User'
    });
    syncToCloud(newItinerary, null);
  };

  const handleBillChange = (field, value) => {
    const newBill = { ...splitBill, [field]: value };
    syncToCloud(null, newBill);
  };

  const refreshWeather = () => {
    setWeatherLoading(true);
    setTimeout(() => {
      const newItinerary = [...itinerary];
      newItinerary.forEach(day => {
        day.events.forEach(ev => {
          ev.temp = Math.floor(Math.random() * (30 - 18) + 18);
          ev.weather = ['sunny', 'cloudy', 'rain'][Math.floor(Math.random() * 3)];
        });
      });
      syncToCloud(newItinerary, null);
      setWeatherLoading(false);
    }, 1500);
  };

  // --- Views ---

  const ItineraryView = () => {
    const dayData = itinerary[selectedDay - 1];
    const [msgInput, setMsgInput] = useState('');
    const [copySuccess, setCopySuccess] = useState(false);

    const copyItinerary = () => {
      const text = `📅 ${dayData.date}: ${dayData.title}\n🎯 重點: ${dayData.focus}\n` + 
        dayData.events.map(e => `⏰ ${e.time} ${e.location} - ${e.desc}`).join('\n');
      
      navigator.clipboard.writeText(text).then(() => {
        setCopySuccess(true);
        setTimeout(() => setCopySuccess(false), 2000);
      });
    };

    return (
      <div className="flex flex-col h-full overflow-hidden relative text-white">
        <WaterTownBackground />

        {/* Date Selector */}
        <div className="z-10 bg-slate-900/30 backdrop-blur-md border-b border-teal-500/20 pt-4 pb-2">
           <div className="flex justify-between items-center px-4 mb-2">
              <h2 className="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-teal-200 to-cyan-100 drop-shadow">
                {dayData.date}: {dayData.title}
              </h2>
              <div className="flex gap-2">
                <a 
                  href="https://www.canva.com/design/DAG-T9kK_7A/W3RMeUIeSsDeN5OjZ8NeQA/view?utm_content=DAG-T9kK_7A&utm_campaign=designshare&utm_medium=link2&utm_source=uniquelinks&utlId=h60e34481e7" 
                  target="_blank" 
                  rel="noopener noreferrer"
                  className="p-2 rounded-full hover:bg-white/10 transition text-cyan-300 relative"
                  title="View Presentation"
                >
                  <ExternalLink size={20} />
                </a>
                <button onClick={copyItinerary} className="p-2 rounded-full hover:bg-white/10 transition text-cyan-300 relative">
                  {copySuccess ? <CheckCircle size={20} className="text-green-400"/> : <Copy size={20} />}
                </button>
                <button onClick={refreshWeather} disabled={weatherLoading} className="p-2 rounded-full hover:bg-white/10 transition">
                   {weatherLoading ? <div className="animate-spin w-5 h-5 border-2 border-cyan-400 border-t-transparent rounded-full"/> : <CloudRain size={20} className="text-cyan-300"/>}
                </button>
              </div>
           </div>
           
           <div className="flex overflow-x-auto space-x-4 px-4 pb-2 scrollbar-hide">
             {itinerary.map((d, idx) => (
               <button 
                 key={d.day}
                 onClick={() => handleDaySelect(d.day)}
                 className={`flex-shrink-0 flex flex-col items-center justify-center w-16 h-20 rounded-2xl transition-all duration-300 border backdrop-blur-sm ${
                   selectedDay === d.day 
                   ? 'bg-gradient-to-b from-teal-800/80 to-slate-900/80 border-teal-400 shadow-[0_0_15px_rgba(45,212,191,0.3)] transform scale-105' 
                   : 'bg-white/5 border-white/10 text-gray-400 hover:bg-white/10'
                 }`}
               >
                 <span className="text-xs font-bold uppercase tracking-wider">Day</span>
                 <span className="text-2xl font-black font-sans">{d.day}</span>
               </button>
             ))}
           </div>
        </div>

        {/* Main Itinerary Content */}
        <div id="day-content" className="flex-1 overflow-y-auto z-10 p-4 space-y-6 scroll-smooth">
          <ThemeHeader theme={dayData.theme} />

          <div className="bg-slate-900/40 backdrop-blur-md border border-teal-500/30 p-4 rounded-xl shadow-lg">
             <div className="flex items-center gap-2 mb-2 text-teal-300">
                <Navigation size={18} />
                <h3 className="font-bold">今日學習重點</h3>
             </div>
             <p className="text-sm text-teal-50/90 leading-relaxed">{dayData.focus}</p>
          </div>

          <div className="relative border-l-2 border-teal-500/20 ml-3 space-y-8 pb-10 mt-6">
            {dayData.events.map((event) => (
              <div key={event.id} className="relative pl-6 group">
                <div className={`absolute left-[-9px] top-0 w-4 h-4 rounded-full border-2 ${
                  event.type === 'tech' ? 'bg-cyan-900 border-cyan-400' : 
                  'bg-teal-900 border-teal-400'
                } z-10 group-hover:scale-125 transition-transform duration-300 shadow-[0_0_10px_rgba(0,0,0,0.5)]`} />
                
                <div className="bg-slate-900/60 backdrop-blur-md border border-white/10 rounded-xl p-4 hover:border-teal-400/50 transition-colors duration-300 shadow-md">
                   <div className="flex justify-between items-start mb-2">
                      <div>
                        <span className="text-2xl font-light text-white font-mono">{event.time}</span>
                        <h4 className="text-lg font-bold text-teal-200 mt-1">{event.location}</h4>
                      </div>
                      <div className="flex flex-col items-end gap-2">
                        <WeatherBadge type={event.weather} temp={event.temp} />
                        <span className={`px-2 py-0.5 rounded text-[10px] uppercase tracking-wider border border-white/10 ${
                          event.type === 'tech' ? 'bg-cyan-900/50 text-cyan-300' : 
                          event.type === 'food' ? 'bg-orange-900/50 text-orange-300' :
                          'bg-teal-900/50 text-teal-300'
                        }`}>
                          {event.type}
                        </span>
                      </div>
                   </div>

                   <p className="text-gray-300 text-sm mb-3">{event.desc}</p>
                   
                   {event.learning && (
                     <div className="flex gap-2 items-start bg-black/30 p-2 rounded-lg mb-3 border border-white/5">
                       <Cpu size={14} className="mt-1 text-teal-400 shrink-0"/>
                       <p className="text-xs text-teal-100/70 italic">{event.learning}</p>
                     </div>
                   )}

                   <div className="flex gap-3 mt-4 border-t border-white/5 pt-3">
                      <button className="flex-1 flex items-center justify-center gap-2 py-2 bg-white/5 rounded hover:bg-white/10 transition text-xs text-gray-400">
                        <Camera size={14} /> Add Photo
                      </button>
                      <button onClick={() => setShowEditModal(event.id)} className="p-2 rounded hover:bg-cyan-500/20 hover:text-cyan-400 text-gray-400 transition">
                        <Edit2 size={14} />
                      </button>
                      <button onClick={() => deleteEvent(selectedDay - 1, event.id)} className="p-2 rounded hover:bg-red-500/20 hover:text-red-400 text-gray-400 transition">
                        <Trash2 size={14} />
                      </button>
                   </div>
                </div>
              </div>
            ))}
            
            <button onClick={() => addEvent(selectedDay - 1)} className="ml-6 flex items-center gap-2 text-sm text-teal-400 hover:text-teal-300 transition">
              <div className="w-8 h-8 rounded-full border border-dashed border-teal-500 flex items-center justify-center">
                <Plus size={16} />
              </div>
              新增行程
            </button>
          </div>

          {/* Message Board with Cloud Sync */}
          <div className="mt-8 bg-slate-900/60 backdrop-blur-md rounded-xl p-4 border border-teal-500/20 shadow-lg">
            <h3 className="text-sm font-bold text-gray-400 uppercase tracking-widest mb-4 flex items-center gap-2">
              <Share2 size={14} /> 雲端留言板 {dbConnected ? <div className="w-2 h-2 rounded-full bg-teal-500 shadow-[0_0_10px_#14b8a6]"/> : <div className="w-2 h-2 rounded-full bg-red-500"/>}
            </h3>
            <div className="space-y-3 mb-4 max-h-40 overflow-y-auto">
              {dayData.journal.length === 0 ? (
                <p className="text-xs text-gray-500 italic text-center py-4">還沒有留言...</p>
              ) : (
                dayData.journal.map(j => (
                  <div key={j.id} className="bg-white/5 p-3 rounded-lg text-sm border border-white/5">
                    <div className="flex justify-between items-baseline mb-1">
                      <span className="text-teal-400 font-bold text-xs">{j.sender || 'User'}</span>
                      <span className="text-[10px] text-gray-500">{j.time}</span>
                    </div>
                    <p className="text-gray-200">{j.text}</p>
                  </div>
                ))
              )}
            </div>
            <div className="flex gap-2">
               <input 
                 type="text" 
                 value={msgInput}
                 onChange={(e) => setMsgInput(e.target.value)}
                 placeholder="今天學到了什麼..."
                 className="flex-1 bg-black/40 border border-white/10 rounded-lg px-3 text-sm text-white focus:outline-none focus:border-teal-500 placeholder-gray-600"
                 onKeyDown={(e) => e.key === 'Enter' && (addJournalEntry(selectedDay - 1, msgInput), setMsgInput(''))}
               />
               <button onClick={() => {addJournalEntry(selectedDay - 1, msgInput); setMsgInput('');}} className="p-2 bg-teal-700/80 rounded-lg hover:bg-teal-600 transition text-white">
                 <Send size={16} />
               </button>
            </div>
          </div>
          <div className="h-20" />
        </div>

        {/* Edit Modal Overlay */}
        {showEditModal && (
          <div className="absolute inset-0 z-50 bg-black/80 backdrop-blur-sm flex items-center justify-center p-4">
             <div className="bg-slate-900 border border-teal-500/30 w-full max-w-sm rounded-2xl p-6 shadow-2xl">
               <h3 className="text-lg font-bold text-white mb-4">編輯行程</h3>
               {(() => {
                 const ev = dayData.events.find(e => e.id === showEditModal);
                 if (!ev) return null;
                 return (
                   <div className="space-y-3">
                     <div>
                       <label className="text-xs text-gray-400 block mb-1">時間</label>
                       <input type="time" value={ev.time} onChange={(e) => updateEvent(selectedDay-1, {...ev, time: e.target.value})} className="w-full bg-black/40 border border-white/10 rounded p-2 text-white" />
                     </div>
                     <div>
                       <label className="text-xs text-gray-400 block mb-1">地點</label>
                       <input type="text" value={ev.location} onChange={(e) => updateEvent(selectedDay-1, {...ev, location: e.target.value})} className="w-full bg-black/40 border border-white/10 rounded p-2 text-white" />
                     </div>
                     <div>
                       <label className="text-xs text-gray-400 block mb-1">簡介</label>
                       <textarea value={ev.desc} onChange={(e) => updateEvent(selectedDay-1, {...ev, desc: e.target.value})} className="w-full bg-black/40 border border-white/10 rounded p-2 text-white h-20" />
                     </div>
                     <div className="flex gap-2 mt-4">
                       <button onClick={() => setShowEditModal(null)} className="flex-1 py-2 bg-gray-700 rounded text-gray-300">取消</button>
                       <button onClick={() => setShowEditModal(null)} className="flex-1 py-2 bg-teal-700 rounded text-white font-bold">保存</button>
                     </div>
                   </div>
                 );
               })()}
             </div>
          </div>
        )}
      </div>
    );
  };

  const MapView = () => {
    const dayData = itinerary[selectedDay - 1];
    const [selectedPin, setSelectedPin] = useState(null);

    return (
      <div className="h-full bg-[#e8eaed] relative flex flex-col font-sans">
        <div className="absolute top-4 left-4 right-4 z-20">
           <div className="bg-white/90 backdrop-blur rounded-lg shadow-md p-3 flex items-center gap-3 border border-teal-100">
              <Menu className="text-gray-500" size={20} />
              <div className="flex-1 text-sm text-gray-700 font-medium">{dayData.title}</div>
              <Mic className="text-gray-400" size={20} />
              <div className="w-8 h-8 rounded-full bg-teal-600 flex items-center justify-center text-white text-xs font-bold shadow-sm">U</div>
           </div>
           <div className="flex gap-2 mt-3 overflow-x-auto scrollbar-hide">
              <button className="bg-white/90 text-teal-800 text-xs px-3 py-1.5 rounded-full shadow-sm border border-teal-100 whitespace-nowrap">餐廳</button>
              <button className="bg-white/90 text-teal-800 text-xs px-3 py-1.5 rounded-full shadow-sm border border-teal-100 whitespace-nowrap">景點</button>
              <button className="bg-white/90 text-teal-800 text-xs px-3 py-1.5 rounded-full shadow-sm border border-teal-100 whitespace-nowrap">酒店</button>
              <button className="bg-white/90 text-teal-800 text-xs px-3 py-1.5 rounded-full shadow-sm border border-teal-100 whitespace-nowrap">更多</button>
           </div>
        </div>

        <div className="flex-1 overflow-auto relative cursor-grab active:cursor-grabbing">
           <div className="w-[800px] h-[800px] bg-[#eef2f3] relative">
              <svg className="absolute inset-0 w-full h-full pointer-events-none">
                 <path d="M0,400 Q150,350 300,450 T600,400 T800,500 L800,800 L0,800 Z" fill="#d1fae5" opacity="0.5" /> 
                 <path d="M100,0 Q150,200 100,400 T50,800 L0,800 L0,0 Z" fill="#cffafe" opacity="0.8" /> 
                 <path d="M500,200 Q600,150 700,250 T800,200 L800,0 L500,0 Z" fill="#cffafe" opacity="0.8" /> 
              </svg>
              <div className="absolute inset-0" style={{ backgroundImage: 'linear-gradient(#cbd5e1 1px, transparent 1px), linear-gradient(90deg, #cbd5e1 1px, transparent 1px)', backgroundSize: '50px 50px', opacity: 0.3 }}></div>
              <svg className="absolute inset-0 w-full h-full pointer-events-none">
                 <path d="M0,100 L800,150" stroke="white" strokeWidth="12" fill="none" />
                 <path d="M200,0 L250,800" stroke="white" strokeWidth="12" fill="none" />
                 <path d="M0,500 L800,550" stroke="white" strokeWidth="10" fill="none" />
                 <path d="M500,0 L450,800" stroke="white" strokeWidth="10" fill="none" />
              </svg>
              <div className="absolute top-[350px] left-[200px] z-10">
                 <div className="w-16 h-16 bg-teal-500/20 rounded-full flex items-center justify-center animate-pulse">
                    <div className="w-4 h-4 bg-white rounded-full flex items-center justify-center shadow-lg"><div className="w-3 h-3 bg-teal-600 rounded-full"></div></div>
                 </div>
              </div>

              {dayData.events.map((event) => (
                <div key={event.id} className="absolute transform -translate-x-1/2 -translate-y-full transition-all duration-300 hover:scale-110 z-20" style={{ left: event.x, top: event.y }} onClick={() => setSelectedPin(event)}>
                   <div className={`flex flex-col items-center cursor-pointer ${selectedPin?.id === event.id ? 'scale-125 z-50' : ''}`}>
                      <div className={`w-8 h-8 rounded-full border-2 border-white shadow-md flex items-center justify-center text-white ${event.type === 'tech' ? 'bg-cyan-500' : event.type === 'food' ? 'bg-orange-500' : 'bg-teal-500'}`}>
                         {event.type === 'tech' ? <Cpu size={14} /> : event.type === 'food' ? <Utensils size={14} /> : <MapPin size={14} />}
                      </div>
                      <div className="w-0 h-0 border-l-[6px] border-l-transparent border-r-[6px] border-r-transparent border-t-[8px] border-t-white mt-[-2px]"></div>
                      <div className="w-2 h-1 bg-black/20 rounded-full blur-[1px]"></div>
                      <span className="mt-1 text-[10px] font-bold text-teal-900 bg-white/90 px-1.5 py-0.5 rounded shadow-sm border border-teal-100 whitespace-nowrap">{event.location}</span>
                   </div>
                </div>
              ))}
           </div>
        </div>
        {selectedPin && (
          <div className="absolute bottom-20 left-4 right-4 bg-white/95 backdrop-blur rounded-xl shadow-2xl p-4 z-30 border border-teal-100 animate-in slide-in-from-bottom-10 fade-in duration-300">
             <div className="flex justify-between items-start">
                <div>
                   <h3 className="text-lg font-bold text-gray-800">{selectedPin.location}</h3>
                   <div className="flex items-center gap-2 mt-1">
                      <span className="text-sm text-gray-600">{selectedPin.time}</span>
                      <span className="w-1 h-1 bg-gray-300 rounded-full"></span>
                      <span className={`text-xs px-2 py-0.5 rounded-full ${selectedPin.type === 'tech' ? 'bg-blue-100 text-blue-700' : 'bg-teal-100 text-teal-700'}`}>{selectedPin.type}</span>
                   </div>
                </div>
                <button onClick={() => setSelectedPin(null)} className="p-1 bg-gray-100 rounded-full text-gray-500"><X size={16} /></button>
             </div>
             <p className="text-gray-600 text-sm mt-3 leading-relaxed">{selectedPin.desc}</p>
             <div className="flex gap-3 mt-4">
                <button className="flex-1 bg-teal-600 text-white py-2 rounded-full font-medium text-sm flex items-center justify-center gap-2 shadow-lg shadow-teal-200"><Navigation size={16} /> 路線</button>
                <button className="flex-1 bg-white border border-teal-600 text-teal-600 py-2 rounded-full font-medium text-sm">開始導航</button>
             </div>
          </div>
        )}
        <div className="absolute bottom-24 right-4 flex flex-col gap-3 z-20">
           <button className="w-10 h-10 bg-white rounded-full shadow-md flex items-center justify-center text-gray-600 border border-gray-200"><Layers size={20} /></button>
           <button className="w-10 h-10 bg-white rounded-full shadow-md flex items-center justify-center text-teal-600 border border-gray-200"><Locate size={20} /></button>
           <button className="w-10 h-10 bg-teal-600 rounded-lg shadow-lg flex items-center justify-center text-white"><ArrowRightCircle size={20} /></button>
        </div>
      </div>
    );
  };

  const BudgetView = () => {
    const calculateSplit = () => {
       if(!splitBill.amount || !splitBill.sharers) return 0;
       return (parseFloat(splitBill.amount) / parseInt(splitBill.sharers)).toFixed(1);
    };

    return (
      <div className="flex flex-col h-full overflow-hidden relative text-white">
        <WaterTownBackground />
        
        <div className="z-10 p-4 overflow-y-auto h-full">
          <h2 className="text-xl font-bold mb-6 bg-clip-text text-transparent bg-gradient-to-r from-teal-200 to-cyan-100 drop-shadow">
            財政預算 & 分帳 {dbConnected ? <span className="text-xs text-teal-300 font-normal border border-teal-500/50 rounded px-2 align-middle ml-2">Cloud Sync ON</span> : null}
          </h2>

          <div className="bg-slate-900/60 backdrop-blur-md border border-teal-500/30 rounded-2xl p-5 mb-6 shadow-lg relative overflow-hidden">
             <div className="absolute top-[-10px] right-[-10px] w-20 h-20 bg-teal-500/20 rounded-full blur-xl"></div>
             <h3 className="text-teal-300 font-bold mb-4 flex items-center gap-2">
               <DollarSign size={18} /> 快速分帳
             </h3>
             <div className="space-y-3">
               <div className="flex gap-2">
                  <input type="number" placeholder="總金額 ($)" value={splitBill.amount} onChange={e => handleBillChange('amount', e.target.value)} className="flex-1 bg-black/40 border border-white/10 rounded-lg p-2 text-white focus:border-teal-400 outline-none placeholder-gray-500" />
                  <input type="number" placeholder="人數" value={splitBill.sharers} onChange={e => handleBillChange('sharers', e.target.value)} className="w-20 bg-black/40 border border-white/10 rounded-lg p-2 text-white focus:border-teal-400 outline-none" />
               </div>
               <div className="flex items-center justify-between bg-black/30 rounded-lg p-3 mt-2 border border-white/5">
                  <span className="text-gray-400 text-sm">每人應付:</span>
                  <span className="text-2xl font-bold text-cyan-400">${calculateSplit()}</span>
               </div>
               <button className="w-full py-2 bg-teal-700/90 hover:bg-teal-600 rounded-lg text-sm font-bold transition mt-2 border border-teal-500/30">發送給朋友 (Simulated)</button>
             </div>
          </div>

          <h3 className="text-gray-400 text-sm font-bold uppercase tracking-wider mb-3 pl-1">參考價格 (人均)</h3>
          <div className="space-y-2 pb-20">
            {BUDGET_ITEMS.map((item, idx) => (
              <div key={idx} className="flex justify-between items-center bg-slate-900/40 backdrop-blur-sm border border-teal-500/10 p-3 rounded-xl hover:bg-white/5 transition">
                 <span className="text-gray-300 text-sm">{item.item}</span>
                 <span className="text-teal-300 font-mono font-bold">${item.price}</span>
              </div>
            ))}
            <div className="mt-6 pt-4 border-t border-white/10 flex justify-between items-end">
               <span className="text-gray-500 text-xs">Based on provided doc</span>
               <div className="text-right">
                  <span className="block text-gray-400 text-xs">Estimated Total</span>
                  <span className="text-2xl font-bold text-white">$5,817</span>
               </div>
            </div>
          </div>
        </div>
      </div>
    );
  };

  return (
    <div className="w-full max-w-md mx-auto h-screen bg-slate-900 shadow-2xl overflow-hidden flex flex-col font-sans">
      <div className="flex-1 overflow-hidden relative">
        {activeTab === 'itinerary' && <ItineraryView />}
        {activeTab === 'map' && <MapView />}
        {activeTab === 'budget' && <BudgetView />}
      </div>
      <div className="bg-slate-900/90 backdrop-blur-lg border-t border-teal-500/20 p-2 px-6 flex justify-between items-center z-50">
         <button onClick={() => setActiveTab('itinerary')} className={`flex flex-col items-center gap-1 transition ${activeTab === 'itinerary' ? 'text-teal-400 scale-110' : 'text-gray-500 hover:text-gray-300'}`}><Calendar size={20} /><span className="text-[10px] font-bold">行程</span></button>
         <button onClick={() => setActiveTab('map')} className={`flex flex-col items-center gap-1 transition ${activeTab === 'map' ? 'text-teal-400 scale-110' : 'text-gray-500 hover:text-gray-300'}`}><div className={`p-3 rounded-full -mt-8 border-4 border-slate-900 ${activeTab === 'map' ? 'bg-gradient-to-r from-teal-500 to-cyan-500 text-white shadow-[0_0_20px_rgba(45,212,191,0.6)]' : 'bg-slate-800 text-gray-400'}`}><Navigation size={24} fill={activeTab === 'map' ? "currentColor" : "none"} /></div><span className="text-[10px] font-bold">地圖</span></button>
         <button onClick={() => setActiveTab('budget')} className={`flex flex-col items-center gap-1 transition ${activeTab === 'budget' ? 'text-teal-400 scale-110' : 'text-gray-500 hover:text-gray-300'}`}><DollarSign size={20} /><span className="text-[10px] font-bold">分帳</span></button>
      </div>
    </div>
  );
}# STCEC
