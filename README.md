# movie-lobby-api
A backend API for a movie lobby built with Node.js, TypeScript, and MongoDB.
Movie Lobby API

A backend API for a movie lobby in OTT applications, built with Node.js, TypeScript, and MongoDB.

Features

List all movies.

Search movies by title or genre.

Add, update, or delete movies (admin-only functionality).

Use caching to enhance performance (optional).

Prerequisites

Ensure you have the following installed:

Node.js and NPM

MongoDB

VS Code (or any text editor of your choice)

Setup

1. Clone the Repository

git clone https://github.com/<your-username>/movie-lobby-api.git
cd movie-lobby-api

2. Install Dependencies

npm install

3. Configure Environment Variables

Create a .env file in the root directory and add the following:

MONGO_URI=mongodb://localhost:27017/movieLobby
PORT=5000

4. Run the Application

Start the server:

npm run dev

The server will run at http://localhost:5000.

API Endpoints

1. List All Movies

Endpoint: GET /api/movies

Description: Fetch all movies in the lobby.

Sample Response:

[
  {
    "_id": "64a0bc...",
    "title": "Inception",
    "genre": "Sci-Fi",
    "rating": 9,
    "streamingLink": "http://example.com"
  }
]

2. Search Movies

Endpoint: GET /api/search?q={query}

Description: Search for movies by title or genre.

Sample Request:

GET /api/search?q=sci-fi

Sample Response:

[
  {
    "_id": "64a0bc...",
    "title": "Interstellar",
    "genre": "Sci-Fi",
    "rating": 9,
    "streamingLink": "http://example.com"
  }
]

3. Add a New Movie

Endpoint: POST /api/movies

Description: Add a new movie to the lobby (admin-only).

Headers:

{
  "role": "admin"
}

Sample Request Body:

{
  "title": "The Matrix",
  "genre": "Sci-Fi",
  "rating": 9,
  "streamingLink": "http://example.com"
}

Sample Response:

{
  "_id": "64a0bd...",
  "title": "The Matrix",
  "genre": "Sci-Fi",
  "rating": 9,
  "streamingLink": "http://example.com"
}

4. Update Movie Information

Endpoint: PUT /api/movies/:id

Description: Update an existing movie's information (admin-only).

Headers:

{
  "role": "admin"
}

Sample Request Body:

{
  "title": "The Matrix Reloaded",
  "rating": 8.5
}

Sample Response:

{
  "_id": "64a0bd...",
  "title": "The Matrix Reloaded",
  "genre": "Sci-Fi",
  "rating": 8.5,
  "streamingLink": "http://example.com"
}

5. Delete a Movie

Endpoint: DELETE /api/movies/:id

Description: Remove a movie from the lobby (admin-only).

Headers:

{
  "role": "admin"
}

Sample Response:

{
  "message": "Movie deleted"
}

Testing

Run Tests

Ensure the server is not running, then execute:

npm test

Testing Frameworks Used

Jest: Unit and integration tests.

Supertest: HTTP assertions.

Linting

Ensure your code follows best practices:

npm run lint

Deployment

To deploy the application:

Set up a production MongoDB database.

Update the .env file with the production MONGO_URI.

Use a process manager like PM2 for deployment:

npm install -g pm2
pm2 start dist/server.js --name "movie-lobby-api"

License

This project is licensed under the MIT License.

Source Code

1. Movie Model

import mongoose from 'mongoose';

const movieSchema = new mongoose.Schema({
  title: { type: String, required: true },
  genre: { type: String, required: true },
  rating: { type: Number, required: true, min: 0, max: 10 },
  streamingLink: { type: String, required: true },
});

export default mongoose.model('Movie', movieSchema);

2. Database Connection

import mongoose from 'mongoose';

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGO_URI || '', { useNewUrlParser: true, useUnifiedTopology: true });
    console.log('MongoDB Connected...');
  } catch (err) {
    console.error(err);
    process.exit(1);
  }
};

export default connectDB;

3. Auth Middleware

import { Request, Response, NextFunction } from 'express';

export const adminAuth = (req: Request, res: Response, next: NextFunction) => {
  const { role } = req.headers;
  if (role !== 'admin') {
    return res.status(403).json({ message: 'Forbidden: Admins only' });
  }
  next();
};

4. Movie Controller

import { Request, Response } from 'express';
import Movie from '../models/Movie';

// List all movies
export const getMovies = async (req: Request, res: Response) => {
  const movies = await Movie.find();
  res.json(movies);
};

// Search movies by title or genre
export const searchMovies = async (req: Request, res: Response) => {
  const { q } = req.query;
  const movies = await Movie.find({
    $or: [
      { title: { $regex: q as string, $options: 'i' } },
      { genre: { $regex: q as string, $options: 'i' } },
    ],
  });
  res.json(movies);
};

// Add a new movie
export const addMovie = async (req: Request, res: Response) => {
  const newMovie = new Movie(req.body);
  await newMovie.save();
  res.status(201).json(newMovie);
};

// Update movie information
export const updateMovie = async (req: Request, res: Response) => {
  const { id } = req.params;
  const updatedMovie = await Movie.findByIdAndUpdate(id, req.body, { new: true });
  res.json(updatedMovie);
};

// Delete a movie
export const deleteMovie = async (req: Request, res: Response) => {
  const { id } = req.params;
  await Movie.findByIdAndDelete(id);
  res.json({ message: 'Movie deleted' });
};

5. Routes

import express from 'express';
import {
  getMovies,
  searchMovies,
  addMovie,
  updateMovie,
  deleteMovie,
} from '../controllers/movieController';
import { adminAuth } from '../middleware/authMiddleware';

const router = express.Router();

router.get('/movies', getMovies);
router.get('/search', searchMovies);
router.post('/movies', adminAuth, addMovie);
router.put('/movies/:id', adminAuth, updateMovie);
router.delete('/movies/:id', adminAuth, deleteMovie);

export default router;

6. App Setup

import express from 'express';
import bodyParser from 'body-parser';
import cors from 'cors';
import movieRoutes from './routes/movieRoutes';

const app = express();
app.use(cors());
app.use(bodyParser.json());
app.use('/api', movieRoutes);

export default app;

7. Server Entry

import app from './app';
import connectDB from './utils/connectDB';

const PORT = process.env.PORT || 5000;

connectDB();
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));

8. Tests

import request from 'supertest';
import app from '../app';

describe('Movie API Tests', () => {
  it('GET /movies - should return a list of movies', async () => {
    const res = await request(app).get('/api/movies');
    expect(res.statusCode).toBe(200);
  });

  it('POST /movies - should add a new movie', async () => {
    const res = await request(app)
      .post('/api/movies')
      .set('role', 'admin')
      .send({
        title: 'Inception',
        genre: 'Sci-Fi',
        rating: 9,
        streamingLink: 'http://example.com',
      });
    expect(res.statusCode).toBe(201);
  });
});

