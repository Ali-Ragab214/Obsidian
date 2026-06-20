 هبعت الهيكل الأساسي وبعدين كل فايل:

---

## Structure

```
src/
├── api/
│   └── axios.js
├── store/
│   └── cartStore.js
├── layouts/
│   └── MainLayout.jsx
├── pages/
│   ├── Home/index.jsx
│   ├── Products/index.jsx
│   ├── ProductDetails/index.jsx
│   ├── Cart/index.jsx
│   └── Checkout/index.jsx
├── components/
│   ├── ProductCard.jsx
│   └── Navbar.jsx
└── router/
    └── index.jsx
```

---

## api/axios.js

```js
import axios from 'axios';

const api = axios.create({
    baseURL: 'https://fakestoreapi.com',
});

export default api;
```

---

## store/cartStore.js

```js
import { create } from 'zustand';

const useCartStore = create((set) => ({
    items: [],

    addItem: (product) => set((state) => {
        const exists = state.items.find(i => i.id === product.id);
        if (exists) {
            return {
                items: state.items.map(i =>
                    i.id === product.id ? { ...i, qty: i.qty + 1 } : i
                )
            };
        }
        return { items: [...state.items, { ...product, qty: 1 }] };
    }),

    removeItem: (id) => set((state) => ({
        items: state.items.filter(i => i.id !== id)
    })),

    clearCart: () => set({ items: [] }),

    total: (state) => state.items.reduce((acc, i) => acc + i.price * i.qty, 0),
}));

export default useCartStore;
```

---

## layouts/MainLayout.jsx

```jsx
import { Outlet } from 'react-router-dom';
import Navbar from '../components/Navbar';

export default function MainLayout() {
    return (
        <>
            <Navbar />
            <main style={{ padding: '20px' }}>
                <Outlet />
            </main>
        </>
    );
}
```

---

## components/Navbar.jsx

```jsx
import { Link } from 'react-router-dom';
import useCartStore from '../store/cartStore';

export default function Navbar() {
    const items = useCartStore(state => state.items);

    return (
        <nav style={{ display: 'flex', gap: '20px', padding: '10px', background: '#333', color: '#fff' }}>
            <Link to="/" style={{ color: '#fff' }}>Home</Link>
            <Link to="/products" style={{ color: '#fff' }}>Products</Link>
            <Link to="/cart" style={{ color: '#fff' }}>
                Cart ({items.length})
            </Link>
        </nav>
    );
}
```

---

## components/ProductCard.jsx

```jsx
import { Link } from 'react-router-dom';
import useCartStore from '../store/cartStore';

export default function ProductCard({ product }) {
    const addItem = useCartStore(state => state.addItem);

    return (
        <div style={{ border: '1px solid #ddd', padding: '10px', borderRadius: '8px' }}>
            <img src={product.image} alt={product.title} style={{ width: '100px' }} />
            <h4>{product.title.slice(0, 30)}...</h4>
            <p>${product.price}</p>
            <Link to={`/products/${product.id}`}>Details</Link>
            <button onClick={() => addItem(product)}>Add to Cart</button>
        </div>
    );
}
```

---

## pages/Home/index.jsx

```jsx
import { useState, useEffect } from 'react';
import api from '../../api/axios';
import ProductCard from '../../components/ProductCard';

export default function Home() {
    const [featured, setFeatured] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        api.get('/products?limit=4')
            .then(res => setFeatured(res.data))
            .finally(() => setLoading(false));
    }, []);

    if (loading) return <p>Loading...</p>;

    return (
        <div>
            <h1>Welcome to Shop</h1>
            <div style={{ display: 'grid', gridTemplateColumns: 'repeat(4, 1fr)', gap: '20px' }}>
                {featured.map(p => <ProductCard key={p.id} product={p} />)}
            </div>
        </div>
    );
}
```

---

## pages/Products/index.jsx

```jsx
import { useState, useEffect } from 'react';
import api from '../../api/axios';
import ProductCard from '../../components/ProductCard';

export default function Products() {
    const [products, setProducts] = useState([]);
    const [loading, setLoading] = useState(true);

    useEffect(() => {
        api.get('/products')
            .then(res => setProducts(res.data))
            .finally(() => setLoading(false));
    }, []);

    if (loading) return <p>Loading...</p>;

    return (
        <div>
            <h1>All Products</h1>
            <div style={{ display: 'grid', gridTemplateColumns: 'repeat(4, 1fr)', gap: '20px' }}>
                {products.map(p => <ProductCard key={p.id} product={p} />)}
            </div>
        </div>
    );
}
```

---

## pages/ProductDetails/index.jsx

```jsx
import { useState, useEffect } from 'react';
import { useParams } from 'react-router-dom';
import api from '../../api/axios';
import useCartStore from '../../store/cartStore';

export default function ProductDetails() {
    const { id } = useParams();
    const [product, setProduct] = useState(null);
    const [loading, setLoading] = useState(true);
    const addItem = useCartStore(state => state.addItem);

    useEffect(() => {
        api.get(`/products/${id}`)
            .then(res => setProduct(res.data))
            .finally(() => setLoading(false));
    }, [id]);

    if (loading) return <p>Loading...</p>;

    return (
        <div>
            <img src={product.image} alt={product.title} style={{ width: '200px' }} />
            <h2>{product.title}</h2>
            <p>{product.description}</p>
            <p>${product.price}</p>
            <button onClick={() => addItem(product)}>Add to Cart</button>
        </div>
    );
}
```

---

## pages/Cart/index.jsx

```jsx
import { Link } from 'react-router-dom';
import useCartStore from '../../store/cartStore';

export default function Cart() {
    const { items, removeItem } = useCartStore();
    const total = items.reduce((acc, i) => acc + i.price * i.qty, 0);

    if (items.length === 0) return <p>Cart is empty</p>;

    return (
        <div>
            <h1>Cart</h1>
            {items.map(item => (
                <div key={item.id} style={{ display: 'flex', gap: '20px', marginBottom: '10px' }}>
                    <img src={item.image} alt={item.title} style={{ width: '60px' }} />
                    <p>{item.title.slice(0, 30)}...</p>
                    <p>x{item.qty}</p>
                    <p>${(item.price * item.qty).toFixed(2)}</p>
                    <button onClick={() => removeItem(item.id)}>Remove</button>
                </div>
            ))}
            <h3>Total: ${total.toFixed(2)}</h3>
            <Link to="/checkout">
                <button>Checkout</button>
            </Link>
        </div>
    );
}
```

---

## pages/Checkout/index.jsx

```jsx
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import useCartStore from '../../store/cartStore';

export default function Checkout() {
    const { items, clearCart } = useCartStore();
    const navigate = useNavigate();
    const [form, setForm] = useState({ name: '', address: '', card: '' });
    const total = items.reduce((acc, i) => acc + i.price * i.qty, 0);

    const handleSubmit = () => {
        if (!form.name || !form.address || !form.card) return alert('Fill all fields');
        clearCart();
        alert('Order placed successfully!');
        navigate('/');
    };

    return (
        <div>
            <h1>Checkout</h1>
            <input placeholder="Name" onChange={e => setForm({ ...form, name: e.target.value })} />
            <input placeholder="Address" onChange={e => setForm({ ...form, address: e.target.value })} />
            <input placeholder="Card Number" onChange={e => setForm({ ...form, card: e.target.value })} />
            <h3>Total: ${total.toFixed(2)}</h3>
            <button onClick={handleSubmit}>Place Order</button>
        </div>
    );
}
```

---

## router/index.jsx

```jsx
import { createBrowserRouter, lazy, Suspense } from 'react-router-dom';
import MainLayout from '../layouts/MainLayout';

const Home = lazy(() => import('../pages/Home'));
const Products = lazy(() => import('../pages/Products'));
const ProductDetails = lazy(() => import('../pages/ProductDetails'));
const Cart = lazy(() => import('../pages/Cart'));
const Checkout = lazy(() => import('../pages/Checkout'));

export const router = createBrowserRouter([
    {
        element: <MainLayout />,
        children: [
            { path: '/', element: <Suspense fallback={<p>Loading...</p>}><Home /></Suspense> },
            { path: '/products', element: <Suspense fallback={<p>Loading...</p>}><Products /></Suspense> },
            { path: '/products/:id', element: <Suspense fallback={<p>Loading...</p>}><ProductDetails /></Suspense> },
            { path: '/cart', element: <Suspense fallback={<p>Loading...</p>}><Cart /></Suspense> },
            { path: '/checkout', element: <Suspense fallback={<p>Loading...</p>}><Checkout /></Suspense> },
        ]
    }
]);
```

---

## main.jsx

```jsx
import { RouterProvider } from 'react-router-dom';
import { router } from './router';
import ReactDOM from 'react-dom/client';

ReactDOM.createRoot(document.getElementById('root')).render(
    <RouterProvider router={router} />
);
```