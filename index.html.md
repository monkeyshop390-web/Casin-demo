<!doctype html>  
<html lang="es" class="dark">  
<head>  
  <meta charset="UTF-8" />  
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no" />  
  <title>CASIN PREMIUM — Demo Móvil & Web</title>  
  <link rel="preconnect" href="https://fonts.googleapis.com" />  
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />  
  <link href="https://fonts.googleapis.com/css2?family=Cinzel:wght@600;700;800&family=JetBrains+Mono:wght@500;700&family=Plus+Jakarta+Sans:wght@400;500;600;700&display=swap" rel="stylesheet" />  
  <script src="https://cdn.tailwindcss.com"></script>  
  <script>  
    tailwind.config = {  
      theme: {  
        extend: {  
          fontFamily: { display: ["Cinzel", "serif"], sans: ["Plus Jakarta Sans", "sans-serif"], mono: ["JetBrains Mono", "monospace"] },  
          boxShadow: { gold: "0 0 24px rgba(212,175,55,0.28)", violet: "0 0 32px rgba(124,58,237,0.35)" }  
        }  
      }  
    };  
  </script>  
  <style>  
    body {  
      font-family: "Plus Jakarta Sans", system-ui, sans-serif;  
      background: #06040a radial-gradient(circle at 15% 10%, rgba(124,58,237,0.18), transparent 45%) fixed;  
      overflow-x: hidden; -webkit-tap-highlight-color: transparent;  
    }  
    .perspective-1000 { perspective: 1000px; }  
    .preserve-3d { transform-style: preserve-3d; }  
    .backface-hidden { backface-visibility: hidden; -webkit-backface-visibility: hidden; }  
    .rotate-y-180 { transform: rotateY(180deg); }  
    @keyframes reelBlurSpin { 0% { transform: translateY(0); } 100% { transform: translateY(-50%); } }  
    .animate-reel-spin { animation: reelBlurSpin 0.18s linear infinite; }  
    @keyframes cardDealIn { 0% { opacity: 0; transform: translateY(-18px) scale(0.9); } 100% { opacity: 1; transform: translateY(0) scale(1); } }  
    .animate-card-deal { animation: cardDealIn 0.3s cubic-bezier(0.22,1,0.36,1) forwards; }  
    @keyframes shimmerGold { 0% { background-position: -200% 0; } 100% { background-position: 200% 0; } }  
    .gold-shimmer {  
      background: linear-gradient(90deg, #b8891f 0%, #f9e076 35%, #d4af37 65%, #f9e076 85%, #b8891f 100%);  
      background-size: 200% auto; -webkit-background-clip: text; background-clip: text; color: transparent;  
      animation: shimmerGold 6s linear infinite;  
    }  
  </style>  
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>  
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>  
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>  
</head>  
<body class="text-slate-100 antialiased">  
  <div id="root"></div>  
  <script type="text/babel">  
    const { useState, useEffect, useRef, useCallback } = React;  
  
    /* === ALMACENAMIENTO SEGURO === */  
    const memStore = {};  
    const safeStorage = {  
      get(k) { try { return localStorage.getItem(k); } catch { return memStore[k] || null; } },  
      set(k, v) { try { localStorage.setItem(k, v); } catch { memStore[k] = v; } }  
    };  
  
    /* === AUDIO WEB API === */  
    const soundFX = {  
      ctx: null, enabled: true,  
      getCtx() {  
        if (!this.enabled) return null;  
        try {  
          if (!this.ctx) {  
            const C = window.AudioContext || window.webkitAudioContext;  
            if (C) this.ctx = new C();  
          }  
          if (this.ctx && this.ctx.state === "suspended") this.ctx.resume().catch(()=>{});  
          return this.ctx;  
        } catch { return null; }  
      },  
      tone(f, d, type="sine", g=0.06, delay=0) {  
        const ctx = this.getCtx(); if (!ctx) return;  
        try {  
          const now = ctx.currentTime + delay, osc = ctx.createOscillator(), gain = ctx.createGain();  
          osc.type = type; osc.frequency.setValueAtTime(f, now);  
          gain.gain.setValueAtTime(0.001, now);  
          gain.gain.exponentialRampToValueAtTime(g, now + 0.015);  
          gain.gain.exponentialRampToValueAtTime(0.001, now + d);  
          osc.connect(gain); gain.connect(ctx.destination);  
          osc.start(now); osc.stop(now + d + 0.02);  
        } catch {}  
      },  
      click() { this.tone(520, 0.05, "triangle", 0.05); },  
      chip() { this.tone(1180, 0.06, "sine", 0.05); this.tone(1560, 0.04, "triangle", 0.04, 0.02); },  
      card() { this.tone(320, 0.07, "triangle", 0.05); },  
      tick() { this.tone(240, 0.04, "square", 0.025); },  
      wheel() { for (let i=0;i<7;i++) this.tone(420+i*25, 0.04, "triangle", 0.03, i*0.09); },  
      win() { [523,659,784,1046].forEach((n,i)=>this.tone(n, 0.2, "triangle", 0.07, i*0.08)); },  
      lose() { this.tone(260, 0.15, "sawtooth", 0.03); this.tone(200, 0.2, "sawtooth", 0.025, 0.11); }  
    };  
  
    /* === DATOS DEMO === */  
    const AVATARS = [  
      { id: "crown", label: "Soberano" }, { id: "prism", label: "Prisma" },  
      { id: "eclipse", label: "Eclipse" }, { id: "aegis", label: "Égida" },  
      { id: "nova", label: "Nova" }, { id: "oracle", label: "Oráculo" }  
    ];  
    const GAMES = [  
      { id: "cp-slots", title: "CP SLOTS", sub: "Aethelgard Vault 5×3", cat: "Tragamonedas Original", desc: "5 rodillos, símbolos imperiales SVG, 5 líneas iluminadas y tabla de pagos.", featured: true, popular: true, min: 25 },  
      { id: "cp-roulette", title: "CP ROULETTE", sub: "Ruleta Europea", cat: "Mesa Clásica", desc: "Rueda europea de 37 casillas, bola animada y paño completo de apuestas.", featured: true, popular: true, min: 10 },  
      { id: "cp-blackjack", title: "CP BLACKJACK", sub: "Mesa VIP 21", cat: "Cartas Estratégicas", desc: "6 barajas, As suave/duro automático, doblar y Blackjack natural 3:2.", featured: true, popular: false, min: 25 },  
      { id: "cp-baccarat", title: "CP BACCARAT", sub: "Punto & Banca", cat: "Cartas de Élite", desc: "Reglas oficiales de tercera carta, cálculo módulo 10 y tablero histórico.", featured: false, popular: true, min: 25 }  
    ];  
    const LEADERBOARD = [  
      { id: "p1", name: "Valeria Vance", avatar: "crown", points: 48650, winRate: 58, fav: "CP BLACKJACK" },  
      { id: "p2", name: "Mateo Sterling", avatar: "eclipse", points: 41200, winRate: 54, fav: "CP ROULETTE" },  
      { id: "p3", name: "Kaelen Voss", avatar: "prism", points: 36900, winRate: 52, fav: "CP SLOTS" },  
      { id: "p4", name: "Lucía Mendoza", avatar: "nova", points: 29450, winRate: 51, fav: "CP BACCARAT" },  
      { id: "p5", name: "Dante Alighieri", avatar: "aegis", points: 22800, winRate: 49, fav: "CP ROULETTE" }  
    ];  
    const NEWS = [  
      { id: "n1", date: "Nov 2026", tag: "ESTUDIO", title: "Colección CASIN PREMIUM ORIGINALS", desc: "Cuatro motores propios programados desde cero con arte vectorial y sonido en tiempo real.", hl: "4 Juegos Activos" },  
      { id: "n2", date: "Motor 3D", tag: "RULETA", title: "Sincronía angular de bola", desc: "Distribución europea oficial de 37 casillas con caída precisa en el número ganador.", hl: "Física 60fps" },  
      { id: "n3", date: "Demo", tag: "SEGURIDAD", title: "Entorno 100% Simulado", desc: "Puntos ficticios sin valor económico, sin depósitos ni pasarelas financieras.", hl: "Modo Demo Certificado" }  
    ];  
    const PROMOS = [  
      { id: "pr1", title: "Bono Cortesía VIP", badge: "BIENVENIDA DEMO", desc: "Créditos ficticios extra para probar las 4 mesas.", pts: 2500, hrs: 24 },  
      { id: "pr2", title: "Recarga Nocturna", badge: "ENTRETENIMIENTO", desc: "Ideal para sesiones largas entre amigos.", pts: 1200, hrs: 6 },  
      { id: "pr3", title: "Impulso Aethelgard", badge: "SLOTS DEMO", desc: "Puntos virtuales extra para los rodillos.", pts: 800, hrs: 3 }  
    ];  
  
    /* === MOTORES PUROS DE JUEGO === */  
    // SLOTS  
    const SYMS = [  
      { id: "crown", name: "Corona", w: 4, p: {3:8,4:24,5:60} },  
      { id: "diamond", name: "Diamante", w: 6, p: {3:5,4:15,5:38} },  
      { id: "scepter", name: "Cetro", w: 8, p: {3:4,4:10,5:25} },  
      { id: "ring", name: "Anillo", w: 10, p: {3:3,4:8,5:18} },  
      { id: "coin", name: "Medalla CP", w: 12, p: {3:2.5,4:6,5:14} },  
      { id: "amber", name: "Ámbar", w: 15, p: {3:2,4:4,5:10} },  
      { id: "clover", name: "Estrella", w: 18, p: {3:1.5,4:3,5:7} }  
    ];  
    const PAYLINES = [  
      { id: 1, name: "Central", rows: [1,1,1,1,1], color: "#F9E076" },  
      { id: 2, name: "Superior", rows: [0,0,0,0,0], color: "#A78BFA" },  
      { id: 3, name: "Inferior", rows: [2,2,2,2,2], color: "#34D399" },  
      { id: 4, name: "Corona V", rows: [0,1,2,1,0], color: "#F472B6" },  
      { id: 5, name: "Pirámide", rows: [2,1,0,1,2], color: "#38BDF8" }  
    ];  
    function runSlots(stake) {  
      const totW = SYMS.reduce((a,s)=>a+s.w,0);  
      const pick = () => { let r=Math.random()*totW; for (const s of SYMS){ r-=s.w; if(r<=0) return s.id; } return "clover"; };  
      const grid = Array.from({length:5}, ()=>Array.from({length:3}, pick));  
      const wins = [];  
      for (const l of PAYLINES) {  
        const first = grid[0][l.rows[0]];  
        let c = 1;  
        for (let col=1; col<5; col++) { if (grid[col][l.rows[col]] === first) c++; else break; }  
        if (c >= 3) {  
          const def = SYMS.find(s=>s.id===first);  
          const winPts = Math.max(1, Math.round((stake/5) * def.p[c] * 2.25));  
          wins.push({ ...l, sym: first, count: c, winPts, coords: Array.from({length:c}, (_,i)=>[i, l.rows[i]]) });  
        }  
      }  
      return { grid, wins, totalWin: wins.reduce((s,w)=>s+w.winPts, 0) };  
    }  
  
    // ROULETTE  
    const WHEEL = [0,32,15,19,4,21,2,25,17,34,6,27,13,36,11,30,8,23,10,5,24,16,33,1,20,14,31,9,22,18,29,7,28,12,35,3,26];  
    const REDS = new Set([1,3,5,7,9,12,14,16,18,19,21,23,25,27,30,32,34,36]);  
    const rColor = (n) => n === 0 ? "green" : REDS.has(n) ? "red" : "black";  
    function runRoulette(bets) {  
      const idx = Math.floor(Math.random() * WHEEL.length), num = WHEEL[idx], col = rColor(num);  
      let pay = 0;  
      for (const b of bets) {  
        let m = 0;  
        if (b.type==="straight" && b.num===num) m=36;  
        else if (b.type==="red" && col==="red") m=2;  
        else if (b.type==="black" && col==="black") m=2;  
        else if (b.type==="even" && num!==0 && num%2===0) m=2;  
        else if (b.type==="odd" && num!==0 && num%2===1) m=2;  
        else if (b.type==="low" && num>=1 && num<=18) m=2;  
        else if (b.type==="high" && num>=19 && num<=36) m=2;  
        else if (b.type==="d1" && num>=1 && num<=12) m=3;  
        else if (b.type==="d2" && num>=13 && num<=24) m=3;  
        else if (b.type==="d3" && num>=25 && num<=36) m=3;  
        else if (b.type==="c1" && num!==0 && num%3===1) m=3;  
        else if (b.type==="c2" && num!==0 && num%3===2) m=3;  
        else if (b.type==="c3" && num!==0 && num%3===0) m=3;  
        pay += b.amt * m;  
      }  
      return { num, idx, col, stake: bets.reduce((s,b)=>s+b.amt,0), pay };  
    }  
  
    // BLACKJACK & BACCARAT SHOE  
    const SUITS = ["spades","hearts","diamonds","clubs"];  
    const RANKS = ["A","2","3","4","5","6","7","8","9","10","J","Q","K"];  
    function makeShoe(decks=6) {  
      const s=[]; let k=0;  
      for (let d=0;d<decks;d++) for (const suit of SUITS) for (const rank of RANKS) s.push({id:`${d}-${suit}-${rank}-${k++}`,suit,rank});  
      for (let i=s.length-1;i>0;i--) { const j=Math.floor(Math.random()*(i+1)); [s[i],s[j]]=[s[j],s[i]]; }  
      return s;  
    }  
    function evalBJ(cards) {  
      let tot=0, aces=0;  
      for (const c of cards) {  
        if (c.rank==="A") { aces++; tot+=11; }  
        else if (["K","Q","J","10"].includes(c.rank)) tot+=10;  
        else tot+=parseInt(c.rank,10);  
      }  
      while (tot>21 && aces>0) { tot-=10; aces--; }  
      return { tot, soft: aces>0, bj: cards.length===2 && tot===21, bust: tot>21 };  
    }  
    function resBJ(pC, dC, bet) {  
      const p = evalBJ(pC), d = evalBJ(dC);  
      if (p.bust) return { out:"bust", pay:0, msg:"Te pasaste de 21. Gana la banca." };  
      if (p.bj && !d.bj) return { out:"bj", pay:Math.round(bet*2.5), msg:"¡BLACKJACK NATURAL! Pago 3:2." };  
      if (d.bj && !p.bj) return { out:"lose", pay:0, msg:"El repartidor tiene Blackjack." };  
      if (p.bj && d.bj) return { out:"push", pay:bet, msg:"Doble Blackjack. Empate." };  
      if (d.bust) return { out:"win", pay:bet*2, msg:"Repartidor se pasó de 21. ¡Ganaste!" };  
      if (p.tot > d.tot) return { out:"win", pay:bet*2, msg:`Tu ${p.tot} vence al ${d.tot} del repartidor.` };  
      if (p.tot < d.tot) return { out:"lose", pay:0, msg:`Repartidor gana (${d.tot} vs ${p.tot}).` };  
      return { out:"push", pay:bet, msg:`Empate en ${p.tot}. Recuperas puntos.` };  
    }  
    const bacVal = (c) => c.rank==="A"?1 : ["10","J","Q","K"].includes(c.rank)?0 : parseInt(c.rank,10);  
    const bacTot = (cs) => cs.reduce((s,c)=>s+bacVal(c),0)%10;  
    function runBaccarat(shoeIn, bets) {  
      const shoe = shoeIn.length<12 ? makeShoe(8) : [...shoeIn];  
      const draw = () => shoe.pop();  
      const pC=[draw(),draw()], bC=[draw(),draw()];  
      let pT=bacTot(pC), bT=bacTot(bC), note="";  
      if (pT>=8 || bT>=8) {  
        note = `Mano Natural (${pT>=8?"Jugador "+pT:"Banca "+bT}). Ambos se plantan.`;  
      } else {  
        let p3=null;  
        if (pT<=5) { const c=draw(); pC.push(c); p3=bacVal(c); pT=bacTot(pC); note=`Jugador pidió (${c.rank}). `; }  
        else note="Jugador se plantó. ";  
        let bDraw = false;  
        if (p3===null) bDraw = bT<=5;  
        else if (bT<=2) bDraw=true;  
        else if (bT===3) bDraw=p3!==8;  
        else if (bT===4) bDraw=p3>=2&&p3<=7;  
        else if (bT===5) bDraw=p3>=4&&p3<=7;  
        else if (bT===6) bDraw=p3===6||p3===7;  
        if (bDraw) { const c=draw(); bC.push(c); bT=bacTot(bC); note+=`Banca pidió (${c.rank}).`; }  
        else note+="Banca se plantó.";  
      }  
      const win = pT>bT?"player":bT>pT?"banker":"tie";  
      const stake = bets.player+bets.banker+bets.tie;  
      let pay = 0;  
      if (win==="player") pay=bets.player*2;  
      else if (win==="banker") pay=Math.round(bets.banker*1.95);  
      else pay=bets.tie*9+bets.player+bets.banker;  
      return { pC, bC, pT, bT, win, note, stake, pay, shoe };  
    }  
  
    /* === COMPONENTES VISUALES SVG === */  
    const BrandLogo = ({ compact=false }) => (  
      <div className="flex items-center gap-2.5 select-none">  
        <svg width={compact?34:42} height={compact?34:42} viewBox="0 0 64 64" fill="none" className="shrink-0">  
          <defs>  
            <linearGradient id="gG" x1="0" y1="0" x2="64" y2="64"><stop offset="0%" stopColor="#F9E076"/><stop offset="50%" stopColor="#D4AF37"/><stop offset="100%" stopColor="#8C6512"/></linearGradient>  
            <linearGradient id="gV" x1="8" y1="8" x2="56" y2="56"><stop offset="0%" stopColor="#4C1D95"/><stop offset="100%" stopColor="#12091F"/></linearGradient>  
          </defs>  
          <polygon points="20,4 44,4 60,20 60,44 44,60 20,60 4,44 4,20" fill="url(#gV)" stroke="url(#gG)" strokeWidth="2.5"/>  
          <path d="M23 19L28 14L32 18L36 14L41 19H23Z" fill="url(#gG)"/>  
          <path d="M29 25C22.5 25 18 29.5 18 35.5C18 41.5 22.5 46 29 46C31.5 46 33.4 45.3 35 44" stroke="url(#gG)" strokeWidth="3.2" strokeLinecap="round"/>  
          <path d="M34 45V25H41.5C45 25 47.5 27.2 47.5 30.8C47.5 34.4 45 36.6 41.5 36.6H34" stroke="#F8FAFC" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round"/>  
        </svg>  
        {!compact && (  
          <div className="leading-none">  
            <div className="font-display text-base sm:text-lg tracking-[0.18em] font-bold gold-shimmer">CASIN PREMIUM</div>  
            <div className="text-[9px] tracking-[0.28em] text-violet-300/80 uppercase mt-1">Originals Studio</div>  
          </div>  
        )}  
      </div>  
    );  
  
    const AvatarIcon = ({ avatar, size=36 }) => (  
      <svg width={size} height={size} viewBox="0 0 48 48" fill="none" className="shrink-0">  
        <circle cx="24" cy="24" r="22" fill="#230A42" stroke="#D4AF37" strokeWidth="2"/>  
        {avatar==="crown" && <path d="M13 31L15 18L21 23L24 15L27 23L33 18L35 31H13Z" fill="#F5D061"/>}  
        {avatar==="prism" && <polygon points="24,10 37,24 24,38 11,24" stroke="#F5D061" strokeWidth="2.5" fill="#6D28D9" fillOpacity="0.4"/>}  
        {avatar==="eclipse" && <><circle cx="23" cy="24" r="10" fill="#F5D061"/><circle cx="27" cy="22" r="8.5" fill="#230A42"/></>}  
        {avatar==="aegis" && <path d="M24 12L35 16V25C35 31 29.5 35.5 24 37C18.5 35.5 13 31 13 25V16L24 12Z" stroke="#F5D061" strokeWidth="2.5" fill="#4C1D95"/>}  
        {avatar==="nova" && <path d="M24 10L27 21L38 24L27 27L24 38L21 27L10 24L21 21L24 10Z" fill="#F5D061"/>}  
        {avatar==="oracle" && <><path d="M10 24C14 17 34 17 38 24C34 31 14 31 10 24Z" stroke="#F5D061" strokeWidth="2.2"/><circle cx="24" cy="24" r="4" fill="#F5D061"/></>}  
      </svg>  
    );  
  
    const SlotSym = ({ id, size=40 }) => {  
      if (id==="crown") return <svg width={size} height={size} viewBox="0 0 64 64"><path d="M12 46L10 22L22 30L32 14L42 30L54 22L52 46H12Z" fill="#F5D061" stroke="#FEF9C3" strokeWidth="1.5"/><rect x="12" y="48" width="40" height="5" rx="2" fill="#D4AF37"/></svg>;  
      if (id==="diamond") return <svg width={size} height={size} viewBox="0 0 64 64"><polygon points="20,14 44,14 56,28 32,54 8,28" fill="#7C3AED" stroke="#DDD6FE" strokeWidth="2"/><polygon points="25,28 39,28 32,54" fill="#A78BFA"/></svg>;  
      if (id==="scepter") return <svg width={size} height={size} viewBox="0 0 64 64"><line x1="16" y1="50" x2="42" y2="24" stroke="#D4AF37" strokeWidth="5" strokeLinecap="round"/><circle cx="45" cy="21" r="9" fill="#5B21B6" stroke="#FDE68A" strokeWidth="2.5"/></svg>;  
      if (id==="ring") return <svg width={size} height={size} viewBox="0 0 64 64"><circle cx="32" cy="36" r="14" fill="none" stroke="#F5D061" strokeWidth="6"/><polygon points="32,12 40,21 32,26 24,21" fill="#34D399"/></svg>;  
      if (id==="coin") return <svg width={size} height={size} viewBox="0 0 64 64"><circle cx="32" cy="32" r="20" fill="#B45309" stroke="#FDE68A" strokeWidth="3"/><text x="32" y="37" textAnchor="middle" fill="#FEF08A" fontSize="14" fontWeight="800">CP</text></svg>;  
      if (id==="amber") return <svg width={size} height={size} viewBox="0 0 64 64"><polygon points="32,10 50,24 44,50 20,50 14,24" fill="#F59E0B" stroke="#FDE68A" strokeWidth="2"/></svg>;  
      return <svg width={size} height={size} viewBox="0 0 64 64"><path d="M32 10L37 25L52 25L40 34L45 49L32 40L19 49L24 34L12 25L27 25L32 10Z" fill="#38BDF8" stroke="#BAE6FD" strokeWidth="2"/></svg>;  
    };  
  
    const PlayingCard = ({ card, faceDown=false, small=false }) => {  
      const red = card && (card.suit==="hearts" || card.suit==="diamonds");  
      const sym = card ? ({spades:"♠",hearts:"♥",diamonds:"♦",clubs:"♣"}[card.suit]) : "";  
      const dims = small ? "w-14 h-20" : "w-16 h-24 sm:w-22 sm:h-32";  
      return (  
        <div className={`${dims} perspective-1000 animate-card-deal shrink-0`}>  
          <div className={`relative w-full h-full transition-transform duration-500 preserve-3d ${faceDown?"rotate-y-180":""}`}>  
            <div className="absolute inset-0 backface-hidden rounded-xl bg-gradient-to-br from-[#FFFDF7] to-[#EDE7DA] border border-amber-300 shadow-lg flex flex-col justify-between p-1.5 select-none">  
              {card && <>  
                <div className={`leading-none font-bold font-mono text-xs ${red?"text-rose-700":"text-slate-900"}`}>{card.rank}<div className="text-[10px]">{sym}</div></div>  
                <div className={`text-center text-xl sm:text-2xl ${red?"text-rose-600":"text-slate-900"}`}>{sym}</div>  
                <div className={`leading-none font-bold font-mono text-xs self-end rotate-180 ${red?"text-rose-700":"text-slate-900"}`}>{card.rank}</div>  
              </>}  
            </div>  
            <div className="absolute inset-0 backface-hidden rotate-y-180 rounded-xl bg-gradient-to-br from-[#1C0B36] to-[#0B0614] border-2 border-amber-400/70 p-1 flex items-center justify-center">  
              <div className="w-full h-full rounded-lg border border-amber-400/40 flex items-center justify-center font-display text-[9px] text-amber-200">CP</div>  
            </div>  
          </div>  
        </div>  
      );  
    };  
  
    const GameArt = ({ id }) => (  
      <div className="relative h-36 bg-[#0B0615] overflow-hidden rounded-t-2xl">  
        <svg viewBox="0 0 400 180" className="w-full h-full object-cover">  
          <rect width="400" height="180" fill={id==="cp-blackjack"?"#052E24":id==="cp-baccarat"?"#2A1205":"#160B2E"}/>  
          {id==="cp-slots" && <><rect x="75" y="34" width="70" height="110" rx="8" fill="#0E071B" stroke="#D4AF37"/><rect x="165" y="26" width="70" height="126" rx="8" fill="#1C0F38" stroke="#F5D061" strokeWidth="2"/><rect x="255" y="34" width="70" height="110" rx="8" fill="#0E071B" stroke="#D4AF37"/><path d="M180 105L177 75L192 85L200 65L208 85L223 75L220 105H180Z" fill="#F5D061"/></>}  
          {id==="cp-roulette" && <><circle cx="200" cy="90" r="64" fill="#180D2C" stroke="#D4AF37" strokeWidth="3"/><circle cx="200" cy="90" r="46" fill="none" stroke="#B91C1C" strokeWidth="12" strokeDasharray="10 10"/><circle cx="200" cy="90" r="10" fill="#F9E076"/><circle cx="248" cy="60" r="5" fill="#fff"/></>}  
          {id==="cp-blackjack" && <><rect x="135" y="32" width="60" height="92" rx="8" fill="#FAF5EB" stroke="#D4AF37" transform="rotate(-8 165 78)"/><rect x="195" y="28" width="60" height="92" rx="8" fill="#FAF5EB" stroke="#D4AF37" transform="rotate(8 225 74)"/><text x="155" y="88" fill="#111" fontSize="24" fontWeight="800">21</text></>}  
          {id==="cp-baccarat" && <><rect x="120" y="36" width="60" height="90" rx="8" fill="#1B0E33" stroke="#F5D061" strokeWidth="2"/><rect x="220" y="36" width="60" height="90" rx="8" fill="#FAF5EB" stroke="#D4AF37" strokeWidth="2"/><text x="250" y="90" textAnchor="middle" fill="#7C3AED" fontSize="26" fontWeight="800">9</text></>}  
        </svg>  
        <div className="absolute inset-0 bg-gradient-to-t from-[#120B20] via-transparent to-transparent"/>  
      </div>  
    );  
  
    const Btn = ({ variant="gold", className="", onClick, disabled, children, type="button" }) => {  
      const s = {  
        gold: "bg-gradient-to-r from-[#B8891F] via-[#F4D06F] to-[#C89626] text-slate-950 font-bold shadow-gold",  
        violet: "bg-gradient-to-r from-violet-700 to-purple-600 text-white font-semibold border border-violet-400/30 shadow-violet",  
        ghost: "bg-white/5 text-slate-200 border border-white/15"  
      }[variant];  
      return (  
        <button type={type} disabled={disabled} onClick={(e)=>{ if(!disabled) soundFX.click(); onClick && onClick(e); }}  
          className={`px-4 py-2.5 rounded-xl text-xs sm:text-sm transition active:scale-95 disabled:opacity-40 ${s} ${className}`}>  
          {children}  
        </button>  
      );  
    };  
  
    const Chips = ({ vals, sel, onSel, disabled }) => (  
      <div className="flex flex-wrap gap-2">  
        {vals.map(v=>(  
          <button key={v} type="button" disabled={disabled} onClick={()=>{ soundFX.chip(); onSel(v); }}  
            className={`w-11 h-11 rounded-full font-mono text-[11px] font-bold border-2 border-dashed transition ${sel===v?"bg-amber-400 text-slate-950 border-white scale-110 shadow-gold":"bg-violet-950 text-amber-200 border-amber-400/50"}`}>  
            {v}  
          </button>  
        ))}  
      </div>  
    );  
  
    /* === JUEGO 1: CP SLOTS === */  
    const CpSlots = ({ points, onBack, onRecord }) => {  
      const [stake, setStake] = useState(50);  
      const [grid, setGrid] = useState(()=>runSlots(50).grid);  
      const [spins, setSpins] = useState([false,false,false,false,false]);  
      const [res, setRes] = useState(null);  
      const [hist, setHist] = useState([]);  
      const [paytable, setPaytable] = useState(false);  
      const busy = spins.some(Boolean);  
  
      const spin = () => {  
        if (busy || points < stake) return;  
        setRes(null); setSpins([true,true,true,true,true]);  
        const r = runSlots(stake);  
        [550,800,1050,1300,1550].forEach((ms, col) => {  
          setTimeout(() => {  
            soundFX.tick();  
            setGrid(p => { const n=[...p]; n[col]=r.grid[col]; return n; });  
            setSpins(p => { const n=[...p]; n[col]=false; return n; });  
            if (col === 4) {  
              setRes(r);  
              if (r.totalWin > 0) soundFX.win(); else soundFX.lose();  
              setHist(h => [{ stake, win: r.totalWin }, ...h.slice(0,5)]);  
              onRecord({ gameId:"cp-slots", gameTitle:"CP SLOTS", stake, payout:r.totalWin, net:r.totalWin-stake,  
                summary: r.wins.length ? `${r.wins.length} línea(s) · +${r.totalWin} pts` : "Sin premio" });  
            }  
          }, ms);  
        });  
      };  
      const hits = new Set(res?.wins.flatMap(w=>w.coords.map(([c,r])=>`${c}-${r}`)) || []);  
  
      return (  
        <div className="space-y-4">  
          <div className="flex flex-wrap items-center justify-between gap-2">  
            <button onClick={onBack} className="text-xs text-violet-300">← Catálogo</button>  
            <div className="flex gap-2">  
              <Btn variant="ghost" onClick={()=>setPaytable(v=>!v)}>{paytable?"Ocultar Tabla":"Tabla Pagos"}</Btn>  
              <Btn variant="ghost" disabled={busy} onClick={()=>{ setRes(null); setGrid(runSlots(stake).grid); }}>Nueva</Btn>  
            </div>  
          </div>  
          <h2 className="font-display text-xl sm:text-2xl font-bold gold-shimmer">CP SLOTS · Aethelgard Vault</h2>  
  
          {paytable && (  
            <div className="rounded-2xl bg-[#130C22] border border-white/10 p-4 grid grid-cols-2 sm:grid-cols-4 gap-2">  
              {SYMS.map(s=>(  
                <div key={s.id} className="p-2 rounded-xl bg-white/5 flex flex-col items-center text-center">  
                  <SlotSym id={s.id} size={30}/>  
                  <span className="text-[11px] font-semibold mt-1">{s.name}</span>  
                  <span className="text-[10px] font-mono text-amber-300">3×:{s.p[3]} 4×:{s.p[4]} 5×:{s.p[5]}</span>  
                </div>  
              ))}  
            </div>  
          )}  
  
          <div className="rounded-2xl bg-[#120B20] border border-amber-400/25 p-4 space-y-4">  
            <div className="relative rounded-xl bg-[#07040D] border border-amber-500/40 p-2 sm:p-3">  
              <div className="grid grid-cols-5 gap-1.5 sm:gap-2.5">  
                {grid.map((col, ci) => (  
                  <div key={ci} className="rounded-lg bg-[#150D26] border border-white/10 overflow-hidden h-[174px] sm:h-[225px] flex flex-col justify-between p-1">  
                    {spins[ci] ? (  
                      <div className="animate-reel-spin flex flex-col items-center gap-3 py-1 opacity-80">  
                        {SYMS.slice(0,5).map((s,k)=><SlotSym key={k} id={s.id} size={34}/>)}  
                      </div>  
                    ) : col.map((sym, ri) => (  
                      <div key={ri} className={`h-[52px] sm:h-[68px] rounded flex items-center justify-center ${hits.has(`${ci}-${ri}`)?"bg-amber-400/25 ring-2 ring-amber-300":""}`}>  
                        <SlotSym id={sym} size={36}/>  
                      </div>  
                    ))}  
                  </div>  
                ))}  
              </div>  
              {res && res.wins.length>0 && !busy && (  
                <svg viewBox="0 0 500 300" className="absolute inset-0 w-full h-full pointer-events-none p-3">  
                  {res.wins.map(w=><polyline key={w.id} points={w.coords.map(([c,r])=>`${c*100+50},${r*100+50}`).join(" ")} fill="none" stroke={w.color} strokeWidth="6" strokeLinecap="round"/>)}  
                </svg>  
              )}  
            </div>  
  
            <div className="rounded-xl bg-black/40 border border-white/10 px-3 py-2.5 text-xs sm:text-sm">  
              {busy ? <span className="text-violet-300 animate-pulse">Girando rodillos...</span> :  
               res ? (res.totalWin>0 ? <span className="text-amber-300 font-bold">¡GANASTE +{res.totalWin} PTS!</span> : <span className="text-slate-400">Sin combinación ganadora.</span>) :  
               <span className="text-slate-400">Selecciona puntos y pulsa Girar.</span>}  
            </div>  
  
            <div className="flex flex-wrap items-center justify-between gap-3">  
              <Chips vals={[25,50,100,250]} sel={stake} onSel={setStake} disabled={busy}/>  
              <Btn onClick={spin} disabled={busy || points<stake} className="w-full sm:w-auto px-7 py-3">  
                {busy?"Girando...":points<stake?"Sin puntos":`Girar (${stake} PTS)`}  
              </Btn>  
            </div>  
          </div>  
  
          {hist.length>0 && (  
            <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 text-xs space-y-1">  
              <div className="text-amber-300 font-semibold uppercase mb-1">Historial reciente</div>  
              {hist.map((h,i)=><div key={i} className="flex justify-between text-slate-300"><span>Giro {h.stake} PTS</span><span className="font-mono text-amber-200">{h.win>0?`+${h.win}`:0} PTS</span></div>)}  
            </div>  
          )}  
        </div>  
      );  
    };  
  
    /* === JUEGO 2: CP ROULETTE === */  
    const SLICE = 360 / WHEEL.length;  
    const polar = (cx,cy,r,deg) => { const rad=((deg-90)*Math.PI)/180; return { x:cx+r*Math.cos(rad), y:cy+r*Math.sin(rad) }; };  
    const arc = (cx,cy,rO,rI,a1,a2) => {  
      const p1=polar(cx,cy,rO,a1), p2=polar(cx,cy,rO,a2), p3=polar(cx,cy,rI,a2), p4=polar(cx,cy,rI,a1);  
      return `M${p1.x} ${p1.y} A${rO} ${rO} 0 0 1 ${p2.x} ${p2.y} L${p3.x} ${p3.y} A${rI} ${rI} 0 0 0 ${p4.x} ${p4.y} Z`;  
    };  
    const CpRoulette = ({ points, onBack, onRecord }) => {  
      const [chip, setChip] = useState(50);  
      const [bets, setBets] = useState([]);  
      const [spinning, setSpinning] = useState(false);  
      const [wAng, setWAng] = useState(0);  
      const [bAng, setBAng] = useState(0);  
      const [dropped, setDropped] = useState(true);  
      const [last, setLast] = useState(null);  
      const [recents, setRecents] = useState([17,32,0,8,23]);  
      const stake = bets.reduce((s,b)=>s+b.amt,0);  
  
      const addBet = (type, label, num) => {  
        if (spinning || points < stake + chip) return;  
        soundFX.chip();  
        const id = num!==undefined ? `${type}-${num}` : type;  
        setBets(p => p.find(b=>b.id===id) ? p.map(b=>b.id===id?{...b,amt:b.amt+chip}:b) : [...p,{id,type,label,num,amt:chip}]);  
      };  
      const getAmt = (id) => bets.find(b=>b.id===id)?.amt || 0;  
  
      const spin = () => {  
        if (spinning || !bets.length || points < stake) return;  
        setSpinning(true); setDropped(false); setLast(null); soundFX.wheel();  
        const r = runRoulette(bets);  
        const nw = wAng + 5*360 + Math.floor(Math.random()*360);  
        const target = (r.idx*SLICE + nw) % 360;  
        const base = ((bAng%360)+360)%360;  
        const nb = bAng - 4*360 + (((target - base + 360)%360) - 360);  
        setWAng(nw); setBAng(nb);  
        setTimeout(()=>setDropped(true), 2200);  
        setTimeout(()=>{  
          setSpinning(false); setLast(r); setRecents(p=>[r.num,...p].slice(0,10));  
          if (r.pay>0) soundFX.win(); else soundFX.lose();  
          onRecord({ gameId:"cp-roulette", gameTitle:"CP ROULETTE", stake:r.stake, payout:r.pay, net:r.pay-r.stake,  
            summary:`Salió ${r.num} (${r.col.toUpperCase()}) · Retorno ${r.pay} pts` });  
        }, 3800);  
      };  
  
      return (  
        <div className="space-y-4">  
          <div className="flex items-center justify-between">  
            <button onClick={onBack} className="text-xs text-violet-300">← Catálogo</button>  
            <div className="flex gap-1">{recents.slice(0,7).map((n,i)=>{  
              const c=rColor(n);  
              return <span key={i} className={`w-6 h-6 rounded text-[10px] font-mono font-bold flex items-center justify-center ${c==="green"?"bg-emerald-700":c==="red"?"bg-rose-700":"bg-slate-800"}`}>{n}</span>;  
            })}</div>  
          </div>  
          <h2 className="font-display text-xl sm:text-2xl font-bold gold-shimmer">CP ROULETTE · Ruleta Europea</h2>  
  
          <div className="grid lg:grid-cols-[310px_1fr] gap-4 items-start">  
            <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 flex flex-col items-center">  
              <svg viewBox="0 0 320 320" className="w-60 h-60 sm:w-64 sm:h-64">  
                <circle cx="160" cy="160" r="154" fill="#1E1033" stroke="#D4AF37" strokeWidth="3"/>  
                <g style={{transform:`rotate(${wAng}deg)`,transformOrigin:"160px 160px",transition:spinning?"transform 3.7s cubic-bezier(0.12,0.85,0.18,1)":"none"}}>  
                  {WHEEL.map((n,i)=>{  
                    const a1=i*SLICE-SLICE/2, a2=a1+SLICE, c=rColor(n), tp=polar(160,160,118,i*SLICE);  
                    return (  
                      <g key={n}>  
                        <path d={arc(160,160,134,88,a1,a2)} fill={c==="green"?"#059669":c==="red"?"#B91C1C":"#111827"} stroke="#D4AF37" strokeWidth="0.6"/>  
                        <text x={tp.x} y={tp.y} fill="#fff" fontSize="8.5" fontWeight="700" textAnchor="middle" dominantBaseline="middle" transform={`rotate(${i*SLICE},${tp.x},${tp.y})`}>{n}</text>  
                      </g>  
                    );  
                  })}  
                  <circle cx="160" cy="160" r="88" fill="#0D0817" stroke="#D4AF37" strokeWidth="2"/>  
                  <path d="M160 88V232M88 160H232" stroke="#D4AF37" strokeWidth="3"/>  
                  <circle cx="160" cy="160" r="18" fill="#D4AF37"/>  
                </g>  
                <g style={{transform:`rotate(${bAng}deg)`,transformOrigin:"160px 160px",transition:spinning?"transform 3.7s cubic-bezier(0.12,0.85,0.18,1)":"none"}}>  
                  <circle cx="160" cy={dropped?"58":"22"} r="6" fill="#fff" style={{transition:"cy 1.1s ease-in"}}/>  
                </g>  
              </svg>  
              <div className="mt-3 w-full text-center rounded-xl bg-black/40 border border-white/10 p-2.5 text-xs">  
                {spinning ? <span className="text-amber-300 animate-pulse">Bola en juego...</span> :  
                 last ? <span className="font-bold text-white">Salió {last.num} · <span className="text-amber-300">{last.pay>0?`+${last.pay} PTS`:"Sin premio"}</span></span> :  
                 <span className="text-slate-400">Coloca tus fichas en el paño.</span>}  
              </div>  
            </div>  
  
            <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 space-y-4">  
              <div className="flex flex-wrap justify-between items-center gap-2">  
                <Chips vals={[10,50,100,500]} sel={chip} onSel={setChip} disabled={spinning}/>  
                <div className="flex gap-2">  
                  <Btn variant="ghost" onClick={()=>setBets([])} disabled={spinning||!bets.length}>Limpiar</Btn>  
                  <Btn onClick={spin} disabled={spinning||!bets.length||points<stake}>{spinning?"Girando...":`Girar (${stake})`}</Btn>  
                </div>  
              </div>  
  
              <div className="overflow-x-auto pb-2">  
                <div className="min-w-[540px] space-y-1.5">  
                  <div className="grid grid-cols-[44px_repeat(12,1fr)_44px] gap-1">  
                    <button onClick={()=>addBet("straight","0",0)} className="row-span-3 rounded bg-emerald-700 border border-amber-400/40 font-mono text-xs font-bold relative">  
                      0{getAmt("straight-0")>0 && <span className="block text-[9px] text-amber-300">{getAmt("straight-0")}</span>}  
                    </button>  
                    {[3,2,1].map(ro=>(  
                      <React.Fragment key={ro}>  
                        {Array.from({length:12},(_,col)=>{  
                          const n=col*3+ro, c=rColor(n), a=getAmt(`straight-${n}`);  
                          return (  
                            <button key={n} onClick={()=>addBet("straight",`#${n}`,n)}  
                              className={`h-10 rounded font-mono text-xs font-bold border border-amber-400/20 relative ${c==="red"?"bg-rose-800":"bg-slate-900"}`}>  
                              {n}{a>0 && <span className="absolute -top-1 -right-1 px-1 rounded-full bg-amber-300 text-slate-950 text-[8px]">{a}</span>}  
                            </button>  
                          );  
                        })}  
                        <button onClick={()=>addBet(ro===3?"c3":ro===2?"c2":"c1",`Col ${ro}`)} className="h-10 rounded bg-violet-950 border border-amber-400/30 text-[10px] font-bold text-amber-200 relative">  
                          2:1{getAmt(ro===3?"c3":ro===2?"c2":"c1")>0 && <span className="absolute -top-1 -right-1 px-1 rounded-full bg-amber-300 text-slate-950 text-[8px]">{getAmt(ro===3?"c3":ro===2?"c2":"c1")}</span>}  
                        </button>  
                      </React.Fragment>  
                    ))}  
                  </div>  
                  <div className="grid grid-cols-3 gap-1 px-12">  
                    {[["d1","1-12"],["d2","13-24"],["d3","25-36"]].map(([t,l])=>(  
                      <button key={t} onClick={()=>addBet(t,l)} className="h-9 rounded bg-white/5 border border-amber-400/25 text-xs font-semibold">  
                        {l} {getAmt(t)>0 && <span className="text-amber-300">({getAmt(t)})</span>}  
                      </button>  
                    ))}  
                  </div>  
                  <div className="grid grid-cols-6 gap-1 px-12">  
                    {[["low","1-18","bg-white/5"],["even","PAR","bg-white/5"],["red","ROJO","bg-rose-800"],["black","NEGRO","bg-slate-900"],["odd","IMPAR","bg-white/5"],["high","19-36","bg-white/5"]].map(([t,l,bg])=>(  
                      <button key={t} onClick={()=>addBet(t,l)} className={`h-9 rounded ${bg} border border-amber-400/25 text-[11px] font-bold relative`}>  
                        {l}{getAmt(t)>0 && <span className="absolute -top-1 -right-1 px-1 rounded-full bg-amber-300 text-slate-950 text-[8px]">{getAmt(t)}</span>}  
                      </button>  
                    ))}  
                  </div>  
                </div>  
              </div>  
            </div>  
          </div>  
        </div>  
      );  
    };  
  
    /* === JUEGO 3: CP BLACKJACK === */  
    const CpBlackjack = ({ points, onBack, onRecord }) => {  
      const shoe = useRef(makeShoe(6));  
      const [bet, setBet] = useState(100);  
      const [stake, setStake] = useState(100);  
      const [phase, setPhase] = useState("bet");  
      const [pC, setPC] = useState([]);  
      const [dC, setDC] = useState([]);  
      const [hide, setHide] = useState(true);  
      const [msg, setMsg] = useState("Elige tu apuesta y reparte las cartas.");  
  
      const draw = () => { if (shoe.current.length<20) shoe.current=makeShoe(6); soundFX.card(); return shoe.current.pop(); };  
      const finish = (pc, dc, st) => {  
        setHide(false); setPhase("end");  
        const r = resBJ(pc, dc, st);  
        setMsg(r.msg);  
        if (r.pay>st) soundFX.win(); else if (r.pay===0) soundFX.lose();  
        onRecord({ gameId:"cp-blackjack", gameTitle:"CP BLACKJACK", stake:st, payout:r.pay, net:r.pay-st,  
          summary:`${r.msg} (${evalBJ(pc).tot} vs ${evalBJ(dc).tot})` });  
      };  
      const dealerGo = (pc, dc0, st) => {  
        setPhase("dealer"); setHide(false);  
        const dc = [...dc0];  
        const step = () => {  
          if (evalBJ(dc).tot < 17) { dc.push(draw()); setDC([...dc]); setTimeout(step, 500); }  
          else finish(pc, dc, st);  
        };  
        setTimeout(step, 400);  
      };  
      const deal = () => {  
        if (points < bet) return;  
        setStake(bet); setHide(true);  
        const pc=[draw(),draw()], dc=[draw(),draw()];  
        setPC(pc); setDC(dc);  
        if (evalBJ(pc).bj || evalBJ(dc).bj) { finish(pc, dc, bet); return; }  
        setPhase("play"); setMsg("Tu turno: Pedir Carta, Plantarse o Doblar.");  
      };  
      const hit = () => {  
        const next=[...pC, draw()]; setPC(next);  
        const ev = evalBJ(next);  
        if (ev.bust) finish(next, dC, stake);  
        else if (ev.tot===21) dealerGo(next, dC, stake);  
      };  
      const dbl = () => {  
        if (pC.length!==2 || points<stake*2) return;  
        const ns=stake*2; setStake(ns);  
        const next=[...pC, draw()]; setPC(next);  
        if (evalBJ(next).bust) finish(next, dC, ns); else dealerGo(next, dC, ns);  
      };  
  
      const pe = evalBJ(pC), de = evalBJ(hide && dC.length ? [dC[0]] : dC);  
  
      return (  
        <div className="space-y-4">  
          <button onClick={onBack} className="text-xs text-violet-300">← Catálogo</button>  
          <h2 className="font-display text-xl sm:text-2xl font-bold gold-shimmer">CP BLACKJACK · Mesa VIP 21</h2>  
          <div className="rounded-2xl bg-[radial-gradient(circle_at_top,rgba(16,185,129,0.18),rgba(10,6,18,0.96))] border border-emerald-400/25 p-5 space-y-5">  
            <div className="text-center space-y-2">  
              <div className="text-xs uppercase tracking-widest text-slate-300">  
                Repartidor {dC.length>0 && <span className="ml-2 px-2 py-0.5 rounded-full bg-black/50 text-amber-300 font-mono">{hide?`${de.tot} + ?`:de.tot}</span>}  
              </div>  
              <div className="min-h-[104px] flex justify-center gap-2 flex-wrap">  
                {dC.length===0 ? <span className="text-xs text-slate-500 self-center">Mesa lista</span> : dC.map((c,i)=><PlayingCard key={c.id} card={c} faceDown={i===1&&hide}/>)}  
              </div>  
            </div>  
  
            <div className="rounded-xl bg-black/45 border border-amber-400/25 p-3 text-center text-xs sm:text-sm font-semibold text-amber-200">{msg}</div>  
  
            <div className="text-center space-y-2">  
              <div className="min-h-[104px] flex justify-center gap-2 flex-wrap">  
                {pC.length===0 ? <span className="text-xs text-slate-500 self-center">Tu mano</span> : pC.map(c=><PlayingCard key={c.id} card={c}/>)}  
              </div>  
              <div className="text-xs uppercase tracking-widest text-slate-300">  
                Jugador {pC.length>0 && <span className="ml-2 px-2 py-0.5 rounded-full bg-black/50 text-violet-200 font-mono">{pe.tot}{pe.soft?" (Suave)":""}</span>}  
              </div>  
            </div>  
  
            <div className="pt-4 border-t border-white/10 flex flex-wrap items-center justify-between gap-3">  
              {phase==="bet" || phase==="end" ? (  
                <>  
                  <Chips vals={[25,50,100,250]} sel={bet} onSel={setBet}/>  
                  <div className="flex gap-2 w-full sm:w-auto">  
                    {phase==="end" && <Btn variant="ghost" onClick={()=>{setPhase("bet");setPC([]);setDC([]);setMsg("Lista para nueva mano.");}}>Limpiar</Btn>}  
                    <Btn className="flex-1 sm:flex-none" onClick={deal} disabled={points<bet}>{points<bet?"Sin puntos":`Repartir (${bet} PTS)`}</Btn>  
                  </div>  
                </>  
              ) : (  
                <div className="w-full flex flex-wrap justify-center gap-2">  
                  <Btn onClick={hit} disabled={phase!=="play"}>Pedir Carta</Btn>  
                  <Btn variant="violet" onClick={()=>dealerGo(pC,dC,stake)} disabled={phase!=="play"}>Plantarse</Btn>  
                  <Btn variant="ghost" onClick={dbl} disabled={phase!=="play"||pC.length!==2||points<stake*2}>Doblar ({stake*2})</Btn>  
                </div>  
              )}  
            </div>  
          </div>  
        </div>  
      );  
    };  
  
    /* === JUEGO 4: CP BACCARAT === */  
    const CpBaccarat = ({ points, onBack, onRecord }) => {  
      const shoe = useRef(makeShoe(8));  
      const [chip, setChip] = useState(50);  
      const [bets, setBets] = useState({ player:0, banker:0, tie:0 });  
      const [busy, setBusy] = useState(false);  
      const [res, setRes] = useState(null);  
      const [road, setRoad] = useState(["banker","player","banker","tie"]);  
      const stake = bets.player+bets.banker+bets.tie;  
  
      const add = (k) => { if (!busy && points>=stake+chip) { soundFX.chip(); setBets(p=>({...p,[k]:p[k]+chip})); } };  
      const deal = () => {  
        if (busy || !stake || points<stake) return;  
        setBusy(true); soundFX.card();  
        const r = runBaccarat(shoe.current, bets);  
        shoe.current = r.shoe;  
        setTimeout(()=>{  
          soundFX.card(); setRes(r); setBusy(false); setRoad(p=>[r.win,...p].slice(0,12));  
          if (r.pay>r.stake) soundFX.win(); else if (r.pay===0) soundFX.lose();  
          const lbl = r.win==="player"?"Jugador":r.win==="banker"?"Banca":"Empate";  
          onRecord({ gameId:"cp-baccarat", gameTitle:"CP BACCARAT", stake:r.stake, payout:r.pay, net:r.pay-r.stake,  
            summary:`Ganó ${lbl} (${r.pT} vs ${r.bT}) · Retorno ${r.pay} pts` });  
        }, 550);  
      };  
  
      return (  
        <div className="space-y-4">  
          <div className="flex items-center justify-between">  
            <button onClick={onBack} className="text-xs text-violet-300">← Catálogo</button>  
            <div className="flex gap-1">{road.slice(0,8).map((w,i)=>(  
              <span key={i} className={`w-6 h-6 rounded-full text-[10px] font-bold flex items-center justify-center ${w==="player"?"bg-sky-600":w==="banker"?"bg-rose-700":"bg-emerald-700"}`}>  
                {w==="player"?"J":w==="banker"?"B":"E"}  
              </span>  
            ))}</div>  
          </div>  
          <h2 className="font-display text-xl sm:text-2xl font-bold gold-shimmer">CP BACCARAT · Punto & Banca</h2>  
  
          <div className="rounded-2xl bg-[#120B20] border border-amber-400/25 p-4 sm:p-6 space-y-4">  
            <div className="grid grid-cols-1 sm:grid-cols-2 gap-3">  
              <div className={`p-3 rounded-xl border ${res?.win==="player"?"border-sky-400 bg-sky-950/40":"border-white/10 bg-black/30"}`}>  
                <div className="flex justify-between text-xs font-bold text-sky-300 mb-2"><span>JUGADOR</span><span className="font-mono">{res?`Pts: ${res.pT}`:"—"}</span></div>  
                <div className="min-h-[84px] flex justify-center gap-1.5">{res?res.pC.map(c=><PlayingCard key={c.id} card={c} small/>):<span className="text-xs text-slate-500 self-center">Cartas Jugador</span>}</div>  
              </div>  
              <div className={`p-3 rounded-xl border ${res?.win==="banker"?"border-rose-400 bg-rose-950/40":"border-white/10 bg-black/30"}`}>  
                <div className="flex justify-between text-xs font-bold text-rose-300 mb-2"><span>BANCA</span><span className="font-mono">{res?`Pts: ${res.bT}`:"—"}</span></div>  
                <div className="min-h-[84px] flex justify-center gap-1.5">{res?res.bC.map(c=><PlayingCard key={c.id} card={c} small/>):<span className="text-xs text-slate-500 self-center">Cartas Banca</span>}</div>  
              </div>  
            </div>  
  
            <div className="rounded-xl bg-black/45 border border-white/10 p-2.5 text-center text-xs text-amber-200">  
              {busy ? "Evaluando tercera carta..." : res ? `${res.note} Retorno: ${res.pay} PTS.` : "Toca Jugador, Empate o Banca para apostar."}  
            </div>  
  
            <div className="grid grid-cols-3 gap-2">  
              {[["player","JUGADOR","1:1","border-sky-400/40 bg-sky-950/40"],["tie","EMPATE","8:1","border-emerald-400/40 bg-emerald-950/40"],["banker","BANCA","0.95:1","border-rose-400/40 bg-rose-950/40"]].map(([k,t,odds,cls])=>(  
                <button key={k} onClick={()=>add(k)} disabled={busy} className={`p-3 rounded-xl border text-left ${cls}`}>  
                  <div className="text-[11px] font-bold">{t}</div>  
                  <div className="text-[10px] text-slate-300">{odds}</div>  
                  <div className="font-mono font-bold text-sm mt-1 text-amber-200">{bets[k]}</div>  
                </button>  
              ))}  
            </div>  
  
            <div className="flex flex-wrap items-center justify-between gap-3 pt-2 border-t border-white/10">  
              <Chips vals={[25,50,100,250]} sel={chip} onSel={setChip} disabled={busy}/>  
              <div className="flex gap-2 w-full sm:w-auto">  
                <Btn variant="ghost" onClick={()=>{setRes(null);setBets({player:0,banker:0,tie:0});}} disabled={busy}>Limpiar</Btn>  
                <Btn className="flex-1 sm:flex-none" onClick={deal} disabled={busy||!stake||points<stake}>{busy?"Repartiendo...":`Repartir (${stake})`}</Btn>  
              </div>  
            </div>  
          </div>  
        </div>  
      );  
    };  
  
    /* === APP PRINCIPAL Y 9 SECCIONES === */  
    const DEFAULT_STATE = {  
      profile: { name:"Invitado VIP", avatar:"crown", points:10000, level:1, played:0, won:0, best:0, sound:true },  
      favorites: ["cp-slots","cp-roulette"],  
      history: [],  
      claimed: {}  
    };  
  
    function App() {  
      const [state, setState] = useState(() => {  
        try {  
          const raw = safeStorage.get("cp_demo_v1");  
          return raw ? { ...DEFAULT_STATE, ...JSON.parse(raw) } : DEFAULT_STATE;  
        } catch { return DEFAULT_STATE; }  
      });  
      const [sec, setSec] = useState("inicio");  
      const [game, setGame] = useState(null);  
      const [menuOpen, setMenuOpen] = useState(false);  
      const [promoMsg, setPromoMsg] = useState("");  
      const [nameInput, setNameInput] = useState(state.profile.name);  
  
      useEffect(() => {  
        soundFX.enabled = state.profile.sound;  
        safeStorage.set("cp_demo_v1", JSON.stringify(state));  
      }, [state]);  
  
      const nav = (s) => { soundFX.click(); setGame(null); setSec(s); setMenuOpen(false); window.scrollTo(0,0); };  
      const launch = (id) => { soundFX.click(); setGame(id); setMenuOpen(false); window.scrollTo(0,0); };  
  
      const recordMatch = useCallback((m) => {  
        setState(prev => {  
          const pts = Math.max(0, prev.profile.points - m.stake + m.payout);  
          const played = prev.profile.played + 1;  
          const won = prev.profile.won + (m.payout > m.stake ? 1 : 0);  
          const rec = { ...m, id: `${Date.now()}-${Math.random()}`, time: new Date().toLocaleTimeString("es-ES",{hour:"2-digit",minute:"2-digit"}) };  
          return {  
            ...prev,  
            profile: { ...prev.profile, points:pts, played, won, best:Math.max(prev.profile.best, m.net), level:Math.floor(played/5)+1 },  
            history: [rec, ...prev.history].slice(0,40)  
          };  
        });  
      }, []);  
  
      const toggleFav = (id) => setState(p => ({ ...p, favorites: p.favorites.includes(id) ? p.favorites.filter(x=>x!==id) : [...p.favorites, id] }));  
  
      const NAV = [  
        ["inicio","Inicio"],["juegos","Juegos"],["novedades","Novedades"],["promociones","Promos Demo"],  
        ["ranking","Ranking"],["perfil","Perfil"],["favoritos","Favoritos"],["historial","Historial"],["soporte","Soporte"]  
      ];  
  
      const { profile } = state;  
      const winRate = profile.played ? Math.round((profile.won/profile.played)*100) : 0;  
      const fullRank = [...LEADERBOARD, { id:"me", name:`${profile.name} (Tú)`, avatar:profile.avatar, points:profile.points, winRate, fav:"ORIGINALS", me:true }].sort((a,b)=>b.points-a.points);  
  
      return (  
        <div className="min-h-screen flex flex-col lg:flex-row pb-16 lg:pb-0">  
          {/* Sidebar Desktop */}  
          <aside className="hidden lg:flex lg:w-60 lg:flex-col lg:fixed lg:inset-y-0 border-r border-white/10 bg-[#08050F]/95 p-4 justify-between z-40">  
            <div className="space-y-6">  
              <button onClick={()=>nav("inicio")} className="text-left"><BrandLogo/></button>  
              <nav className="space-y-1">  
                {NAV.map(([id,lbl])=>(  
                  <button key={id} onClick={()=>nav(id)} className={`w-full text-left px-3 py-2 rounded-xl text-sm ${!game&&sec===id?"bg-violet-700/35 text-amber-200 border border-amber-400/35":"text-slate-300 hover:bg-white/5"}`}>  
                    {lbl}  
                  </button>  
                ))}  
              </nav>  
            </div>  
            <div className="text-[11px] text-slate-400 bg-white/5 p-3 rounded-xl">Modo Demo · Puntos sin valor económico.</div>  
          </aside>  
  
          <div className="flex-1 lg:pl-60 flex flex-col">  
            {/* Topbar Móvil y Desktop */}  
            <header className="sticky top-0 z-30 border-b border-white/10 bg-[#08050F]/90 backdrop-blur-xl px-3 sm:px-6 py-3 flex items-center justify-between gap-2">  
              <div className="flex items-center gap-2">  
                <button onClick={()=>setMenuOpen(v=>!v)} className="lg:hidden px-2.5 py-1.5 rounded-lg bg-white/5 border border-white/15 text-xs font-semibold">Menú</button>  
                <div className="lg:hidden"><BrandLogo compact/></div>  
                <span className="hidden md:inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full bg-amber-500/10 border border-amber-400/30 text-[10px] text-amber-200">  
                  <span className="w-1.5 h-1.5 rounded-full bg-emerald-400"/> MODO DEMO SIMULADO  
                </span>  
              </div>  
              <div className="flex items-center gap-2">  
                <div className="px-2.5 py-1 rounded-xl bg-amber-500/10 border border-amber-400/35 font-mono font-bold text-xs text-amber-300">  
                  {profile.points.toLocaleString()} PTS  
                </div>  
                <button onClick={()=>nav("perfil")} className="p-0.5 rounded-full border border-amber-400/40">  
                  <AvatarIcon avatar={profile.avatar} size={28}/>  
                </button>  
              </div>  
            </header>  
  
            {/* Menú desplegable móvil */}  
            {menuOpen && (  
              <div className="lg:hidden fixed inset-0 top-[54px] z-40 bg-[#07040D]/95 p-4 grid grid-cols-2 gap-2 content-start overflow-y-auto">  
                {NAV.map(([id,lbl])=>(  
                  <button key={id} onClick={()=>nav(id)} className={`p-3 rounded-xl text-sm font-semibold border text-left ${!game&&sec===id?"bg-violet-700/35 border-amber-400/40 text-amber-200":"bg-white/5 border-white/10"}`}>{lbl}</button>  
                ))}  
              </div>  
            )}  
  
            <main className="flex-1 p-3 sm:p-6 max-w-6xl w-full mx-auto">  
              {game==="cp-slots" && <CpSlots points={profile.points} onBack={()=>nav("juegos")} onRecord={recordMatch}/>}  
              {game==="cp-roulette" && <CpRoulette points={profile.points} onBack={()=>nav("juegos")} onRecord={recordMatch}/>}  
              {game==="cp-blackjack" && <CpBlackjack points={profile.points} onBack={()=>nav("juegos")} onRecord={recordMatch}/>}  
              {game==="cp-baccarat" && <CpBaccarat points={profile.points} onBack={()=>nav("juegos")} onRecord={recordMatch}/>}  
  
              {!game && sec==="inicio" && (  
                <div className="space-y-6">  
                  <section className="rounded-3xl border border-amber-400/30 bg-gradient-to-br from-[#1B0E35] via-[#0B0614] to-[#1A1025] p-5 sm:p-8 space-y-4">  
                    <span className="inline-block px-3 py-1 rounded-full border border-amber-400/40 bg-violet-950 text-[10px] tracking-[0.2em] text-amber-200 uppercase">CASIN PREMIUM ORIGINALS</span>  
                    <BrandLogo/>  
                    <h1 className="font-display text-2xl sm:text-4xl font-bold text-white">Entretenimiento de Casino <span className="gold-shimmer">de Alta Precisión</span></h1>  
                    <p className="text-xs sm:text-sm text-slate-300 max-w-xl">Cuatro juegos originales programados desde cero. Créditos virtuales gratuitos sin valor económico.</p>  
                    <div className="flex flex-wrap gap-2">  
                      <Btn onClick={()=>nav("juegos")}>Ver Juegos</Btn>  
                      <Btn variant="violet" onClick={()=>launch("cp-slots")}>Jugar Slots</Btn>  
                      <Btn variant="ghost" onClick={()=>nav("promociones")}>Bonos Demo</Btn>  
                    </div>  
                  </section>  
  
                  <section className="space-y-3">  
                    <h2 className="font-display text-lg sm:text-xl font-bold">Juegos Destacados</h2>  
                    <div className="grid sm:grid-cols-2 lg:grid-cols-3 gap-4">  
                      {GAMES.filter(g=>g.featured).map(g=>(  
                        <div key={g.id} className="rounded-2xl bg-[#120B20] border border-white/10 overflow-hidden flex flex-col">  
                          <GameArt id={g.id}/>  
                          <div className="p-4 flex-1 flex flex-col justify-between gap-3">  
                            <div>  
                              <div className="flex justify-between items-center">  
                                <span className="text-[10px] text-violet-300 uppercase">{g.cat}</span>  
                                <button onClick={()=>toggleFav(g.id)} className="text-xs text-amber-300">{state.favorites.includes(g.id)?"★":"☆"}</button>  
                              </div>  
                              <h3 className="font-display font-bold text-lg mt-0.5">{g.title}</h3>  
                              <p className="text-xs text-slate-400 mt-1">{g.desc}</p>  
                            </div>  
                            <Btn onClick={()=>launch(g.id)} className="w-full">Jugar Ahora</Btn>  
                          </div>  
                        </div>  
                      ))}  
                    </div>  
                  </section>  
  
                  <div className="grid sm:grid-cols-2 gap-4">  
                    <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 space-y-2">  
                      <div className="font-display font-bold text-amber-300">Populares</div>  
                      {GAMES.filter(g=>g.popular).map(g=>(  
                        <div key={g.id} className="flex items-center justify-between p-2.5 rounded-xl bg-white/5">  
                          <div><div className="text-sm font-semibold">{g.title}</div><div className="text-[11px] text-slate-400">{g.sub}</div></div>  
                          <Btn variant="violet" onClick={()=>launch(g.id)}>Abrir</Btn>  
                        </div>  
                      ))}  
                    </div>  
                    <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 space-y-2">  
                      <div className="font-display font-bold text-white">Top Ranking Demo</div>  
                      {LEADERBOARD.slice(0,4).map((l,i)=>(  
                        <div key={l.id} className="flex items-center justify-between text-xs p-2 rounded-lg bg-white/5">  
                          <div className="flex items-center gap-2"><span className="font-mono text-amber-300">#{i+1}</span><AvatarIcon avatar={l.avatar} size={24}/><span>{l.name}</span></div>  
                          <span className="font-mono text-amber-200">{l.points.toLocaleString()} PTS</span>  
                        </div>  
                      ))}  
                    </div>  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="juegos" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">CASIN PREMIUM ORIGINALS</h2>  
                  <div className="grid sm:grid-cols-2 gap-4">  
                    {GAMES.map(g=>(  
                      <div key={g.id} className="rounded-2xl bg-[#120B20] border border-white/10 overflow-hidden flex flex-col">  
                        <GameArt id={g.id}/>  
                        <div className="p-4 flex-1 flex flex-col justify-between gap-3">  
                          <div>  
                            <div className="flex justify-between"><span className="text-[10px] text-violet-300 uppercase">{g.cat}</span>  
                              <button onClick={()=>toggleFav(g.id)} className="text-xs text-amber-300">{state.favorites.includes(g.id)?"★ Favorito":"☆ Guardar"}</button></div>  
                            <h3 className="font-display text-xl font-bold mt-1">{g.title}</h3>  
                            <p className="text-xs text-slate-300 mt-1">{g.desc}</p>  
                          </div>  
                          <Btn onClick={()=>launch(g.id)}>Iniciar Partida</Btn>  
                        </div>  
                      </div>  
                    ))}  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="novedades" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Novedades</h2>  
                  <div className="grid sm:grid-cols-3 gap-4">  
                    {NEWS.map(n=>(  
                      <div key={n.id} className="rounded-2xl bg-[#120B20] border border-white/10 p-4 space-y-2">  
                        <div className="flex justify-between text-[10px] text-amber-300 font-bold"><span>{n.tag}</span><span>{n.date}</span></div>  
                        <h3 className="font-display font-bold text-base">{n.title}</h3>  
                        <p className="text-xs text-slate-300">{n.desc}</p>  
                        <div className="text-[11px] font-mono text-emerald-300 pt-2 border-t border-white/10">✓ {n.hl}</div>  
                      </div>  
                    ))}  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="promociones" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Promociones Demo</h2>  
                  {promoMsg && <div className="p-3 rounded-xl bg-emerald-500/15 border border-emerald-400/40 text-xs text-emerald-200">{promoMsg}</div>}  
                  <div className="grid sm:grid-cols-3 gap-4">  
                    {PROMOS.map(p=>{  
                      const cd = Date.now() - (state.claimed[p.id]||0) < p.hrs*3600000;  
                      return (  
                        <div key={p.id} className="rounded-2xl bg-[#120B20] border border-amber-400/20 p-4 flex flex-col justify-between gap-4">  
                          <div>  
                            <div className="text-[10px] text-amber-300 font-bold">{p.badge}</div>  
                            <h3 className="font-display text-lg font-bold mt-1">{p.title}</h3>  
                            <p className="text-xs text-slate-300 mt-1">{p.desc}</p>  
                          </div>  
                          <div>  
                            <div className="font-mono text-xl font-bold text-amber-300 mb-2">+{p.pts} PTS</div>  
                            <Btn className="w-full" disabled={cd} onClick={()=>{  
                              setState(prev=>({...prev, profile:{...prev.profile, points:prev.profile.points+p.pts}, claimed:{...prev.claimed, [p.id]:Date.now()}}));  
                              setPromoMsg(`+${p.pts} PTS virtuales añadidos.`);  
                            }}>{cd?`Espera ${p.hrs}h`:"Reclamar Demo"}</Btn>  
                          </div>  
                        </div>  
                      );  
                    })}  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="ranking" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Ranking de Jugadores</h2>  
                  <div className="rounded-2xl bg-[#120B20] border border-white/10 overflow-x-auto">  
                    <table className="w-full text-left text-xs sm:text-sm">  
                      <thead className="bg-white/5 text-slate-400"><tr><th className="p-3">#</th><th className="p-3">Jugador</th><th className="p-3">Éxito</th><th className="p-3 text-right">Puntos</th></tr></thead>  
                      <tbody className="divide-y divide-white/5">  
                        {fullRank.map((r,i)=>(  
                          <tr key={r.id} className={r.me?"bg-violet-600/20 font-bold":""}>  
                            <td className="p-3 font-mono text-amber-300">{i+1}</td>  
                            <td className="p-3 flex items-center gap-2"><AvatarIcon avatar={r.avatar} size={26}/><span>{r.name}</span></td>  
                            <td className="p-3 font-mono">{r.winRate}%</td>  
                            <td className="p-3 text-right font-mono text-amber-200">{r.points.toLocaleString()}</td>  
                          </tr>  
                        ))}  
                      </tbody>  
                    </table>  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="perfil" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Perfil de Jugador</h2>  
                  <div className="rounded-2xl bg-[#120B20] border border-white/10 p-4 space-y-4">  
                    <div className="flex items-center gap-3">  
                      <AvatarIcon avatar={profile.avatar} size={56}/>  
                      <div><div className="font-bold text-lg">{profile.name}</div><div className="text-xs font-mono text-amber-300">Nivel {profile.level} · {profile.points.toLocaleString()} PTS</div></div>  
                    </div>  
                    <div className="flex gap-2">  
                      <input value={nameInput} maxLength={22} onChange={e=>setNameInput(e.target.value)} className="flex-1 rounded-xl bg-black/50 border border-white/15 px-3 py-2 text-sm"/>  
                      <Btn onClick={()=>nameInput.trim()&&setState(p=>({...p,profile:{...p.profile,name:nameInput.trim()}}))}>Guardar</Btn>  
                    </div>  
                    <div className="grid grid-cols-3 sm:grid-cols-6 gap-2">  
                      {AVATARS.map(a=>(  
                        <button key={a.id} onClick={()=>setState(p=>({...p,profile:{...p.profile,avatar:a.id}}))}  
                          className={`p-2 rounded-xl border flex flex-col items-center ${profile.avatar===a.id?"border-amber-400 bg-amber-400/15":"border-white/10"}`}>  
                          <AvatarIcon avatar={a.id} size={30}/><span className="text-[10px] mt-1">{a.label}</span>  
                        </button>  
                      ))}  
                    </div>  
                    <div className="grid grid-cols-2 sm:grid-cols-4 gap-2 text-xs">  
                      <div className="p-3 rounded-xl bg-black/40">Partidas: <b>{profile.played}</b></div>  
                      <div className="p-3 rounded-xl bg-black/40">Victorias: <b>{profile.won}</b></div>  
                      <div className="p-3 rounded-xl bg-black/40">Éxito: <b>{winRate}%</b></div>  
                      <div className="p-3 rounded-xl bg-black/40">Mejor: <b>+{profile.best}</b></div>  
                    </div>  
                    <div className="flex flex-wrap gap-2 pt-2">  
                      <Btn variant={profile.sound?"violet":"ghost"} onClick={()=>setState(p=>({...p,profile:{...p.profile,sound:!p.profile.sound}}))}>{profile.sound?"Sonido: ON":"Sonido: OFF"}</Btn>  
                      <Btn onClick={()=>setState(p=>({...p,profile:{...p.profile,points:p.profile.points+5000}}))}>+5,000 PTS Gratis</Btn>  
                      <Btn variant="ghost" onClick={()=>setState(DEFAULT_STATE)}>Reiniciar Demo</Btn>  
                    </div>  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="favoritos" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Favoritos</h2>  
                  {state.favorites.length===0 ? <div className="p-6 rounded-2xl bg-[#120B20] text-sm text-slate-400">Sin favoritos aún.</div> : (  
                    <div className="grid sm:grid-cols-2 gap-4">  
                      {GAMES.filter(g=>state.favorites.includes(g.id)).map(g=>(  
                        <div key={g.id} className="rounded-2xl bg-[#120B20] border border-white/10 overflow-hidden">  
                          <GameArt id={g.id}/>  
                          <div className="p-4 flex justify-between items-center">  
                            <div><div className="font-display font-bold">{g.title}</div><div className="text-xs text-slate-400">{g.sub}</div></div>  
                            <div className="flex gap-2"><Btn variant="ghost" onClick={()=>toggleFav(g.id)}>Quitar</Btn><Btn onClick={()=>launch(g.id)}>Jugar</Btn></div>  
                          </div>  
                        </div>  
                      ))}  
                    </div>  
                  )}  
                </div>  
              )}  
  
              {!game && sec==="historial" && (  
                <div className="space-y-4">  
                  <div className="flex justify-between items-center">  
                    <h2 className="font-display text-2xl font-bold">Historial</h2>  
                    {state.history.length>0 && <Btn variant="ghost" onClick={()=>setState(p=>({...p,history:[]}))}>Limpiar</Btn>}  
                  </div>  
                  <div className="rounded-2xl bg-[#120B20] border border-white/10 overflow-x-auto">  
                    {state.history.length===0 ? <div className="p-6 text-sm text-slate-400">Sin partidas aún.</div> : (  
                      <table className="w-full text-left text-xs">  
                        <thead className="bg-white/5 text-slate-400"><tr><th className="p-3">Hora</th><th className="p-3">Juego</th><th className="p-3">Resumen</th><th className="p-3 text-right">Balance</th></tr></thead>  
                        <tbody className="divide-y divide-white/5">  
                          {state.history.map(h=>(  
                            <tr key={h.id}><td className="p-3 font-mono text-slate-400">{h.time}</td><td className="p-3 font-semibold">{h.gameTitle}</td><td className="p-3">{h.summary}</td>  
                              <td className={`p-3 text-right font-mono font-bold ${h.net>0?"text-emerald-400":h.net<0?"text-rose-400":"text-amber-300"}`}>{h.net>0?`+${h.net}`:h.net}</td></tr>  
                          ))}  
                        </tbody>  
                      </table>  
                    )}  
                  </div>  
                </div>  
              )}  
  
              {!game && sec==="soporte" && (  
                <div className="space-y-4">  
                  <h2 className="font-display text-2xl font-bold">Soporte y Modo Demo</h2>  
                  <div className="rounded-2xl bg-amber-500/10 border border-amber-400/30 p-4 text-xs sm:text-sm text-amber-100">  
                    CASIN PREMIUM es una demostración recreativa. Todos los puntos y premios son ficticios y simulados localmente. No existen pagos, depósitos ni retiros reales.  
                  </div>  
                  <div className="grid sm:grid-cols-2 gap-3 text-xs sm:text-sm">  
                    <div className="p-4 rounded-2xl bg-[#120B20] border border-white/10"><b className="text-amber-300">CP SLOTS:</b> 5 líneas de izquierda a derecha (3, 4 o 5 iguales).</div>  
                    <div className="p-4 rounded-2xl bg-[#120B20] border border-white/10"><b className="text-amber-300">CP ROULETTE:</b> Europea 0-36. Pleno 35:1, docena 2:1, suertes 1:1.</div>  
                    <div className="p-4 rounded-2xl bg-[#120B20] border border-white/10"><b className="text-amber-300">CP BLACKJACK:</b> Natural paga 3:2. Crupier pide hasta 17.</div>  
                    <div className="p-4 rounded-2xl bg-[#120B20] border border-white/10"><b className="text-amber-300">CP BACCARAT:</b> Gana mano más cercana a 9 con regla oficial de 3ª carta.</div>  
                  </div>  
                </div>  
              )}  
            </main>  
          </div>  
  
          {/* Barra inferior rápida en celulares */}  
          <nav className="lg:hidden fixed bottom-0 inset-x-0 z-30 bg-[#08050F]/95 border-t border-white/10 grid grid-cols-4 text-[11px] py-2">  
            {[["inicio","Inicio"],["juegos","Juegos"],["ranking","Ranking"],["perfil","Perfil"]].map(([id,lbl])=>(  
              <button key={id} onClick={()=>nav(id)} className={`font-semibold ${!game&&sec===id?"text-amber-300":"text-slate-400"}`}>{lbl}</button>  
            ))}  
          </nav>  
        </div>  
      );  
    }  
  
    ReactDOM.createRoot(document.getElementById("root")).render(<App />);  
  </script>  
</body>  
</html>  
