# -<!DOCTYPE html>
<html lang="en" class="scroll-smooth">
<head>
  <meta charset="UTF-8"/>
  <meta name="viewport" content="width=device-width,initial-scale=1"/>
  <title>PlayWith – Streamers</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
  <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" />
  <style>
    :root {
      --color-primary: #FF7A00;
      --color-cream: #FFF8E7;
    }
    html.dark {
      --color-cream: #1E1E1E;
    }
  </style>
</head>

<body class="bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-100 transition-colors">
  <div id="root"></div>

  <script type="text/babel">
    const { useState, useEffect, createContext, useContext } = React;

    /* ---------- CONFIG ---------- */
    const CONFIG = {
      streamers: [{ id: 1, name: 'John “JJ”', game: 'UFC 5', price: 10, avatar: 'https://i.pravatar.cc/200?u=jj', rating: 4.9, reviews: 128 }],
    };

    /* ---------- AUTH CONTEXT ---------- */
    const AuthContext = createContext(null);
    const useAuth = () => useContext(AuthContext);

    const AuthProvider = ({ children }) => {
      const [user, setUser] = useState(() => JSON.parse(localStorage.getItem('user')));
      const login = (email) => {
        const u = { email, avatar: `https://i.pravatar.cc/200?u=${email}` };
        localStorage.setItem('user', JSON.stringify(u));
        setUser(u);
      };
      const logout = () => {
        localStorage.removeItem('user');
        setUser(null);
      };
      const register = login; // demo: same as login
      return <AuthContext.Provider value={{ user, login, logout, register }}>{children}</AuthContext.Provider>;
    };

    /* ---------- THEME ---------- */
    const useTheme = () => {
      const [theme, setTheme] = useState(localStorage.theme ?? 'light');
      useEffect(() => {
        document.documentElement.classList.toggle('dark', theme === 'dark');
        localStorage.theme = theme;
      }, [theme]);
      return [theme, () => setTheme(t => t === 'light' ? 'dark' : 'light')];
    };

    /* ---------- MODAL ---------- */
    const Modal = ({ open, onClose, title, children }) => {
      if (!open) return null;
      return (
        <div className="fixed inset-0 bg-black/60 flex items-center justify-center z-50 px-4" onClick={onClose}>
          <div className="bg-white dark:bg-gray-800 rounded-xl shadow-xl p-6 w-full max-w-sm" onClick={e => e.stopPropagation()}>
            <h2 className="text-xl font-bold mb-4">{title}</h2>
            {children}
          </div>
        </div>
      );
    };

    /* ---------- NAVBAR ---------- */
    const Navbar = ({ toggleTheme }) => {
      const { user, logout } = useAuth();
      return (
        <nav className="bg-white/80 dark:bg-gray-800/80 backdrop-blur-md shadow-sm sticky top-0 z-40">
          <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div className="flex justify-between items-center h-16">
              <span className="text-2xl font-bold" style={{color: 'var(--color-primary)'}}>PlayWith</span>
              <div className="flex items-center space-x-4">
                <a href="#hero" className="hover:text-orange-600">Home</a>
                <a href="#streamers" className="hover:text-orange-600">Streamers</a>
                {user ? (
                  <>
                    <img src={user.avatar} alt="avatar" className="w-8 h-8 rounded-full"/>
                    <button onClick={logout} className="text-sm underline">Logout</button>
                  </>
                ) : null}
                <button onClick={toggleTheme} className="p-1 rounded-full hover:bg-gray-200 dark:hover:bg-gray-700">
                  <i className="fas fa-moon dark:hidden"></i>
                  <i className="fas fa-sun hidden dark:inline"></i>
                </button>
              </div>
            </div>
          </div>
        </nav>
      );
    };

    /* ---------- HERO ---------- */
    const Hero = () => (
      <section id="hero" className="py-20 text-center bg-gradient-to-br from-[var(--color-primary)] to-[var(--color-cream)] dark:to-gray-800">
        <h1 className="text-5xl font-bold text-white mb-4">Play Games With Pro Streamers</h1>
        <p className="text-xl text-white/90 max-w-lg mx-auto mb-8">Join live games, chat with streamers, and have fun while paying to play!</p>
      </section>
    );

    /* ---------- STREAMER CARD ---------- */
    const StreamerCard = ({ streamer }) => {
      const { user } = useAuth();
      const [showLogin, setShowLogin] = useState(false);
      const handleJoin = () => user ? null : setShowLogin(true);
      return (
        <>
          <section id="streamers" className="py-12">
            <div className="max-w-4xl mx-auto px-4">
              <h2 className="text-3xl font-bold mb-8 text-center">Featured Streamer</h2>
              <div className="bg-white dark:bg-gray-800 rounded-2xl shadow-lg p-6 md:p-8 flex flex-col md:flex-row items-center gap-6">
                <img src={streamer.avatar} alt={streamer.name} className="w-32 h-32 rounded-full object-cover"/>
                <div className="flex-1 text-center md:text-left">
                  <h3 className="text-2xl font-bold">{streamer.name}</h3>
                  <p className="text-gray-600 dark:text-gray-300 mb-1">Playing <span className="font-semibold text-orange-600">{streamer.game}</span></p>
                  <p className="text-sm text-gray-500 dark:text-gray-400">★ {streamer.rating} ({streamer.reviews} reviews)</p>
                  <button
                    onClick={handleJoin}
                    className="mt-4 bg-orange-500 text-white font-semibold py-3 px-6 rounded-full shadow hover:bg-orange-600 transition"
                  >
                    {user ? `Join Game ($${streamer.price})` : 'Login to Join'}
                  </button>
                </div>
              </div>
            </div>
          </section>
          {showLogin && <LoginModal onClose={() => setShowLogin(false)} />}
        </>
      );
    };

    /* ---------- LOGIN / REGISTER MODAL ---------- */
    const LoginModal = ({ onClose }) => {
      const { login, register } = useAuth();
      const [isLogin, setIsLogin] = useState(true);
      const [email, setEmail] = useState('');
      const handleSubmit = (e) => {
        e.preventDefault();
        if (!email) return;
        isLogin ? login(email) : register(email);
        onClose();
      };
      return (
        <Modal open onClose={onClose} title={isLogin ? 'Log In' : 'Sign Up'}>
          <form onSubmit={handleSubmit} className="space-y-4">
            <input type="email" value={email} onChange={e => setEmail(e.target.value)} placeholder="Email" required
              className="w-full border rounded-lg px-3 py-2 focus:ring-orange-500"/>
            <button type="submit" className="w-full bg-orange-500 text-white rounded-lg py-2 font-semibold">
              {isLogin ? 'Log In' : 'Create Account'}
            </button>
            <p className="text-sm text-center">
              {isLogin ? "Don't have an account?" : 'Already have one?'}{' '}
              <button type="button" onClick={() => setIsLogin(!isLogin)} className="underline text-orange-600">
                {isLogin ? 'Sign up' : 'Log in'}
              </button>
            </p>
          </form>
        </Modal>
      );
    };

    /* ---------- PAYMENT ---------- */
    const PaymentForm = ({ streamer }) => {
      const { user } = useAuth();
      if (!user) return null;
      const [name, setName] = useState(user.email.split('@')[0]);
      const [amount, setAmount] = useState(String(streamer.price));
      const [loading, setLoading] = useState(false);

      const handleSubmit = e => {
        e.preventDefault();
        setLoading(true);
        setTimeout(() => {
          alert(`Success! $${amount} paid by ${name}.`);
          setLoading(false);
        }, 1500);
      };

      return (
        <section id="payment" className="py-12">
          <div className="max-w-md mx-auto px-4">
            <div className="bg-white dark:bg-gray-800 rounded-2xl shadow-lg p-6 md:p-8">
              <h2 className="text-2xl font-bold mb-2">Checkout</h2>
              <form onSubmit={handleSubmit} className="space-y-4">
                <input value={name} onChange={e => setName(e.target.value)} placeholder="Name" required className="w-full border rounded-lg px-3 py-2"/>
                <input type="number" min="1" step="0.01" value={amount} onChange={e => setAmount(e.target.value)} required className="w-full border rounded-lg px-3 py-2"/>
                <button disabled={loading} className="w-full bg-orange-500 text-white rounded-lg py-2 font-semibold">
                  {loading ? 'Processing...' : `Pay $${amount}`}
                </button>
              </form>
            </div>
          </div>
        </section>
      );
    };

    /* ---------- FOOTER ---------- */
    const Footer = () => (
      <footer id="contact" className="bg-white dark:bg-gray-800 py-10 mt-16">
        <div className="max-w-7xl mx-auto px-4 text-center text-gray-600 dark:text-gray-400">
          <p>© {new Date().getFullYear()} PlayWith. All rights reserved.</p>
          <div className="flex justify-center space-x-6 mt-4">
            <a href="https://twitter.com" className="hover:text-orange-500"><i className="fab fa-twitter"/></a>
            <a href="https://instagram.com" className="hover:text-orange-500"><i className="fab fa-instagram"/></a>
            <a href="https://discord.com" className="hover:text-orange-500"><i className="fab fa-discord"/></a>
          </div>
        </div>
      </footer>
    );

    /* ---------- APP ---------- */
    const App = () => {
      const [theme, toggleTheme] = useTheme();
      const [streamer] = useState(CONFIG.streamers[0]);
      return (
        <AuthProvider>
          <Navbar toggleTheme={toggleTheme}/>
          <Hero/>
          <StreamerCard streamer={streamer}/>
          <PaymentForm streamer={streamer}/>
          <Footer/>
        </AuthProvider>
      );
    };

    ReactDOM.render(<App />, document.getElementById('root'));
  </script>
</body>
</html>
