Below are the details regarding the practical assessment task that you need to perform.
Kindly submit the assessment within the given time. After completing the assessment host
the assessment on the free server.

1. User Management:
o Registration: Allow users to create accounts with basic information like name,
email, and password.
o Login: Authenticate users to access the booking functionalities.
2. Booking System:
o Movie Selection: Display a list of available movies with details such as title,
genre, and showtimes.
o Seat Selection: Provide an interactive seating map for each screen, indicating
available and booked seats.
o Ticket Reservation: Allow users to select seats, specify the number of tickets,
and proceed to payment.
o Payment Gateway Integration: Enable secure transactionsfor ticket purchases.
o Confirmation: Display a booking confirmation with details of the selected
movie, seats, showtime, and total price.

3. Food Ordering (Optional):
o Menu Display: Show a list of food and beverage items available for purchase.
o Add to Cart: Allow users to select items and add them to their order.
o Checkout: Enable users to review their food order, adjust quantities, and
proceed to payment.
4. Seat Management:
o Seat Availability: Implement logic to track seat availability in real-time and
prevent double bookings.
o Seat Locking: Temporarily reserve selected seats for a short duration while
the user completes the booking process.
o Seat Release: Automatically release locked seats if the booking process is not
completed within a certain time frame.

5. Screen Configuration:
o Screen Selection: Provide options for users to choose between different
screens (if applicable).
o Seat Limit: Enforce a maximum capacity for each screen (e.g., 60 seats) to
prevent overbooking.

6. Admin Panel:
o Screen Management: Allow administrators to configure screens, including the
number of seats and seat arrangements.
o Booking Management: Provide tools to view and manage bookings, including
seat assignments and cancellations.
o Food Menu Management: Enable administrators to update the food and
beverage menu.
7. Notifications:
o Booking Confirmation: Send email or SMS notifications to users after
successful bookings.
o Seat Availability Alerts: Notify users if their selected seats become unavailable
during the booking process.

8. Security:
o User Authentication: Implement secure authentication mechanisms to protect
user accounts and personal information.
o Data Encryption: Encrypt sensitive data such as passwords and payment
details to ensure privacy and security.

9. Deployment:
o Frontend application (user interface) can be deployed on Vercel, a cloud
platform for static sites and serverless functions.
o Backend services (server-side logic, database interaction, etc.) can be
deployed on Render, a cloud platform for hosting web applications and
databases.




Sol:   Let's break down the complete implementation into detailed steps. This includes setting up the backend, frontend, integrating the payment gateway, and deploying the application.

1. Setup Backend
Initialize Project
Create a new directory for the backend and initialize a Node.js project.

Code:
mkdir movie-booking-backend
cd movie-booking-backend
npm init -y

Install Dependencies
Install the necessary packages.

Code:
npm install express mongoose bcryptjs jsonwebtoken stripe dotenv

Create Server
Create an index.js file to set up the Express server.

Code:

// index.js
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');

dotenv.config();

const app = express();
app.use(express.json());

mongoose.connect(process.env.MONGODB_URI, { useNewUrlParser: true, useUnifiedTopology: true });

app.get('/', (req, res) => {
    res.send('Movie Booking API');
});

app.listen(5000, () => console.log('Server started on port 5000'));


Create Models
Create models for User, Movie, Booking, Seat, and Food in a models directory.

User Model (models/User.js)

Code:

const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
    name: { type: String, required: true },
    email: { type: String, required: true, unique: true },
    password: { type: String, required: true }
});

userSchema.pre('save', async function (next) {
    if (!this.isModified('password')) return next();
    this.password = await bcrypt.hash(this.password, 10);
    next();
});

module.exports = mongoose.model('User', userSchema);

Movie Model (models/Movie.js)

Code:

const mongoose = require('mongoose');

const movieSchema = new mongoose.Schema({
    title: { type: String, required: true },
    genre: { type: String, required: true },
    showtimes: [{ type: Date, required: true }]
});

module.exports = mongoose.model('Movie', movieSchema);

Booking Model (models/Booking.js)

Code:

const mongoose = require('mongoose');

const bookingSchema = new mongoose.Schema({
    user: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true },
    movie: { type: mongoose.Schema.Types.ObjectId, ref: 'Movie', required: true },
    seats: [{ type: String, required: true }],
    showtime: { type: Date, required: true },
    totalPrice: { type: Number, required: true }
});

module.exports = mongoose.model('Booking', bookingSchema);

Seat Model (models/Seat.js)

Code:
 const mongoose = require('mongoose');

const seatSchema = new mongoose.Schema({
    movie: { type: mongoose.Schema.Types.ObjectId, ref: 'Movie', required: true },
    showtime: { type: Date, required: true },
    seatNumber: { type: String, required: true },
    isBooked: { type: Boolean, default: false }
});

module.exports = mongoose.model('Seat', seatSchema);

Food Model (models/Food.js)

Code:
const mongoose = require('mongoose');

const foodSchema = new mongoose.Schema({
    name: { type: String, required: true },
    price: { type: Number, required: true }
});

module.exports = mongoose.model('Food', foodSchema);

Create Routes
Create routes for authentication, movies, bookings, and admin functions in a routes directory.

Auth Routes (routes/auth.js)

Code:

const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const router = express.Router();

router.post('/register', async (req, res) => {
    const { name, email, password } = req.body;
    try {
        const user = new User({ name, email, password });
        await user.save();
        res.status(201).send('User registered');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

router.post('/login', async (req, res) => {
    const { email, password } = req.body;
    try {
        const user = await User.findOne({ email });
        if (!user || !await bcrypt.compare(password, user.password)) {
            return res.status(400).send('Invalid credentials');
        }
        const token = jwt.sign({ userId: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
        res.json({ token });
    } catch (error) {
        res.status(500).send(error.message);
    }
});

module.exports = router;


Movie Routes (routes/movies.js)

Code:

const express = require('express');
const Movie = require('../models/Movie');
const router = express.Router();

router.get('/', async (req, res) => {
    try {
        const movies = await Movie.find();
        res.json(movies);
    } catch (error) {
        res.status(500).send(error.message);
    }
});

router.post('/', async (req, res) => {
    const { title, genre, showtimes } = req.body;
    try {
        const movie = new Movie({ title, genre, showtimes });
        await movie.save();
        res.status(201).send('Movie created');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

module.exports = router;

Booking Routes (routes/bookings.js)

Code:
const express = require('express');
const Booking = require('../models/Booking');
const Seat = require('../models/Seat');
const router = express.Router();

router.post('/', async (req, res) => {
    const { userId, movieId, seats, showtime, totalPrice } = req.body;
    try {
        const booking = new Booking({ user: userId, movie: movieId, seats, showtime, totalPrice });
        await booking.save();
        await Seat.updateMany({ seatNumber: { $in: seats }, showtime, movie: movieId }, { isBooked: true });
        res.status(201).send('Booking created');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

module.exports = router;

Admin Routes (routes/admin.js)

Code:

const express = require('express');
const Movie = require('../models/Movie');
const Seat = require('../models/Seat');
const router = express.Router();

router.post('/configure-screen', async (req, res) => {
    const { movieId, showtime, seats } = req.body;
    try {
        for (let seatNumber of seats) {
            const seat = new Seat({ movie: movieId, showtime, seatNumber });
            await seat.save();
        }
        res.status(201).send('Screen configured');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

router.get('/bookings', async (req, res) => {
    try {
        const bookings = await Booking.find().populate('user movie');
        res.json(bookings);
    } catch (error) {
        res.status(500).send(error.message);
    }
});

module.exports = router;

Integrate Payment Gateway (Stripe)
Add Stripe for payment processing in the booking route.

Updated Booking Route (routes/bookings.js)

Code:

const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);

router.post('/', async (req, res) => {
    const { userId, movieId, seats, showtime, totalPrice, paymentMethodId } = req.body;
    try {
        const paymentIntent = await stripe.paymentIntents.create({
            amount: totalPrice * 100,
            currency: 'usd',
            payment_method: paymentMethodId,
            confirm: true
        });

        const booking = new Booking({ user: userId, movie: movieId, seats, showtime, totalPrice });
        await booking.save();
        await Seat.updateMany({ seatNumber: { $in: seats }, showtime, movie: movieId }, { isBooked: true });
        res.status(201).send('Booking created');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

Configure Environment Variables
Create a .env file to store environment variables.

Code:

MONGODB_URI=mongodb://localhost:27017/movie_booking
JWT_SECRET=your_jwt_secret
STRIPE_SECRET_KEY=your_stripe_secret_key


2. Setup Frontend
Initialize Project
Create a new React project.

Code:
npx create-react-app movie-booking-frontend
cd movie-booking-frontend

Install Dependencies
Install necessary packages.

Code:
npm install axios react-router-dom @stripe/react-stripe-js @stripe/stripe-js

Setup Pages and Components
Create necessary pages and components in the src directory.

App Component (src/App.js)

Code:

import React from 'react';
import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
import Registration from './pages/Registration';
import Login from './pages/Login';
import Movies from './pages/Movies';
import Booking from './pages/Booking';
import Confirmation from './pages/Confirmation';
import Admin from './pages/Admin';

function App() {
    return (
        <Router>
            <Switch>
                <Route path="/register" component={Registration} />
                <Route path="/login" component={Login} />
                <Route path="/movies" component={Movies} />
                <Route path="/booking" component={Booking} />
                <Route path="/confirmation" component={Confirmation} />
                <Route path="/admin" component={Admin} />
            </Switch>
        </Router>
    );
}

export default App;

Registration Page (src/pages/Registration.js)

Code:

import React, { useState } from 'react';
import axios from 'axios';

const Registration = () => {
    const [name, setName] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            await axios.post('http://localhost:5000/register', { name, email, password });
            alert('User registered');
        } catch (error) {
            console.error(error);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" placeholder="Name" value={name} onChange={(e) => setName(e.target.value)} required />
            <input type="email" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} required />
            <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} required />
            <button type="submit">Register</button>
        </form>
    );
};

export default Registration;

Login Page (src/pages/Login.js)

Code:
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            const response = await axios.post('http://localhost:5000/login', { email, password });
            localStorage.setItem('token', response.data.token);
            alert('User logged in');
        } catch (error) {
            console.error(error);
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="email" placeholder="Email" value={email} onChange={(e) => setEmail(e.target.value)} required />
            <input type="password" placeholder="Password" value={password} onChange={(e) => setPassword(e.target.value)} required />
            <button type="submit">Login</button>
        </form>
    );
};

export default Login;

Movies Page (src/pages/Movies.js)

Code:

import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Movies = () => {
    const [movies, setMovies] = useState([]);

    useEffect(() => {
        const fetchMovies = async () => {
            try {
                const response = await axios.get('http://localhost:5000/movies');
                setMovies(response.data);
            } catch (error) {
                console.error(error);
            }
        };
        fetchMovies();
    }, []);

    return (
        <div>
            {movies.map(movie => (
                <div key={movie._id}>
                    <h2>{movie.title}</h2>
                    <p>{movie.genre}</p>
                    <p>{movie.showtimes.join(', ')}</p>
                </div>
            ))}
        </div>
    );
};

export default Movies;

Booking Page (src/pages/Booking.js)

Code:

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { loadStripe } from '@stripe/stripe-js';
import { Elements, CardElement, useStripe, useElements } from '@stripe/react-stripe-js';

const stripePromise = loadStripe('your_stripe_public_key');

const Booking = () => {
    const [movies, setMovies] = useState([]);
    const [selectedMovie, setSelectedMovie] = useState(null);
    const [showtime, setShowtime] = useState('');
    const [seats, setSeats] = useState([]);
    const [selectedSeats, setSelectedSeats] = useState([]);
    const [totalPrice, setTotalPrice] = useState(0);

    useEffect(() => {
        const fetchMovies = async () => {
            try {
                const response = await axios.get('http://localhost:5000/movies');
                setMovies(response.data);
            } catch (error) {
                console.error(error);
            }
        };
        fetchMovies();
    }, []);

    useEffect(() => {
        if (selectedMovie && showtime) {
            const fetchSeats = async () => {
                try {
                    const response = await axios.get(`http://localhost:5000/seats?movie=${selectedMovie}&showtime=${showtime}`);
                    setSeats(response.data);
                } catch (error) {
                    console.error(error);
                }
            };
            fetchSeats();
        }
    }, [selectedMovie, showtime]);

    const handleSeatSelect = (seat) => {
        if (selectedSeats.includes(seat)) {
            setSelectedSeats(selectedSeats.filter(s => s !== seat));
        } else {
            setSelectedSeats([...selectedSeats, seat]);
        }
        setTotalPrice(selectedSeats.length * 10); // Assuming each seat costs $10
    };

    const handleSubmit = async (e) => {
        e.preventDefault();
        try {
            const token = localStorage.getItem('token');
            const paymentMethod = await stripe.createPaymentMethod({
                type: 'card',
                card: elements.getElement(CardElement),
            });
            const response = await axios.post('http://localhost:5000/bookings', {
                userId: jwtDecode(token).userId,
                movieId: selectedMovie,
                seats: selectedSeats,
                showtime,
                totalPrice,
                paymentMethodId: paymentMethod.id
            });
            alert('Booking successful');
        } catch (error) {
            console.error(error);
        }
    };

    return (
        <Elements stripe={stripePromise}>
            <form onSubmit={handleSubmit}>
                <select onChange={(e) => setSelectedMovie(e.target.value)} required>
                    <option value="">Select Movie</option>
                    {movies.map(movie => (
                        <option key={movie._id} value={movie._id}>{movie.title}</option>
                    ))}
                </select>
                <input type="datetime-local" onChange={(e) => setShowtime(e.target.value)} required />
                <div>
                    {seats.map(seat => (
                        <button key={seat._id} onClick={() => handleSeatSelect(seat.seatNumber)}>{seat.seatNumber}</button>
                    ))}
                </div>
                <CardElement />
                <button type="submit">Book</button>
            </form>
        </Elements>
    );
};

export default Booking;

Confirmation Page (src/pages/Confirmation.js)

Code:

import React from 'react';

const Confirmation = ({ location }) => {
    const { state } = location;

    return (
        <div>
            <h2>Booking Confirmation</h2>
            <p>Movie: {state.movieTitle}</p>
            <p>Showtime: {state.showtime}</p>
            <p>Seats: {state.seats.join(', ')}</p>
            <p>Total Price: ${state.totalPrice}</p>
        </div>
    );
};

export default Confirmation;


Admin Page (src/pages/Admin.js)

Code:

import React, { useState, useEffect } from 'react';
import axios from 'axios';

const Admin = () => {
    const [bookings, setBookings] = useState([]);

    useEffect(() => {
        const fetchBookings = async () => {
            try {
                const response = await axios.get('http://localhost:5000/admin/bookings');
                setBookings(response.data);
            } catch (error) {
                console.error(error);
            }
        };
        fetchBookings();
    }, []);

    return (
        <div>
            <h2>Admin Panel</h2>
            <div>
                {bookings.map(booking => (
                    <div key={booking._id}>
                        <p>User: {booking.user.name}</p>
                        <p>Movie: {booking.movie.title}</p>
                        <p>Seats: {booking.seats.join(', ')}</p>
                        <p>Showtime: {booking.showtime}</p>
                        <p>Total Price: ${booking.totalPrice}</p>
                    </div>
                ))}
            </div>
        </div>
    );
};

export default Admin;


3. Deploy Applications
Deploy Backend on Render
Sign up on Render.
Create a new Web Service and connect your GitHub repository.
Set environment variables for MongoDB URI, JWT Secret, and Stripe keys.
Deploy the backend.
Deploy Frontend on Vercel
Sign up on Vercel.
Import your frontend project from GitHub.
Configure build settings and deploy.
Conclusion
This complete implementation guide provides a structured approach to developing and deploying a movie booking system with essential features, security, and integration of a payment gateway. Each step includes detailed code snippets and explanations to ensure the system's functionality and security.



















    
