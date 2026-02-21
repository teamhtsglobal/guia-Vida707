<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Eco Vida Studio 707 - Guía Digital</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- React & ReactDOM -->
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <!-- Babel para JSX -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600;800;900&display=swap');
        body { font-family: 'Inter', sans-serif; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: transparent; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #e2e8f0; border-radius: 10px; }
        .animate-fade-in { animation: fadeIn 0.3s ease-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
    </style>
</head>
<body class="bg-gray-50 text-gray-900">
    <div id="root"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;
        
        // Componente Icon corregido para Lucide CDN
        const Icon = ({ name, size = 24, color = "currentColor", className = "" }) => {
            const iconRef = useRef(null);
            
            useEffect(() => {
                if (iconRef.current && window.lucide) {
                    // Limpiar contenido previo
                    iconRef.current.innerHTML = '';
                    // Crear el elemento con el atributo de lucide
                    const i = document.createElement('i');
                    i.setAttribute('data-lucide', name);
                    iconRef.current.appendChild(i);
                    
                    // Renderizar el icono usando la API de lucide
                    window.lucide.createIcons({
                        icons: window.lucide.icons,
                        attrs: {
                            width: size,
                            height: size,
                            stroke: color,
                            class: className
                        }
                    });
                }
            }, [name, size, color, className]);

            return <span ref={iconRef} className="inline-flex items-center justify-center" />;
        };

        // --- Configuración API Gemini ---
        const apiKey = ""; 

        const callGemini = async (prompt, systemPrompt, language) => {
            let delay = 1000;
            for (let i = 0; i < 5; i++) {
                try {
                    const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            contents: [{ parts: [{ text: prompt }] }],
                            systemInstruction: { parts: [{ text: systemPrompt }] }
                        })
                    });
                    if (!response.ok) throw new Error('API Error');
                    const data = await response.json();
                    return data.candidates?.[0]?.content?.parts?.[0]?.text;
                } catch (error) {
                    if (i === 4) return language === 'es' ? "Error de conexión con la IA." : "AI connection error.";
                    await new Promise(r => setTimeout(r, delay));
                    delay *= 2;
                }
            }
        };

        const App = () => {
            const [currentPage, setCurrentPage] = useState('home');
            const [language, setLanguage] = useState('es');
            const [aiLoading, setAiLoading] = useState(false);
            const [itineraryInput, setItineraryInput] = useState('');
            const [itineraryResult, setItineraryResult] = useState('');
            const [chatInput, setChatInput] = useState('');
            const [chatHistory, setChatHistory] = useState([]);

            const whatsappBaseUrl = "https://api.whatsapp.com/send/?phone=593980464670&type=phone_number&app_absent=0";
            const getWhatsAppLink = (text) => `${whatsappBaseUrl}&text=${encodeURIComponent(text)}`;

            const t = {
                es: {
                    title: "ECO VIDA 707",
                    subtitle: "Guía Digital del Huésped",
                    reserve: "Reservar ahora",
                    waMsg: "Hola, me gustaría reservar un área social en el Edificio Vida.",
                    welcome: "¡Hola! He diseñado este espacio pensando en tu bienestar y comodidad. Espero que disfrutes de las vistas espectaculares al volcán Pichincha y de la energía de Quito. ¡Haz de este loft tu hogar!",
                    sections: {
                        welcome: "Bienvenida", planner: "Planificador ✨", assistant: "Asistente ✨",
                        checkin: "Check-in/out", wifi: "Wi-Fi", rules: "Reglas",
                        shops: "Tiendas", food: "Restaurantes", visit: "Lugares",
                        transport: "Transporte", faq: "Preguntas", review: "Reseñas"
                    },
                    ui: {
                        back: "Volver", ci: "Check-in (3:00 PM)", co: "Check-out (11:00 AM)",
                        coWarning: "DEJA EL TAG CON EL GUARDIA EN RECEPCIÓN.",
                        limits: "Tiempos Máximos", plannerPrompt: "¿Qué te gustaría hacer?",
                        plannerPlaceholder: "Ej: 3 días con niños",
                        assistantIntro: "¡Hola! Pregúntame sobre el Wi-Fi, el check-out o lugares cercanos. ✨",
                        assistantPlaceholder: "Escribe tu duda...",
                        reviewTitle: "¿Cómo estuvo tu estancia?",
                        reviewBtn: "Dejar Reseña en Airbnb"
                    }
                },
                en: {
                    title: "ECO VIDA 707",
                    subtitle: "Guest Digital Guide",
                    reserve: "Reserve Now",
                    waMsg: "Hi, I would like to reserve a social area at Edificio Vida.",
                    welcome: "Hi! I've designed this space with your well-being and comfort in mind. I hope you enjoy the spectacular views of the Pichincha volcano and the energy of Quito. Make this loft your home!",
                    sections: {
                        welcome: "Welcome", planner: "Planner ✨", assistant: "Assistant ✨",
                        checkin: "Check-in/out", wifi: "Wi-Fi", rules: "Rules",
                        shops: "Shops", food: "Restaurants", visit: "Places",
                        transport: "Transport", faq: "FAQ", review: "Reviews"
                    },
                    ui: {
                        back: "Back", ci: "Check-in (3:00 PM)", co: "Check-out (11:00 AM)",
                        coWarning: "LEAVE THE TAG WITH THE GUARD AT RECEPTION.",
                        limits: "Time Limits", plannerPrompt: "What would you like to do?",
                        plannerPlaceholder: "e.g. 3 days with kids",
                        assistantIntro: "Hi! Ask me about Wi-Fi, check-out, or nearby places. ✨",
                        assistantPlaceholder: "Type your question...",
                        reviewTitle: "How was your stay?",
                        reviewBtn: "Leave Airbnb Review"
                    }
                }
            };

            const cur = t[language];

            const shops = [
                { name: "Smart T", link: "https://maps.app.goo.gl/PKghHKgE4aETQHXT9", icon: "smartphone", desc: language === 'es' ? "Tecnología y conveniencia." : "Tech and convenience." },
                { name: "Mall el Jardín", link: "https://maps.app.goo.gl/SgdcinLwBjSxSiCw9", icon: "shopping-bag", desc: language === 'es' ? "Centro comercial principal." : "Main shopping mall." },
                { name: "Tuti República", link: "https://maps.app.goo.gl/FW8uQm4SvfwNTuZ69", icon: "store", desc: language === 'es' ? "Supermercado económico cercano." : "Cheap nearby supermarket." }
            ];

            const restaurants = [
                { name: "Ichiryu Ramen", link: "https://maps.app.goo.gl/LuLHHMREUx3XsywS9", icon: "soup", desc: language === 'es' ? "Excelente ramen tradicional." : "Excellent traditional ramen." },
                { name: "Tradiciones (Wyndham)", link: "https://maps.app.goo.gl/Qz4nWxjBwfv7XDhT7", icon: "utensils", desc: language === 'es' ? "Gastronomía local." : "Local cuisine." },
                { name: "Freshii - La Carolina", link: "https://maps.app.goo.gl/TECWG4F8NNVT5NJX6", icon: "leaf", desc: language === 'es' ? "Opciones saludables." : "Healthy options." }
            ];

            const places = [
                { name: "Jardín Botánico", link: "https://maps.app.goo.gl/Ta2drVGLrJwUVmVa6", icon: "flower-2", desc: language === 'es' ? "Naturaleza en la ciudad." : "Nature in the city." },
                { name: "Parque la Carolina", link: "https://maps.app.goo.gl/CiZzmeJqiCYqzq7t9", icon: "trees", desc: language === 'es' ? "Corazón recreativo." : "Recreational heart." },
                { name: "Metro La Carolina", link: "https://maps.app.goo.gl/R8LEBYfVBdpMzXK39", icon: "train-front", desc: language === 'es' ? "Conexión rápida." : "Fast connection." }
            ];

            const handlePlanner = async () => {
                if (!itineraryInput) return;
                setAiLoading(true);
                const prompt = `Itinerario para: ${itineraryInput}. En Quito, cerca de La Carolina.`;
                const system = `Eres un guía experto en Quito. El usuario se hospeda en Eco Vida Studio 707 (Azuay y Amazonas). Sugiere planes incluyendo el Parque La Carolina, Jardín Botánico y los restaurantes del sector. Responde en ${language === 'es' ? 'Español' : 'Inglés'}. Nunca menciones el nombre Padi.`;
                try {
                    const res = await callGemini(prompt, system, language);
                    setItineraryResult(res);
                } catch (err) {
                    setItineraryResult(language === 'es' ? "Error al generar itinerario." : "Error generating itinerary.");
                } finally {
                    setAiLoading(false);
                }
            };

            const handleAssistant = async () => {
                if (!chatInput) return;
                const msg = chatInput;
                setChatInput('');
                setChatHistory(p => [...p, { role: 'user', text: msg }]);
                setAiLoading(true);
                const system = `Eres el asistente virtual del Studio 707. Datos clave: WiFi (EcoVida_707_HighSpeed / VidaQuito2024*), Check-in 3PM (QR+TAG), Check-out 11AM (dejar TAG en recepción). Áreas sociales: Gym (2h), Piscina (2h), Coworking (3h). Ubicación: Azuay y Amazonas. Responde de forma servicial en el idioma del usuario. Nunca menciones el nombre Padi.`;
                try {
                    const res = await callGemini(msg, system, language);
                    setChatHistory(p => [...p, { role: 'ai', text: res }]);
                } catch (err) {
                    setChatHistory(p => [...p, { role: 'ai', text: language === 'es' ? "Error de conexión." : "Connection error." }]);
                } finally {
                    setAiLoading(false);
                }
            };

            return (
                <div className="max-w-md mx-auto min-h-screen bg-gray-50 flex flex-col">
                    <header className="bg-white p-6 border-b flex justify-between items-center sticky top-0 z-50 shadow-sm">
                        <div>
                            <h1 className="font-black text-xl text-emerald-600 tracking-tighter leading-none">{cur.title}</h1>
                            <p className="text-[10px] text-gray-400 font-bold uppercase tracking-widest">{cur.subtitle}</p>
                        </div>
                        <button onClick={() => setLanguage(l => l === 'es' ? 'en' : 'es')} className="bg-gray-100 text-[10px] font-bold px-3 py-1 rounded-full border border-gray-200">
                            {language === 'es' ? 'EN 🇺🇸' : 'ES 🇪🇨'}
                        </button>
                    </header>

                    <main className="flex-1 p-6 overflow-x-hidden">
                        {currentPage === 'home' ? (
                            <div className="grid grid-cols-2 gap-3 animate-fade-in">
                                {Object.entries(cur.sections).map(([key, label]) => {
                                    const iconMap = { 
                                        welcome: "home", planner: "calendar", assistant: "sparkles", 
                                        checkin: "key", wifi: "wifi", rules: "file-text", 
                                        shops: "shopping-bag", food: "utensils", visit: "camera", 
                                        transport: "car", faq: "help-circle", review: "star" 
                                    };
                                    const colors = { 
                                        welcome: "bg-emerald-50", planner: "bg-emerald-100 border-emerald-200 border", 
                                        assistant: "bg-blue-100 border-blue-200 border", wifi: "bg-indigo-50", 
                                        checkin: "bg-amber-50", rules: "bg-rose-50", shops: "bg-green-50", 
                                        food: "bg-orange-50", visit: "bg-sky-50", transport: "bg-slate-50", 
                                        faq: "bg-purple-50", review: "bg-yellow-50" 
                                    };
                                    return (
                                        <button key={key} onClick={() => setCurrentPage(key)} className={`${colors[key]} p-5 rounded-2xl flex flex-col items-center gap-2 shadow-sm transition-transform active:scale-95`}>
                                            <Icon name={iconMap[key]} size={22} className="text-gray-700" />
                                            <span className="text-[10px] font-black uppercase text-gray-600 tracking-tighter">{label}</span>
                                        </button>
                                    );
                                })}
                            </div>
                        ) : (
                            <div className="animate-fade-in space-y-4">
                                <button onClick={() => setCurrentPage('home')} className="flex items-center gap-2 text-xs font-bold text-gray-400 mb-4 hover:text-gray-600">
                                    <Icon name="chevron-left" size={16} /> {cur.ui.back}
                                </button>
                                
                                {currentPage === 'welcome' && (
                                    <div className="space-y-4">
                                        <div className="rounded-2xl overflow-hidden h-40 bg-gray-200 shadow-md">
                                            <img src="https://images.unsplash.com/photo-1590381105924-c72589b9ef3f?q=80&w=600" className="w-full h-full object-cover" />
                                        </div>
                                        <div className="bg-white p-6 rounded-2xl border shadow-sm">
                                            <p className="italic text-sm text-gray-600 leading-relaxed">"{cur.welcome}"</p>
                                        </div>
                                    </div>
                                )}

                                {currentPage === 'wifi' && (
                                    <div className="bg-white p-8 rounded-2xl border border-indigo-100 text-center shadow-sm">
                                        <Icon name="wifi" size={40} className="text-indigo-500 mb-4" />
                                        <p className="text-[10px] text-gray-400 font-bold uppercase mb-1 tracking-widest">Network / Red</p>
                                        <p className="font-bold text-lg mb-4">EcoVida_707_HighSpeed</p>
                                        <p className="text-[10px] text-gray-400 font-bold uppercase mb-1 tracking-widest">Password / Clave</p>
                                        <p className="font-mono font-bold text-lg text-emerald-600">VidaQuito2024*</p>
                                    </div>
                                )}

                                {currentPage === 'checkin' && (
                                    <div className="space-y-4">
                                        <div className="bg-white p-5 rounded-2xl border-l-4 border-emerald-500 shadow-sm">
                                            <h3 className="font-bold text-emerald-600 mb-2">{cur.ui.ci}</h3>
                                            <ol className="text-xs text-gray-600 space-y-2 list-decimal list-inside">
                                                {cur.es ? [
                                                    "Presenta tu código QR en recepción.",
                                                    "Recibe tu TAG de acceso físico.",
                                                    "El TAG abre el piso 7 y la cerradura de la suite."
                                                ].map((s,i)=><li key={i}>{s}</li>) : [
                                                    "Show your QR code at reception.",
                                                    "Get your physical access TAG.",
                                                    "The TAG opens the 7th floor and the suite lock."
                                                ].map((s,i)=><li key={i}>{s}</li>)}
                                            </ol>
                                        </div>
                                        <div className="bg-rose-50 p-5 rounded-2xl border-l-4 border-rose-500 shadow-sm">
                                            <h3 className="font-bold text-rose-600 mb-1">{cur.ui.co}</h3>
                                            <p className="text-xs font-black text-rose-700 underline mb-2 uppercase">{cur.ui.coWarning}</p>
                                            <p className="text-[10px] text-rose-600">{cur.ui.coSteps}</p>
                                        </div>
                                    </div>
                                )}

                                {currentPage === 'rules' && (
                                    <div className="bg-white p-6 rounded-2xl border shadow-sm">
                                        <h3 className="font-bold text-rose-500 flex items-center gap-2 mb-4"><Icon name="clock" size={18}/> {cur.ui.limits}</h3>
                                        <div className="space-y-2 text-xs">
                                            <div className="flex justify-between p-2 bg-gray-50 rounded-lg"><span>Gym / TRX</span><span className="font-bold">Max 2h</span></div>
                                            <div className="flex justify-between p-2 bg-gray-50 rounded-lg"><span>Pool / Jacuzzi</span><span className="font-bold">Max 2h</span></div>
                                            <div className="flex justify-between p-2 bg-gray-50 rounded-lg"><span>Coworking</span><span className="font-bold">Max 3h</span></div>
                                        </div>
                                        <a href={getWhatsAppLink(cur.waMsg)} target="_blank" className="mt-6 w-full bg-emerald-500 text-white py-3 rounded-xl font-bold flex justify-center gap-2 shadow-lg shadow-emerald-100">
                                            <Icon name="message-circle" size={18} /> {cur.reserve}
                                        </a>
                                    </div>
                                )}

                                {['shops', 'food', 'visit'].includes(currentPage) && (
                                    <div className="space-y-3">
                                        {(currentPage === 'shops' ? shops : currentPage === 'food' ? restaurants : places).map((item, i) => (
                                            <a key={i} href={item.link} target="_blank" className="bg-white p-4 rounded-xl border flex items-center justify-between hover:border-blue-200 transition-colors shadow-sm">
                                                <div className="flex items-center gap-3">
                                                    <div className="p-2 bg-gray-50 rounded-lg"><Icon name={item.icon} size={18} className="text-gray-400" /></div>
                                                    <div>
                                                        <p className="font-bold text-sm">{item.name}</p>
                                                        <p className="text-[10px] text-gray-500">{item.desc}</p>
                                                    </div>
                                                </div>
                                                <Icon name="external-link" size={16} className="text-blue-500" />
                                            </a>
                                        ))}
                                    </div>
                                )}

                                {currentPage === 'planner' && (
                                    <div className="space-y-4">
                                        <div className="bg-white p-5 rounded-2xl border border-emerald-200 shadow-sm">
                                            <h3 className="font-bold text-emerald-700 text-sm mb-3 flex items-center gap-2"><Icon name="calendar" size={18}/> {cur.ui.plannerTitle}</h3>
                                            <div className="flex gap-2">
                                                <input value={itineraryInput} onChange={e=>setItineraryInput(e.target.value)} placeholder={cur.ui.plannerPlaceholder} className="flex-1 p-2 text-xs border rounded-lg outline-none focus:ring-1 focus:ring-emerald-500" />
                                                <button onClick={handlePlanner} disabled={aiLoading} className="bg-emerald-600 text-white p-2 rounded-lg hover:bg-emerald-700 disabled:opacity-50">
                                                    {aiLoading ? <Icon name="loader-2" className="animate-spin" /> : <Icon name="sparkles" />}
                                                </button>
                                            </div>
                                            {itineraryResult && <div className="mt-4 text-[11px] text-gray-600 leading-relaxed bg-emerald-50 p-4 rounded-xl border border-emerald-100 italic whitespace-pre-line">{itineraryResult}</div>}
                                        </div>
                                    </div>
                                )}

                                {currentPage === 'assistant' && (
                                    <div className="flex flex-col h-[60vh]">
                                        <div className="flex-1 overflow-y-auto space-y-3 pr-1 custom-scrollbar pb-4">
                                            <div className="bg-blue-50 p-3 rounded-2xl text-[11px] italic border border-blue-100 text-blue-800">
                                                {cur.ui.assistantIntro}
                                            </div>
                                            {chatHistory.map((m, i) => (
                                                <div key={i} className={`flex ${m.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                                                    <div className={`max-w-[85%] p-3 rounded-2xl text-[11px] shadow-sm ${m.role === 'user' ? 'bg-blue-600 text-white rounded-tr-none' : 'bg-white border border-gray-100 text-gray-700 rounded-tl-none'}`}>
                                                        {m.text}
                                                    </div>
                                                </div>
                                            ))}
                                            {aiLoading && <div className="flex justify-center"><Icon name="loader-2" size={20} className="animate-spin text-blue-500" /></div>}
                                        </div>
                                        <div className="mt-2 flex gap-2 bg-white p-2 border rounded-xl shadow-lg">
                                            <input value={chatInput} onChange={e=>setChatInput(e.target.value)} onKeyDown={e=>e.key==='Enter'&&handleAssistant()} className="flex-1 text-xs outline-none p-2" placeholder={cur.ui.assistantPlaceholder} />
                                            <button onClick={handleAssistant} className="bg-blue-600 text-white p-2 rounded-lg hover:bg-blue-700"><Icon name="send" size={16} /></button>
                                        </div>
                                    </div>
                                )}

                                {currentPage === 'review' && (
                                    <div className="text-center py-10 animate-fade-in">
                                        <Icon name="star" size={48} className="text-yellow-400 mx-auto mb-4" />
                                        <h3 className="font-bold text-lg mb-6 tracking-tight">{cur.ui.reviewTitle}</h3>
                                        <a href="https://airbnb.com/h/vida707" target="_blank" className="inline-block bg-rose-500 text-white px-8 py-4 rounded-2xl font-bold shadow-xl shadow-rose-100 uppercase text-xs tracking-widest hover:bg-rose-600 transition-colors">
                                            {cur.ui.reviewBtn}
                                        </a>
                                    </div>
                                )}
                            </div>
                        )}
                    </main>

                    {currentPage === 'home' && (
                        <div className="fixed bottom-6 right-6 flex flex-col gap-3">
                            <button onClick={() => setCurrentPage('assistant')} className="w-14 h-14 bg-blue-600 text-white rounded-full flex items-center justify-center shadow-xl transition-transform active:scale-90 hover:bg-blue-700">
                                <Icon name="sparkles" size={24} />
                            </button>
                            <a href={getWhatsAppLink(cur.waMsg)} target="_blank" className="w-14 h-14 bg-emerald-500 text-white rounded-full flex items-center justify-center shadow-xl transition-transform active:scale-90 hover:bg-emerald-600">
                                <Icon name="message-circle" size={24} />
                            </a>
                        </div>
                    )}
                    
                    <footer className="p-8 text-center text-gray-300 text-[8px] font-bold uppercase tracking-[0.4em]">
                        EcoVida • Vida707 2026
                    </footer>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
