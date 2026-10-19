import React, { useState, useEffect, useRef } from 'react';

// Danh sách các loại phần quà (Tổng cộng đúng 30 ô cho 3 trang)
const prizeConfig = [
  { name: 'Kẹo mút vui vẻ', icon: '🍭', count: 10, isSpecial: false, type: 'sweet', color: 'text-pink-400' },
  { name: 'Nhẫn ngọt ngào', icon: '💍', count: 5, isSpecial: false, type: 'sweet', color: 'text-cyan-300' },
  { name: 'Bánh socola', icon: '🍫', count: 6, isSpecial: false, type: 'normal', color: 'text-amber-400' },
  { name: 'Kẹp tóc hạnh phúc', icon: '🎀', count: 4, isSpecial: false, type: 'sweet', color: 'text-rose-400' },
  { name: 'QUÀ ĐẶC BIỆT', icon: '🎁', count: 1, isSpecial: true, type: 'special', color: 'text-yellow-300' },
  { name: 'Chúc may mắn', icon: '🍀', count: 4, isSpecial: false, type: 'normal', color: 'text-emerald-400' },
];

// Bộ tạo âm thanh tổng hợp bằng Web Audio API không cần URL bên ngoài
const playSound = (type) => {
  try {
    const AudioContext = window.AudioContext || window.webkitAudioContext;
    if (!AudioContext) return;
    const ctx = new AudioContext();

    if (type === 'open') {
      // Âm thanh pop nhẹ khi mở ô
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.setValueAtTime(300, ctx.currentTime);
      osc.frequency.exponentialRampToValueAtTime(600, ctx.currentTime + 0.15);
      gain.gain.setValueAtTime(0.3, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.15);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.15);
    } else if (type === 'sweet') {
      // Âm thanh hợp âm lấp lánh
      const notes = [523.25, 659.25, 783.99, 1046.50]; // C5, E5, G5, C6
      notes.forEach((freq, idx) => {
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.type = 'triangle';
        osc.frequency.value = freq;
        gain.gain.setValueAtTime(0.15, ctx.currentTime + idx * 0.08);
        gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + idx * 0.08 + 0.3);
        osc.connect(gain);
        gain.connect(ctx.destination);
        osc.start(ctx.currentTime + idx * 0.08);
        osc.stop(ctx.currentTime + idx * 0.08 + 0.3);
      });
    } else if (type === 'special') {
      // Nhạc chiến thắng hào hùng cho Quà Đặc Biệt
      const fanfare = [392, 523.25, 659.25, 783.99, 1046.50];
      fanfare.forEach((freq, idx) => {
        const osc = ctx.createOscillator();
        const gain = ctx.createGain();
        osc.type = 'sine';
        osc.frequency.value = freq;
        const startTime = ctx.currentTime + idx * 0.12;
        const duration = idx === fanfare.length - 1 ? 0.8 : 0.2;
        gain.gain.setValueAtTime(0.25, startTime);
        gain.gain.exponentialRampToValueAtTime(0.001, startTime + duration);
        osc.connect(gain);
        gain.connect(ctx.destination);
        osc.start(startTime);
        osc.stop(startTime + duration);
      });
    }
  } catch (e) {
    console.warn('Audio playback not allowed without user interaction first', e);
  }
};

const shuffleArray = (array) => {
  const newArray = [...array];
  for (let i = newArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
  }
  return newArray;
};

const generateBoxes = () => {
  let initialPrizes = [];
  prizeConfig.forEach((prize) => {
    for (let i = 0; i < prize.count; i++) {
      initialPrizes.push({ ...prize });
    }
  });
  
  const shuffled = shuffleArray(initialPrizes);
  return shuffled.map((prize, index) => ({
    id: index,
    ...prize,
    isOpen: false,
  }));
};

const ConfettiOverlay = () => {
  const [pieces, setPieces] = useState([]);

  useEffect(() => {
    const newPieces = Array.from({ length: 90 }).map((_, i) => ({
      id: i,
      left: `${Math.random() * 100}vw`,
      animationDuration: `${2.5 + Math.random() * 2.5}s`,
      animationDelay: `${Math.random() * 0.4}s`,
      color: ['#FFD700', '#FF69B4', '#00FFFF', '#FF4500', '#7FFF00', '#E6E6FA'][Math.floor(Math.random() * 6)],
      shape: Math.random() > 0.5 ? 'circle' : 'square',
      size: `${8 + Math.random() * 10}px`,
    }));
    setPieces(newPieces);
  }, []);

  return (
    <div className="fixed inset-0 pointer-events-none z-[100] overflow-hidden">
      {pieces.map((p) => (
        <div
          key={p.id}
          className="absolute -top-5"
          style={{
            left: p.left,
            width: p.size,
            height: p.shape === 'circle' ? p.size : `${parseInt(p.size) * 1.5}px`,
            backgroundColor: p.color,
            borderRadius: p.shape === 'circle' ? '50%' : '2px',
            animationName: 'confettiFall',
            animationDuration: p.animationDuration,
            animationTimingFunction: 'linear',
            animationIterationCount: 'infinite',
            animationDelay: p.animationDelay,
            boxShadow: `0 0 8px ${p.color}`,
          }}
        />
      ))}
    </div>
  );
};

const FloatingHearts = () => {
  const elements = ['❤️', '💖', '✨', '💕', '⭐', '🌸'];
  
  const hearts = React.useMemo(() => {
    return Array.from({ length: 7 }).map((_, i) => ({
      id: i,
      element: elements[i % elements.length],
      duration: `${1.2 + Math.random() * 0.8}s`,
      delay: `${i * 0.1}s`,
      left: `${20 + (i * 12) + (Math.random() * 10)}%`,
    }));
  }, []);

  return (
    <div className="absolute inset-0 pointer-events-none flex justify-center items-center z-40 overflow-visible">
      {hearts.map((h) => (
        <span
          key={h.id}
          className="absolute text-2xl select-none"
          style={{
            animationName: 'floatUp',
            animationDuration: h.duration,
            animationTimingFunction: 'ease-out',
            animationFillMode: 'forwards',
            animationDelay: h.delay,
            left: h.left,
            top: '50%',
          }}
        >
          {h.element}
        </span>
      ))}
    </div>
  );
};

export default function App() {
  const [boxes, setBoxes] = useState([]);
  const [currentPage, setCurrentPage] = useState(0);
  const [showConfetti, setShowConfetti] = useState(false);
  const [showResetModal, setShowResetModal] = useState(false);
  const [specialPrizeModal, setSpecialPrizeModal] = useState(false);
  const [activeBoxAnim, setActiveBoxAnim] = useState(null);

  const itemsPerPage = 10;
  const totalPages = 3;

  useEffect(() => {
    setBoxes(generateBoxes());
  }, []);

  const handleOpenBox = (id, boxType) => {
    // Phát âm thanh tương ứng
    if (boxType === 'special') {
      playSound('special');
    } else if (boxType === 'sweet') {
      playSound('sweet');
    } else {
      playSound('open');
    }

    setActiveBoxAnim(id);

    setBoxes((prevBoxes) =>
      prevBoxes.map((box) =>
        box.id === id ? { ...box, isOpen: true } : box
      )
    );

    if (boxType === 'special') {
      setShowConfetti(true);
      setSpecialPrizeModal(true);
      setTimeout(() => setShowConfetti(false), 8000);
    }
  };

  const confirmReset = () => {
    setBoxes(generateBoxes());
    setCurrentPage(0);
    setShowConfetti(false);
    setShowResetModal(false);
    setSpecialPrizeModal(false);
  };

  const nextPage = () => {
    if (currentPage < totalPages - 1) setCurrentPage((prev) => prev + 1);
  };

  const prevPage = () => {
    if (currentPage > 0) setCurrentPage((prev) => prev - 1);
  };

  const currentBoxes = boxes.slice(
    currentPage * itemsPerPage,
    (currentPage + 1) * itemsPerPage
  );

  const openedCount = boxes.filter((b) => b.isOpen).length;

  // Thống kê quà đã mở
  const collectedSummary = prizeConfig.map((prize) => {
    const countOpened = boxes.filter((b) => b.isOpen && b.name === prize.name).length;
    return { ...prize, countOpened };
  });

  return (
    <>
      {/* CSS tùy chỉnh bổ sung cho các hiệu ứng 3D chuẩn xác */}
      <style>
        {`
          @keyframes confettiFall {
            0% { transform: translateY(-5vh) rotate(0deg) scale(1); opacity: 1; }
            100% { transform: translateY(105vh) rotate(720deg) scale(0.4); opacity: 0; }
          }
          @keyframes floatUp {
            0% { transform: translateY(0) scale(0.6) rotate(-10deg); opacity: 0; }
            30% { opacity: 1; }
            100% { transform: translateY(-120px) scale(1.3) rotate(15deg); opacity: 0; }
          }
          @keyframes shimmer {
            0% { transform: translateX(-100%); }
            100% { transform: translateX(200%); }
          }
          @keyframes glowPulse {
            0%, 100% { box-shadow: 0 0 15px rgba(255, 215, 0, 0.4), inset 0 0 10px rgba(255, 215, 0, 0.2); }
            50% { box-shadow: 0 0 35px rgba(255, 215, 0, 0.8), inset 0 0 20px rgba(255, 215, 0, 0.5); }
          }
          
          .perspective-container {
            perspective: 1000px;
          }
          .transform-preserve-3d {
            transform-style: preserve-3d;
          }
          .backface-hidden {
            backface-visibility: hidden;
            -webkit-backface-visibility: hidden;
          }
          .rotate-y-180 {
            transform: rotateY(180deg);
          }
          .rotate-y-0 {
            transform: rotateY(0deg);
          }
          .glass-card {
            background: rgba(255, 255, 255, 0.04);
            backdrop-filter: blur(12px);
            -webkit-backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 215, 0, 0.25);
          }
          .gold-gradient-text {
            background: linear-gradient(135deg, #FFF8DC 0%, #FFD700 50%, #FFA500 100%);
            -webkit-background-clip: text;
            -webkit-text-fill-color: transparent;
            filter: drop-shadow(0px 2px 8px rgba(255, 215, 0, 0.3));
          }
        `}
      </style>

      {/* Main Container */}
      <div className="min-h-screen bg-[radial-gradient(ellipse_at_top,_var(--tw-gradient-stops))] from-slate-900 via-indigo-950 to-black text-white flex flex-col items-center py-8 px-4 font-sans relative overflow-x-hidden select-none">
        
        {/* Confetti khi trúng quà đặc biệt */}
        {showConfetti && <ConfettiOverlay />}

        {/* Đốm sáng hiệu ứng huyền ảo ở nền */}
        <div className="absolute top-0 left-1/4 w-96 h-96 bg-yellow-500/10 rounded-full filter blur-[120px] pointer-events-none"></div>
        <div className="absolute bottom-10 right-1/4 w-96 h-96 bg-purple-600/15 rounded-full filter blur-[120px] pointer-events-none"></div>

        <div className="w-full max-w-5xl flex flex-col items-center relative z-10">
          
          {/* Header */}
          <header className="text-center mb-6">
            <div className="inline-block px-4 py-1.5 rounded-full glass-card text-xs uppercase tracking-widest text-yellow-300/80 mb-3 border border-yellow-500/20">
              Sự Kiện Đặc Biệt • Bốc Thăm Trúng Thưởng
            </div>
            <h1 className="text-4xl md:text-6xl font-black gold-gradient-text uppercase tracking-wider mb-2">
              Hộp Quà Bí Mật
            </h1>
            <p className="text-slate-300 text-sm md:text-base font-light max-w-md mx-auto">
              Chạm vào mỗi hộp quà để mở khóa những bất ngờ độc đáo
            </p>
          </header>

          {/* Thanh Tiến Độ Tiến Trình */}
          <div className="w-full max-w-md mb-8 px-2 flex flex-col items-center gap-1.5">
            <div className="flex justify-between w-full text-xs font-semibold tracking-wider text-yellow-200/80">
              <span>Tiến độ mở quà</span>
              <span>{openedCount} / 30 Hộp đã mở</span>
            </div>
            <div className="w-full h-2.5 bg-slate-800/80 rounded-full overflow-hidden border border-yellow-500/20 p-0.5">
              <div 
                className="h-full bg-gradient-to-r from-yellow-500 to-amber-300 rounded-full transition-all duration-500 shadow-[0_0_10px_rgba(255,215,0,0.5)]"
                style={{ width: `${(openedCount / 30) * 100}%` }}
              ></div>
            </div>
          </div>

          {/* Lưới 10 Ô Quà (5x2) trên mỗi Slide */}
          <div className="grid grid-cols-2 sm:grid-cols-5 gap-4 md:gap-6 w-full mb-8">
            {currentBoxes.map((box) => (
              <div
                key={box.id}
                className="perspective-container h-40 md:h-48 w-full"
              >
                <button
                  type="button"
                  onClick={() => !box.isOpen && handleOpenBox(box.id, box.type)}
                  disabled={box.isOpen}
                  aria-label={`Ô quà số ${box.id + 1}`}
                  className={`w-full h-full relative transition-transform duration-700 transform-preserve-3d text-left focus:outline-none ${
                    box.isOpen ? 'rotate-y-180 cursor-default' : 'hover:-translate-y-2 hover:scale-[1.02] cursor-pointer'
                  }`}
                >
                  {/* MẶT TRƯỚC (Hộp quà chưa mở) */}
                  <div className="absolute inset-0 w-full h-full rounded-2xl glass-card overflow-hidden flex flex-col items-center justify-center p-4 backface-hidden border border-yellow-500/30 hover:border-yellow-400 shadow-lg">
                    {/* Đường sáng lướt qua */}
                    <div className="absolute inset-0 bg-gradient-to-r from-transparent via-yellow-200/10 to-transparent -skew-x-12 animate-[shimmer_3s_infinite]"></div>
                    
                    {/* Số thứ tự ô */}
                    <div className="w-14 h-14 rounded-full border border-yellow-400/40 bg-gradient-to-br from-yellow-500/20 via-yellow-500/5 to-transparent flex items-center justify-center mb-3 shadow-inner">
                      <span className="text-2xl font-serif font-bold text-yellow-300">{box.id + 1}</span>
                    </div>

                    <span className="text-yellow-200/90 text-xs tracking-widest uppercase font-semibold">Chạm Để Mở</span>
                  </div>

                  {/* MẶT SAU (Phần quà revealed) */}
                  <div className={`absolute inset-0 w-full h-full rounded-2xl flex flex-col items-center justify-center p-3 text-center backface-hidden rotate-y-180 bg-gradient-to-b from-slate-900 to-slate-950 ${
                    box.isSpecial 
                      ? 'border-2 border-yellow-400 shadow-[0_0_30px_rgba(255,215,0,0.6)] animate-[glowPulse_2s_infinite]' 
                      : 'border border-slate-700 shadow-xl'
                  }`}>
                    {/* Hiệu ứng bay tim nếu trúng đồ ngọt */}
                    {box.isOpen && (box.type === 'sweet' || box.type === 'special') && <FloatingHearts />}

                    <span className={`text-5xl md:text-6xl mb-2 filter drop-shadow-md transition-transform ${
                      box.isSpecial ? 'animate-bounce' : 'animate-pulse'
                    }`}>
                      {box.icon}
                    </span>

                    <span className={`font-bold text-xs md:text-sm uppercase tracking-wider leading-snug ${box.color}`}>
                      {box.name}
                    </span>

                    {box.isSpecial && (
                      <span className="mt-1 px-2 py-0.5 rounded bg-yellow-400 text-slate-950 font-black text-[10px] tracking-widest uppercase shadow">
                        Đặc Biệt!
                      </span>
                    )}
                  </div>
                </button>
              </div>
            ))}
          </div>

          {/* Nút Điều Huống Phân Trang & Điều Khiển */}
          <div className="flex flex-col items-center gap-5 w-full">
            
            {/* Phân trang */}
            <div className="flex items-center justify-between w-full max-w-sm glass-card rounded-full p-1.5 border border-yellow-500/30">
              <button
                onClick={prevPage}
                disabled={currentPage === 0}
                className={`px-5 py-2 rounded-full font-bold uppercase tracking-wider text-xs transition-all ${
                  currentPage === 0
                    ? 'text-slate-600 bg-transparent cursor-not-allowed'
                    : 'text-slate-950 bg-gradient-to-r from-yellow-300 to-amber-400 hover:shadow-[0_0_15px_rgba(255,215,0,0.5)] active:scale-95'
                }`}
              >
                &#8592; Trang Trước
              </button>

              <span className="font-extrabold text-yellow-300 text-sm tracking-widest px-2">
                Trang {currentPage + 1} / {totalPages}
              </span>

              <button
                onClick={nextPage}
                disabled={currentPage === totalPages - 1}
                className={`px-5 py-2 rounded-full font-bold uppercase tracking-wider text-xs transition-all ${
                  currentPage === totalPages - 1
                    ? 'text-slate-600 bg-transparent cursor-not-allowed'
                    : 'text-slate-950 bg-gradient-to-r from-amber-400 to-yellow-300 hover:shadow-[0_0_15px_rgba(255,215,0,0.5)] active:scale-95'
                }`}
              >
                Trang Sau &#8594;
              </button>
            </div>

            {/* Nút Chơi lại */}
            <button
              onClick={() => setShowResetModal(true)}
              className="flex items-center gap-2 text-yellow-300/70 hover:text-yellow-300 transition-colors text-xs font-semibold uppercase tracking-widest py-2 px-4 rounded-lg hover:bg-white/5"
            >
              <span className="text-sm">🔄</span> Xáo Trộn & Chơi Lại Ván Mới
            </button>
          </div>

          {/* Bảng Thống Kê Các Phần Quà Đã Thu Thập */}
          <div className="mt-10 w-full glass-card rounded-2xl p-5 border border-yellow-500/20">
            <h3 className="text-sm font-bold text-yellow-300 uppercase tracking-wider mb-4 text-center">
              Danh Sách Phần Quà Trong Hệ Thống (30 Ô)
            </h3>
            <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-6 gap-3">
              {collectedSummary.map((item, idx) => (
                <div key={idx} className="bg-slate-900/60 rounded-xl p-3 border border-slate-800 flex flex-col items-center text-center">
                  <span className="text-2xl mb-1">{item.icon}</span>
                  <span className="text-xs font-semibold text-slate-300 mb-1 line-clamp-1">{item.name}</span>
                  <span className="text-[11px] text-yellow-400/90 font-mono">
                    Đã mở: {item.countOpened}/{item.count}
                  </span>
                </div>
              ))}
            </div>
          </div>

        </div>

        {/* Custom Confirmation Modal (Thay thế window.confirm) */}
        {showResetModal && (
          <div className="fixed inset-0 z-[120] flex items-center justify-center p-4 bg-black/70 backdrop-blur-md animate-fadeIn">
            <div className="glass-card max-w-sm w-full p-6 rounded-2xl border border-yellow-500/40 text-center shadow-2xl">
              <div className="text-4xl mb-3">🎲</div>
              <h3 className="text-lg font-bold text-yellow-300 mb-2">Bắt đầu ván mới?</h3>
              <p className="text-slate-300 text-xs mb-6 leading-relaxed">
                Tất cả 30 hộp quà sẽ được niêm phong lại và xáo trộn hoàn toàn ngẫu nhiên.
              </p>
              <div className="flex gap-3 justify-center">
                <button
                  onClick={() => setShowResetModal(false)}
                  className="px-4 py-2 rounded-xl text-xs font-bold text-slate-300 bg-slate-800 hover:bg-slate-700 transition"
                >
                  Hủy Bỏ
                </button>
                <button
                  onClick={confirmReset}
                  className="px-5 py-2 rounded-xl text-xs font-bold text-slate-950 bg-gradient-to-r from-yellow-400 to-amber-500 hover:brightness-110 shadow-lg transition"
                >
                  Đồng Ý Xáo Trộn
                </button>
              </div>
            </div>
          </div>
        )}

        {/* Modal Chúc Mừng Quà Đặc Biệt */}
        {specialPrizeModal && (
          <div className="fixed inset-0 z-[130] flex items-center justify-center p-4 bg-black/80 backdrop-blur-md animate-fadeIn">
            <div className="glass-card max-w-md w-full p-8 rounded-3xl border-2 border-yellow-400 text-center shadow-[0_0_50px_rgba(255,215,0,0.5)] relative overflow-hidden">
              <div className="absolute -top-10 -right-10 w-32 h-32 bg-yellow-400/20 rounded-full blur-2xl"></div>
              
              <div className="text-6xl mb-4 animate-bounce">🎁</div>
              <h2 className="text-2xl font-black gold-gradient-text uppercase tracking-widest mb-2">
                Chúc Mừng!
              </h2>
              <p className="text-yellow-200 text-base font-semibold mb-4">
                Bạn đã mở trúng <span className="text-yellow-400 font-bold underline">QUÀ ĐẶC BIỆT</span> duy nhất!
              </p>
              <p className="text-slate-300 text-xs mb-6 leading-relaxed">
                Hãy lưu lại thông tin hoặc chụp màn hình để nhận phần thưởng đặc biệt này nhé!
              </p>
              
              <button
                onClick={() => setSpecialPrizeModal(false)}
                className="w-full py-3 rounded-xl font-extrabold text-xs uppercase tracking-widest text-slate-950 bg-gradient-to-r from-yellow-300 via-amber-400 to-yellow-500 shadow-lg hover:brightness-110 transition active:scale-95"
              >
                Tuyệt Vời • Nhận Quà
              </button>
            </div>
          </div>
        )}

      </div>
    </>
  );
}
