import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { initializeApp } from "firebase/app";
import { getAuth, signInWithPopup, GoogleAuthProvider, FacebookAuthProvider, createUserWithEmailAndPassword, signInWithEmailAndPassword, signOut } from "firebase/auth";
import Razorpay from "razorpay";

const firebaseConfig = {
  apiKey: "YOUR_FIREBASE_API_KEY",
  authDomain: "YOUR_FIREBASE_AUTH_DOMAIN",
  projectId: "YOUR_FIREBASE_PROJECT_ID",
  storageBucket: "YOUR_FIREBASE_STORAGE_BUCKET",
  messagingSenderId: "YOUR_FIREBASE_MESSAGING_SENDER_ID",
  appId: "YOUR_FIREBASE_APP_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const googleProvider = new GoogleAuthProvider();
const facebookProvider = new FacebookAuthProvider();

export default function Home() {
  const [cart, setCart] = useState([]);
  const [wishlist, setWishlist] = useState([]);
  const [ratings, setRatings] = useState({});
  const [orders, setOrders] = useState([]);
  const [location, setLocation] = useState(null);
  const [user, setUser] = useState(null);
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [darkMode, setDarkMode] = useState(false);
  const [isPWAInstalled, setIsPWAInstalled] = useState(false);
  
  useEffect(() => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        (position) => {
          setLocation({
            latitude: position.coords.latitude,
            longitude: position.coords.longitude,
          });
        },
        (error) => {
          console.error("Error fetching location: ", error);
        }
      );
    }

    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault();
      setIsPWAInstalled(true);
    });
  }, []);

  const installPWA = () => {
    if (window.deferredPrompt) {
      window.deferredPrompt.prompt();
    }
  };

  const products = [
    { id: 1, name: "Product 1", price: 20, image: "https://via.placeholder.com/150", description: "This is an amazing product." },
    { id: 2, name: "Product 2", price: 30, image: "https://via.placeholder.com/150", description: "High quality and durable." },
  ];

  const addToCart = (product) => {
    setCart([...cart, product]);
  };

  const addToWishlist = (product) => {
    setWishlist([...wishlist, product]);
  };

  const handlePayment = async () => {
    const amount = cart.reduce((total, item) => total + item.price, 0) * 100;
    const options = {
      key: "YOUR_RAZORPAY_KEY",
      amount: amount,
      currency: "INR",
      name: "OSM COLLECTIONS",
      description: "Purchase from OSM COLLECTIONS",
      handler: (response) => {
        setOrders([...orders, ...cart]);
        setCart([]);
        alert("Payment Successful! Payment ID: " + response.razorpay_payment_id);
      },
      prefill: {
        email: user?.email || "",
      },
      theme: {
        color: "#3399cc",
      },
    };
    const rzp1 = new window.Razorpay(options);
    rzp1.open();
  };

  return (
    <div className={`p-4 ${darkMode ? "bg-gray-900 text-white" : "bg-white text-black"}`}>
      <header className="flex items-center justify-between p-4 bg-blue-600 text-white rounded-lg shadow-md">
        <h1 className="text-2xl font-bold">OSM COLLECTIONS</h1>
        <img src="https://via.placeholder.com/100x50" alt="OSM COLLECTIONS Logo" className="h-12" />
        <Button onClick={() => setDarkMode(!darkMode)}>Toggle Dark Mode</Button>
      </header>
      {isPWAInstalled && <Button onClick={installPWA}>Install OSM COLLECTIONS App</Button>}
      <div className="mt-4 grid grid-cols-2 gap-4">
        {products.map((product) => (
          <div key={product.id} className="border p-4 rounded-lg shadow-lg">
            <img src={product.image} alt={product.name} className="w-full mb-2" />
            <h2 className="text-lg font-semibold">{product.name}</h2>
            <p className="text-gray-700">₹{product.price}</p>
            <p className="text-sm text-gray-500">{product.description}</p>
            <Button onClick={() => addToCart(product)} className="mt-2">Add to Cart</Button>
            <Button onClick={() => addToWishlist(product)} className="mt-2 ml-2">Add to Wishlist</Button>
          </div>
        ))}
      </div>
      <div className="mt-6 p-4 border rounded-lg">
        <h2 className="text-xl font-bold">Cart ({cart.length} items)</h2>
        {cart.length > 0 ? (
          cart.map((item, index) => (
            <p key={index}>{item.name} - ₹{item.price}</p>
          ))
        ) : (
          <p className="text-gray-500">Your cart is empty.</p>
        )}
        {cart.length > 0 && (
          <Button onClick={handlePayment} className="mt-4">Proceed to Payment</Button>
        )}
      </div>
      <div className="mt-6 p-4 border rounded-lg">
        <h2 className="text-xl font-bold">Order History</h2>
        {orders.length > 0 ? (
          orders.map((item, index) => (
            <p key={index}>{item.name} - ₹{item.price}</p>
          ))
        ) : (
          <p className="text-gray-500">No orders placed yet.</p>
        )}
      </div>
    </div>
  );
}
