<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Notes App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* Light gray background */
        }
    </style>
</head>
<body class="min-h-screen flex flex-col items-center py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-4xl w-full bg-white p-6 sm:p-8 rounded-xl shadow-lg">
        <h1 class="text-3xl sm:text-4xl font-bold text-gray-800 text-center mb-8">My Notes</h1>

        <div class="mb-8">
            <input
                type="text"
                id="search-input"
                placeholder="Search notes by title or content..."
                class="w-full p-3 sm:p-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500 transition duration-200 ease-in-out text-gray-700"
            />
        </div>

        <div class="mb-10 p-6 bg-blue-50 border border-blue-200 rounded-xl shadow-md">
            <h2 class="text-2xl font-semibold text-blue-700 mb-4">Create New Note</h2>
            <input
                type="text"
                id="note-title-input"
                placeholder="Note Title"
                class="w-full p-3 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:ring-2 focus:ring-blue-400 text-gray-700"
            />
            <textarea
                id="note-content-input"
                placeholder="Write your note here..."
                rows="5"
                class="w-full p-3 border border-gray-300 rounded-lg mb-4 focus:outline-none focus:ring-2 focus:ring-blue-400 text-gray-700 resize-y"
            ></textarea>
            <button
                id="add-note-button"
                class="w-full bg-blue-600 hover:bg-blue-700 text-white font-semibold py-3 px-6 rounded-lg shadow-md transition duration-300 ease-in-out transform hover:scale-105"
            >
                Add Note
            </button>
        </div>

        <div id="notes-container" class="grid gap-6 sm:grid-cols-2 lg:grid-cols-3">
            </div>
    </div>

    <script>
        // Get references to DOM elements
        const searchInput = document.getElementById('search-input');
        const noteTitleInput = document.getElementById('note-title-input');
        const noteContentInput = document.getElementById('note-content-input');
        const addNoteButton = document.getElementById('add-note-button');
        const notesContainer = document.getElementById('notes-container');

        // Initialize notes array from localStorage or as an empty array
        let notes = JSON.parse(localStorage.getItem('notes')) || [];

        /**
         * Saves the current notes array to localStorage.
         */
        function saveNotes() {
            localStorage.setItem('notes', JSON.stringify(notes));
        }

        /**
         * Renders the notes to the DOM.
         * @param {Array} notesToRender - The array of notes to display.
         */
        function renderNotes(notesToRender) {
            notesContainer.innerHTML = ''; // Clear existing notes
            if (notesToRender.length === 0) {
                notesContainer.innerHTML = `
                    <p class="col-span-full text-center text-gray-500 text-lg py-8">
                        No notes found. Start by creating a new one!
                    </p>
                `;
                return;
            }

            notesToRender.forEach(note => {
                // Create a div for each note
                const noteCard = document.createElement('div');
                noteCard.id = `note-${note.id}`; // Assign a unique ID for easy targeting
                noteCard.className = 'bg-white p-5 rounded-xl shadow-md border border-gray-200 flex flex-col justify-between transition duration-300 ease-in-out transform hover:scale-102';

                // Format the timestamp for display
                const date = new Date(note.timestamp);
                const formattedDate = date.toLocaleString('en-US', {
                    year: 'numeric',
                    month: 'short',
                    day: 'numeric',
                    hour: '2-digit',
                    minute: '2-digit'
                });

                noteCard.innerHTML = `
                    <div>
                        <h3 class="text-xl font-semibold text-gray-800 mb-2">${note.title}</h3>
                        <p class="text-gray-600 text-sm mb-4">${note.content}</p>
                    </div>
                    <div class="flex justify-between items-center text-gray-500 text-xs mt-auto pt-2 border-t border-gray-100">
                        <span>${formattedDate}</span>
                        <button
                            data-id="${note.id}"
                            class="delete-note-button bg-red-500 hover:bg-red-600 text-white text-xs font-medium py-1.5 px-3 rounded-md transition duration-200 ease-in-out transform hover:scale-105"
                        >
                            Delete
                        </button>
                    </div>
                `;
                notesContainer.appendChild(noteCard);
            });

            // Add event listeners to all new delete buttons
            document.querySelectorAll('.delete-note-button').forEach(button => {
                button.addEventListener('click', (event) => {
                    const noteIdToDelete = event.target.dataset.id;
                    deleteNote(noteIdToDelete);
                });
            });
        }

        /**
         * Adds a new note to the notes array and updates the display.
         */
        function addNote() {
            const title = noteTitleInput.value.trim();
            const content = noteContentInput.value.trim();

            if (title === '' || content === '') {
                alert('Please enter both a title and content for your note.'); // Using alert for simplicity, consider a custom modal for production
                return;
            }

            // Create a unique ID for the new note
            const newNote = {
                id: Date.now().toString(), // Simple unique ID based on timestamp
                title: title,
                content: content,
                timestamp: new Date().toISOString() // Store timestamp for sorting/display
            };

            notes.unshift(newNote); // Add new note to the beginning of the array
            saveNotes(); // Save to localStorage
            renderNotes(notes); // Re-render all notes

            // Clear input fields
            noteTitleInput.value = '';
            noteContentInput.value = '';
        }

        /**
         * Deletes a note from the notes array and updates the display.
         * @param {string} id - The ID of the note to delete.
         */
        function deleteNote(id) {
            // Filter out the note with the matching ID
            notes = notes.filter(note => note.id !== id);
            saveNotes(); // Save updated notes to localStorage
            renderNotes(notes); // Re-render the notes
        }

        /**
         * Filters and renders notes based on the search input.
         */
        function searchNotes() {
            const searchTerm = searchInput.value.toLowerCase();
            const filteredNotes = notes.filter(note =>
                note.title.toLowerCase().includes(searchTerm) ||
                note.content.toLowerCase().includes(searchTerm)
            );
            renderNotes(filteredNotes);
        }

        // Event Listeners
        addNoteButton.addEventListener('click', addNote);
        searchInput.addEventListener('input', searchNotes);

        // Initial render of notes when the page loads
        document.addEventListener('DOMContentLoaded', () => {
            renderNotes(notes);
        });
    </script>
</body>
</html>
