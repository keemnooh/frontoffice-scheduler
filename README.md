<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>프런트오피스 스케쥴러 - 실시간 일정 공유</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- FontAwesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts (Noto Sans KR & Inter) -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=Noto+Sans+KR:wght@300;400;500;700&display=swap" rel="stylesheet">
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        team1: {
                            light: '#e0e7ff',
                            DEFAULT: '#4f46e5',
                            dark: '#3730a3',
                            bg: '#eef2ff',
                            border: '#c7d2fe'
                        },
                        team2: {
                            light: '#ccfbf1',
                            DEFAULT: '#0d9488',
                            dark: '#115e59',
                            bg: '#f0fdf4',
                            border: '#99f6e4'
                        }
                    },
                    fontFamily: {
                        sans: ['Noto Sans KR', 'Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>
    <style>
        body { font-family: 'Noto Sans KR', sans-serif; background-color: #f8fafc; }
        .custom-scrollbar::-webkit-scrollbar { width: 4px; height: 4px; }
        .custom-scrollbar::-webkit-scrollbar-track { background: #f1f5f9; }
        .custom-scrollbar::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 4px; }
        .custom-scrollbar::-webkit-scrollbar-thumb:hover { background: #94a3b8; }
        /* Pulse for online live indicator */
        @keyframes pulse-ring {
            0% { transform: scale(0.95); opacity: 0.8; }
            50% { transform: scale(1.1); opacity: 0.4; }
            100% { transform: scale(0.95); opacity: 0.8; }
        }
        .live-dot { animation: pulse-ring 2s infinite ease-in-out; }
    </style>
</head>
<body class="bg-slate-50 text-slate-800 min-h-screen flex flex-col selection:bg-indigo-100 selection:text-indigo-800">

    <!-- Navigation Header -->
    <header class="bg-white border-b border-slate-200 sticky top-0 z-30 shadow-sm">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between h-16 items-center">
                <!-- Title & Brand -->
                <div class="flex items-center space-x-3">
                    <div class="w-10 h-10 rounded-xl bg-gradient-to-tr from-indigo-600 via-purple-600 to-pink-500 flex items-center justify-center text-white font-bold shadow-md shrink-0">
                        <i class="fa-solid fa-newspaper text-lg"></i>
                    </div>
                    <div>
                        <div class="flex items-center gap-2">
                            <h1 class="text-base sm:text-xl font-bold text-slate-900 leading-tight">프런트오피스 스케쥴러</h1>
                            <span id="syncBadge" class="hidden sm:inline-flex items-center gap-1.5 px-2 py-0.5 rounded-full text-[11px] font-semibold bg-emerald-50 text-emerald-700 border border-emerald-200">
                                <span class="w-2 h-2 rounded-full bg-emerald-500 live-dot"></span> 실시간 연동 중
                            </span>
                        </div>
                        <p class="text-xs text-slate-500 hidden sm:block">인스타그램 매거진 기획 & 릴스 콘텐츠 교차 일정 관리</p>
                    </div>
                </div>

                <!-- Team Info Badges & Action Buttons -->
                <div class="flex items-center space-x-2 sm:space-x-3">
                    <div class="hidden lg:flex items-center space-x-2 bg-indigo-50 border border-indigo-100 px-3 py-1.5 rounded-lg">
                        <span class="w-2.5 h-2.5 rounded-full bg-indigo-600"></span>
                        <span class="text-xs font-semibold text-indigo-900">1팀: 박지훈</span>
                    </div>
                    <div class="hidden lg:flex items-center space-x-2 bg-teal-50 border border-teal-100 px-3 py-1.5 rounded-lg">
                        <span class="w-2.5 h-2.5 rounded-full bg-teal-600"></span>
                        <span class="text-xs font-semibold text-teal-900">2팀: 김정훈</span>
                    </div>

                    <!-- Share Link Button -->
                    <button onclick="shareLink()" class="bg-slate-100 hover:bg-slate-200 text-slate-700 text-xs sm:text-sm font-semibold px-3 py-2 rounded-xl transition flex items-center gap-1.5 border border-slate-200" title="팀원 공유 링크 복사">
                        <i class="fa-solid fa-share-nodes text-indigo-600"></i>
                        <span class="hidden xs:inline">공유하기</span>
                    </button>

                    <!-- Add Event Button -->
                    <button onclick="openModal()" class="bg-indigo-600 hover:bg-indigo-700 active:scale-95 text-white text-xs sm:text-sm font-semibold px-3.5 py-2 rounded-xl shadow-sm transition flex items-center gap-1.5">
                        <i class="fa-solid fa-plus text-xs"></i>
                        <span>새 일정</span>
                    </button>
                </div>
            </div>
        </div>
    </header>

    <!-- Global Toast Notification Container -->
    <div id="toastContainer" class="fixed bottom-5 right-5 z-50 flex flex-col space-y-2 pointer-events-none"></div>

    <!-- Main Content Container -->
    <main class="flex-1 max-w-7xl w-full mx-auto px-3 sm:px-6 lg:px-8 py-4 sm:py-6">

        <!-- Banner for Conflict Warning Summary -->
        <div id="conflictAlertBanner" class="hidden mb-4 sm:mb-6 bg-amber-50 border-l-4 border-amber-500 p-3.5 sm:p-4 rounded-r-xl shadow-sm flex items-start space-x-3">
            <i class="fa-solid fa-triangle-exclamation text-amber-500 text-lg sm:text-xl mt-0.5 shrink-0"></i>
            <div class="flex-1">
                <h3 class="text-xs sm:text-sm font-bold text-amber-800">동일 날짜 발행 일정이 존재합니다!</h3>
                <p class="text-[11px] sm:text-xs text-amber-700 mt-0.5" id="conflictAlertText">1팀과 2팀의 기획/릴스 콘텐츠 발행 일정이 같은 날에 배치되어 있습니다.</p>
            </div>
        </div>

        <!-- Filter and View Switching Controls -->
        <div class="bg-white p-3 sm:p-4 rounded-2xl border border-slate-200 shadow-sm mb-4 sm:mb-6 flex flex-col lg:flex-row lg:items-center justify-between gap-3 sm:gap-4">
            
            <!-- Controls Group Left -->
            <div class="flex flex-wrap items-center gap-2 sm:gap-3">
                <!-- Date Selector Navigation -->
                <div class="flex items-center space-x-1 bg-slate-100 p-1 rounded-xl">
                    <button onclick="changeMonth(-1)" class="p-1.5 hover:bg-white rounded-lg text-slate-600 transition"><i class="fa-solid fa-chevron-left text-xs"></i></button>
                    <span id="currentMonthLabel" class="text-xs sm:text-sm font-bold text-slate-800 px-2 sm:px-3 min-w-[90px] text-center">2026년 9월</span>
                    <button onclick="changeMonth(1)" class="p-1.5 hover:bg-white rounded-lg text-slate-600 transition"><i class="fa-solid fa-chevron-right text-xs"></i></button>
                </div>

                <!-- Team Filter -->
                <div class="flex items-center space-x-1 border-l border-slate-200 pl-2 sm:pl-3">
                    <span class="text-xs font-medium text-slate-500 mr-1 hidden xs:inline">팀:</span>
                    <button onclick="setTeamFilter('ALL')" id="filterTeamAll" class="px-2.5 py-1.5 text-xs font-semibold rounded-lg bg-slate-900 text-white transition">전체</button>
                    <button onclick="setTeamFilter('TEAM1')" id="filterTeam1" class="px-2.5 py-1.5 text-xs font-semibold rounded-lg bg-slate-100 text-slate-600 hover:bg-indigo-50 hover:text-indigo-600 transition">1팀</button>
                    <button onclick="setTeamFilter('TEAM2')" id="filterTeam2" class="px-2.5 py-1.5 text-xs font-semibold rounded-lg bg-slate-100 text-slate-600 hover:bg-teal-50 hover:text-teal-600 transition">2팀</button>
                </div>

                <!-- Format Filter -->
                <div class="flex items-center space-x-1 border-l border-slate-200 pl-2 sm:pl-3">
                    <span class="text-xs font-medium text-slate-500 mr-1 hidden xs:inline">포맷:</span>
                    <select id="formatFilter" onchange="applyFilters()" class="text-xs font-semibold border border-slate-300 rounded-lg px-2 py-1.5 bg-white text-slate-700 focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="ALL">전체 포맷</option>
                        <option value="기획 콘텐츠">기획 콘텐츠</option>
                        <option value="릴스 콘텐츠">릴스 콘텐츠</option>
                    </select>
                </div>
            </div>

            <!-- Controls Group Right (View Mode Toggle & Stats) -->
            <div class="flex items-center justify-between lg:justify-end space-x-3 border-t lg:border-t-0 pt-2.5 lg:pt-0 border-slate-100">
                <div class="text-xs text-slate-500 font-medium">
                    총 <span id="totalCount" class="font-bold text-slate-900">0</span>개 콘텐츠
                </div>
                
                <div class="flex bg-slate-100 p-1 rounded-xl">
                    <button onclick="switchView('calendar')" id="btnViewCalendar" class="flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold bg-white text-slate-800 shadow-sm transition">
                        <i class="fa-regular fa-calendar-days"></i> 캘린더
                    </button>
                    <button onclick="switchView('list')" id="btnViewList" class="flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold text-slate-600 hover:text-slate-900 transition">
                        <i class="fa-solid fa-list-ul"></i> 목록
                    </button>
                </div>
            </div>
        </div>

        <!-- CALENDAR VIEW -->
        <div id="calendarView" class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
            <!-- Calendar Days Header -->
            <div class="grid grid-cols-7 border-b border-slate-200 bg-slate-50 text-center text-xs font-bold text-slate-600 py-2.5">
                <div class="text-rose-500">일</div>
                <div>월</div>
                <div>화</div>
                <div>수</div>
                <div>목</div>
                <div>금</div>
                <div class="text-indigo-500">토</div>
            </div>
            <!-- Dynamic Calendar Days Grid -->
            <div id="calendarGrid" class="grid grid-cols-7 auto-rows-fr divide-x divide-y divide-slate-100 min-h-[500px] sm:min-h-[650px]">
                <!-- Rendered dynamically by JS -->
            </div>
        </div>

        <!-- LIST / TIMELINE VIEW -->
        <div id="listView" class="hidden space-y-4">
            <div class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse min-w-[640px]">
                        <thead>
                            <tr class="bg-slate-50 text-slate-600 text-xs font-bold border-b border-slate-200">
                                <th class="p-3.5">발행 일시</th>
                                <th class="p-3.5">담당 팀</th>
                                <th class="p-3.5">포맷 유형</th>
                                <th class="p-3.5">콘텐츠 제목 / 주제</th>
                                <th class="p-3.5">진행 상태</th>
                                <th class="p-3.5">비고 및 링크</th>
                                <th class="p-3.5 text-right">관리</th>
                            </tr>
                        </thead>
                        <tbody id="listTableBody" class="divide-y divide-slate-100 text-xs sm:text-sm">
                            <!-- Rendered dynamically by JS -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- Modal for Adding/Editing Content Schedule -->
    <div id="scheduleModal" class="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-lg overflow-hidden border border-slate-100 transform transition-all">
            <!-- Modal Header -->
            <div class="bg-slate-900 text-white px-6 py-4 flex justify-between items-center">
                <h3 id="modalTitle" class="text-base sm:text-lg font-bold">새 콘텐츠 일정 추가</h3>
                <button onclick="closeModal()" class="text-slate-400 hover:text-white transition">
                    <i class="fa-solid fa-xmark text-lg"></i>
                </button>
            </div>

            <!-- Modal Form Body -->
            <form id="scheduleForm" onsubmit="saveSchedule(event)" class="p-5 sm:p-6 space-y-4">
                <input type="hidden" id="scheduleId">

                <!-- Title Input -->
                <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">콘텐츠 제목 / 주제 <span class="text-rose-500">*</span></label>
                    <input type="text" id="formTitle" required placeholder="예: [특집] 9월 가을 신상 기획 룩북" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                </div>

                <!-- Team & Format Grid -->
                <div class="grid grid-cols-2 gap-3 sm:gap-4">
                    <!-- Team Selector -->
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">담당 팀 <span class="text-rose-500">*</span></label>
                        <select id="formTeam" required class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="TEAM1">1팀 (박지훈 팀장)</option>
                            <option value="TEAM2">2팀 (김정훈 팀장)</option>
                        </select>
                    </div>
                    <!-- Format Type Selector -->
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">포맷 유형 <span class="text-rose-500">*</span></label>
                        <select id="formFormat" required class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500">
                            <option value="기획 콘텐츠">기획 콘텐츠</option>
                            <option value="릴스 콘텐츠">릴스 콘텐츠</option>
                        </select>
                    </div>
                </div>

                <!-- Date & Time Grid -->
                <div class="grid grid-cols-2 gap-3 sm:gap-4">
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">발행 일자 <span class="text-rose-500">*</span></label>
                        <input type="date" id="formDate" required class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">발행 시간 <span class="text-rose-500">*</span></label>
                        <input type="time" id="formTime" required class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500">
                    </div>
                </div>

                <!-- Status Selector -->
                <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">진행 상태</label>
                    <select id="formStatus" class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm bg-white focus:outline-none focus:ring-2 focus:ring-indigo-500">
                        <option value="기획 중">기획 중</option>
                        <option value="제작 중">제작 중</option>
                        <option value="승인 완료">승인 완료</option>
                        <option value="발행 완료">발행 완료</option>
                    </select>
                </div>

                <!-- Notes Input -->
                <div>
                    <label class="block text-xs font-bold text-slate-700 mb-1">비고 및 레퍼런스 링크</label>
                    <textarea id="formNotes" rows="2" placeholder="참고 링크나 팀원 공유사항을 입력하세요." class="w-full px-3 py-2 border border-slate-300 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"></textarea>
                </div>

                <!-- Action Buttons -->
                <div class="flex justify-between items-center pt-3 border-t border-slate-100">
                    <button type="button" id="btnDeleteModal" onclick="confirmDeleteFromModal()" class="hidden text-xs font-semibold text-rose-600 hover:text-rose-700 hover:bg-rose-50 px-3 py-2 rounded-xl transition flex items-center gap-1.5">
                        <i class="fa-solid fa-trash-can"></i> 일정 삭제
                    </button>
                    <div class="flex space-x-2 ml-auto">
                        <button type="button" onclick="closeModal()" class="px-4 py-2 border border-slate-300 rounded-xl text-xs font-semibold text-slate-600 hover:bg-slate-50 transition">취소</button>
                        <button type="submit" id="btnSubmitForm" class="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 text-white rounded-xl text-xs font-semibold shadow-sm transition">저장하기</button>
                    </div>
                </div>
            </form>
        </div>
    </div>

    <!-- Delete Confirmation Modal -->
    <div id="deleteConfirmModal" class="fixed inset-0 bg-slate-900/40 backdrop-blur-sm z-50 flex items-center justify-center hidden p-4">
        <div class="bg-white rounded-2xl shadow-xl w-full max-w-sm p-6 text-center border border-slate-100 transform transition-all">
            <div class="w-12 h-12 bg-rose-100 text-rose-600 rounded-full flex items-center justify-center mx-auto mb-4 text-xl">
                <i class="fa-solid fa-trash-can"></i>
            </div>
            <h4 class="text-base sm:text-lg font-bold text-slate-900 mb-1">일정을 삭제하시겠습니까?</h4>
            <p class="text-xs text-slate-500 mb-6">실시간으로 연동되어 모든 팀원의 화면에서 삭제됩니다.</p>
            <div class="flex justify-center space-x-3">
                <button onclick="closeDeleteModal()" class="w-1/2 py-2.5 border border-slate-300 rounded-xl text-xs font-semibold text-slate-700 hover:bg-slate-50 transition">취소</button>
                <button id="btnExecuteDelete" onclick="executeDelete()" class="w-1/2 py-2.5 bg-rose-600 hover:bg-rose-700 text-white rounded-xl text-xs font-semibold shadow-sm transition">삭제하기</button>
            </div>
        </div>
    </div>

    <!-- Firebase Imports and Logic -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, doc, setDoc, addDoc, updateDoc, deleteDoc, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase state
        let db, auth;
        let unsubscribeSchedules = null;
        
        // Configuration fallback
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'frontoffice-scheduler';
        const firebaseConfig = typeof __firebase_config !== 'undefined' 
            ? JSON.parse(__firebase_config) 
            : { apiKey: "demo-api-key", authDomain: "demo.firebaseapp.com", projectId: "demo-project" };

        // Default initial data if database is empty
        const defaultSchedules = [
            {
                title: '9월 FW 트렌드 컬러 가이드',
                team: 'TEAM1',
                format: '기획 콘텐츠',
                date: '2026-09-18',
                time: '18:00',
                status: '제작 중',
                notes: '이미지 10장 가로 카드뉴스 형태'
            },
            {
                title: '성수동 핫플 카페 TOP 5',
                team: 'TEAM2',
                format: '릴스 콘텐츠',
                date: '2026-09-18',
                time: '20:00',
                status: '승인 완료',
                notes: '숏폼 30초 음원 저작권 확인 완료'
            },
            {
                title: '에디터 픽 가을 데일리룩 인터뷰',
                team: 'TEAM1',
                format: '릴스 콘텐츠',
                date: '2026-09-22',
                time: '19:00',
                status: '기획 중',
                notes: '현장 인터뷰 영상 편집 예정'
            },
            {
                title: '인디 브랜드 디자이너 스토리',
                team: 'TEAM2',
                format: '기획 콘텐츠',
                date: '2026-09-22',
                time: '18:00',
                status: '제작 중',
                notes: '서면 인터뷰 자료 수집 완료'
            }
        ];

        // Global Application State
        window.schedules = [];
        window.currentYear = 2026;
        window.currentMonth = 8; // September (0-indexed)
        window.currentTeamFilter = 'ALL';
        window.currentFormatFilter = 'ALL';
        window.currentView = 'calendar';
        window.pendingDeleteId = null;

        // Initialize App & Firebase Auth
        window.onload = async function() {
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Authenticate (Custom Token or Anonymous)
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }

                // Attach real-time Firestore listener
                setupRealtimeListener();
            } catch (err) {
                console.warn("Firebase initialized in standalone/local mode:", err);
                // Fallback local memory mode if config is invalid
                window.schedules = defaultSchedules.map((item, idx) => ({ ...item, id: 'demo-' + idx }));
                renderView();
            }

            updateMonthLabel();
        };

        // Strict Path Rule: /artifacts/{appId}/public/data/schedules
        function getSchedulesCollection() {
            return collection(db, 'artifacts', appId, 'public', 'data', 'schedules');
        }

        function setupRealtimeListener() {
            if (!auth.currentUser) return;
            const colRef = getSchedulesCollection();

            // Realtime Sync Listener (Rule 2: Fetch all & filter in JS memory)
            unsubscribeSchedules = onSnapshot(colRef, (snapshot) => {
                const loaded = [];
                snapshot.forEach(docSnap => {
                    loaded.push({ id: docSnap.id, ...docSnap.data() });
                });

                // Seed default data on initial setup if collection is empty
                if (loaded.length === 0 && snapshot.metadata.hasPendingWrites === false) {
                    seedDefaultData();
                    return;
                }

                window.schedules = loaded;
                document.getElementById('syncBadge').classList.remove('hidden');
                renderView();
            }, (error) => {
                console.error("Firestore listener error:", error);
                showToast("실시간 데이터를 불러오는 도중 오류가 발생했습니다.", "danger");
            });
        }

        async function seedDefaultData() {
            try {
                const colRef = getSchedulesCollection();
                for (const item of defaultSchedules) {
                    await addDoc(colRef, item);
                }
            } catch(e) {
                console.error("Failed to seed initial schedules:", e);
            }
        }

        // Global UI Functions
        window.showToast = function(message, type = 'success') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = `pointer-events-auto flex items-center gap-2 px-4 py-3 rounded-xl shadow-xl text-xs font-semibold text-white transition-all transform translate-y-2 opacity-0 ${type === 'danger' ? 'bg-rose-600' : 'bg-slate-900'}`;
            toast.innerHTML = `
                <i class="fa-solid ${type === 'danger' ? 'fa-triangle-exclamation' : 'fa-circle-check'} text-sm"></i>
                <span>${message}</span>
            `;
            container.appendChild(toast);

            setTimeout(() => toast.classList.remove('translate-y-2', 'opacity-0'), 10);
            setTimeout(() => {
                toast.classList.add('opacity-0');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        };

        window.shareLink = function() {
            const currentUrl = window.location.href;
            if (navigator.clipboard) {
                navigator.clipboard.writeText(currentUrl).then(() => {
                    showToast("스케쥴러 공유 링크가 복사되었습니다! 팀원들에게 전달하세요.");
                }).catch(() => fallbackCopy(currentUrl));
            } else {
                fallbackCopy(currentUrl);
            }
        };

        function fallbackCopy(text) {
            const input = document.createElement('input');
            input.value = text;
            document.body.appendChild(input);
            input.select();
            document.execCommand('copy');
            document.body.removeChild(input);
            showToast("스케쥴러 공유 링크가 복사되었습니다!");
        }

        window.switchView = function(view) {
            window.currentView = view;
            const btnCal = document.getElementById('btnViewCalendar');
            const btnList = document.getElementById('btnViewList');
            const viewCal = document.getElementById('calendarView');
            const viewList = document.getElementById('listView');

            if (view === 'calendar') {
                viewCal.classList.remove('hidden');
                viewList.classList.add('hidden');
                btnCal.className = "flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold bg-white text-slate-800 shadow-sm transition";
                btnList.className = "flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold text-slate-600 hover:text-slate-900 transition";
            } else {
                viewCal.classList.add('hidden');
                viewList.classList.remove('hidden');
                btnList.className = "flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold bg-white text-slate-800 shadow-sm transition";
                btnCal.className = "flex items-center gap-1.5 px-3 py-1 rounded-lg text-xs font-semibold text-slate-600 hover:text-slate-900 transition";
            }
            renderView();
        };

        window.setTeamFilter = function(team) {
            window.currentTeamFilter = team;
            ['ALL', 'TEAM1', 'TEAM2'].forEach(t => {
                const btn = document.getElementById(`filterTeam${t === 'ALL' ? 'All' : t === 'TEAM1' ? '1' : '2'}`);
                if (t === team) {
                    btn.className = "px-2.5 py-1.5 text-xs font-semibold rounded-lg bg-slate-900 text-white transition";
                } else {
                    btn.className = "px-2.5 py-1.5 text-xs font-semibold rounded-lg bg-slate-100 text-slate-600 hover:bg-slate-200 transition";
                }
            });
            renderView();
        };

        window.applyFilters = function() {
            window.currentFormatFilter = document.getElementById('formatFilter').value;
            renderView();
        };

        window.changeMonth = function(delta) {
            window.currentMonth += delta;
            if (window.currentMonth < 0) {
                window.currentMonth = 11;
                window.currentYear--;
            } else if (window.currentMonth > 11) {
                window.currentMonth = 0;
                window.currentYear++;
            }
            updateMonthLabel();
            renderView();
        };

        function updateMonthLabel() {
            document.getElementById('currentMonthLabel').textContent = `${window.currentYear}년 ${window.currentMonth + 1}월`;
        }

        function getFilteredSchedules() {
            return (window.schedules || []).filter(item => {
                if (!item.date) return false;
                const itemDate = new Date(item.date);
                const matchesMonth = itemDate.getFullYear() === window.currentYear && itemDate.getMonth() === window.currentMonth;
                const matchesTeam = window.currentTeamFilter === 'ALL' || item.team === window.currentTeamFilter;
                const matchesFormat = window.currentFormatFilter === 'ALL' || item.format === window.currentFormatFilter;
                return matchesMonth && matchesTeam && matchesFormat;
            });
        }

        window.renderView = function() {
            const filtered = getFilteredSchedules();
            document.getElementById('totalCount').textContent = filtered.length;

            checkScheduleConflicts();

            if (window.currentView === 'calendar') {
                renderCalendar(filtered);
            } else {
                renderList(filtered);
            }
        };

        function checkScheduleConflicts() {
            const dateMap = {};
            (window.schedules || []).forEach(s => {
                if (!s.date) return;
                if (!dateMap[s.date]) dateMap[s.date] = new Set();
                dateMap[s.date].add(s.team);
            });

            const conflictDates = Object.keys(dateMap).filter(d => dateMap[d].size > 1);
            const banner = document.getElementById('conflictAlertBanner');
            const alertText = document.getElementById('conflictAlertText');

            if (conflictDates.length > 0) {
                const formattedDates = conflictDates.map(d => {
                    const parts = d.split('-');
                    return `${parts[1]}월 ${parts[2]}일`;
                }).join(', ');
                alertText.innerHTML = `<strong>${formattedDates}</strong>에 1팀(박지훈)과 2팀(김정훈)의 콘텐츠 발행 일정이 동일 날짜에 배치되어 있습니다. 발행 시간을 점검해주세요!`;
                banner.classList.remove('hidden');
            } else {
                banner.classList.add('hidden');
            }
        }

        function renderCalendar(filteredItems) {
            const grid = document.getElementById('calendarGrid');
            grid.innerHTML = '';

            const firstDay = new Date(window.currentYear, window.currentMonth, 1).getDay();
            const daysInMonth = new Date(window.currentYear, window.currentMonth + 1, 0).getDate();

            for (let i = 0; i < firstDay; i++) {
                const emptyCell = document.createElement('div');
                emptyCell.className = 'bg-slate-50/40 p-1 sm:p-2 min-h-[70px] sm:min-h-[110px]';
                grid.appendChild(emptyCell);
            }

            for (let day = 1; day <= daysInMonth; day++) {
                const dateStr = `${window.currentYear}-${String(window.currentMonth + 1).padStart(2, '0')}-${String(day).padStart(2, '0')}`;
                const dayItems = filteredItems.filter(item => item.date === dateStr);

                const allTeamsOnDate = (window.schedules || []).filter(s => s.date === dateStr).map(s => s.team);
                const isConflict = allTeamsOnDate.includes('TEAM1') && allTeamsOnDate.includes('TEAM2');

                const cell = document.createElement('div');
                cell.className = `p-1 sm:p-2 min-h-[70px] sm:min-h-[110px] flex flex-col justify-between transition hover:bg-slate-50 ${isConflict ? 'bg-amber-50/50' : ''}`;

                let cellHeaderHtml = `
                    <div class="flex justify-between items-center mb-1">
                        <span class="text-[11px] sm:text-xs font-bold ${new Date(dateStr).getDay() === 0 ? 'text-rose-500' : new Date(dateStr).getDay() === 6 ? 'text-indigo-500' : 'text-slate-700'}">${day}</span>
                        ${isConflict ? '<span class="text-[9px] sm:text-[10px] font-bold bg-amber-100 text-amber-800 px-1 py-0.2 rounded flex items-center gap-0.5" title="일정 중복 경고"><i class="fa-solid fa-triangle-exclamation text-amber-600"></i><span class="hidden sm:inline">중복</span></span>' : ''}
                    </div>
                `;

                let itemsHtml = '<div class="space-y-1 overflow-y-auto max-h-[85px] custom-scrollbar flex-1">';
                dayItems.sort((a,b) => (a.time || '').localeCompare(b.time || '')).forEach(item => {
                    const isTeam1 = item.team === 'TEAM1';
                    const badgeBg = isTeam1 ? 'bg-indigo-50 text-indigo-700 border-indigo-200' : 'bg-teal-50 text-teal-700 border-teal-200';
                    const icon = item.format === '릴스 콘텐츠' ? 'fa-clapperboard' : 'fa-file-lines';

                    itemsHtml += `
                        <div onclick="editSchedule('${item.id}')" class="group relative p-1 sm:p-1.5 rounded-lg border text-[11px] cursor-pointer hover:shadow-sm transition ${badgeBg}">
                            <div class="flex items-center justify-between font-bold text-[10px] sm:text-[11px] leading-tight">
                                <span class="truncate">${isTeam1 ? '1팀' : '2팀'} • ${item.time || ''}</span>
                                <div class="flex items-center space-x-1">
                                    <i class="fa-solid ${icon} text-[9px] opacity-75"></i>
                                    <button onclick="event.stopPropagation(); promptDelete('${item.id}');" class="opacity-0 group-hover:opacity-100 text-rose-500 hover:text-rose-700 p-0.5 transition" title="삭제">
                                        <i class="fa-solid fa-trash-can text-[9px]"></i>
                                    </button>
                                </div>
                            </div>
                            <div class="font-medium text-[10px] sm:text-[11px] truncate mt-0.5 pr-2 hidden sm:block">${item.title}</div>
                        </div>
                    `;
                });
                itemsHtml += '</div>';

                cell.innerHTML = cellHeaderHtml + itemsHtml;
                grid.appendChild(cell);
            }
        }

        function renderList(filteredItems) {
            const tbody = document.getElementById('listTableBody');
            tbody.innerHTML = '';

            if (filteredItems.length === 0) {
                tbody.innerHTML = `<tr><td colspan="7" class="text-center py-8 text-slate-400">등록된 일정이 없습니다.</td></tr>`;
                return;
            }

            filteredItems.sort((a,b) => (a.date || '').localeCompare(b.date || '') || (a.time || '').localeCompare(b.time || '')).forEach(item => {
                const isTeam1 = item.team === 'TEAM1';
                const tr = document.createElement('tr');
                tr.className = "hover:bg-slate-50 transition border-b border-slate-100";

                let statusBadgeClass = 'bg-slate-100 text-slate-600';
                if (item.status === '승인 완료') statusBadgeClass = 'bg-blue-100 text-blue-700';
                if (item.status === '발행 완료') statusBadgeClass = 'bg-emerald-100 text-emerald-700';
                if (item.status === '제작 중') statusBadgeClass = 'bg-purple-100 text-purple-700';

                tr.innerHTML = `
                    <td class="p-3.5 font-semibold text-slate-700">${item.date} <span class="text-xs text-slate-400">(${item.time})</span></td>
                    <td class="p-3.5">
                        <span class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-lg text-xs font-semibold ${isTeam1 ? 'bg-indigo-50 text-indigo-700 border border-indigo-200' : 'bg-teal-50 text-teal-700 border border-teal-200'}">
                            <span class="w-1.5 h-1.5 rounded-full ${isTeam1 ? 'bg-indigo-600' : 'bg-teal-600'}"></span>
                            ${isTeam1 ? '1팀 (박지훈)' : '2팀 (김정훈)'}
                        </span>
                    </td>
                    <td class="p-3.5">
                        <span class="inline-flex items-center gap-1 text-xs font-medium text-slate-600">
                            <i class="fa-solid ${item.format === '릴스 콘텐츠' ? 'fa-clapperboard text-pink-500' : 'fa-file-lines text-blue-500'}"></i>
                            ${item.format}
                        </span>
                    </td>
                    <td class="p-3.5 font-bold text-slate-900">${item.title}</td>
                    <td class="p-3.5">
                        <span class="px-2 py-1 rounded-md text-xs font-semibold ${statusBadgeClass}">${item.status}</span>
                    </td>
                    <td class="p-3.5 text-xs text-slate-500 truncate max-w-[150px]">${item.notes || '-'}</td>
                    <td class="p-3.5 text-right space-x-1">
                        <button onclick="editSchedule('${item.id}')" class="p-1.5 text-slate-400 hover:text-indigo-600 transition" title="수정"><i class="fa-solid fa-pen-to-square"></i></button>
                        <button onclick="promptDelete('${item.id}')" class="p-1.5 text-slate-400 hover:text-rose-600 transition" title="삭제"><i class="fa-solid fa-trash-can"></i></button>
                    </td>
                `;
                tbody.appendChild(tr);
            });
        }

        window.openModal = function(isEdit = false) {
            const btnDeleteModal = document.getElementById('btnDeleteModal');
            if (!isEdit) {
                document.getElementById('scheduleForm').reset();
                document.getElementById('scheduleId').value = '';
                document.getElementById('modalTitle').textContent = '새 콘텐츠 일정 추가';
                document.getElementById('formDate').value = `${window.currentYear}-${String(window.currentMonth + 1).padStart(2, '0')}-15`;
                document.getElementById('formTime').value = '18:00';
                btnDeleteModal.classList.add('hidden');
            } else {
                btnDeleteModal.classList.remove('hidden');
            }
            document.getElementById('scheduleModal').classList.remove('hidden');
        };

        window.closeModal = function() {
            document.getElementById('scheduleModal').classList.add('hidden');
        };

        window.editSchedule = function(id) {
            const item = (window.schedules || []).find(s => s.id === id);
            if (!item) return;

            document.getElementById('scheduleId').value = item.id;
            document.getElementById('formTitle').value = item.title;
            document.getElementById('formTeam').value = item.team;
            document.getElementById('formFormat').value = item.format;
            document.getElementById('formDate').value = item.date;
            document.getElementById('formTime').value = item.time;
            document.getElementById('formStatus').value = item.status;
            document.getElementById('formNotes').value = item.notes || '';

            document.getElementById('modalTitle').textContent = '콘텐츠 일정 수정';
            openModal(true);
        };

        window.saveSchedule = async function(e) {
            e.preventDefault();
            const btnSubmit = document.getElementById('btnSubmitForm');
            btnSubmit.disabled = true;
            btnSubmit.innerHTML = `<i class="fa-solid fa-spinner fa-spin"></i> 저장 중...`;

            const id = document.getElementById('scheduleId').value;
            const payload = {
                title: document.getElementById('formTitle').value.trim(),
                team: document.getElementById('formTeam').value,
                format: document.getElementById('formFormat').value,
                date: document.getElementById('formDate').value,
                time: document.getElementById('formTime').value,
                status: document.getElementById('formStatus').value,
                notes: document.getElementById('formNotes').value.trim(),
                updatedAt: new Date().toISOString()
            };

            try {
                if (id) {
                    // Update document
                    const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'schedules', id);
                    await updateDoc(docRef, payload);
                    showToast('일정이 수정되었습니다. 모든 팀원 화면에 공유됩니다.');
                } else {
                    // Add new document
                    const colRef = getSchedulesCollection();
                    await addDoc(colRef, payload);
                    showToast('새 일정이 추가되어 팀 전체에 실시간 공유되었습니다!');
                }
                closeModal();
            } catch (err) {
                console.error("Save error:", err);
                showToast("저장하는 중 오류가 발생했습니다.", "danger");
            } finally {
                btnSubmit.disabled = false;
                btnSubmit.textContent = "저장하기";
            }
        };

        window.promptDelete = function(id) {
            window.pendingDeleteId = id;
            document.getElementById('deleteConfirmModal').classList.remove('hidden');
        };

        window.confirmDeleteFromModal = function() {
            const id = document.getElementById('scheduleId').value;
            if (id) {
                closeModal();
                promptDelete(id);
            }
        };

        window.closeDeleteModal = function() {
            window.pendingDeleteId = null;
            document.getElementById('deleteConfirmModal').classList.add('hidden');
        };

        window.executeDelete = async function() {
            if (!window.pendingDeleteId) return;

            const btnDel = document.getElementById('btnExecuteDelete');
            btnDel.disabled = true;

            try {
                const docRef = doc(db, 'artifacts', appId, 'public', 'data', 'schedules', window.pendingDeleteId);
                await deleteDoc(docRef);
                showToast('일정이 삭제되었습니다.', 'danger');
                closeDeleteModal();
            } catch (err) {
                console.error("Delete error:", err);
                showToast("삭제 중 오류가 발생했습니다.", "danger");
            } finally {
                btnDel.disabled = false;
            }
        };
    </script>
</body>
</html>
