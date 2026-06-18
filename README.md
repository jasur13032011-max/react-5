# react-5
Mana, siz so'ragan barcha talablarga to'liq javob beradigan, har bir context qiymati useMemo bilan optimallashtirilgan, o'zining custom hook'lariga ega va xavfsizlik tekshiruvlari kiritilgan to'liq tizim.

Siz loyihangizda src/context/ papkasini ochib, quyidagi fayllarni joylashtirishingiz mumkin.

1. Context Fayllari (src/context/)
📄 UserContext.js
JavaScript
import React, { createContext, useState, useEffect, useMemo, useContext } from 'react';

const UserContext = createContext(null);

export function UserProvider({ children }) {
  const [user, setUser] = useState(() => {
    const saved = localStorage.getItem('app-user');
    return saved ? JSON.parse(saved) : null;
  });

  useEffect(() => {
    if (user) {
      localStorage.setItem('app-user', JSON.stringify(user));
    } else {
      localStorage.removeItem('app-user');
    }
  }, [user]);

  const login = (username) => setUser({ name: username, role: 'student' });
  const logout = () => setUser(null);

  // useMemo orqali qiymatni barqarorlashtirish
  const value = useMemo(() => ({ user, login, logout }), [user]);

  return <UserContext.Provider value={value}>{children}</UserContext.Provider>;
}

export function useUser() {
  const context = useContext(UserContext);
  if (context === undefined || context === null) {
    throw new Error('useUser faqat UserProvider ichida ishlatilishi shart!');
  }
  return context;
}
📄 ThemeContext.js
JavaScript
import React, { createContext, useState, useEffect, useMemo, useContext } from 'react';

const ThemeContext = createContext(null);

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState(() => {
    return localStorage.getItem('app-theme') || 'light';
  });

  useEffect(() => {
    localStorage.setItem('app-theme', theme);
    // CSS variables uchun body klassini o'zgartirish
    document.body.className = theme;
  }, [theme]);

  const toggleTheme = () => setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));

  const value = useMemo(() => ({ theme, toggleTheme }), [theme]);

  return <ThemeContext.Provider value={value}>{children}</ThemeContext.Provider>;
}

export function useTheme() {
  const context = useContext(ThemeContext);
  if (context === undefined || context === null) {
    throw new Error('useTheme faqat ThemeProvider ichida ishlatilishi shart!');
  }
  return context;
}
📄 CartContext.js
JavaScript
import React, { createContext, useState, useMemo, useContext } from 'react';

const CartContext = createContext(null);

export function CartProvider({ children }) {
  const [cartItems, setCartItems] = useState([]);

  const addToCart = (product) => {
    setCartItems((prevItems) => {
      const exists = prevItems.find((item) => item.id === product.id);
      if (exists) {
        return prevItems.map((item) =>
          item.id === product.id ? { ...item, quantity: item.quantity + 1 } : item
        );
      }
      return [...prevItems, { ...product, quantity: 1 }];
    });
  };

  const removeFromCart = (productId) => {
    setCartItems((prevItems) => prevItems.filter((item) => item.id !== productId));
  };

  const clearCart = () => setCartItems([]);

  // Jami summani hisoblash
  const totalSum = cartItems.reduce((sum, item) => sum + item.price * item.quantity, 0);

  const value = useMemo(
    () => ({ cartItems, addToCart, removeFromCart, clearCart, totalSum }),
    [cartItems, totalSum]
  );

  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

export function useCart() {
  const context = useContext(CartContext);
  if (context === undefined || context === null) {
    throw new Error('useCart faqat CartProvider ichida ishlatilishi shart!');
  }
  return context;
}
2. Asosiy Ilova App.js (Provider'larni o'rash)
JavaScript
import React from 'react';
import { UserProvider } from './context/UserContext';
import { ThemeProvider } from './context/ThemeContext';
import { CartProvider } from './context/CartContext';
import Header from './components/Header';
import ProductList from './components/ProductList';
import CartPage from './components/CartPage';
import './App.css';

export default function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <CartProvider>
          <div className="app-layout">
            <Header />
            <main className="main-grid">
              <ProductList />
              <CartPage />
            </main>
          </div>
        </CartProvider>
      </UserProvider>
    </ThemeProvider>
  );
}
3. Komponentlar (src/components/)
📄 Header.js (Context'lardan bevosita ma'lumot oladi)
JavaScript
import React from 'react';
import { useUser } from '../context/UserContext';
import { useTheme } from '../context/ThemeContext';
import { useCart } from '../context/CartContext';

export default function Header() {
  const { user, login, logout } = useUser();
  const { theme, toggleTheme } = useTheme();
  const { cartItems } = useCart();

  // Savatchadagi umumiy maxsulotlar soni
  const totalCount = cartItems.reduce((count, item) => count + item.quantity, 0);

  return (
    <header className="site-header">
      <div className="brand">🛒 MiniShop</div>
      
      <div className="header-actions">
        {/* Tema */}
        <button onClick={toggleTheme} className="btn-icon">
          {theme === 'light' ? '🌙 Dark Mode' : '☀️ Light Mode'}
        </button>

        {/* Savatcha signali */}
        <div className="cart-badge">Savatcha: <strong>{totalCount} ta</strong></div>

        {/* Profil */}
        {user ? (
          <div className="user-profile">
            <span>👤 {user.name}</span>
            <button onClick={logout} className="btn-sm btn-danger">Chiqish</button>
          </div>
        ) : (
          <button onClick={() => login('Jasur')} className="btn-sm btn-success">Kirish</button>
        )}
      </div>
    </header>
  );
}
📄 ProductList.js & Mahsulot Kartochkasi
JavaScript
import React from 'react';
import { useCart } from '../context/CartContext';

const PRODUCTS = [
  { id: 1, name: 'Simsiz Quloqchin', price: 250000 },
  { id: 2, name: 'Aqlli Soat', price: 450000 },
  { id: 3, name: 'Mexanik Klaviatura', price: 600000 },
];

export default function ProductList() {
  const { addToCart } = useCart(); // Bevosita useCart hook'i

  return (
    <div className="products-section">
      <h2>📦 Mahsulotlar</h2>
      <div className="product-grid">
        {PRODUCTS.map((product) => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p className="price">{product.price.toLocaleString()} so'm</p>
            <button onClick={() => addToCart(product)} className="btn-primary">
              Savatchaga qo'shish
            </button>
          </div>
        ))}
      </div>
    </div>
  );
}
📄 CartPage.js (Savatcha sahifasi)
JavaScript
import React from 'react';
import { useCart } from '../context/CartContext';

export default function CartPage() {
  const { cartItems, removeFromCart, clearCart, totalSum } = useCart();

  return (
    <div className="cart-section">
      <h2>🛒 Savatchangiz</h2>
      
      {cartItems.length === 0 ? (
        <p className="empty-cart">Savatchangiz hozircha bo'sh.</p>
      ) : (
        <div className="cart-box">
          <div className="cart-list">
            {cartItems.map((item) => (
              <div key={item.id} className="cart-item">
                <div className="item-info">
                  <h4>{item.name}</h4>
                  <small>{item.price.toLocaleString()} so'm x {item.quantity}</small>
                </div>
                <button onClick={() => removeFromCart(item.id)} className="btn-text-danger">
                  O'chirish
                </button>
              </div>
            ))}
          </div>

          <div className="cart-footer">
            <h3>Jami: {totalSum.toLocaleString()} so'm</h3>
            <button onClick={clearCart} className="btn-secondary">Savatchani tozalash</button>
          </div>
        </div>
      )}
    </div>
  );
}
4. Global CSS Variables bilan Dizayn (src/App.css)
Dizayn to'liq CSS o'zgaruvchilari (Variables) asosida qurilgan. ThemeContext orqali body klassi o'zgarganda butun ranglar palitrasi ham avtomatik o'zgaradi.

CSS
/* CSS Dinamik Ranglar */
body.light {
  --bg-color: #f8fafc;
  --card-bg: #ffffff;
  --text-color: #1e293b;
  --border-color: #e2e8f0;
  --header-bg: #0f172a;
  --header-text: #ffffff;
}

body.dark {
  --bg-color: #0f172a;
  --card-bg: #1e293b;
  --text-color: #f8fafc;
  --border-color: #334155;
  --header-bg: #1e293b;
  --header-text: #ffffff;
}

body {
  margin: 0;
  background-color: var(--bg-color);
  color: var(--text-color);
  font-family: system-ui, -apple-system, sans-serif;
  transition: background 0.3s, color 0.3s;
}

.app-layout {
  min-height: 100vh;
}

/* Header CSS */
.site-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: var(--header-bg);
  color: var(--header-text);
  padding: 15px 40px;
  box-shadow: 0 4px 6px -1px rgba(0,0,0,0.1);
}
.brand { font-size: 1.4rem; font-weight: bold; }
.header-actions { display: flex; gap: 20px; align-items: center; }

/* Grid Layout */
.main-grid {
  max-width: 1100px;
  margin: 40px auto;
  display: grid;
  grid-template-columns: 2fr 1fr;
  gap: 30px;
  padding: 0 20px;
}

/* Kartochkalar va Bo'limlar */
.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 20px;
  margin-top: 20px;
}
.product-card, .cart-section {
  background: var(--card-bg);
  border: 1px solid var(--border-color);
  padding: 20px;
  border-radius: 12px;
}
.price { font-size: 1.2rem; font-weight: bold; color: #10b981; }

/* Savatcha elementlari */
.cart-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid var(--border-color);
}
.cart-footer { margin-top: 20px; padding-top: 15px; border-top: 2px solid var(--border-color); }

/* Tugmalar */
button { padding: 8px 14px; border-radius: 6px; border: none; cursor: pointer; font-weight: 500; }
.btn-primary { background: #3b82f6; color: white; width: 100%; }
.btn-sm { padding: 4px 10px; font-size: 0.85rem; }
.btn-success { background: #10b981; color: white; }
.btn-danger { background: #ef4444; color: white; }
.btn-secondary { background: #64748b; color: white; width: 100%; }
.btn-text-danger { background: transparent; color: #ef4444; padding: 0; }
Muhim texnik yechimlar:
Xavfsizlik (Tashqarida ishlash taqiqi): Har bir custom hook ichida (if (context === null)) tekshiruvi bor. Agar kimdir useCartni CartProviderdan tashqarida ishlatsa, React xatoni ko'rsatmay loyihani chalkashtirib yubormasligi uchun aniq tushunarli Error xabari otiladi.

useMemo bilan Optimizatsiya: Provider ichidagi value obyektlari useMemo bilan o'ralgan. Agar ota komponent boshqa sabab bilan qayta render bo'lsa ham, Context qiymatining xotiradagi manzili o'zgarmaydi va bolalar ortiqcha render bo'lishdan himoyalanadi.

LocalStorage Sinxronizatsiyasi: Foydalanuvchi ma'lumotlari va tanlangan sayt temasi useEffect yordamida localStorage bilan to'liq bog'langan.
