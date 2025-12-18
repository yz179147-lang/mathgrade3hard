<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>小小分析師：高階邏輯挑戰賽</title>
    
    <!-- 引入 React 和 ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    
    <!-- 引入 Babel 用來解析 JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    
    <!-- 引入 Tailwind CSS 進行排版 -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- 設定 Tailwind -->
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Helvetica', 'Arial', 'PingFang TC', 'Heiti TC', 'Microsoft JhengHei', 'sans-serif'],
                    },
                    colors: {
                        brand: '#0f172a', 
                        accent: '#f59e0b',
                    },
                    animation: {
                        'fade-in': 'fadeIn 0.5s ease-out forwards',
                    },
                    keyframes: {
                        fadeIn: {
                            '0%': { opacity: '0', transform: 'translateY(10px)' },
                            '100%': { opacity: '1', transform: 'translateY(0)' },
                        }
                    }
                }
            }
        }
    </script>

    <style>
        body {
            -webkit-tap-highlight-color: transparent;
            -webkit-touch-callout: none;
            overscroll-behavior-y: none;
        }
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen flex flex-col">
    <div id="root" class="flex-grow flex flex-col"></div>

    <script type="text/babel">
        const { useState, useEffect } = React;

        // --- SVG 圖示庫 ---
        const Icon = ({ d, children, className, ...props }) => (
            <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" className={`w-6 h-6 ${className}`} {...props}>
                {d ? <path d={d} /> : children}
            </svg>
        );

        const Icons = {
            Brain: (props) => <Icon {...props}><path d="M9.5 2A2.5 2.5 0 0 1 12 4.5v15a2.5 2.5 0 0 1-4.96.44 2.5 2.5 0 0 1-2.96-3.08 3 3 0 0 1-.34-5.58 2.5 2.5 0 0 1 1.32-4.24 2.5 2.5 0 0 1 1.98-3A2.5 2.5 0 0 1 9.5 2Z"/><path d="M14.5 2A2.5 2.5 0 0 0 12 4.5v15a2.5 2.5 0 0 0 4.96.44 2.5 2.5 0 0 0 2.96-3.08 3 3 0 0 0 .34-5.58 2.5 2.5 0 0 0-1.32-4.24 2.5 2.5 0 0 0-1.98-3A2.5 2.5 0 0 0 14.5 2Z"/></Icon>,
            Leaf: (props) => <Icon d="M11 20A7 7 0 0 1 9.8 6.1C15.5 5 17 4.48 19 2c1 2 2 4.18 2 8 0 5.5-4.78 10-10 10Z" {...props}><path d="M2 21c0-3 1.85-5.36 5.08-6C9.5 14.52 12 13 13 12"/></Icon>,
            Truck: (props) => <Icon d="M10 17h4V5H2v12h3" {...props}><path d="M20 17h2v-3.34a4 4 0 0 0-1.17-2.83L19 9h-5"/><path d="M14 17h1"/><circle cx="7.5" cy="17.5" r="2.5"/><circle cx="17.5" cy="17.5" r="2.5"/></Icon>,
            Sun: (props) => <Icon {...props}><circle cx="12" cy="12" r="4"/><path d="M12 2v2"/><path d="M12 20v2"/><path d="M4.93 4.93l1.41 1.41"/><path d="M17.66 17.66l1.41 1.41"/><path d="M2 12h2"/><path d="M20 12h2"/><path d="M6.34 17.66l-1.41 1.41"/><path d="M19.07 4.93l-1.41 1.41"/></Icon>,
            Wifi: (props) => <Icon d="M5 12.55a11 11 0 0 1 14.08 0" {...props}><path d="M1.42 9a16 16 0 0 1 21.16 0"/><path d="M8.53 16.11a6 6 0 0 1 6.95 0"/><line x1="12" y1="20" x2="12.01" y2="20"/></Icon>,
            Box: (props) => <Icon d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z" {...props}><polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/></Icon>,
            Check: (props) => <Icon d="M20 6 9 17l-5-5" {...props} />,
            X: (props) => <Icon d="M18 6 6 18M6 6l12 12" {...props} />,
            ChevronRight: (props) => <Icon d="m9 18 6-6-6-6" {...props} />,
            ChevronLeft: (props) => <Icon d="m15 18-6-6 6-6" {...props} />,
            Trophy: (props) => <Icon d="M6 9H4.5a2.5 2.5 0 0 1 0-5H6" {...props}><path d="M18 9h1.5a2.5 2.5 0 0 0 0-5H18"/><path d="M4 22h16"/><path d="M10 14.66V17a2 2 0 0 0-2 2v3h8v-3a2 2 0 0 0-2-2v-2.34"/><path d="M22 2h-4a2.5 2.5 0 0 0-1-1.34 2.5 2.5 0 0 0-3 0 2.5 2.5 0 0 0-3 0 2.5 2.5 0 0 0-1 1.34H2"/><path d="M6 2v7a6 6 0 0 0 6 6 6 6 0 0 0 6-6V2"/></Icon>,
            Refresh: (props) => <Icon d="M3 12a9 9 0 0 1 9-9 9.75 9.75 0 0 1 6.74 2.74L21 8" {...props}><path d="M21 3v5h-5"/><path d="M21 12a9 9 0 0 1-9 9 9.75 9.75 0 0 1-6.74-2.74L3 16"/><path d="M8 16H3v5"/></Icon>
        };

        // --- 進階題庫與視覺化 (移除所有提示性括號，優化邏輯) ---

        const problemSets = [
            {
                id: 1,
                title: "減碳旅行日記 (碳排放計算)",
                icon: <Icons.Leaf className="text-green-500" />,
                story: "小綠是環保小尖兵，她正在規劃從台北去高雄的旅行，單趟距離是 300 公里。她給自己設定了一個目標：整趟來回的總碳排放量不能超過 50 公斤。她決定去程搭高鐵，回程搭爸爸開的休旅車。",
                visualType: "carbon_chart",
                questions: [
                    { q: "請計算小綠去程搭乘高鐵會產生多少公斤的碳排放？", a: "12 公斤 (300 ÷ 25 = 12)" },
                    { q: "回程搭爸爸的休旅車，會產生多少公斤的碳排放？", a: "60 公斤 (300 ÷ 5 = 60)" },
                    { q: "請問這趟「來回旅行」總共產生的碳排放量是多少公斤？", a: "72 公斤 (12 + 60 = 72)" },
                    { q: "這趟旅行的總排放量與小綠設定的目標相差多少公斤？", a: "相差 22 公斤 (72 - 50 = 22，超標了)" },
                    { q: "如果回程改成搭乘長途客運，能不能達成她的環保目標？請以計算結果說明。", a: "可以。去程12 + 回程客運10 (300÷30) = 22公斤，小於目標 50 公斤。" }
                ]
            },
            {
                id: 2,
                title: "外送平台的精打細算 (最佳化)",
                icon: <Icons.Truck className="text-orange-500" />,
                story: "媽媽要用「胖胖達」App 訂購晚餐。她看上了兩家餐廳：「快樂炸雞」和「健康滷味」。\n平台規定：\n1. 每家餐廳的運費是分開計算的。\n2. 單一家餐廳訂單滿 400 元，該筆運費可以折抵 20 元。\n3. 會員優惠：如果兩家餐廳的「食物總金額」加起來超過 1000 元，可以再折價 50 元。",
                visualType: "delivery_app",
                questions: [
                    { q: "如果在快樂炸雞點了 450 元的食物，這家店最後需支付的總金額是多少？", a: "480 元 (食物450 + 運費50 - 運費折抵20)" },
                    { q: "媽媽在健康滷味點了 300 元的食物，這家店最後需支付的總金額是多少？", a: "330 元 (食物300 + 運費30，未滿400無折抵)" },
                    { q: "承上兩題，如果不考慮會員優惠，這兩張訂單分開結帳，總共要花多少錢？", a: "810 元 (480 + 330)" },
                    { q: "媽媽後來決定在健康滷味多點一些，讓食物金額變成 600 元，而炸雞維持 450 元。請問現在兩家食物總金額是否符合會員優惠資格？", a: "符合。總金額 1050 元 (450+600)，大於 1000 元。" },
                    { q: "在修正後的訂單金額下（炸雞450、滷味600），計算所有折扣後，媽媽最後總共要付多少錢？", a: "1040 元。計算：(炸雞450+運費30) + (滷味600+運費10) - 會員折50。註：炸雞運費折抵後剩30，滷味運費折抵後剩10。總式：480 + 610 - 50 = 1040。" }
                ]
            },
            {
                id: 3,
                title: "智慧家庭的太陽能電池 (時間與容量)",
                icon: <Icons.Sun className="text-yellow-500" />,
                story: "阿強家屋頂裝了太陽能板和一顆超大蓄電池。這顆電池的總容量是 100 度電。\n運作規則：\n● 白天 (06:00~18:00)：太陽能板發電，每小時可以幫電池充入 8 度電。但家裡同時會消耗 3 度電。\n● 晚上 (18:00~06:00)：沒有太陽，只消耗電池裡的電，每小時消耗 5 度電。",
                visualType: "solar_system",
                questions: [
                    { q: "白天時，電池實際上每小時會增加多少度電？", a: "5 度電 (充電8 - 消耗3 = 5)" },
                    { q: "早上 06:00 時電池是空的，到了中午 12:00，電池裡會有多少度電？", a: "30 度電 (5度 x 6小時)" },
                    { q: "從早上發電到下午 18:00 太陽下山時，電池裡累積了多少度電？", a: "60 度電 (5度 x 12小時)" },
                    { q: "晚上 18:00 開始只耗電不充電。到了晚上 23:00，電池裡的電會剩下多少？", a: "35 度 (60 - (5度 x 5小時) = 35)" },
                    { q: "這顆電池在 18:00 儲存的電量，足夠供應家裡用到隔天早上 06:00 嗎？", a: "剛剛好足夠。晚上共12小時，需 12 x 5 = 60度，電池剛好有60度。" }
                ]
            },
            {
                id: 4,
                title: "網紅影片上傳大作戰 (速率與單位)",
                icon: <Icons.Wifi className="text-blue-500" />,
                story: "YouTuber 傑克拍了一部高畫質影片，檔案大小是 3000 MB。他家裡的網路上傳速度不穩定。\n● 前 10 分鐘：網速很快，每分鐘可以傳 150 MB。\n● 10 分鐘後：網速變慢，每分鐘只能傳 50 MB。",
                visualType: "upload_screen",
                questions: [
                    { q: "前 10 分鐘網速很快的時候，總共上傳了多少檔案？", a: "1500 MB (150 x 10)" },
                    { q: "這部影片還剩下多少容量還沒上傳？", a: "1500 MB (3000 - 1500)" },
                    { q: "剩下的檔案用慢速網路傳輸，還需要幾分鐘才能傳完？", a: "30 分鐘 (1500 ÷ 50)" },
                    { q: "傑克從早上 10:00 開始上傳，請問影片全部傳完是幾點幾分？", a: "10:40 (前10分鐘 + 後30分鐘 = 總共40分鐘)" },
                    { q: "如果傑克想要在 25 分鐘內傳完整部影片，他的平均網速每分鐘至少要多少 MB？", a: "120 MB (3000 ÷ 25)" }
                ]
            },
            {
                id: 5,
                title: "無人機物流中心 (重量與路徑)",
                icon: <Icons.Box className="text-purple-500" />,
                story: "未來的物流中心使用無人機搬貨。無人機代號「飛飛」，它的最大載重量是 5 公斤。\n貨架上有三種包裹：\nA 包裹：2 公斤 500 公克\nB 包裹：1800 公克\nC 包裹：850 公克\n飛飛每次出發都要消耗電力，所以希望能一次拿越多越好。",
                visualType: "drone_warehouse",
                questions: [
                    { q: "請將 A 包裹的重量換算成公克。", a: "2500 公克" },
                    { q: "如果飛飛一次抓取 A、B、C 三個包裹各一個，總重量是多少公克？", a: "5150 公克 (2500 + 1800 + 850)" },
                    { q: "承上題，飛飛能夠一次載走這三個包裹嗎？為什麼？", a: "不能，因為 5150 公克大於最大載重 5000 公克 (5公斤)。" },
                    { q: "如果飛飛必須一次載走兩個 A 包裹，請問它超重了嗎？", a: "沒有，剛好符合 (2500 x 2 = 5000)。" },
                    { q: "飛飛想要一次載走最多數量的 C 包裹，請問它最多可以載幾個而不用分趟？", a: "5 個 (850x5=4250, 850x6=5100 會超重)" }
                ]
            }
        ];

        // --- 視覺化組件 (SVG Components) ---

        const VisualContainer = ({ children, title }) => (
            <div className="bg-white p-4 rounded-xl shadow-sm border border-slate-200 my-4 overflow-hidden">
                <div className="text-xs font-bold text-slate-400 mb-4 uppercase tracking-wider border-b pb-2">{title}</div>
                <div className="flex justify-center items-center w-full overflow-x-auto no-scrollbar">
                    {children}
                </div>
            </div>
        );

        // 1. 碳足跡圖表
        const CarbonChart = () => (
            <div className="w-full max-w-sm">
                <div className="flex items-end justify-around h-40 border-b-2 border-slate-300 pb-2">
                    <div className="flex flex-col items-center gap-2 group">
                        <div className="text-xs font-bold text-slate-500 mb-1">高鐵</div>
                        <div className="w-12 bg-green-400 rounded-t-md relative group-hover:bg-green-500 transition-all h-[40%] flex items-end justify-center">
                            <span className="text-[10px] text-white font-bold mb-1">25km/kg</span>
                        </div>
                    </div>
                    <div className="flex flex-col items-center gap-2 group">
                        <div className="text-xs font-bold text-slate-500 mb-1">客運</div>
                        <div className="w-12 bg-blue-400 rounded-t-md relative group-hover:bg-blue-500 transition-all h-[60%] flex items-end justify-center">
                            <span className="text-[10px] text-white font-bold mb-1">30km/kg</span>
                        </div>
                    </div>
                    <div className="flex flex-col items-center gap-2 group">
                        <div className="text-xs font-bold text-slate-500 mb-1">休旅車</div>
                        <div className="w-12 bg-red-400 rounded-t-md relative group-hover:bg-red-500 transition-all h-[90%] flex items-end justify-center">
                            <span className="text-[10px] text-white font-bold mb-1">5km/kg</span>
                        </div>
                    </div>
                </div>
                <div className="text-center text-xs text-slate-400 mt-2">每排放 1 公斤 CO2 可行駛距離 (數值越大越環保)</div>
                <div className="mt-4 bg-slate-100 p-2 rounded text-xs text-slate-600 text-center font-mono">
                    旅程距離：單趟 300 公里
                </div>
            </div>
        );

        // 2. 外送 APP 介面
        const DeliveryApp = () => (
            <div className="w-64 bg-gray-50 border-2 border-gray-200 rounded-2xl p-4 font-sans shadow-inner">
                <div className="flex justify-between items-center border-b pb-2 mb-2">
                    <div className="font-black text-lg text-orange-500">胖胖達 eats</div>
                    <div className="text-xs bg-orange-100 text-orange-600 px-2 py-1 rounded-full">會員</div>
                </div>
                
                <div className="space-y-3">
                    <div className="bg-white p-3 rounded-lg shadow-sm border border-gray-100">
                        <div className="flex justify-between font-bold text-sm">🍗 快樂炸雞</div>
                        <div className="text-xs text-gray-500 mt-1">運費：$50</div>
                        <div className="text-[10px] text-green-600 mt-1">滿 $400 運費折 $20</div>
                    </div>

                    <div className="bg-white p-3 rounded-lg shadow-sm border border-gray-100">
                        <div className="flex justify-between font-bold text-sm">🥗 健康滷味</div>
                        <div className="text-xs text-gray-500 mt-1">運費：$30</div>
                        <div className="text-[10px] text-green-600 mt-1">滿 $400 運費折 $20</div>
                    </div>
                </div>

                <div className="mt-4 pt-2 border-t border-dashed border-gray-300">
                    <div className="flex justify-between items-center">
                        <span className="text-xs font-bold text-gray-600">全單優惠</span>
                        <span className="text-[10px] bg-red-100 text-red-600 px-1 rounded">滿1000折50</span>
                    </div>
                </div>
            </div>
        );

        // 3. 太陽能系統
        const SolarSystem = () => (
            <div className="w-full max-w-md bg-slate-800 rounded-xl p-4 text-white">
                <div className="flex justify-between items-center mb-6">
                    <div className="flex items-center gap-2">
                        <Icons.Sun className="text-yellow-400" />
                        <span className="text-sm font-bold">太陽能板</span>
                    </div>
                    <div className="h-1 w-16 bg-yellow-400/30 rounded-full overflow-hidden">
                        <div className="h-full bg-yellow-400 w-full animate-pulse"></div>
                    </div>
                    <div className="text-xs text-yellow-400">+8度/hr (白天)</div>
                </div>

                <div className="flex justify-between items-center mb-6">
                    <div className="flex items-center gap-2">
                        <div className="w-6 h-6 border-2 border-green-400 rounded flex items-center justify-center">
                            <div className="w-4 h-3 bg-green-400"></div>
                        </div>
                        <span className="text-sm font-bold">蓄電池</span>
                    </div>
                    <div className="text-xs text-green-400">容量上限：100度</div>
                </div>

                <div className="flex justify-between items-center">
                    <div className="flex items-center gap-2">
                        <div className="w-6 h-6 bg-blue-500 rounded-full flex items-center justify-center text-[10px]">🏠</div>
                        <span className="text-sm font-bold">家庭用電</span>
                    </div>
                    <div className="text-xs text-red-400 text-right">
                        <div>白天消耗：-3度/hr</div>
                        <div>晚上消耗：-5度/hr</div>
                    </div>
                </div>
            </div>
        );

        // 4. 上傳進度條
        const UploadScreen = () => (
            <div className="w-full max-w-sm bg-white p-4 border rounded-xl shadow-sm">
                <div className="flex justify-between mb-2">
                    <span className="font-bold text-sm text-slate-700">正在上傳：Vlog_Final.mp4</span>
                    <span className="text-xs text-slate-500 font-mono">3000 MB</span>
                </div>
                
                {/* Timeline */}
                <div className="relative h-12 bg-slate-100 rounded-lg w-full mb-4 flex overflow-hidden">
                    <div className="w-[25%] bg-blue-500 h-full flex items-center justify-center text-[10px] text-white font-bold border-r border-white/20">
                        前10分<br/>150MB/分
                    </div>
                    <div className="w-[75%] bg-orange-400 h-full flex items-center justify-center text-[10px] text-white font-bold">
                        10分後<br/>50MB/分
                    </div>
                </div>

                <div className="flex justify-between text-xs text-slate-500">
                    <span>0 min</span>
                    <span>10 min</span>
                    <span>?? min</span>
                </div>
            </div>
        );

        // 5. 無人機倉庫
        const DroneWarehouse = () => (
            <div className="flex gap-4 items-center justify-center flex-wrap">
                <div className="flex flex-col items-center">
                    <div className="w-20 h-20 bg-purple-100 border-2 border-purple-500 rounded-lg flex flex-col items-center justify-center relative">
                        <div className="absolute -top-2 -right-2 bg-purple-600 text-white text-[10px] px-2 py-0.5 rounded-full">MAX 5kg</div>
                        <Icons.Box className="w-8 h-8 text-purple-600 mb-1" />
                        <span className="text-xs font-bold text-purple-800">飛飛</span>
                    </div>
                </div>
                
                <div className="text-2xl text-slate-300">➜</div>

                <div className="flex gap-2">
                    <div className="w-16 h-20 bg-slate-50 border border-slate-300 rounded flex flex-col items-center justify-center p-1">
                        <div className="text-xl">📦</div>
                        <div className="text-xs font-bold mt-1">A</div>
                        <div className="text-[10px] text-slate-500">2kg 500g</div>
                    </div>
                    <div className="w-16 h-20 bg-slate-50 border border-slate-300 rounded flex flex-col items-center justify-center p-1">
                        <div className="text-xl">📦</div>
                        <div className="text-xs font-bold mt-1">B</div>
                        <div className="text-[10px] text-slate-500">1800g</div>
                    </div>
                    <div className="w-16 h-20 bg-slate-50 border border-slate-300 rounded flex flex-col items-center justify-center p-1">
                        <div className="text-xl">📦</div>
                        <div className="text-xs font-bold mt-1">C</div>
                        <div className="text-[10px] text-slate-500">850g</div>
                    </div>
                </div>
            </div>
        );

        // --- 主程式邏輯 (App) ---

        const ScoreCard = ({ score, total, onRestart }) => {
            const percentage = Math.round((score / total) * 100);
            let message = "";
            let colorClass = "";

            if (percentage === 100) {
                message = "太神了！邏輯鬼才就是你！🏆";
                colorClass = "text-yellow-500";
            } else if (percentage >= 80) {
                message = "超強的！分析大師！🌟";
                colorClass = "text-green-500";
            } else if (percentage >= 60) {
                message = "不錯喔！觀念很清楚！👍";
                colorClass = "text-blue-500";
            } else {
                message = "再挑戰一次，你會更強！💪";
                colorClass = "text-orange-500";
            }

            return (
                <div className="bg-white p-8 rounded-2xl shadow-xl max-w-md w-full mx-auto text-center animate-pop-in border-4 border-slate-100">
                    <div className={`text-6xl mb-6 flex justify-center ${colorClass}`}>
                        <Icons.Trophy className="w-24 h-24" />
                    </div>
                    <h2 className="text-2xl font-black mb-2 text-slate-800">挑戰結果</h2>
                    <p className={`text-lg font-bold mb-8 ${colorClass}`}>{message}</p>
                    
                    <div className="flex justify-center gap-4 mb-8">
                        <div className="bg-slate-50 p-4 rounded-xl border border-slate-200 w-32">
                            <div className="text-xs text-slate-500 uppercase font-bold mb-1">答對題數</div>
                            <div className="text-3xl font-black text-slate-800">{score}<span className="text-lg text-slate-400">/{total}</span></div>
                        </div>
                        <div className="bg-slate-50 p-4 rounded-xl border border-slate-200 w-32">
                            <div className="text-xs text-slate-500 uppercase font-bold mb-1">正確率</div>
                            <div className="text-3xl font-black text-slate-800">{percentage}<span className="text-sm text-slate-400">%</span></div>
                        </div>
                    </div>

                    <button 
                        onClick={onRestart}
                        className="w-full bg-brand text-white py-4 rounded-xl font-bold text-lg hover:bg-slate-800 transition-all flex items-center justify-center gap-2 shadow-lg shadow-brand/20"
                    >
                        <Icons.Refresh className="w-5 h-5" />
                        再次挑戰
                    </button>
                </div>
            );
        };

        const App = () => {
            const [currentIndex, setCurrentIndex] = useState(0);
            const [revealed, setRevealed] = useState({}); // { "setIndex-qIndex": boolean }
            const [results, setResults] = useState({}); // { "setIndex-qIndex": boolean (true=correct, false=wrong) }
            const [finished, setFinished] = useState(false);

            const currentSet = problemSets[currentIndex];

            const handleReveal = (qIdx) => {
                setRevealed(prev => ({ ...prev, [`${currentIndex}-${qIdx}`]: true }));
            };

            const handleMark = (qIdx, isCorrect) => {
                setResults(prev => ({ ...prev, [`${currentIndex}-${qIdx}`]: isCorrect }));
            };

            const nextSet = () => {
                if (currentIndex < problemSets.length - 1) {
                    setCurrentIndex(prev => prev + 1);
                    window.scrollTo({ top: 0, behavior: 'smooth' });
                } else {
                    setFinished(true);
                }
            };

            const prevSet = () => {
                if (currentIndex > 0) {
                    setCurrentIndex(prev => prev - 1);
                    window.scrollTo({ top: 0, behavior: 'smooth' });
                }
            };

            const restart = () => {
                setCurrentIndex(0);
                setRevealed({});
                setResults({});
                setFinished(false);
                window.scrollTo({ top: 0, behavior: 'smooth' });
            };

            const renderVisual = (type) => {
                switch(type) {
                    case 'carbon_chart': return <CarbonChart />;
                    case 'delivery_app': return <DeliveryApp />;
                    case 'solar_system': return <SolarSystem />;
                    case 'upload_screen': return <UploadScreen />;
                    case 'drone_warehouse': return <DroneWarehouse />;
                    default: return null;
                }
            };

            const totalScore = Object.values(results).filter(r => r === true).length;
            const totalQuestions = problemSets.reduce((sum, set) => sum + set.questions.length, 0);

            if (finished) {
                return (
                    <div className="min-h-screen flex flex-col items-center justify-center p-4">
                        <ScoreCard score={totalScore} total={totalQuestions} onRestart={restart} />
                        <footer className="mt-12 text-slate-400 text-sm font-medium">
                            由Bob 憶源老師設計
                        </footer>
                    </div>
                );
            }

            return (
                <>
                    <header className="bg-brand text-white p-4 sticky top-0 z-50 shadow-md backdrop-blur-sm bg-opacity-95">
                        <div className="max-w-3xl mx-auto flex justify-between items-center">
                            <div className="flex items-center gap-3">
                                <div className="bg-white/10 p-2 rounded-lg">
                                    <Icons.Brain className="w-6 h-6 text-white" />
                                </div>
                                <div>
                                    <h1 className="font-bold text-lg leading-tight">小小分析師</h1>
                                    <div className="text-[10px] text-white/60 font-medium tracking-wider">PRO CHALLENGE</div>
                                </div>
                            </div>
                            <div className="flex items-center gap-3">
                                <div className="hidden md:block text-xs font-bold bg-white/10 px-3 py-1 rounded-full text-white/80">
                                    高階邏輯
                                </div>
                                <div className="text-sm font-bold bg-accent text-white px-3 py-1 rounded-full shadow-lg shadow-orange-500/20">
                                    得分: {totalScore}
                                </div>
                            </div>
                        </div>
                    </header>

                    <main className="flex-grow max-w-3xl mx-auto w-full p-4 md:p-6 pb-20">
                        {/* Progress Bar */}
                        <div className="flex items-center gap-4 mb-6">
                            <span className="text-xs font-bold text-slate-400 whitespace-nowrap">Level {currentIndex + 1} / {problemSets.length}</span>
                            <div className="h-2 flex-grow bg-slate-200 rounded-full overflow-hidden">
                                <div 
                                    className="h-full bg-brand transition-all duration-500 ease-out" 
                                    style={{ width: `${((currentIndex + 1) / problemSets.length) * 100}%` }}
                                ></div>
                            </div>
                        </div>

                        {/* Story Card */}
                        <div className="bg-white rounded-2xl shadow-sm border border-slate-200 overflow-hidden mb-6 animate-fade-in">
                            <div className="bg-slate-50 p-4 border-b border-slate-100 flex items-center gap-3">
                                <div className="p-2 bg-white rounded-lg shadow-sm border border-slate-100">
                                    {currentSet.icon}
                                </div>
                                <h2 className="font-bold text-lg text-slate-800">{currentSet.title}</h2>
                            </div>
                            <div className="p-5">
                                <div className="text-slate-600 leading-relaxed text-base mb-6 font-medium">
                                    {currentSet.story.split('\n').map((line, i) => (
                                        <p key={i} className="mb-2 last:mb-0">{line}</p>
                                    ))}
                                </div>
                                <VisualContainer title="數據圖表">
                                    {renderVisual(currentSet.visualType)}
                                </VisualContainer>
                            </div>
                        </div>

                        {/* Questions */}
                        <div className="space-y-4">
                            <div className="flex items-center gap-2 mb-2">
                                <span className="bg-accent w-1 h-6 rounded-full"></span>
                                <h3 className="font-bold text-slate-800">挑戰題組</h3>
                            </div>

                            {currentSet.questions.map((q, idx) => {
                                const qKey = `${currentIndex}-${idx}`;
                                const isRevealed = revealed[qKey];
                                const result = results[qKey];

                                return (
                                    <div key={idx} className="bg-white p-5 rounded-xl shadow-sm border border-slate-200 transition-all hover:shadow-md">
                                        <div className="flex gap-4">
                                            <div className={`flex-shrink-0 w-8 h-8 rounded-full flex items-center justify-center font-bold text-sm transition-colors ${
                                                result === true ? 'bg-green-100 text-green-700' :
                                                result === false ? 'bg-red-100 text-red-700' :
                                                'bg-slate-100 text-slate-500'
                                            }`}>
                                                {result === true ? <Icons.Check className="w-5 h-5" /> : 
                                                 result === false ? <Icons.X className="w-5 h-5" /> : 
                                                 idx + 1}
                                            </div>
                                            <div className="flex-grow">
                                                <p className="font-bold text-slate-800 mb-3 text-base">{q.q}</p>
                                                
                                                <input 
                                                    type="text" 
                                                    className="w-full bg-slate-50 border-2 border-slate-100 rounded-lg px-4 py-3 text-slate-700 focus:outline-none focus:border-brand/30 focus:bg-white transition-all placeholder:text-slate-300 mb-4 font-medium"
                                                    placeholder="輸入你的答案..."
                                                />

                                                {!isRevealed ? (
                                                    <button 
                                                        onClick={() => handleReveal(idx)}
                                                        className="text-sm font-bold text-brand bg-brand/5 hover:bg-brand/10 px-4 py-2 rounded-lg transition-colors"
                                                    >
                                                        查看解答
                                                    </button>
                                                ) : (
                                                    <div className="animate-fade-in">
                                                        <div className="bg-slate-50 border border-slate-200 rounded-lg p-3 mb-3 text-sm text-slate-700">
                                                            <span className="font-bold text-brand mr-2">💡 解析：</span>{q.a}
                                                        </div>
                                                        
                                                        {result === undefined && (
                                                            <div className="flex gap-2">
                                                                <button onClick={() => handleMark(idx, true)} className="flex-1 bg-green-500 hover:bg-green-600 text-white py-2 rounded-lg text-sm font-bold transition-colors flex items-center justify-center gap-1">
                                                                    <Icons.Check className="w-4 h-4" /> 答對
                                                                </button>
                                                                <button onClick={() => handleMark(idx, false)} className="flex-1 bg-slate-200 hover:bg-slate-300 text-slate-600 py-2 rounded-lg text-sm font-bold transition-colors flex items-center justify-center gap-1">
                                                                    <Icons.X className="w-4 h-4" /> 答錯
                                                                </button>
                                                            </div>
                                                        )}
                                                    </div>
                                                )}
                                            </div>
                                        </div>
                                    </div>
                                );
                            })}
                        </div>

                        {/* Navigation */}
                        <div className="flex gap-4 mt-8 pt-6 border-t border-slate-200">
                            <button 
                                onClick={prevSet}
                                disabled={currentIndex === 0}
                                className={`flex-1 py-3 rounded-xl font-bold flex items-center justify-center gap-2 transition-all ${
                                    currentIndex === 0 
                                    ? 'text-slate-300 cursor-not-allowed' 
                                    : 'bg-white border border-slate-300 text-slate-600 hover:bg-slate-50'
                                }`}
                            >
                                <Icons.ChevronLeft className="w-5 h-5" /> 上一題
                            </button>
                            <button 
                                onClick={nextSet}
                                className={`flex-1 py-3 rounded-xl font-bold flex items-center justify-center gap-2 text-white shadow-lg shadow-brand/20 transition-all ${
                                    currentIndex === problemSets.length - 1
                                    ? 'bg-green-600 hover:bg-green-700'
                                    : 'bg-brand hover:bg-slate-800'
                                }`}
                            >
                                {currentIndex === problemSets.length - 1 ? '送出成績' : '下一題'} 
                                <Icons.ChevronRight className="w-5 h-5" />
                            </button>
                        </div>
                    </main>

                    <footer className="bg-white border-t border-slate-200 py-6 text-center">
                        <p className="text-slate-400 text-sm font-medium">由Bob 憶源老師設計</p>
                        <p className="text-slate-300 text-xs mt-1">© 2025 Math Challenge Pro</p>
                    </footer>
                </>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
