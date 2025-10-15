import React, { useState } from "react";
import axios from "axios";
import { useNavigate } from "react-router-dom";
import { toast } from 'react-toastify';
 
const API_BASE = "http://localhost:3000/api/v1";
 
const Signup = () => {
  const [name, setName] = useState("");
  const [role, setRole] = useState("Student");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const navigate = useNavigate();
 
  const handleSignup = async (e) => {
    e.preventDefault();

    if (!name || !email || !password || !role) {
      toast.error("All fields are required");
      return;
    }

    const studentEmailRegex = /^[^\s@]+@stu\.com$/;
    const instructorEmailRegex = /^[^\s@]+@ins\.com$/;

    if (role === "Student" && !studentEmailRegex.test(email)) {
      toast.error("Student email must end with @stu.com");
      return;
    }

    if (role === "Instructor" && !instructorEmailRegex.test(email)) {
      toast.error("Instructor email must end with @ins.com");
      return;
    }

    const passwordRegex = /^(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*()_+[\]{};':"\\|,.<>/?]).{8,}$/;
    if (!passwordRegex.test(password)) {
      toast.error("Password must be at least 8 characters long, include one uppercase letter, one number, and one special character");
      return;
    }

    try {
      const existingUser = await axios.get(`${API_BASE}/users?email=${email}`);

      const userExists = Array.isArray(existingUser.data)
        ? existingUser.data.length > 0
        : existingUser.data.users?.some((u) => u.email === email);

      if (userExists) {
        toast.error("User already exists. Please login.");
        return;
      }

      const newUser = {
        name,
        role,
        email,
        password,
        username: email, // for login compatibility
      };

      await axios.post(`${API_BASE}/users`, newUser);
      toast.success("Account created successfully!");
      navigate("/login");
    } catch (err) {
      // Handling server-side errors, e.g., if the POST itself fails due to validation/conflict
      // which wasn't caught by the GET check.
      if (err.response && err.response.data && err.response.data.message) {
        toast.error(err.response.data.message);
      } else {
        toast.error("Something went wrong. Try again.");
      }
      console.error("Signup failed:", err);
    }
  };
 
  return (
    <div className="flex justify-center items-center min-h-screen bg-gray-100">
      <form
        onSubmit={handleSignup}
        className="bg-white p-8 rounded-xl shadow-lg w-96"
      >
        <h2 className="text-2xl font-bold mb-6 text-center">Sign Up</h2>

        <input
          type="text"
          placeholder="Full Name"
          className="w-full p-3 border rounded mb-4"
          value={name}
          onChange={(e) => setName(e.target.value)}
          required
        />
 
        <select
          className="w-full p-3 border rounded mb-4"
          value={role}
          onChange={(e) => setRole(e.target.value)}
        >
          <option value="Student">Student</option>
          <option value="Instructor">Instructor</option>
        </select>
 
        <input
          type="email"
          placeholder="Email"
          className="w-full p-3 border rounded mb-4"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          required
        />
 
        <input
          type="password"
          placeholder="Password"
          className="w-full p-3 border rounded mb-4"
          value={password}
          onChange={(e) => setPassword(e.target.value)}
          required
        />
 
        <button
          type="submit"
          className="w-full bg-indigo-600 text-white p-3 rounded hover:bg-indigo-700"
        >
          Sign Up
        </button>
      </form>
    </div>
  );
};
 
export default Signup;
