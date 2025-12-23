import React, { createContext, useContext, useReducer, useEffect } from "react";
import { BrowserRouter, Routes, Route, Link, useParams } from "react-router-dom";

/* ---------------- PRODUCT DATA ---------------- */
const products = [
  {
    id: 1,
    title: "iPhone 15",
    price: 79999,
    rating: 4.6,
    image: "https://via.placeholder.com/200",
    description: "Latest Apple iPhone with powerful performance."
  },
  {
    id: 2,
    title: "Samsung Galaxy S24",
    price: 69999,
    rating: 4.5,
    image: "https://via.placeholder.com/200",
    description: "Flagship Samsung phone with amazing display."
  },
  {
    id: 3,
    title: "Sony Headphones",
    price: 19999,
    rating: 4.4,
    image: "https://via.placeholder.com/200",
    description: "Noise cancelling premium headphones."
  }
];

/* ---------------- CART CONTEXT ---------------- */
const CartContext = createContext();

const cartReducer = (state, action) => {
  switch (action.type) {
    case "ADD":
      const exists = state.find(i => i.id === action.product.id);
      if (exists) {
        return state.map(i =>
          i.id === action.product.id
            ? { ...i, qty: i.qty + 1 }
            : i
        );
      }
      return [...state, { ...action.product, qty: 1 }];

    case "REMOVE":
      return state.filter(i => i.id !== action.id);

    case "INC":
      return state.map(i =>
        i.id === action.id ? { ...i, qty: i.qty + 1 } : i
      );

    case "DEC":
      return state.map(i =>
        i.id === action.id && i.qty > 1
          ? { ...i, qty: i.qty - 1 }
          : i
      );

    case "LOAD":
      return action.data;

    default:
      return state;
  }
};

const CartProvider = ({ children }) => {
  const [cart, dispatch] = useReducer(cartReducer, []);

  useEffect(() => {
    const saved = JSON.parse(localStorage.getItem("cart"));
    if (saved) dispatch({ type: "LOAD", data: saved });
  }, []);

  useEffect(() => {
    localStorage.setItem("cart", JSON.stringify(cart));
  }, [cart]);

  return (
    <CartContext.Provider value={{ cart, dispatch }}>
      {children}
    </CartContext.Provider>
  );
};

const useCart = () => useContext(CartContext);

/* ---------------- NAVBAR ---------------- */
function Navbar() {
  const { cart } = useCart();
  return (
    <nav style={styles.nav}>
      <h2>🛒 Amazon Clone</h2>
      <div>
        <Link to="/">Home</Link>
        <Link to="/cart">Cart ({cart.length})</Link>
      </div>
    </nav>
  );
}

/* ---------------- HOME ---------------- */
function Home() {
  return (
    <div style={styles.grid}>
      {products.map(p => (
        <div key={p.id} style={styles.card}>
          <img src={p.image} alt="" />
          <h3>{p.title}</h3>
          <p>₹{p.price}</p>
          <p>⭐ {p.rating}</p>
          <Link to={`/product/${p.id}`}>View</Link>
        </div>
      ))}
    </div>
  );
}

/* ---------------- PRODUCT DETAIL ---------------- */
function ProductDetail() {
  const { id } = useParams();
  const { dispatch } = useCart();
  const product = products.find(p => p.id === Number(id));

  return (
    <div style={styles.container}>
      <img src={product.image} alt="" />
      <h2>{product.title}</h2>
      <p>{product.description}</p>
      <p>₹{product.price}</p>
      <button onClick={() => dispatch({ type: "ADD", product })}>
        Add to Cart
      </button>
    </div>
  );
}

/* ---------------- CART ---------------- */
function Cart() {
  const { cart, dispatch } = useCart();
  const total = cart.reduce((sum, i) => sum + i.price * i.qty, 0);

  return (
    <div style={styles.container}>
      <h2>Your Cart</h2>

      {cart.length === 0 && <p>Cart is empty</p>}

      {cart.map(item => (
        <div key={item.id} style={styles.cartItem}>
          <h4>{item.title}</h4>
          <p>₹{item.price}</p>
          <div>
            <button onClick={() => dispatch({ type: "DEC", id: item.id })}>-</button>
            <span> {item.qty} </span>
            <button onClick={() => dispatch({ type: "INC", id: item.id })}>+</button>
          </div>
          <button onClick={() => dispatch({ type: "REMOVE", id: item.id })}>
            Remove
          </button>
        </div>
      ))}

      <h3>Total: ₹{total}</h3>
    </div>
  );
}

/* ---------------- APP ---------------- */
export default function App() {
  return (
    <CartProvider>
      <BrowserRouter>
        <Navbar />
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/product/:id" element={<ProductDetail />} />
          <Route path="/cart" element={<Cart />} />
        </Routes>
      </BrowserRouter>
    </CartProvider>
  );
}

/* ---------------- STYLES ---------------- */
const styles = {
  nav: {
    display: "flex",
    justifyContent: "space-between",
    padding: "15px",
    background: "#232f3e",
    color: "#fff"
  },
  grid: {
    display: "grid",
    gridTemplateColumns: "repeat(auto-fit,minmax(200px,1fr))",
    gap: "15px",
    padding: "20px"
  },
  card: {
    background: "#fff",
    padding: "15px",
    textAlign: "center",
    borderRadius: "8px"
  },
  container: {
    padding: "20px"
  },
  cartItem: {
    borderBottom: "1px solid #ccc",
    marginBottom: "10px",
    paddingBottom: "10px"
  }
};

