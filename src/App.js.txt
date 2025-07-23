import React, { useState, useEffect } from 'react';
import { initializeApp } from 'firebase/app';
import { 
    getAuth, 
    onAuthStateChanged, 
    signInWithEmailAndPassword, 
    signOut,
    createUserWithEmailAndPassword // Hanya untuk setup awal
} from 'firebase/auth';
import { 
    getFirestore, 
    collection, 
    onSnapshot, 
    addDoc, 
    doc, 
    updateDoc, 
    deleteDoc,
    setDoc,
    serverTimestamp
} from 'firebase/firestore';

// --- KONFIGURASI FIREBASE ---
// Menjadi seperti ini:
const firebaseConfig = {
  apiKey: process.env.REACT_APP_API_KEY,
  authDomain: process.env.REACT_APP_AUTH_DOMAIN,
  projectId: process.env.REACT_APP_PROJECT_ID,
  storageBucket: process.env.REACT_APP_STORAGE_BUCKET,
  messagingSenderId: process.env.REACT_APP_MESSAGING_SENDER_ID,
  appId: process.env.REACT_APP_APP_ID,
  measurementId: process.env.REACT_APP_MEASUREMENT_ID
}

// --- Inisialisasi Firebase ---
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

// --- Komponen Utama Aplikasi Admin ---
export default function App() {
    const [user, setUser] = useState(null);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        // Listener untuk status otentikasi
        const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
            setUser(currentUser);
            setLoading(false);
        });
        return () => unsubscribe(); // Cleanup listener
    }, []);

    if (loading) {
        return <div className="flex items-center justify-center h-screen bg-slate-100"><div className="text-xl font-medium">Memuat...</div></div>;
    }

    return user ? <AdminDashboard /> : <LoginPage />;
}

// --- Komponen Halaman Login ---
function LoginPage() {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');
    const [error, setError] = useState('');
    const [loading, setLoading] = useState(false);
    
    // --- FUNGSI UNTUK MEMBUAT USER ADMIN PERTAMA KALI ---
    // Panggil fungsi ini dari console browser sekali untuk membuat akun admin
    // window.createAdmin('admin@darulhuda.com', 'passwordkuat123');
    useEffect(() => {
        window.createAdmin = async (email, pass) => {
            try {
                const userCredential = await createUserWithEmailAndPassword(auth, email, pass);
                console.log('Admin user created:', userCredential.user);
                alert('User admin berhasil dibuat!');
            } catch (error) {
                console.error('Error creating admin user:', error);
                alert('Gagal membuat user admin: ' + error.message);
            }
        };
    }, []);


    const handleLogin = async (e) => {
        e.preventDefault();
        setError('');
        setLoading(true);
        try {
            await signInWithEmailAndPassword(auth, email, password);
        } catch (err) {
            setError("Email atau password salah. Silakan coba lagi.");
            console.error(err);
        } finally {
            setLoading(false);
        }
    };

    return (
        <div className="flex items-center justify-center min-h-screen bg-slate-100">
            <div className="w-full max-w-md p-8 space-y-6 bg-white rounded-xl shadow-lg">
                <div className="text-center">
                    <h1 className="text-3xl font-bold text-slate-800">Panel Admin</h1>
                    <p className="text-slate-500">Pondok Pesantren Darul Huda</p>
                </div>
                <form onSubmit={handleLogin} className="space-y-6">
                    <div>
                        <label htmlFor="email" className="block text-sm font-medium text-slate-700">Email</label>
                        <input
                            id="email"
                            type="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            required
                            className="w-full px-3 py-2 mt-1 border border-slate-300 rounded-md shadow-sm focus:ring-teal-500 focus:border-teal-500"
                            placeholder="email@anda.com"
                        />
                    </div>
                    <div>
                        <label htmlFor="password" className="block text-sm font-medium text-slate-700">Password</label>
                        <input
                            id="password"
                            type="password"
                            value={password}
                            onChange={(e) => setPassword(e.target.value)}
                            required
                            className="w-full px-3 py-2 mt-1 border border-slate-300 rounded-md shadow-sm focus:ring-teal-500 focus:border-teal-500"
                            placeholder="••••••••"
                        />
                    </div>
                    {error && <p className="text-sm text-red-600">{error}</p>}
                    <div>
                        <button type="submit" disabled={loading} className="w-full px-4 py-2 font-semibold text-white bg-teal-600 rounded-md hover:bg-teal-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-teal-500 disabled:bg-slate-400">
                            {loading ? 'Memproses...' : 'Login'}
                        </button>
                    </div>
                </form>
            </div>
        </div>
    );
}

// --- Komponen Dashboard Admin ---
function AdminDashboard() {
    const [currentPage, setCurrentPage] = useState('program');

    const renderPage = () => {
        switch (currentPage) {
            case 'program':
                return <ProgramManager />;
            case 'berita':
                return <NewsManager />;
            case 'galeri':
                return <GalleryManager />;
            case 'konten':
                return <ContentManager />;
            default:
                return <ProgramManager />;
        }
    };

    return (
        <div className="flex h-screen bg-slate-100">
            {/* Sidebar */}
            <aside className="w-64 bg-slate-800 text-white flex flex-col">
                <div className="p-6 text-2xl font-bold border-b border-slate-700">
                    Admin Darul Huda
                </div>
                <nav className="flex-1 p-4 space-y-2">
                    <button onClick={() => setCurrentPage('program')} className={`w-full text-left px-4 py-2 rounded-md ${currentPage === 'program' ? 'bg-teal-600' : 'hover:bg-slate-700'}`}>Kelola Program</button>
                    <button onClick={() => setCurrentPage('berita')} className={`w-full text-left px-4 py-2 rounded-md ${currentPage === 'berita' ? 'bg-teal-600' : 'hover:bg-slate-700'}`}>Kelola Berita</button>
                    <button onClick={() => setCurrentPage('galeri')} className={`w-full text-left px-4 py-2 rounded-md ${currentPage === 'galeri' ? 'bg-teal-600' : 'hover:bg-slate-700'}`}>Kelola Galeri</button>
                    <button onClick={() => setCurrentPage('konten')} className={`w-full text-left px-4 py-2 rounded-md ${currentPage === 'konten' ? 'bg-teal-600' : 'hover:bg-slate-700'}`}>Kelola Konten Statis</button>
                </nav>
                <div className="p-4 border-t border-slate-700">
                    <button onClick={() => signOut(auth)} className="w-full px-4 py-2 font-semibold text-white bg-red-600 rounded-md hover:bg-red-700">
                        Logout
                    </button>
                </div>
            </aside>

            {/* Main Content */}
            <main className="flex-1 p-8 overflow-y-auto">
                {renderPage()}
            </main>
        </div>
    );
}

// --- Komponen untuk Mengelola Program ---
function ProgramManager() {
    const [programs, setPrograms] = useState([]);
    const [title, setTitle] = useState('');
    const [description, setDescription] = useState('');
    const [icon, setIcon] = useState('book-marked');
    const [editingId, setEditingId] = useState(null);

    useEffect(() => {
        const unsubscribe = onSnapshot(collection(db, 'programs'), (snapshot) => {
            const programsData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setPrograms(programsData);
        });
        return () => unsubscribe();
    }, []);

    const handleSubmit = async (e) => {
        e.preventDefault();
        if (!title || !description) return;

        if (editingId) {
            const docRef = doc(db, 'programs', editingId);
            await updateDoc(docRef, { title, description, icon });
        } else {
            await addDoc(collection(db, 'programs'), { title, description, icon });
        }
        resetForm();
    };

    const handleEdit = (program) => {
        setEditingId(program.id);
        setTitle(program.title);
        setDescription(program.description);
        setIcon(program.icon);
    };
    
    const handleDelete = async (id) => {
        if (window.confirm("Apakah Anda yakin ingin menghapus program ini?")) {
            await deleteDoc(doc(db, 'programs', id));
        }
    };

    const resetForm = () => {
        setEditingId(null);
        setTitle('');
        setDescription('');
        setIcon('book-marked');
    };

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800 mb-6">Kelola Program Unggulan</h2>
            <div className="bg-white p-6 rounded-lg shadow-md mb-8">
                <h3 className="text-xl font-semibold mb-4">{editingId ? 'Edit Program' : 'Tambah Program Baru'}</h3>
                <form onSubmit={handleSubmit} className="space-y-4">
                    <input type="text" placeholder="Judul Program" value={title} onChange={e => setTitle(e.target.value)} className="w-full p-2 border rounded" required />
                    <textarea placeholder="Deskripsi Program" value={description} onChange={e => setDescription(e.target.value)} className="w-full p-2 border rounded" rows="3" required></textarea>
                    <input type="text" placeholder="Nama Ikon Lucide (cth: book-marked)" value={icon} onChange={e => setIcon(e.target.value)} className="w-full p-2 border rounded" required />
                    <div className="flex gap-4">
                        <button type="submit" className="px-4 py-2 text-white bg-teal-600 rounded hover:bg-teal-700">{editingId ? 'Update' : 'Simpan'}</button>
                        {editingId && <button type="button" onClick={resetForm} className="px-4 py-2 text-slate-700 bg-slate-200 rounded hover:bg-slate-300">Batal</button>}
                    </div>
                </form>
            </div>
            <div className="bg-white p-6 rounded-lg shadow-md">
                <h3 className="text-xl font-semibold mb-4">Daftar Program Saat Ini</h3>
                <div className="space-y-4">
                    {programs.map(program => (
                        <div key={program.id} className="p-4 border rounded-md flex justify-between items-center">
                            <div>
                                <h4 className="font-bold text-lg">{program.title}</h4>
                                <p className="text-slate-600">{program.description}</p>
                                <small className="text-slate-400">Ikon: {program.icon}</small>
                            </div>
                            <div className="flex gap-2">
                                <button onClick={() => handleEdit(program)} className="px-3 py-1 text-sm text-white bg-blue-500 rounded hover:bg-blue-600">Edit</button>
                                <button onClick={() => handleDelete(program.id)} className="px-3 py-1 text-sm text-white bg-red-500 rounded hover:bg-red-600">Hapus</button>
                            </div>
                        </div>
                    ))}
                </div>
            </div>
        </div>
    );
}

// --- Komponen untuk Mengelola Berita ---
function NewsManager() {
    const [news, setNews] = useState([]);
    const [title, setTitle] = useState('');
    const [imageUrl, setImageUrl] = useState(''); // State untuk URL gambar
    const [content, setContent] = useState('');
    const [editingId, setEditingId] = useState(null);

    useEffect(() => {
        const unsubscribe = onSnapshot(collection(db, 'news'), (snapshot) => {
            const newsData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            newsData.sort((a, b) => (b.createdAt?.toDate() || 0) - (a.createdAt?.toDate() || 0));
            setNews(newsData);
        });
        return () => unsubscribe();
    }, []);

    const handleSubmit = async (e) => {
        e.preventDefault();
        if (!title || !content) return;

        const dataToSave = { 
            title, 
            imageUrl, 
            content 
        };

        if (editingId) {
            const docRef = doc(db, 'news', editingId);
            await updateDoc(docRef, dataToSave);
        } else {
            await addDoc(collection(db, 'news'), { 
                ...dataToSave,
                createdAt: serverTimestamp() 
            });
        }
        resetForm();
    };

    const handleEdit = (post) => {
        setEditingId(post.id);
        setTitle(post.title);
        setImageUrl(post.imageUrl || ''); // Isi field gambar saat edit
        setContent(post.content);
    };

    const handleDelete = async (id) => {
        if (window.confirm("Apakah Anda yakin ingin menghapus berita ini?")) {
            await deleteDoc(doc(db, 'news', id));
        }
    };

    const resetForm = () => {
        setEditingId(null);
        setTitle('');
        setImageUrl(''); // Reset field gambar
        setContent('');
    };

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800 mb-6">Kelola Berita</h2>
            
            <div className="bg-white p-6 rounded-lg shadow-md mb-8">
                <h3 className="text-xl font-semibold mb-4">{editingId ? 'Edit Berita' : 'Tulis Berita Baru'}</h3>
                <form onSubmit={handleSubmit} className="space-y-4">
                    <input type="text" placeholder="Judul Berita" value={title} onChange={e => setTitle(e.target.value)} className="w-full p-2 border rounded" required />
                    <input type="url" placeholder="URL Gambar Utama Berita" value={imageUrl} onChange={e => setImageUrl(e.target.value)} className="w-full p-2 border rounded" />
                    <textarea placeholder="Isi Berita..." value={content} onChange={e => setContent(e.target.value)} className="w-full p-2 border rounded" rows="8" required></textarea>
                    <div className="flex gap-4">
                        <button type="submit" className="px-4 py-2 text-white bg-teal-600 rounded hover:bg-teal-700">{editingId ? 'Update Berita' : 'Terbitkan'}</button>
                        {editingId && <button type="button" onClick={resetForm} className="px-4 py-2 text-slate-700 bg-slate-200 rounded hover:bg-slate-300">Batal</button>}
                    </div>
                </form>
            </div>

            <div className="bg-white p-6 rounded-lg shadow-md">
                <h3 className="text-xl font-semibold mb-4">Daftar Berita</h3>
                <div className="space-y-4">
                    {news.map(post => (
                        <div key={post.id} className="p-4 border rounded-md flex items-start">
                             {post.imageUrl && (
                                <img 
                                    src={post.imageUrl} 
                                    alt={post.title} 
                                    className="w-32 h-24 object-cover rounded-md mr-4 flex-shrink-0"
                                    onError={(e) => { e.target.onerror = null; e.target.src='https://placehold.co/128x96/e2e8f0/94a3b8?text=Gambar+Rusak'; }}
                                />
                            )}
                            <div className="flex-grow">
                                <h4 className="font-bold text-lg">{post.title}</h4>
                                <p className="text-slate-600 mt-1 line-clamp-2">{post.content}</p>
                                <small className="text-slate-400">
                                    Diterbitkan pada: {post.createdAt ? new Date(post.createdAt.seconds * 1000).toLocaleDateString('id-ID') : 'Baru saja'}
                                </small>
                            </div>
                            <div className="flex gap-2 flex-shrink-0 ml-4">
                                <button onClick={() => handleEdit(post)} className="px-3 py-1 text-sm text-white bg-blue-500 rounded hover:bg-blue-600">Edit</button>
                                <button onClick={() => handleDelete(post.id)} className="px-3 py-1 text-sm text-white bg-red-500 rounded hover:bg-red-600">Hapus</button>
                            </div>
                        </div>
                    ))}
                </div>
            </div>
        </div>
    );
}

// --- Komponen untuk Mengelola Galeri ---
function GalleryManager() {
    const [gallery, setGallery] = useState([]);
    const [imageUrl, setImageUrl] = useState('');
    const [altText, setAltText] = useState('');
    
    useEffect(() => {
        const unsubscribe = onSnapshot(collection(db, 'gallery'), (snapshot) => {
            const galleryData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
            setGallery(galleryData);
        });
        return () => unsubscribe();
    }, []);

    const handleSubmit = async (e) => {
        e.preventDefault();
        if (!imageUrl) return;
        await addDoc(collection(db, 'gallery'), { imageUrl, altText, createdAt: new Date() });
        setImageUrl('');
        setAltText('');
    };

    const handleDelete = async (id) => {
        if (window.confirm("Apakah Anda yakin ingin menghapus gambar ini?")) {
            await deleteDoc(doc(db, 'gallery', id));
        }
    };

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800 mb-6">Kelola Galeri</h2>
            
            <div className="bg-white p-6 rounded-lg shadow-md mb-8">
                <h3 className="text-xl font-semibold mb-4">Tambah Gambar Baru</h3>
                <form onSubmit={handleSubmit} className="space-y-4">
                    <input type="url" placeholder="URL Gambar" value={imageUrl} onChange={e => setImageUrl(e.target.value)} className="w-full p-2 border rounded" required />
                    <input type="text" placeholder="Teks Alternatif (Deskripsi Gambar)" value={altText} onChange={e => setAltText(e.target.value)} className="w-full p-2 border rounded" />
                    <button type="submit" className="px-4 py-2 text-white bg-teal-600 rounded hover:bg-teal-700">Tambah ke Galeri</button>
                </form>
            </div>
            
            <div className="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
                {gallery.map(item => (
                    <div key={item.id} className="relative group">
                        <img src={item.imageUrl} alt={item.altText} className="w-full h-48 object-cover rounded-lg shadow-md" />
                        <div className="absolute inset-0 bg-black bg-opacity-40 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity">
                            <button onClick={() => handleDelete(item.id)} className="px-4 py-2 text-white bg-red-600 rounded-md hover:bg-red-700">Hapus</button>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    );
}

// --- Komponen untuk Mengelola Konten Statis ---
function ContentManager() {
    const [about, setAbout] = useState({ title: '', subtitle: '', body: '' });
    const [contact, setContact] = useState({ address: '', phone1: '', phone2: '', email: '' });
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        const fetchContent = async () => {
            setLoading(true);
            const aboutDoc = doc(db, 'site_content', 'about_us');
            const contactDoc = doc(db, 'site_content', 'contact_info');
            
            onSnapshot(aboutDoc, (docSnap) => {
                if (docSnap.exists()) setAbout(docSnap.data());
            });

            onSnapshot(contactDoc, (docSnap) => {
                if (docSnap.exists()) setContact(docSnap.data());
            });
            setLoading(false);
        };
        fetchContent();
    }, []);

    const handleAboutSave = async () => {
        const docRef = doc(db, 'site_content', 'about_us');
        await setDoc(docRef, about, { merge: true });
        alert('Informasi "Tentang Kami" berhasil disimpan!');
    };
    
    const handleContactSave = async () => {
        const docRef = doc(db, 'site_content', 'contact_info');
        await setDoc(docRef, contact, { merge: true });
        alert('Informasi Kontak berhasil disimpan!');
    };

    if (loading) return <div>Memuat konten...</div>;

    return (
        <div>
            <h2 className="text-3xl font-bold text-slate-800 mb-6">Kelola Konten Statis</h2>
            
            <div className="bg-white p-6 rounded-lg shadow-md mb-8">
                <h3 className="text-xl font-semibold mb-4">Bagian "Tentang Kami"</h3>
                <div className="space-y-4">
                    <input type="text" placeholder="Judul" value={about.title} onChange={e => setAbout({...about, title: e.target.value})} className="w-full p-2 border rounded" />
                    <input type="text" placeholder="Sub Judul" value={about.subtitle} onChange={e => setAbout({...about, subtitle: e.target.value})} className="w-full p-2 border rounded" />
                    <textarea placeholder="Isi Konten" value={about.body} onChange={e => setAbout({...about, body: e.target.value})} className="w-full p-2 border rounded" rows="5"></textarea>
                    <button onClick={handleAboutSave} className="px-4 py-2 text-white bg-teal-600 rounded hover:bg-teal-700">Simpan Perubahan</button>
                </div>
            </div>

            <div className="bg-white p-6 rounded-lg shadow-md">
                <h3 className="text-xl font-semibold mb-4">Informasi Kontak</h3>
                <div className="space-y-4">
                    <textarea placeholder="Alamat" value={contact.address} onChange={e => setContact({...contact, address: e.target.value})} className="w-full p-2 border rounded" rows="3"></textarea>
                    <input type="text" placeholder="Telepon Kantor" value={contact.phone1} onChange={e => setContact({...contact, phone1: e.target.value})} className="w-full p-2 border rounded" />
                    <input type="text" placeholder="Telepon Panitia PSB" value={contact.phone2} onChange={e => setContact({...contact, phone2: e.target.value})} className="w-full p-2 border rounded" />
                    <input type="email" placeholder="Email" value={contact.email} onChange={e => setContact({...contact, email: e.target.value})} className="w-full p-2 border rounded" />
                    <button onClick={handleContactSave} className="px-4 py-2 text-white bg-teal-600 rounded hover:bg-teal-700">Simpan Perubahan</button>
                </div>
            </div>
        </div>
    );
}
