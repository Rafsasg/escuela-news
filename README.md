import React, { useState, useEffect, createContext, useContext, useCallback } from 'react';
import { ChevronLeft, ChevronRight, Sun, Moon, Bell, Search, Image as ImageIcon, Video, FileText, PlusCircle, Trash2, MoreVertical, ThumbsUp, Heart, Award, Star, MessageCircle, X, CheckCircle, Calendar, GalleryHorizontal, User, BarChart2, Lightbulb, Send } from 'lucide-react';

// CONTEXTO DE AUTENTICACIÓN Y TEMA
const AuthContext = createContext(null);
const ThemeContext = createContext(null);

// DATOS DE EJEMPLO (MOCK DATA)
// En una aplicación real, esto vendría de una base de datos como Firestore.
const initialUsers = {
  'admin@school.edu': { name: 'Admin Directivo', role: 'admin', photo: 'https://placehold.co/100x100/6366f1/ffffff?text=A' },
  'teacher@school.edu': { name: 'Prof. Ana María', role: 'teacher', photo: 'https://placehold.co/100x100/ec4899/ffffff?text=P' },
  'student@school.edu': { name: 'Carlos Valderrama', role: 'student', grade: '10°A', photo: 'https://placehold.co/100x100/22c55e/ffffff?text=C' },
  'parent@school.edu': { name: 'Familia Rodriguez', role: 'parent', photo: 'https://placehold.co/100x100/f97316/ffffff?text=F' },
};

const initialNews = [
    {
        id: 1,
        title: '¡Gran victoria en el torneo intercolegial de fútbol!',
        category: 'logros',
        author: 'Prof. Ana María',
        date: '2025-06-28',
        isFeatured: true,
        imageUrl: 'https://placehold.co/800x400/16a34a/ffffff?text=Fútbol',
        content: 'Nuestro equipo de fútbol ha demostrado una vez más su talento y dedicación al ganar la final del torneo intercolegial. ¡Felicitaciones a todos los jugadores y al entrenador por este increíble logro que nos llena de orgullo! La final se disputó en un partido emocionante que mantuvo a todos al borde de sus asientos.'
    },
    {
        id: 2,
        title: 'Convocatoria para el club de ciencias y tecnología',
        category: 'convocatorias',
        author: 'Admin Directivo',
        date: '2025-06-27',
        isFeatured: true,
        imageUrl: 'https://placehold.co/800x400/0ea5e9/ffffff?text=Ciencia',
        content: 'Se abre la convocatoria para todos los estudiantes interesados en formar parte del club de ciencias. Las inscripciones estarán abiertas hasta el 15 de julio. ¡No pierdas la oportunidad de explorar el fascinante mundo de la ciencia y la tecnología! Los proyectos de este año incluyen robótica, astronomía y biotecnología.'
    },
    {
        id: 3,
        title: 'Aviso importante: Suspensión de clases por jornada pedagógica',
        category: 'avisos',
        author: 'Admin Directivo',
        date: '2025-06-29',
        isUrgent: true,
        content: 'Se informa a toda la comunidad educativa que el próximo viernes, 4 de julio, se suspenderán las clases por una jornada de capacitación docente. Agradecemos su comprensión.'
    },
    {
        id: 4,
        title: 'Festival de Arte y Cultura: una explosión de talento',
        category: 'eventos',
        author: 'Prof. Ana María',
        date: '2025-06-26',
        imageUrl: 'https://placehold.co/800x400/d946ef/ffffff?text=Arte',
        content: 'El pasado fin de semana celebramos nuestro festival anual de arte, con presentaciones de música, teatro y exposiciones de pintura. Fue una muestra increíble del talento de nuestros estudiantes.'
    }
];

// HOOKS PERSONALIZADOS
const useAuth = () => useContext(AuthContext);
const useTheme = () => useContext(ThemeContext);

// COMPONENTES DE UI BÁSICOS
const Button = ({ children, onClick, className = '', variant = 'primary', ...props }) => {
    const variants = {
        primary: 'bg-blue-600 text-white hover:bg-blue-700',
        secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 dark:bg-gray-700 dark:text-gray-200 dark:hover:bg-gray-600',
        danger: 'bg-red-600 text-white hover:bg-red-700',
        ghost: 'bg-transparent text-gray-600 hover:bg-gray-100 dark:text-gray-300 dark:hover:bg-gray-700',
    };
    return <button onClick={onClick} className={`px-4 py-2 rounded-lg font-semibold transition-colors duration-200 focus:outline-none focus:ring-2 focus:ring-offset-2 dark:focus:ring-offset-gray-900 focus:ring-blue-500 ${variants[variant]} ${className}`} {...props}>{children}</button>;
};

const Card = ({ children, className = '' }) => (
    <div className={`bg-white dark:bg-gray-800 shadow-md rounded-xl overflow-hidden ${className}`}>
        {children}
    </div>
);

const Modal = ({ isOpen, onClose, title, children }) => {
    if (!isOpen) return null;
    return (
        <div className="fixed inset-0 bg-black bg-opacity-50 z-50 flex justify-center items-center p-4">
            <div className="bg-white dark:bg-gray-800 rounded-lg shadow-xl w-full max-w-2xl max-h-[90vh] flex flex-col">
                <div className="flex justify-between items-center p-4 border-b dark:border-gray-700">
                    <h2 className="text-xl font-bold text-gray-800 dark:text-gray-100">{title}</h2>
                    <Button onClick={onClose} variant="ghost" className="!p-2 rounded-full"><X size={24} /></Button>
                </div>
                <div className="p-6 overflow-y-auto">{children}</div>
            </div>
        </div>
    );
};

// COMPONENTES DE LA APLICACIÓN
const UrgentBanner = ({ message, onClose }) => (
    <div className="bg-red-500 text-white p-3 text-center font-bold flex justify-center items-center relative">
        <Bell size={20} className="mr-3" />
        <span>{message}</span>
        <button onClick={onClose} className="absolute right-4 top-1/2 -translate-y-1/2"><X size={20} /></button>
    </div>
);

const NewsCarousel = ({ news, onSelectNews }) => {
    const [currentIndex, setCurrentIndex] = useState(0);
    const featuredNews = news.filter(n => n.isFeatured);

    const nextSlide = useCallback(() => {
        setCurrentIndex(prev => (prev + 1) % featuredNews.length);
    }, [featuredNews.length]);

    const prevSlide = () => {
        setCurrentIndex(prev => (prev - 1 + featuredNews.length) % featuredNews.length);
    };

    useEffect(() => {
        const timer = setInterval(nextSlide, 5000);
        return () => clearInterval(timer);
    }, [nextSlide]);
    
    if (featuredNews.length === 0) return null;

    return (
        <div className="relative w-full h-64 md:h-96 rounded-2xl overflow-hidden shadow-2xl group">
            <div className="w-full h-full flex transition-transform duration-700 ease-in-out" style={{ transform: `translateX(-${currentIndex * 100}%)` }}>
                {featuredNews.map((item) => (
                    <div key={item.id} className="w-full h-full flex-shrink-0 relative cursor-pointer" onClick={() => onSelectNews(item)}>
                        <img src={item.imageUrl} alt={item.title} className="w-full h-full object-cover" />
                        <div className="absolute inset-0 bg-gradient-to-t from-black/70 to-transparent"></div>
                        <div className="absolute bottom-0 left-0 p-4 md:p-8">
                            <h2 className="text-white text-2xl md:text-4xl font-bold">{item.title}</h2>
                            <p className="text-gray-200 mt-2">{item.author}</p>
                        </div>
                    </div>
                ))}
            </div>
            <button onClick={prevSlide} className="absolute top-1/2 left-4 -translate-y-1/2 bg-white/50 hover:bg-white/80 p-2 rounded-full text-gray-800 transition-opacity opacity-0 group-hover:opacity-100">
                <ChevronLeft size={24} />
            </button>
            <button onClick={nextSlide} className="absolute top-1/2 right-4 -translate-y-1/2 bg-white/50 hover:bg-white/80 p-2 rounded-full text-gray-800 transition-opacity opacity-0 group-hover:opacity-100">
                <ChevronRight size={24} />
            </button>
        </div>
    );
};

const CategoryPill = ({ category, selected, onClick }) => {
    const styles = {
        eventos: 'bg-purple-100 text-purple-800 dark:bg-purple-900 dark:text-purple-200',
        avisos: 'bg-yellow-100 text-yellow-800 dark:bg-yellow-900 dark:text-yellow-200',
        logros: 'bg-green-100 text-green-800 dark:bg-green-900 dark:text-green-200',
        convocatorias: 'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-200',
        default: 'bg-gray-100 text-gray-800 dark:bg-gray-700 dark:text-gray-200'
    };
    const selectedStyle = 'ring-2 ring-offset-2 dark:ring-offset-gray-900 ring-blue-500';

    return (
        <button onClick={onClick} className={`px-4 py-1.5 rounded-full text-sm font-semibold transition-all duration-200 ${styles[category] || styles.default} ${selected ? selectedStyle : ''}`}>
            {category.charAt(0).toUpperCase() + category.slice(1)}
        </button>
    );
};

const NewsCard = ({ news, onSelectNews }) => (
    <Card className="hover:shadow-lg transition-shadow duration-300 cursor-pointer" onClick={() => onSelectNews(news)}>
        {news.imageUrl && <img src={news.imageUrl} alt={news.title} className="w-full h-48 object-cover" />}
        <div className="p-4">
            <CategoryPill category={news.category} onClick={(e) => { e.stopPropagation(); /* handle filter */ }} />
            <h3 className="font-bold text-lg mt-3 text-gray-800 dark:text-gray-100">{news.title}</h3>
            <p className="text-gray-500 dark:text-gray-400 text-sm mt-2">{news.author} • {new Date(news.date).toLocaleDateString()}</p>
        </div>
    </Card>
);

const Reaction = ({ icon: Icon, count, active, onClick, label }) => (
    <button onClick={onClick} className={`flex items-center space-x-1.5 text-gray-600 dark:text-gray-300 hover:text-blue-500 dark:hover:text-blue-400 transition-colors duration-200 p-2 rounded-lg hover:bg-gray-100 dark:hover:bg-gray-700 ${active ? 'text-blue-600 dark:text-blue-500' : ''}`}>
        <Icon size={20} className={active ? 'fill-current' : ''} />
        <span className="font-semibold text-sm">{count}</span>
         <span className="hidden md:inline">{label}</span>
    </button>
);

const CommentSection = ({ comments, onAddComment }) => {
    const { user } = useAuth();
    const [newComment, setNewComment] = useState("");

    const handleCommentSubmit = (e) => {
        e.preventDefault();
        if (newComment.trim()) {
            onAddComment({ author: user.name, text: newComment, date: new Date().toISOString() });
            setNewComment("");
        }
    };
    
    if(!user || user.role === 'parent') return <div className="p-4 text-center text-gray-500 dark:text-gray-400">Inicia sesión como estudiante o docente para comentar.</div>;

    return (
        <div className="mt-8">
            <h3 className="text-xl font-bold mb-4 text-gray-800 dark:text-gray-100">Comentarios</h3>
            <form onSubmit={handleCommentSubmit} className="flex items-center space-x-2 mb-6">
                <img src={user.photo} alt={user.name} className="w-10 h-10 rounded-full" />
                <input
                    type="text"
                    value={newComment}
                    onChange={(e) => setNewComment(e.target.value)}
                    placeholder="Escribe un comentario..."
                    className="w-full bg-gray-100 dark:bg-gray-700 border-transparent focus:border-blue-500 focus:ring-blue-500 rounded-full px-4 py-2"
                />
                <Button type="submit" className="!p-2.5 rounded-full"><Send size={20} /></Button>
            </form>
            <div className="space-y-4">
                {comments.map((comment, index) => (
                    <div key={index} className="flex items-start space-x-3">
                        <img src={`https://placehold.co/40x40/7c3aed/ffffff?text=${comment.author.charAt(0)}`} alt={comment.author} className="w-10 h-10 rounded-full" />
                        <div className="bg-gray-100 dark:bg-gray-700 rounded-lg p-3 flex-1">
                            <p className="font-semibold text-gray-800 dark:text-gray-200">{comment.author}</p>
                            <p className="text-gray-600 dark:text-gray-300">{comment.text}</p>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    );
};

// VISTAS/PÁGINAS
const HomePage = ({ news, setNews, onSelectNews }) => {
    const [filter, setFilter] = useState('all');
    const [showUrgent, setShowUrgent] = useState(true);
    
    const urgentNews = news.find(n => n.isUrgent);
    
    const filteredNews = news
        .filter(n => filter === 'all' || n.category === filter)
        .sort((a, b) => new Date(b.date) - new Date(a.date));

    const categories = [...new Set(news.map(n => n.category))];

    return (
        <div className="space-y-8">
            {showUrgent && urgentNews && <UrgentBanner message={urgentNews.title} onClose={() => setShowUrgent(false)} />}
            
            <NewsCarousel news={news} onSelectNews={onSelectNews} />
            
            <div className="flex flex-wrap gap-2 items-center">
                <h2 className="text-2xl font-bold mr-4 text-gray-900 dark:text-white">Noticias Recientes</h2>
                <CategoryPill category="all" selected={filter === 'all'} onClick={() => setFilter('all')} />
                {categories.map(cat => <CategoryPill key={cat} category={cat} selected={filter === cat} onClick={() => setFilter(cat)} />)}
            </div>

            <div className="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
                {filteredNews.map(item => <NewsCard key={item.id} news={item} onSelectNews={onSelectNews} />)}
            </div>
        </div>
    );
};

const NewsDetailView = ({ newsItem, onClose, onComment, onReact }) => {
    const { user } = useAuth();
    
    const [reactions, setReactions] = useState(newsItem.reactions || { like: 12, proud: 5, wow: 3, congrats: 8 });
    const [myReactions, setMyReactions] = useState(newsItem.myReactions || {});
    const [comments, setComments] = useState(newsItem.comments || [{author: 'Lucía Gómez', text: '¡Qué gran noticia!', date: '2025-06-28'}]);

    const handleReaction = (type) => {
      // Solo permite reaccionar a estudiantes y docentes
      if (!user || !['student', 'teacher', 'admin'].includes(user.role)) return;

      setReactions(prev => ({ ...prev, [type]: myReactions[type] ? prev[type] - 1 : prev[type] + 1 }));
      setMyReactions(prev => ({ ...prev, [type]: !prev[type] }));
      // onReact(newsItem.id, type);
    };

    const handleAddComment = (comment) => {
        setComments(prev => [comment, ...prev]);
        // onComment(newsItem.id, comment);
    }
    
    const reactionTypes = [
        { type: 'like', icon: ThumbsUp, label: 'Me gusta' },
        { type: 'proud', icon: Award, label: 'Enorgullece' },
        { type: 'wow', icon: Star, label: 'Wow' },
        { type: 'congrats', icon: Heart, label: 'Felicitaciones' },
    ];

    return (
        <Modal isOpen={true} onClose={onClose} title={newsItem.title}>
            {newsItem.imageUrl && <img src={newsItem.imageUrl} alt={newsItem.title} className="w-full h-64 object-cover rounded-lg mb-4" />}
            <div className="flex justify-between items-center text-sm text-gray-500 dark:text-gray-400 mb-4">
                <span>Por: <strong>{newsItem.author}</strong></span>
                <span>{new Date(newsItem.date).toLocaleDateString('es-ES', { year: 'numeric', month: 'long', day: 'numeric' })}</span>
            </div>
            <p className="text-gray-700 dark:text-gray-300 whitespace-pre-wrap leading-relaxed">{newsItem.content}</p>

            <div className="mt-6 pt-4 border-t dark:border-gray-700">
                <div className="flex flex-wrap items-center gap-x-2 gap-y-2">
                    {reactionTypes.map(r => (
                        <Reaction 
                            key={r.type}
                            icon={r.icon}
                            count={reactions[r.type]}
                            active={myReactions[r.type]}
                            onClick={() => handleReaction(r.type)}
                            label={r.label}
                        />
                    ))}
                </div>
            </div>

            <CommentSection comments={comments} onAddComment={handleAddComment} />
        </Modal>
    );
};

const AdminDashboard = ({ onAddNews }) => {
    const { user } = useAuth();
    const [title, setTitle] = useState('');
    const [content, setContent] = useState('');
    const [category, setCategory] = useState('eventos');
    const [imageUrl, setImageUrl] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        const newNews = {
            id: Date.now(),
            title,
            content,
            category,
            imageUrl,
            author: user.name,
            date: new Date().toISOString().split('T')[0],
            isFeatured: false,
            isUrgent: false,
        };
        onAddNews(newNews);
        // Reset form
        setTitle(''); setContent(''); setCategory('eventos'); setImageUrl('');
    };
    
    if (!user || !['admin', 'teacher'].includes(user.role)) {
        return <div className="text-center p-8">No tienes permisos para acceder a esta sección.</div>
    }

    return (
        <div className="max-w-4xl mx-auto">
            <h1 className="text-3xl font-bold mb-6 text-gray-900 dark:text-white">Panel de Publicación</h1>
            <form onSubmit={handleSubmit} className="space-y-6 bg-white dark:bg-gray-800 p-8 rounded-2xl shadow-lg">
                <div>
                    <label htmlFor="title" className="block text-sm font-medium text-gray-700 dark:text-gray-300">Título de la Noticia</label>
                    <input type="text" id="title" value={title} onChange={e => setTitle(e.target.value)} className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white" required />
                </div>
                <div>
                    <label htmlFor="content" className="block text-sm font-medium text-gray-700 dark:text-gray-300">Contenido</label>
                    <textarea id="content" rows="6" value={content} onChange={e => setContent(e.target.value)} className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white" required></textarea>
                </div>
                <div>
                    <label htmlFor="category" className="block text-sm font-medium text-gray-700 dark:text-gray-300">Categoría</label>
                    <select id="category" value={category} onChange={e => setCategory(e.target.value)} className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white">
                        <option value="eventos">Eventos</option>
                        <option value="avisos">Avisos</option>
                        <option value="logros">Logros</option>
                        <option value="convocatorias">Convocatorias</option>
                    </select>
                </div>
                <div>
                    <label htmlFor="imageUrl" className="block text-sm font-medium text-gray-700 dark:text-gray-300">URL de la Imagen (Opcional)</label>
                    <input type="text" id="imageUrl" value={imageUrl} onChange={e => setImageUrl(e.target.value)} placeholder="https://ejemplo.com/imagen.jpg" className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white" />
                </div>
                <div className="flex justify-end space-x-4">
                    <Button type="button" variant="secondary">Vista Previa</Button>
                    <Button type="submit" variant="primary">Publicar Noticia</Button>
                </div>
            </form>
        </div>
    );
};

const LoginPage = ({ onLogin }) => {
    const [email, setEmail] = useState('');
    return (
        <div className="flex items-center justify-center min-h-screen bg-gray-100 dark:bg-gray-900">
            <Card className="w-full max-w-sm">
                <div className="p-8">
                    <h2 className="text-2xl font-bold text-center text-gray-800 dark:text-white">Iniciar Sesión</h2>
                    <p className="text-center text-gray-500 dark:text-gray-400 mt-2">Usa un correo de la lista para probar los roles.</p>
                    <div className="mt-6">
                        <label htmlFor="email" className="block text-sm font-medium text-gray-700 dark:text-gray-300">Correo Institucional</label>
                        <select
                            id="email"
                            value={email}
                            onChange={(e) => setEmail(e.target.value)}
                            className="mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-indigo-500 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white p-2"
                        >
                            <option value="">Selecciona un usuario</option>
                            {Object.keys(initialUsers).map(userEmail => (
                                <option key={userEmail} value={userEmail}>{userEmail} ({initialUsers[userEmail].role})</option>
                            ))}
                        </select>
                    </div>
                    <Button onClick={() => onLogin(email)} className="w-full mt-6" disabled={!email}>
                        Entrar
                    </Button>
                </div>
            </Card>
        </div>
    );
};


// COMPONENTE PRINCIPAL DE LA APLICACIÓN
function App() {
    // ESTADO GENERAL
    const [theme, setTheme] = useState('light');
    const [user, setUser] = useState(null);
    const [news, setNews] = useState(initialNews);
    const [selectedNews, setSelectedNews] = useState(null);
    const [currentPage, setCurrentPage] = useState('home'); // home, admin, profile, etc.
    
    // EFECTO PARA TEMA OSCURO
    useEffect(() => {
        if (theme === 'dark') {
            document.documentElement.classList.add('dark');
        } else {
            document.documentElement.classList.remove('dark');
        }
    }, [theme]);

    // MANEJADORES DE EVENTOS
    const toggleTheme = () => setTheme(theme === 'light' ? 'dark' : 'light');
    
    const handleLogin = (email) => {
        const foundUser = initialUsers[email];
        if (foundUser) {
            setUser({ ...foundUser, email });
            setCurrentPage('home');
        } else {
            alert("Usuario no encontrado.");
        }
    };
    
    const handleLogout = () => {
        setUser(null);
        setCurrentPage('home');
    };
    
    const handleAddNews = (newArticle) => {
        setNews([newArticle, ...news]);
        setCurrentPage('home');
    };

    // RENDERIZADO
    if (!user) {
        return <ThemeContext.Provider value={{ theme, toggleTheme }}><AuthContext.Provider value={{ user, onLogin: handleLogin, onLogout: handleLogout }}><LoginPage onLogin={handleLogin} /></AuthContext.Provider></ThemeContext.Provider>;
    }

    const renderPage = () => {
        switch (currentPage) {
            case 'admin':
                return <AdminDashboard onAddNews={handleAddNews} />;
            case 'home':
            default:
                return <HomePage news={news} setNews={setNews} onSelectNews={setSelectedNews} />;
        }
    };

    return (
        <ThemeContext.Provider value={{ theme, toggleTheme }}>
            <AuthContext.Provider value={{ user, onLogin: handleLogin, onLogout: handleLogout }}>
                <div className="bg-gray-50 dark:bg-gray-900 min-h-screen text-gray-900 dark:text-gray-100 transition-colors duration-300">
                    <header className="bg-white/80 dark:bg-gray-800/80 backdrop-blur-sm sticky top-0 z-40 shadow-sm">
                        <nav className="container mx-auto px-4 sm:px-6 lg:px-8">
                            <div className="flex items-center justify-between h-16">
                                <div className="flex items-center space-x-4">
                                    <div className="font-bold text-xl text-blue-600 dark:text-blue-400 cursor-pointer" onClick={() => setCurrentPage('home')}>
                                        NotiColegio
                                    </div>
                                    <div className="hidden md:flex space-x-4">
                                         <a href="#" onClick={() => setCurrentPage('home')} className="px-3 py-2 rounded-md text-sm font-medium hover:bg-gray-200 dark:hover:bg-gray-700">Inicio</a>
                                         { (user.role === 'admin' || user.role === 'teacher') && <a href="#" onClick={() => setCurrentPage('admin')} className="px-3 py-2 rounded-md text-sm font-medium hover:bg-gray-200 dark:hover:bg-gray-700">Publicar</a>}
                                         <a href="#" className="px-3 py-2 rounded-md text-sm font-medium hover:bg-gray-200 dark:hover:bg-gray-700">Calendario</a>
                                         <a href="#" className="px-3 py-2 rounded-md text-sm font-medium hover:bg-gray-200 dark:hover:bg-gray-700">Galería</a>
                                    </div>
                                </div>
                                <div className="flex items-center space-x-3">
                                    <Button onClick={toggleTheme} variant="ghost" className="!p-2 rounded-full">
                                        {theme === 'light' ? <Moon size={20} /> : <Sun size={20} />}
                                    </Button>
                                    <div className="relative">
                                        <Button variant="ghost" className="!p-2 rounded-full"><Bell size={20} /></Button>
                                        <span className="absolute -top-1 -right-1 flex h-3 w-3">
                                          <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-red-400 opacity-75"></span>
                                          <span className="relative inline-flex rounded-full h-3 w-3 bg-red-500"></span>
                                        </span>
                                    </div>
                                    <div className="relative group">
                                      <img src={user.photo} alt={user.name} className="w-10 h-10 rounded-full cursor-pointer"/>
                                      <div className="absolute right-0 mt-2 w-48 bg-white dark:bg-gray-800 rounded-md shadow-lg py-1 z-50 hidden group-hover:block">
                                        <div className="px-4 py-2 text-sm text-gray-700 dark:text-gray-200">
                                            <p className="font-bold">{user.name}</p>
                                            <p className="text-xs text-gray-500 dark:text-gray-400">{user.role}</p>
                                        </div>
                                        <div className="border-t border-gray-100 dark:border-gray-700"></div>
                                        <a href="#" className="block px-4 py-2 text-sm text-gray-700 dark:text-gray-200 hover:bg-gray-100 dark:hover:bg-gray-700">Mi Perfil</a>
                                        <a href="#" onClick={handleLogout} className="block px-4 py-2 text-sm text-red-600 dark:text-red-400 hover:bg-gray-100 dark:hover:bg-gray-700">Cerrar Sesión</a>
                                      </div>
                                    </div>
                                </div>
                            </div>
                        </nav>
                    </header>
                    <main className="container mx-auto p-4 sm:p-6 lg:p-8">
                        {renderPage()}
                    </main>

                    {selectedNews && (
                        <NewsDetailView
                            newsItem={selectedNews}
                            onClose={() => setSelectedNews(null)}
                            onComment={() => {}}
                            onReact={() => {}}
                        />
                    )}
                </div>
            </AuthContext.Provider>
        </ThemeContext.Provider>
    );
}

export default App;
