# Harvie.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Safe Space PH - Mental Wellness Prototype (Community Enabled)</title>
    <!-- Load Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Load Firebase libraries -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, collection, query, orderBy, onSnapshot, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Expose Firebase modules globally for use in the main script block
        window.firebase = { 
            initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, 
            getFirestore, doc, addDoc, collection, query, orderBy, onSnapshot, setLogLevel, serverTimestamp
        };
    </script>
    <style>
        /* Custom font and base styles */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }

        /* Breathing Timer CSS */
        .breathing-circle {
            width: 150px;
            height: 150px;
            background-color: #4DB6AC; /* calm-teal */
            border-radius: 50%;
            transition: transform 4s ease-in-out;
            box-shadow: 0 0 20px rgba(77, 182, 172, 0.7);
        }

        .breathe-in {
            transform: scale(1.5);
        }
        .breathe-out {
            transform: scale(1.0);
        }
    </style>
    <script>
        // Customizing the Tailwind theme for soft, mental-wellness colors
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        'soft-blue': '#E0F7FA',  /* Light Cyan/Sky Blue for background */
                        'calm-teal': '#4DB6AC',  /* Medium Teal for accents/primary button */
                        'dark-teal': '#004D40',  /* Deep Teal for text/hover state */
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-soft-blue min-h-screen flex flex-col">

    <!-- Main Application Container -->
    <div id="app" class="flex-grow flex flex-col items-center justify-start p-4">
        <header class="w-full max-w-4xl text-center py-4 sm:py-6 sticky top-0 bg-white/90 backdrop-blur-sm z-10 rounded-b-xl shadow-md mb-4">
            <h1 class="text-4xl font-extrabold text-calm-teal">Safe Space <span class="text-dark-teal">PH</span></h1>
        </header>

        <!-- Dynamic Content Area -->
        <main id="content" class="w-full max-w-xl bg-white shadow-2xl rounded-3xl p-6 sm:p-8 flex-grow mb-8 transition-all duration-300"></main>

        <!-- Navigation Bar -->
        <nav class="w-full max-w-xl bg-white rounded-full p-2 shadow-xl sticky bottom-4 z-20">
            <ul class="flex justify-around text-xs sm:text-sm font-medium">
                <li class="nav-item">
                    <button onclick="window.app.navigateTo('home')" class="p-3 rounded-full flex flex-col items-center transition-colors duration-200 hover:bg-soft-blue text-dark-teal">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" /></svg>
                        Home
                    </button>
                </li>
                <li class="nav-item">
                    <button onclick="window.app.navigateTo('journal')" class="p-3 rounded-full flex flex-col items-center transition-colors duration-200 hover:bg-soft-blue text-dark-teal">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" /></svg>
                        Community
                    </button>
                </li>
                <li class="nav-item">
                    <button onclick="window.app.navigateTo('mood')" class="p-3 rounded-full flex flex-col items-center transition-colors duration-200 hover:bg-soft-blue text-dark-teal">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14.828 14.828a4 4 0 01-5.656 0M9 10h.01M15 10h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                        Mood
                    </button>
                </li>
                <li class="nav-item">
                    <button onclick="window.app.navigateTo('tools')" class="p-3 rounded-full flex flex-col items-center transition-colors duration-200 hover:bg-soft-blue text-dark-teal">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.75 17L19.5 7.25M17 19.5L7.25 9.75" /></svg>
                        Tools
                    </button>
                </li>
                <li class="nav-item">
                    <button onclick="window.app.navigateTo('resources')" class="p-3 rounded-full flex flex-col items-center transition-colors duration-200 hover:bg-soft-blue text-dark-teal">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.409 9.333 5 8 5a2 2 0 00-2 2v10a2 2 0 002 2c1.638 0 3.328-0.9 4-2M12 6.253c1.168-0.844 2.667-1.253 4-1.253a2 2 0 012 2v10a2 2 0 01-2 2c-1.638 0-3.328-0.9-4-2" /></svg>
                        Resources
                    </button>
                </li>
            </ul>
        </nav>
        
        <!-- Status Message Box -->
        <div id="status-message" class="fixed bottom-24 p-4 rounded-xl shadow-2xl bg-dark-teal text-white hidden transition-opacity duration-300"></div>

    </div>

    <script type="module">
        // Ensure Firebase modules are loaded via the script block above
        if (typeof window.firebase === 'undefined') {
            console.error("Firebase modules failed to load.");
        } else {
            // Main application logic is defined here
            
            const { 
                initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, 
                getFirestore, doc, addDoc, collection, query, orderBy, onSnapshot, setLogLevel, serverTimestamp
            } = window.firebase;

            // Global State and Firebase variables
            let db;
            let auth;
            let userId = 'loading...';
            let anonymousName = 'Loading User';
            let currentPage = 'home';
            let breathingTimer = null;
            let currentAffirmationIndex = 0;
            let publicEntries = [];
            let commentsCache = {}; // Cache to hold comments: { entryId: [{...comment...}] }
            let entrySnapshotUnsubscribe = null;
            let commentUnsubscribes = {};

            const contentEl = document.getElementById('content');
            const statusEl = document.getElementById('status-message');

            const AFFIRMATIONS = [
                "I am deserving of peace and happiness.",
                "My feelings are valid, and I am safe to express them.",
                "I am capable of handling today's challenges.",
                "I choose to be kind to myself and others.",
                "This moment is temporary, and I will get through it.",
                "I am enough."
            ];

            // --- Utility Functions ---

            /** Generates a consistent anonymous name based on the userId */
            const getAnonymousName = (id) => {
                if (!id || id === 'loading...') return 'Loading User';
                const themes = ["Star", "Moon", "Sun", "Cloud", "Wave", "Stone", "Tree", "Flower", "Bird", "Deer"];
                // Use a numerical representation of the last few chars for deterministic naming
                const numericPart = id.slice(-4).split('').map(c => c.charCodeAt(0)).join('');
                const number = parseInt(numericPart.slice(-4) || '0', 10);
                const themeIndex = number % themes.length;
                return `${themes[themeIndex]} #${number}`;
            };

            /** Formats timestamp to a readable string */
            const formatTimestamp = (timestamp) => {
                if (!timestamp) return 'Just now';
                if (timestamp.toDate) return timestamp.toDate().toLocaleString();
                return new Date(timestamp).toLocaleString();
            };

            /** Displays a temporary status message */
            const showStatus = (message, duration = 3000) => {
                statusEl.textContent = message;
                statusEl.classList.remove('hidden', 'opacity-0');
                statusEl.classList.add('opacity-100');
                
                setTimeout(() => {
                    statusEl.classList.remove('opacity-100');
                    statusEl.classList.add('opacity-0');
                    setTimeout(() => statusEl.classList.add('hidden'), 500);
                }, duration);
            };

            // --- Firebase Initialization and Auth ---

            async function initFirebase() {
                try {
                    setLogLevel('Debug');
                    const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                    
                    if (!firebaseConfig || !firebaseConfig.apiKey) {
                        console.warn("Firebase config is missing. Journal saving will not work.");
                        db = null;
                        return;
                    }

                    const app = initializeApp(firebaseConfig);
                    auth = getAuth(app);
                    db = getFirestore(app);

                    // Sign in using the custom token or anonymously
                    const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

                    if (initialAuthToken) {
                        await signInWithCustomToken(auth, initialAuthToken);
                    } else {
                        await signInAnonymously(auth);
                    }

                    onAuthStateChanged(auth, (user) => {
                        if (user) {
                            userId = user.uid;
                        } else {
                            // Fallback for unexpected sign-out, although signInAnonymously should keep a session
                            userId = crypto.randomUUID(); 
                        }
                        anonymousName = getAnonymousName(userId);
                        console.log("Firebase Auth Ready. User ID:", userId, "Anon Name:", anonymousName);
                        
                        // Start listening to data once authenticated
                        if (currentPage === 'journal') setupJournalListeners();
                        renderPage(); // Rerender to show the session ID
                    });

                } catch (error) {
                    console.error("Firebase Initialization Error:", error);
                    showStatus("Error connecting to the Safe Space server.", 5000);
                    userId = 'error-session';
                    db = null; 
                }
            }

            // --- Real-time Data Listeners ---

            /** Attaches listeners for entries and their comments */
            const setupJournalListeners = () => {
                if (!db || entrySnapshotUnsubscribe) return; // Prevent multiple listeners

                const entriesRef = collection(db, `/artifacts/${__app_id}/public/data/journal_entries`);
                const entriesQuery = query(entriesRef, orderBy('timestamp', 'desc'));

                // 1. Listen for Public Entries
                entrySnapshotUnsubscribe = onSnapshot(entriesQuery, (snapshot) => {
                    publicEntries = snapshot.docs.map(doc => ({
                        id: doc.id,
                        ...doc.data()
                    }));
                    
                    // 2. Manage Comment Listeners for all visible entries
                    const currentEntryIds = publicEntries.map(e => e.id);
                    const activeCommentIds = Object.keys(commentUnsubscribes);

                    // Stop listeners for entries that were removed
                    activeCommentIds.forEach(id => {
                        if (!currentEntryIds.includes(id)) {
                            commentUnsubscribes[id](); // Unsubscribe
                            delete commentUnsubscribes[id];
                            delete commentsCache[id];
                        }
                    });

                    // Start listeners for new entries
                    currentEntryIds.forEach(id => {
                        if (!activeCommentIds.includes(id)) {
                            startCommentListener(id);
                        }
                    });

                    renderPage();
                }, (error) => {
                    console.error("Error listening to entries:", error);
                });
            };

            /** Attaches a listener for comments on a specific entry */
            const startCommentListener = (entryId) => {
                const commentsRef = collection(db, `/artifacts/${__app_id}/public/data/journal_entries/${entryId}/comments`);
                const commentsQuery = query(commentsRef, orderBy('timestamp', 'asc'));

                commentUnsubscribes[entryId] = onSnapshot(commentsQuery, (snapshot) => {
                    commentsCache[entryId] = snapshot.docs.map(doc => ({
                        id: doc.id,
                        ...doc.data()
                    }));
                    renderPage(); // Rerender the journal page whenever comments update
                }, (error) => {
                    console.error(`Error listening to comments for ${entryId}:`, error);
                });
            };

            /** Stops all Firestore listeners */
            const cleanupJournalListeners = () => {
                if (entrySnapshotUnsubscribe) {
                    entrySnapshotUnsubscribe();
                    entrySnapshotUnsubscribe = null;
                }
                Object.values(commentUnsubscribes).forEach(unsubscribe => unsubscribe());
                commentUnsubscribes = {};
                publicEntries = [];
                commentsCache = {};
            };


            // --- Component Render Functions ---

            // A. Homepage
            const renderHome = () => `
                <div class="text-center">
                    <h2 class="text-3xl sm:text-4xl font-semibold text-gray-700 mb-4 leading-snug">
                        Welcome. <br>Take a deep breath.
                    </h2>
                    <p class="text-base sm:text-lg text-gray-600 max-w-md mx-auto mb-10">
                        This is a judgment-free zone where your feelings are valid. We're here to listen, support, and help you find a moment of peace.
                    </p>

                    <!-- "Write your feelings" quick-access button -->
                    <button onclick="window.app.navigateTo('journal')"
                            class="w-full sm:w-2/3 px-8 py-5 bg-calm-teal hover:bg-dark-teal text-white text-xl sm:text-2xl font-bold rounded-full shadow-lg hover:shadow-xl transition-all duration-300 transform hover:scale-105">
                        Write Your Feelings
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 inline-block ml-3 -mt-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                            <path stroke-linecap="round" stroke-linejoin="round" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
                        </svg>
                    </button>
                </div>
            `;

            // B. Anonymous Community/Journal Page
            
            const handleJournalSave = async () => {
                const entryText = document.getElementById('journal-input').value.trim();

                if (!entryText) {
                    showStatus("Please write something before posting.");
                    return;
                }
                if (!db || userId === 'error-session' || userId === 'loading...') {
                    showStatus("Server not ready. Your entry was NOT saved. Please try again later.", 5000);
                    return;
                }
                
                const saveButton = document.getElementById('save-btn');
                saveButton.disabled = true;
                saveButton.textContent = 'Posting...';

                try {
                    // Save to the public collection
                    const publicRef = collection(db, `/artifacts/${__app_id}/public/data/journal_entries`);
                    
                    await addDoc(publicRef, {
                        content: entryText,
                        authorName: anonymousName,
                        authorId: userId,
                        timestamp: serverTimestamp(),
                    });

                    document.getElementById('journal-input').value = '';
                    showStatus("Journal entry posted anonymously to the community!", 3000);

                } catch (e) {
                    console.error("Error adding document: ", e);
                    showStatus("Error posting entry.", 5000);
                } finally {
                    saveButton.disabled = false;
                    saveButton.textContent = 'Post Entry Anonymously';
                }
            };

            const handleCommentPost = async (entryId) => {
                const inputId = `comment-input-${entryId}`;
                const commentText = document.getElementById(inputId).value.trim();

                if (!commentText) {
                    showStatus("Please write a comment.");
                    return;
                }

                if (!db) {
                    showStatus("Server not ready. Comment not saved.", 5000);
                    return;
                }

                const postButton = document.getElementById(`comment-btn-${entryId}`);
                postButton.disabled = true;
                postButton.textContent = 'Sending...';

                try {
                    // Reference the subcollection of the specific public entry
                    const commentsRef = collection(db, `/artifacts/${__app_id}/public/data/journal_entries/${entryId}/comments`);
                    
                    await addDoc(commentsRef, {
                        content: commentText,
                        authorName: anonymousName, // Use the current user's anonymous name
                        authorId: userId,
                        timestamp: serverTimestamp(),
                    });

                    document.getElementById(inputId).value = '';
                    showStatus("Comment posted!", 3000);

                } catch (e) {
                    console.error("Error adding comment: ", e);
                    showStatus("Error posting comment.", 5000);
                } finally {
                    postButton.disabled = false;
                    postButton.textContent = 'Reply';
                }
            };
            
            const renderEntry = (entry) => {
                const comments = commentsCache[entry.id] || [];

                return `
                    <div class="bg-soft-blue p-4 rounded-xl shadow-lg mb-6 border-b-4 border-calm-teal/70">
                        <!-- Entry Content -->
                        <p class="text-sm font-semibold text-dark-teal mb-2">${entry.authorName} says:</p>
                        <p class="text-gray-700 whitespace-pre-wrap mb-4">${entry.content}</p>
                        <p class="text-xs text-gray-500">${formatTimestamp(entry.timestamp)}</p>

                        <!-- Comments Section -->
                        <div class="mt-4 pt-3 border-t border-calm-teal/30 space-y-3">
                            <h4 class="text-xs font-bold text-gray-600">${comments.length} anonymous replies</h4>
                            
                            <!-- Display Comments -->
                            ${comments.map(comment => `
                                <div class="bg-white p-3 rounded-lg shadow-inner border-l-2 border-gray-300">
                                    <p class="text-xs font-medium text-dark-teal mb-1">${comment.authorName}</p>
                                    <p class="text-sm text-gray-700">${comment.content}</p>
                                    <p class="text-xs text-gray-400 mt-1 text-right">${formatTimestamp(comment.timestamp)}</p>
                                </div>
                            `).join('')}

                            <!-- Comment Input Form -->
                            <div class="pt-2">
                                <textarea id="comment-input-${entry.id}" 
                                          class="w-full p-2 border border-gray-300 rounded-lg text-sm resize-none focus:border-calm-teal" 
                                          rows="2" 
                                          placeholder="Type your supportive reply anonymously..."></textarea>
                                <button id="comment-btn-${entry.id}" 
                                        onclick="window.app.handleCommentPost('${entry.id}')"
                                        class="mt-2 px-4 py-2 bg-calm-teal hover:bg-dark-teal text-white text-sm font-semibold rounded-lg transition duration-200 disabled:opacity-50">
                                    Reply
                                </button>
                            </div>
                        </div>
                    </div>
                `;
            };

            const renderJournal = () => {
                window.app.handleJournalSave = handleJournalSave;
                window.app.handleCommentPost = handleCommentPost;
                
                return `
                    <div class="flex flex-col h-full">
                        <h2 class="text-3xl font-bold text-dark-teal mb-2">Community Safe Space</h2>
                        <p class="text-sm text-gray-500 mb-6">You are posting as: <span class="font-mono text-sm p-1 bg-soft-blue rounded text-calm-teal font-bold">${anonymousName}</span>. Your unique User ID is: ${userId}</p>
                        
                        <!-- New Entry Form -->
                        <div class="mb-8 p-4 bg-gray-50 rounded-xl shadow-lg">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Share Your Feelings Anonymously</h3>
                            <textarea id="journal-input" 
                                class="w-full h-24 p-3 border-2 border-calm-teal/50 rounded-xl focus:border-calm-teal focus:ring focus:ring-calm-teal/20 transition duration-150 resize-none" 
                                placeholder="What's on your mind? Post your entry for anonymous support..."></textarea>
                            <button id="save-btn" onclick="window.app.handleJournalSave()"
                                    class="w-full mt-3 px-6 py-3 bg-calm-teal hover:bg-dark-teal text-white text-lg font-semibold rounded-full shadow-md transition duration-200 disabled:opacity-50 disabled:cursor-not-allowed">
                                Post Entry Anonymously
                            </button>
                        </div>

                        <!-- Community Feed -->
                        <h3 class="font-bold text-xl text-dark-teal mb-4 border-b pb-2">Anonymous Entries Feed (${publicEntries.length} posts)</h3>
                        <div id="entries-feed" class="flex-grow space-y-4">
                            ${publicEntries.length > 0 
                                ? publicEntries.map(renderEntry).join('') 
                                : '<p class="text-center text-gray-500">No entries yet. Be the first to share your feelings!</p>'
                            }
                        </div>
                    </div>
                `;
            };

            // C. Mood Check-In Page (No change)
            const handleMoodCheckIn = (mood) => {
                const messageEl = document.getElementById('mood-message');
                let message = "";
                let colorClass = "";

                if (mood === 'terrible') {
                    message = "It's okay to feel terrible. Try the 5-4-3-2-1 Grounding Tool under 'Self-Help Tools'. You're not alone.";
                    colorClass = "bg-red-100 border-red-500";
                } else if (mood === 'bad') {
                    message = "Feeling bad is part of being human. Take a moment to focus on your breathing with the Breathing Timer.";
                    colorClass = "bg-orange-100 border-orange-500";
                } else if (mood === 'neutral') {
                    message = "A neutral day. Great job checking in! How about sharing a quick thought in the community feed?";
                    colorClass = "bg-yellow-100 border-yellow-500";
                } else if (mood === 'good') {
                    message = "That's wonderful! Carry this feeling by reading a positive affirmation.";
                    colorClass = "bg-green-100 border-green-500";
                } else if (mood === 'great') {
                    message = "Fantastic! Spread that energy. Remember this moment and share some positivity in the community.";
                    colorClass = "bg-teal-100 border-teal-500";
                }

                messageEl.innerHTML = `<p class="font-semibold mb-2">Thank you for checking in!</p><p>${message}</p>`;
                messageEl.className = `p-4 border-l-4 rounded-lg shadow-inner mt-6 text-gray-700 transition duration-300 ${colorClass}`;
                messageEl.classList.remove('hidden');
                
                showStatus(`Checked in as ${mood}!`);
            };

            const renderMood = () => {
                window.app.handleMoodCheckIn = handleMoodCheckIn;
                return `
                    <div class="text-center">
                        <h2 class="text-3xl font-bold text-dark-teal mb-8">How Are You Feeling Today?</h2>
                        
                        <div class="flex justify-around items-center space-x-2">
                            ${[
                                { mood: 'terrible', emoji: '😭', color: 'text-red-500' },
                                { mood: 'bad', emoji: '😞', color: 'text-orange-500' },
                                { mood: 'neutral', emoji: '😐', color: 'text-yellow-500' },
                                { mood: 'good', emoji: '😊', color: 'text-green-500' },
                                { mood: 'great', emoji: '🥳', color: 'text-calm-teal' }
                            ].map(item => `
                                <button onclick="window.app.handleMoodCheckIn('${item.mood}')"
                                        class="flex flex-col items-center p-3 sm:p-4 rounded-xl hover:bg-soft-blue transition duration-200 transform hover:scale-105 focus:outline-none focus:ring-2 focus:ring-calm-teal">
                                    <span class="text-4xl sm:text-5xl">${item.emoji}</span>
                                    <span class="mt-2 text-gray-600 font-medium text-xs sm:text-sm capitalize">${item.mood}</span>
                                </button>
                            `).join('')}
                        </div>

                        <!-- Grounding/Response Message -->
                        <div id="mood-message" class="hidden text-left mt-6"></div>
                    </div>
                `;
            };
            
            // D. Resource Library Page (No change)
            const renderResources = () => `
                <div>
                    <h2 class="text-3xl font-bold text-dark-teal mb-4">Resource Library</h2>
                    <p class="text-gray-600 mb-6">Find professional help, useful content, and guidance.</p>

                    <div class="space-y-6">
                        <!-- Helplines -->
                        <div class="p-4 bg-soft-blue rounded-xl shadow-md">
                            <h3 class="font-bold text-xl text-dark-teal mb-2">24/7 National Helplines (PH)</h3>
                            <ul class="space-y-1 text-gray-700 text-sm">
                                <li class="font-medium"><span class="font-mono text-calm-teal">•</span> National Crisis Hotline: <a href="tel:0917-899-8727" class="underline">0917-899-8727</a></li>
                                <li class="font-medium"><span class="font-mono text-calm-teal">•</span> NCMH Crisis Hotline: <a href="tel:0917-899-8727" class="underline">(02) 8989-8727</a></li>
                            </ul>
                        </div>

                        <!-- Videos & Articles -->
                        <div class="p-4 bg-gray-50 rounded-xl shadow-md">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Self-Care Guides</h3>
                            <ul class="space-y-3 text-sm">
                                <li>
                                    <a href="#" class="text-calm-teal hover:underline font-medium">Video: 5 Ways to Manage Anxiety in Class <span class="text-xs text-gray-400">(Placeholder)</span></a>
                                    <p class="text-xs text-gray-500">Short animated guide on simple coping mechanisms.</p>
                                </li>
                                <li>
                                    <a href="#" class="text-calm-teal hover:underline font-medium">Article: The Power of Journaling for Stress Relief <span class="text-xs text-gray-400">(Placeholder)</span></a>
                                    <p class="text-xs text-gray-500">Tips for starting and maintaining a reflective practice.</p>
                                </li>
                            </ul>
                        </div>

                        <!-- Tips Section -->
                         <div class="p-4 bg-gray-50 rounded-xl shadow-md">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Quick Tips: Academics</h3>
                            <ul class="list-disc list-inside text-gray-700 space-y-1 text-sm">
                                <li>Break large tasks into small, manageable chunks.</li>
                                <li>Schedule specific 'worry time' to contain anxiety.</li>
                                <li>Prioritize sleep over an extra hour of study.</li>
                            </ul>
                        </div>
                    </div>
                </div>
            `;
            
            // E. Self-Help Tools Page (No change)
            const groundingActivities = [
                { title: "5-4-3-2-1 Grounding", desc: "List 5 things you can see, 4 things you can touch, 3 things you can hear, 2 things you can smell, and 1 thing you can taste." },
                { title: "Soles of Your Feet", desc: "Press the soles of your feet firmly into the ground. Focus completely on the physical sensation." },
                { title: "Ice Cube Anchor", desc: "Hold an ice cube in your hand. Focus only on the cold, sharp sensation until it grounds you in the present." }
            ];

            const startBreathingTimer = () => {
                const circle = document.getElementById('breathing-circle');
                const instruction = document.getElementById('breathing-instruction');
                const startBtn = document.getElementById('start-breathing-btn');

                if (breathingTimer) {
                    clearInterval(breathingTimer);
                    breathingTimer = null;
                    circle.className = 'breathing-circle mx-auto my-6';
                    instruction.textContent = 'Timer Stopped. Click Start to begin.';
                    startBtn.textContent = 'Start Breathing Timer (4s-4s-6s)';
                    return;
                }

                startBtn.textContent = 'Stop Timer';

                let step = 0;
                
                const cycle = () => {
                    if (step === 0) {
                        instruction.textContent = 'INHALE deeply (4)';
                        circle.classList.add('breathe-in');
                        step = 1;
                        setTimeout(cycle, 4000);
                    } 
                    else if (step === 1) {
                        instruction.textContent = 'HOLD your breath (4)';
                        step = 2;
                        setTimeout(cycle, 4000);
                    }
                    else if (step === 2) {
                        instruction.textContent = 'EXHALE slowly (6)';
                        circle.classList.remove('breathe-in');
                        step = 0; // Reset
                        setTimeout(cycle, 6000);
                    }
                };
                
                cycle();
                breathingTimer = setInterval(cycle, 14000); 
            };
            
            const generateAffirmation = () => {
                const affirmationEl = document.getElementById('current-affirmation');
                
                const affirmation = AFFIRMATIONS[currentAffirmationIndex];
                currentAffirmationIndex = (currentAffirmationIndex + 1) % AFFIRMATIONS.length;

                affirmationEl.textContent = affirmation;
            };

            const renderTools = () => {
                if (breathingTimer) {
                    clearInterval(breathingTimer);
                    breathingTimer = null;
                }

                window.app.startBreathingTimer = startBreathingTimer;
                window.app.generateAffirmation = generateAffirmation;
                
                setTimeout(generateAffirmation, 0);

                return `
                    <div>
                        <h2 class="text-3xl font-bold text-dark-teal mb-6">Self-Help Tools</h2>
                        
                        <!-- Breathing Timer -->
                        <div class="mb-8 p-4 bg-soft-blue rounded-xl shadow-md text-center">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Breathing Timer</h3>
                            <p id="breathing-instruction" class="text-lg text-gray-700 mb-4 font-semibold">Click Start to begin.</p>
                            
                            <div id="breathing-circle" class="breathing-circle mx-auto my-6"></div>

                            <button id="start-breathing-btn" onclick="window.app.startBreathingTimer()"
                                    class="px-6 py-2 bg-calm-teal hover:bg-dark-teal text-white font-medium rounded-full transition duration-200">
                                Start Breathing Timer (4s-4s-6s)
                            </button>
                        </div>
                        
                        <!-- Affirmation Generator -->
                        <div class="mb-8 p-4 bg-gray-50 rounded-xl shadow-md text-center">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Affirmation Generator</h3>
                            <p id="current-affirmation" class="text-lg font-medium text-gray-700 h-10 flex items-center justify-center italic">
                                Loading a positive thought...
                            </p>
                            <button onclick="window.app.generateAffirmation()"
                                    class="mt-4 px-6 py-2 border border-calm-teal text-calm-teal hover:bg-calm-teal hover:text-white font-medium rounded-full transition duration-200">
                                Next Affirmation
                            </button>
                        </div>

                        <!-- Grounding Activities -->
                        <div class="p-4 bg-gray-50 rounded-xl shadow-md">
                            <h3 class="font-bold text-xl text-dark-teal mb-3">Grounding Activities</h3>
                            <ul class="space-y-4">
                                ${groundingActivities.map(activity => `
                                    <li class="border-b pb-2 last:border-b-0">
                                        <p class="font-semibold text-gray-800">${activity.title}</p>
                                        <p class="text-sm text-gray-600">${activity.desc}</p>
                                    </li>
                                `).join('')}
                            </ul>
                        </div>

                    </div>
                `;
            };

            // --- Router and Main Render Logic ---

            const navigateTo = (page) => {
                // Cleanup listeners when navigating away from the Journal/Community page
                if (currentPage === 'journal' && page !== 'journal') {
                    cleanupJournalListeners();
                }

                // Stop timer if navigating away from the tools page
                if (page !== 'tools' && breathingTimer) {
                    clearInterval(breathingTimer);
                    breathingTimer = null;
                }

                currentPage = page;
                renderPage();
                
                // Setup listeners when navigating TO the Journal/Community page
                if (currentPage === 'journal' && db && userId !== 'loading...') {
                    setupJournalListeners();
                }
            };

            const renderPage = () => {
                let htmlContent = '';
                switch (currentPage) {
                    case 'journal':
                        htmlContent = renderJournal();
                        break;
                    case 'mood':
                        htmlContent = renderMood();
                        break;
                    case 'resources':
                        htmlContent = renderResources();
                        break;
                    case 'tools':
                        htmlContent = renderTools();
                        break;
                    case 'home':
                    default:
                        htmlContent = renderHome();
                        break;
                }
                contentEl.innerHTML = htmlContent;
                window.scrollTo(0, 0); // Scroll to top on page change
            };

            // Expose public API methods to the window object
            window.app = {
                navigateTo,
                handleJournalSave,
                handleCommentPost,
                handleMoodCheckIn,
                startBreathingTimer,
                generateAffirmation
            };

            // Start the application
            window.addEventListener('load', () => {
                initFirebase();
                renderPage();
            });
        }
    </script>
</body>
</html>

