


<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>HSK 3.0 대비 과제방</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    
    <!-- Babel -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>

    <style>
        body { font-family: 'Pretendard', 'Malgun Gothic', sans-serif; background-color: #f3f4f6; }
        .hide-scrollbar::-webkit-scrollbar { display: none; }
        .hide-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
        
        /* 아날로그 채점 애니메이션 */
        @keyframes drawLine {
            to { stroke-dashoffset: 0; }
        }
        @keyframes popIn {
            0% { transform: scale(0.3) rotate(-15deg); opacity: 0; }
            50% { transform: scale(1.15) rotate(5deg); opacity: 1; }
            100% { transform: scale(1) rotate(0deg); opacity: 0.85; }
        }
        .mark-animate {
            animation: drawLine 0.4s ease-out forwards, popIn 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
            transform-origin: center;
        }
        .mark-animate-fast {
            animation: drawLine 0.2s ease-out forwards, popIn 0.3s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
            transform-origin: center;
        }
    </style>
</head>
<body>
    <div id="root"></div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, getDoc, onSnapshot, deleteDoc, updateDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase Configuration injected by environment
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const app = Object.keys(firebaseConfig).length > 0 ? initializeApp(firebaseConfig) : null;
        const auth = app ? getAuth(app) : null;
        const db = app ? getFirestore(app) : null;
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'hsk-academy-mock-id';

        window.firebaseServices = { auth, db, appId, signInAnonymously, signInWithCustomToken, onAuthStateChanged, signOut, collection, doc, setDoc, getDoc, onSnapshot, deleteDoc, updateDoc };
    </script>

    <script type="text/babel" data-type="module">
        const { useState, useEffect, useRef } = React;

        // --- Icons Component (Inline SVG for Lucide) ---
        const Icon = ({ name, size = 24, className = "", strokeWidth = 2 }) => {
            const icons = {
                Archive: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><rect width="20" height="5" x="2" y="4" rx="1"/><path d="M4 9v9a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V9"/><path d="M10 13h4"/></svg>,
                BookOpen: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"/><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"/></svg>,
                Headphones: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M3 14h3a2 2 0 0 1 2 2v3a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-7a9 9 0 0 1 18 0v7a2 2 0 0 1-2 2h-1a2 2 0 0 1-2-2v-3a2 2 0 0 1 2-2h3"/></svg>,
                PenTool: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m12 19 7-7 3 3-7 7-3-3z"/><path d="m18 13-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"/><path d="m2 2 7.586 7.586"/><circle cx="11" cy="11" r="2"/></svg>,
                MessageSquare: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/></svg>,
                LogOut: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><polyline points="16 17 21 12 16 7"/><line x1="21" x2="9" y1="12" y2="12"/></svg>,
                CheckCircle: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/></svg>,
                Lock: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><rect width="18" height="11" x="3" y="11" rx="2" ry="2"/><path d="M7 11V7a5 5 0 0 1 10 0v4"/></svg>,
                PlayCircle: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><circle cx="12" cy="12" r="10"/><polygon points="10 8 16 12 10 16 10 8"/></svg>,
                Activity: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/></svg>,
                ShieldAlert: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M20 13c0 5-3.5 7.5-7.66 8.95a1 1 0 0 1-.67-.01C7.5 20.5 4 18 4 13V6a1 1 0 0 1 1-1c2-1 4-3 5.96-8.9a1 1 0 0 1 1.08 0C14 2 16 4 18 5a1 1 0 0 1 1 1z"/><path d="M12 8v4"/><path d="M12 16h.01"/></svg>,
                LayoutDashboard: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><rect width="7" height="9" x="3" y="3" rx="1"/><rect width="7" height="5" x="14" y="3" rx="1"/><rect width="7" height="9" x="14" y="12" rx="1"/><rect width="7" height="5" x="3" y="16" rx="1"/></svg>,
                FileText: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M15 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7Z"/><path d="M14 2v4a2 2 0 0 0 2 2h4"/><path d="M10 9H8"/><path d="M16 13H8"/><path d="M16 17H8"/></svg>,
                Users: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M16 21v-2a4 4 0 0 0-4-4H6a4 4 0 0 0-4 4v2"/><circle cx="9" cy="7" r="4"/><path d="M22 21v-2a4 4 0 0 0-3-3.87"/><path d="M16 3.13a4 4 0 0 1 0 7.75"/></svg>,
                Eye: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M2 12s3-7 10-7 10 7 10 7-3 7-10 7-10-7-10-7Z"/><circle cx="12" cy="12" r="3"/></svg>,
                Plus: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M5 12h14"/><path d="M12 5v14"/></svg>,
                Trash2: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M3 6h18"/><path d="M19 6v14c0 1-1 2-2 2H7c-1 0-2-1-2-2V6"/><path d="M8 6V4c0-1 1-2 2-2h4c1 0 2 1 2 2v2"/><line x1="10" y1="11" x2="10" y2="17"/><line x1="14" y1="11" x2="14" y2="17"/></svg>,
                Edit3: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M12 20h9"/><path d="M16.5 3.5a2.121 2.121 0 0 1 3 3L7 19l-4 1 1-4L16.5 3.5z"/></svg>,
                ChevronLeft: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m15 18-6-6 6-6"/></svg>,
                ChevronRight: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m9 18 6-6-6-6"/></svg>,
                X: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M18 6 6 18"/><path d="m6 6 12 12"/></svg>,
                Sparkles: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="m12 3-1.912 5.813a2 2 0 0 1-1.275 1.275L3 12l5.813 1.912a2 2 0 0 1 1.275 1.275L12 21l1.912-5.813a2 2 0 0 1 1.275-1.275L21 12l-5.813-1.912a2 2 0 0 1-1.275-1.275L12 3Z"/><path d="M5 3v4"/><path d="M19 17v4"/><path d="M3 5h4"/><path d="M17 19h4"/></svg>,
                Mic: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" x2="12" y1="19" y2="22"/></svg>,
                Square: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><rect width="18" height="18" x="3" y="3" rx="2"/></svg>,
                Volume2: <svg xmlns="http://www.w3.org/2000/svg" width={size} height={size} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth={strokeWidth} strokeLinecap="round" strokeLinejoin="round" className={className}><polygon points="11 5 6 9 2 9 2 15 6 15 11 19 11 5"/><path d="M15.54 8.46a5 5 0 0 1 0 7.07"/><path d="M19.07 4.93a10 10 0 0 1 0 14.14"/></svg>
            };
            return icons[name] || null;
        };

        // --- Main App Component ---
        const App = () => {
            const [user, setUser] = useState(null);
            const [isAuthLoading, setIsAuthLoading] = useState(true);
            const [academyLoggedIn, setAcademyLoggedIn] = useState(false);
            const [userInfo, setUserInfo] = useState({ name: '', email: '', id: '', isAdmin: false, level: '6급' });
            const [activeMenu, setActiveMenu] = useState('listening');

            const [submissions, setSubmissions] = useState([]);
            const [tasks, setTasks] = useState([]);
            const [usersList, setUsersList] = useState([]);

            useEffect(() => {
                if (!user) return;
                const { db, appId, collection, onSnapshot } = window.firebaseServices;
                if (!db) return;

                // 과제 콘텐츠(Tasks) 데이터베이스 실시간 리스너
                const tasksRef = collection(db, 'artifacts', appId, 'public', 'data', 'tasks');
                const unsubTasks = onSnapshot(tasksRef, (snapshot) => {
                    const fetchedTasks = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    fetchedTasks.sort((a, b) => b.id - a.id); // 최신순 정렬
                    setTasks(fetchedTasks);
                }, (error) => console.error("Tasks fetch error:", error));

                // 학생 제출물(Submissions) 데이터베이스 실시간 리스너
                const subsRef = collection(db, 'artifacts', appId, 'public', 'data', 'submissions');
                const unsubSubs = onSnapshot(subsRef, (snapshot) => {
                    const fetchedSubs = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    fetchedSubs.sort((a, b) => new Date(b.submittedAt) - new Date(a.submittedAt)); // 최신 제출순 정렬
                    setSubmissions(fetchedSubs);
                }, (error) => console.error("Subs fetch error:", error));

                // 학생 목록(Users) 데이터베이스 실시간 리스너
                const usersRef = collection(db, 'artifacts', appId, 'public', 'data', 'users');
                const unsubUsers = onSnapshot(usersRef, (snapshot) => {
                    const fetchedUsers = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    setUsersList(fetchedUsers);
                }, (error) => console.error("Users fetch error:", error));

                return () => { unsubTasks(); unsubSubs(); unsubUsers(); };
            }, [user]);

            const handleAddSubmission = async (data) => {
                const { db, appId, doc, setDoc } = window.firebaseServices;
                if (!db) return;
                const newId = `SUB-${Date.now()}-${Math.floor(Math.random() * 1000)}`;
                const newSub = {
                    id: newId,
                    studentId: userInfo.id,
                    studentName: userInfo.name,
                    submittedAt: new Date().toISOString(),
                    status: 'pending',
                    ...data
                };
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'submissions', newId), newSub);
            };

            const handleApproveSubmission = async (id) => {
                const { db, appId, doc, updateDoc } = window.firebaseServices;
                if (!db) return;
                await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'submissions', id), { status: 'approved', approvedAt: new Date().toISOString() });
            };

            const handleSaveNewTask = async (newTaskData) => {
                const { db, appId, doc, setDoc } = window.firebaseServices;
                if (!db) return;
                const newId = newTaskData.id || Date.now().toString();
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'tasks', newId.toString()), { ...newTaskData, id: newId.toString() });
            };

            const handleDeleteTask = async (id) => {
                const { db, appId, doc, deleteDoc } = window.firebaseServices;
                if (!db) return;
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'tasks', id.toString()));
            };

            const handleUpdateTaskStatus = async (id, currentStatus) => {
                const { db, appId, doc, updateDoc } = window.firebaseServices;
                if (!db) return;
                await updateDoc(doc(db, 'artifacts', appId, 'public', 'data', 'tasks', id.toString()), { status: currentStatus === '활성' ? '비활성' : '활성' });
            };

            const handleSaveUser = async (userData) => {
                const { db, appId, doc, setDoc } = window.firebaseServices;
                if (!db) return;
                const newId = userData.id || `STU-${Date.now().toString().slice(-6)}`;
                await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'users', newId), { ...userData, id: newId });
            };

            const handleDeleteUser = async (id) => {
                const { db, appId, doc, deleteDoc } = window.firebaseServices;
                if (!db) return;
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'users', id));
            };

            useEffect(() => {
                const initAuth = async () => {
                    const { auth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } = window.firebaseServices;
                    if (!auth) {
                        setIsAuthLoading(false);
                        return;
                    }

                    try {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    } catch (error) {
                        console.error("Auth error:", error);
                    }

                    const unsubscribe = onAuthStateChanged(auth, (u) => {
                        setUser(u);
                        setIsAuthLoading(false);
                    });
                    return () => unsubscribe();
                };
                initAuth();
            }, []);

            const handleLogin = (email, password) => {
                if (email === "yijoonho@hsktc.com" && password === "3300") {
                    setUserInfo({ name: "이준호", email, id: "ADMIN-001", isAdmin: true, level: '관리자' });
                    setAcademyLoggedIn(true);
                    setActiveMenu('admin');
                    return true;
                } else {
                    const foundUser = usersList.find(u => u.email === email && u.password === password);
                    if (foundUser) {
                        setUserInfo({ name: foundUser.name, email: foundUser.email, id: foundUser.id, isAdmin: false, level: foundUser.level || '6급' });
                        setAcademyLoggedIn(true);
                        setActiveMenu('listening');
                        return true;
                    } else if(password === "HSK300" && email.trim() !== "") {
                        const name = email.split('@')[0] || "학생"; 
                        setUserInfo({ name, email, id: `STU-${Math.floor(Math.random()*10000)}`, isAdmin: false, level: '6급' });
                        setAcademyLoggedIn(true);
                        setActiveMenu('listening');
                        return true;
                    }
                }
                return false;
            };

            const handleLogout = () => {
                setAcademyLoggedIn(false);
                setUserInfo({name: '', email: '', id: '', isAdmin: false, level: '6급'});
            };

            if (isAuthLoading) {
                return <div className="min-h-screen flex items-center justify-center bg-gray-100"><div className="animate-spin rounded-full h-12 w-12 border-b-2 border-indigo-600"></div></div>;
            }

            if (!academyLoggedIn) {
                return <LoginScreen onLogin={handleLogin} />;
            }

            return (
                <div className="flex h-screen bg-gray-50 overflow-hidden text-gray-800">
                    <Sidebar activeMenu={activeMenu} setActiveMenu={setActiveMenu} handleLogout={handleLogout} userName={userInfo.name} userInfo={userInfo} isAdmin={userInfo.isAdmin} />
                    <main className="flex-1 overflow-y-auto p-6 md:p-10">
                        <header className="mb-8 border-b pb-4 flex justify-between items-center">
                            <div>
                                <h1 className="text-3xl font-bold text-gray-900">HSK 3.0 대비 과제방</h1>
                                <p className="text-gray-500 mt-1">오늘의 학습 목표를 달성해 보세요.</p>
                            </div>
                            <div className="hidden md:flex items-center space-x-2 bg-white px-4 py-2 rounded-full shadow-sm border border-gray-100">
                                <span className="w-2 h-2 bg-green-500 rounded-full animate-pulse"></span>
                                <span className="text-sm font-medium text-gray-600">{userInfo.name} {userInfo.isAdmin ? '관리자' : '학생'} 접속중</span>
                            </div>
                        </header>
                        
                        <div className="bg-white rounded-xl shadow-sm border border-gray-200 min-h-[70vh]">
                            {activeMenu === 'listening' && !userInfo.isAdmin && <ListeningModule />}
                            {activeMenu === 'reading' && !userInfo.isAdmin && <ReadingModule setActiveMenu={setActiveMenu} onSubmit={handleAddSubmission} tasks={tasks} submissions={submissions} userInfo={userInfo} />}
                            {activeMenu === 'writing' && !userInfo.isAdmin && <WritingRoleplayModule studentName={userInfo.name}/>}
                            {activeMenu === 'archive' && !userInfo.isAdmin && <ArchiveDashboard userInfo={userInfo} submissions={submissions} />}
                            {activeMenu === 'admin' && userInfo.isAdmin && <AdminDashboard submissions={submissions} onApprove={handleApproveSubmission} tasks={tasks} onSaveNewTask={handleSaveNewTask} onDeleteTask={handleDeleteTask} onUpdateTaskStatus={handleUpdateTaskStatus} usersList={usersList} onSaveUser={handleSaveUser} onDeleteUser={handleDeleteUser} />}
                        </div>
                    </main>
                </div>
            );
        };

        const LoginScreen = ({ onLogin }) => {
            const [email, setEmail] = useState('');
            const [password, setPassword] = useState('');
            const [error, setError] = useState('');

            const handleSubmit = () => {
                const success = onLogin(email, password);
                if (!success) {
                    setError('학원 계정(이메일)과 정확한 비밀번호를 확인해주세요.');
                }
            };

            return (
                <div className="min-h-screen flex flex-col items-center justify-center bg-[#0b1121] px-4 font-sans">
                    <div className="text-center mb-8 flex flex-col items-center">
                        <div className="w-14 h-14 bg-[#1e293b]/50 rounded-2xl flex items-center justify-center mb-6 border border-slate-700/50 shadow-[0_0_15px_rgba(59,130,246,0.15)]">
                            <Icon name="Archive" size={26} className="text-[#3b82f6]" />
                        </div>
                        <h1 className="text-3xl font-bold text-white tracking-tight mb-3">
                            HSKTC <span className="text-[#3b82f6] font-normal">Archive</span>
                        </h1>
                        <p className="text-[10px] text-gray-400 tracking-[0.25em] mb-10 font-medium">
                            HSK TRAINING CENTER ARCHIVE
                        </p>
                        <p className="text-gray-200 text-[15px] leading-relaxed">
                            "당신의 모든 학습 기록이<br/>합격의 데이터가 됩니다."
                        </p>
                    </div>

                    <div className="bg-white max-w-[360px] w-full rounded-[1.25rem] shadow-2xl overflow-hidden">
                        <div className="bg-[#f4f5f7] px-4 py-3.5 flex items-center justify-center relative border-b border-gray-100">
                            <div className="absolute left-4 flex gap-1.5">
                                <div className="w-2.5 h-2.5 bg-[#ff5f56] rounded-full"></div>
                            </div>
                            <div className="flex items-center gap-1.5 text-gray-500">
                                <Icon name="Lock" size={13} strokeWidth={2.5} />
                                <span className="text-[10px] font-bold tracking-widest mt-0.5">AUTHORIZED PERSONNEL ONLY</span>
                            </div>
                        </div>

                        <div className="p-7 space-y-5">
                            {error && <div className="bg-rose-50 text-rose-500 text-xs font-bold p-3 rounded-lg text-center animate-in fade-in zoom-in-95">{error}</div>}
                            <div>
                                <label className="block text-[13px] font-bold text-gray-700 mb-2">학원 계정 (이메일)</label>
                                <input 
                                    type="email" 
                                    value={email} 
                                    onChange={(e) => {setEmail(e.target.value); setError('');}} 
                                    placeholder="student@hsktc.com" 
                                    className="w-full px-4 py-3.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-[15px] placeholder-gray-400" 
                                />
                            </div>
                            <div>
                                <label className="block text-[13px] font-bold text-gray-700 mb-2">접속 비밀번호</label>
                                <input 
                                    type="password" 
                                    value={password} 
                                    onChange={(e) => {setPassword(e.target.value); setError('');}} 
                                    placeholder="••••••••" 
                                    className="w-full px-4 py-3.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none transition-all text-lg placeholder-gray-400 tracking-[0.2em]" 
                                    onKeyDown={e => {if(e.key==='Enter') handleSubmit();}}
                                />
                            </div>
                            <button 
                                onClick={handleSubmit} 
                                className="w-full bg-[#2563eb] hover:bg-blue-700 text-white font-bold py-4 px-4 rounded-xl transition-colors shadow-[0_4px_14px_0_rgba(37,99,235,0.39)] hover:shadow-[0_6px_20px_rgba(37,99,235,0.23)] flex justify-center items-center text-[15px] mt-3">
                                아카이브 접속하기
                            </button>
                        </div>
                        
                        <div className="pb-7 text-center">
                            <p className="text-[11px] text-gray-500">
                                접속 권한이 없으신가요? 선생님께 계정 승인을 요청하세요.
                            </p>
                        </div>
                    </div>
                </div>
            );
        };

        const Sidebar = ({ activeMenu, setActiveMenu, handleLogout, userName, userInfo, isAdmin }) => {
            const menuItems = isAdmin ? [
                { id: 'admin', label: '관리자 대시보드', icon: 'ShieldAlert' }
            ] : [
                { id: 'listening', label: '듣기 훈련', icon: 'Headphones' },
                { id: 'reading', label: '읽기 훈련', icon: 'BookOpen' },
                { id: 'writing', label: 'Layered Writing & Role-play', icon: 'MessageSquare' },
                { id: 'archive', label: '학습 기록 (Archive)', icon: 'Archive' }
            ];

            return (
                <div className="w-[280px] bg-white border-r border-gray-100 flex flex-col hidden md:flex z-10 shadow-[4px_0_24px_rgba(0,0,0,0.02)] relative font-sans">
                    <div className="p-7 pb-4">
                        <h2 className="text-[13px] font-extrabold text-gray-400 tracking-widest mb-6">과제 메뉴</h2>
                        
                        <div className="bg-gray-50 rounded-[1.25rem] p-5 mb-4 border border-gray-100/80 shadow-sm transition-all hover:shadow-md">
                            <div className="flex items-center gap-4">
                                <div className="w-12 h-12 rounded-full bg-indigo-100 text-indigo-600 flex items-center justify-center font-black text-xl shadow-inner">
                                    {userName ? userName.charAt(0) : 'S'}
                                </div>
                                <div className="flex flex-col">
                                    <div className="text-gray-800 font-extrabold text-[15px]">성명: {userName}</div>
                                    {!isAdmin && <div className="text-indigo-600 font-bold text-[11px] mt-1 bg-indigo-100/80 px-2.5 py-1 rounded-md inline-block text-center tracking-wide">레벨: {userInfo?.level || 'HSK 6급'}</div>}
                                    {isAdmin && <div className="text-rose-600 font-bold text-[11px] mt-1 bg-rose-100/80 px-2.5 py-1 rounded-md inline-block text-center tracking-wide">관리자 계정</div>}
                                </div>
                            </div>
                        </div>
                    </div>
                    
                    <div className="flex-1 px-5 space-y-2">
                        {menuItems.map(item => (
                            <button key={item.id} onClick={() => setActiveMenu(item.id)}
                                className={`w-full flex items-center gap-4 px-5 py-4 rounded-[1.25rem] transition-all duration-300 font-bold text-[15px] ${
                                    activeMenu === item.id 
                                    ? 'bg-[#f0f4ff] text-[#4f46e5] shadow-[0_2px_10px_rgba(79,70,229,0.1)]' 
                                    : 'text-gray-500 hover:bg-gray-50 hover:text-gray-800'
                                }`}>
                                <Icon name={item.icon} size={22} className={activeMenu === item.id ? 'text-[#4f46e5]' : 'text-gray-400'} strokeWidth={activeMenu === item.id ? 2.5 : 2} />
                                {item.label}
                            </button>
                        ))}
                    </div>

                    <div className="p-5 border-t border-gray-50 mt-auto">
                        <button onClick={handleLogout} className="w-full flex items-center justify-center gap-2 p-3 text-gray-400 hover:text-rose-500 hover:bg-rose-50 rounded-xl transition-all font-bold text-sm">
                            <Icon name="LogOut" size={18} /> 로그아웃
                        </button>
                    </div>
                </div>
            );
        };

        const ListeningModule = () => {
            const [activeSubTab, setActiveSubTab] = useState('shadowing');

            return (
                <div className="flex flex-col h-full">
                    <div className="flex border-b border-gray-100 bg-gray-50 rounded-t-xl px-4 pt-4 gap-2 overflow-x-auto hide-scrollbar">
                        <button onClick={() => setActiveSubTab('shadowing')} className={`shrink-0 px-6 py-3 font-bold text-sm rounded-t-xl transition-colors ${activeSubTab === 'shadowing' ? 'bg-white border-t border-x border-gray-200 text-indigo-600 shadow-[0_2px_0_0_white]' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-100 border-transparent'}`}>
                            1. 섀도잉
                        </button>
                        <button onClick={() => setActiveSubTab('echoing')} className={`shrink-0 px-6 py-3 font-bold text-sm rounded-t-xl transition-colors ${activeSubTab === 'echoing' ? 'bg-white border-t border-x border-gray-200 text-indigo-600 shadow-[0_2px_0_0_white]' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-100 border-transparent'}`}>
                            2. 에코잉 (듣고 말하기)
                        </button>
                        <button onClick={() => setActiveSubTab('dictation')} className={`shrink-0 px-6 py-3 font-bold text-sm rounded-t-xl transition-colors ${activeSubTab === 'dictation' ? 'bg-white border-t border-x border-gray-200 text-indigo-600 shadow-[0_2px_0_0_white]' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-100 border-transparent'}`}>
                            3. 받아쓰기
                        </button>
                    </div>
                    
                    <div className="p-6 md:p-8 flex-1">
                        {activeSubTab === 'shadowing' && (
                            <div className="space-y-6 animate-in fade-in slide-in-from-bottom-2 duration-300">
                                <div className="flex items-center gap-3 border-b pb-4">
                                    <div className="p-2 bg-indigo-100 text-indigo-600 rounded-lg">
                                        <Icon name="Headphones" size={24} />
                                    </div>
                                    <div>
                                        <h3 className="text-xl font-extrabold text-gray-800">음성 섀도잉 훈련</h3>
                                        <p className="text-sm text-gray-500 font-medium mt-1">원어민의 발음과 똑같은 속도로 따라 말하기</p>
                                    </div>
                                </div>
                                <div className="bg-indigo-50/50 p-6 rounded-2xl flex items-center justify-between border border-indigo-100 shadow-sm">
                                    <div className="flex items-center gap-5">
                                        <button className="w-14 h-14 bg-indigo-600 text-white rounded-full flex items-center justify-center hover:bg-indigo-700 transition-all shadow-md hover:scale-105">
                                            <Icon name="PlayCircle" size={28} />
                                        </button>
                                        <div>
                                            <div className="font-extrabold text-gray-800 text-lg">Track 01. 비즈니스 미팅 상황</div>
                                            <div className="text-sm text-indigo-500 font-semibold mt-0.5">00:00 / 02:30</div>
                                        </div>
                                    </div>
                                    <div className="hidden sm:block w-48 h-2.5 bg-gray-200 rounded-full overflow-hidden shadow-inner">
                                        <div className="w-1/3 h-full bg-indigo-500 rounded-full"></div>
                                    </div>
                                </div>
                                <div className="bg-white border-2 border-dashed border-gray-200 p-8 rounded-2xl text-center space-y-4 hover:border-indigo-300 transition-colors">
                                    <button className="bg-white border border-gray-200 px-8 py-3.5 rounded-xl font-bold text-gray-700 hover:bg-gray-50 shadow-sm flex items-center gap-2 mx-auto transition-all">
                                        <Icon name="Headphones" size={20}/>
                                        내 음성 녹음 시작
                                    </button>
                                </div>
                            </div>
                        )}
                        {activeSubTab === 'echoing' && (
                            <div className="space-y-6 animate-in fade-in slide-in-from-bottom-2 duration-300">
                                <div className="flex items-center gap-3 border-b pb-4">
                                    <div className="p-2 bg-blue-100 text-blue-600 rounded-lg">
                                        <Icon name="MessageSquare" size={24} />
                                    </div>
                                    <div>
                                        <h3 className="text-xl font-extrabold text-gray-800">음성 에코잉 훈련</h3>
                                        <p className="text-sm text-gray-500 font-medium mt-1">문장을 듣고 난 뒤에 기억하여 그대로 말하기</p>
                                    </div>
                                </div>
                                <div className="bg-blue-50/50 p-8 rounded-2xl flex flex-col items-center justify-center border border-blue-100 gap-4 shadow-sm relative overflow-hidden">
                                    <div className="absolute top-0 left-0 w-full h-1 bg-gradient-to-r from-blue-300 to-indigo-500"></div>
                                    <div className="w-16 h-16 bg-white border border-blue-100 text-blue-600 rounded-full flex items-center justify-center hover:bg-blue-600 hover:text-white transition-all shadow-md cursor-pointer hover:scale-105">
                                        <Icon name="PlayCircle" size={32} />
                                    </div>
                                    <div className="text-center">
                                        <div className="font-extrabold text-gray-800 text-lg mb-1">Track 02. 일상 대화문 (문장 단위)</div>
                                        <div className="text-sm text-gray-500 font-medium">원어민의 문장이 끝난 후, 기억하여 정확하게 따라해 보세요.</div>
                                    </div>
                                </div>
                                <div className="bg-white border-2 border-dashed border-gray-200 p-8 rounded-2xl text-center space-y-4 hover:border-rose-300 transition-colors group">
                                    <div className="w-14 h-14 bg-rose-50 text-rose-500 rounded-full flex items-center justify-center mx-auto mb-2 cursor-pointer shadow-sm border border-rose-100 group-hover:bg-rose-500 group-hover:text-white transition-all">
                                        <svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2.5" strokeLinecap="round" strokeLinejoin="round"><path d="M12 2a3 3 0 0 0-3 3v7a3 3 0 0 0 6 0V5a3 3 0 0 0-3-3Z"/><path d="M19 10v2a7 7 0 0 1-14 0v-2"/><line x1="12" x2="12" y1="19" y2="22"/></svg>
                                    </div>
                                    <button className="font-bold text-gray-700 text-lg">
                                        여기를 눌러 에코잉 녹음 시작
                                    </button>
                                    <p className="text-xs text-gray-400 font-medium">녹음을 위해 마이크 권한 허용이 필요합니다.</p>
                                </div>
                            </div>
                        )}
                        {activeSubTab === 'dictation' && (
                            <div className="space-y-6 animate-in fade-in slide-in-from-bottom-2 duration-300">
                                <div className="flex items-center gap-3 border-b pb-4">
                                    <div className="p-2 bg-orange-100 text-orange-600 rounded-lg">
                                        <Icon name="PenTool" size={24} />
                                    </div>
                                    <div>
                                        <h3 className="text-xl font-extrabold text-gray-800">구문 받아쓰기</h3>
                                        <p className="text-sm text-gray-500 font-medium mt-1">들은 문장을 정확한 중국어로 타이핑하세요</p>
                                    </div>
                                </div>
                                <div className="bg-orange-50/50 p-5 rounded-2xl flex items-center gap-4 text-orange-800 border border-orange-100 shadow-sm">
                                    <Icon name="PlayCircle" size={32} className="cursor-pointer hover:text-orange-600 transition-colors drop-shadow-sm"/>
                                    <div>
                                        <span className="font-extrabold text-lg block">문장 1</span>
                                        <span className="text-sm text-orange-600/80 font-medium mt-0.5">3번 반복 재생</span>
                                    </div>
                                </div>
                                <div className="relative">
                                    <textarea className="w-full h-36 p-5 border-2 border-gray-200 rounded-2xl focus:ring-4 focus:ring-indigo-500/20 focus:border-indigo-500 outline-none resize-none font-medium text-lg text-gray-800 transition-all shadow-sm" placeholder="들은 문장을 중국어(병음 또는 한자)로 작성하세요..."></textarea>
                                    <div className="absolute bottom-4 right-4 text-xs font-bold text-gray-300">병음 입력기 지원</div>
                                </div>
                                <div className="flex justify-end">
                                    <button className="bg-indigo-600 text-white px-8 py-3.5 rounded-xl font-bold hover:bg-indigo-700 transition-colors shadow-md hover:shadow-lg hover:-translate-y-0.5">
                                        제출 및 정답 확인
                                    </button>
                                </div>
                            </div>
                        )}
                    </div>
                </div>
            );
        };

        // --- 새로 추가/복구된 ReadingModule 컴포넌트 ---
        const ReadingModule = ({ setActiveMenu, onSubmit, tasks, submissions, userInfo }) => {
            const [activeTask, setActiveTask] = useState(null);

            const readingTasks = tasks.filter(t => t.category === '읽기 훈련' && t.status === '활성');

            const getTaskStatus = (taskId) => {
                const sub = submissions.find(s => s.studentId === userInfo.id && s.title === tasks.find(t => t.id === taskId)?.title);
                if (!sub) return 'available';
                return sub.status;
            };

            if (activeTask) {
                return (
                    <div className="flex flex-col h-full animate-in fade-in duration-300">
                        <div className="mb-4 shrink-0 px-6 pt-6">
                             <button onClick={() => setActiveTask(null)} className="flex items-center gap-2 text-gray-500 hover:text-indigo-600 font-bold transition-colors">
                                 <Icon name="ChevronLeft" size={20}/> 과제 목록으로 돌아가기
                             </button>
                        </div>
                        <div className="flex-1 overflow-hidden px-6 pb-6">
                             {activeTask.type === 'Scaffold Cloze' && (
                                  <ClozeSystem setActiveMenu={setActiveMenu} onSubmit={onSubmit} taskConfig={activeTask} />
                             )}
                             {activeTask.type === '시역 훈련' && (
                                  <SightTranslationSystem setActiveMenu={setActiveMenu} onSubmit={onSubmit} taskConfig={activeTask} />
                             )}
                        </div>
                    </div>
                );
            }

            return (
                <div className="space-y-6 animate-in fade-in duration-300 h-full flex flex-col p-6 md:p-8">
                    <div className="flex items-center gap-3 border-b pb-4 shrink-0">
                        <div className="p-2 bg-indigo-100 text-indigo-600 rounded-lg">
                            <Icon name="BookOpen" size={24} />
                        </div>
                        <div>
                            <h3 className="text-xl font-extrabold text-gray-800">읽기 훈련 (Reading)</h3>
                            <p className="text-sm text-gray-500 font-medium mt-1">AI 튜터와 함께하는 구조화된 읽기 및 번역 훈련</p>
                        </div>
                    </div>

                    <div className="flex-1 overflow-y-auto pb-6">
                        <div className="grid grid-cols-1 xl:grid-cols-2 gap-4">
                            {readingTasks.length > 0 ? readingTasks.map(task => {
                                const status = getTaskStatus(task.id);
                                return (
                                    <div key={task.id} className="bg-white border border-gray-200 rounded-2xl p-5 shadow-sm hover:shadow-md transition-all flex flex-col justify-between group">
                                        <div>
                                            <div className="flex justify-between items-start mb-3">
                                                <span className="bg-indigo-50 text-indigo-600 text-xs font-bold px-2.5 py-1 rounded-md border border-indigo-100">
                                                    {task.type}
                                                </span>
                                                {status === 'approved' && <span className="text-green-500 text-xs font-bold flex items-center gap-1"><Icon name="CheckCircle" size={14}/> 완료됨</span>}
                                                {status === 'pending' && <span className="text-orange-500 text-xs font-bold flex items-center gap-1"><Icon name="Activity" size={14}/> 채점 대기중</span>}
                                            </div>
                                            <h4 className="font-bold text-gray-800 text-lg mb-2 group-hover:text-indigo-600 transition-colors">{task.title}</h4>
                                            <p className="text-sm text-gray-500 line-clamp-2">{task.taskData?.summary || task.summary || 'AI 기반 스마트 읽기 훈련 콘텐츠입니다.'}</p>
                                        </div>
                                        <div className="mt-5 pt-4 border-t border-gray-100">
                                            <button 
                                                onClick={() => setActiveTask(task)} 
                                                disabled={status !== 'available'}
                                                className={`w-full py-2.5 rounded-xl font-bold flex items-center justify-center gap-2 transition-all ${
                                                    status === 'available' 
                                                    ? 'bg-gray-800 text-white hover:bg-gray-900 shadow-sm' 
                                                    : 'bg-gray-100 text-gray-400 cursor-not-allowed'
                                                }`}
                                            >
                                                {status === 'available' ? '학습 시작하기' : status === 'pending' ? '제출 완료' : '학습 완료'}
                                            </button>
                                        </div>
                                    </div>
                                );
                            }) : (
                                <div className="col-span-full py-16 text-center text-gray-400 bg-gray-50 rounded-2xl border-2 border-dashed border-gray-200">
                                    <Icon name="BookOpen" size={48} className="mx-auto mb-4 opacity-20"/>
                                    <p className="font-medium text-gray-500">현재 배정된 읽기 훈련 과제가 없습니다.</p>
                                </div>
                            )}
                        </div>
                    </div>
                </div>
            );
        };

        const AnalogMark = ({ isCorrect, isPartial }) => {
            const circlePath = "M50,15 c-25,0 -40,15 -40,35 c0,22 18,38 40,38 c25,0 40,-15 40,-38 c0,-20 -15,-32 -35,-35";
            const trianglePath = "M50,15 l35,65 l-70,0 z";
            const slashPath = "M20,80 l60,-60";

            if (isPartial) {
                return (
                    <svg className="absolute -inset-2 w-[calc(100%+1rem)] h-[calc(100%+1rem)] pointer-events-none z-20" viewBox="0 0 100 100">
                        <path d={trianglePath} fill="none" stroke="#f59e0b" strokeWidth="5" strokeLinecap="round" strokeLinejoin="round" className="mark-animate" style={{ strokeDasharray: 250, strokeDashoffset: 250 }} />
                    </svg>
                );
            } else if (isCorrect) {
                return (
                    <svg className="absolute -inset-2 w-[calc(100%+1rem)] h-[calc(100%+1rem)] pointer-events-none z-20" viewBox="0 0 100 100">
                        <path d={circlePath} fill="none" stroke="#ef4444" strokeWidth="5" strokeLinecap="round" className="mark-animate" style={{ strokeDasharray: 300, strokeDashoffset: 300 }} />
                    </svg>
                );
            } else {
                return (
                     <svg className="absolute -inset-2 w-[calc(100%+1rem)] h-[calc(100%+1rem)] pointer-events-none z-20" viewBox="0 0 100 100">
                        <path d={slashPath} fill="none" stroke="#ef4444" strokeWidth="5" strokeLinecap="round" className="mark-animate-fast" style={{ strokeDasharray: 150, strokeDashoffset: 150 }} />
                    </svg>
                );
            }
        };

        const SightTranslationSystem = ({ setActiveMenu, onSubmit, taskConfig }) => {
            const [phase, setPhase] = useState('review');
            
            const isDynamic = taskConfig ? !taskConfig.isLegacy : false;
            const currentTitle = taskConfig ? taskConfig.title : '시역 훈련 (Sight Translation)';
            
            const chunks = isDynamic && taskConfig && taskConfig.taskData && taskConfig.taskData.chunks ? taskConfig.taskData.chunks : [
                { ko: "연구에 따르면,", cn: "有研究表明：", pinyin: "yǒu yánjiū biǎomíng:" },
                { ko: "지표면 온도의 변화 문제와 비교할 때,", cn: "与地表温度的变化问题相比，", pinyin: "yǔ dìbiǎo wēndù de biànhuà wèntí xiāng bǐ," },
                { ko: "근해 구역에 사는 사람들은 해수면 상승 문제에 더 관심을 갖습니다.", cn: "生活在近海区域的人们更关心海平面上升的问题；", pinyin: "shēnghuó zài jìnhǎi qūyù de rénmen gèng guānxīn hǎipíngmiàn shàngshēng de wèntí;" }
            ];

            const handleStartRecord = () => {
                setPhase('record');
            };

            const handleStopRecord = () => {
                onSubmit({
                    title: currentTitle,
                    type: 'reading',
                    score: 95
                });
                setActiveMenu('archive');
            };

            return (
                <div className="flex flex-col h-full animate-in fade-in duration-300">
                    <div className="flex items-center justify-between shrink-0 mb-4">
                        <h3 className="text-xl font-bold">{currentTitle}</h3>
                        <span className="bg-indigo-100 text-indigo-700 font-bold px-3 py-1 rounded-full text-sm shadow-sm">
                            {phase === 'review' ? 'Step 1: 청크 사전 학습' : 'Step 2: 시역 통역 및 녹음'}
                        </span>
                    </div>

                    <div className="bg-white border border-gray-200 rounded-xl overflow-hidden shadow-sm flex flex-col flex-1 h-[75vh] min-h-[600px] relative">
                        <div className="p-6 md:p-8 flex-1 overflow-y-auto relative z-10 bg-slate-50">
                            {phase === 'review' && (
                                <div className="space-y-4 max-w-4xl mx-auto">
                                    <div className="bg-indigo-50 border border-indigo-100 p-5 rounded-xl mb-6 text-sm text-indigo-800 font-medium leading-relaxed shadow-sm">
                                        <Icon name="Volume2" size={20} className="inline mr-2 mb-1"/>
                                        아래 한국어-중국어 의미 단위(청크)를 매칭하여 미리 학습하세요. 
                                        <br/>학습이 끝나면 하단의 <strong>[녹음 시작]</strong> 버튼을 눌러주세요. 다음 화면에서는 중국어가 가려지고 한국어만 보이며, 즉시 중국어로 통역하여 말하는 훈련을 진행합니다.
                                    </div>
                                    <div className="space-y-3">
                                        {chunks.map((c, i) => (
                                            <div key={i} className="flex flex-col md:flex-row bg-white border border-gray-200 rounded-xl shadow-sm overflow-hidden hover:border-indigo-300 transition-colors">
                                                <div className="w-full md:w-1/2 p-5 bg-gray-50/50 border-b md:border-b-0 md:border-r border-gray-100 flex items-center">
                                                    <span className="font-bold text-gray-700 text-[15px] leading-relaxed">{c.ko}</span>
                                                </div>
                                                <div className="w-full md:w-1/2 p-5 flex flex-col justify-center bg-white">
                                                    <span className="font-serif text-[17px] font-bold text-gray-900 mb-1">{c.cn}</span>
                                                    <span className="text-indigo-500 font-medium text-[13px] tracking-wide">{c.pinyin}</span>
                                                </div>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            )}
                            {phase === 'record' && (
                                <div className="space-y-4 max-w-4xl mx-auto pb-24">
                                    <div className="bg-rose-50 border border-rose-100 p-5 rounded-xl mb-6 text-sm text-rose-800 font-bold flex items-center justify-between shadow-sm">
                                        <div className="flex items-center">
                                            <Icon name="Mic" size={20} className="inline mr-2 text-rose-600 animate-pulse"/>
                                            녹음이 진행 중입니다. 한국어 의미 단위를 보고 즉시 중국어로 소리내어 번역하세요.
                                        </div>
                                        <div className="flex gap-1">
                                            <div className="w-2.5 h-2.5 bg-rose-500 rounded-full animate-bounce"></div>
                                            <div className="w-2.5 h-2.5 bg-rose-500 rounded-full animate-bounce" style={{animationDelay: '0.2s'}}></div>
                                            <div className="w-2.5 h-2.5 bg-rose-500 rounded-full animate-bounce" style={{animationDelay: '0.4s'}}></div>
                                        </div>
                                    </div>
                                    <div className="space-y-3">
                                        {chunks.map((c, i) => (
                                            <div key={i} className="bg-white border border-gray-200 rounded-xl shadow-sm p-6 flex items-center min-h-[90px] border-l-4 border-l-indigo-500 hover:bg-indigo-50/30 transition-colors">
                                                <span className="font-extrabold text-gray-800 text-lg leading-relaxed">{c.ko}</span>
                                            </div>
                                        ))}
                                    </div>
                                </div>
                            )}
                        </div>

                        <div className="p-5 bg-white border-t border-gray-200 shrink-0 flex justify-center shadow-[0_-4px_10px_rgba(0,0,0,0.03)] z-30">
                            {phase === 'review' ? (
                                <button onClick={handleStartRecord} className="w-full max-w-xl bg-rose-600 text-white py-4 rounded-xl font-bold hover:bg-rose-700 transition-colors shadow-md flex items-center justify-center gap-2 text-lg">
                                    <Icon name="Mic" size={22}/> 한국어만 보고 중국어 통역 녹음 시작
                                </button>
                            ) : (
                                <button onClick={handleStopRecord} className="w-full max-w-xl bg-gray-800 text-white py-4 rounded-xl font-bold hover:bg-gray-900 transition-colors shadow-md flex items-center justify-center gap-2 text-lg animate-pulse">
                                    <Icon name="Square" size={22}/> 녹음 완료 및 과제 제출
                                </button>
                            )}
                        </div>
                    </div>
                </div>
            );
        };

        const ClozeSystem = ({ setActiveMenu, onSubmit, taskConfig }) => {
            const [step, setStep] = useState(1);
            const [inputs, setInputs] = useState({ 1: {}, 2: {}, 3: {}, 4: {} });
            const [isEvaluating, setIsEvaluating] = useState(false);
            const [evaluationResults, setEvaluationResults] = useState({ 1: null, 2: null, 3: null, 4: null });
            const [scores, setScores] = useState({ 1: 0, 2: 0, 3: 0, 4: 0 });
            const [evaluatedSteps, setEvaluatedSteps] = useState({ 1: false, 2: false, 3: false, 4: false });

            const isDynamic = taskConfig ? !taskConfig.isLegacy : false;
            const dynamicData = isDynamic && taskConfig ? taskConfig.taskData : null;
            const currentTitle = taskConfig ? taskConfig.title : '';

            const correctAnswers = isDynamic && dynamicData ? dynamicData.correctAnswers : {
                1: {
                    1: "博客", 2: "陈旧", 3: "当今社会知识更新的速度加快", 4: "倍增", 5: "知识爆炸",
                    6: "应付一辈子", 7: "已完全不可能了", 8: "终身教育", 9: "单一", 10: "激烈",
                    11: "司空见惯", 12: "害怕", 13: "日子会不好过", 14: "积压物资", 15: "唯一的办法就是多学几手",
                    16: "一专多能", 17: "一棵树上吊死", 18: "脆弱", 19: "打退堂鼓", 20: "稍有不顺利情绪就降为零",
                    21: "这样的人在今后的竞争中必然不易成功", 22: "节奏", 23: "竞争压力加大", 24: "障碍", 25: "第四种是目光短浅的人",
                    26: "鼠目寸光", 27: "你能看多远，你便能走多远", 28: "规划", 29: "不吃香", 30: "以后更不受欢迎"
                },
                2: { 1: "博客", 2: "陈旧", 3: "倍增", 4: "单一", 5: "激烈", 6: "害怕", 7: "脆弱", 8: "节奏", 9: "障碍", 10: "规划" },
                3: { 1: "知识爆炸", 2: "应付一辈子", 3: "终身教育", 4: "司空见惯", 5: "积压物资", 6: "一专多能", 7: "一棵树上吊死", 8: "打退堂鼓", 9: "鼠目寸光", 10: "不吃香" },
                4: { 1: "当今社会知识更新的速度加快", 2: "已完全不可能了", 3: "日子会不好过", 4: "唯一的办法就是多学几手", 5: "稍有不顺利情绪就降为零", 6: "这样的人在今后的竞争中必然不易成功", 7: "竞争压力加大", 8: "第四种是目光短浅的人", 9: "你能看多远，你便能走多远", 10: "以后更不受欢迎" }
            };

            const dictionaryList = isDynamic && dynamicData ? dynamicData.dictionaryList : [
                { word: "博客", pinyin: "bókè", mean: "블로그" },
                { word: "陈旧", pinyin: "chénjiù", mean: "낡다, 진부하다" },
                { word: "当今社会知识更新的速度加快", pinyin: "dāngjīn shèhuì zhīshì gēngxīn de sùdù jiākuài", mean: "오늘날 사회는 지식 갱신 속도가 빨라진다" },
                { word: "倍增", pinyin: "bèizēng", mean: "배가되다, 급증하다" },
                { word: "知识爆炸", pinyin: "zhīshì bàozhà", mean: "지식 폭발" },
                { word: "应付一辈子", pinyin: "yìngfù yíbèizi", mean: "평생을 대처하다" },
                { word: "已完全不可能了", pinyin: "yǐ wánquán bù kěnéng le", mean: "이미 완전히 불가능해졌다" },
                { word: "终身教育", pinyin: "zhōngshēn jiàoyù", mean: "평생 교육" },
                { word: "单一", pinyin: "dānyī", mean: "단일하다, 단순하다" },
                { word: "激烈", pinyin: "jīliè", mean: "치열하다, 격렬하다" },
                { word: "司空见惯", pinyin: "sīkōng jiànguàn", mean: "자주 보아 예사롭다" },
                { word: "害怕", pinyin: "hàipà", mean: "두려워하다" },
                { word: "日子会不好过", pinyin: "rìzi huì bù hǎoguò", mean: "생활이 힘들어질 것이다" },
                { word: "积压物资", pinyin: "jīyā wùzī", mean: "체화 물자, 쌓인 재고" },
                { word: "唯一的办法就是多学几手", pinyin: "wéiyī de bànfǎ jiùshì duō xué jǐ shǒu", mean: "유일한 방법은 몇 가지 기술을 더 배우는 것이다" },
                { word: "一专多能", pinyin: "yìzhuānduōnéng", mean: "한 가지 전문 지식과 여러 능력을 갖추다" },
                { word: "一棵树上吊死", pinyin: "yì kē shù shàng diàosǐ", mean: "한 나무에 목매어 죽다, 융통성이 없다" },
                { word: "脆弱", pinyin: "cuìruò", mean: "연약하다, 취약하다" },
                { word: "打退堂鼓", pinyin: "dǎ tuìtánggǔ", mean: "꽁무니를 빼다, 중도에 포기하다" },
                { word: "稍有不顺利情绪就降为零", pinyin: "shāo yǒu bù shùnlì qíngxù jiù jiàng wéi líng", mean: "조금만 순조롭지 않아도 의욕이 바닥으로 떨어진다" },
                { word: "这样的人在今后的竞争中必然不易成功", pinyin: "zhèyàng de rén zài jīn hòu de jìngzhēng zhōng bìrán bù yì chénggōng", mean: "이런 사람은 향후 경쟁에서 결코 성공하기 쉽지 않다" },
                { word: "节奏", pinyin: "jiézòu", mean: "리듬, 템포" },
                { word: "竞争压力加大", pinyin: "jìngzhēng yālì jiādà", mean: "경쟁 압력이 커지다" },
                { word: "障碍", pinyin: "zhàng'ài", mean: "장애" },
                { word: "第四种是目光短浅的人", pinyin: "dì sì zhǒng shì mùguāng duǎnqiǎn de rén", mean: "네 번째는 근시안적인 사람이다" },
                { word: "鼠目寸光", pinyin: "shǔmùcùnguāng", mean: "쥐의 눈처럼 근시안적이다" },
                { word: "你能看多远，你便能走多远", pinyin: "nǐ néng kàn duō yuǎn, nǐ biàn néng zǒu duō yuǎn", mean: "당신이 멀리 볼 수 있는 만큼 멀리 갈 수 있다" },
                { word: "规划", pinyin: "guīhuà", mean: "계획, 기획" },
                { word: "不吃香", pinyin: "bù chīxiāng", mean: "인기가 없다, 환영받지 못하다" },
                { word: "以后更不受欢迎", pinyin: "yǐhòu gèng bù shòu huānyíng", mean: "나중에는 더욱 환영받지 못할 것이다" }
            ];

            const handleSubmit = () => {
                let earned = 0; let totalP = 0;
                [1,2,3,4].forEach(s => {
                    const stepTotal = correctAnswers && correctAnswers[s] ? Object.keys(correctAnswers[s]).length : 0;
                    if(stepTotal > 0) {
                        earned += (scores[s] / stepTotal);
                        totalP += 1;
                    }
                });
                const finalScore = totalP > 0 ? Math.round((earned / totalP) * 100) : 0;
                
                onSubmit({
                    title: currentTitle,
                    type: 'reading',
                    score: finalScore
                });
                setActiveMenu('archive');
            };

            const handleInputChange = (targetStep, index, value) => {
                setInputs(prev => ({
                    ...prev,
                    [targetStep]: {
                        ...prev[targetStep],
                        [index]: value
                    }
                }));
            };

            const formatFeedback = (text) => {
                if (!text) return '';
                let formatted = text.replace(/\*\*(.*?)\*\*/g, '<strong class="text-indigo-700 bg-indigo-50 px-1 rounded">$1</strong>');
                formatted = formatted.replace(/\*(.*?)\*/g, '<em class="text-indigo-600 font-bold">$1</em>');
                formatted = formatted.replace(/\n/g, '<br/>');
                return formatted;
            };

            const handleEvaluate = async () => {
                setIsEvaluating(true);
                const totalItems = correctAnswers && correctAnswers[step] ? Object.keys(correctAnswers[step]).length : 0;
                
                if(totalItems === 0) {
                    setEvaluatedSteps(prev => ({...prev, [step]: true}));
                    setIsEvaluating(false);
                    return;
                }

                const apiKey = ""; 
                const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
                const promptText = `당신은 학생들을 진심으로 응원하고 칭찬하는 다정한 HSK 중국어 선생님입니다.
학생이 중국어 빈칸 채우기 과제인 Scaffold Cloze System 의 Step ${step} 문제를 방금 풀었습니다.

[정답 목록 (문제 번호: 정답)]
${JSON.stringify(correctAnswers[step], null, 2)}

[학생이 제출한 답안 (문제 번호: 학생 답안)]
${JSON.stringify(inputs[step] || {}, null, 2)}

아래 지침에 따라 JSON 형식으로 평가 결과를 반환해주세요:
1. 총점(totalScore)은 100점 만점 기준으로 환산하여 계산하세요. (문맥과 의미가 통하면 부분 점수도 인정해 주세요.)
2. 각 문제별(evaluations)로 정답 여부(isCorrect: 의미상 맞으면 true), 부분 점수 여부(isPartial: 의미는 통하지만 어법/철자가 약간 틀린 경우 true), 피드백(feedback)을 포함하세요. (비워둔 칸은 무조건 오답 처리)
3. 선생님의 메시지(teacherMessage)에는 점수를 밝게 알려주고(예: "와! 100점 만점에 85점이에요! 🎉"), 학생의 노력에 대한 듬뿍 담긴 칭찬과 따뜻한 격려를 작성해주세요. (마크다운 사용 가능)`;

                const payload = {
                    contents: [{ parts: [{ text: promptText }] }],
                    systemInstruction: { parts: [{ text: "당신은 학생의 답안을 아날로그 감성으로 100점 만점 기준으로 채점하며, 의미가 맞다면 부분 점수를 인정해주는 친절한 HSK 강사입니다." }] },
                    generationConfig: {
                        responseMimeType: "application/json",
                        responseSchema: {
                            type: "OBJECT",
                            properties: {
                                totalScore: { type: "INTEGER", description: "100점 만점 기준의 총점" },
                                evaluations: {
                                    type: "ARRAY",
                                    items: {
                                        type: "OBJECT",
                                        properties: {
                                            id: { type: "INTEGER", description: "문제 번호" },
                                            isCorrect: { type: "BOOLEAN", description: "의미상 맞으면 true" },
                                            isPartial: { type: "BOOLEAN", description: "부분 점수인 경우 true" },
                                            feedback: { type: "STRING", description: "오답/부분점수일 경우 정답과 이유 설명" }
                                        }
                                    }
                                },
                                teacherMessage: { type: "STRING", description: "칭찬과 격려 멘트" }
                            },
                            required: ["totalScore", "evaluations", "teacherMessage"]
                        }
                    }
                };

                let resultObj = null;
                const maxRetries = 5;
                const delays = [1000, 2000, 4000, 8000, 16000];

                for (let i = 0; i < maxRetries; i++) {
                    try {
                        const response = await fetch(url, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (!response.ok) throw new Error('API Request Failed');
                        const data = await response.json();
                        const textData = data.candidates?.[0]?.content?.parts?.[0]?.text;
                        if(textData) {
                            resultObj = JSON.parse(textData);
                            break;
                        }
                    } catch (err) {
                        if (i === maxRetries - 1) {
                            console.error("Evaluation Error:", err);
                        } else {
                            await new Promise(r => setTimeout(r, delays[i]));
                        }
                    }
                }

                if(!resultObj) {
                    let mockScore = 0;
                    for(let i=1; i<=totalItems; i++) {
                        if((inputs[step] && inputs[step][i] ? inputs[step][i] : '').trim() === correctAnswers[step][i]) mockScore++;
                    }
                    resultObj = {
                        totalScore: Math.round((mockScore/totalItems)*100) || 0,
                        evaluations: Object.keys(correctAnswers[step]).map(id => ({ id: parseInt(id), isCorrect: (inputs[step] && inputs[step][id] ? inputs[step][id] : '').trim() === correctAnswers[step][id], isPartial: false, feedback: `정답은 '${correctAnswers[step][id]}' 입니다.` })),
                        teacherMessage: `채점 시스템 응답 지연으로 자동 채점되었습니다. (100점 만점 중 ${Math.round((mockScore/totalItems)*100)}점)`
                    };
                }

                setEvaluationResults(prev => ({ ...prev, [step]: resultObj }));
                setScores(prev => ({...prev, [step]: resultObj.totalScore}));
                setEvaluatedSteps(prev => ({...prev, [step]: true}));
                setIsEvaluating(false);
            };

            const renderBlank = (stepMap, currentStep, answer, pinyin) => {
                const index = stepMap[currentStep];
                const isTarget = index !== undefined;

                if (isTarget) {
                    const isClause = answer.length > 6;
                    const widthStyle = { width: isClause ? `min(100%, max(10rem, ${answer.length * 1.1}em))` : `max(3.5rem, ${answer.length * 1.3}em)` };
                    const userText = (inputs[currentStep] && inputs[currentStep][index] ? inputs[currentStep][index] : '').trim();

                    if (evaluatedSteps[currentStep]) {
                        const exactMatch = userText === correctAnswers[currentStep][index];
                        const evalData = evaluationResults[currentStep] && evaluationResults[currentStep].evaluations ? evaluationResults[currentStep].evaluations.find(e => e.id === index) : null;
                        
                        const isCorrect = exactMatch || (evalData ? evalData.isCorrect : false);
                        const isPartial = !exactMatch && (evalData ? evalData.isPartial : false);

                        return (
                            <span className="inline-flex flex-col items-center mx-1 align-bottom relative group" title={currentStep === 1 ? pinyin : undefined}>
                                <span className="text-[12px] text-gray-400 font-bold mb-[-4px]">{index}</span>
                                <div className="relative inline-block text-center px-1 pb-1 font-sans text-base text-gray-700 bg-gray-50 border-b-2 border-gray-300 min-w-[3.5rem]" style={widthStyle}>
                                    <span className="relative z-10 break-all">{userText || '\u00A0\u00A0\u00A0'}</span>
                                    <AnalogMark isCorrect={isCorrect} isPartial={isPartial} />
                                </div>
                            </span>
                        );
                    }

                    return (
                        <span 
                            className="inline-flex flex-col items-center mx-1 align-bottom relative group" 
                            title={currentStep === 1 ? pinyin : undefined}
                        >
                            <span className="text-[12px] text-gray-400 font-bold mb-[-4px]">{index}</span>
                            <input
                                type="text"
                                value={inputs[currentStep] && inputs[currentStep][index] ? inputs[currentStep][index] : ''}
                                onChange={(e) => handleInputChange(currentStep, index, e.target.value)}
                                placeholder={currentStep === 1 ? answer : ""}
                                className={`border-b-2 px-1 pb-1 text-center outline-none transition-all font-sans text-base max-w-full 
                                ${currentStep === 1 
                                    ? "border-dashed border-gray-400 text-blue-700 bg-gray-50 focus:bg-white placeholder-gray-400/50 focus:placeholder-transparent" 
                                    : "border-blue-400 text-blue-700 bg-blue-50/30 focus:border-blue-600 focus:bg-transparent"}`}
                                style={widthStyle}
                            />
                        </span>
                    );
                }
                
                const mainStep = Object.keys(stepMap).find(k => Number(k) > 1);
                const passed = mainStep && currentStep > Number(mainStep);
                if (passed) {
                    return <span className="mx-1 text-indigo-600 font-bold bg-indigo-50 px-1 rounded" title={currentStep === 1 ? pinyin : undefined}>{answer}</span>;
                }
                return <span className="mx-1" title={currentStep === 1 ? pinyin : undefined}>{answer}</span>;
            };

            const T = ({ children, pinyin, currentStep }) => {
                if (currentStep === 1) {
                    return (
                        <span title={pinyin} className="cursor-help border-b border-dotted border-gray-300 mx-0.5 hover:bg-gray-100 transition-colors">
                            {children}
                        </span>
                    );
                }
                return <span>{children}</span>;
            };

            if (step === 5) {
                return (
                    <div className="space-y-6 animate-in fade-in slide-in-from-bottom-2 duration-300 flex flex-col items-center justify-center py-10">
                        <div className="w-16 h-16 bg-green-100 text-green-600 rounded-full flex items-center justify-center mb-4">
                            <Icon name="CheckCircle" size={32} />
                        </div>
                        <h3 className="text-2xl font-bold text-gray-800">학습 완료!</h3>
                        <p className="text-gray-500 mb-6">모든 단계의 빈칸 학습과 채점을 완료했습니다.</p>
                        
                        <div className="bg-gray-50 rounded-xl p-6 w-full max-w-md border border-gray-200">
                            <h4 className="font-bold text-gray-700 mb-4">단계별 결과 요약 (100점 만점)</h4>
                            <div className="space-y-3">
                                {[1,2,3,4].map(s => {
                                    if(correctAnswers && correctAnswers[s] && Object.keys(correctAnswers[s]).length > 0) {
                                        return (
                                            <div key={s} className="flex justify-between items-center bg-white p-3 rounded-lg border shadow-sm">
                                                <span className="text-sm font-medium text-gray-600">Step {s}</span>
                                                <span className="text-indigo-600 font-bold">{scores[s]} 점</span>
                                            </div>
                                        );
                                    }
                                    return null;
                                })}
                            </div>
                        </div>

                        <div className="flex gap-3 mt-4">
                            <button onClick={() => {
                                setStep(1); setInputs({1:{},2:{},3:{},4:{}}); setEvaluatedSteps({1:false,2:false,3:false,4:false}); setScores({1:0,2:0,3:0,4:0}); setEvaluationResults({1:null,2:null,3:null,4:null});
                            }} className="px-6 py-2.5 rounded-xl border border-gray-300 text-gray-700 font-medium hover:bg-gray-50 transition-colors">
                                처음부터 다시하기
                            </button>
                            <button onClick={handleSubmit} className="px-6 py-2.5 rounded-xl bg-indigo-600 text-white font-bold hover:bg-indigo-700 transition-colors shadow-md">
                                과제 최종 제출하기
                            </button>
                        </div>
                    </div>
                );
            }

            return (
                <div className="space-y-4 animate-in fade-in slide-in-from-bottom-2 duration-300 h-full flex flex-col">
                    <div className="flex items-center justify-between shrink-0">
                        <h3 className="text-xl font-bold">{currentTitle}</h3>
                        <div className="flex gap-2">
                            {[1, 2, 3, 4].map(s => (
                                <div key={s} className={`w-8 h-8 rounded-full flex items-center justify-center font-bold text-sm ${step === s ? 'bg-indigo-600 text-white shadow-md scale-110 transition-transform' : step > s ? 'bg-green-500 text-white' : 'bg-gray-200 text-gray-500'}`}>
                                    {step > s ? <Icon name="CheckCircle" size={16}/> : s}
                                </div>
                            ))}
                        </div>
                    </div>

                    <div className="bg-white border border-gray-200 rounded-xl overflow-hidden shadow-sm flex flex-col flex-1 h-[80vh] min-h-[700px]">
                        
                        <div className="p-6 md:p-8 flex-1 overflow-y-auto relative z-10 bg-white">
                            <h5 className="font-bold text-center mb-6 text-xl text-gray-800">{currentTitle}</h5>
                            
                            <div className="text-base md:text-lg leading-loose text-gray-800 space-y-4 font-serif relative z-10">
                                {isDynamic && dynamicData && dynamicData.tokens ? (
                                    <p>
                                        {dynamicData.tokens.map((tok, i) => {
                                            if (tok.type === 'text') {
                                                return <span key={i} dangerouslySetInnerHTML={{ __html: tok.value.replace(/\n/g, '<br/>') }} />;
                                            } else {
                                                return <React.Fragment key={i}>{renderBlank(tok.stepMap, step, tok.value, tok.pinyin)}</React.Fragment>;
                                            }
                                        })}
                                    </p>
                                ) : (
                                    <>
                                        <p>
                                            <T pinyin="Zài yí gè péngyǒu de " currentStep={step}>在一个朋友的</T>
                                            {renderBlank({1: 1, 2: 1}, step, '博客', 'bókè')}
                                            <T pinyin=" zhōng yǒu zhèyàng yī piān wénzhāng, tā bǎ shèhuì shàng bù chénggōng de rén fēnchéng yǐxià sì zhǒng lèixíng:" currentStep={step}>中有这样一篇文章，她把社会上不成功的人分成以下四种类型：</T>
                                        </p>
                                        <p>
                                            <T pinyin="Dì yī zhǒng shì zhīshì " currentStep={step}>第一种是知识</T>
                                            {renderBlank({1: 2, 2: 2}, step, '陈旧', 'chénjiù')}
                                            <T pinyin=" de rén。" currentStep={step}>的人。</T>
                                            {renderBlank({1: 3, 4: 1}, step, '当今社会知识更新的速度加快', 'dāngjīn shèhuì zhīshì gēngxīn de sùdù jiākuài')}
                                            <T pinyin="，zhīshì " currentStep={step}>，知识</T>
                                            {renderBlank({1: 4, 2: 3}, step, '倍增', 'bèizēng')}
                                            <T pinyin=" de zhōuqī yuè lái yuè duǎn，rénlèi zhēnzhèng jìnrùle " currentStep={step}>的周期越来越短，人类真正进入了</T>
                                            {renderBlank({1: 5, 3: 1}, step, '知识爆炸', 'zhīshì bàozhà')}
                                            <T pinyin=" de shídài。Shēnghuó zài zhège shídài，rènhé rén dōu bìxū búduàn xuéxí，gēngxīn zhīshì，xiǎng kào xuéxiào xuédào de zhīshì " currentStep={step}>的时代。生活在这个时代，任何人都必须不断学习，更新知识，想靠学校学到的知识</T>
                                            {renderBlank({1: 6, 3: 2}, step, '应付一辈子', 'yìngfù yíbèizi')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 7, 4: 2}, step, '已完全不可能了', 'yǐ wánquán bù kěnéng le')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 8, 3: 3}, step, '终身教育', 'zhōngshēn jiàoyù')}
                                            <T pinyin=" yīng guànchuān rén de yìshēng。" currentStep={step}>应贯穿人的一生。</T>
                                        </p>
                                        <p>
                                            <T pinyin="Dì èr zhǒng shì jìnéng " currentStep={step}>第二种是技能</T>
                                            {renderBlank({1: 9, 2: 4}, step, '单一', 'dānyī')}
                                            <T pinyin=" de rén。Shèhuì jìngzhēng yuè lái yuè " currentStep={step}>的人。社会竞争越来越</T>
                                            {renderBlank({1: 10, 2: 5}, step, '激烈', 'jīliè')}
                                            <T pinyin="，pínfán de jiùyè hé xiàgǎng jiāng chéngwéi " currentStep={step}>，频繁的就业和下岗将成为</T>
                                            {renderBlank({1: 11, 3: 4}, step, '司空见惯', 'sīkōng jiànguàn')}
                                            <T pinyin=" de shì。Zhǐ huì zuò yī zhǒng gōngzuò，huàn yí gè gǎngwèi jiù " currentStep={step}>的事。只会做一种工作，换一个岗位就</T>
                                            {renderBlank({1: 12, 2: 6}, step, '害怕', 'hàipà')}
                                            <T pinyin=" de rén，" currentStep={step}>的人，</T>
                                            {renderBlank({1: 13, 4: 3}, step, '日子会不好过', 'rìzi huì bù hǎoguò')}
                                            <T pinyin="。Yào xiǎng bìmiǎn zài zhíchǎng zhōng chéngwéi “" currentStep={step}>。要想避免在职场中成为“</T>
                                            {renderBlank({1: 14, 3: 5}, step, '积压物资', 'jīyā wùzī')}
                                            <T pinyin="”，" currentStep={step}>”，</T>
                                            {renderBlank({1: 15, 4: 4}, step, '唯一的办法就是多学几手', 'wéiyī de bànfǎ jiùshì duō xué jǐ shǒu')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 16, 3: 6}, step, '一专多能', 'yìzhuānduōnéng')}
                                            <T pinyin="，zhǐyǒu zhèyàng，cái bù zhìyú “" currentStep={step}>，只有这样，才不至于“</T>
                                            {renderBlank({1: 17, 3: 7}, step, '一棵树上吊死', 'yì kē shù shàng diàosǐ')}
                                            <T pinyin="”。" currentStep={step}>”。</T>
                                        </p>
                                        <p>
                                            <T pinyin="Dì sān zhǒng shì xīnlǐ " currentStep={step}>第三种是心理</T>
                                            {renderBlank({1: 18, 2: 7}, step, '脆弱', 'cuìruò')}
                                            <T pinyin=" de rén。Yù dào yīdiǎnr kùnnán jiù " currentStep={step}>的人。遇到一点儿困难就</T>
                                            {renderBlank({1: 19, 3: 8}, step, '打退堂鼓', 'dǎ tuìtánggǔ')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 20, 4: 5}, step, '稍有不顺利情绪就降为零', 'shāo yǒu bù shùnlì qíngxù jiù jiàng wéi líng')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 21, 4: 6}, step, '这样的人在今后的竞争中必然不易成功', 'zhèyàng de rén zài jīn hòu de jìngzhēng zhōng bìrán bù yì chénggōng')}
                                            <T pinyin="。Yóuyú shēnghuó " currentStep={step}>。由于生活</T>
                                            {renderBlank({1: 22, 2: 8}, step, '节奏', 'jiézòu')}
                                            <T pinyin=" jiākuài，" currentStep={step}>加快，</T>
                                            {renderBlank({1: 23, 4: 7}, step, '竞争压力加大', 'jìngzhēng yālì jiādà')}
                                            <T pinyin="，yǒu xīnlǐ " currentStep={step}>，有心理</T>
                                            {renderBlank({1: 24, 2: 9}, step, '障碍', 'zhàng\'ài')}
                                            <T pinyin=" huò xīnlǐ jíbìng de rén zhújiàn zēngduō，shénjīng jǐnzhāng、xīnlǐ cuìruò chéngle dūshì xiàndàibìng。" currentStep={step}>或心理疾病的人逐渐增多，神经紧张、心理脆弱成了都市现代病。</T>
                                        </p>
                                        <p>
                                            {renderBlank({1: 25, 4: 8}, step, '第四种是目光短浅的人', 'dì sì zhǒng shì mùguāng duǎnqiǎn de rén')}
                                            <T pinyin="。" currentStep={step}>。</T>
                                            {renderBlank({1: 26, 3: 9}, step, '鼠目寸光', 'shǔmùcùnguāng')}
                                            <T pinyin=" nán chéng dàshì，mùguāng yuǎndà cái yì chénggōng。Yǒu jù huà shuō de hǎo：“" currentStep={step}>难成大事，目光远大才易成功。有句话说得好：“</T>
                                            {renderBlank({1: 27, 4: 9}, step, '你能看多远，你便能走多远', 'nǐ néng kàn duō yuǎn, nǐ biàn néng zǒu duō yuǎn')}
                                            <T pinyin="。” Yí gè zǔzhī de chéngzhǎng xūyào " currentStep={step}>。”一个组织的成长需要</T>
                                            {renderBlank({1: 28, 2: 10}, step, '规划', 'guīhuà')}
                                            <T pinyin="，yí gè rén de chéngzhǎng xūyào shèjì，zhǐ kànjiàn bíjiān xiàbiān yī xiǎo kuài dìfāng de rén，xiànzài " currentStep={step}>，一个人的成长需要设计，只看见鼻尖下边一小块地方的人，现在</T>
                                            {renderBlank({1: 29, 3: 10}, step, '不吃香', 'bù chīxiāng')}
                                            <T pinyin="，" currentStep={step}>，</T>
                                            {renderBlank({1: 30, 4: 10}, step, '以后更不受欢迎', 'yǐhòu gèng bù shòu huānyíng')}
                                            <T pinyin="。" currentStep={step}>。</T>
                                        </p>
                                    </>
                                )}
                            </div>
                        </div>

                        <div className="w-full h-[50%] md:h-[45%] bg-[#f8fafc] flex flex-col shrink-0 border-t-2 border-gray-200 relative z-20 shadow-[0_-10px_20px_rgba(0,0,0,0.02)]">
                            <div className="p-6 md:p-8 flex-1 overflow-y-auto">
                                {step === 1 && (
                                    <div className="animate-in fade-in max-w-4xl mx-auto">
                                        <h4 className="font-bold text-indigo-700 flex items-center gap-2 mb-2"><span className="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-xs">Step 1</span> 워밍업 연습란</h4>
                                        <p className="text-gray-600 text-sm leading-relaxed mb-4">본문에 옅게 표시된 정답을 덮어쓰며 앞으로 나올 빈칸을 미리 연습해 보세요. <br/><strong className="text-indigo-600 mt-1 inline-block">💡 툴팁 기능:</strong> 본문에 마우스를 올리면 한어병음이 나타납니다!</p>
                                        
                                        <div className="bg-white rounded-xl border border-gray-200 shadow-md overflow-hidden mt-6 flex flex-col">
                                            <div className="bg-indigo-50/50 px-5 py-3 font-bold text-[13px] text-indigo-800 border-b border-indigo-100 shrink-0 flex justify-between items-center">
                                                <span>연습 어휘/구문 리스트</span>
                                                <span className="text-indigo-600 font-bold bg-white border border-indigo-100 px-3 py-1 rounded-full shadow-sm">총 {dictionaryList.length}개</span>
                                            </div>
                                            <div className="overflow-y-auto max-h-64 relative">
                                                <table className="w-full text-left border-collapse">
                                                    <thead className="bg-gray-50 sticky top-0 z-20 shadow-sm border-b border-gray-200">
                                                        <tr className="text-[12px] text-gray-500 uppercase tracking-wider">
                                                            <th className="py-3 px-4 font-extrabold w-12 text-center border-r border-gray-200">No.</th>
                                                            <th className="py-3 px-4 font-extrabold w-[30%] border-r border-gray-200">어휘/구문</th>
                                                            <th className="py-3 px-4 font-extrabold w-[25%] border-r border-gray-200">병음</th>
                                                            <th className="py-3 px-4 font-extrabold">의미</th>
                                                        </tr>
                                                    </thead>
                                                    <tbody className="divide-y divide-gray-100 bg-white">
                                                        {dictionaryList.map((item, i) => (
                                                            <tr key={i} className="hover:bg-indigo-50/80 transition-all duration-200 text-sm group cursor-default">
                                                                <td className="py-3 px-4 text-center text-gray-400 font-bold group-hover:text-indigo-600 transition-colors border-r border-gray-100">{i+1}</td>
                                                                <td className="py-3 px-4 font-bold text-gray-800 font-serif break-keep leading-snug border-r border-gray-100">{item.word}</td>
                                                                <td className="py-3 px-4 text-indigo-500 text-[13px] tracking-wide font-medium border-r border-gray-100">{item.pinyin}</td>
                                                                <td className="py-3 px-4 text-gray-600 text-[13px] break-keep leading-snug">{item.mean}</td>
                                                            </tr>
                                                        ))}
                                                    </tbody>
                                                </table>
                                            </div>
                                        </div>
                                    </div>
                                )}
                                {step === 2 && (
                                    <div className="animate-in fade-in max-w-4xl mx-auto">
                                        <h4 className="font-bold text-indigo-700 flex items-center gap-2 mb-2"><span className="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-xs">Step 2</span> 핵심 키워드 중심</h4>
                                        <p className="text-gray-600 text-sm leading-relaxed mb-4">문장의 핵심을 파악하는 주요 단어를 본문 빈칸에 채워보세요.</p>
                                    </div>
                                )}
                                {step === 3 && (
                                    <div className="animate-in fade-in max-w-4xl mx-auto">
                                        <h4 className="font-bold text-indigo-700 flex items-center gap-2 mb-2"><span className="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-xs">Step 3</span> 주요 구문 및 사자성어</h4>
                                        <p className="text-gray-600 text-sm leading-relaxed mb-4">중국어의 맛을 살리는 주요 구문과 사자성어를 빈칸에 채워보세요.</p>
                                    </div>
                                )}
                                {step === 4 && (
                                    <div className="animate-in fade-in max-w-4xl mx-auto">
                                        <h4 className="font-bold text-indigo-700 flex items-center gap-2 mb-2"><span className="bg-indigo-100 text-indigo-800 px-2 py-1 rounded text-xs">Step 4</span> 고난도 문장 자율 복원</h4>
                                        <p className="text-gray-600 text-sm leading-relaxed mb-4">문맥을 이해하고 주요 문장(절 단위)을 스스로 복원해 빈칸에 채워보세요.</p>
                                    </div>
                                )}

                                {evaluatedSteps[step] && evaluationResults[step] && (
                                    <div className="max-w-4xl mx-auto bg-indigo-50/50 p-5 rounded-xl border border-indigo-200 text-sm text-gray-800 mt-4 animate-in slide-in-from-bottom-2 shadow-inner">
                                        <div className="leading-relaxed" dangerouslySetInnerHTML={{ __html: formatFeedback(evaluationResults[step].teacherMessage) }} />
                                        
                                        {evaluationResults[step].evaluations && evaluationResults[step].evaluations.some(e => !e.isCorrect || e.isPartial) && (
                                            <div className="mt-4 pt-4 border-t border-indigo-200/60 space-y-3">
                                                <h5 className="font-bold text-indigo-800 text-xs">📝 오답 및 첨삭 노트</h5>
                                                {evaluationResults[step].evaluations.filter(e => !e.isCorrect || e.isPartial).map(e => (
                                                    <div key={e.id} className="bg-white p-3 rounded-lg border border-indigo-100 text-xs leading-relaxed shadow-sm">
                                                        <span className={`font-bold mr-2 ${e.isPartial ? 'text-yellow-600' : 'text-red-500'}`}>Q{e.id}.</span> 
                                                        {e.feedback}
                                                    </div>
                                                ))}
                                            </div>
                                        )}
                                    </div>
                                )}
                            </div>

                            <div className="p-4 md:p-5 bg-white border-t border-gray-200 shrink-0 flex justify-center shadow-[0_-4px_10px_rgba(0,0,0,0.03)] z-30">
                                {!evaluatedSteps[step] ? (
                                    <div className="w-full max-w-xl space-y-2">
                                        <p className="text-xs text-center text-gray-400">빈칸을 비워두면 오답 처리됩니다.</p>
                                        <button onClick={handleEvaluate} disabled={isEvaluating} className="w-full bg-indigo-600 text-white py-3.5 rounded-xl font-bold hover:bg-indigo-700 transition-colors shadow-md flex items-center justify-center gap-2 disabled:opacity-50">
                                            {isEvaluating ? (
                                                <><span className="animate-spin w-5 h-5 border-2 border-white border-t-transparent rounded-full"></span> AI 튜터 채점 중...</>
                                            ) : (
                                                <><Icon name="CheckCircle" size={20}/> 정답 제출 및 채점 받기</>
                                            )}
                                        </button>
                                    </div>
                                ) : (
                                    <button onClick={() => setStep(step + 1)} className="w-full max-w-xl bg-gray-800 text-white py-3.5 rounded-xl font-bold hover:bg-gray-900 transition-colors shadow-md flex items-center justify-center gap-2 animate-pulse">
                                        {step < 4 ? `Step ${step + 1} 단계로 이동` : '최종 결과 보기'}
                                    </button>
                                )}
                            </div>
                        </div>

                    </div>
                </div>
            );
        };

        const ArchiveDashboard = ({ userInfo, submissions }) => {
            const timeline = Array.from({ length: 140 }, () => Math.floor(Math.random() * 4));
            
            const mySubmissions = submissions ? submissions.filter(s => s.studentId === (userInfo ? userInfo.id : null)) : [];
            const myApproved = mySubmissions.filter(s => s.status === 'approved');
            const myPending = mySubmissions.filter(s => s.status === 'pending');

            const totalCount = 142 + myApproved.length;

            return (
                <div className="space-y-6 animate-in fade-in duration-300">
                    <div className="bg-white p-6 rounded-2xl border shadow-sm">
                        <div className="flex justify-between items-start mb-6">
                            <div>
                                <h3 className="text-2xl font-bold text-gray-800 mb-1">오늘도 정진하고 계시네요! 🔥</h3>
                                <p className="text-gray-500 text-sm">당신의 성장이 기록되고 있습니다.</p>
                            </div>
                        </div>
                        
                        <div className="grid grid-cols-2 gap-4 max-w-sm mb-8">
                            <div className="bg-blue-50/50 p-4 rounded-xl border border-blue-100 flex flex-col items-center justify-center">
                                <div className="text-sm text-gray-500 font-medium mb-1">누적 과제 수</div>
                                <div className="text-3xl font-bold text-blue-600">{totalCount}<span className="text-sm font-normal text-gray-400 ml-1">개</span></div>
                            </div>
                            <div className="bg-orange-50/50 p-4 rounded-xl border border-orange-100 flex flex-col items-center justify-center">
                                <div className="text-sm text-gray-500 font-medium mb-1">연속 출석</div>
                                <div className="text-3xl font-bold text-orange-500">12<span className="text-sm font-normal text-gray-400 ml-1">일</span></div>
                            </div>
                        </div>

                        <div className="mb-6">
                            <div className="flex justify-between items-center mb-4">
                                <h4 className="font-bold text-gray-700 flex items-center gap-2">
                                    <Icon name="Activity" size={18} className="text-gray-400"/> Archived Timeline
                                </h4>
                                <span className="text-xs text-gray-400 font-medium px-2 py-1 bg-gray-100 rounded">최근 6개월</span>
                            </div>
                            <div className="bg-gray-50 p-4 rounded-xl border overflow-x-auto hide-scrollbar">
                                <div className="grid grid-rows-5 gap-1.5 min-w-[600px] w-full" style={{ gridAutoFlow: 'column', gridAutoColumns: '12px' }}>
                                    {timeline.map((level, i) => (
                                        <div key={i} className={`w-3 h-3 rounded-sm ${level === 0 ? 'bg-gray-200' : level === 1 ? 'bg-blue-300' : level === 2 ? 'bg-blue-500' : 'bg-blue-700'}`} title={`${level}개의 과제 완료`}></div>
                                    ))}
                                </div>
                                <div className="flex justify-end items-center gap-2 mt-3 text-xs text-gray-500 font-medium">
                                    <span>적음</span>
                                    <div className="flex gap-1">
                                        <div className="w-3 h-3 rounded-sm bg-gray-200"></div>
                                        <div className="w-3 h-3 rounded-sm bg-blue-300"></div>
                                        <div className="w-3 h-3 rounded-sm bg-blue-500"></div>
                                        <div className="w-3 h-3 rounded-sm bg-blue-700"></div>
                                    </div>
                                    <span>많음</span>
                                </div>
                            </div>
                        </div>
                    </div>

                    <div className="bg-white p-6 rounded-2xl border shadow-sm">
                        <h4 className="font-bold text-gray-700 flex items-center gap-2 mb-4">
                            <Icon name="CheckCircle" size={18} className="text-green-500"/> 최근 완료한 과제
                        </h4>
                        <div className="space-y-3">
                            {myApproved.length > 0 ? myApproved.map(sub => (
                                <div key={sub.id} className="flex justify-between items-center p-4 bg-gray-50 border border-gray-100 rounded-xl hover:bg-gray-100 transition-colors">
                                    <div>
                                        <div className="text-xs text-gray-400 font-medium mb-1">구분: {sub.type === 'reading' ? '읽기' : sub.type === 'listening' ? '듣기' : '쓰기'}</div>
                                        <div className="font-bold text-gray-800 text-sm">{sub.title}</div>
                                    </div>
                                    <div className="flex items-center gap-4">
                                        <span className="text-blue-600 font-bold bg-blue-50 px-3 py-1 rounded-full text-sm border border-blue-100">{sub.score} 점</span>
                                        <button className="text-sm font-medium text-gray-500 hover:text-gray-800">기록 보기</button>
                                    </div>
                                </div>
                            )) : (
                                <div className="text-sm text-gray-500 py-2 px-2">아직 완료되어 결재된 과제가 없습니다.</div>
                            )}

                            <div className="flex justify-between items-center p-4 bg-gray-50 border border-gray-100 rounded-xl hover:bg-gray-100 transition-colors opacity-60 mt-4">
                                <div>
                                    <div className="text-xs text-gray-400 font-medium mb-1">구분: 어휘</div>
                                    <div className="font-bold text-gray-800 text-sm">HSK 5급 필수 어휘 100선 작문 테스트 (이전 기록)</div>
                                </div>
                                <div className="flex items-center gap-4">
                                    <span className="text-blue-600 font-bold bg-blue-50 px-3 py-1 rounded-full text-sm border border-blue-100">95 점</span>
                                    <button className="text-sm font-medium text-gray-500 hover:text-gray-800">기록 보기</button>
                                </div>
                            </div>
                        </div>
                    </div>

                    {myPending.length > 0 && (
                        <div className="bg-white p-6 rounded-2xl border shadow-sm">
                            <h4 className="font-bold text-gray-700 flex items-center gap-2 mb-4">
                                <Icon name="Activity" size={18} className="text-orange-500"/> 결재 대기 중인 과제
                            </h4>
                            <div className="space-y-3">
                                {myPending.map(sub => (
                                    <div key={sub.id} className="flex justify-between items-center p-4 bg-orange-50/30 border border-orange-100 rounded-xl">
                                        <div>
                                            <div className="text-xs text-orange-500 font-bold mb-1">상태: 선생님 확인 중...</div>
                                            <div className="font-bold text-gray-800 text-sm">{sub.title}</div>
                                            <div className="text-xs text-gray-500 mt-1">제출일시: {new Date(sub.submittedAt).toLocaleTimeString('ko-KR', { hour: '2-digit', minute: '2-digit' })}</div>
                                        </div>
                                        <div className="flex items-center gap-4">
                                            <span className="text-orange-700 font-bold bg-white px-3 py-1 rounded-full text-sm border border-orange-200 shadow-sm">
                                                AI 채점: {sub.score}점
                                            </span>
                                        </div>
                                    </div>
                                ))}
                            </div>
                        </div>
                    )}
                </div>
            )
        };

        const AdminSummary = ({ submissions, onApprove }) => {
            const pendingSubmissions = submissions ? submissions.filter(s => s.status === 'pending') : [];
            const totalSubmissionsCount = pendingSubmissions.length;

            const timeAgo = (dateString) => {
                const date = new Date(dateString);
                const minutes = Math.floor((new Date() - date) / 60000);
                if (minutes < 1) return '방금 전';
                if (minutes < 60) return `${minutes}분 전`;
                return `${Math.floor(minutes / 60)}시간 전`;
            };

            return (
                <div className="space-y-8 animate-in fade-in">
                    <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
                        <div className="bg-white p-5 rounded-2xl border shadow-sm">
                            <div className="text-sm font-medium text-gray-500 mb-2">미확인 제출 과제</div>
                            <div className="flex items-end gap-2 mb-3">
                                <span className="text-4xl font-bold text-rose-500">{totalSubmissionsCount}</span>
                                <span className="text-gray-500 font-medium mb-1">건</span>
                            </div>
                            <div className="text-xs text-gray-500 bg-gray-50 p-2 rounded flex items-center gap-1">
                                <Icon name="MessageSquare" size={14}/> 피드백 및 채점 필요
                            </div>
                        </div>
                        <div className="bg-white p-5 rounded-2xl border shadow-sm">
                            <div className="text-sm font-medium text-gray-500 mb-2">승인 대기 중인 학생</div>
                            <div className="flex items-end gap-2 mb-3">
                                <span className="text-4xl font-bold text-blue-600">3</span>
                                <span className="text-gray-500 font-medium mb-1">명</span>
                            </div>
                            <div className="text-xs text-gray-500 bg-gray-50 p-2 rounded">
                                학생 관리 메뉴 이동
                            </div>
                        </div>
                        <div className="bg-white p-5 rounded-2xl border shadow-sm">
                            <div className="text-sm font-medium text-gray-500 mb-2">오늘 접속한 학생</div>
                            <div className="flex items-end gap-2 mb-3">
                                <span className="text-4xl font-bold text-gray-800">42</span>
                                <span className="text-gray-400 font-medium mb-1">/ 50명</span>
                            </div>
                            <div className="text-xs text-gray-500 bg-gray-50 p-2 rounded">
                                출석률 84%
                            </div>
                        </div>
                    </div>

                    <div className="bg-white rounded-2xl border shadow-sm overflow-hidden">
                        <div className="p-5 border-b flex justify-between items-center bg-gray-50/50">
                            <h4 className="font-bold text-gray-800">최근 과제 제출 (AI 채점 완료 / 선생님 결재 대기)</h4>
                            <button className="text-sm text-indigo-600 font-medium hover:underline">모두 보기</button>
                        </div>
                        <div className="overflow-x-auto">
                            <table className="w-full text-left border-collapse">
                                <thead>
                                    <tr className="bg-white text-gray-400 text-xs uppercase tracking-wider border-b">
                                        <th className="p-4 font-medium">학생명 (Archive No.)</th>
                                        <th className="p-4 font-medium">과제명</th>
                                        <th className="p-4 font-medium">AI 채점 점수</th>
                                        <th className="p-4 font-medium">제출일시</th>
                                        <th className="p-4 font-medium text-right">관리</th>
                                    </tr>
                                </thead>
                                <tbody className="text-sm">
                                    {pendingSubmissions.length > 0 ? pendingSubmissions.map(sub => (
                                        <tr key={sub.id} className="border-b hover:bg-gray-50 transition-colors">
                                            <td className="p-4 font-medium text-gray-800">{sub.studentName} <span className="text-gray-400 text-xs font-normal">{sub.studentId}</span></td>
                                            <td className="p-4 text-gray-600">{sub.title}</td>
                                            <td className="p-4 font-bold text-indigo-600">{sub.score} 점</td>
                                            <td className="p-4 text-gray-500">{timeAgo(sub.submittedAt)}</td>
                                            <td className="p-4 text-right">
                                                <button onClick={() => onApprove(sub.id)} className="bg-indigo-600 text-white px-4 py-2 rounded-xl text-xs font-bold hover:bg-indigo-700 shadow-sm flex items-center justify-center gap-1 ml-auto transition-colors">
                                                    <Icon name="CheckCircle" size={14}/> 확인 및 결재
                                                </button>
                                            </td>
                                        </tr>
                                    )) : (
                                        <tr>
                                            <td colSpan="5" className="p-10 text-center text-gray-500">결재 대기 중인 과제가 없습니다.</td>
                                        </tr>
                                    )}
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
            );
        };

        const TaskCreationModal = ({ onClose, onSave, editData }) => {
            const [level, setLevel] = useState(editData ? editData.level : '6급');
            const [category, setCategory] = useState(editData ? editData.category : '읽기 훈련');
            const [subType, setSubType] = useState(editData ? editData.type : 'Scaffold Cloze');
            const [title, setTitle] = useState(editData ? editData.title : '');
            const [sourceText, setSourceText] = useState(editData ? editData.sourceText : '');
            const [isGenerating, setIsGenerating] = useState(false);
            const [aiResult, setAiResult] = useState(null);
            const [parsedData, setParsedData] = useState(null);
            const [errorMsg, setErrorMsg] = useState('');

            const categories = {
                '듣기 훈련': ['섀도잉', '에코잉', '받아쓰기'],
                '읽기 훈련': ['Scaffold Cloze', '시역 훈련'],
                'Layered Writing & Role-play': ['HSK Layered Writing', 'Role-play']
            };

            useEffect(() => {
                if (!editData || category !== editData.category) {
                    setSubType(categories[category][0]);
                }
            }, [category]);

            const handleGenerate = async () => {
                setErrorMsg('');
                if (!sourceText.trim()) {
                    setErrorMsg('소스 자료(중국어 원문)를 입력해주세요.');
                    return;
                }
                setIsGenerating(true);
                setAiResult(null);
                setParsedData(null);

                const apiKey = ""; 
                const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;
                
                const isSightTranslation = subType === '시역 훈련';
                
                let promptText = '';
                let schemaSchema = {};

                if (isSightTranslation) {
                    promptText = `다음 중국어 원문을 분석하여 HSK ${level} '시역 훈련' 데이터를 JSON 형식으로 생성해줘.
                    [요구사항]
                    1. title: 과제 제목 (한국어, 15자 이내)
                    2. summary: 지문 난이도 및 학습 포인트 요약 (한국어, 1~2줄)
                    3. chunks: 원문을 의미 단위(청크)로 자른 배열.
                       - 각 청크는 {"ko": "자연스러운 한국어 번역", "cn": "해당 중국어 청크 원문", "pinyin": "한어병음"} 형태.
                    
                    [원문]
                    ${sourceText}`;
                    
                    schemaSchema = {
                        type: "OBJECT",
                        properties: {
                            title: { type: "STRING" },
                            summary: { type: "STRING" },
                            chunks: {
                                type: "ARRAY",
                                items: {
                                    type: "OBJECT",
                                    properties: {
                                        ko: { type: "STRING" },
                                        cn: { type: "STRING" },
                                        pinyin: { type: "STRING" }
                                    }
                                }
                            }
                        }
                    };
                } else {
                    promptText = `다음 중국어 원문을 분석하여 HSK ${level} 'Scaffold Cloze' 훈련용 데이터를 JSON 형식으로 생성해줘.
                    [요구사항]
                    1. title: 과제 제목 (한국어, 15자 이내)
                    2. summary: 지문 난이도 및 학습 포인트 요약 (한국어, 1~2줄)
                    3. tokens: 원문을 순서대로 배열한 토큰 리스트. (텍스트 토큰과 빈칸 토큰)
                       - 일반 텍스트는 {"type": "text", "value": "원문텍스트"} (줄바꿈 기호 \n 포함 가능)
                       - 빈칸(blank)은 {"type": "blank", "value": "정답", "pinyin": "병음", "meaning": "한국어뜻", "step": 스텝번호}
                    4. 빈칸(blank) 할당 규칙 (반드시 총 30개의 빈칸을 생성할 것):
                       - Step 2 (핵심 단어): 단어 위주로 정확히 10개 선택 (step: 2)
                       - Step 3 (사자성어/구문): 구문 위주로 정확히 10개 선택 (step: 3)
                       - Step 4 (문장/절): 의미 단위의 절을 정확히 10개 선택 (step: 4)
                       * Step 1은 2, 3, 4단계의 모든 빈칸(30개)을 모아 자동으로 워밍업 단계로 구성되므로 별도로 지정할 필요 없음.
                    
                    [원문]
                    ${sourceText}`;
                    
                    schemaSchema = {
                        type: "OBJECT",
                        properties: {
                            title: { type: "STRING" },
                            summary: { type: "STRING" },
                            tokens: {
                                type: "ARRAY",
                                items: {
                                    type: "OBJECT",
                                    properties: {
                                        type: { type: "STRING", description: "'text' or 'blank'" },
                                        value: { type: "STRING" },
                                        pinyin: { type: "STRING" },
                                        meaning: { type: "STRING" },
                                        step: { type: "INTEGER" }
                                    }
                                }
                            }
                        }
                    };
                }

                const payload = {
                    contents: [{ parts: [{ text: promptText }] }],
                    generationConfig: { 
                        responseMimeType: "application/json",
                        responseSchema: schemaSchema
                    }
                };

                let resultObj = null;
                const maxRetries = 5;
                const delays = [1000, 2000, 4000, 8000, 16000];

                for (let i = 0; i < maxRetries; i++) {
                    try {
                        const response = await fetch(url, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (!response.ok) throw new Error('API Request Failed');
                        
                        const data = await response.json();
                        const resultText = data.candidates?.[0]?.content?.parts?.[0]?.text;
                        
                        if (resultText) {
                            resultObj = JSON.parse(resultText);
                        }
                        break;
                    } catch (e) {
                        if (i === maxRetries - 1) {
                            console.error("AI Generation Error:", e);
                            if (isSightTranslation) {
                                resultObj = {
                                    title: "지구 온난화와 기후 공학",
                                    summary: "기후 변화에 대한 전문가들의 다양한 의견을 분석하는 고급 시역 훈련입니다.",
                                    chunks: [
                                        { ko: "연구에 따르면,", cn: "有研究表明：", pinyin: "yǒu yánjiū biǎomíng:" },
                                        { ko: "지표면 온도의 변화 문제와 비교할 때,", cn: "与地表温度的变化问题相比，", pinyin: "yǔ dìbiǎo wēndù de biànhuà wèntí xiāng bǐ," },
                                        { ko: "근해 구역에 사는 사람들은 해수면 상승 문제에 더 관심을 갖습니다.", cn: "生活在近海区域的人们更关心海平面上升的问题；", pinyin: "shēnghuó zài jìnhǎi qūyù de rénmen gèng guānxīn hǎipíngmiàn shàngshēng de wèntí;" }
                                    ]
                                };
                            } else {
                                resultObj = {
                                    title: "일사량 조절과 지구 온난화",
                                    summary: "HSK 6급 수준의 환경·과학 지문으로, 태양 복사량 조절을 통한 온난화 해결 방안과 그에 따른 부작용을 분석합니다.",
                                    tokens: [
                                        { type: "text", value: "阳光或者在上层大气中放置" },
                                        { type: "blank", value: "气溶胶", pinyin: "qìróngjiāo", meaning: "에어로졸", step: 2 },
                                        { type: "text", value: "粒子模拟火山爆发效应等。\n有研究表明：与地表温度的变化问题相比，生活在" },
                                        { type: "blank", value: "近海区域", pinyin: "jìnhǎi qūyù", meaning: "근해 구역", step: 2 },
                                        { type: "text", value: "的人们更关心海平面上升的问题；而那些远离海域的人们则认为，影响农业生产或能源消耗的气温上升率问题更值得" },
                                        { type: "blank", value: "关注", pinyin: "guānzhù", meaning: "관심을 갖다", step: 2 },
                                        { type: "text", value: "。\n" },
                                        { type: "blank", value: "有科学家指出：强制性地阻止海平面升高可能导致气温快速下降", pinyin: "yǒu kēxuéjiā zhǐchū...", meaning: "과학자들은 강제적으로 해수면 상승을 막는 것이 기온 급강하를 초래할 수 있다고 지적한다", step: 4 },
                                        { type: "text", value: "。温度" },
                                        { type: "blank", value: "骤降", pinyin: "zhòujiàng", meaning: "급강하하다", step: 3 },
                                        { type: "text", value: "的危害性可能会超过因碳排放导致全球变暖所带来的危害性，温度下降的速度超过了地球上动植物所能适应的程度，" },
                                        { type: "blank", value: "将是人们需要面对的另一个大问题", pinyin: "jiāng shì rénmen xūyào miànduì de...", meaning: "사람들이 직면해야 할 또 다른 큰 문제가 될 것이다", step: 4 },
                                        { type: "text", value: "。" }
                                    ]
                                };
                            }
                        } else {
                            await new Promise(r => setTimeout(r, delays[i]));
                        }
                    }
                }

                if (resultObj) {
                    if (!title) setTitle(resultObj.title || '새로운 과제');
                    setAiResult(`[AI 분석 완료] 난이도 및 포인트: ${resultObj.summary || '분석이 완료되었습니다. (요약 생략됨)'}`);
                    setParsedData(resultObj);
                }
                
                setIsGenerating(false);
            };

            const handleSaveClick = () => {
                setErrorMsg('');
                if (!title.trim() || (!sourceText.trim() && !(editData && editData.isLegacy))) {
                    setErrorMsg('과제 제목과 소스 자료를 모두 작성해야 합니다.');
                    return;
                }
                
                const hasValidData = parsedData || (editData && editData.taskData) || (editData && editData.isLegacy);
                if ((subType === 'Scaffold Cloze' || subType === '시역 훈련') && !hasValidData) {
                    setErrorMsg('먼저 AI 콘텐츠 자동 제작 버튼을 눌러 데이터를 생성해 주세요.');
                    return;
                }

                let taskData = editData ? editData.taskData : null;
                
                if (subType === 'Scaffold Cloze' && parsedData && Array.isArray(parsedData.tokens)) {
                    let dictList = [];
                    let answers = { 1: {}, 2: {}, 3: {}, 4: {} };
                    let globalIndex = 1;
                    let stepCounts = { 1: 1, 2: 1, 3: 1, 4: 1, 5: 1 };

                    const processedTokens = parsedData.tokens.map(tok => {
                        if (tok.type === 'blank') {
                            let s = parseInt(tok.step);
                            if (isNaN(s) || s < 2 || s > 4) s = 2;
                            
                            if (!answers[s]) answers[s] = {};
                            if (!stepCounts[s]) stepCounts[s] = 1;

                            const currentStepIdx = stepCounts[s]++;
                            const gIdx = globalIndex++;
                            const wordValue = tok.value || '';

                            dictList.push({
                                word: wordValue,
                                pinyin: tok.pinyin || '',
                                mean: tok.meaning || ''
                            });

                            answers[1][gIdx] = wordValue;
                            answers[s][currentStepIdx] = wordValue;

                            return {
                                ...tok,
                                value: wordValue,
                                stepMap: { 1: gIdx, [s]: currentStepIdx }
                            };
                        }
                        return { ...tok, value: tok.value || '' };
                    });

                    taskData = {
                        dictionaryList: dictList,
                        correctAnswers: answers,
                        tokens: processedTokens
                    };
                } else if (subType === '시역 훈련' && parsedData && Array.isArray(parsedData.chunks)) {
                    taskData = {
                        chunks: parsedData.chunks.map(c => ({
                            ko: c.ko || '',
                            cn: c.cn || '',
                            pinyin: c.pinyin || ''
                        }))
                    };
                }

                onSave({
                    id: editData ? editData.id : undefined,
                    level,
                    category,
                    title,
                    type: subType,
                    status: editData ? editData.status : '활성',
                    date: editData ? editData.date : new Date().toISOString().split('T')[0],
                    isLegacy: false,
                    taskData,
                    sourceText
                });
                
                onClose();
            };

            return (
                <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4">
                    <div className="bg-white w-full max-w-4xl rounded-2xl shadow-2xl flex flex-col max-h-[90vh] animate-in fade-in zoom-in-95 duration-200">
                        <div className="px-6 py-4 border-b border-gray-100 flex justify-between items-center bg-gray-50/50 rounded-t-2xl">
                            <h3 className="font-bold text-lg text-gray-800 flex items-center gap-2">
                                <div className="p-1.5 bg-indigo-600 text-white rounded-lg"><Icon name={editData ? "Edit3" : "Sparkles"} size={18} /></div>
                                {editData ? '과제 콘텐츠 편집' : '새 과제 콘텐츠 AI 에디터'}
                            </h3>
                            <button onClick={onClose} className="text-gray-400 hover:text-gray-700 hover:bg-gray-100 p-1.5 rounded-lg transition-colors">
                                <Icon name="X" size={20} />
                            </button>
                        </div>
                        
                        <div className="p-6 flex-1 overflow-y-auto space-y-6">
                            {errorMsg && (
                                <div className="bg-rose-50 text-rose-600 border border-rose-200 p-4 rounded-xl text-sm font-bold animate-in fade-in flex items-center gap-3">
                                    <Icon name="ShieldAlert" size={20} />
                                    {errorMsg}
                                </div>
                            )}
                            <div className="flex flex-col md:flex-row gap-6">
                                <div className="w-full md:w-1/3 space-y-5">
                                    <div>
                                        <label className="block text-sm font-bold text-gray-700 mb-2">대상 급수</label>
                                        <select value={level} onChange={e => setLevel(e.target.value)} className="w-full px-4 py-2.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-indigo-500 outline-none text-sm font-medium">
                                            {['1-2급', '3급', '4급', '5급', '6급', '7-9급'].map(l => <option key={l} value={l}>{l}</option>)}
                                        </select>
                                    </div>
                                    <div>
                                        <label className="block text-sm font-bold text-gray-700 mb-2">훈련 유형</label>
                                        <div className="flex flex-col gap-2">
                                            {Object.keys(categories).map(cat => (
                                                <button key={cat} onClick={() => setCategory(cat)} className={`px-4 py-2.5 rounded-xl text-sm font-bold text-left transition-all ${category === cat ? 'bg-indigo-50 text-indigo-700 border border-indigo-200' : 'bg-white text-gray-600 border border-gray-200 hover:bg-gray-50'}`}>
                                                    {cat}
                                                </button>
                                            ))}
                                        </div>
                                    </div>
                                </div>
                                <div className="w-full md:w-2/3 space-y-5">
                                    <div>
                                        <label className="block text-sm font-bold text-gray-700 mb-2">세부 항목 설정</label>
                                        <div className="flex flex-wrap gap-2">
                                            {categories[category].map(sub => (
                                                <button key={sub} onClick={() => setSubType(sub)} className={`px-4 py-2 rounded-full text-xs font-bold transition-all ${subType === sub ? 'bg-gray-800 text-white shadow-md' : 'bg-gray-100 text-gray-500 hover:bg-gray-200'}`}>
                                                    {sub}
                                                </button>
                                            ))}
                                        </div>
                                    </div>
                                    <div>
                                        <label className="block text-sm font-bold text-gray-700 mb-2 flex justify-between">
                                            <span>소스 자료 입력</span>
                                            <span className="text-xs font-normal text-indigo-500">원문을 복사해 붙여넣으세요.</span>
                                        </label>
                                        <textarea 
                                            value={sourceText} 
                                            onChange={e => {setSourceText(e.target.value); setErrorMsg('');}} 
                                            placeholder="중국어 원문을 여기에 붙여넣으면 AI가 난이도를 분석하고 훈련 유형에 맞게 자동으로 콘텐츠 분해 및 생성합니다..."
                                            className="w-full h-40 p-4 border border-gray-200 rounded-xl focus:ring-2 focus:ring-indigo-500 outline-none resize-none text-sm font-serif leading-relaxed"
                                        />
                                    </div>
                                </div>
                            </div>
                            
                            <div className="bg-indigo-50/50 border border-indigo-100 rounded-xl p-5">
                                <div className="flex flex-col sm:flex-row gap-4 items-end">
                                    <div className="flex-1 w-full">
                                        <label className="block text-xs font-bold text-indigo-800 mb-1">과제 제목</label>
                                        <input 
                                            type="text" 
                                            value={title} 
                                            onChange={e => {setTitle(e.target.value); setErrorMsg('');}} 
                                            placeholder="AI가 원문을 분석하여 추천 제목을 지어줍니다 (직접 수정 가능)"
                                            className="w-full px-4 py-2.5 rounded-lg border border-indigo-200 focus:border-indigo-500 outline-none text-sm font-bold text-gray-800 bg-white"
                                        />
                                    </div>
                                    <button onClick={handleGenerate} disabled={isGenerating} className="w-full sm:w-auto px-6 py-2.5 bg-indigo-600 text-white rounded-lg font-bold hover:bg-indigo-700 transition-colors shadow-md flex items-center justify-center gap-2 shrink-0 disabled:opacity-50">
                                        {isGenerating ? <span className="animate-spin w-4 h-4 border-2 border-white border-t-transparent rounded-full"></span> : <Icon name="Sparkles" size={16}/>}
                                        {editData && parsedData ? 'AI 콘텐츠 재제작' : 'AI 콘텐츠 자동 제작'}
                                    </button>
                                </div>
                                {aiResult && (
                                    <div className="mt-4 p-3 bg-white rounded-lg border border-indigo-100 text-sm text-gray-700 leading-relaxed shadow-sm">
                                        <span className="font-bold text-indigo-600">AI 피드백: </span>{aiResult}
                                    </div>
                                )}
                            </div>
                        </div>

                        <div className="px-6 py-4 border-t border-gray-100 bg-gray-50/50 rounded-b-2xl flex justify-end gap-3">
                            <button onClick={onClose} className="px-5 py-2.5 rounded-xl font-bold text-gray-600 hover:bg-gray-200 transition-colors text-sm">취소</button>
                            <button onClick={handleSaveClick} className="px-6 py-2.5 rounded-xl bg-gray-800 text-white font-bold hover:bg-gray-900 transition-colors shadow-md text-sm flex items-center gap-2">
                                <Icon name="CheckCircle" size={18} />
                                {editData ? '수정 내용 저장하기' : '과제 저장 및 확정하기'}
                            </button>
                        </div>
                    </div>
                </div>
            );
        };

        const AdminTaskManager = ({ onPreview, tasks, onSaveNewTask, onDeleteTask, onUpdateTaskStatus }) => {
            const [level, setLevel] = useState('6급');
            const [category, setCategory] = useState('읽기 훈련');
            const [isAddingTask, setIsAddingTask] = useState(false);
            const [editingTask, setEditingTask] = useState(null);
            const [confirmingDelete, setConfirmingDelete] = useState(null);

            const filteredTasks = tasks.filter(t => t.level === level && t.category === category);

            const handleToggleStatus = (id, currentStatus) => {
                onUpdateTaskStatus(id, currentStatus);
            };

            const handleDelete = (id) => {
                setConfirmingDelete(id);
            };

            const confirmDelete = (id) => {
                onDeleteTask(id);
                setConfirmingDelete(null);
            };

            const handleSaveEditedTask = (updatedTaskData) => {
                onSaveNewTask(updatedTaskData);
                setEditingTask(null);
            };

            const levels = ['1-2급', '3급', '4급', '5급', '6급', '7-9급'];
            const categories = ['듣기 훈련', '읽기 훈련', 'Layered Writing & Role-play'];

            return (
                <>
                <div className="flex flex-col xl:flex-row gap-6 animate-in fade-in h-full">
                    <div className="w-full xl:w-72 shrink-0 space-y-6 flex flex-col">
                        <div className="bg-white p-6 rounded-2xl border shadow-sm">
                             <h3 className="font-bold text-gray-800 mb-4 flex items-center gap-2"><Icon name="Archive" size={18}/> HSK 급수 선택</h3>
                             <div className="flex flex-wrap gap-2">
                                 {levels.map(l => (
                                     <button key={l} onClick={() => setLevel(l)} 
                                        className={`px-4 py-2.5 rounded-xl text-sm font-bold transition-all flex-grow text-center ${
                                            level === l 
                                            ? 'bg-indigo-600 text-white shadow-md' 
                                            : 'bg-gray-50 text-gray-500 hover:bg-gray-100 border border-gray-200'
                                        }`}>
                                        {l}
                                     </button>
                                 ))}
                             </div>
                        </div>

                        <div className="bg-white p-6 rounded-2xl border shadow-sm flex-1">
                             <h3 className="font-bold text-gray-800 mb-4 flex items-center gap-2"><Icon name="FileText" size={18}/> 과제 카테고리 (영역)</h3>
                             <div className="flex flex-col gap-2">
                                 {categories.map(c => (
                                     <button key={c} onClick={() => setCategory(c)} 
                                        className={`px-5 py-3.5 rounded-xl text-[13px] font-bold text-left transition-all flex items-center justify-between ${
                                            category === c 
                                            ? 'bg-blue-50 text-blue-700 border-2 border-blue-200 shadow-sm' 
                                            : 'bg-white text-gray-600 border-2 border-transparent hover:bg-gray-50'
                                        }`}>
                                        {c}
                                        {category === c && <Icon name="CheckCircle" size={16}/>}
                                     </button>
                                 ))}
                             </div>
                        </div>
                    </div>

                    <div className="flex-1 bg-white rounded-2xl border shadow-sm overflow-hidden flex flex-col min-h-[500px]">
                        <div className="p-5 md:p-6 border-b flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 bg-gray-50/50">
                            <div>
                                <h4 className="font-extrabold text-lg text-gray-800"><span className="text-indigo-600">{level} / {category}</span> 콘텐츠</h4>
                                <p className="text-xs text-gray-500 mt-1">학생들에게 배정될 과제 콘텐츠를 편집하고 연동할 수 있습니다.</p>
                            </div>
                            <button onClick={() => setIsAddingTask(true)} className="bg-[#4f46e5] text-white px-5 py-2.5 rounded-xl text-sm font-bold hover:bg-indigo-700 shadow-md flex items-center gap-2 transition-all">
                                <Icon name="Plus" size={18} strokeWidth={3}/> 새 과제 추가
                            </button>
                        </div>

                        <div className="flex-1 overflow-x-auto p-0">
                            <table className="w-full text-left border-collapse min-w-[700px]">
                                <thead>
                                    <tr className="bg-gray-50 text-gray-500 text-[11px] uppercase tracking-wider border-b border-gray-200">
                                        <th className="p-4 font-bold w-12 text-center">ID</th>
                                        <th className="p-4 font-bold">과제 제목</th>
                                        <th className="p-4 font-bold w-36">유형</th>
                                        <th className="p-4 font-bold w-28 text-center">상태</th>
                                        <th className="p-4 font-bold w-32">생성일</th>
                                        <th className="p-4 font-bold w-48 text-center">관리 (편집/미리보기)</th>
                                    </tr>
                                </thead>
                                <tbody className="text-sm">
                                    {filteredTasks.length > 0 ? filteredTasks.map(t => (
                                        <tr key={t.id} className="border-b border-gray-100 hover:bg-indigo-50/30 transition-colors group">
                                            <td className="p-4 text-center text-gray-400 font-bold">{t.id}</td>
                                            <td className="p-4 font-bold text-gray-800 group-hover:text-indigo-700 transition-colors">{t.title}</td>
                                            <td className="p-4 text-gray-600 font-medium">
                                                <span className="bg-gray-100 px-2.5 py-1 rounded-md text-xs">{t.type}</span>
                                            </td>
                                            <td className="p-4 text-center">
                                                <button 
                                                    onClick={() => handleToggleStatus(t.id, t.status)}
                                                    className={`px-3 py-1 rounded-full text-[11px] font-bold cursor-pointer transition-all shadow-sm hover:scale-105 active:scale-95 ${
                                                        t.status === '활성' ? 'bg-green-100 text-green-700 border border-green-200 hover:bg-green-200' : 'bg-gray-100 text-gray-500 border border-gray-200 hover:bg-gray-200'
                                                    }`}
                                                    title="클릭하여 활성/비활성 전환"
                                                >
                                                    {t.status}
                                                </button>
                                            </td>
                                            <td className="p-4 text-gray-400 text-xs font-medium">{t.date}</td>
                                            <td className="p-4">
                                                <div className="flex items-center justify-center gap-2">
                                                    {(t.type === 'Scaffold Cloze' || t.type === '시역 훈련') ? (
                                                        <button onClick={() => onPreview(t)} className="p-2 bg-blue-50 text-blue-600 rounded-lg hover:bg-blue-600 hover:text-white transition-all shadow-sm group-hover:scale-105" title="실제 화면 미리보기 연동">
                                                            <Icon name="Eye" size={18}/>
                                                        </button>
                                                    ) : (
                                                        <button onClick={() => {}} className="p-2 bg-gray-50 text-gray-400 rounded-lg hover:bg-gray-200 transition-all" title="미리보기 (준비중)">
                                                            <Icon name="Eye" size={18}/>
                                                        </button>
                                                    )}
                                                    <button onClick={() => setEditingTask(t)} className="p-2 bg-gray-50 text-gray-600 rounded-lg hover:bg-indigo-50 hover:text-indigo-600 transition-all shadow-sm hover:scale-105" title="콘텐츠 편집">
                                                        <Icon name="Edit3" size={18}/>
                                                    </button>
                                                    {confirmingDelete === t.id ? (
                                                        <button onClick={() => confirmDelete(t.id)} className="p-2 bg-rose-500 text-white rounded-lg hover:bg-rose-600 transition-all shadow-sm group-hover:scale-105" title="삭제 확정">
                                                            <Icon name="CheckCircle" size={18}/>
                                                        </button>
                                                    ) : (
                                                        <button onClick={() => handleDelete(t.id)} className="p-2 bg-gray-50 text-gray-600 rounded-lg hover:bg-rose-50 hover:text-rose-600 transition-all shadow-sm hover:scale-105" title="삭제">
                                                            <Icon name="Trash2" size={18}/>
                                                        </button>
                                                    )}
                                                </div>
                                            </td>
                                        </tr>
                                    )) : (
                                        <tr>
                                            <td colSpan="6" className="p-16 text-center text-gray-400">
                                                <div className="flex flex-col items-center gap-3">
                                                    <Icon name="Archive" size={32} className="opacity-20"/>
                                                    <p>해당 카테고리에 등록된 과제 콘텐츠가 없습니다.</p>
                                                </div>
                                            </td>
                                        </tr>
                                    )}
                                </tbody>
                            </table>
                        </div>
                    </div>
                </div>
                
                {/* AI 과제 추가 및 편집 모달 마운트 */}
                {(isAddingTask || editingTask) && (
                    <TaskCreationModal 
                        editData={editingTask}
                        onClose={() => { setIsAddingTask(false); setEditingTask(null); }} 
                        onSave={(data) => {
                            if (editingTask) {
                                handleSaveEditedTask(data);
                            } else {
                                onSaveNewTask(data);
                            }
                        }} 
                    />
                )}
                </>
            );
        };

        const StudentCreationModal = ({ onClose, onSave, editData }) => {
            const [name, setName] = useState(editData ? editData.name : '');
            const [email, setEmail] = useState(editData ? editData.email : '');
            const [password, setPassword] = useState(editData ? editData.password : '');
            const [level, setLevel] = useState(editData ? editData.level : '6급');
            const [errorMsg, setErrorMsg] = useState('');

            const handleSave = () => {
                if (!name.trim() || !email.trim() || !password.trim()) {
                    setErrorMsg('이름, 이메일, 비밀번호를 모두 입력해주세요.');
                    return;
                }
                onSave({
                    id: editData ? editData.id : null,
                    name,
                    email,
                    password,
                    level,
                    createdAt: editData ? editData.createdAt : new Date().toISOString()
                });
                onClose();
            };

            return (
                <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm z-50 flex items-center justify-center p-4">
                    <div className="bg-white w-full max-w-md rounded-2xl shadow-2xl flex flex-col animate-in fade-in zoom-in-95 duration-200">
                        <div className="px-6 py-4 border-b border-gray-100 flex justify-between items-center bg-gray-50/50 rounded-t-2xl">
                            <h3 className="font-bold text-lg text-gray-800 flex items-center gap-2">
                                <div className="p-1.5 bg-indigo-600 text-white rounded-lg"><Icon name={editData ? "Edit3" : "Users"} size={18} /></div>
                                {editData ? '학생 정보 수정' : '신규 학생 등록'}
                            </h3>
                            <button onClick={onClose} className="text-gray-400 hover:text-gray-700 hover:bg-gray-100 p-1.5 rounded-lg transition-colors">
                                <Icon name="X" size={20} />
                            </button>
                        </div>
                        
                        <div className="p-6 space-y-4">
                            {errorMsg && <div className="bg-rose-50 text-rose-600 p-3 rounded-lg text-sm font-bold">{errorMsg}</div>}
                            <div>
                                <label className="block text-sm font-bold text-gray-700 mb-1">학생 이름</label>
                                <input type="text" value={name} onChange={e => setName(e.target.value)} placeholder="홍길동" className="w-full px-4 py-2.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-indigo-500 outline-none text-sm" />
                            </div>
                            <div>
                                <label className="block text-sm font-bold text-gray-700 mb-1">로그인 이메일</label>
                                <input type="email" value={email} onChange={e => setEmail(e.target.value)} placeholder="student@hsktc.com" className="w-full px-4 py-2.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-indigo-500 outline-none text-sm" />
                            </div>
                            <div>
                                <label className="block text-sm font-bold text-gray-700 mb-1">비밀번호</label>
                                <input type="text" value={password} onChange={e => setPassword(e.target.value)} placeholder="접속 비밀번호" className="w-full px-4 py-2.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-indigo-500 outline-none text-sm" />
                            </div>
                            <div>
                                <label className="block text-sm font-bold text-gray-700 mb-1">목표/현재 급수</label>
                                <select value={level} onChange={e => setLevel(e.target.value)} className="w-full px-4 py-2.5 rounded-xl border border-gray-200 focus:ring-2 focus:ring-indigo-500 outline-none text-sm font-medium">
                                    {['1-2급', '3급', '4급', '5급', '6급', '7-9급'].map(l => <option key={l} value={l}>{l}</option>)}
                                </select>
                            </div>
                        </div>

                        <div className="px-6 py-4 border-t border-gray-100 bg-gray-50/50 rounded-b-2xl flex justify-end gap-3">
                            <button onClick={onClose} className="px-5 py-2.5 rounded-xl font-bold text-gray-600 hover:bg-gray-200 transition-colors text-sm">취소</button>
                            <button onClick={handleSave} className="px-6 py-2.5 rounded-xl bg-indigo-600 text-white font-bold hover:bg-indigo-700 transition-colors shadow-md text-sm">
                                {editData ? '수정 내용 저장' : '학생 계정 생성'}
                            </button>
                        </div>
                    </div>
                </div>
            );
        };

        const AdminStudentManager = ({ usersList, onSaveUser, onDeleteUser }) => {
            const [isAdding, setIsAdding] = useState(false);
            const [editingUser, setEditingUser] = useState(null);
            const [confirmingDelete, setConfirmingDelete] = useState(null);

            const handleDelete = (id) => {
                setConfirmingDelete(id);
            };

            const confirmDeleteUser = (id) => {
                onDeleteUser(id);
                setConfirmingDelete(null);
            };

            return (
                <div className="bg-white rounded-2xl border shadow-sm overflow-hidden flex flex-col min-h-[500px] animate-in fade-in">
                    <div className="p-5 md:p-6 border-b flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 bg-gray-50/50">
                        <div>
                            <h4 className="font-extrabold text-lg text-gray-800">등록된 학생 목록</h4>
                            <p className="text-xs text-gray-500 mt-1">학생 계정을 생성하고, 급수 및 비밀번호를 관리할 수 있습니다.</p>
                        </div>
                        <button onClick={() => setIsAdding(true)} className="bg-[#4f46e5] text-white px-5 py-2.5 rounded-xl text-sm font-bold hover:bg-indigo-700 shadow-md flex items-center gap-2 transition-all">
                            <Icon name="Plus" size={18} strokeWidth={3}/> 새 학생 추가
                        </button>
                    </div>

                    <div className="flex-1 overflow-x-auto p-0">
                        <table className="w-full text-left border-collapse min-w-[700px]">
                            <thead>
                                <tr className="bg-gray-50 text-gray-500 text-[11px] uppercase tracking-wider border-b border-gray-200">
                                    <th className="p-4 font-bold w-16 text-center">ID</th>
                                    <th className="p-4 font-bold w-32">이름</th>
                                    <th className="p-4 font-bold">이메일 계정</th>
                                    <th className="p-4 font-bold w-32">비밀번호</th>
                                    <th className="p-4 font-bold w-24 text-center">급수</th>
                                    <th className="p-4 font-bold w-32 text-center">등록일</th>
                                    <th className="p-4 font-bold w-32 text-center">관리</th>
                                </tr>
                            </thead>
                            <tbody className="text-sm">
                                {usersList.length > 0 ? usersList.map(u => (
                                    <tr key={u.id} className="border-b border-gray-100 hover:bg-indigo-50/30 transition-colors group">
                                        <td className="p-4 text-center text-gray-400 font-bold text-xs">{u.id}</td>
                                        <td className="p-4 font-bold text-gray-800 group-hover:text-indigo-700 transition-colors">{u.name}</td>
                                        <td className="p-4 text-gray-600 font-medium">{u.email}</td>
                                        <td className="p-4 text-gray-400 font-mono tracking-wider">{u.password}</td>
                                        <td className="p-4 text-center">
                                            <span className="bg-blue-50 text-blue-600 border border-blue-200 px-2.5 py-1 rounded-md text-xs font-bold">{u.level}</span>
                                        </td>
                                        <td className="p-4 text-gray-400 text-xs font-medium text-center">{new Date(u.createdAt).toLocaleDateString()}</td>
                                        <td className="p-4 text-center">
                                            <div className="flex items-center justify-center gap-2">
                                                <button onClick={() => setEditingUser(u)} className="p-2 bg-gray-50 text-gray-600 rounded-lg hover:bg-indigo-50 hover:text-indigo-600 transition-all shadow-sm hover:scale-105" title="정보 수정">
                                                    <Icon name="Edit3" size={18}/>
                                                </button>
                                                {confirmingDelete === u.id ? (
                                                    <button onClick={() => confirmDeleteUser(u.id)} className="p-2 bg-rose-500 text-white rounded-lg hover:bg-rose-600 transition-all shadow-sm group-hover:scale-105" title="삭제 확정">
                                                        <Icon name="CheckCircle" size={18}/>
                                                    </button>
                                                ) : (
                                                    <button onClick={() => handleDelete(u.id)} className="p-2 bg-gray-50 text-gray-600 rounded-lg hover:bg-rose-50 hover:text-rose-600 transition-all shadow-sm hover:scale-105" title="계정 삭제">
                                                        <Icon name="Trash2" size={18}/>
                                                    </button>
                                                )}
                                            </div>
                                        </td>
                                    </tr>
                                )) : (
                                    <tr>
                                        <td colSpan="7" className="p-16 text-center text-gray-400">
                                            <div className="flex flex-col items-center gap-3">
                                                <Icon name="Users" size={32} className="opacity-20"/>
                                                <p>등록된 학생이 없습니다. 우측 상단 버튼을 눌러 학생을 추가해주세요.</p>
                                            </div>
                                        </td>
                                    </tr>
                                )}
                            </tbody>
                        </table>
                    </div>

                    {(isAdding || editingUser) && (
                        <StudentCreationModal 
                            editData={editingUser}
                            onClose={() => { setIsAdding(false); setEditingUser(null); }}
                            onSave={(userData) => {
                                onSaveUser(userData);
                                setIsAdding(false);
                                setEditingUser(null);
                            }}
                        />
                    )}
                </div>
            );
        };

        const AdminDashboard = ({ submissions, onApprove, tasks, onSaveNewTask, onDeleteTask, onUpdateTaskStatus, usersList, onSaveUser, onDeleteUser }) => {
            const [activeTab, setActiveTab] = useState('tasks');
            const [previewMode, setPreviewMode] = useState(false);

            if (previewMode) {
                return (
                    <div className="flex flex-col h-full animate-in fade-in duration-300">
                        <div className="mb-4 shrink-0">
                             <button onClick={() => setPreviewMode(false)} className="flex items-center gap-2 text-gray-500 hover:text-indigo-600 font-bold transition-colors">
                                 <Icon name="ChevronLeft" size={20}/> 과제 콘텐츠 목록으로 돌아가기
                             </button>
                        </div>
                        <div className="flex-1 border-[3px] border-indigo-400 rounded-xl overflow-hidden shadow-[0_10px_40px_rgba(79,70,229,0.15)] relative flex flex-col bg-slate-50">
                             <div className="bg-indigo-600 text-white text-xs font-bold py-2 px-4 z-50 text-center tracking-widest flex items-center justify-center gap-2 shrink-0">
                                 <Icon name="Eye" size={16}/> 미리보기 모드 (실제 학생에게 보여지는 화면입니다)
                             </div>
                             <div className="flex-1 p-4 md:p-6 overflow-y-auto">
                                 {previewMode.type === 'Scaffold Cloze' && (
                                      <ClozeSystem setActiveMenu={() => setPreviewMode(false)} onSubmit={(data) => { setPreviewMode(false); }} taskConfig={previewMode} />
                                 )}
                                 {previewMode.type === '시역 훈련' && (
                                      <SightTranslationSystem setActiveMenu={() => setPreviewMode(false)} onSubmit={(data) => { setPreviewMode(false); }} taskConfig={previewMode} />
                                 )}
                             </div>
                        </div>
                    </div>
                );
            }

            return (
                <div className="flex flex-col h-full animate-in fade-in duration-300">
                    <div className="flex justify-between items-end mb-6 shrink-0">
                        <div>
                            <h2 className="text-2xl font-bold text-gray-800 mb-1 flex items-center gap-2">
                                <div className="p-1.5 bg-indigo-600 text-white rounded"><Icon name="ShieldAlert" size={20}/></div>
                                관리자 대시보드
                            </h2>
                            <p className="text-gray-500 text-sm">학생들의 과제 제출 현황과 기록을 관리하세요.</p>
                        </div>
                        <div className="px-3 py-1.5 bg-rose-50 text-rose-600 font-bold text-xs rounded-full border border-rose-200">
                            Teacher Mode
                        </div>
                    </div>

                    <div className="flex gap-6 border-b border-gray-200 mb-6 shrink-0">
                        <button onClick={() => setActiveTab('summary')} className={`pb-3 border-b-2 px-2 flex items-center gap-2 transition-colors ${activeTab === 'summary' ? 'border-indigo-600 font-bold text-indigo-600' : 'border-transparent font-medium text-gray-500 hover:text-gray-800'}`}>
                            <Icon name="LayoutDashboard" size={18}/> 요약 현황
                        </button>
                        <button onClick={() => setActiveTab('tasks')} className={`pb-3 border-b-2 px-2 flex items-center gap-2 transition-colors ${activeTab === 'tasks' ? 'border-indigo-600 font-bold text-indigo-600' : 'border-transparent font-medium text-gray-500 hover:text-gray-800'}`}>
                            <Icon name="FileText" size={18}/> 과제 콘텐츠 관리
                        </button>
                        <button onClick={() => setActiveTab('students')} className={`pb-3 border-b-2 px-2 flex items-center gap-2 transition-colors ${activeTab === 'students' ? 'border-indigo-600 font-bold text-indigo-600' : 'border-transparent font-medium text-gray-500 hover:text-gray-800'}`}>
                            <Icon name="Users" size={18}/> 학생 관리
                        </button>
                    </div>

                    <div className="flex-1 overflow-y-auto pb-10 hide-scrollbar">
                        {activeTab === 'summary' && <AdminSummary submissions={submissions} onApprove={onApprove} />}
                        {activeTab === 'tasks' && <AdminTaskManager onPreview={(t) => setPreviewMode(t)} tasks={tasks} onSaveNewTask={onSaveNewTask} onDeleteTask={onDeleteTask} onUpdateTaskStatus={onUpdateTaskStatus} />}
                        {activeTab === 'students' && <AdminStudentManager usersList={usersList} onSaveUser={onSaveUser} onDeleteUser={onDeleteUser} />}
                    </div>
                </div>
            )
        };

        const LayeredWritingChat = ({studentName}) => {
            const [isStarted, setIsStarted] = useState(false);
            const [messages, setMessages] = useState([]);
            const [input, setInput] = useState('');
            const [isGenerating, setIsGenerating] = useState(false);

            const systemInstruction = `
            너는 중국어 스크립트를 바탕으로 학생들이 논리적이고 깊이 있는 글을 쓰도록 돕는 'Layered Writing' 전문가이자 교육자야.

            [가이드 원칙: Layer 3단계]
            Layer 1: Core (30%) - 뼈대 구축: 핵심 키워드와 중심 문장을 통해 소스 텍스트의 본질을 포착했는지 확인해. 부족하면 질문을 통해 유도해.
            Layer 2: Logic (60%) - 연결고리 생성: 세부 맥락과 인과관계를 추가했는지 확인해. 특히 접속사 등을 활용한 문장 간의 논리적 연결성을 중점적으로 분석해.
            Layer 3: Creative (100%) - 사고의 확장: 자신의 언어로 재구성했는지, 주관적 견해나 응용 사례가 포함되었는지 확인하여 사고의 깊이를 확장해.

            [채점 및 통과 규칙]
            종합 평가 지표: 각 단계에서 ①내용의 논리성, ②표현의 정확성과 적절성, ③구조적 완성도를 종합하여 100점 만점의 백분율 점수를 산출하여 반드시 학생에게 알려 줘.
            70%의 벽: 점수가 70% 이상일 때만 "Pass"를 주고 다음 단계로 넘어가. 70% 미만이라면 어떤 지표에서 부족했는지 설명하고 다시 쓰도록 격려해.
            단계별 진행: 절대 한꺼번에 작성하게 하지 마. 반드시 Layer 1이 통과되어야 Layer 2로 진행할 수 있어.

            [상호작용 스타일]
            질문 중심: 즉각적인 교정보다는 "어떻게 하면 논리가 더 선명해질까?" 또는 "이 문장들을 잇는 더 적절한 접속사는 무엇일까?"와 같은 질문을 던져.
            언어 원칙: 모든 설명과 피드백은 한국어로 하되, 중국어 문장에 대한 구체적인 수정 제안이나 예시는 중국어로 제공해.
            톤앤매너: 친절하고 따뜻하며, 학생의 성장을 진심으로 응원하는 전문 교육자의 톤을 유지해.
            `;

            const startSession = () => {
                setIsStarted(true);
                setMessages([{
                    role: 'ai',
                    text: `안녕하세요, ${studentName} 학생! HSKTC Layered Writing 튜터입니다.\n\n오늘은 '성공하지 못하는 사람의 4가지 유형'이라는 주제로 글쓰기를 진행해 볼 거예요.\n\n먼저 **Layer 1: Core (뼈대 구축)** 단계입니다. 핵심 키워드와 중심 문장을 활용하여 본문의 핵심 내용을 중국어로 간단히 작성해 볼까요?`
                }]);
            };

            const handleSend = async () => {
                if(!input.trim() || isGenerating) return;

                const userText = input;
                setInput('');
                const newHistory = [...messages, { role: 'user', text: userText }];
                setMessages(newHistory);
                setIsGenerating(true);

                const apiKey = ""; 
                const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

                const contents = newHistory.map(msg => ({
                    role: msg.role === 'ai' ? 'model' : 'user',
                    parts: [{ text: msg.text }]
                }));

                const payload = {
                    contents: contents,
                    systemInstruction: { parts: [{ text: systemInstruction }] }
                };

                const maxRetries = 5;
                const delays = [1000, 2000, 4000, 8000, 16000];
                let aiText = "죄송합니다. 오류가 발생했습니다.";

                for (let i = 0; i < maxRetries; i++) {
                    try {
                        const response = await fetch(url, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (!response.ok) throw new Error('API Request Failed');
                        const data = await response.json();
                        aiText = data.candidates?.[0]?.content?.parts?.[0]?.text || "답변을 생성하지 못했습니다.";
                        break;
                    } catch (err) {
                        if (i === maxRetries - 1) aiText = "네트워크 오류로 답변을 생성하지 못했습니다. 다시 시도해 주세요.";
                        else await new Promise(r => setTimeout(r, delays[i]));
                    }
                }

                setMessages(prev => [...prev, { role: 'ai', text: aiText }]);
                setIsGenerating(false);
            };

            if (!isStarted) {
                return (
                    <div className="bg-white p-8 rounded-xl border shadow-sm h-full flex flex-col items-center justify-center text-center">
                        <div className="w-16 h-16 bg-blue-100 text-blue-600 rounded-full flex items-center justify-center mb-4 shadow-sm border border-blue-200">
                            <Icon name="PenTool" size={32}/>
                        </div>
                        <h3 className="text-xl font-bold mb-2 text-gray-800">HSKTC Layered Writing AI 튜터</h3>
                        <p className="text-gray-500 max-w-md text-sm leading-relaxed">
                            학생들의 단계적 사고 확장을 돕습니다. 소스 텍스트를 분석하여 
                            <br/><span className="font-bold text-blue-600">뼈대(Core) → 연결(Logic) → 확장(Creative)</span>의 
                            <br/>3단계 글쓰기 과정을 가이드하고 피드백을 제공합니다.
                        </p>
                        <button onClick={startSession} className="mt-8 bg-blue-600 text-white px-8 py-3 rounded-xl font-bold hover:bg-blue-700 transition-colors shadow-md flex items-center gap-2">
                            글쓰기 훈련 시작
                        </button>
                    </div>
                );
            }

            return (
                <div className="flex flex-col h-full bg-white rounded-xl border shadow-sm overflow-hidden">
                    <div className="bg-blue-600 text-white p-4 flex items-center justify-between">
                        <div className="flex items-center gap-3">
                            <div className="w-10 h-10 bg-white/20 rounded-full flex items-center justify-center backdrop-blur-sm">
                                <Icon name="PenTool" size={20} />
                            </div>
                            <div>
                                <div className="font-bold text-sm">Layered Writing AI 튜터</div>
                                <div className="text-xs text-blue-100 opacity-90">3단계 글쓰기 가이드 및 피드백</div>
                            </div>
                        </div>
                        <button onClick={() => {setIsStarted(false); setMessages([]); setInput('');}} className="text-xs font-medium bg-blue-700/50 hover:bg-blue-800/50 px-3 py-1.5 rounded-lg transition-colors">
                            초기화
                        </button>
                    </div>

                    <div className="flex-1 overflow-y-auto p-4 space-y-5 bg-slate-50">
                        {messages.map((msg, idx) => (
                            <div key={idx} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                                <div className={`max-w-[85%] rounded-2xl px-5 py-4 text-sm shadow-sm leading-relaxed ${
                                    msg.role === 'user'
                                    ? 'bg-blue-600 text-white rounded-tr-none'
                                    : 'bg-white border text-gray-800 rounded-tl-none'
                                }`}>
                                    {msg.role === 'user' && <div className="font-bold text-xs text-blue-200 mb-2">{studentName}</div>}
                                    {msg.role === 'ai' && <div className="font-bold text-xs text-gray-400 mb-2">AI 교육자</div>}
                                    <div dangerouslySetInnerHTML={{ __html: msg.text.replace(/\n/g, '<br/>').replace(/\*\*(.*?)\*\*/g, '<strong class="text-blue-700 bg-blue-50 px-1 rounded">$1</strong>') }} />
                                </div>
                            </div>
                        ))}
                        {isGenerating && (
                            <div className="flex justify-start">
                                <div className="bg-white border text-gray-500 rounded-2xl rounded-tl-none px-5 py-4 text-sm shadow-sm flex items-center gap-2">
                                    <span className="animate-bounce inline-block w-2 h-2 bg-gray-400 rounded-full"></span>
                                    <span className="animate-bounce inline-block w-2 h-2 bg-gray-400 rounded-full" style={{animationDelay: '0.2s'}}></span>
                                    <span className="animate-bounce inline-block w-2 h-2 bg-gray-400 rounded-full" style={{animationDelay: '0.4s'}}></span>
                                    <span className="ml-2 text-xs font-medium">피드백 분석 중...</span>
                                </div>
                            </div>
                        )}
                    </div>

                    <div className="p-4 bg-white border-t">
                        <div className="flex flex-col gap-3">
                            <textarea
                                value={input}
                                onChange={e => setInput(e.target.value)}
                                onKeyDown={e => {if(e.key==='Enter' && !e.shiftKey) { e.preventDefault(); handleSend();}}}
                                disabled={isGenerating}
                                placeholder="중국어로 글을 작성해주세요... (Shift+Enter로 줄바꿈)"
                                className="w-full border border-gray-300 rounded-xl px-4 py-3 text-sm focus:ring-2 focus:ring-blue-500 focus:border-blue-500 outline-none resize-none h-24 disabled:bg-gray-50 transition-all"
                            />
                            <div className="flex justify-between items-center">
                                <span className="text-xs text-gray-400">📝 단계별 규칙에 맞게 작성하면 AI가 채점합니다.</span>
                                <button onClick={handleSend} disabled={isGenerating || !input.trim()} className="bg-blue-600 text-white px-8 py-2.5 rounded-xl hover:bg-blue-700 transition-colors shadow-sm font-bold disabled:opacity-50 disabled:cursor-not-allowed flex items-center gap-2">
                                    {isGenerating ? '전송 중...' : '제출 및 피드백 받기'}
                                </button>
                            </div>
                        </div>
                    </div>
                </div>
            );
        };

        const WritingRoleplayModule = ({studentName}) => {
             const [activeSubTab, setActiveSubTab] = useState('layered');

             return (
                <div className="flex flex-col h-full bg-gray-50 rounded-xl">
                    <div className="flex border-b border-gray-200 bg-white rounded-t-xl px-4 pt-4 gap-2">
                        <button onClick={() => setActiveSubTab('layered')} className={`px-6 py-3 font-medium text-sm rounded-t-lg transition-colors ${activeSubTab === 'layered' ? 'bg-gray-50 border-t border-x border-gray-200 text-blue-700 shadow-[0_2px_0_0_#f9fafb]' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-50 border-transparent'}`}>
                            4. HSKTC Layered Writing
                        </button>
                        <button onClick={() => setActiveSubTab('roleplay')} className={`px-6 py-3 font-medium text-sm rounded-t-lg transition-colors ${activeSubTab === 'roleplay' ? 'bg-gray-50 border-t border-x border-gray-200 text-indigo-700 shadow-[0_2px_0_0_#f9fafb]' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-50 border-transparent'}`}>
                            5. 🚀 Logic & Role-play (대립 토론)
                        </button>
                    </div>

                    <div className="flex-1 overflow-hidden flex flex-col p-4 md:p-6">
                        {activeSubTab === 'layered' && <LayeredWritingChat studentName={studentName} />}
                        {activeSubTab === 'roleplay' && <RoleplayChat studentName={studentName} />}
                    </div>
                </div>
             )
        };

        const RoleplayChat = ({studentName}) => {
            const [messages, setMessages] = useState([
                { role: 'system', text: '재택근무의 효율성에 대해 토론을 시작합니다. 저는 재택근무를 반대하는 보수적인 경영진 역할을 맡았습니다. 첫 번째 질문입니다.' },
                { role: 'ai', text: '학생분, 재택근무가 소통의 부재를 야기하고 부서 간의 협업 효율을 떨어뜨린다고 생각하지 않으십니까? 대면 미팅 없이 어떻게 창의적인 아이디어가 나오겠습니까?' }
            ]);
            const [input, setInput] = useState('');

            const handleSend = () => {
                if(!input.trim()) return;
                
                const newMessages = [...messages, { role: 'user', text: input }];
                setMessages(newMessages);
                setInput('');

                setTimeout(() => {
                    setMessages(prev => [...prev, { 
                        role: 'ai', 
                        text: `논리적 전개는 좋습니다만, 어휘 선택이 평이하군요. 비즈니스 상황에 맞게 단순히 '서로 돕는다'가 아니라 **'상호 보완(取长补短)'**이나 **'효율 증대(事半功倍)'** 같은 고급 사자성어를 활용하여 다시 한번 설득해 보시겠어요?` 
                    }]);
                }, 1500);
            };

            return (
                <div className="flex flex-col h-full bg-white rounded-xl border shadow-sm overflow-hidden">
                    <div className="bg-indigo-600 text-white p-4 flex items-center gap-3">
                        <div className="w-10 h-10 bg-white/20 rounded-full flex items-center justify-center backdrop-blur-sm">
                            <Icon name="MessageSquare" size={20} />
                        </div>
                        <div>
                            <div className="font-bold text-sm">토론형 상황극 AI 튜터</div>
                            <div className="text-xs text-indigo-100 opacity-90">논리 방어 및 고급 어휘 체화 훈련</div>
                        </div>
                    </div>
                    
                    <div className="flex-1 overflow-y-auto p-4 space-y-4 bg-slate-50">
                        {messages.map((msg, idx) => (
                            <div key={idx} className={`flex ${msg.role === 'user' ? 'justify-end' : 'justify-start'}`}>
                                {msg.role === 'system' ? (
                                    <div className="w-full text-center my-2">
                                        <span className="bg-gray-200 text-gray-600 text-xs px-3 py-1 rounded-full">{msg.text}</span>
                                    </div>
                                ) : (
                                    <div className={`max-w-[75%] rounded-2xl px-5 py-3 text-sm shadow-sm ${
                                        msg.role === 'user' 
                                        ? 'bg-indigo-600 text-white rounded-tr-none' 
                                        : 'bg-white border text-gray-800 rounded-tl-none'
                                    }`}>
                                        {msg.role === 'user' && <div className="font-bold text-xs text-indigo-200 mb-1">{studentName}</div>}
                                        {msg.role === 'ai' && <div className="font-bold text-xs text-gray-400 mb-1">AI 경영진</div>}
                                        <div dangerouslySetInnerHTML={{ __html: msg.text.replace(/\*\*(.*?)\*\*/g, '<strong class="text-indigo-600 bg-indigo-50 px-1 rounded">$1</strong>') }} />
                                    </div>
                                )}
                            </div>
                        ))}
                    </div>

                    <div className="p-4 bg-white border-t">
                        <div className="flex items-center gap-2">
                            <textarea 
                                value={input} 
                                onChange={e => setInput(e.target.value)}
                                onKeyDown={e => {if(e.key==='Enter' && !e.shiftKey) { e.preventDefault(); handleSend();}}}
                                placeholder="'연결(Logic)' 구조를 활용하여 답변을 작성하세요... (Shift+Enter로 줄바꿈)" 
                                className="flex-1 border border-gray-300 rounded-xl px-4 py-3 text-sm focus:ring-2 focus:ring-indigo-500 outline-none resize-none h-14"
                            />
                            <button onClick={handleSend} className="bg-indigo-600 text-white p-3 h-14 rounded-xl hover:bg-indigo-700 transition-colors shadow-sm px-6 font-bold">
                                전송
                            </button>
                        </div>
                    </div>
                </div>
            )
        }

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>