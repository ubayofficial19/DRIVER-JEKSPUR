<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>JEKSPUR - Status Driver Ojek Kampus</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .status-dot {
            width: 12px;
            height: 12px;
            border-radius: 50%;
            display: inline-block;
            margin-right: 8px;
            flex-shrink: 0;
        }
        .status-ready { background-color: #22c55e; } /* green-500 */
        .status-not-ready { background-color: #ef4444; } /* red-500 */
        .toast {
            visibility: hidden;
            min-width: 250px;
            margin-left: -125px;
            background-color: #333;
            color: #fff;
            text-align: center;
            border-radius: 8px;
            padding: 16px;
            position: fixed;
            z-index: 1;
            left: 50%;
            bottom: 30px;
            opacity: 0;
            transition: opacity 0.5s, visibility 0.5s;
        }
        .toast.show {
            visibility: visible;
            opacity: 1;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800 antialiased">

    <div class="container mx-auto p-4 md:p-8 max-w-5xl">
        <!-- Header -->
        <header class="text-center mb-8">
            <h1 class="text-4xl font-bold bg-gradient-to-r from-green-500 to-green-700 text-transparent bg-clip-text">
                JEKSPUR
            </h1>
            <p class="text-gray-600 mt-2">Panel Status Driver Ojek Kampus</p>
        </header>

        <!-- Form Tambah Driver -->
        <div class="bg-white p-6 rounded-xl shadow-lg mb-8">
            <h2 class="text-xl font-semibold mb-4 text-gray-900">Tambah Driver Baru</h2>
            <form id="add-driver-form" class="flex flex-col sm:flex-row gap-4">
                <input type="text" id="driver-name" placeholder="Masukkan nama driver" class="flex-grow bg-gray-50 border border-gray-300 text-gray-900 placeholder-gray-400 rounded-lg px-4 py-3 focus:outline-none focus:ring-2 focus:ring-green-500" required>
                <button type="submit" class="bg-green-600 hover:bg-green-700 text-white font-bold py-3 px-6 rounded-lg transition duration-300 ease-in-out transform hover:scale-105">
                    Tambah Driver
                </button>
            </form>
        </div>

        <!-- Tampilan Status Driver -->
        <div id="loading-state" class="text-center py-10">
            <p class="text-gray-500">Memuat data driver...</p>
        </div>

        <div id="driver-lists" class="hidden grid grid-cols-1 md:grid-cols-2 gap-8">
            <!-- Kolom Driver Ready -->
            <div>
                <h2 class="text-2xl font-bold mb-4 flex items-center">
                    <span class="status-dot status-ready"></span>
                    Ready
                </h2>
                <div id="ready-drivers-list" class="space-y-4">
                    <!-- Kartu driver akan ditambahkan di sini oleh JavaScript -->
                     <p id="no-ready-drivers" class="text-gray-500 italic">Belum ada driver yang ready.</p>
                </div>
            </div>

            <!-- Kolom Driver Tidak Ready -->
            <div>
                <h2 class="text-2xl font-bold mb-4 flex items-center">
                    <span class="status-dot status-not-ready"></span>
                    Tidak Ready
                </h2>
                <div id="not-ready-drivers-list" class="space-y-4">
                    <!-- Kartu driver akan ditambahkan di sini oleh JavaScript -->
                     <p id="no-not-ready-drivers" class="text-gray-500 italic">Semua driver sedang ready.</p>
                </div>
            </div>
        </div>

    </div>
    
    <!-- Toast Notification -->
    <div id="toast" class="toast">Pesan notifikasi</div>

    <script type="module">
        // --- Konfigurasi dan Inisialisasi Firebase ---
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, updateDoc, deleteDoc, serverTimestamp, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";

        // Menggunakan variabel global yang disediakan oleh environment
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'jekspur-default-app';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        
        // Aktifkan log untuk debugging jika diperlukan
        // setLogLevel('debug');

        const DRIVERS_COLLECTION_PATH = `/artifacts/${appId}/public/data/drivers`;
        const driversCollection = collection(db, DRIVERS_COLLECTION_PATH);

        // --- Variabel Global & Elemen DOM ---
        const addDriverForm = document.getElementById('add-driver-form');
        const driverNameInput = document.getElementById('driver-name');
        const readyDriversList = document.getElementById('ready-drivers-list');
        const notReadyDriversList = document.getElementById('not-ready-drivers-list');
        const noReadyDrivers = document.getElementById('no-ready-drivers');
        const noNotReadyDrivers = document.getElementById('no-not-ready-drivers');
        const loadingState = document.getElementById('loading-state');
        const driverLists = document.getElementById('driver-lists');

        // --- Fungsi Notifikasi Toast ---
        function showToast(message) {
            const toast = document.getElementById('toast');
            toast.textContent = message;
            toast.classList.add('show');
            setTimeout(() => {
                toast.classList.remove('show');
            }, 3000);
        }

        // --- Fungsi Render ---
        const renderDrivers = (drivers) => {
            readyDriversList.innerHTML = '';
            notReadyDriversList.innerHTML = '';
            
            const readyDrivers = drivers.filter(d => d.status === 'ready');
            const notReadyDrivers = dri Avers.filter(d => d.status === 'tidak_ready');

            if (readyDrivers.length === 0) {
                 readyDriversList.innerHTML = `<p class="text-gray-500 italic">Belum ada driver yang ready.</p>`;
            } else {
                readyDrivers.forEach(driver => {
                    const driverCard = createDriverCard(driver);
                    readyDriversList.appendChild(driverCard);
                });
            }

            if (notReadyDrivers.length === 0) {
                 notReadyDriversList.innerHTML = `<p class="text-gray-500 italic">Semua driver sedang ready.</p>`;
            } else {
                 notReadyDrivers.forEach(driver => {
                    const driverCard = createDriverCard(driver);
                    notReadyDriversList.appendChild(driverCard);
                });
            }
        };

        const createDriverCard = (driver) => {
            const div = document.createElement('div');
            div.className = 'bg-white p-4 rounded-lg shadow-md flex items-center justify-between transition-transform transform hover:scale-102 border border-gray-200';
            div.setAttribute('data-id', driver.id);

            const isReady = driver.status === 'ready';
            const statusDotClass = isReady ? 'status-ready' : 'status-not-ready';
            const buttonText = isReady ? 'Set Tidak Ready' : 'Set Ready';
            const buttonClass = isReady 
                ? 'bg-yellow-600 hover:bg-yellow-700' 
                : 'bg-green-600 hover:bg-green-700';

            div.innerHTML = `
                <div class="flex items-center">
                    <span class="status-dot ${statusDotClass}"></span>
                    <p class="font-semibold text-lg text-gray-900">${driver.name}</p>
                </div>
                <div class="flex gap-2">
                    <button class="toggle-status-btn text-xs font-bold py-2 px-3 rounded-md transition duration-300 ${buttonClass}" data-status="${driver.status}">${buttonText}</button>
                    <button class="delete-driver-btn bg-red-600 hover:bg-red-700 text-xs font-bold py-2 px-3 rounded-md transition duration-300">Hapus</button>
                </div>
            `;
            return div;
        };

        // --- Logika Firebase ---
        const handleAuth = async () => {
            onAuthStateChanged(auth, (user) => {
                if (user) {
                    console.log("User is signed in:", user.uid);
                    // Setelah user terautentikasi, mulai listen ke data Firestore
                    listenForDrivers();
                    loadingState.style.display = 'none';
                    driverLists.classList.remove('hidden');
                } else {
                    console.log("User is signed out.");
                }
            });

            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Authentication error:", error);
                loadingState.textContent = "Gagal melakukan autentikasi. Silakan refresh halaman.";
            }
        };

        const listenForDrivers = () => {
            onSnapshot(driversCollection, (snapshot) => {
                const drivers = [];
                snapshot.docs.forEach(doc => {
                    drivers.push({ ...doc.data(), id: doc.id });
                });
                // Urutkan berdasarkan nama
                drivers.sort((a, b) => a.name.localeCompare(b.name));
                renderDrivers(drivers);
            }, (error) => {
                console.error("Error fetching drivers:", error);
                showToast("Gagal memuat data driver.");
            });
        };

        const addDriver = async (name) => {
            if (!name.trim()) {
                showToast("Nama driver tidak boleh kosong.");
                return;
            }
            try {
                await addDoc(driversCollection, {
                    name: name.trim(),
                    status: 'tidak_ready', // Status default saat driver baru ditambahkan
                    createdAt: serverTimestamp()
                });
                showToast(`Driver "${name}" berhasil ditambahkan.`);
                driverNameInput.value = '';
            } catch (error) {
                console.error("Error adding driver: ", error);
                showToast("Gagal menambahkan driver.");
            }
        };
        
        const toggleDriverStatus = async (id, currentStatus) => {
            const newStatus = currentStatus === 'ready' ? 'tidak_ready' : 'ready';
            const driverRef = doc(db, DRIVERS_COLLECTION_PATH, id);
            try {
                await updateDoc(driverRef, { status: newStatus });
                showToast("Status driver berhasil diubah.");
            } catch (error) {
                console.error("Error updating status: ", error);
                showToast("Gagal mengubah status.");
            }
        };

        const deleteDriver = async (id, name) => {
             // Untuk keamanan, kita tidak menggunakan confirm() karena bisa diblokir.
             // Di aplikasi nyata, ini akan diganti dengan modal custom.
             // Untuk contoh ini, kita langsung hapus saja.
            const driverRef = doc(db, DRIVERS_COLLECTION_PATH, id);
            try {
                await deleteDoc(driverRef);
                showToast(`Driver "${name}" berhasil dihapus.`);
            } catch (error) {
                console.error("Error deleting driver: ", error);
                showToast("Gagal menghapus driver.");
            }
        };

        // --- Event Listeners ---
        addDriverForm.addEventListener('submit', (e) => {
            e.preventDefault();
            addDriver(driverNameInput.value);
        });

        // Event delegation untuk tombol di dalam list
        document.body.addEventListener('click', (e) => {
            const card = e.target.closest('[data-id]');
            if (!card) return;

            const driverId = card.dataset.id;
            const driverName = card.querySelector('p').textContent;

            if (e.target.classList.contains('toggle-status-btn')) {
                const currentStatus = e.target.dataset.status;
                toggleDriverStatus(driverId, currentStatus);
            }

            if (e.target.classList.contains('delete-driver-btn')) {
                // Idealnya ada konfirmasi modal di sini.
                deleteDriver(driverId, driverName);
            }
        });

        // --- Mulai Aplikasi ---
        handleAuth();

    </script>
</body>
</html>
