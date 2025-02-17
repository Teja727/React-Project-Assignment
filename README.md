import React, { useEffect, useState } from "react";
import { BrowserRouter as Router, Routes, Route, useNavigate } from "react-router-dom";
import { Provider } from "react-redux";
import { useSpring, animated } from "react-spring";
import { GoogleAuthProvider, signInWithPopup, onAuthStateChanged, signOut } from "firebase/auth";
import { auth } from "./firebaseConfig";
import { Button, TextField, Container, Typography, Paper } from "@mui/material";
import store from "./store";
import "./App.css";

// Google Authentication Component
function Auth() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    onAuthStateChanged(auth, (currentUser) => {
      setUser(currentUser);
    });
  }, []);

  const handleSignIn = async () => {
    const provider = new GoogleAuthProvider();
    try {
      await signInWithPopup(auth, provider);
    } catch (error) {
      console.error("Error signing in", error);
    }
  };

  const handleSignOut = async () => {
    await signOut(auth);
  };

  return (
    <Container>
      {user ? (
        <>
          <Typography variant="h6">Welcome, {user.displayName}</Typography>
          <Button variant="contained" color="secondary" onClick={handleSignOut}>Sign Out</Button>
        </>
      ) : (
        <Button variant="contained" color="primary" onClick={handleSignIn}>Sign In with Google</Button>
      )}
    </Container>
  );
}

// Counter Component with Bezier curve transition and persistence
function Counter() {
  const [count, setCount] = useState(() => Number(localStorage.getItem("counterValue")) || 0);
  
  useEffect(() => {
    localStorage.setItem("counterValue", count);
  }, [count]);

  const backgroundStyle = useSpring({
    backgroundColor: `rgba(0, 150, 255, ${Math.pow(count, 0.5) * 0.1})`,
    config: { tension: 150, friction: 20 },
  });

  return (
    <animated.div style={{ ...backgroundStyle, padding: "20px", textAlign: "center", borderRadius: "10px" }}>
      <Typography variant="h4">Counter: {count}</Typography>
      <Button variant="contained" color="primary" onClick={() => setCount(count + 1)}>Increment</Button>
      <Button variant="contained" color="secondary" onClick={() => setCount(count - 1)}>Decrement</Button>
      <Button variant="contained" color="error" onClick={() => setCount(0)}>Reset</Button>
    </animated.div>
  );
}

// User Data Form with auto-generated ID, validation, and error handling
function UserForm() {
  const [formData, setFormData] = useState(() => JSON.parse(localStorage.getItem("userFormData")) || { id: Date.now(), name: "", address: "", email: "", phone: "" });
  const [isDirty, setIsDirty] = useState(false);
  const navigate = useNavigate();

  useEffect(() => {
    const handleUnload = (e) => {
      if (isDirty) {
        e.preventDefault();
        e.returnValue = "You have unsaved changes!";
      }
    };
    window.addEventListener("beforeunload", handleUnload);
    return () => window.removeEventListener("beforeunload", handleUnload);
  }, [isDirty]);

  const handleChange = (e) => {
    setIsDirty(true);
    setFormData({ ...formData, [e.target.name]: e.target.value });
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    localStorage.setItem("userFormData", JSON.stringify(formData));
    setIsDirty(false);
    alert("User Data Saved!");
    navigate("/dashboard");
  };

  return (
    <Container>
      <Paper elevation={3} style={{ padding: "20px" }}>
        <Typography variant="h5">User Form</Typography>
        <form onSubmit={handleSubmit}>
          <TextField label="Name" name="name" value={formData.name} onChange={handleChange} fullWidth required margin="normal" />
          <TextField label="Address" name="address" value={formData.address} onChange={handleChange} fullWidth required margin="normal" />
          <TextField label="Email" name="email" value={formData.email} onChange={handleChange} type="email" fullWidth required margin="normal" />
          <TextField label="Phone" name="phone" value={formData.phone} onChange={handleChange} fullWidth required margin="normal" />
          <Button type="submit" variant="contained" color="primary">Save</Button>
        </form>
      </Paper>
    </Container>
  );
}

function App() {
  return (
    <Provider store={store}>
      <Router>
        <Auth />
        <Routes>
          <Route path="/" element={<Counter />} />
          <Route path="/form" element={<UserForm />} />
          <Route path="/editor" element={<RichTextEditor />} />
          <Route path="/dashboard" element={<Dashboard />} />
        </Routes>
      </Router>
    </Provider>
  );
}

export default App;
