import React, { useState, useEffect, useMemo } from 'react';
import { 
  Shield, 
  Flame, 
  History, 
  LogOut, 
  Users, 
  Settings, 
  RefreshCw, 
  FileSpreadsheet, 
  ChevronRight, 
  User, 
  Lock,
  Trophy,
  AlertCircle,
  Image as ImageIcon,
  Calendar,
  Clock
} from 'lucide-react';

// --- MOCK DATA & UTILS (Mô phỏng Backend) ---

// Helper format ngày tháng kiểu Việt Nam
const formatDateVN = (isoString) => {
  if (!isoString) return '--';
  return new Date(isoString).toLocaleString('vi-VN', {
    weekday: 'short', // Thứ
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  });
};

const INITIAL_USERS = [
  { id: 'u1', email: 'admin@pegasus.com', name: 'Pegasus Admin', role: 'admin', history: [], currentNumber: null, allocatedAt: null },
  { id: 'u2', email: 'member1@pegasus.com', name: 'Chiến Binh 1', role: 'member', history: [12, 45], currentNumber: null, allocatedAt: null },
  { id: 'u3', email: 'member2@pegasus.com', name: 'Chiến Binh 2', role: 'member', history: [5], currentNumber: null, allocatedAt: null },
  { id: 'u4', email: 'member3@pegasus.com', name: 'Chiến Binh 3', role: 'member', history: [], currentNumber: null, allocatedAt: null },
];

const generateId = () => Math.random().toString(36).substr(2, 9);

// --- CORE ALGORITHM ---
const allocateWeeklyNumbers = (users, maxRange = 99) => {
  const fullPool = Array.from({ length: maxRange }, (_, i) => i + 1);
  const shuffledUsers = [...users].sort(() => Math.random() - 0.5);
  
  const weeklyAllocations = new Map(); 
  const results = [];
  const errors = [];
  const timestamp = new Date().toISOString(); // Lấy thời gian hiện tại
  
  const newUsersState = shuffledUsers.map(user => {
    const userHistorySet = new Set(user.history);
    const availableNumbers = fullPool.filter(num => 
      !userHistorySet.has(num) && !weeklyAllocations.has(num)
    );
    
    if (availableNumbers.length === 0) {
      errors.push(`Không thể cấp số cho ${user.name} (Đã full số hoặc hết quỹ số)`);
      return user; 
    }
    
    const luckyNumber = availableNumbers[Math.floor(Math.random() * availableNumbers.length)];
    weeklyAllocations.set(luckyNumber, user.id);
    const newHistory = [luckyNumber, ...user.history];
    results.push({ name: user.name, number: luckyNumber });
    
    return {
      ...user,
      currentNumber: luckyNumber,
      allocatedAt: timestamp, // Lưu thời gian cấp
      history: newHistory
    };
  });

  return {
    updatedUsers: newUsersState,
    allocations: Array.from(weeklyAllocations.entries()),
    success: errors.length === 0,
    errors
  };
};

// --- COMPONENTS ---

// 1. MASCOT COMPONENT
const Mascot = ({ action = 'idle', className = '' }) => {
  // Ưu tiên 1: Ảnh gốc của bạn (Cần file này trong thư mục public khi deploy)
  const primarySrc = "/image_cd00be.png"; 
  
  // Ưu tiên 2: Ảnh dự phòng Online (Hiện ngay trên preview nếu chưa có file gốc)
  // Sử dụng ảnh style chiến binh lửa để phù hợp theme
  const fallbackSrc = "https://images.unsplash.com/photo-1635322966219-b75ed3a9eb8c?q=80&w=800&auto=format&fit=crop";

  const [useFallback, setUseFallback] = useState(false);

  const getAnimationClass = () => {
    switch (action) {
      case 'wave': return 'animate-wave origin-bottom-right';
      case 'celebrate': return 'animate-bounce-custom scale-110';
      case 'holding': return 'translate-x-4 scale-105';
      default: return 'animate-float';
    }
  };

  return (
    <div className={`relative group transition-all duration-500 z-10 ${className}`}>
      {/* Hiệu ứng hào quang */}
      <div className={`absolute inset-0 bg-gradient-to-tr from-orange-600 via-red-500 to-yellow-400 rounded-full blur-3xl opacity-40 group-hover:opacity-60 transition-opacity duration-700 ${action === 'celebrate' ? 'animate-pulse' : ''}`}></div>
      
      <div className={`relative transform transition-transform duration-700 ${getAnimationClass()}`}>
        <img 
          src={useFallback ? fallbackSrc : primarySrc} 
          onError={() => setUseFallback(true)} // Tự động chuyển sang ảnh dự phòng nếu lỗi
          alt="Pegasus Mascot" 
          className="w-56 h-56 md:w-72 md:h-72 object-contain drop-shadow-[0_10px_10px_rgba(234,88,12,0.5)] filter hover:brightness-110 transition-all duration-300"
        />
        
        {/* Bong bóng thoại */}
        {action === 'celebrate' && (
          <div className="absolute -top-6 -right-6 bg-white text-red-600 font-bold px-5 py-3 rounded-full shadow-xl animate-bounce z-20 border-2 border-orange-100 transform rotate-12">
            Tuyệt vời! 🎉
          </div>
        )}
        {action === 'wave' && (
           <div className="absolute top-0 -left-8 bg-white text-gray-800 font-bold px-5 py-3 rounded-tl-none rounded-2xl shadow-xl z-20 border-2 border-orange-100 animate-fade-in-up">
            Xin chào! 👋
          </div>
        )}
      </div>
      <style>{`
        @keyframes wave { 0%, 100% { transform: rotate(0deg); } 25% { transform: rotate(-5deg); } 75% { transform: rotate(5deg); } }
        @keyframes float { 0%, 100% { transform: translateY(0px); } 50% { transform: translateY(-10px); } }
        @keyframes bounce-custom { 0%, 100% { transform: translateY(0) scale(1.1); } 50% { transform: translateY(-20px) scale(1.1); } }
        .animate-wave { animation: wave 3s infinite ease-in-out; }
        .animate-float { animation: float 4s infinite ease-in-out; }
        .animate-bounce-custom { animation: bounce-custom 1s infinite; }
      `}</style>
    </div>
  );
};

// 2. LOGIN SCREEN
const LoginScreen = ({ onLogin, onRegister }) => {
  const [isRegistering, setIsRegistering] = useState(false);
  const [formData, setFormData] = useState({ email: '', password: '', name: '' });
  const [error, setError] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    setError('');
    if (isRegistering) {
      if (!formData.name || !formData.email || !formData.password) {
        setError("Vui lòng điền đầy đủ thông tin!");
        return;
      }
      onRegister(formData);
    } else {
      if (!formData.email || !formData.password) {
        setError("Vui lòng nhập email và mật khẩu!");
        return;
      }
      onLogin(formData.email, formData.password);
    }
  };

  return (
    <div className="flex flex-col md:flex-row h-screen w-full bg-gradient-to-br from-red-900 via-red-700 to-orange-600 overflow-hidden font-sans">
      <div className="md:w-1/2 flex flex-col items-center justify-center p-8 text-white text-center relative overflow-hidden">
        <div className="absolute inset-0 bg-[url('https://www.transparenttextures.com/patterns/cubes.png')] opacity-10"></div>
        <div className="absolute top-1/4 left-1/4 w-96 h-96 bg-orange-500 rounded-full mix-blend-overlay filter blur-3xl opacity-20 animate-pulse"></div>
        <div className="mb-10 z-10 transform hover:scale-105 transition-transform duration-500">
           <Mascot action="wave" />
        </div>
        <h1 className="text-5xl md:text-6xl font-extrabold mb-4 tracking-tight z-10 drop-shadow-md">
          PEGASUS <span className="text-orange-300">CHAPTER</span>
        </h1>
        <p className="text-xl text-red-100 max-w-md z-10 font-light leading-relaxed">
          Nơi hội tụ của những chiến binh tinh nhuệ và những con số may mắn hàng tuần.
        </p>
      </div>

      <div className="md:w-1/2 flex items-center justify-center p-6 bg-white/5 backdrop-blur-sm relative">
        <div className="w-full max-w-md bg-white/95 backdrop-blur-xl rounded-3xl shadow-2xl p-8 border border-white/40 relative z-20">
          <div className="flex items-center justify-center mb-6">
            <div className="p-3 bg-red-50 rounded-full text-red-600 shadow-inner">
               <Flame size={40} fill="currentColor" className="animate-pulse" />
            </div>
          </div>
          <h2 className="text-3xl font-bold text-center text-gray-800 mb-8 tracking-tight">
            {isRegistering ? 'Gia Nhập Đội Ngũ' : 'Chào Mừng Trở Lại'}
          </h2>
          {error && (
            <div className="mb-6 p-4 bg-red-50 text-red-700 rounded-xl flex items-center text-sm border border-red-100">
              <AlertCircle size={18} className="mr-3 flex-shrink-0" /> {error}
            </div>
          )}
          <form onSubmit={handleSubmit} className="space-y-5">
            {isRegistering && (
              <div>
                <label className="block text-sm font-bold text-gray-700 mb-2 ml-1">Họ và Tên</label>
                <div className="relative group">
                  <User className="absolute left-3 top-3.5 text-gray-400 group-focus-within:text-orange-500 transition-colors" size={20} />
                  <input
                    type="text"
                    className="w-full pl-11 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:bg-white focus:ring-2 focus:ring-orange-500 focus:border-orange-500 outline-none transition-all shadow-sm"
                    placeholder="VD: Nguyễn Văn A"
                    value={formData.name}
                    onChange={e => setFormData({...formData, name: e.target.value})}
                  />
                </div>
              </div>
            )}
            <div>
              <label className="block text-sm font-bold text-gray-700 mb-2 ml-1">Email</label>
              <div className="relative group">
                <Users className="absolute left-3 top-3.5 text-gray-400 group-focus-within:text-orange-500 transition-colors" size={20} />
                <input
                  type="email"
                  className="w-full pl-11 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:bg-white focus:ring-2 focus:ring-orange-500 focus:border-orange-500 outline-none transition-all shadow-sm"
                  placeholder="name@example.com"
                  value={formData.email}
                  onChange={e => setFormData({...formData, email: e.target.value})}
                />
              </div>
            </div>
            <div>
              <label className="block text-sm font-bold text-gray-700 mb-2 ml-1">Mật khẩu</label>
              <div className="relative group">
                <Lock className="absolute left-3 top-3.5 text-gray-400 group-focus-within:text-orange-500 transition-colors" size={20} />
                <input
                  type="password"
                  className="w-full pl-11 pr-4 py-3 bg-gray-50 border border-gray-200 rounded-xl focus:bg-white focus:ring-2 focus:ring-orange-500 focus:border-orange-500 outline-none transition-all shadow-sm"
                  placeholder="••••••••"
                  value={formData.password}
                  onChange={e => setFormData({...formData, password: e.target.value})}
                />
              </div>
            </div>
            <button 
              type="submit"
              className="w-full mt-4 bg-gradient-to-r from-red-600 to-orange-500 hover:from-red-700 hover:to-orange-600 text-white font-bold py-3.5 rounded-xl shadow-lg shadow-orange-500/30 transform transition-all active:scale-95 flex items-center justify-center group"
            >
              {isRegistering ? 'Đăng Ký Ngay' : 'Đăng Nhập'} 
              <ChevronRight className="ml-2 group-hover:translate-x-1 transition-transform" size={20} />
            </button>
          </form>
          <div className="mt-8 text-center text-sm text-gray-600">
            {isRegistering ? 'Đã là chiến binh?' : 'Chưa có tài khoản?'} 
            <button 
              onClick={() => setIsRegistering(!isRegistering)}
              className="ml-2 text-red-600 font-bold hover:underline hover:text-red-700 transition"
            >
              {isRegistering ? 'Đăng nhập' : 'Tham gia ngay'}
            </button>
          </div>
        </div>
      </div>
    </div>
  );
};

// 3. MEMBER DASHBOARD
const MemberDashboard = ({ user, onLogout }) => {
  const currentNumStr = user.currentNumber ? user.currentNumber.toString().padStart(2, '0') : '--';
  const mascotAction = user.currentNumber ? 'celebrate' : 'idle';

  return (
    <div className="min-h-screen bg-gray-50 flex flex-col font-sans">
      <header className="bg-white/80 backdrop-blur-md shadow-sm sticky top-0 z-50 border-b border-gray-100">
        <div className="max-w-7xl mx-auto px-4 py-3 flex justify-between items-center">
          <div className="flex items-center space-x-2">
            <div className="bg-orange-100 p-2 rounded-lg text-orange-600">
              <Flame size={24} fill="currentColor" />
            </div>
            <span className="font-extrabold text-xl text-gray-800 tracking-tight hidden md:block">
              Pegasus <span className="text-orange-600">Chapter</span>
            </span>
          </div>
          <div className="flex items-center space-x-4">
             <div className="hidden md:flex flex-col items-end mr-2">
                <span className="text-gray-800 font-bold text-sm">{user.name}</span>
                <span className="text-gray-500 text-xs uppercase tracking-wide">Member</span>
             </div>
            <button onClick={onLogout} className="p-2.5 bg-gray-100 text-gray-600 rounded-xl hover:bg-red-50 hover:text-red-600 transition-colors" title="Đăng xuất">
              <LogOut size={20} />
            </button>
          </div>
        </div>
      </header>

      <main className="flex-1 max-w-5xl mx-auto w-full p-4 md:p-8 space-y-8">
        <div className="bg-gradient-to-br from-red-600 via-red-500 to-orange-500 rounded-[2.5rem] p-8 md:p-12 text-white shadow-2xl shadow-orange-500/30 relative overflow-hidden flex flex-col md:flex-row items-center justify-between min-h-[350px]">
          <div className="absolute top-0 right-0 w-96 h-96 bg-white opacity-[0.07] rounded-full blur-3xl -translate-y-1/2 translate-x-1/2 pointer-events-none"></div>
          <div className="absolute bottom-0 left-0 w-64 h-64 bg-yellow-400 opacity-[0.1] rounded-full blur-3xl translate-y-1/2 -translate-x-1/4 pointer-events-none"></div>
          
          <div className="z-10 flex-1 text-center md:text-left mb-8 md:mb-0 relative">
            <div className="inline-flex items-center space-x-2 bg-white/20 backdrop-blur-sm px-4 py-1.5 rounded-full text-sm font-medium mb-4 border border-white/20">
              <RefreshCw size={14} className="animate-spin-slow" />
              <span>Cập nhật mới nhất: {formatDateVN(user.allocatedAt)}</span>
            </div>
            <h2 className="text-3xl md:text-4xl font-bold mb-2">Số May Mắn Của Bạn</h2>
            <p className="text-red-100 mb-8 max-w-md opacity-90">Hãy mang theo nguồn năng lượng này để bứt phá trong tuần mới!</p>
            
            <div className="inline-block relative group">
              <div className="absolute inset-0 bg-white blur-xl opacity-20 group-hover:opacity-40 transition-opacity rounded-full"></div>
              <span className="relative z-10 text-[10rem] leading-none font-black tracking-tighter drop-shadow-2xl text-transparent bg-clip-text bg-gradient-to-b from-white to-orange-100">
                {currentNumStr}
              </span>
              {user.currentNumber && (
                <div className="absolute -top-6 -right-10 animate-bounce delay-75">
                  <Trophy className="text-yellow-300 drop-shadow-lg w-16 h-16 transform rotate-12" fill="currentColor" />
                </div>
              )}
            </div>
          </div>
          <div className="z-10 md:mr-4 flex-shrink-0">
            <Mascot action={mascotAction} />
          </div>
        </div>

        <div className="bg-white rounded-3xl shadow-xl shadow-gray-200/50 p-8 border border-gray-100">
          <div className="flex items-center justify-between mb-8">
            <div className="flex items-center space-x-3">
              <div className="p-2.5 bg-blue-50 text-blue-600 rounded-xl">
                 <History size={24} />
              </div>
              <h3 className="text-2xl font-bold text-gray-800">Lịch Sử Cấp Số</h3>
            </div>
            <span className="text-sm font-medium text-gray-400 bg-gray-50 px-3 py-1 rounded-lg">
              {user.history.length} lần nhận số
            </span>
          </div>

          {user.history.length === 0 ? (
            <div className="flex flex-col items-center justify-center py-12 text-gray-400 bg-gray-50 rounded-2xl border border-dashed border-gray-200">
              <FileSpreadsheet size={48} className="mb-3 opacity-50" />
              <p>Chưa có lịch sử cấp số nào.</p>
            </div>
          ) : (
            <div className="grid grid-cols-4 sm:grid-cols-6 md:grid-cols-8 lg:grid-cols-10 gap-4">
              {user.history.map((num, idx) => (
                <div key={idx} className="group aspect-square flex flex-col items-center justify-center bg-white rounded-2xl border border-gray-100 shadow-sm hover:shadow-md hover:border-orange-200 hover:bg-orange-50 transition-all cursor-default relative overflow-hidden">
                  <span className="text-[10px] text-gray-300 absolute top-2 right-2 group-hover:text-orange-300">#{user.history.length - idx}</span>
                  <span className="text-2xl font-bold text-gray-700 group-hover:text-orange-600 group-hover:scale-110 transition-transform">
                    {num.toString().padStart(2, '0')}
                  </span>
                </div>
              ))}
            </div>
          )}
        </div>
      </main>
    </div>
  );
};

// 4. ADMIN DASHBOARD
const AdminDashboard = ({ users, onLogout, onGenerate, onSyncSheets, isRegistrationOpen, toggleRegistration }) => {
  const [loading, setLoading] = useState(false);
  const [syncing, setSyncing] = useState(false);

  const handleGenerate = () => {
    // Logic xác định tuần áp dụng: 
    // Nếu hôm nay là T7 hoặc CN -> Áp dụng cho tuần sau.
    // Nếu hôm nay là T2-T6 -> Áp dụng ngay cho tuần này.
    const today = new Date().getDay(); // 0-Sun, 6-Sat
    const isWeekend = today === 6 || today === 0;
    const weekLabel = isWeekend ? "TUẦN SAU" : "NGAY TUẦN NÀY";

    if (window.confirm(`BẠN MUỐN CẤP SỐ CHO ${weekLabel}?\n(Hành động này sẽ cập nhật số và thời gian cấp cho tất cả thành viên)`)) {
      setLoading(true);
      setTimeout(() => {
        onGenerate();
        setLoading(false);
      }, 1500); 
    }
  };

  const handleSync = () => {
    setSyncing(true);
    setTimeout(() => {
      onSyncSheets();
      setSyncing(false);
    }, 2000);
  };

  return (
    <div className="min-h-screen bg-gray-50 font-sans">
      <div className="flex h-screen overflow-hidden">
        <aside className="w-72 bg-white shadow-2xl z-20 hidden md:flex flex-col border-r border-gray-100">
          <div className="p-8 border-b border-gray-100 flex items-center space-x-3">
            <div className="bg-red-600 text-white p-2 rounded-lg">
               <Shield size={24} />
            </div>
            <span className="font-extrabold text-xl text-gray-800 tracking-tight">Pegasus Admin</span>
          </div>
          <nav className="flex-1 p-6 space-y-3">
            <div className="flex items-center space-x-3 p-3.5 bg-red-50 text-red-700 rounded-xl font-bold cursor-pointer transition-colors">
              <Users size={20} /> <span>Quản Lý Thành Viên</span>
            </div>
            <div className="flex items-center space-x-3 p-3.5 text-gray-500 hover:bg-gray-50 hover:text-gray-800 rounded-xl font-medium cursor-not-allowed opacity-60">
              <Settings size={20} /> <span>Cấu hình Hệ thống</span>
            </div>
          </nav>
          <div className="p-6 border-t border-gray-100">
             <button onClick={onLogout} className="flex items-center justify-center space-x-2 text-gray-500 hover:text-red-600 hover:bg-red-50 w-full py-3 rounded-xl transition-all font-medium">
               <LogOut size={20} /> <span>Đăng xuất an toàn</span>
             </button>
          </div>
        </aside>

        <main className="flex-1 overflow-y-auto p-6 md:p-10">
          <div className="flex justify-between items-center mb-10">
            <div>
              <h1 className="text-3xl font-bold text-gray-800 mb-1">Dashboard Quản Trị</h1>
              <p className="text-gray-500 text-sm">Chào mừng quay trở lại, Admin!</p>
            </div>
            <div className="bg-white px-5 py-2.5 rounded-xl shadow-sm border border-gray-100 flex items-center space-x-3">
              <div className="w-2 h-2 rounded-full bg-green-500 animate-pulse"></div>
              <span className="text-sm text-gray-500">Thành viên Active: <span className="font-bold text-gray-800 text-base ml-1">{users.filter(u => u.role !== 'admin').length}</span></span>
            </div>
          </div>

          <div className="grid grid-cols-1 md:grid-cols-3 gap-6 mb-10">
            {/* Card 1: Generate */}
            <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 hover:shadow-md transition-shadow group">
              <div className="flex justify-between items-start mb-5">
                <div className="p-3.5 bg-orange-50 rounded-xl text-orange-600 group-hover:bg-orange-600 group-hover:text-white transition-colors">
                  <RefreshCw size={26} className={loading ? 'animate-spin' : ''} />
                </div>
                <span className="text-[10px] font-bold bg-gray-100 text-gray-500 px-2.5 py-1 rounded-md uppercase tracking-wide">Thứ 7</span>
              </div>
              <h3 className="text-lg font-bold text-gray-800 mb-1">Cấp Số Định Danh</h3>
              <p className="text-sm text-gray-500 mb-6 leading-relaxed">Chạy thuật toán Randomize & Allocate số. Tự động xác định tuần dựa trên thời gian thực.</p>
              <button 
                onClick={handleGenerate}
                disabled={loading}
                className="w-full bg-red-600 text-white py-2.5 rounded-xl font-bold hover:bg-red-700 active:scale-95 transition-all disabled:opacity-50 disabled:active:scale-100 shadow-lg shadow-red-200"
              >
                {loading ? 'Đang xử lý thuật toán...' : 'Kích hoạt Cấp Số'}
              </button>
            </div>

            {/* Card 2: Sync Sheets */}
            <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 hover:shadow-md transition-shadow group">
              <div className="flex justify-between items-start mb-5">
                <div className="p-3.5 bg-green-50 rounded-xl text-green-600 group-hover:bg-green-600 group-hover:text-white transition-colors">
                  <FileSpreadsheet size={26} />
                </div>
                <span className="text-[10px] font-bold bg-green-50 text-green-700 px-2.5 py-1 rounded-md uppercase tracking-wide">API Connected</span>
              </div>
              <h3 className="text-lg font-bold text-gray-800 mb-1">Đồng Bộ Báo Cáo</h3>
              <p className="text-sm text-gray-500 mb-6 leading-relaxed">Ghi dữ liệu (gồm cả ngày giờ cấp) lên Google Sheets.</p>
              <button 
                onClick={handleSync}
                disabled={syncing}
                className="w-full bg-white border-2 border-gray-100 text-gray-700 py-2.5 rounded-xl font-bold hover:border-green-500 hover:text-green-600 active:scale-95 transition-all flex items-center justify-center space-x-2"
              >
                {syncing ? <span>Đang đẩy dữ liệu...</span> : <span>Sync to Sheets</span>}
              </button>
            </div>

            {/* Card 3: Settings */}
            <div className="bg-white p-6 rounded-2xl shadow-sm border border-gray-100 hover:shadow-md transition-shadow group">
               <div className="flex justify-between items-start mb-5">
                <div className={`p-3.5 rounded-xl transition-colors ${isRegistrationOpen ? 'bg-blue-50 text-blue-600 group-hover:bg-blue-600 group-hover:text-white' : 'bg-gray-100 text-gray-500'}`}>
                  <Settings size={26} />
                </div>
                <div className={`w-3 h-3 rounded-full mt-1 ${isRegistrationOpen ? 'bg-green-500 shadow-[0_0_8px_rgba(34,197,94,0.6)]' : 'bg-red-500'}`}></div>
              </div>
              <h3 className="text-lg font-bold text-gray-800 mb-1">Cổng Đăng Ký</h3>
              <p className="text-sm text-gray-500 mb-6 leading-relaxed">Trạng thái hiện tại: <span className="font-medium text-gray-800">{isRegistrationOpen ? 'Đang mở (Public)' : 'Đã đóng (Private)'}</span></p>
              <button 
                onClick={toggleRegistration}
                className={`w-full py-2.5 rounded-xl font-bold transition-all active:scale-95 ${isRegistrationOpen ? 'bg-blue-600 text-white hover:bg-blue-700 shadow-lg shadow-blue-200' : 'bg-gray-100 text-gray-600 hover:bg-gray-200'}`}
              >
                {isRegistrationOpen ? 'Khóa Cổng Đăng Ký' : 'Mở Cổng Đăng Ký'}
              </button>
            </div>
          </div>

          <div className="bg-white rounded-2xl shadow-sm border border-gray-200 overflow-hidden">
            <div className="px-6 py-5 border-b border-gray-100 bg-gray-50/50 flex justify-between items-center">
              <h3 className="font-bold text-gray-800 text-lg">Danh Sách Thành Viên & Số Hiện Tại</h3>
              <button className="text-sm text-red-600 font-medium hover:underline">Xem tất cả</button>
            </div>
            <div className="overflow-x-auto">
              <table className="w-full text-left">
                <thead>
                  <tr className="text-gray-400 border-b border-gray-100 text-xs uppercase tracking-wider bg-white">
                    <th className="px-6 py-4 font-semibold">Thành viên</th>
                    <th className="px-6 py-4 font-semibold">Email</th>
                    <th className="px-6 py-4 font-semibold text-center">Số Tuần Này</th>
                    <th className="px-6 py-4 font-semibold">Thời Gian Cấp</th>
                  </tr>
                </thead>
                <tbody className="divide-y divide-gray-50">
                  {users.filter(u => u.role !== 'admin').map(user => (
                    <tr key={user.id} className="hover:bg-gray-50/80 transition-colors group">
                      <td className="px-6 py-4 font-medium text-gray-800 flex items-center space-x-3">
                         <div className="w-10 h-10 rounded-full bg-gradient-to-br from-red-500 to-orange-400 flex items-center justify-center text-white text-sm font-bold shadow-sm ring-2 ring-white">
                           {user.name.charAt(0)}
                         </div>
                         <span className="group-hover:text-red-600 transition-colors">{user.name}</span>
                      </td>
                      <td className="px-6 py-4 text-gray-500 text-sm font-medium">{user.email}</td>
                      <td className="px-6 py-4 text-center">
                        {user.currentNumber ? (
                          <span className="inline-block px-4 py-1.5 bg-red-100 text-red-700 rounded-full font-bold shadow-sm">
                            {user.currentNumber.toString().padStart(2,'0')}
                          </span>
                        ) : (
                          <span className="text-gray-300 font-medium text-xl">--</span>
                        )}
                      </td>
                      <td className="px-6 py-4 text-gray-500 text-sm font-mono flex items-center space-x-2">
                        {user.allocatedAt ? (
                          <>
                            <Clock size={14} className="text-gray-400" />
                            <span>{formatDateVN(user.allocatedAt)}</span>
                          </>
                        ) : (
                          <span className="text-gray-300">--/--/----</span>
                        )}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </main>
      </div>
    </div>
  );
};

// --- MAIN APP ---
const App = () => {
  const [users, setUsers] = useState(INITIAL_USERS);
  const [currentUser, setCurrentUser] = useState(null);
  const [isRegOpen, setIsRegOpen] = useState(true);
  
  const handleLogin = (email, password) => {
    const user = users.find(u => u.email === email);
    if (user) {
      if (password === 'admin123' && user.role === 'admin') setCurrentUser(user);
      else if (password.length > 0) setCurrentUser(user);
      else alert("Sai mật khẩu (Demo: nhập bất kỳ)");
    } else {
      alert("Không tìm thấy user!");
    }
  };

  const handleRegister = (data) => {
    if (!isRegOpen) {
      alert("Hệ thống đang tạm khóa đăng ký!");
      return;
    }
    const newUser = {
      id: generateId(),
      email: data.email,
      name: data.name,
      role: 'member',
      history: [],
      currentNumber: null,
      allocatedAt: null
    };
    setUsers([...users, newUser]);
    setCurrentUser(newUser);
  };

  const handleLogout = () => setCurrentUser(null);

  const runWeeklyAllocation = () => {
    const members = users.filter(u => u.role === 'member');
    const admins = users.filter(u => u.role === 'admin');
    
    const { updatedUsers, success, errors } = allocateWeeklyNumbers(members);
    
    if (!success) {
      alert("Cảnh báo: " + errors.join('\n'));
    }

    const mergedUsers = [...admins, ...updatedUsers];
    setUsers(mergedUsers);
  };

  const syncToGoogleSheets = () => {
    const payload = users
      .filter(u => u.role === 'member')
      .map(u => ({
        timestamp: new Date().toISOString(),
        name: u.name,
        email: u.email,
        currentNumber: u.currentNumber,
        allocatedAt: u.allocatedAt, // Thêm thời gian cấp vào payload
        history: u.history.join(',')
      }));
    
    console.log(">>> Đang gửi đến Google Sheets API Endpoint:", payload);
    alert("Đã đồng bộ thành công " + payload.length + " bản ghi lên Google Sheets!");
  };

  if (!currentUser) {
    return <LoginScreen onLogin={handleLogin} onRegister={handleRegister} />;
  }

  if (currentUser.role === 'admin') {
    return (
      <AdminDashboard 
        users={users} 
        onLogout={handleLogout}
        onGenerate={runWeeklyAllocation}
        onSyncSheets={syncToGoogleSheets}
        isRegistrationOpen={isRegOpen}
        toggleRegistration={() => setIsRegOpen(!isRegOpen)}
      />
    );
  }

  const activeUserLive = users.find(u => u.id === currentUser.id) || currentUser;

  return <MemberDashboard user={activeUserLive} onLogout={handleLogout} />;
};

export default App;
